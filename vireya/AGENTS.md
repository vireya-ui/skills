# Maintainer notes (this repo only)

This file is **not** distributed. It stays here to brief contributors editing the skill.

## Distribution surface

The Vireya skill ships through three channels — keep them in sync when editing:

1. **`SKILL.md` + `references/*.md` (this directory)** — the skill consumed by Claude Code (and anything that reads `skills/vireya/`). Slim dispatcher (~110 lines) + 6 references on demand.
2. **`packages/ui/LLMS.md`** — bundled inside the `@vireya/ui` npm tarball, lands at `node_modules/@vireya/ui/LLMS.md` for the consumer. **Comprehensive offline doc** — duplicates the canonical setup, Section primitive, color pairing, CTA matrix, self-audit. Edited manually; not auto-generated.
3. **`https://www.vireya.dev/llms.txt` + `/llms-full.txt` + `/llms/<file>.md`** — served by `apps/www/src/app/llms*` route handlers, reading from `apps/www/src/lib/llms-source.ts`, which in turn reads `skills/vireya/SKILL.md` and `skills/vireya/references/*.md`. Editing the skill here automatically updates the web routes.

`llms-source.ts` only reads `SKILL.md` and the six named references (`design`, `recipes`, `setup`, `blocks`, `tokens`, `gotchas`). New files in `skills/vireya/` don't get served unless the metadata array there is extended. **This `AGENTS.md` is therefore not exposed.**

## Editing rules

- **SKILL.md must stay slim.** Target ≤200 lines. Push deep content into `references/*.md`. The file is loaded into the agent's context every time the skill triggers — every byte costs tokens.
- **References are on-demand.** Don't duplicate content from one reference into another; cross-link instead.
- **No internal paths in distributed files.** Anything mentioning `packages/ui/`, `apps/www/`, or `skills/vireya/` belongs in this `AGENTS.md` only. Distributed files reference `node_modules/@vireya/*` and `references/*.md`.
- **`packages/ui/LLMS.md` is a consumer-facing artifact.** When editing canonical setup, Section primitive, color pairing, CTA matrix, emphasis rule, or self-audit — update there too. Other content (token catalog, design judgment, gotchas) lives only in references.
- **No `<Text size={N}>` numeric escape** anywhere in distributed docs (removed in 0.1.0). Always semantic names.
- **No legacy token references** (`--v-font-N`, `--v-radius-N`, `--v-radius` bare). Always semantic (`--v-font-size-body`, `--v-radius-md`, etc.). Run `node scripts/migrate-tokens.mjs <path>` if you accidentally introduce them.

## Sync checklist when changing rules

If you change any of:
- Non-negotiable rules
- Token cheat sheet
- Self-audit checklist
- Setup recipe (icon map, layout files, ThemeProvider wiring)

Update both `SKILL.md` and `packages/ui/LLMS.md`.

## Distribution divergence by design

`SKILL.md` is a dispatcher → no inline setup recipe, no inline Section primitive, no inline contrast matrix. `LLMS.md` is comprehensive offline → keeps all of those inline so an offline npm consumer never needs to ask. They're not duplicates anymore: different audiences, different envelopes.
