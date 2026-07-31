# Changelog

All notable changes to this fork are documented here. Versions are the fork's own
release **tags** (`v1.0.x`), which are independent of the upstream
`livekit-plugins-google` package version.

> **How to read this:** each fork tag = a snapshot of upstream `livekit-plugins-google`
> at a specific version, **plus** the local patches listed under each entry. Pick the
> tag whose `livekit-agents` compatibility matches the version your project is pinned to
> (see [Which tag should I use?](#which-tag-should-i-use)).
>
> Descriptions below are derived from the actual `git diff` between consecutive tags.

## Compatibility matrix

| Fork tag | Upstream plugin base | `livekit` (SDK) | `livekit-agents` | pin in `pyproject.toml` |
| -------- | -------------------- | --------------- | ---------------- | ----------------------- |
| `v1.0.5` | 1.6.7                | 1.1.13          | 1.6.7            | `livekit-agents>=1.6.7` |
| `v1.0.4` | 1.6.4                | 1.1.12          | 1.6.4            | `livekit-agents>=1.6.4` |
| `v1.0.3` | 1.5.17               | 1.1.8           | 1.5.17           | `livekit-agents>=1.5.17`|
| `v1.0.2` | 1.5.4                | 1.1.5           | 1.5.4            | `livekit-agents>=1.5.4` |
| `v1.0.1` | —                    | not documented  | ≥ 1.5.0          | `livekit-agents>=1.5.0` |
| `v1.0.0` | —                    | not documented  | ≥ 1.5.0          | `livekit-agents>=1.5.0` |

The `livekit` (SDK) column was only tracked from `v1.0.2` onward. For `v1.0.0`/`v1.0.1`,
only the `livekit-agents>=1.5.0` floor from `pyproject.toml` is known.

The fork tracks `livekit-agents` closely: the plugin depends on internal
`livekit-agents` APIs, so a fork tag must be paired with the matching
`livekit-agents` version above.

---

## v1.0.5 — sync to upstream plugin 1.6.7 (livekit-agents 1.6.7)

**Compatibility:** `livekit==1.1.13`, `livekit-agents==1.6.7`.
**Base:** upstream `livekit-plugins-google` 1.6.7 (synced from 1.6.4).

**Upstream sync brought in** (`llm.py`, `stt.py`, `realtime_api.py`, `utils.py`,
`version.py`, `pyproject.toml`): the 1.6.4 → 1.6.7 plugin changes, incl. an `llm.py`
refactor and `stt.py` updates.

**Fork patches added in this tag:**
- **Realtime tool params preserve pre-filled values.** In `realtime/realtime_api.py`,
  `create_tools_config(...)` is now called with `use_parameters_json_schema=True`
  (upstream defaults it to `False` for realtime). This sends the **full raw JSON
  schema** (`parameters_json_schema`) for MCP/raw tools instead of the lossy
  simplified schema, so pre-filled parameters (e.g. `automationId`, `version`) reach
  Gemini and the model produces valid calls.
  - Context: upstream defaults to `False` because early Gemini **Live** models
    rejected `parameters_json_schema` and returned empty args
    ([googleapis/python-genai#1147](https://github.com/googleapis/python-genai/issues/1147)).
    Current Gemini 3.x realtime models accept it. **If you point this at an older Live
    model that errors or omits args on connect, revert this flag to `False`.**
- **Keep `default` in `_GeminiJsonSchema.simplify()`** (`utils.py`). The simplify path
  is still used for plain `@function_tool` tools (and structured-output schemas). It
  previously dropped the JSON-schema `default` key, silently losing defaults on
  non-raw tools; `default` is now preserved (`$schema`, `additionalProperties`, and
  `title` are still stripped, as Gemini rejects them).
- **Docs:** added this `CHANGELOG.md` and expanded `README.md`.

**Carried-forward fork patches** (from earlier tags, still present): realtime
stash-and-replay of tool results across `update_tools`; conditionalized generation
triggers for non-Vertex models.

---

## v1.0.4 — sync to upstream plugin 1.6.4 (livekit-agents 1.6.4)

**Compatibility:** `livekit==1.1.12`, `livekit-agents==1.6.4`.
**Base:** upstream `livekit-plugins-google` 1.6.4 (synced from 1.5.17).

Pure upstream sync (no new fork-specific patch). Diff vs `v1.0.3`:
- `realtime_api.py`: substantial update (~+105 lines) from the 1.6.4 plugin.
- `beta/gemini_tts.py`, `llm.py`, `stt.py`, `utils.py`: upstream refinements.
- `pyproject.toml`: `livekit-agents` pin raised to `>=1.6.4`.

Carried-forward fork patches remain in place.

## v1.0.3 — sync to upstream plugin 1.5.17 (livekit-agents 1.5.17)

**Compatibility:** `livekit==1.1.8`, `livekit-agents==1.5.17`.
**Base:** upstream `livekit-plugins-google` 1.5.17 (synced from 1.5.4).

Pure upstream sync. Diff vs `v1.0.2`:
- **New file `aiplatform_llm.py`** (~+322 lines) — Vertex AI Platform LLM support from
  upstream.
- `llm.py` (~+58), `stt.py`, `realtime_api.py`, `utils.py`, `models.py`: upstream
  updates.
- `pyproject.toml`: `livekit-agents` pin raised to `>=1.5.17`.

Carried-forward fork patches remain in place.

## v1.0.2 — pin to upstream plugin 1.5.4 (livekit-agents 1.5.4)

**Compatibility:** `livekit==1.1.5`, `livekit-agents==1.5.4`.

**Fork patch added in this tag:**
- **Conditionalize generation triggers for non-Vertex models** (`realtime_api.py`).
  Only fire the relevant generation triggers on the Gemini API (non-Vertex) path.

Also raised the `livekit-agents` pin to `>=1.5.4` in `pyproject.toml`.

## v1.0.1 — realtime tool-result durability (livekit-agents ≥ 1.5.0)

**Compatibility:** `livekit-agents>=1.5.0` (livekit SDK version not documented).

**Fork patch added in this tag:**
- **Stash and replay tool result after `update_tools`** (`realtime_api.py`, +34/−10).
  When the realtime session restarts to apply a tool change, the in-flight tool
  result is stashed and replayed to the new session instead of being lost.

## v1.0.0 — initial fork release (livekit-agents ≥ 1.5.0)

**Compatibility:** `livekit-agents>=1.5.0` (livekit SDK version not documented).

First tagged release. Vendors the full `livekit-plugins-google` package under
`livekit/plugins/google/` (`llm.py`, `stt.py`, `tts.py`, `models.py`, `utils.py`,
`tools.py`, `beta/gemini_tts.py`, and the `realtime/` module) with **Gemini realtime
support, including Gemini 3.1 realtime**. Adds `LICENSE` and `NOTICE`.

---

## Which tag should I use?

Match the tag to the **`livekit-agents`** version your project already uses:

- On `livekit-agents==1.6.7` → use **`v1.0.5`**
- On `livekit-agents==1.6.4` → use **`v1.0.4`**
- On `livekit-agents==1.5.17` → use **`v1.0.3`**
- On `livekit-agents==1.5.4` → use **`v1.0.2`**
- On `livekit-agents` 1.5.0–1.5.3 → use **`v1.0.1`** (or `v1.0.0`)

Install a specific tag directly from Git:

```bash
pip install "git+https://github.com/heershah434/livekit-google-plugin.git@v1.0.5"
```

Or pin it in `requirements.txt` / `pyproject.toml`:

```
livekit-plugins-google @ git+https://github.com/heershah434/livekit-google-plugin.git@v1.0.5
```

> Do not mix a fork tag with a mismatched `livekit-agents` version — the plugin
> relies on internal `livekit-agents` APIs and will break across incompatible
> versions.
