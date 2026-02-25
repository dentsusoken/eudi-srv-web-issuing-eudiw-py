# EUDIW Issuer (dentsusoken fork)

This is a fork of [eudi-srv-web-issuing-eudiw-py](https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py).

For full documentation, see the [original README](https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/blob/main/README.md).

## How this project was created

Run from the vecrea-id repository root. Create the dentsusoken fork on GitHub first.

```bash
cd vecrea-id
git submodule add https://github.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py projects/eudi-srv-web-issuing-eudiw-py
cd projects/eudi-srv-web-issuing-eudiw-py
git remote rename origin upstream
git remote add origin https://github.com/dentsusoken/eudi-srv-web-issuing-eudiw-py
git fetch upstream
git checkout main
git reset --hard upstream/main
git push -u origin main
```

Note: `.gitmodules` in vecrea-id was later updated to point to the dentsusoken fork. The submodule references commits (e.g. this custom README) that exist only in our fork, not in the original. GitHub uses the `.gitmodules` URL to build the submodule link, so it must point to the fork. The fork is now [public](https://github.com/dentsusoken/eudi-srv-web-issuing-eudiw-py), so the link works for everyone.

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
