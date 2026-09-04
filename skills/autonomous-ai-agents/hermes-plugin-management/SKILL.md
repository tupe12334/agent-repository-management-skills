---
name: hermes-plugin-management
description: "Use when managing durable native Hermes plugins as Git repositories and submodules."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hermes, plugins, configuration, repositories, submodules]
    related_skills: [agent-repository-management]
---

# Hermes Plugin Management

Use this skill when creating, migrating, updating, or delivering a durable native Hermes plugin that lives under `$HERMES_HOME/plugins/`.

This skill covers native Hermes plugins, not Electron desktop plugins. For desktop panes, commands, and UI extensions, use `hermes-desktop-plugins`.

## Principle

Each durable Hermes plugin owns its source, history, tests, releases, and documentation in a dedicated Git repository. A Hermes configuration repository consumes a reviewed, pinned plugin revision as a Git submodule; it does not vendor plugin source.

## Ownership boundary

- A plugin repository owns `plugin.yaml`, implementation files, tests, plugin-specific documentation, and its own generated-file ignore rules.
- The Hermes configuration repository owns plugin installation topology: the `plugins/<name>` gitlink, `.gitmodules` declaration, enablement/configuration, and operator documentation.
- Choose each plugin repository's visibility deliberately. A personal agent configuration repository being private does not make every plugin private, and a public plugin does not authorize publishing another plugin.
- Do not confuse an ordinary directory with an independent repository. From `plugins/<name>`, `git rev-parse` can resolve to the parent configuration repository; inspect local `.git` metadata and the parent index first.
- Do not track the Hermes application source in either the configuration repository or a plugin repository.

## Delivery workflow

1. Inventory the requested plugin paths before mutating them: parent index entries, `.gitignore` coverage, child `.git` metadata, remote URL, branch, cleanliness, and `HEAD...origin/main` ancestry.
2. For an in-tree plugin without a dedicated repository, create or verify its intended remote, initialize the plugin repository, add a narrow `.gitignore` for generated artifacts, commit the current source, push a feature branch, open and merge a PR, then verify the merged remote `main` SHA.
3. For an existing child checkout, preserve a clean unmerged feature branch by pushing it before checking out the current remote `main`. Never discard a child commit merely to make a parent gitlink appear current.
4. In the configuration repository, replace parent-tracked plugin files with a gitlink using `git rm -r --cached plugins/<name>` and `git submodule add --force -b main <url> plugins/<name>`. Remove only obsolete ignore rules that would hide the tracked submodule path.
5. Keep the child-repository delivery and the parent gitlink delivery separate. Merge and verify the child PR first; then advance the parent to the verified child SHA in its own focused PR.
6. Verify every initialized child is clean and at its intended remote branch tip. Run each plugin's focused tests or compile checks, `git diff --check`, `git submodule status --recursive`, and `HERMES_HOME=<home> hermes config check`.
7. Confirm Hermes can discover the plugin at the resulting path and that any required opt-in enablement is configured. Restart Hermes or start a new session when plugin code is not dynamically reloaded.

## Routine synchronization

- For a normal configuration sync, initialize recorded plugin revisions with `git submodule update --init --recursive`; do not float plugin versions.
- Advance plugins with `git submodule update --remote --recursive` only when intentionally preparing a parent gitlink update for review and delivery.
- Treat a dirty child, a branch mismatch, a wrong remote, or a gitlink that differs from the parent-recorded SHA as a delivery condition to inspect—not as permission to reset, clean, force-push, or bypass hooks.

## Completion evidence

Report the plugin repository URL, merged plugin SHA, parent gitlink commit URL and SHA, the exact checks run, and confirmation that the active configuration checkout is pinned to the verified child revision.
