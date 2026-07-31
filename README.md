# livekit-google-plugin (patched fork)

A fork of LiveKit's [`livekit-plugins-google`](https://github.com/livekit/agents/tree/main/livekit-plugins/livekit-plugins-google)
that carries a small set of local patches on top of specific upstream releases —
primarily fixes and tweaks to the **Gemini realtime (Live API)** path that we need
before they are available (or fixed) in an upstream release.

The importable package is still `livekit.plugins.google` and its API matches
upstream; this fork only changes behavior where a patch is noted below.

## Why this fork exists

The upstream plugin is tightly coupled to internal `livekit-agents` APIs, and some
behaviors we depend on are either not yet upstream or are upstream defaults that
don't fit our Gemini realtime + MCP setup. Rather than wait for an upstream release
cycle, we:

1. **Sync** the fork to a specific upstream `livekit-plugins-google` version (pinned
   to a known-good `livekit-agents` version), and
2. **Re-apply our local patches** on top.

Each fork release is a Git **tag** (`v1.0.x`). These tags are independent of the
upstream plugin version number.

### What this fork patches (vs. upstream)

- **Realtime tool parameters preserve pre-filled values.** Upstream routes realtime
  tool schemas through a lossy "simplify" step (dropping `default` and sending the
  simplified `parameters` schema) because early Gemini Live models rejected the full
  `parameters_json_schema`. That step stripped pre-filled MCP parameters
  (e.g. `automationId`, `version`), so the model sent invalid arguments and the MCP
  server rejected the call. This fork sends the full raw JSON schema for MCP/raw
  tools and preserves `default` for plain function tools. See
  [CHANGELOG.md](CHANGELOG.md) → **v1.0.5** for the full rationale and the caveat for
  older Live models.
- **Realtime stability tweaks:** stash-and-replay of tool results across
  `update_tools` reconnects, and conditionalized generation triggers for non-Vertex
  (Gemini API) models.

For the exact per-tag differences, see **[CHANGELOG.md](CHANGELOG.md)**.

## Installation

Install a specific tag directly from Git — always pin a tag, never `main`:

```bash
pip install "git+https://github.com/heershah434/livekit-google-plugin.git@v1.0.5"
```

Or in `requirements.txt` / `pyproject.toml`:

```
livekit-plugins-google @ git+https://github.com/heershah434/livekit-google-plugin.git@v1.0.5
```

## Choosing a version

Pick the fork tag that matches the **`livekit-agents`** version your project uses —
the plugin depends on internal `livekit-agents` APIs, so mismatched versions break.

| Fork tag | `livekit-plugins-google` | `livekit` | `livekit-agents` |
| -------- | ------------------------ | --------- | ---------------- |
| `v1.0.5` | 1.6.7                    | 1.1.13    | 1.6.7            |
| `v1.0.4` | 1.6.4                    | 1.1.12    | 1.6.4            |
| `v1.0.3` | 1.5.17                   | 1.1.8     | 1.5.17           |
| `v1.0.2` | 1.5.4                    | 1.1.5     | 1.5.4            |

Full details and older tags: **[CHANGELOG.md](CHANGELOG.md)**. All published tags:
<https://github.com/heershah434/livekit-google-plugin/tags>.

## Relationship to upstream

This is a downstream patch set. When a patch lands upstream, we drop it from the fork
on the next sync. Upstream source and docs:

- Source: <https://github.com/livekit/agents>
- Docs: <https://docs.livekit.io>

Licensed under Apache-2.0 (see [LICENSE](LICENSE) and [NOTICE](NOTICE)).
