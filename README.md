<div align="center">

# 🛡️ dsh-permission-rules
- **1024 store channel**: `npm i -g dsh1024` once, then `dsh1024 plugin --profile web add dsh-permission-rules` (counts toward the [deepseek1024.com](https://deepseek1024.com) install ranking).
[![Gitee](https://img.shields.io/badge/Gitee-mirror-c71d23?logo=gitee)](https://gitee.com/perrylink/dsh-permission-rules)
[![dshfind](https://dshfind.com/api/badge/PerryLink/dsh-permission-rules?metric=downloads)](https://dshfind.com/plugins/PerryLink/dsh-permission-rules?ref=badge)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/PerryLink/dsh-permission-rules/badge)](https://api.securityscorecards.dev/projects/github.com/PerryLink/dsh-permission-rules)

**Claude Code-style declarative permission rules for DeepSeek Harness.**

*Rules decide what is known. A reviewer model decides what is not.*

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![DSH plugin](https://img.shields.io/badge/dsh--plugin-✅-green)](https://github.com/topics/dsh-plugin)
[![dsh-doctor](https://raw.githubusercontent.com/PerryLink/dsh-plugin-doctor/main/badges/PerryLink__dsh-permission-rules.svg)](https://github.com/PerryLink/dsh-plugin-doctor#verified-徽章)
[![DSH Market](https://raw.githubusercontent.com/2BingLing/dsh-market/master/assets/readme/badge-top-rated.svg)](https://dsh.market/)
[![Node](https://img.shields.io/badge/node-%5E22.19%20%7C%7C%20%3E%3D24-brightgreen.svg)](#)
[![CI](https://img.shields.io/github/actions/workflow/status/PerryLink/dsh-permission-rules/ci.yml?branch=main&label=CI)](https://github.com/PerryLink/dsh-permission-rules/actions)
[![Version](https://img.shields.io/github/v/tag/PerryLink/dsh-permission-rules?label=version)](https://github.com/PerryLink/dsh-permission-rules/releases)
[![npm version](https://img.shields.io/npm/v/dsh-permission-rules)](https://www.npmjs.com/package/dsh-permission-rules)
[![npm downloads](https://img.shields.io/npm/dm/dsh-permission-rules)](https://www.npmjs.com/package/dsh-permission-rules)

[English](README.md) · [简体中文](README-zh.md) · [Español](README-es.md) · [Português](README-pt.md) · [हिन्दी](README-hi.md)

</div>

---

**📖 Ecosystem knowledge base** — measured data, not marketing: [plugin development guide · plugin-selection data · maintenance criteria](https://perrylink.github.io/dsh-plugin-guide/).

<!-- star-cta -->
## ⭐ 如果它帮到了你

这个插件是 [DSH 插件家族](https://github.com/PerryLink)的一员（40+ 个，全部 Apache-2.0）。如果你在用，**给个 star** —— 它不会解锁任何功能，但会让下一个人在搜索里更容易找到它。

*English:* part of a 40+ plugin family for DeepSeek Harness. If it is useful, **a star helps the next person find it** — nothing is gated behind it.


## What is dsh-permission-rules?

Claude Code-style declarative permission rules for DeepSeek Harness.

Rules decide what is known. A reviewer model decides what is not.

![Terminal demo of dsh-permission-rules: dsh-permission-rules — deny/ask rules in one YAML file](https://raw.githubusercontent.com/PerryLink/dsh-permission-rules/main/docs/assets/dsh-permission-rules-demo.png)

![Animated terminal demo of dsh-permission-rules](https://raw.githubusercontent.com/PerryLink/dsh-permission-rules/main/docs/assets/dsh-permission-rules-demo.gif)

*The same run, animated.*

## Compatibility

| Surface | Status |
|---|---|
| Harness | DeepSeek Harness `dsh-v0.2.1-alpha.1` (adapted 2026-09-22, full gate chain): its plugin config is the live settings contract — every field is declared `.volatile()`, the Loader hands `apply` a live reference per field, and a saved edit is committed into the running references WITHOUT remounting the plugin, with out-of-range values refused by the schema before anything is written. A host whose Schemastery predates `volatile()` (before 3.18.3) still mounts and runs this plugin — the fields simply stay ordinary values and the live config form is unavailable — because the marker is applied through a capability probe rather than assumed. `Session.append` on this line still cannot stamp the `ignorable` marker (its third argument is the surface-intent bag, and the envelope field survives for stored-log reads only), so the whole `0.2-alpha` line is pre-checked as unmarked and session-log audit stays disabled by default — `append` reads only `sourceEventSeqs`/`surfaceOp` and freezes the envelope, so it has no code path that could stamp the marker; the earlier `0.1.7-alpha` (adapted 2026-09-22), `0.1.5-rc.2` (adapted 2026-09-09) and `0.1.3-alpha` lines keep the same surface-only append signature, and every `0.2` prerelease is classified with them (a STABLE `0.2.x` release is left to the append probe, so a restored marker surface is adopted without a new plugin release). Those lines' log migrations refuse unclassified plugin events even when marked, so `strip` v1 audit rows before a `0.1.3-alpha` host opens the log and v2 audit rows before a `0.1.5-alpha` host migrates it (native v3 logs only need `repair`). |
| Node | `^22.19.0 || >=24.0.0` |
| Platforms | All (host + web settings client) |
| Model | Any (deny/ask reasons surface through tool results) |

## What you get

`dsh-permission-rules` puts an ordered **`allow` / `deny` / `ask`** rule list in front of every tool call on the `tools/pre-execute` waterfall — deterministic, instant, auditable, and written by you in plain YAML:

- **`deny`** blocks the call; the rule's `reason` becomes the model-visible error.
- **`ask`** rides the official approval seam (mount `dsh-auto-review` for a second-model answerer, or a human answers; with neither, the harness fails closed).
- **`allow`** (and no-match) strictly delegates via `next()` — downstream listeners are never short-circuited.

Every hit **and** every passthrough is audit-logged as a `permissionRules/decision` session event (log-only — nothing extra is injected into the model context).

- **Rich matching** — tool-name globs (including `mcp__*`), agent-identity selectors (`main` / `subagent` / `preset:*`), argument key/value globs **or** regexes (with `!pattern` negation and an `absent` key dimension), workspace-relative path globs at **any nesting depth**, `when` host conditions (env vars, platform), and **shell command decomposition** (`argv`: command word, argument tokens, pipeline signature) for token-precise command matching.
- **Built-in high-risk baseline** — a shipped deny/ask ruleset (destructive commands, privilege escalation, download-and-execute, sensitive paths) enabled by default and appended after user rules so a nearer user rule can override it; toggle with `builtin.enabled`.
- **Hierarchical rule files** — optional `searchUp` merges every `.dsh/rules.yaml` from the session cwd to the filesystem root, nearest first.
- **Dry-run rollout** — `enforce: false` audits what the policy *would* do while passing every call through.
- **Hot reload** — Chokidar watch with debounce; a broken edit keeps the previous rules, never crashes. On a WSL host, or for a rule file under `/mnt/<drive>`, the watch switches to polling because native change events are unreliable there.
- **Fail loud** — invalid YAML, unknown actions/fields, bad globs/regexes, backtracking-prone patterns, or more than `maxRules` rules fail the load.

## Rule syntax

```yaml
# <project>/.dsh/rules.yaml
rules:
  - match: { tools: [bash, pwsh], params: { command: "git push*" }, paths: ["**/secrets/**"] }
    action: deny
    reason: "No pushes from protected paths"

  - match: { tools: [edit, write] }
    action: ask
    reason: "File writes need confirmation"
```

- **Match dimensions** — `tools` (globs, incl. `mcp__*`), `agents` (`main` / `subagent` / `preset:<name>`; unknown identity never matches — fail closed), `params` (key/value globs or regexes, `!pattern` negation, `absent` key dimension), `paths` (workspace-relative globs extracted at any nesting depth), `when` (`env` var globs/regexes + a closed `platform` list), and `network` (`domains` / `ips` / `ports` / `schemes` — globs, wildcards, CIDRs, port ranges).
- **Actions** — `allow` / `deny` / `ask`, evaluated in file order, first match wins.
- **Rule metadata** — `enabled: false` (visible but inert), `description`, `tags`; unknown fields fail the load.
- **Schema** — a JSON Schema ships at [docs/rules-format.schema.json](docs/rules-format.schema.json) (editor completion via `# yaml-language-server: $schema=...`); the full vocabulary and a 5-rule security baseline live in [docs/rules-format.en.md](docs/rules-format.en.md).

## Network policy

A Codex-style **process-level network policy**: shell subprocess traffic flows through a built-in local **HTTP/CONNECT proxy**, and every connection is decided by ordered network rules or by three modes mapped onto the official sandbox presets:

- **`deny-all`** — the read-only sandbox preset: block all outbound.
- **`whitelist`** — the workspace-write preset: allow listed targets, `unlisted: ask` (or `deny`) for the rest.
- **`allow-all`** — the danger-full-access preset: allow everything.
- **`auto`** (default) — follows the sandbox preset; on hosts without the sandbox-policy service it resolves to `autoFallback` (`allow-all`).

- **Matching** — `match.network` with `domains` / `ips` / `ports` / `schemes` (globs, wildcards, CIDRs, port ranges; numeric YAML ports are accepted). URL-candidate extraction on the `tools/pre-execute` hot path fires on web-tool arguments and URLs embedded in bash/pwsh command text; loopback targets can short-circuit rules per `loopback` policy. IPv4-mapped IPv6 literals are normalized to their IPv4 form before matching, and the proxy connects on the addresses the decision was made on — never a second DNS resolution — a connection whose adjudication resolved no address fails closed (502) instead of dialing the name.
- **Upstream chaining** — `network.upstreamProxy` (default `off`) sends the connections this proxy **allows** through an upstream proxy: CONNECT asks the upstream for a tunnel (`CONNECT host:port`) and plain-HTTP requests are forwarded to it in absolute form. A blocked target never reaches the upstream — it still gets this plugin's structured 403. Two cases never chain even when an upstream is configured: a **loopback** target (a proxy outside this host cannot route its loopback) and any decision produced by an **`ips`-scoped rule** (chaining hands the hostname to the upstream, so "the connection lands on an address the rules saw" — issue #21 — would stop holding exactly where the rules cared about the address; those decisions keep dialing the adjudicated address directly), plus a target whose scheme has no usable upstream. The upstream's own hostname is operator configuration, not agent input, and is not adjudicated by these rules. A URL carrying credentials is never emitted raw: warnings, `/rules network` and the settings snapshot mask the password (`http://user:***@host:port`). An unreachable upstream, a timeout (10 s), or a non-2xx answer yields **502** — there is deliberately no silent fallback to a direct dial, so a misconfiguration stays visible.
- **Audit** — denied connections append `permissionRules/network` to the owning session (same adaptive `ignorable` gate), with block counters and recent interceptions in `/rules network` and the settings page.
- **Diagnosis** — blocked connections carry a `[network: …]` message naming the blocked target, the mode or rule that decided, and the remediation.
- **Allow from the settings page** — every recent interception offers an **Allow** action. It writes ONE minimal `match: { network: { domains: [<host>] } }` / `action: allow` rule at the **top** (index 0) of the nearest rule file that actually judges that connection: the project file of the workspace the block was attributed to (the absolute `rulesFile` when one is configured), or, for a session-less host-level block, the file the host chain resolves (absolute `rulesFile` → existing `<processCwd>/<rulesFile>` → configured `fallbackPath` → `<processCwd>/<rulesFile>`, created) — and when several workspaces are loaded, the page first offers a picker, because any loaded workspace chain outranks the host chain. Index 0 because rules are first-match-wins: appended after an existing `deny` the rule would be dead text. `domains` only, because that dimension is subdomain-inclusive and port/scheme-agnostic, so the host is not re-blocked on its next port; existing comments and untouched rules survive, and the text goes through the same validation gate as a hand edit. The change takes effect immediately — the cached workspace chains AND the session-less host chain are re-read, so no restart and no `/rules reload` — and the decision is recomputed, so the notice reports the REAL outcome and never claims success while the connection is still blocked. It refuses, leaving the file untouched, an unknown workspace, a target outside the known rule sources, the read-only built-in baseline, and a file it cannot read or parse; an already-allowed connection writes nothing. `network.allowHostAction: false` hides the button and makes the RPC refuse.

## Quick start

```sh
dsh plugin --profile web add github:PerryLink/dsh-permission-rules
```

```sh
# 1. install the bundle into your profile
dsh plugin --profile web add github:PerryLink/dsh-permission-rules

# or from npm (published releases)
dsh plugin --profile web add dsh-permission-rules

# 2. restart and verify the row
dsh --profile web --dump-config | grep -A4 'id: permission-rules'
```

## Install & uninstall

- **git channel** (latest `main`): `dsh plugin --profile web add github:PerryLink/dsh-permission-rules` — the `prepare` script builds with production dependencies only.
- **npm channel** (published releases): `dsh plugin --profile web add dsh-permission-rules`.
- **tarball channel**: `pnpm pack` in this repo, then `dsh plugin --profile web add ./dsh-permission-rules-<version>.tgz`.
- **uninstall**: `dsh plugin --profile web remove dsh-permission-rules`.

## Configuration

All tunables are Schemastery `Config` fields (changeable from cordis.yml). An id-targeted override replaces the whole row — restate every key you need. On a `0.1.7-alpha` host every field is declared `.volatile()`, so it is ALSO editable in place from the Plugins settings page: the edit is committed into the running references without remounting the plugin, the runtime reads the new value on its next use, and a change to `network.proxyPort`, `proxyBind`, `injectEnv` or `noProxy` rebinds the local proxy immediately. The numeric bounds live in the schema, and the config editor resolves the merged config against that schema before persisting, so an out-of-range value is refused at write time. On a host whose Schemastery predates `volatile()` these fields stay cordis.yml-only.

| Key | Default | Meaning |
|---|---|---|
| `rulesFile` | `.dsh/rules.yaml` | Rule file location; relative = resolved against the calling session's cwd, absolute = global and validated at mount |
| `fallbackPath` | *(none)* | Rule file used when per-cwd discovery finds nothing; validated at mount |
| `badFilePolicy` | `fail` | Bad rule file: `fail` errors the pending tool call loudly; `ignore-with-warning` warns and continues empty |
| `maxRules` | `256` | Hard cap on rule count across the effective source chain |
| `maxCachedWorkspaces` | `512` | Hard cap on cached per-workspace rule loads (LRU eviction) |
| `patternMode` | `glob` | `params`/`paths`/`when.env` pattern flavor: `glob` or `regex` (tool names are always globs) |
| `watch` | `true` | Chokidar watch + reload on change |
| `watchStabilityThresholdMs` | `200` | Reload debounce window (ms) |
| `language` | `en` | `/rules` output language: `en`, `zh`, `es`, `pt`, `hi` |
| `caseInsensitivePaths` | *(win32)* | `paths` patterns and workspace-root comparison ignore ASCII case; `true` on Windows |
| `audit` | `all` | Audit granularity: `all` logs every hit AND passthrough; `hits` skips passthrough events |
| `searchUp` | `false` | Walk parent directories from the session cwd and merge every found rule file, nearest first |
| `maxGlobStars` | `2` | Hard cap on unbounded `*`/`**` quantifiers per glob pattern |
| `enforce` | `true` | `false` = dry-run mode: deny/ask hits are audit-logged with a `dryRun` marker and every call passes through |
| `allowUnmarkedAudit` | `false` | Pre-marker hosts drop the `ignorable` marker; the plugin disables session-log audit with a warning. Set `true` to opt back in |
| `network.enabled` | `true` | Master switch for the proxy, env injection, and web-tool mode defaults |
| `network.mode` | `auto` | Policy mode: `auto` follows the sandbox preset, or `deny-all` / `whitelist` / `allow-all` |
| `network.autoFallback` | `allow-all` | Mode used when `auto` has no sandbox-policy service |
| `network.unlisted` | `ask` | Whitelist-mode handling of targets no rule matched: `ask` or `deny` |
| `network.proxyBind` | `127.0.0.1` | Local proxy bind address (loopback only) |
| `network.proxyPort` | `0` | Local proxy port; `0` picks a free ephemeral port |
| `network.proxyMaxRecent` | `100` | Cap on recent-block records kept for the settings page |
| `network.loopback` | `allow` | Loopback targets: `allow` (Codex parity) or `policy` |
| `network.injectEnv` | `true` | Whether proxy environment variables are injected for subprocesses |
| `network.noProxy` | `clear` | Subprocess NO_PROXY handling: `clear` enforces the policy or `preserve` |
| `network.upstreamProxy` | `off` | Upstream proxy for connections this plugin ALLOWS: `off` dials directly, `inherit` reuses the launch environment's proxy names, or an explicit `http(s)://` proxy URL |
| `network.allowHostAction` | `true` | Settings-page per-block "allow this host" action: `false` hides the Allow button and makes the `permissionRules/allowHost` RPC refuse (the rule editor is not affected) |
| `builtin.enabled` | `true` | Built-in high-risk baseline: `false` disables the shipped deny/ask ruleset entirely |
| `builtin.path` | *(shipped)* | Replacement baseline file (absolute, or relative to `process.cwd()`); validated at mount |

## Tools & surfaces

| Surface | Kind | Notes |
|---|---|---|
| `tools/pre-execute` | listener | First-match allow/deny/ask rules + network URL-candidate extraction |
| `/rules` | command | `list` · `reload` · `decisions [n]` · `test <tool> <json>` |
| `permissionRules/decision` | event | Log-only audit for every hit and passthrough |
| `permissionRules/network` | event | Proxy-layer audit for blocked connections |
| HTTP/CONNECT proxy | service | Built-in local proxy governing shell subprocess traffic |
| settings page | client | Network-mode editor, rule editor, block counters, recent interceptions with a per-block allow action |

```
/rules                        list the active rules, their source files, and any last-reload error
/rules list                   explicit alias for the bare listing
/rules reload                 re-read the rule-file chain for this workspace
/rules decisions [n]          show the last n permission decisions of this session (default 10)
/rules test <tool> <json>     dry-evaluate the rules against a hypothetical call
```

`/rules test` also accepts leading flags: `--cwd <dir>`, `--env KEY=VALUE` (repeatable), `--agent <selector>` (repeatable), and `--platform <name>`. In multi-file chains (e.g. `searchUp`), every listed rule line is attributed to its own source file.

## Permissions & data

- **Permissions**: declares `files:read`, `files:watch`, `files:write`, `session:append`, and `network:outbound` in its workshop manifest. `ask` decisions ride the official approval seam — nothing is re-implemented or bypassed.
- **Data**: rule files are read from disk; no rule data is written. No model calls, no reviewer subagents.
- **Session log**: `permissionRules/decision` is never injected into the model context and is appended with the envelope's `ignorable: true` marker so any harness build loads the log.

## Security boundaries

- **Policy, not a kernel.** `paths` candidates come only from a documented set of argument keys (at