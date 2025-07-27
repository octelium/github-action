# Octelium GitHub Action

This is a GitHub action to connect to your Octelium _Cluster_ and access its _Services_.

You can connect using an authentication token by setting it as a secret in your repository. Here is an example:

```yaml
- name: Octelium
    uses: octelium/github-action@master
    with:
      domain: example.com
      auth-token: ${{ secrets.OCTELIUM_AUTH_TOKEN }}
```

You can also use extra flags for the `octelium connect` command via the `args` input as follows:

```yaml
- name: Octelium
    uses: octelium/github-action@master
    with:
      domain: example.com
      auth-token: ${{ secrets.OCTELIUM_AUTH_TOKEN }}
      args: "--serve svc1 --no-dns"
```


You can also authenticate to your Octelium Cluster in a "secret-less" way using GitHub's own OIDC issued identity token assertions (read more [here](https://octelium.com/docs/octelium/latest/management/core/identity-providers#oidc-assertion)). Here is an example:


```yaml
- name: Octelium
    uses: octelium/github-action@master
    with:
      domain: example.com
      assertion-idp: <YOUR_IDP_NAME>
```
