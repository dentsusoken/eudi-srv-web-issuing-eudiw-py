# EUDIW Issuer (dentsusoken fork)

This is a fork of [eudi-srv-web-issuing-eudiw-py](https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py).

For full documentation, see the [original README](https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/blob/main/README.md).

## Remote configuration

| Remote   | URL                                                       |
|----------|-----------------------------------------------------------|
| origin   | https://github.com/dentsusoken/eudi-srv-web-issuing-eudiw-py |
| upstream | https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py |

### Initial setup (first-time clone)

When you clone vecrea-id, the submodule only has `origin` configured. Add `upstream` before using the branch workflow:

```bash
cd projects/eudi-srv-web-issuing-eudiw-py
git remote add upstream https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py
```

## Working with branches

### Creating a new branch

```bash
cd projects/eudi-srv-web-issuing-eudiw-py
git fetch upstream
git checkout -b <branch-name> upstream/main
```

### Updating main from upstream

To sync `main` with the original repository:

```bash
cd projects/eudi-srv-web-issuing-eudiw-py
git checkout main
git fetch upstream
git rebase upstream/main
```

### Updating a branch (other than main) from upstream

To sync a branch with the latest upstream:

```bash
cd projects/eudi-srv-web-issuing-eudiw-py
git checkout <branch-name>
git fetch upstream
git rebase upstream/main
```
