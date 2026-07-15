# Enfyra Documentation Agent Guide

This repository owns user-facing Enfyra documentation. Keep this file as a thin router; detailed authoring and synchronization behavior belongs in the matching skill.

## Hard Rules

- English content under `en/` is canonical. Every published English document must have a Vietnamese document at the identical relative path under `vi/`.
- Vietnamese documents use natural Vietnamese and preserve technical terms when translation would make them less precise. Do not translate identifiers, API paths, code, package names, environment variables, or Enfyra runtime syntax.
- Every localized document except the repository root `README.md` declares an ASCII, slash-separated public slug in frontmatter.
- Write user documentation around goals, workflows, expected results, safety, and troubleshooting. Internal tool-call sequences, acknowledgement keys, repository macros, and implementation contracts belong in source skills or developer references unless a user must type them directly.
- Preserve Markdown heading hierarchy, code-fence balance, links, tables, and executable examples across languages.
- Do not seed documentation manually. The source-controlled Markdown is synchronized only through `../enfyra-landing-page/scripts/sync_docs.py` under the `enfyra-landing-docs-sync` skill.
- Do not commit, push, publish, or run a production synchronization unless the user has authorized that action.

## Required Skill Routing

- Writing, translating, reviewing, reorganizing, or locating documentation: `.codex/skills/enfyra-docs-authoring/SKILL.md`
- Synchronizing Markdown into landing content sets: `../enfyra-landing-page/.codex/skills/enfyra-landing-docs-sync/SKILL.md`
- Changing the landing docs reader, navigation, search, or rendering: `../enfyra-landing-page/.codex/skills/enfyra-landing-docs/SKILL.md`

## Repository Map

- `en/`: canonical English documents.
- `vi/`: Vietnamese documents with locale-specific public slugs.
- `en/README.md` and `vi/README.md`: repository entrypoints, directory maps, and learning paths.
- `getting-started/`: installation and first result.
- `api-reference/`: public REST usage.
- `integrations/`: external frameworks and MCP user setup.
- `app/`: Enfyra Admin usage and UI extension behavior.
- `server/`: runtime concepts and developer scripting references.
- `examples/`: task-oriented recipes.
- `cloud/` and `docker/`: managed and self-hosted deployment paths.

## Verification

Run from `../enfyra-landing-page`:

```bash
yarn docs:check
```

Also run `git diff --check` in both repositories after documentation changes.
