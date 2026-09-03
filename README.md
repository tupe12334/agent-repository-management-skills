# Agent Repository Management Skills

This repository is the source of truth for reusable guidance on backing up and managing an autonomous agent's declarative configuration and reusable skills.

It does not manage, mirror, or back up the agent application's source code. Application source belongs to its upstream product repository; this library governs only the configuration repository that an operator owns.

## Contents

- `skills/autonomous-ai-agents/agent-repository-management/` — establish and operate a repository as the canonical, auditable backup of agent configuration and skills.

## Consumer contract

A consumer repository tracks declarative configuration and reusable skills, while excluding agent application source code, machine-local secrets, and runtime state. Changes to this library are delivered here first through a reviewed merge; consumers then update only their submodule gitlink.
