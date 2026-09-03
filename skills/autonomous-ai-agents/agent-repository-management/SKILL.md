---
name: agent-repository-management
description: "Use when backing up an agent's configuration and reusable skills."
version: 1.3.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [agents, configuration, backups, skills, source-of-truth, submodules]
    related_skills: []
---

# Agent Repository Management

Use this skill when creating, operating, or changing an operator-owned repository that is the auditable source of truth for an autonomous agent's declarative configuration and reusable skills.

## Scope boundary

This repository type backs up the agent's configuration, not the agent application's source code.

- Track configuration such as `config.yaml`, reusable skills, configuration templates, hooks, managed integration definitions, and setup documentation.
- Do not track, mirror, vendor, or submodule the installed agent application source. That source remains in its upstream product repository and is maintained through its own development workflow.
- Do not treat a configuration backup as a deployable copy of the agent application. It restores operator-owned configuration after the agent application is installed separately.
- Keep machine-local secrets, chat-specific data, and mutable runtime state outside version control. Always exclude database files (for example, `*.db`, `*.sqlite`, and `*.sqlite3`) and session/conversation files (for example, `session.json` and `sessions/`), along with `.env`, credentials/auth files, memory stores, logs, caches, and generated runtime state. A Git commit is not a substitute for secure secret storage or a backup of live state.
- Make a personal agent configuration repository private by default. Change its visibility only when the user explicitly directs that repository to be public or otherwise shared; never infer publication intent from a repository name, template, or existing local files.

## Configuration backup model

1. Name one canonical remote, default branch, and local canonical checkout for each agent configuration repository. Verify the exact remote before making changes; directory names are not evidence of ownership.
2. Verify the remote visibility before the first push and after any ownership transfer. Personal agent configuration repositories must be private unless the user has explicitly directed otherwise.
3. Declare the tracked configuration and skill paths, plus ignored secret, chat-specific, and runtime paths. Add explicit ignore rules for database files and session/conversation state; review them before the first push and whenever a new agent feature adds local state.
4. Treat direct edits in a live agent home as drift unless that home itself is the canonical checkout. Make durable configuration and skill changes in the owning repository, then synchronize the live installation deliberately.
5. Before activating or synchronizing a repository with a live agent home, compare it with the live home and use a dry-run preview. Do not overwrite local state without explicit authorization.
6. Validate the candidate configuration with the agent's own configuration checker using the repository as its home, when available. Restart or open a new agent session when configuration or skills are not dynamically reloaded.

## Skills and shared repositories

1. A reusable skill library belongs in its own repository when multiple agents or configuration repositories consume it. Make that library the sole authoring location; consumers must not copy or fork individual skill content into their own trees.
2. Consume a shared skill library as a pinned Git submodule. Record its URL, path, and intended branch in `.gitmodules`; initialize it at the parent-recorded gitlink rather than floating it during routine synchronization.
3. Deliver changes in dependency order: validate and merge the skill-library change first, then update and deliver each consumer repository's gitlink. Verify both remote commits.
4. Preserve pre-existing dirty submodules and unrelated parent changes. Do not reset, clean, force-push, or bypass hooks to make a submodule update convenient.
5. Verify the agent's skill discovery layout after updating the gitlink. A submodule being cloned is not proof that the runtime can discover its skills.

## Change delivery

1. Discover prerequisites before editing: repository root, current branch, remote, clean/dirty state, upstream ancestry, submodules, and repository-provided validation commands.
2. Synchronize with the canonical remote branch before starting. When the main checkout has unrelated changes, use a clean worktree from the current remote tip and preserve the existing work.
3. Make the smallest scoped configuration or skill change. Do not bundle agent source code, database files, session/conversation files, other runtime files, secrets, local checkouts, or another agent's configuration.
4. Run the repository's real validation and `git diff --check`. Commit only after reviewing the intended diff.
5. Push a focused branch, open a pull request, resolve in-scope feedback and check failures, merge, then verify the merge SHA on the remote canonical branch.
6. Synchronize the canonical local checkout to that verified remote merge. Confirm its `HEAD` equals `origin/<default-branch>` before reporting delivery complete.

## Completion evidence

Report the configuration repository URL, merged commit URL and SHA, target branch, validation actually run, and whether the local canonical checkout equals the remote. Also report any preserved unrelated changes separately; never describe a dirty workspace as clean.
