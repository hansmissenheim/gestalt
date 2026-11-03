# SSH

## OpenSSH Server

Enable and configure the built-in Windows OpenSSH server. Run everything on this page from an elevated PowerShell prompt.

### Install

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server
```

### sshd_config

`C:\ProgramData\ssh\sshd_config` stays at the shipped default. The part that matters is its last block: members of the Administrators group read their keys from a shared file instead of `~\.ssh\authorized_keys` (see [Authorized Keys](#authorized-keys)).

```text
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

The default shell is also left alone, so a login lands in `cmd.exe`.

### Service

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

### Redirection Guard

Since the February 2026 updates (KB5074105 / KB5077181), `sshd.exe` runs with Windows' [Redirection Guard](https://www.microsoft.com/en-us/msrc/blog/2025/06/redirectionguard-mitigating-unsafe-junction-traversal-in-windows) enforced. An SSH session then refuses to follow any junction that wasn't created by an administrator, which includes every `apps\<app>\current` junction Scoop creates from a normal prompt. Every Scoop shim fails over SSH:

```text
Shim: Could not determine if target is a GUI app. Assuming console.
Shim: Could not create process with command '"C:\Users\Hans\AppData\Local\Scoop\apps\gh\current\bin\gh.exe" ...'.
```

Turn the mitigation off for `sshd.exe` only, backing up its current value first:

```powershell
$key = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sshd.exe'
reg export 'HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sshd.exe' "$HOME\sshd-ifeo-backup.reg" /y
Set-ItemProperty $key -Name MitigationOptions -Type Binary -Value ([byte[]]::new(19))
```

The stock value is 19 bytes ending in `10`; the last byte is what enables Redirection Guard. Each SSH connection starts a fresh `sshd.exe`, so new connections pick the change up without restarting the service. Restore the old value with `reg import` on the backup.

!!! warning
    Redirection Guard protects privileged processes from junctions planted by an unprivileged user. Disabling it for `sshd` is acceptable here because logins are key-only and the machine is single-user, but it is still a weaker setup than the default.

!!! note
    A Windows update or an OpenSSH reinstall can write the mitigation back. If shims break over SSH again, check the value first.

Upstream reports: [ScoopInstaller/Scoop#6594](https://github.com/ScoopInstaller/Scoop/issues/6594), [ScoopInstaller/Scoop#6612](https://github.com/ScoopInstaller/Scoop/issues/6612).

## Authorized Keys

`hans` is an administrator, so the key goes in `C:\ProgramData\ssh\administrators_authorized_keys`, not `~\.ssh\authorized_keys`. The file must be writable only by Administrators and SYSTEM, or sshd ignores it.

```powershell
Set-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value "<public key of the connecting machine>"
icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

## Firewall

Installing the server creates the `OpenSSH-Server-In-TCP` rule, enabled for the **Private** profile only. Wi-Fi connects as Public by default, so mark the home network Private:

```powershell
Set-NetConnectionProfile -InterfaceAlias "Wi-Fi" -NetworkCategory Private
```

Check that sshd is listening and the rule applies:

```powershell
netstat -an | findstr ":22"
Get-NetFirewallRule -Name OpenSSH-Server-In-TCP | Select Name, Enabled, Profile
```

## Troubleshooting

An SSH login is a network logon, and for an administrator it gets a fully elevated token without a UAC prompt. Two things behave differently from a local terminal because of that.

### Scoop shims fail

The `Shim: Could not create process` error means Redirection Guard blocked a junction. Confirm it in the mitigation log:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Security-Mitigations/KernelMode'; Id=32} -MaxEvents 5 | Format-List TimeCreated, Message
```

The lasting fix is [turning Redirection Guard off](#redirection-guard). As a stopgap, recreate the junctions from an elevated prompt, which marks them trusted until Scoop rewrites them from a normal prompt:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Scoop\apps\*\current" -Force | Where-Object LinkType -eq Junction | ForEach-Object {
    $link = $_.FullName
    $target = $_.Target -join ''
    attrib -R /L $link
    cmd /c rmdir "$link"
    cmd /c mklink /J "$link" "$target" | Out-Null
    attrib +R /L $link
}
```

!!! danger
    Remove a junction with `cmd /c rmdir`, never `Remove-Item`. Windows PowerShell 5.1's `Remove-Item` can follow the junction and delete the app's files.

### Credential Manager is unavailable

A key-based login can't unlock Windows Credential Manager, so tools that keep secrets there act as if they were never signed in (`gh` reports its token as invalid). Store their credentials in a file instead; see [GitHub CLI](development.md#github-cli).
