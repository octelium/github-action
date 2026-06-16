# Octelium GitHub Action

Connect to your [Octelium](https://octelium.com) Cluster from GitHub Actions workflows and reach its Services over the established tunnel.

The action installs the Octelium client on the runner, connects to your Cluster in the background, and keeps the connection available for the remaining steps in the same job. It supports both secret-less authentication using GitHub's OIDC identity token and authentication tokens stored as secrets.

## Quick start

Secret-less, using GitHub's OIDC assertion:

```yaml
permissions:
  id-token: write # required for the OIDC assertion

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Connect to Octelium
        uses: octelium/github-action@v1
        with:
          domain: example.com

      - name: Reach a Service over the tunnel
        run: curl http://demo-nginx
```

## Authentication

The action authenticates in one of two ways.

### GitHub OIDC assertion (default, secret-less)

When no `auth-token` is provided, the action authenticates using GitHub's own OIDC identity token through Octelium's `github-actions` assertion. This requires no secrets, but your workflow (or the job) must grant the `id-token: write` permission so the runner can mint the OIDC token:

```yaml
permissions:
  id-token: write

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: octelium/github-action@v1
        with:
          domain: example.com
```

Your Cluster must have a matching OIDC assertion IdentityProvider configured. Read more about OIDC assertions [here](https://octelium.com/docs/octelium/latest/management/core/identity-providers#oidc-assertion).

You can override the assertion audience with the `audience` input:

```yaml
      - uses: octelium/github-action@v1
        with:
          domain: example.com
          audience: my-custom-aud
```

### Authentication token (secret)

Provide an authentication token via the `auth-token` input, typically from a repository, organization, or environment secret. When `auth-token` is set it takes precedence and the OIDC assertion is not used, so the `id-token: write` permission is not needed:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: octelium/github-action@v1
        with:
          domain: example.com
          auth-token: ${{ secrets.OCTELIUM_AUTH_TOKEN }}
```

The token is only ever passed through an environment variable, never on the command line, and GitHub masks registered secret values in logs. Note that secrets are not available to workflows triggered by `pull_request` from forked repositories; use the OIDC assertion path for those.

## Inputs

| Input          | Required | Default   | Description                                                                                  |
| -------------- | -------- | --------- | -------------------------------------------------------------------------------------------- |
| `domain`       | yes      |           | Your Octelium Cluster domain.                                                                |
| `auth-token`   | no       |           | Authentication token. If set, it is used instead of the GitHub OIDC assertion.              |
| `audience`     | no       |           | Override the audience of the GitHub Actions OIDC assertion.                                  |
| `args`         | no       |           | Extra arguments passed to `octelium connect`, separated by whitespace.                       |
| `wait`         | no       | `4`       | Seconds to wait after starting the connection before continuing.                            |
| `command`      | no       | `connect` | The operation to run: `connect` or `logout`.                                                |
| `dev`          | no       |           | Enable dev mode. Intended for development and testing.                                       |
| `insecure-tls` | no       |           | Disable TLS certificate verification. For testing only; do not use against production.      |

## Passing extra connect flags

Use the `args` input to pass any additional flags to `octelium connect`. For example, to publish Services to host ports, or to serve Services:

```yaml
      - uses: octelium/github-action@v1
        with:
          domain: example.com
          args: "--publish svc1:8080 --publish svc2.ns1:3000"
```

```yaml
      - uses: octelium/github-action@v1
        with:
          domain: example.com
          args: "--serve svc1 --serve svc2.ns1 --no-dns"
```

The value is split on whitespace into separate arguments. Refer to `octelium connect --help` for the full list of flags.

## Cleaning up

The connection runs in the background for the rest of the job. To tear it down explicitly, call the action again with `command: logout`. Using `if: always()` ensures cleanup runs even if a previous step failed:

```yaml
      - name: Logout
        if: always()
        uses: octelium/github-action@v1
        with:
          domain: example.com
          command: logout
```

The logout step must run in the same job as the connection. Because this is a composite action it cannot run cleanup automatically after the job, so an explicit logout step is the way to do it. The connection is in any case scoped to a single job and does not carry over to other jobs.

## Full example

```yaml
name: integration-tests
on: push

permissions:
  id-token: write

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Connect to Octelium
        uses: octelium/github-action@v1
        with:
          domain: example.com

      - name: Access an Octelium Service
        run: curl http://demo-nginx

      - name: Logout
        if: always()
        uses: octelium/github-action@v1
        with:
          domain: example.com
          command: logout
```

## Platform support

Linux and macOS runners are supported. Windows support is experimental.