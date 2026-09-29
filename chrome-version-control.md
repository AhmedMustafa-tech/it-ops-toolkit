# Chrome version control on Windows (PowerShell)

When a Chrome release breaks a business-critical web app, you need to **roll back and stop auto-updates**, then **undo it cleanly** once the vendor ships a fix. Push these through your MDM or RMM (JumpCloud commands, Intune scripts…) running as **System**.

Chrome's Google Update app GUID is `{8A69D345-D564-463C-AFF1-A69D9E530F96}`.

## 1. Pin Chrome to a major version and block updates

```powershell
$reg  = "HKLM:\SOFTWARE\Policies\Google\Update"
$guid = "{8A69D345-D564-463C-AFF1-A69D9E530F96}"
New-Item -Path $reg -Force | Out-Null

# Stay on major version 124 (change to what you need)
New-ItemProperty -Path $reg -Name "TargetVersionPrefix$guid" -Value "124." -PropertyType String -Force | Out-Null
# 0 = updates disabled for Chrome
New-ItemProperty -Path $reg -Name "Update$guid" -Value 0 -PropertyType DWord -Force | Out-Null
Write-Output "Chrome pinned to 124 and updates disabled"
```

> Pinning alone does **not** downgrade an installed newer version. Install the older build first, or enable `RollbackToTargetVersion$guid = 1` together with the prefix.

## 2. Remove the pin and re-enable updates

```powershell
$reg  = "HKLM:\SOFTWARE\Policies\Google\Update"
$guid = "{8A69D345-D564-463C-AFF1-A69D9E530F96}"
foreach ($name in "TargetVersionPrefix$guid","Update$guid","RollbackToTargetVersion$guid") {
  Remove-ItemProperty -Path $reg -Name $name -ErrorAction SilentlyContinue
}
Write-Output "Chrome update policy restored"
```

## 3. Check what a device is actually running

```powershell
(Get-Item "C:\Program Files\Google\Chrome\Application\chrome.exe").VersionInfo.ProductVersion
```

## Lessons

- **Your patch policy can undo your fix.** Check that your MDM isn't set to force "latest browser" on the same devices.
- Track affected users in one shared list, and confirm each one is on the target version before you close the incident.
- Write down in the RCA which version was actually at fault, and say "not confirmed" if the evidence is mixed.
