# MuxPod - Agent Instructions

## Project Reference

See `CLAUDE.md` for project overview, tech stack, directory structure, and development commands.

## Fork-specific Changes

This repository is a fork of [moezakura/mux-pod](https://github.com/moezakura/mux-pod).
For fork-specific patches, modifications, and learnings, see `AGENTS-fork.md`.

## Code Style

- **Code comments**: Always in English.

## Branch & PR Workflow

1. **Work on `fork/kargnas`**: All development happens on this branch first.
2. **On completion, ask the user**: "새 브랜치로 옮겨서 PR 만들까요?" — do NOT auto-create branches.
3. **If user agrees**, create a new branch from `fork/kargnas` with the relevant commits.
4. **Fork-only files**: `AGENTS.md` and `AGENTS-fork.md` must NOT be included in PR branches. These files live only on `fork/kargnas`. When cherry-picking or creating PR branches, exclude them.
5. PRs target `upstream/main` (moezakura/mux-pod).
6. **PR title**: English (Conventional Commits format).
7. **PR body**: Japanese (upstream maintainer's language).
