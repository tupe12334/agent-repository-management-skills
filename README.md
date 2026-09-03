# Agent Repository Management Skills

This repository is the source of truth for reusable guidance on managing repositories that back autonomous agents: their source code, configuration, skills, and durable operational knowledge.

It is intentionally separate from any one agent's configuration repository. Consumers add this repository as a Git submodule and advance its pinned commit only after a validated upstream change is merged.

## Contents

- `skills/autonomous-ai-agents/agent-repository-management/` — establish and operate a repository as the canonical, auditable source of agent configuration and skills.

## Consumer contract

A consumer repository should track declarative configuration and reusable skills, while excluding machine-local secrets and runtime state. Changes to this library are delivered here first through a reviewed merge; consumers then update only their submodule gitlink.
