---
name: guidedtrack-builder
description: Create, edit, debug, and review GuidedTrack programs with strict documentation-backed syntax compliance. Use when an agent needs to turn requirements into GuidedTrack code, modify existing .gt files, add survey questions, branching, experiments, randomization, timing, media, scoring, pages, loops, or verify that code uses only documented GuidedTrack keywords and patterns. Also handles pushing finished programs to GuidedTrack.com for testing (Mac, Linux, and Windows/Git Bash) and reading run data back for debugging.
---

# GuidedTrack Builder

*Merged 2026-07-24 from the original skill, Isaac's study-recreation additions, Greg's Windows adaptation, and field-tested improvements from a production study build. `bin/gt` includes portable fixes and a `pull` subcommand.*

Build GuidedTrack programs by following the local language guide exactly. Treat GuidedTrack as a strict DSL: do not improvise keywords, syntax, or control-flow constructs.

## Official Documentation

If the local guide is incomplete, unclear, or appears stale, consult the official GuidedTrack docs at [docs.guidedtrack.com](https://docs.guidedtrack.com/).

- **Programmatic full-text search (best for agents):** download https://docs.guidedtrack.com/search-index.json - a static ~2MB JSON whose `docs` key holds every documentation entry (~600) with `title`, full `content` text, and `permalink` (with section anchors). Search the entries' text locally, ranking by any-term matches (do not require every term); the top match's `content` usually contains the answer directly. Retry with fewer or synonymous terms before concluding a topic is undocumented.
- **Sitemap:** https://docs.guidedtrack.com/sitemap/ - static list of every documentation page.
- **Browser search UI:** https://docs.guidedtrack.com/search/?query=terms (URL-encoded). For humans in a real browser ONLY - results render client-side, so a programmatic fetch always shows "No results" even when matches exist; never conclude a topic is missing from a fetched search page.
- Sections: the [GuidedTrack Manual](https://docs.guidedtrack.com/manual/) for core concepts and usage; the [Research Guide](https://docs.guidedtrack.com/research-guide/) for survey, experiment, and study-design guidance; the [Function & Keyword API](https://docs.guidedtrack.com/api/) for exact keyword and method semantics.
- Use the [Status page](https://status.guidedtrack.com/) when the issue looks like a platform or service problem rather than a code problem.

For GuidedTrack questions that are not resolved by the local reference, search the official docs before answering from memory.

## This skill's official home — contributing improvements back

The canonical version of this skill lives at **https://github.com/willfind/GuidedTrackAICodingAgentSkill** (your local copy of this skill is a clone of it). Whenever you make a meaningful improvement here — a new tested pattern, a `bin/gt` fix, or a hard-won caveat worth sharing — **ask the user whether to contribute it back to that repo so everyone using the skill benefits.** The mechanics are in [CONTRIBUTING.md](CONTRIBUTING.md) (branch → commit → open a pull request). **Never push to GitHub or open a pull request without the user's explicit permission.**

## Required Workflow

1. Read [references/complete_guide.md](references/complete_guide.md) before generating, editing, or reviewing GuidedTrack code. If the task needs data that outlives a single run — quotas, running averages, logins, piping answers between surveys — also read [references/custom-services.md](references/custom-services.md).
2. If the local guide does not fully answer the task, consult [docs.guidedtrack.com](https://docs.guidedtrack.com/) (see the search-index.json method above) and use the relevant official section before proceeding.
3. When recreating an existing study, first make a compact internal outline of its participant-visible screens: grouped content, display conditions, and next-screen destinations. Use the strongest available structural evidence. Inspect visual sources visually, and do not assume that a physical PDF page equals one survey screen. If a material boundary remains ambiguous, ask the user or state the assumption.
4. Identify the requested features, question types, navigation, data handling, and response format.
5. Inventory the keywords and sub-keywords that appear in the retrieved guide and match each requested feature to documented syntax.
6. If a needed keyword or sub-keyword does not appear in the retrieved guide, do not use it. Find a documented alternative or note the limitation in a GuidedTrack comment.
7. Generate or edit code using only documented keywords and the exact syntax patterns shown in the guide.
8. Run the validation checklist below before responding.

## Output Contract

- Return only GuidedTrack code and `--` comments.
- Do not include prose, explanations, or Markdown fences.
- Keep output within 100 lines for simple requests; match the scale of the task otherwise (real programs can run to thousands of lines).
- Use descriptive variable names (letters, digits, and underscores only - no spaces or special characters) such as `userName`, `stressScore`, or `selectedOption`.
- Prefer simple, generalizable code over clever or compressed code.

## Validation Checklist

- Every keyword and sub-keyword starts with `*`.
- Keywords that take a value use `*keyword: value` (colon + space); flag-style keywords (`*quit`, `*clear`, `*blank`, `*shuffle`, `*confirm`, `*other`, `*throwaway`, `*reset`, `*page`, `*html`, `*list`, and bare `*component`) stand alone with no colon.
- Indentation uses tabs only, exactly one tab per nesting level.
- Answer choices sit exactly one tab below `*question`.
- Variable assignments use single-line `>> name = value` expressions.
- Variable names contain only letters, digits, and underscores.
- Every keyword and sub-keyword used appears in [references/complete_guide.md](references/complete_guide.md).
- Question types and sub-keywords match documented patterns.
- When recreating a study, participant-visible screen grouping and order match the source on every reachable branch; any unavoidable mismatch is explicitly noted.
- Loops that can run many iterations start their body with a `*trigger:` line (see the guide's loop section).
- The final output contains only GuidedTrack code and comments.

**Automating the checklist.** Most of the rules above can be checked mechanically by GTLint, a third-party linter and formatter for `.gt` files. When you are writing local files and it is available, run it rather than checking by hand — see [references/gtlint.md](references/gtlint.md). It is optional and cannot reach the browser editor, so when it is unavailable, work the checklist manually and move on.

## Authoring Rules

- Use plain text for screen text. Use `*header:` only for headings. Use `*question:` only when an answer is required.
- For new programs without a source study, default to one question per page. When recreating a study, preserve its participant-visible page breaks and question groupings; source fidelity overrides this default. Do not merge or split source screens for convenience.
- Use `*page` when multiple items must share a screen, and always indent content under it.
- Put at least one blank line before each `*question`.
- Prefer 7-point Likert scales for bipolar subjective questions and 5-point Likert scales for one-sided subjective questions.
- Use number inputs for factual numeric entry. Use sliders only for 0-100 style responses such as percentages or frequencies.
- Use `*if: not (...)` instead of `!=`.
- Do not use `*else:` or `+=`.
- When media is requested, use the exact placeholder URLs from the guide instead of inventing URLs.
- Give every repeated question a distinct `*save:` variable name (e.g. `{itemId}_rating`) - run-data pages show only question TEXT, and the data CSV keys columns by variable name, so distinct save names are the only reliable join key. When the name can only be computed at run time (looping over items, for instance), use `data::store(name, value)` instead — see the guide's `data::store` section.

## Block Structure

- Indentation defines nesting. A block ends at the first later line whose indentation
  returns to the parent level.
- **A comment at column 0 does NOT close a block.** Comment lines (`--...`) are ignored
  for nesting, so the block continues past them. This matters most when generating or
  appending code programmatically: the rule "the block ends at the first column-0 line"
  is wrong, and following it will silently place new content OUTSIDE the block it was
  meant to join.

  ```
  *randomize: all
  	*group
  		>> id = "a"
  --this comment does not close the block; the groups below are still inside it
  	*group
  		>> id = "b"
  *if: somethingElse          <- this is what actually closes the *randomize block
  ```

  Real failure this prevents: appending new `*group` blocks to the END of a large
  rate-images file placed them after a trailing top-level `*if:` section rather than
  inside the `*randomize: all` block. They would have been shown unrandomised, in file
  order, and only to participants who entered that conditional.

  When inserting generated blocks: locate the block end as *the first column-0 line that
  is not a comment*, insert before it, and then assert that the expected number of items
  really do sit inside the intended block before pushing.

## Editing And Review

- When editing existing GuidedTrack code, preserve the user's text and flow unless the request requires structural changes.
- When reviewing GuidedTrack code, prioritize undocumented keywords, bad indentation, invalid question structure, unsupported operators, and response-format violations.
- If the user asks for a fix, patch minimally and keep the result within documented syntax.

## Custom Services (server-side backend)

When a program needs state that outlives a single run — enforce a quota, compare a participant to a running average, log people in, pipe answers between surveys — the mechanism is a **custom service**: server-side JavaScript plus a database hosted inside GuidedTrack, called from the program with `*service:`. See [references/custom-services.md](references/custom-services.md) for the route syntax, the `guidedtrack-db` API, and worked examples.

Two things to get right before writing any code:

- **Custom services are configured in the browser, not in `.gt` files.** Creating the service, its routes, its tables, and its route JavaScript all happen on guidedtrack.com; there is no `gt` subcommand and no documented API. Write the route code and the `*service:` block, then walk the user through pasting them in — and do not report the service as set up until they have.
- **The program must be connected to the service** (program Settings → Services → **Add internal service**, *not* "Add external service", which is for third-party APIs). Nothing in the program's source shows whether this was done, so an unconnected service looks like broken code.

## Pushing to GuidedTrack

When the user asks to upload, push, or test a program on GuidedTrack.com, follow these steps.

### Prerequisites

**Before proceeding, locate `gt` and `jq`. Stop only if neither PATH nor `~/bin/` has them.**

```bash
# Check PATH first, then fall back to ~/bin/ — non-login shells (including the
# Claude Code Bash tool) often do NOT have ~/bin on PATH even when the binaries
# exist there. A "not found" from `command -v` does not mean the binary is missing.
command -v gt || ls ~/bin/gt
command -v jq || ls ~/bin/jq ~/bin/jq.exe
```

If both binaries exist somewhere, proceed — but note the path. When invoking `gt`, use its absolute path (e.g. `~/bin/gt`) AND prefix the command with `PATH="$HOME/bin:$PATH"` so that `gt` can find `jq` internally (it shells out to `jq` and inherits the caller's PATH).

```bash
PATH="$HOME/bin:$PATH" GT_ENV=production ~/bin/gt push -o "<program name>"
```

**If `gt` is missing:** install this skill's copy (it contains portable fixes — a plain jq filter argument instead of bash process substitution, and glob-safe selector handling so program names with spaces work; do not substitute an older copy).

```bash
mkdir -p ~/bin
cp "<path-to-this-skill>/bin/gt" ~/bin/gt
chmod +x ~/bin/gt
```

**If `jq` is missing:**

- macOS Homebrew: `brew install jq`
- macOS direct download: `curl -fsSL -o ~/bin/jq https://github.com/jqlang/jq/releases/latest/download/jq-macos-arm64 && chmod +x ~/bin/jq` (use `jq-macos-amd64` on Intel)
- Linux: install via the system package manager, or download `jq-linux-amd64` the same way.
- Windows: see the Windows notes below.

Do not proceed until both `gt` and `jq` are reachable (either on PATH or via `~/bin/`).

### Non-interactive credentials (optional)

`gt create` and `gt push` prompt for GuidedTrack email, password, and a `production` confirmation. They use plain `read` from stdin (not `/dev/tty`), so piped credentials work and the agent can drive the entire push without the user touching a terminal.

The agent **may offer** this option; the user must **explicitly authorize** before the agent asks for credentials. Do not ask unprompted, and do not store credentials anywhere.

When authorized, prefer reading the password from a file so it never appears on a command line or in the process list:

```bash
cd ~/guidedtrack && { echo '<email>'; cat <path-to-password-file>; echo; echo 'production'; } \
  | PATH="$HOME/bin:$PATH" GT_ENV=production ~/bin/gt push -o "<program name>"
```

**Trap: "Aborting push" with piped credentials.** If the password file has NO trailing newline, the password line and the `production` line merge into one, the confirmation read gets nothing, and gt prints "Aborting push" with no other error. The explicit `echo` after `cat` (as above) prevents this.

**Security note to surface to the user when offering this:** if the password is typed inline (e.g. via `printf`) it will appear in the chat transcript; recommend they rotate it after the session or use a password file.

If the user declines, fall back to giving them the exact command to run themselves in a terminal (Git Bash on Windows, not PowerShell).

### Step 1 — Confirm the program name

The filename written to disk must exactly match the program name as it appears on GuidedTrack.com (case-sensitive, spaces matter). Ask the user for the exact program name if not already known. If the program name contains characters that are invalid in filenames on the current OS (on Windows: `\ / : * ? " < > |`), the filename-matching approach cannot work; use the direct-API fallback below instead.

### Step 2 — Save the code to a file

Save the generated code to `~/guidedtrack/<program-name>` — no file extension. The filename is the program name verbatim. Write the file with LF line endings; if bash later complains about `$'\r': command not found` or pushes garbled content, the file picked up CRLF endings and needs them stripped.

```bash
mkdir -p ~/guidedtrack
```

Keep `~/guidedtrack/` as a clean staging folder containing only files you intend to push — a bare `gt push` iterates over every file in the directory.

### Step 3 — Handle new vs. existing programs

`gt push` matches files by filename against program names on GuidedTrack. If the program does not yet exist, `gt push` silently skips it with "Program named … not found, skipping".

- **Existing program:** proceed directly to Step 4.
- **New program:** run `gt create` first from `~/guidedtrack/` to register the name on GuidedTrack (with empty content), then proceed to Step 4. Confirm with the user before running.

```bash
cd ~/guidedtrack && PATH="$HOME/bin:$PATH" GT_ENV=production ~/bin/gt create
```

`gt create` prompts for email, password, then requires typing `production` to confirm.

### Step 4 — Push the code

Default to `gt push -o "<program name>"` so only the named program is pushed. Always double-quote the name — GuidedTrack program names routinely contain spaces:

```bash
cd ~/guidedtrack && PATH="$HOME/bin:$PATH" GT_ENV=production ~/bin/gt push -o "<program name>"
```

The command prompts for GuidedTrack email and password, then requires typing `production` to confirm before uploading. (Or use the piped pattern above if authorized.)

Use `gt push .` only when the user explicitly wants to push every file in the directory.

A successful push prints `>> Updating "<name>" (id: <id>)... done` and the server compiles the program asynchronously.

### Fallback — direct API calls

If the `gt` script itself misbehaves, replicate its two API calls directly with curl (validated on Mac and Windows; from Windows PowerShell call `curl.exe` explicitly, since bare `curl` in PowerShell 5.1 is an alias for `Invoke-WebRequest`):

```bash
# 1. Find the program id (URL-encode the name; spaces become %20):
curl -sS -u '<email>:<password>' \
  "https://www.guidedtrack.com/programs.json?query=<url-encoded-name>" \
  | jq 'map(select(.name == "<exact program name>"))[0].id'

# 2. PUT the file contents to that id:
cat "./<program name>" \
  | jq --slurp --raw-input '{contents: ., program: {description: ""}}' \
  | curl -sS -X PUT -u '<email>:<password>' -d @- \
      -H 'Content-Type: application/json' \
      "https://www.guidedtrack.com/programs/<id>.json"

# 3. The PUT returns a job_id; confirm the compile finished:
curl -sS -u '<email>:<password>' "https://www.guidedtrack.com/delayed_jobs/<job_id>"
# -> {"status":"finished"}
```

The PUT response also contains `edit_program_url` and a preview `run_path` — share these with the user.

To download a program's current source, use this skill's `gt pull -o "<program name>"` (writes `./<program name>`; prompts for email and password only — no production confirmation, since pulling is read-only). It extracts the source from the `/programs/{id}/edit` page's `<textarea>`; if `gt` is unavailable, fetch that page while authenticated and HTML-unescape the largest textarea yourself.

### Step 5 — Verify it actually compiles, not just that the bytes match

Verified 2026-08-21/22: two syntax errors and one semantic error each reached "verified" state this way. Re-pulling and byte-comparing proves the SOURCE landed. It does NOT prove the program parses: GuidedTrack stores source containing syntax errors and only reports them when the program is opened or run. A push can therefore report "verified" on a program that cannot run at all.

After any push, open or run the program once, or have the user do so, and read the error banner.

**A push can be a transient no-op even when the file is fine.** Separately from a syntax error, `gt push` sometimes prints its success line while the site keeps the old bytes, and pushing the identical file again lands it. Treat one push as an attempt, not a result: re-pull, byte-compare, and retry until the site matches, giving up loudly rather than silently after a few tries. Automate this if you push often — the failure is invisible otherwise, and a half-applied set of related programs is worse than none.

**If a push fails, do not assume your local file survived.** A re-pull to verify overwrites the local copy with the site version, so an unverified push followed by a re-pull can silently revert your work. Keep the version you intend to send in a separate staging copy and compare against that.

**Diagnosing a push that will not land.** Push a version of the SAME file with one comment line added. If that lands, the transport and the program name are fine and your file has a syntax error - bisect it. If it does not land, the problem is the name, credentials, or the CLI. This turns a four-attempt mystery into one probe.

**Scope your own verification assertions to the block you edited.** A blanket check like `"\t\t*goto:" not in source` matches legitimate code elsewhere; when it aborted an edit script before the write, the next push re-sent the OLD file and reported VERIFIED.

### Windows notes

- `gt` is a Bash script: on Windows run it under Git Bash — which is what the Claude Code Bash tool uses. **Never PowerShell or cmd.** In Git Bash, `~` resolves to `C:\Users\<name>`, so `~/bin` and `~/guidedtrack` are ordinary Windows folders.
- jq is `jq.exe` on Windows; Git Bash resolves the bare name `jq` to it automatically. Install without admin rights: `curl -fsSL -o ~/bin/jq.exe https://github.com/jqlang/jq/releases/latest/download/jq-windows-amd64.exe` — or via winget (PowerShell/cmd, installs onto PATH): `winget install jqlang.jq`.
- This skill's `bin/gt` contains the fixes that make Windows work (native jq.exe cannot read bash process substitution; unquoted selectors word-split names with spaces). Always install this skill's copy.

### Edge Cases

- **Silent skip:** if a program reports "not found, skipping," the filename does not exactly match the GuidedTrack program name.
- **`gt create` on existing program:** returns an error but does not abort or overwrite; safe to run.
- **`jq: parse error` / `curl: Failed writing body`:** you are running an old `gt` script with Windows jq.exe. Install this skill's `bin/gt`.
- **`$'\r': command not found`:** CRLF line endings crept into `~/bin/gt` or the program file; rewrite with LF endings.
- **Working directory resets between Bash calls:** chain `cd ~/guidedtrack && ...` inside a single command rather than relying on a prior `cd`.

## Downloading And Reading Run Data

All methods require authentication. The reliable approach is a real browser session: GET /users/sign_in, extract the form's authenticity_token, POST it back with user[email] and user[password], and keep the cookies. (Plain HTTP Basic auth also works for most GET endpoints, but not all.)

### The data CSV (primary method - same file as the Data page's download button)

`GET https://www.guidedtrack.com/programs/{programId}/exports?export_format=csv`

- One row per run; columns are run metadata (Run, Program Version, User, Time Started (UTC), Time Finished (UTC), Minutes Spent, Position, Points) followed by the UNION of every variable set by any exported run - including variables set inside subprograms the run called.
- `export_format=sav` gives an SPSS .sav file instead. These two are the only export formats offered by the UI.
- Omitting `export_format` returns a header-only CSV - it looks like "no data" but just means the parameter is missing.
- The export EXCLUDES runs marked type "test" in the UI. A program with only test runs exports an empty (header-only) file. There are no other query filters (date/version parameters are ignored), so filter rows yourself after download.
- Column count grows as runs progress: a variable's column only exists once some exported run has set it. Do not treat a missing column as a program bug mid-collection.
- Collections and associations export as their literal text representation in one cell; keep them parseable or mirror important values into scalar variables.

### Per-run inspection (works for test runs too - the debugging tool)

- `GET /programs/{id}/runs.json` - every run with id, start/end time.
- `GET /runs/{runId}/answers` - HTML table of one run's answers: question text, answer, timestamp. Parse the `<tr>` rows. Includes test runs, and works while a run is still in progress (great for live monitoring).
- Each answer row links to `/programs/{id}/answers/<hash>` - one page per question NODE. Questions with IDENTICAL text share one hash, distinguished by suffix `_<n>` where n is the 0-INDEXED instance number in program source order. This recovers WHICH of many identically-worded questions an answer belongs to (e.g. which stimulus a rating refers to). Validate any suffix-to-item mapping against known constraints of a run (expected item count, known filters) before trusting it - the indexing is easy to get off by one.
- Program SOURCE: use this skill's `gt pull -o "<program name>"`, or fetch the `/programs/{id}/edit` page - the source sits in the largest `<textarea>` (HTML-unescape it).

### Dead ends (do not rediscover these)

- "Package code" / `generate_zip` / `download` produce the program CODE zip, not response data.
- "Analyze with Hypothesize" transfers data server-side via a one-time transferId; it is not fetchable directly.
- `/runs/{id}/answers.json` and most other guessed JSON endpoints return 404/406.
