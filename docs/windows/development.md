# Development

## WSL

Install WSL from the GitHub MSI, then Arch Linux as the default distro. See [WSL — Arch](../wsl/arch.md#install) for the full setup.

```powershell
wsl --install archlinux
wsl --set-default archlinux
```

## Dotfiles

Dotfiles are managed with chezmoi. See the [dotfiles repo](https://github.com/hansmissenheim/dotfiles).

## mise

Runtime version manager used for language toolchains.

## Git

Global git configuration comes from the dotfiles repo (`~/.config/git/config`). It hands GitHub credentials to the GitHub CLI, so there's no separate Git Credential Manager setup; skip the `credential.helper manager` step Scoop's git suggests.

### GitHub CLI

Install with Scoop, then sign in with the token stored in gh's config file rather than Credential Manager:

```powershell
scoop install gh
gh auth login -h github.com --insecure-storage --git-protocol https --web
gh auth status
```

`--insecure-storage` keeps the token unencrypted in `%APPDATA%\GitHub CLI\hosts.yml`. It's needed because SSH sessions can't read Credential Manager (see [SSH troubleshooting](ssh.md#credential-manager-is-unavailable)), and a keyring login shows up there as an invalid token. Anything running as `hans` can read the file.
