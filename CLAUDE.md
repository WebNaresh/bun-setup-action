# CLAUDE.md

## What is this?

`bun-setup-action` — a GitHub Marketplace Composite Action that replaces 3 CI steps (setup Bun + cache node_modules + install) with 1 step. Solves GitHub Actions cache isolation between branches.

**Marketplace:** https://github.com/marketplace/actions/bun-setup-with-cache
**Repo:** https://github.com/WebNaresh/bun-setup-action

## Problem it solves

GitHub Actions scopes caches by branch. PR cache is invisible to `main` release workflow and vice versa. Every run does full `bun install` (~40s). The `restore-keys` pattern matches any cache with the same prefix regardless of branch. With restored `node_modules`, `bun install` just verifies (~2s).

## Architecture

```
action.yml          — The entire action (composite YAML, no JS/build step)
.releaserc.json     — Semantic release config (auto-publishes on push to main)
.github/workflows/
  release.yml       — Runs semantic-release + moves v1 tag to latest
```

## How action.yml works (step by step)

1. **Setup Bun** — `oven-sh/setup-bun@v2` with user-specified version
2. **Detect node_modules paths** — reads `package.json` workspaces field, resolves actual subdirectories with `package.json`, builds list of `*/node_modules` paths. Falls back to just `node_modules` for non-monorepo.
3. **Cache** — `actions/cache@v4` with `restore-keys` for cross-branch sharing
4. **Install** — `bun install --frozen-lockfile` (fast when cached)

## Inputs/Outputs

| Input | Default | Purpose |
|-------|---------|---------|
| `bun-version` | `latest` | Bun version |
| `working-directory` | `.` | Where to run `bun install` |
| `frozen-lockfile` | `true` | Use `--frozen-lockfile` |
| `cache-key-prefix` | `bun-deps` | Cache key prefix |
| `node_modules-paths` | *(auto)* | Override auto-detected paths |

| Output | Purpose |
|--------|---------|
| `cache-hit` | Exact cache key matched |
| `cache-restored` | Any cache restored (exact or partial) |

## Usage in consumer repos

```yaml
- uses: WebNaresh/bun-setup-action@v1
  with:
    bun-version: '1.3.7'
```

## Versioning & Release

- Semantic release on push to `main` — `fix:` → patch, `feat:` → minor, `BREAKING CHANGE` → major
- After semantic release creates `v1.x.y`, workflow auto-moves the `v1` tag to latest
- Users reference `@v1` — always gets latest v1.x.y
- First Marketplace publish was manual; all subsequent releases auto-appear on Marketplace

## Commands

```bash
# No build step — just edit action.yml and push
git add -A && git commit -m "fix: description" && git push origin main
# Semantic release handles tagging + GitHub Release + v1 tag move automatically
```

## Known consumers

- `WebNaresh/practise_stack` — all 4 workflows (validate, release, publish-leads-sdk, publish-mcp)

## When fixing issues

1. Edit `action.yml` (all logic is there)
2. Test locally by referencing the branch: `uses: WebNaresh/bun-setup-action@main`
3. Commit with conventional prefix (`fix:`, `feat:`)
4. Push to main — semantic release auto-publishes, `v1` tag auto-moves
5. Consumer repos get the fix automatically (they reference `@v1`)

## Common issues

- **"Input required and not supplied: path"** — auto-detect returned empty. Fix: ensure fallback to `node_modules` when no workspaces found.
- **Cache miss on every run** — check `restore-keys` is present. Without it, caches are branch-scoped.
- **Monorepo node_modules missing** — action reads `package.json` workspaces to find all workspace dirs. If a workspace has no `package.json`, it won't be detected → user should pass `node_modules-paths` manually.
