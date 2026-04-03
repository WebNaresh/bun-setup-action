# CLAUDE.md

## What is this?

A reusable GitHub Composite Action that replaces 3 CI steps (setup Bun + cache node_modules + install) with 1 step. Built to solve the problem of GitHub Actions cache isolation between branches — PRs, main, and release workflows all share the same cache.

## Why it exists

GitHub Actions scopes caches by branch. A cache saved during a PR CI run is invisible to the release workflow on `main`, and vice versa. This means every workflow run does a full `bun install` (~40s) even though `node_modules` hasn't changed.

The `restore-keys` pattern fixes this by matching any cache with the same prefix regardless of branch. Combined with always running `bun install` (which takes ~2s when `node_modules` is already cached), this ensures fast installs across all workflows and branches.

## How it works

1. `oven-sh/setup-bun` — installs Bun
2. Auto-detects all `node_modules` directories (supports monorepos)
3. `actions/cache` with `restore-keys` — restores closest matching cache from any branch
4. `bun install --frozen-lockfile` — verifies/installs deps (~2s cached, ~40s cold)

## Architecture

- `action.yml` — the composite action definition (all logic is here)
- No JavaScript, no build step, no dependencies — pure YAML composite action

## Versioning

- `v1` tag points to latest stable — users reference `@v1`
- After changes: commit, move the `v1` tag: `git tag -f v1 && git push -f origin v1`
