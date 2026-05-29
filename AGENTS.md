# Agent Instructions

## Versioning

Always bump the plugin version in `.claude-plugin/plugin.json` when making changes that will be committed.

The project uses semantic versioning (`MAJOR.MINOR.PATCH`):

| Change type | Version field | Example |
|---|---|---|
| New skill or feature (`feat:`) | MINOR — reset PATCH to 0 | `1.15.0` → `1.16.0` |
| Bug fix or docs update (`fix:`, `docs:`) | PATCH | `1.16.0` → `1.16.1` |
| Breaking change | MAJOR — reset MINOR and PATCH to 0 | `1.16.1` → `2.0.0` |

Include the version bump in the same commit as the change, and reference the new version in the commit message (e.g. `feat: add my-skill and bump to v1.16.0`).
