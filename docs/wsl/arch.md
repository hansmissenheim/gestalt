# Arch Linux

## Install

Run from an elevated PowerShell prompt.

### WSL

Install WSL itself from the latest stable [GitHub release](https://github.com/microsoft/WSL/releases) (3.0.3 at the time of writing) and turn on Virtual Machine Platform:

```powershell
$v = gh release view --repo microsoft/WSL --json tagName --jq .tagName
gh release download $v --repo microsoft/WSL --pattern "wsl.$v.0.x64.msi" --dir $env:TEMP --clobber
Get-AuthenticodeSignature "$env:TEMP\wsl.$v.0.x64.msi" | Select Status, @{n='Signer'; e={$_.SignerCertificate.Subject}}
msiexec /i "$env:TEMP\wsl.$v.0.x64.msi" /qn /norestart
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All -NoRestart
Restart-Computer
```

!!! note
    On a fresh Windows 11 install the built-in `wsl.exe` is only a stub. On build 26300 it answered every command except a bare `wsl --install` with "not installed", so `wsl --install --no-distribution` and `wsl --update` don't work. winget's `Microsoft.WSL` package also lagged behind (2.7.14 while GitHub had 3.0.3), which is why the MSI comes straight from the release page.

Check the version after the reboot:

```powershell
wsl --version
```

### Distro

```powershell
wsl --install archlinux --no-launch
wsl --set-default archlinux
wsl -d archlinux
```

This opens a root shell; the rest of the page runs inside it.

## Initial Setup

Root-level setup: password, user creation, wheel group, and `wsl.conf`.

```bash
passwd
pacman -Syu --needed sudo base-devel
useradd -m -G wheel -s /bin/bash hans
passwd hans
echo '%wheel ALL=(ALL:ALL) ALL' > /etc/sudoers.d/10-wheel
chmod 440 /etc/sudoers.d/10-wheel
visudo -c
```

### wsl.conf

`/etc/wsl.conf` turns on systemd and makes `hans` the login user:

```ini title="/etc/wsl.conf"
[boot]
systemd=true

[user]
default=hans
```

Apply it by restarting the distro from PowerShell:

```powershell
wsl --terminate archlinux
wsl -d archlinux
```

## Package Manager

The image initializes and populates the pacman keyring on first boot (`pacman-init.service`), so there's no manual `pacman-key --init`. The first full update happened in [Initial Setup](#initial-setup); later ones:

```bash
sudo pacman -Syu
```

If signature errors show up, re-seed the keyring:

```bash
sudo pacman-key --init
sudo pacman-key --populate archlinux
sudo pacman -Sy archlinux-keyring
```

## Dotfiles

Install chezmoi and apply the dotfiles repo.

```bash
```

## AUR Helper

Install paru for AUR package support.

```bash
```

## Development Tools

Additional tools installed via pacman or paru (mise, etc.).

```bash
```

## Locale

Set the system locale.

```bash
sudo sed -i 's/^#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen
echo 'LANG=en_US.UTF-8' | sudo tee /etc/locale.conf
```

## SSH

Copy SSH keys from Windows and configure the SSH server.

```bash
```
