# Octelium GitHub Action

This is a GitHub action to connect to your Octelium _Cluster_ and access its _Services_.

You can use it as follows:

```yaml
- name: Octelium
    uses: octelium/github-action@v0
    with:
      domain: example.com
      auth-token: ${{ secrets.OCTELIUM_AUTH_TOKEN }}
```