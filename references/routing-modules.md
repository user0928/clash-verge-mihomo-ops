# Routing modules

Apply only the module requested by the user. Each module is a pattern from a previously verified repair, not a reason to rewrite the whole profile.

## Google through a proxy

When the user explicitly wants Google proxied, check the existing rule order first. A historical fix prepended:

```text
DOMAIN-KEYWORD,google,<selected-proxy-group>
```

This was necessary when earlier direct rules won. In the later XSUS cleanup, the broad top-level Google keyword was removed because it overrode more specific subscription logic; the subscription's own Google rule was kept. Therefore, do not blindly add the broad keyword to every profile. Verify the intended result by inspecting the first matching rule for a representative Google hostname.

## XMind selective routing

Extend the existing selective setup without changing TUN, DNS, mode, or unrelated strategy groups. Use the real process name and the live group discovered in the profile. The historical group was `XFX` and the pattern included a `PROCESS-NAME` rule plus narrowly scoped `DOMAIN`, `DOMAIN-SUFFIX`, or `DOMAIN-KEYWORD` rules. Reapply/restart before deciding the rule is live.

## Bilibili live playback

The verified minimal repair was:

1. Merge `tun.mtu: 1500` for the active subscription.
2. Prepend Bilibili process and domain exceptions to `DIRECT`.
3. Route `cm.bilibili.com` explicitly to `REJECT` when it is being misclassified by the global intercept group.
4. Do not globally rewrite the “global reject” group.

Relevant domains observed in the repair included `bilibili.com`, `biliapi.com`, `bilivideo.com`, and `hdslb.com`. Confirm the user's exact application and rule log before adding more.

Validate generated YAML, mihomo syntax, the actual Meta adapter MTU, API status, and a real media read. If TUN-off makes the page work, keep the result as an A/B diagnosis rather than claiming the content source is broken.

## WeChat image/file transfer

Keep the existing TUN and selective-routing baseline. Prepend direct exceptions for the running process names and media domains:

```text
PROCESS-NAME,Weixin.exe,DIRECT
PROCESS-NAME,WeChat.exe,DIRECT
PROCESS-NAME,WeChatAppEx.exe,DIRECT
DOMAIN-SUFFIX,weixin.qq.com,DIRECT
DOMAIN-SUFFIX,wx.qq.com,DIRECT
DOMAIN-SUFFIX,servicewechat.com,DIRECT
DOMAIN-SUFFIX,qpic.cn,DIRECT
DOMAIN-SUFFIX,qlogo.cn,DIRECT
DOMAIN-SUFFIX,mmecimg.com,DIRECT
DOMAIN-SUFFIX,wechat.com,DIRECT
```

If fake-IP is interfering, add the same narrow domains to the active fake-IP filter/merge source. Earlier traffic showed `qlogo.cn` being caught by a broad reject rule before a later Tencent direct rule, so rule order matters. Do not call transport checks such as HTTP 400/405 a successful user transfer; have the user send an image and a small file.

## Ads and exceptions

The current repair set the advertising group to `REJECT` and centralized application exceptions in the dedicated persistent enhancement script. Keep exceptions in one source to avoid duplicate maintenance. Do not turn off all blocking to repair one app.

## Preserve these unrelated areas

Unless explicitly requested, preserve Tailscale rules, existing DNS servers and fake-IP topology, node definitions, strategy-group choices, and the final fallback rule. Never publish a full generated subscription or a controller secret.
