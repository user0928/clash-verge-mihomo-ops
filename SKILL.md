---
name: clash-verge-mihomo-ops
description: Use this skill whenever a Windows user mentions Clash, Clash Verge Rev, mihomo, TUN/网卡模式, fake-IP, 系统代理, OpenAI/Codex 分流, Google 代理, XMind 分流, 微信图片或文件传输, Bilibili 直播卡顿, or repeated reconnects such as “重连 5/5”. It audits and repairs persistent Clash routing safely, preserves unrelated strategy groups, and verifies the generated runtime configuration and real traffic. Prefer this skill even when the user asks only for an explanation, because the important distinction is between a launcher-only proxy and network-layer TUN routing.
---

# Clash Verge Rev + mihomo operations

Use this skill for Windows Clash Verge Rev / mihomo work. The goal is a durable, selective-routing setup: TUN captures the traffic path, rule mode decides which destinations use a proxy, and Windows System Proxy can remain off. This covers both diagnosis and implementation, but every change must be scoped, backed up, and verified against the generated configuration.

## Operating principles

- Treat the active profile and its persistent enhancement files as the source of truth. `clash-verge.yaml` is generated runtime output and is not enough as a durable edit target.
- Discover the active profile before editing. Do not hardcode remembered profile IDs, subscription URLs, node names, or user-specific secrets.
- Preserve Tailscale, DNS servers, node lists, strategy-group selections, and unrelated rules unless the user explicitly requests a change.
- Prefer a minimal prepend/merge change over rewriting a large subscription-generated file.
- Make a timestamped backup before every write. If the active directory is ACL-protected, copy to a workspace file, edit the copy, then write back the exact target after validating it.
- A file edit is not a live fix. Reapply the profile or restart Clash Verge, then inspect the newly generated configuration.
- If the controller at `127.0.0.1:9097` is unavailable, do not keep retrying the API. Restart Clash Verge and verify its generated files and process state instead.
- Never expose subscription URLs, proxy credentials, controller secrets, or full node definitions in reports, examples, commits, or GitHub repositories.

## First classify the request

### Reconnect 5/5 symptom

If every conversation first shows reconnects and only becomes usable after “5/5”, especially when the user is using the web surface and enabling Windows System Proxy temporarily makes it work, treat this as a traffic-path problem. A per-launch `HTTP_PROXY`/`HTTPS_PROXY` wrapper does not cover every desktop or web request and is not the durable baseline.

Use network-layer routing instead:

1. Keep Clash Verge running at login.
2. Enable TUN and automatic routing.
3. Set the global mode to `rule`.
4. Route OpenAI/Codex destinations to the existing ChatGPT/OpenAI proxy group.
5. Keep Windows System Proxy disabled so unrelated applications remain direct.
6. Test more than one launch path without manually toggling System Proxy.

The old `127.0.0.1:7890` launcher is historical context only. On this host the verified mixed port is `127.0.0.1:7897`; always discover the live port instead of assuming either value.

### Application-specific failure

Use the module in `references/routing-modules.md`. Add only the process/domain/fake-IP exception needed by the named application. Do not change a global reject group to fix one application.

### General audit or “网卡模式各项规则” request

Use the baseline and checklist in the references, then report each item separately: mode, mixed port, TUN stack, auto-route, strict-route, interface detection, DNS hijack, fake-IP, MTU, system proxy, autostart, rule order, syntax validation, and real traffic.

## Verified baseline to aim for

The exact profile and node names are host-specific, but the stable shape is:

```yaml
mode: rule
mixed-port: 7897
tun:
  enable: true
  stack: gvisor
  auto-route: true
  strict-route: false
  auto-detect-interface: true
  mtu: 1500
  dns-hijack:
    - any:53
```

Keep the existing `fake-ip` DNS design when it is already working. Add a narrowly scoped `fake-ip-filter` exception only when the target application needs real DNS behavior; do not replace the user's DNS servers from memory.

The verified steady state also has Windows System Proxy disabled, Clash Verge starting through its existing login task, and OpenAI/Codex domains mapped to the existing `ChatGPT` group. The detailed field-by-field checklist is in `references/baseline.md`.

## Persistent configuration workflow

1. Locate the running Clash Verge and mihomo processes, then locate the data directory. The known default is `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev`, but discovery wins.
2. Read the UI/state file, generator defaults, active profile selection, enhancement files, and generated runtime YAML. Identify which file owns each desired field.
3. Record the current values and make a timestamped backup of every file to be edited.
4. Prepare a candidate in the workspace. For rules, use deterministic prepend/merge logic and make it idempotent: running it twice must not duplicate rules.
5. Validate the candidate structure and rule order. Put application exceptions before broad reject/direct rules that would otherwise win first.
6. Write back only the selected persistent file(s), then reapply the profile or restart Clash Verge.
7. Re-read the generated runtime YAML and run mihomo's syntax check. Do not report success based only on the enhancement file.
8. Verify live TUN state, actual adapter MTU, Windows proxy state, rule hits/logs, and a real request from the affected application. Mark endpoint responses such as HTTP 403/405/400 as reachability evidence only, not as proof of a successful login or end-to-end user action.

## Five-part verification report

When the user asks for a checklist or asks whether the setup is “好了”, report these items separately:

1. **Persistent source** — active profile, enhancement/merge file, backup path, and whether unrelated groups were preserved.
2. **Generated runtime** — `mode`, mixed port, TUN fields, DNS/fake-IP, MTU, rule order, and the selected proxy/direct groups.
3. **Syntax** — the exact generated file tested by `verge-mihomo.exe -t -f <file>` and its result.
4. **Windows/live state** — Clash process, login autostart, TUN adapter, adapter MTU, and `ProxyEnable`/System Proxy state.
5. **Traffic** — rule-log hits plus real destination requests. For application bugs, perform a TUN-on/TUN-off A/B test and distinguish a true TUN shutdown from merely closing the Clash UI.

## Failure handling

- If the generated mode falls back to `global`, fix the persistent generator setting, not only the generated file.
- If a rule is present but loses, inspect earlier rules and prepend the minimum exception; do not reorder an entire subscription blindly.
- If a controller API is refused, restart/reapply and inspect generated output and logs.
- If Bilibili or WeChat endpoints answer 200/400/405 but the user has not completed a real transfer/playback action, report the change as transport reachability verified and ask for the final user action.
- If closing the Clash window does not stop TUN, explain that the backend/mihomo process and virtual adapter are still active; perform a true TUN-off comparison.
- If a runtime rule uses `no-resolver`, correct it to mihomo's `no-resolve` spelling while preserving the rule's intended scope.

Read the relevant reference file before making a module change:

- `references/baseline.md` — stable host baseline and ownership of fields.
- `references/routing-modules.md` — OpenAI/Codex, Google, XMind, Bilibili, WeChat, and ad-block exception patterns.
- `references/verification.md` — commands, evidence interpretation, rollback, and boundaries.
