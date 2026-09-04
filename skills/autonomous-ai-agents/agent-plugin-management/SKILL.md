---
name: agent-plugin-management
description: "Use when managing durable agent plugins or extensions as Git repositories and pinned dependencies."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [agents, plugins, extensions, configuration, repositories, submodules]
    related_skills: [agent-repository-management]
---

# Agent Plugin Management

Use this skill when creating, migrating, updating, or delivering a durable plugin or extension for an autonomous agent system.

This is a cross-agent ownership and delivery skill. Before changing plugin files, load the target agent's plugin-development documentation or skill for its manifest, discovery path, enablement, reload, and runtime-verification contract.

## Principle

Each durable agent plugin owns its source, history, tests, releases, and documentation in a dedicated Git repository. An operator's agent-configuration repository consumes a reviewed, pinned plugin revision as a dependency—typically a Git submodule—rather than vendoring plugin source.

## Ownership boundary

- A plugin repository owns its manifest, implementation, tests, plugin-specific documentation, and generated-file ignore rules.
- An agent-configuration repository owns installation topology: the plugin dependency reference, configuration/enablement, and operator documentation.
- Choose each plugin repository's visibility deliberately. A private agent configuration repository does not make every plugin private, and a public plugin does not authorize publishing another plugin.
- Do not confuse an ordinary directory with an independent repository. From a plugin path, `git rev-parse` can resolve to the parent configuration repository; inspect local `.git` metadata and the parent index first.
- Do not track the agent application's source in either the configuration repository or a plugin repository.

## Delivery workflow

1. Discover the target agent's authoritative plugin contract: manifest shape, install/discovery path, enablement, reload behavior, and focused runtime check. Do not import a different agent's assumptions.
2. Inventory requested plugin paths before mutating them: parent index entries, ignore coverage, child `.git` metadata, remote URL, intended branch, cleanliness, and `HEAD...origin/<branch>` ancestry.
3. For an in-tree plugin without a dedicated repository, create or verify its intended remote, initialize the plugin repository, add a narrow ignore file for generated artifacts, commit the current source, push a feature branch, open and merge a PR, then verify the merged remote branch SHA.
4. For an existing child checkout, preserve a clean unmerged feature branch by pushing it before checking out the intended remote branch. Never discard a child commit merely to make a parent dependency reference appear current.
5. When the configuration repository uses Git submodules, replace parent-tracked plugin files with a gitlink, record the remote and intended branch in `.gitmodules`, and remove only obsolete ignore rules that would hide the tracked path. If the agent uses a different dependency mechanism, follow its documented equivalent without copying plugin source into the configuration repository.
6. Keep child-repository delivery and parent dependency-reference delivery separate. Merge and verify the child PR first; then advance the parent to the verified child revision in its own focused PR.
7. Verify every initialized child is clean and at its intended revision. Run the plugin's focused checks, `git diff --check`, the dependency-reference inspection appropriate to the agent, and the target agent's configuration validation.
8. Confirm the target agent discovers and executes the plugin from the resulting installation path. Reload, restart, or begin a new session when the target agent does not dynamically reload plugin code.

## Routine synchronization

- For normal configuration synchronization, initialize the parent-recorded plugin revisions; do not float plugin versions.
- Advance a plugin only when intentionally preparing a reviewed parent dependency-reference update.
- Treat a dirty child, branch mismatch, wrong remote, or dependency reference that differs from the parent-recorded revision as a delivery condition to inspect—not as permission to reset, clean, force-push, or bypass hooks.

## Completion evidence

Report the plugin repository URL, merged plugin SHA, parent dependency-reference commit URL and SHA, the target-agent checks run, and confirmation that the active configuration checkout is pinned to the verified plugin revision.
