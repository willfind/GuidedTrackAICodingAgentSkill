# GTLint — linting and formatting `.gt` files

GTLint checks GuidedTrack source for undefined variables, invalid keywords and sub-keywords, broken `*goto:` targets, bad indentation, and unclosed strings and brackets — and reformats files to a consistent style. It automates most of the Validation Checklist in [SKILL.md](../SKILL.md), so run it whenever it is available instead of eyeballing the same rules by hand.

**Provenance and status.** GTLint is not part of GuidedTrack. It is a third-party tool published as `@jrc03c/gtlint` and maintained independently by a contributor to this skill. It is **optional**: this skill must work without it, so treat a missing GTLint as a non-event and fall back to the manual checklist. The guidance below was verified against version 0.15.5.

## When it applies

- **Local `.gt` files:** yes. Format, then lint, after every write.
- **Code written directly in the guidedtrack.com browser editor:** no — GTLint cannot reach it. If the user is editing in the browser and would benefit, mention that editing locally makes linting possible; do not insist.

Linting is not a substitute for pushing and running the program. It checks syntax and reference errors, not whether the study behaves correctly.

## Detect, then use or skip

```bash
npx --yes @jrc03c/gtlint --version
```

If that fails (no Node, no network, restricted environment), skip GTLint and use the Validation Checklist. Do not install anything globally without asking; `npx` fetches on demand and needs no project manifest.

## The workflow

Format first, then lint — formatting resolves whitespace and spacing issues that would otherwise show up as findings:

```bash
npx --yes @jrc03c/gtlint format --write program.gt
npx --yes @jrc03c/gtlint lint program.gt
```

Both commands accept a directory to process every `.gt` file beneath it. Other useful flags:

- `--quiet` — errors only, suppressing warnings.
- `--format json` — machine-readable output, best when the agent is parsing results.
- `--format compact` — one line per finding.
- `format` without `--write` prints the formatted result to stdout instead of modifying the file.

**Exit codes:** `0` when there are no errors, `1` when there are. Warnings alone still exit `0`, so check the output text rather than the exit code when warnings matter.

## Rules

| Rule | Default | Catches |
|---|---|---|
| `no-undefined-vars` | error | Variable used but never assigned |
| `valid-keyword` | error | Unrecognized `*keyword` |
| `valid-sub-keyword` | error | Sub-keyword not valid under its parent |
| `valid-subkeyword-value` | error | Invalid value for a sub-keyword |
| `required-subkeywords` | error | A required sub-keyword is missing |
| `no-invalid-goto` | error | `*goto:` to a label that does not exist |
| `no-duplicate-labels` | error | Two `*label:`s with the same name |
| `indent-style` | error | Spaces used instead of tabs |
| `correct-indentation` | error | Wrong nesting depth |
| `no-empty-blocks` | error | Keyword with nothing indented under it |
| `no-unclosed-string` | error | Unterminated `"string` |
| `no-unclosed-bracket` | error | Unclosed `[`, `{`, or `(` |
| `no-single-quotes` | error | `'text'` where GuidedTrack requires `"text"` |
| `no-stray-colon` | error | Stray `:` in an expression |
| `no-inline-argument` | error | Inline argument where none is allowed |
| `purchase-subkeyword-constraints` | error | Invalid `*purchase` sub-keyword combination |
| `no-unused-vars` | warn | Variable assigned but never read |
| `no-unused-labels` | warn | `*label:` never targeted |
| `no-unreachable-code` | warn | Code after `*goto:` or `*return` |
| `goto-needs-reset-in-events` | warn | `*goto:` inside `*events` with no `*reset` — see the guide's events section; the compiler rejects this outright, so treat the warning as an error |

Note: `required-subkeywords` does not require `*save:` on questions. Saving every answer is this skill's authoring preference, not a language rule.

## Cross-program and URL variables

GuidedTrack variables are global across `*program:` calls, and run-URL parameters define variables before the first line executes. GTLint sees one file at a time, so it reports both as undefined or unused unless told otherwise. Declare them in comments:

```
-- @from-parent: studyId, participantId
-- @from-url: source, campaign
-- @from-child: signupResult
-- @to-child: userEmail
-- @to-csv: participantAge, totalScore
```

`@from-*` suppresses `no-undefined-vars`; `@to-*` suppresses `no-unused-vars`. `@from-url` is an alias for `@from-parent`, and `@to-csv` an alias for `@to-parent` — the distinct names document intent.

Prefer these over disabling rules: they keep the checks on for everything else in the file.

## Suppressing rules inline

```
-- @gtlint-disable-next-line no-unused-vars
>> debugValue = someExpression

-- @gtlint-disable no-undefined-vars
-- ...region where variables come from elsewhere...
-- @gtlint-enable no-undefined-vars

-- @gtformat-disable
-- ...hand-formatted region the formatter should leave alone...
-- @gtformat-enable

-- @gt-disable-next-line
```

`@gtlint-*` affects linting, `@gtformat-*` formatting, and `@gt-*` both. Each accepts a comma-separated rule list; with no list, all rules are affected.

## Configuration (optional)

Defaults are sensible; add `gtlint.config.js` in the project root only when a project needs different severities or ignore patterns.

```javascript
export default {
  ignore: ['**/node_modules/**', '**/dist/**'],
  lint: {
    noUnusedVars: 'off',
    gotoNeedsResetInEvents: 'error',
  },
}
```

Severities are `"off"`, `"warn"`, or `"error"`. **Rule names are camelCase in the config file and kebab-case in inline directives** — the config loader converts between them, so `noUnusedVars` and `no-unused-vars` both work in the config, but only the kebab-case form works in a comment.
