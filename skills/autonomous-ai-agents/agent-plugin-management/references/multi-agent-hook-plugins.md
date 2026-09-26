# One hook binary as a plugin for several agents

Use when a single CLI hook (one binary that detects the calling agent) should install as a native plugin in each agent. Verified against Claude Code 2.1, Codex CLI 0.154, Gemini CLI 0.39 and Hermes Agent 0.21; re-check each contract against the installed agent before relying on it.

## Per-agent contract

| Agent | Manifest and hook | Install from GitHub | `version` |
|---|---|---|---|
| Claude Code | root `.claude-plugin/marketplace.json` → plugin dir with `.claude-plugin/plugin.json`; `hooks/hooks.json` in the plugin dir is auto-loaded | `/plugin marketplace add <owner>/<repo>`, then `/plugin install <name>@<marketplace>` (separate prompts) | optional; omitted → keyed by commit SHA |
| Codex | root `.agents/plugins/marketplace.json` with `{"source":"local","path":"./plugins/codex"}` → `.codex-plugin/plugin.json` with `"hooks": "./hooks/hooks.json"` | `codex plugin marketplace add <owner>/<repo>` + `codex plugin add <name>@<marketplace>`; user trusts the hook in `/hooks` | optional; omitted → cached as `local` |
| Gemini CLI | root `gemini-extension.json` + root `hooks/hooks.json` (fixed path, not configurable) | `gemini extensions install https://github.com/<owner>/<repo>` | **required**; install fails without it |
| Hermes | `plugin.yaml` + `__init__.py` with `register(ctx)`; hooks are Python callbacks, not shell commands | `hermes plugins install <owner>/<repo>#<subdir> --enable`; manifest `name` wins over the directory name | optional |

Agents whose prompt hooks cannot inject context (and agents without a plugin system) keep their manual install; say so rather than shipping an empty plugin.

## Layout

- Gemini owns the repository root (`gemini-extension.json`, `hooks/hooks.json`). Claude Code also auto-loads `hooks/hooks.json` from a plugin root, so a root-level Claude plugin would load Gemini's events. Put the other agents' plugins in `plugins/<agent>/` and point their marketplaces there.
- Hook commands run the bare binary name; the binary stays a separate install (package manager), and the plugin only registers it.
- A Hermes Python plugin can reuse the binary unchanged: build the same stdin JSON Hermes sends a shell hook for that event, run the binary with a timeout, return `{"context": ...}`; on a missing binary, timeout or bad output, log and return `None` so a turn never breaks.
- Where `version` is required, add a release guard that compares it with the canonical version (e.g. the tag-vs-`Cargo.toml` step) and name both files in the release instructions. Where optional, omit it so nothing drifts.
- Add `__pycache__/` to the repo's ignore file when shipping a Python plugin; the agent compiles it on load and would dirty every checkout that pins the repo.

## Delivery

- One PR per agent. Each edits only its own README install block; blocks separated by an unchanged line merge cleanly, and `git apply --3way` of all patches onto a fresh base proves it.
- In the README, lead with the plugin install. If the manual hook stays documented, warn that using both runs the hook twice; users may prefer dropping the manual config entirely.

## Verify each agent in isolation

Install from the branch into a throwaway home so the user's real configuration is untouched:

- Claude Code: `CLAUDE_CONFIG_DIR=<tmp>`; `claude -p --debug-file <log>` fires `UserPromptSubmit` even when not logged in; grep the log for `Hook UserPromptSubmit (<name>) provided additionalContext`.
- Codex: `CODEX_HOME=<tmp>`; `codex exec --dangerously-bypass-hook-trust --skip-git-repo-check "<prompt>"` runs the hook before the model call.
- Gemini: `HOME=<tmp>`; pre-seed `<tmp>/.gemini/trustedFolders.json` (`{"<abs path>":"TRUST_FOLDER"}`) because the install trust prompt needs a TTY; `settings.json` with `gemini-api-key` auth plus a dummy `GEMINI_API_KEY` fires `BeforeAgent` before the request fails.
- Hermes: a fresh `HERMES_HOME` triggers a full first-run bootstrap (dependency install, UI builds) in the shared install; instead point a throwaway home's `plugins/` at the tree and call the loader in-process (`discover_plugins(force=True)`, then `invoke_hook("pre_llm_call", ...)`).

When the model call cannot run (auth expired, tier retired), prove the hook with a logging shim first on `PATH` that `tee`s stdin and stdout around the real binary. If the binary reads data from `$HOME`, have the shim restore the real `HOME` for the binary only, or a temp home makes it a silent no-op.

Never copy rotating OAuth credentials (e.g. Codex `auth.json`) into a temp home: refresh tokens are single-use, and a copy can burn the user's login. Use the bypass paths above instead.

## Switching a live configuration to the plugin

- Remove the hand-written hook entry in the same change that enables the plugin, or the hook runs twice.
- Env vars set inline in the old hook command must move: Claude Code settings `env`, Hermes `$HERMES_HOME/.env`. Confirm the agent's service PATH (e.g. a launchd plist) contains the binary's directory.
- `claude plugin install` rewrites `settings.json` in its own key order; if it is semantically equal to the committed file, restore the committed formatting.
- A plugin living in a subdirectory of a multi-agent repo does not fit a plugins topology that pins one repo per plugin root: pin the repo under a non-plugin directory (e.g. `sources/<repo>`) and track a symlink `<name> -> sources/<repo>/<plugin-subdir>`; check the scanner treats `sources/` as an empty category.
- Restart the long-running service (gateway) and confirm from its log that the plugin loaded; run one real turn and confirm exactly one injection.
