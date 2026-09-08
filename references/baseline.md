# Baseline and field ownership

This is a reusable baseline, not a complete subscription file. Values that identify the current machine are examples and must be rediscovered.

## Stable end state

| Area | Expected state | Why it matters |
|---|---|---|
| Core | Clash Verge Rev + mihomo | Determines field names and validation command |
| Mode | `rule` | Selective routing; avoids global proxying |
| Mixed port | `7897` on the verified host | The old launcher used `7890`; never assume the port |
| TUN | enabled | Captures traffic regardless of how Codex or an app starts |
| Stack | `gvisor` | Verified working baseline on this host |
| Auto route | `true` | Installs the TUN route |
| Strict route | `false` | Preserve the validated baseline unless a new requirement justifies changing it |
| Interface detection | `true` | Lets mihomo choose the active physical adapter |
| DNS hijack | `any:53` | Keeps DNS on the TUN path |
| TUN MTU | `1500` | Avoids the observed Bilibili live-stream issue from a Meta adapter at 9000 |
| DNS mode | existing `fake-ip` design | Keep the user's working DNS topology |
| Windows System Proxy | disabled | Other applications remain direct |
| Autostart | existing Clash Verge login task | Clash must be ready before ordinary launches |

## File ownership

- `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\verge.yaml`: UI and Verge state flags, including the TUN/System Proxy state exposed by the frontend.
- `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\config.yaml`: persistent generator defaults such as the default `mode`.
- `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\profiles\*.yaml` or `*.js`: profile enhancement/merge/rule sources. The active profile must be discovered first.
- `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\clash-verge.yaml`: generated runtime configuration. Inspect and validate it, but do not treat it as the only persistent edit target.

Historical profile IDs and filenames changed across tasks. Never copy a remembered ID into a new machine or a new subscription without checking the active selection.

## Reconnect 5/5 interpretation

The earlier symptom was: conversations repeatedly entered reconnect, then worked after the fifth retry; manually enabling Windows System Proxy helped, while a launcher-only environment-variable wrapper did not cover every path. The durable correction was to move routing to TUN/rule mode and map the OpenAI/Codex destination set to the existing proxy group.

This is not proof that every 5/5 symptom is caused by Clash. Verify with rule logs and a TUN-off A/B test. A successful HTTP response alone does not prove the application session is healthy.

## OpenAI/Codex baseline rule set

Use the existing group name discovered from the active configuration. The historical group was `ChatGPT`. The destination set included:

```text
chatgpt.com
chat.com
openai.com
oaistatic.com
oaiusercontent.com
openai.com.cdn.cloudflare.net
statsigapi.net
statsig.com
auth0.com
arkoselabs.com
livekit.cloud
sentry.io
browser-intake-datadoghq.com
openaicom.imgix.net
```

Prefer `DOMAIN-SUFFIX` for suffix-based domains and `DOMAIN` for exact hosts. Put these rules before broad reject or direct rules that would otherwise win. Keep the fallback rule unchanged.
