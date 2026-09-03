---
name: agent-repository-management
description: "Use when managing repositories that are an agent's source of truth."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [agents, repositories, source-of-truth, configuration, skills, submodules]
    related_skills: []
---

# Agent Repository Management

Use this skill when creating, operating, or changing a repository that is the auditable source of truth for an autonomous agent's source, declarative configuration, reusable skills, or deployment definitions.

## Ownership model

1. Name one canonical remote, default branch, and local canonical checkout for each managed agent repository. Verify the exact remote before making changes; directory names are not evidence of ownership.
2. Keep declarative, reproducible assets in the repository: configuration templates, checked-in skills, hooks, dependency declarations, and documented setup instructions.
3. Keep machine-local secrets and mutable runtime state outside version control. Typical exclusions include `.env`, credentials/auth files, sessions, memory databases, logs, caches, and generated runtime state. Do not use a repository commit as a substitute for secure secret storage or backup of live state.
4. Treat direct edits in a live agent home as drift unless the home itself is the canonical checkout. Make durable changes in the owning repository and synchronize the live installation deliberately.

## Change delivery

1. Discover prerequisites before editing: repository root, current branch, remote, clean/dirty state, upstream ancestry, submodules, and repository-provided validation commands.
2. Synchronize with the canonical remote branch before starting. When the main checkout has unrelated changes, use a clean worktree from the current remote tip and preserve the existing work.
3. Make the smallest scoped change. Do not bundle unrelated runtime files, secrets, local checkouts, or another agent's configuration.
4. Run the repository's real validation and `git diff --check`. Commit only after the intended diff is reviewed.
5. Push a focused branch, open a pull request, resolve in-scope feedback and check failures, merge, then verify the merge SHA on the remote canonical branch.
6. Synchronize the canonical local checkout to that verified remote merge. Confirm its `HEAD` equals `origin/<default-branch>` before reporting delivery complete.

## Skills and shared repositories

1. A reusable skill library belongs in its own repository when multiple agents or configuration repositories consume it. Make that library the sole authoring location; consumers must not copy or fork individual skill content into their own trees.
2. Consume a shared skill library as a pinned Git submodule. Record its URL, path, and intended branch in `.gitmodules`; initialize it at the parent-recorded gitlink rather than floating it during routine synchronization.
3. Deliver changes in dependency order: validate and merge the skill-library change first, then update and deliver each consumer repository's gitlink. Verify both remote commits.
4. Preserve pre-existing dirty submodules and unrelated parent changes. Do not reset, clean, force-push, or bypass hooks to make a submodule update convenient.
5. When a source-controlled agent configuration repository tracks skills as submodules, verify the agent's skill discovery layout after updating the gitlink. A submodule being cloned is not proof that the runtime can discover its skills.

## Agent configuration repositories

For an agent configuration repository that backs a live agent home:

1. Declare which tracked paths are configuration and reusable skills, and which ignored paths are local runtime state.
2. Before activation or synchronization, compare the repository with the live home and use a dry-run preview. Do not overwrite local state without explicit authorization.
3. Validate the configuration using the agent's own configuration checker with the candidate repository as its home, when available.
4. After a successful merge, update the canonical live checkout, initialize pinned submodules, and rerun configuration validation. Restart or open a new agent session when the runtime does not reload configuration or skills dynamically.

## Completion evidence

Report the canonical repository URL, merged commit URL and SHA, target branch, validation actually run, and whether the local canonical checkout equals the remote. Also report any preserved unrelated changes separately; never describe a dirty workspace as clean.
