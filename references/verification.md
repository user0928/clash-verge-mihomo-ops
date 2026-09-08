# Verification and rollback

## Read-only discovery

Use PowerShell to find the process and configuration directory, then inspect the persistent and generated files:

```powershell
Get-Process | Where-Object { $_.ProcessName -match 'clash|mihomo|verge' } |
  Select-Object ProcessName,Id,Path

$cfgDir = Join-Path $env:APPDATA 'io.github.clash-verge-rev.clash-verge-rev'
Get-ChildItem -LiteralPath $cfgDir -Force
Get-ChildItem -LiteralPath (Join-Path $cfgDir 'profiles') -File -Force
```

Read the active selection before choosing a profile file. Treat generated `clash-verge.yaml` and any check file as runtime evidence, not as the persistent source.

## Syntax validation

Find the bundled mihomo executable instead of assuming a path, then validate the exact generated candidate:

```powershell
& $mihomo -t -f $generatedConfig
```

The useful success evidence is mihomo's configuration-test success for the file that will actually run. A YAML parser check alone is not sufficient because rule semantics and mihomo fields can still be invalid.

## Windows and live state

Check the real adapter and System Proxy state:

```powershell
Get-NetAdapter | Where-Object { $_.Name -match 'Meta|Clash|mihomo|TUN' } |
  Select-Object Name,Status,MacAddress,LinkSpeed

Get-NetIPInterface | Where-Object { $_.InterfaceAlias -match 'Meta|Clash|mihomo|TUN' } |
  Select-Object InterfaceAlias,NlMtu,ConnectionState

Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' |
  Select-Object ProxyEnable,ProxyServer,AutoConfigURL
```

The desired steady state is TUN adapter up, MTU 1500 for the verified baseline, and `ProxyEnable` false. A running `mihomo` process after closing the Verge window means TUN may still be active.

## Traffic evidence

- Check rule logs for an OpenAI/Codex host reaching the selected proxy group and an ordinary host reaching `DIRECT`.
- Test the named application itself. For Bilibili, check the API and media path; for WeChat, actually send an image and a small file; for Codex, start it through more than one path without toggling System Proxy.
- A 403, 400, or 405 can show that an endpoint was reached but cannot prove authentication, playback, or transfer success.
- For an app-specific issue, compare TUN on and TUN truly off. Closing the UI is not the same as disabling the virtual adapter.

## Backup and rollback

Before writing, make a timestamped copy of every persistent file touched. On rollback, restore only the corresponding explicit file after stopping/restarting Clash Verge as needed, then rerun syntax and runtime checks. Never use recursive deletion or broad cleanup to roll back a configuration.

## Reporting format

Report:

1. What changed and in which persistent file.
2. What was deliberately preserved.
3. Backup location.
4. Generated-runtime and mihomo syntax results.
5. Live adapter/System Proxy state.
6. Real traffic evidence and any remaining user action.
