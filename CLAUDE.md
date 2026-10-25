# Day.js

2KB immutable Moment.js-compatible date library. Plain JS (ES6, no TypeScript source), published as UMD + ESM. Default branch is `dev`.

## Architecture
- `src/index.js` – the whole core (`Dayjs` class, `dayjs()` factory, `extend`/`locale`). `constant.js`, `utils.js` are helpers.
- `src/plugin/<name>/index.js` – one dir per plugin, shape `export default (option, DayjsClass, dayjs) => {...}`; patch `DayjsClass.prototype`. Core stays tiny; features belong in plugins.
- `src/locale/<code>.js` – one file per locale (143). First line must be `// Language name [code]` (build parses it via regex). Imports `dayjs` by bare name; the build rewrites this.
- `types/` – hand-written `.d.ts`: `index.d.ts`, `types/plugin/<name>.d.ts`, `types/locale/`.
- `test/` mirrors `src/`: `test/*.test.js` core, `test/plugin/<name>.test.js`, `test/locale/<code>.test.js`, `test/issues/` for regressions.
- `docs/` is only a stub: real docs live at day.js.org (separate repo), so don't write plugin docs here.

## Commands
```bash
npm install                                   # npm, not yarn (yarn.lock is gitignored)
npx jest test/plugin/isToday --coverage=false # one test file; --coverage=false avoids noise
npx jest -t "is today" --coverage=false       # by test name
npm run lint                                  # eslint src/* test/* build/*; pre-commit hook runs this
TZ=Europe/London npx jest test/timezone.test --coverage=false   # timezone tests REQUIRE an explicit TZ
npm test                                      # full suite: reruns timezone tests under 4-5 TZs, then jest with 100% line-coverage gate (~10s)
npm run build                                 # rollup -> dayjs.min.js, locale/, plugin/ in repo root; ends with size-limit (2.99 KB)
npm run babel                                 # ESM build into esm/
```
Known failures on a clean `dev` checkout (6 tests, 3 suites; don't try to fix them, and don't use them as a signal): `dayOfYear` "DayOfYear set", `localizedFormat` "Should not interpolate characters inside square brackets", and four `plugin/timezone` DST cases (2012-03-11 / 2012-11-04) that fail only in `npm test`'s final jest run under the machine's own TZ. The same file passes with `TZ=America/New_York`. Any other failure is yours.

## Conventions
- Style: airbnb-base, **no semicolons**, **no trailing commas**, 2-space indent. Run lint; don't hand-fix style.
- Test idiom: `import dayjs from '../../src'` (never `'dayjs'`), plugin via `dayjs.extend(plugin)`, `MockDate.set(...)` in `beforeEach` / `MockDate.reset()` in `afterEach`, and assertions often compare against `moment(...)` output as the oracle. Flat `it('...')` blocks, no `describe`.
- Coverage must stay at 100% lines (`npm test` enforces it): every new branch in `src/` needs a test.
- Adding a plugin: need `src/plugin/<n>/index.js`, `types/plugin/<n>.d.ts`, `test/plugin/<n>.test.js`. `test/plugin/declarations.test.js` fails if the `.d.ts` is missing.
- Adding a locale: `src/locale/<code>.js` + `test/locale/<code>.test.js` (only some locales have tests).
- Commit messages: semantic (`fix:`, `feat:`, `chore:`), required by CONTRIBUTING.

## Guardrails
- Never hand-edit or commit build output: `/dayjs.min.js`, `/locale/`, `/plugin/`, `/esm/`, `/index.d.ts`, `locale.json`, `coverage/` (all gitignored, root-level). Edit `src/` instead. Don't confuse root `/locale` with `src/locale`.
- Don't edit `package-lock.json` (`npm install` dirties it; `git checkout package-lock.json`) or `CHANGELOG.md` / version fields (semantic-release owns them).
- Don't run bare `jest` / `npm run test-tz` and call timezone failures regressions; set `TZ`.
- Core size is budgeted (2.99 KB gzipped). Don't add code to `src/index.js` for something a plugin can do.
