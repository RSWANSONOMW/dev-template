# McB OS — Azure Naming Standard

This file is the canonical Azure naming reference for McB OS. Load it as a rule
before provisioning, renaming, or referencing any Azure resource. If a resource
name in code, scripts, or documentation doesn't match this standard, flag it —
don't silently "correct" it, since one known exception exists (see below).

## Core Pattern

```
<type>-mcbos-<component>-<env>
```

- **`<type>`** — short resource-type prefix (see table below)
- **`mcbos`** — fixed literal, always present, identifies the McB OS platform
- **`<component>`** — the specific service/purpose within McB OS (e.g. `shared`, `test`, a subsidiary name)
- **`<env>`** — environment suffix: `prod`, `staging`, `dev`, `test`

Segments are separated by hyphens. Where a resource type doesn't support
hyphens (e.g. Storage Accounts, Key Vault names in some cases), segments are
concatenated instead — see the Storage Account row below for the pattern.

## Resource Type Prefixes

| Resource Type            | Prefix | Example                       |
|---------------------------|--------|--------------------------------|
| Resource Group             | `rg`   | `rg-mcbos-shared-prod`         |
| Azure Data Factory         | `adf`  | `adf-mcbos-shared-prod`        |
| Azure SQL Server           | `sql`  | `sql-mcbos-shared-prod`        |
| Azure SQL Database         | `db`   | `db-mcbos-shared-prod`         |
| Storage Account            | `st`   | `stmcbostestdata`              |
| Key Vault                  | `kv`   | *see exception below*          |

> Storage Accounts don't allow hyphens, so the prefix and remaining segments
> are concatenated in lowercase, no separators: `st` + `mcbos` + `<component>` +
> (env, if needed). Example: `stmcbostestdata`.

## Environment and Scope Values in Current Use

- **Subscription:** `mcbos-enterprise`
- **Primary resource group:** `rg-mcbos-shared-prod`
- **Region:** South Central US
- **Environment suffix in use so far:** `prod` (staging/dev not yet provisioned)

## Known Exception — Do Not "Fix"

- **Key Vault name: `mcb-keyvault`**
  This does *not* follow the `kv-mcbos-<component>-<env>` pattern. This is a
  deliberate, known deviation — not an oversight. Claude Code should treat
  this as a fixed, correct name and must not rename it, "standardize" it, or
  flag it as an error in reviews, lint passes, or refactors. If a future
  Key Vault is provisioned, apply the standard pattern to the *new* vault;
  do not retroactively rename the existing one.

## Applying This Standard

When provisioning, referencing, or generating IaC/scripts for a new Azure
resource under McB OS:

1. Identify the resource type → look up its prefix above (if not listed,
   ask rather than guessing a prefix).
2. Confirm the component name (what the resource is for/serving).
3. Confirm the environment (`prod`, `staging`, `dev`, `test`).
4. Assemble as `<type>-mcbos-<component>-<env>`, adjusting for
   hyphen-incompatible resource types.
5. Cross-check against the Known Exception list before flagging any
   existing name as non-conforming.

## Schema-per-Source Note (Data Factory / SQL context)

Staging data lives in a single database (`db-mcbos-shared-prod`) using a
schema-per-source model rather than separate databases per source system
(e.g. `spectrum.*`, future `globalshop.*`). This is an architecture decision,
not a naming one, but it affects how schema names are chosen: schema names
should match the source system name in lowercase, no prefix needed since
they are scoped within the already-named staging database.
