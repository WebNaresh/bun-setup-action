# Bun Setup with Cache

A GitHub Action that sets up Bun, caches `node_modules` across branches, and installs dependencies — all in one step.

## Why?

- **Cross-branch caching** — PR cache is shared with `main` and vice versa via `restore-keys`
- **Auto-detects monorepo** — finds all `node_modules` directories automatically
- **Fast installs** — with cached `node_modules`, `bun install` just verifies (~2s vs ~40s)
- **One step** — replaces 3 workflow steps (setup-bun + cache + install)

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: WebNaresh/bun-setup-action@v1
    with:
      bun-version: '1.3.7'
```

That's it. Replaces this:

```yaml
# Before (15+ lines)
- uses: oven-sh/setup-bun@v2
  with:
    bun-version: '1.3.7'

- uses: actions/cache@v4
  with:
    path: |
      node_modules
      apps/web/node_modules
      packages/*/node_modules
    key: ${{ runner.os }}-deps-${{ hashFiles('bun.lock') }}
    restore-keys: |
      ${{ runner.os }}-deps-

- run: bun install --frozen-lockfile
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `bun-version` | `latest` | Bun version to install |
| `working-directory` | `.` | Working directory for `bun install` |
| `frozen-lockfile` | `true` | Use `--frozen-lockfile` flag |
| `cache-key-prefix` | `bun-deps` | Custom cache key prefix |
| `node_modules-paths` | *(auto-detect)* | Custom paths to cache (newline-separated) |

## Outputs

| Output | Description |
|--------|-------------|
| `cache-hit` | `true` if exact cache key matched |
| `cache-restored` | `true` if any cache was restored (exact or partial) |

## Examples

### Simple project

```yaml
- uses: WebNaresh/bun-setup-action@v1
```

### Monorepo with pinned version

```yaml
- uses: WebNaresh/bun-setup-action@v1
  with:
    bun-version: '1.3.7'
```

### Custom node_modules paths

```yaml
- uses: WebNaresh/bun-setup-action@v1
  with:
    node_modules-paths: |
      node_modules
      apps/web/node_modules
      packages/shared/node_modules
```

### Conditional logic based on cache

```yaml
- uses: WebNaresh/bun-setup-action@v1
  id: setup

- name: Generate Prisma Client
  if: steps.setup.outputs.cache-hit != 'true'
  run: bunx prisma generate
```
