# @sindresorhus/slugify

## What it is

A small ESM-only Node.js library (`@sindresorhus/slugify` v3.0.2, Node >= 20) that turns any string into a URL-safe slug: `slugify('I ♥ Dogs')` → `'i-love-dogs'`. It transliterates Unicode via `@sindresorhus/transliterate`, decamelizes, drops contraction apostrophes and collapses separators. `slugifyWithCounter()` returns a stateful variant that de-duplicates repeated slugs (`foo-bar`, `foo-bar-2`, ...).

## Map

- `index.js` — all runtime code: `slugify()` (default export), `slugifyWithCounter()` (named export), and private helpers `decamelize`, `removeMootSeparators`, `removeCounterSuffix`, `buildPatternSlug`.
- `overridable-replacements.js` — default `customReplacements` table (`&` → ` and `, `🦄` → ` unicorn `, `♥` → ` love `).
- `index.d.ts` — TypeScript types: `Options`, `CountableSlugify`, both function signatures, with JSDoc and `@example` blocks per option.
- `test.js` — AVA tests, one `test('<topic>', ...)` block per feature or option.
- `readme.md` — user docs; `##### <option>` section per option under `## API`.
- `package.json` — scripts, `exports` (`types` + `default`), `files` whitelist, xo config.
- `.editorconfig` — tabs, LF, final newline; YAML uses 2 spaces.
- `.github/workflows/main.yml` — CI: `npm install && npm test` on Node 24 and 20.
- `.npmrc` — `package-lock=false` (no lockfile is committed).
- `.xtend/config.json` — xtend settings (upstream `sindresorhus/slugify`, branch `main`).
- Not present any more (old history only): `replacements.js`, `index.test-d.ts`, `.travis.yml`.

## Build, run, test

No build step; the source is shipped as is.

```sh
npm install            # no lockfile, see .npmrc
npm test               # runs: xo && ava
npx xo                 # lint only
npx xo --fix           # autofix lint
npx ava                # tests only
npx ava --match='counter'   # one test block by title
node -e "import('./index.js').then(m => console.log(m.default('Foo Bar')))"
```

There is no `tsd` type test in the current tree; `index.d.ts` is checked only by xo's TypeScript rules.

## Extension points

- **New option**:
  1. Add a default to the `options = {...}` object in `slugify()` in `index.js`.
  2. Apply it at the right step of the pipeline (order: transliterate/custom replacements → decamelize → lowercase → contraction removal → `patternSlug` replace → strip `\` → `removeMootSeparators` → prepend `_` / append `-`).
  3. If it changes which characters survive, edit `buildPatternSlug()` (it builds the negated character class).
  4. Add a `readonly <name>?: <type>;` to `Options` in `index.d.ts` with a JSDoc block, `@default` and `@example`; copy the `preserveTrailingDash` block.
  5. Add a `##### <name>` section to `readme.md` under `### slugify(string, options?)`; copy `##### preserveTrailingDash`.
  6. Add `test('<name> option', t => {...})` to `test.js`; copy `test('lowercase option', ...)`.
- **New default replacement** (applies only when `transliterate` is true): add a `['<char>', ' <word> ']` pair to `overridable-replacements.js`. Pad with spaces so the word becomes its own slug segment. Update the `@default` line in `index.d.ts` (`customReplacements`) and the readme default list to match. Test it in `test('main', ...)` or `test('custom replacements', ...)`.
- **Language / script support**: transliteration lives in the `@sindresorhus/transliterate` dependency, not here. Add only a regression test here, copying `test('supports Turkish', ...)`; locale-specific behaviour goes through the `locale` option (`test('locale option', ...)`).
- **Decamelize rules**: edit the `decamelize` arrow in `index.js`; add cases to `test('decamelize acronyms', ...)`.
- **Counter behaviour**: edit `slugifyWithCounter()` and `removeCounterSuffix()` in `index.js`; tests go in `test('counter', ...)` and `test('slugifyWithCounter() never returns the same slug twice', ...)`.
- **New exported function**: add a named `export function` in `index.js`, its declaration plus types in `index.d.ts`, a `### name()` section in `readme.md`, and tests. If it needs a new file, add the file to `package.json` `files`, or it will not be published.

## Conventions

- ESM only (`"type": "module"`); relative imports include the `.js` extension (`'./overridable-replacements.js'`).
- Tabs for indentation, single quotes, semicolons, no trailing commas on single lines but trailing commas in multi-line literals; xo enforces all of it. Run `npx xo --fix` before finishing.
- Prefer `String#replaceAll` with `/g` regexes, `String.raw` for regex source strings, `.at(-1)`, `Map`/`Set`. Escape any user-supplied text put into a `RegExp` with `escapeStringRegexp`.
- Regexes must stay linear-time: no unanchored nested quantifiers such as `(?:-\d+)+$`. See the ReDoS comments on `decamelize` and `removeCounterSuffix`, and `test('slugifyWithCounter() is not vulnerable to ReDoS', ...)`. Add a similar timing test for any new regex that touches user input.
- Comments explain *why* in full sentences, often with a `` `input` → `output` `` example. Keep them on the line above the code.
- Errors: `TypeError` for wrong argument types, `Error` for invalid option combinations, with the bad value in backticks in the message.
- Helpers are module-private `const name = (...) => ...` arrows; only `slugify` and `slugifyWithCounter` are exported.
- Every option exists in four places that must agree: the default in `index.js`, `Options` in `index.d.ts`, the readme section and a test. `Options` fields are `readonly` and optional.
- Tests use `t.is(slugify(input, options), expected)`, one assertion per line, grouped by topic in a named `test()` block. Use `t.throws` for errors.
- `engines.node` is `>=20`; don't use APIs newer than Node 20.

## Hot zones

Upstream changes these often (commit counts in recent history). Keep local edits there small, isolated and easy to rebase.

- `package.json` (39) — version bumps and dependency updates on every release. Don't reorder or reformat it; add only the keys you need.
- `index.js` (38) — the whole pipeline. Put local logic in new private helpers or a new module imported by `index.js`, and touch `slugify()` with one-line calls rather than rewriting steps in place. Don't reorder the existing pipeline.
- `test.js` (37) — add local tests as new `test()` blocks at the end of the file, or in a separate file such as `test-local.js` (AVA picks up `test.js` and `test-*.js` by default), not inside existing blocks.
- `readme.md` (30) — add new sections; don't reword upstream text.
- `index.d.ts` (16) — append new fields at the end of `Options`; don't reflow existing JSDoc.
- `overridable-replacements.js` (3) — rarely changed, but a local entry here changes default output for all callers. Prefer passing `customReplacements` at the call site.
- `.github/workflows/main.yml` — upstream owns CI. Put local CI in a separate workflow file.
