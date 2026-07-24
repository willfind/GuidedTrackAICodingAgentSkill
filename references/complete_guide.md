# GuidedTrack Complete Guide

## Table of Contents

- Official documentation and troubleshooting
- Generation workflow
- Response contract
- Critical syntax rules
- Authoring preferences
- Core patterns
- Keyword inventory
- Language notes
- Common misconceptions

## Official Documentation And Troubleshooting

When this local reference is not enough, use the official GuidedTrack documentation at https://docs.guidedtrack.com/. How to navigate it efficiently:

- **Programmatic full-text search (best for agents): download https://docs.guidedtrack.com/search-index.json** - a static ~2MB JSON whose `docs` key maps ids to every documentation entry (~600 of them), each with `title`, full `content` text, and `permalink` (including section anchors like `/api/#collection-sort-direction`). This is the entire documentation corpus in one request. (The `index` key is a serialized lunr index - ignore it; plain text matching over `docs` works fine.) How to use it well:
	- Score entries by counting query-term occurrences in title + content, weighting title hits higher. **Match loosely: rank by ANY-term hits rather than requiring every term** - the docs may not use your exact vocabulary (e.g. "countdown time limit" finds nothing because the docs never say "limit"; "countdown" alone finds it). Retry with fewer or synonymous terms before concluding a topic is undocumented.
	- The `content` field is the FULL section text, not a teaser - the top match usually contains the answer directly, so you often need no second request. Fetch the `permalink` only when you want the rendered page or surrounding context.
	- Re-download per session rather than caching long-term (the file tracks doc updates); at ~600 entries it is also small enough to dump matching entries wholesale into context when a topic spans several sections.
- **Sitemap: https://docs.guidedtrack.com/sitemap/** - a static page listing every documentation page, good for browsing the path structure. Example paths: `/manual/additional-keywords/collections/`, `/manual/additional-keywords/associations/`, `/manual/asking-questions/standard-multiple-choice-questions/`, `/manual/additional-keywords/if-this-then-that/`, `/research-guide/randomizing-groups-and-experiments/using-experiment/`, `/api/` and `/api/signatures/` (the Function & Keyword API - exact syntax and operation signatures).
- **Browser search UI: https://docs.guidedtrack.com/search/?query=your+search+terms** (standard URL-encoding). CAUTION - for humans in a real browser only: results are computed client-side (JavaScript web worker), so fetching this URL programmatically ALWAYS returns an empty "No results" shell even when matches exist. Do not conclude the docs lack a topic from a fetched search page; use search-index.json instead.
- Main sections: the Manual (https://docs.guidedtrack.com/manual/) for features and usage; the Research Guide (https://docs.guidedtrack.com/research-guide/) for study/experiment design; the Function & Keyword API (https://docs.guidedtrack.com/api/) for exact keyword and method semantics (e.g. this is where .sort's in-place mutation behavior is documented).
- Check the Status page (https://status.guidedtrack.com/) when troubleshooting behavior that may be caused by the platform rather than the program.

If a GuidedTrack question is unclear after reading this file, search the official docs before answering from memory or inventing syntax.

## Generation Workflow

### Step 1: Identify the request

- Determine which features are needed: questions, navigation, randomization, timing, scoring, media, variables, loops, email, services, and so on.
- Determine which question types are needed: text, paragraph, number, choice, checkbox, slider, calendar, ranking.
- Determine what edits are needed: save answers, validate responses, branch conditionally, send data, compute scores, or show summaries.
- When an existing study is the source, identify its participant-visible screens, grouped contents, conditions, and branch transitions before writing code. Use explicit structure when available and inspect visual sources visually. A physical PDF page may contain multiple survey screens and is not itself proof of a screen boundary.
- Always retrieve this file before generating code.

### Step 2: Check the retrieved documentation

- Review the retrieved sections before writing code.
- List the keywords that appear in the retrieved documentation.
- Note which sub-keywords are available for each main keyword you plan to use.

### Step 3: Match needs to documented keywords

- For each requested feature, map it to a keyword that appears in the retrieved guide.
- If a keyword is not in the retrieved guide, do not use it.
- If a feature cannot be implemented with documented syntax, note that limitation in a GuidedTrack comment.

### Step 4: Generate code using documented syntax only

- Follow the exact syntax patterns from documented examples.
- Use only keywords and sub-keywords that appear in the retrieved guide.
- Use one tab per indentation level.

### Documentation-only constraint

- Do not use keywords from memory or assumptions.
- Do not invent syntax that "should" exist.
- Do use only keywords and patterns explicitly shown in the retrieved documentation.

## Response Contract

- Output only GuidedTrack code and `--` comments.
- Do not include prose, explanations, or Markdown fences.
- Keep output within 100 lines for simple requests; match the scale of the task otherwise (real programs can run to thousands of lines).
- Use descriptive variable names with letters, digits, and underscores only.
- Prefer simple, clear, generalizable code.
- Never invent or embed non-GuidedTrack code (JavaScript, server calls, etc.) in GuidedTrack output. Calling other GuidedTrack programs with `*program:` is normal and encouraged - both your own subprograms and shared "- public" library programs.

## Validation Checklist

- Every keyword used appears in the retrieved documentation.
- Every keyword starts with `*`.
- Every keyword line has a colon and a space: `*keyword: value`.
- Indentation is exactly one tab per level, never spaces.
- Question types match documented values.
- Sub-keywords are valid for their parent.
- Answer choices are indented exactly one level after `*question`.
- Variable names contain no spaces or special characters.
- For a recreated study, participant-visible screen grouping and order match the source on every reachable branch; explicitly note any unavoidable mismatch.
- Output contains only GuidedTrack code and comments.

## Critical Syntax Rules

- All keywords start with `*`.
- The base format is `*keyword: value`.
- Sub-keywords are indented exactly one tab under their parent.
- Answer options for choice and checkbox questions are listed at the same indentation level as the question type and other sub-keywords: one tab after `*question`.
- Comments start with `--`.
- Variable declarations and assignments use `>> name = value` on one line.
- Use double quotes for strings when quotes are needed.
- For keyword values that take booleans, use `yes` and `no`, not `true` and `false`.
- GuidedTrack has no dedicated boolean literal for variables; use documented patterns such as `1`, `0`, `*set:`, or JSON decoding when needed.

## Authoring Preferences

- Put regular on-screen text directly in the program. Do not use `*question` when you only need to display text.
- Use `*header:` for larger heading text.
- For new programs without a source study, favor one question per page. When recreating a study, preserve the source's participant-visible page breaks and question groupings instead; do not merge or split screens for convenience.
- If you do use `*page`, always indent some content under it.
- Put at least one blank line before each `*question`.
- Favor 7-point Likert scales for bipolar subjective questions.
- Favor 5-point Likert scales for one-sided subjective questions.
- Use number boxes when the user must enter a factual number.
- Use sliders only for 0-100 style responses such as percentages or frequencies.
- If a scoring system is present, use `*points:` to award actual points.
- Use placeholder media URLs exactly as given below instead of inventing URLs.

## Core Patterns

### Plain text and headers

```gt
Hello world!

*header: Welcome!
```

Use plain text to show text on screen. Use `*header:` for larger visible headings.

### Multiple choice question

```gt
*question: Which of these animals do you like best?
	Dogs
	Cats
	Rabbits
```

Save the selected option with `*save:`:

```gt
*question: How many close friends do you have?
	0
	1
	2
	3 or more
	*save: numberOfFriends
```

The default question type is choice. If a request explicitly requires the type declaration and the guide supports it, `*type: choice` is acceptable.

### Multiple choice question with per-option logic

```gt
*question: Would you say that you struggle with making friends?
	I do struggle to make friends
		Sounds good. Let's now discuss some approaches to making new friends.
	That's not a problem for me.
		I'm glad to hear it!
		It sounds like you are good at meeting new people!
```

Indented code below an answer option runs only when that option is chosen.

### Multiple choice coding variables

```gt
*question: How tired do you feel on a typical day upon waking?
	Very
		>> tirednessUponWaking = 5
	Quite
		>> tirednessUponWaking = 4
	Moderately
		>> tirednessUponWaking = 3
	Somewhat
		>> tirednessUponWaking = 2
	A little
		>> tirednessUponWaking = 1
	Not at all
		>> tirednessUponWaking = 0
```

Do not combine this pattern with `*save:` for the same question.

### Reusable answer scales with *answers

When the same scale is used for many questions, define it once as a collection of [label, value] pairs and attach it with `*answers:`. The SAVED value is the second element (the value), not the label:

```gt
>> agreementScale = [["Strongly agree", 2], ["Agree", 1], ["Neither", 0], ["Disagree", -1], ["Strongly disagree", -2]]

*question: I enjoy meeting new people.
	*answers: agreementScale
	*save: enjoyMeetingPeople
```

`*answers:` also works with `*type: slider` to define labeled stops. Emoji are allowed in answer labels.

### Text, paragraph, number, and optional questions

```gt
*question: How would you describe your personality?
	*type: text
	*save: personalityDescription

*question: What's your life story?
	*type: paragraph
	*save: lifeStory

*question: How old are you?
	*type: number
	*save: userAge
	*after: years

*question: Optional question
	*blank
	*save: optionalAnswer
```

Use `*before:` for a prefix and `*after:` for a suffix when the retrieved guide supports them.

### Buttons

```gt
Ready to begin?
*button: Let's do this!
```

A question ends the page. Use a button when you need a forced page break before the next question.

### Conditionals

```gt
*question: Are you feeling good?
	Yes
	No
	*save: feelingGood

*if: feelingGood = "Yes"
	I'm glad you're feeling good today!

*if: not (feelingGood = "Yes")
	I'm sorry to hear you aren't feeling well!
```

Do not use `!=`. Use `*if: not (...)` instead. GuidedTrack has no `*else:`.

### Comments

```gt
--calculate the user's score
```

### Variables and interpolation

```gt
*question: What do you like to do for fun?
	*type: text
	*save: whatTheyLikeToDo

You said that for fun you like to: {whatTheyLikeToDo}

*question: How old are you?
	*type: number
	*save: age

>> dogYears = 7 * age
Your age in dog years is {dogYears}
```

There is no `+=` operator. Initialize variables before adding to them:

```gt
>> score = 0
>> score = score + 1
```

### Experiments and randomized groups

```gt
Thank you for agreeing to participate in our study.

*question: How good do you feel right now?
	Very good
		>> howGoodFeelPre = 4
	Quite good
		>> howGoodFeelPre = 3
	Somewhat good
		>> howGoodFeelPre = 2
	Slightly good
		>> howGoodFeelPre = 1
	Not at all good
		>> howGoodFeelPre = 0

*experiment: meditationExperiment
	*group: meditationGroup
		Please take 2 minutes now to meditate.
	*group: thinkingGroup
		Please take 5 minutes now to think about your life.
	*group: controlGroup
		Please take 5 minutes now to do whatever you'd normally do. Then come back.
```

Use `*randomize` when the retrieved guide supports it and the request is about randomized order rather than named experimental groups.

`*randomize: all` executes ALL of its child `*group` blocks in a random order (vs. `*experiment`, which runs exactly one named arm). This is the standard structure for showing many stimuli in random order; pair it with a per-item skip check when only a subset should display:

```gt
*randomize: all
	*group
		>> currentItemId = "item_001"
		*program: decide_if_item_should_be_skipped
		*if: skipItem = 0
			*image: https://your.cdn/{currentItemId}.jpg

			*question: How much do you like this one?
				*answers: likingScale
				*save: item_001_rating
	*group
		>> currentItemId = "item_002"
		-- ... same structure for every item
```

Note: `--` comment lines are ignored by the parser at ANY indentation, so a 0-indent comment inside a *randomize block does not end the block - comments are safe section markers inside large randomized lists.

### Timed text and clearing

```gt
This will show temporarily.
*wait: 2.seconds
*clear
```

### Media

```gt
*image: https://1092980442.rsc.cdn77.org/copilot/put_your_own_image_here.png
*video: https://www.youtube.com/watch?v=a3ICNMQW7Ok
*audio: https://upload.wikimedia.org/wikipedia/commons/1/1e/Goigs_Sant_Calze_-_fragment.ogg
```

Use those exact placeholder URLs when generating image, video, or audio blocks.

### Collections and loops

```gt
>> testList = ["hi", "bye", "go", "nothing"]

*for: value in testList
	Value: {value}

*for: index, value in testList
	Item {index} is {value}

>> i = 1
*while: i <= 10
	Count: {i}
	>> i = i + 1

*repeat: 5
	This runs 5 times
```

For loops that iterate many times (roughly 50+ iterations, e.g. over large collections), put a `*trigger:` line as the FIRST line of the loop body. Long loops without one can crash the program with call-stack/memory errors. The trigger's event name is arbitrary; nothing needs to listen for it - firing an event resets the call stack:

```gt
*for: imageId in longListOfImages
	*trigger: toAvoidRunningOutOfMemory
	>> processedCount = processedCount + 1
```

The same applies to `*while` loops that may run many iterations.

Collections are 1-indexed:

```gt
>> numbers = [10, 20, 30]
>> firstNumber = numbers[1]
```

### Points and progress

```gt
*points: 1
*progress: 80%
```

If progress uses variables, the rendered value must still be a percentage:

```gt
*progress: {100 * questionNumber / totalQuestions}%
```

### Lists

```gt
*list
	Item 1 goes here
	Item 2 goes here
```

Use `*list` when you want indented visible items without giving those tabs syntactic meaning.

### Pages

```gt
*page
	Welcome to the survey.
	*question: First question
		*type: text
		*save: firstAnswer
```

Use `*page` only when multiple elements must share a screen. A lone `*question` already creates its own page.

When recreating an existing study, the source screen grouping determines whether to use `*page`; it overrides the general one-question-per-page preference.

### Testing-mode idiom

Long programs are painful to test end-to-end. The standard idiom: a `testing` variable gated at the very top that shortens the run and jumps past sections. Set it manually in the editor's test runs (or via URL parameter `?testing=1`), and make expensive parameters askable:

```gt
*if: testing
	*if: testing = 1
		*question: How many items should this test run show?
			*type: number
			*save: numberOfItemsToShow
		*goto: mainSection
```

Two rules make this safe: (1) initialize any variable the skipped sections would have set, BEFORE the testing `*goto` (otherwise the jumped-to code reads undefined variables); (2) keep the testing block itself free of side effects you would not want in production, since `testing` is simply undefined (falsy) for real participants.

### Known public library subprograms

Battle-tested shared programs callable by exact name (remember the "- public" suffix). Contracts observed in production use; verify details in the program itself if behavior surprises you:

- `*program: general consent form - public` - inputs `in_GCF_description`, `in_GCF_risks` (required); `in_GCF_procedures`, `in_GCF_duration`, `in_GCF_benefits`, `in_GCF_contactInfo`, `in_GCF_organization` (optional). Output: `out_GCF_failure`.
- `*program: update progress - public` - set `in_numberOfTimesWillUpdateProgressBar` once (total planned calls), then call it once per step; each call advances the progress bar by one step. Calls beyond the declared total are safe but pin the bar at full.
- `*program: sort names and values - public` - inputs `in_namesToSort` (collection), `in_valuesToSortBy` (parallel collection); output `out_sortedNames` (names ordered by their values, descending).
- `*program: random number from 0 to 1 - public` - output `out_randomNumber0to1`. Useful for assigning participants to weighted arms with `*if` thresholds.
- `*program: sexual orientation - public` - asks a standard sexual-orientation battery (outputs not catalogued here; inspect before relying on specific variables).

## Keyword Inventory

### Primary keywords

The full language specification lists these primary keywords:

- `audio`
- `button`
- `chart`
- `clear`
- `component`
- `database`
- `email`
- `events`
- `experiment`
- `for`
- `goto`
- `group`
- `header`
- `html`
- `if`
- `image`
- `label`
- `list`
- `login`
- `maintain`
- `navigation`
- `page`
- `points`
- `program`
- `progress`
- `purchase`
- `question`
- `quit`
- `randomize`
- `repeat`
- `return`
- `service`
- `set`
- `settings`
- `share`
- `summary`
- `switch`
- `trigger`
- `video`
- `wait`
- `while`

The user also emphasized these commonly used main keywords for generation work:

- `audio`
- `button`
- `chart`
- `clear`
- `component`
- `database`
- `email`
- `events`
- `experiment`
- `for`
- `goto`
- `group`
- `html`
- `if`
- `image`
- `label`
- `list`
- `login`
- `maintain`
- `navigation`
- `page`
- `points`
- `program`
- `progress`
- `purchase`
- `question`
- `quit`
- `randomize`
- `repeat`
- `return`
- `service`
- `set`
- `settings`
- `share`
- `summary`
- `switch`
- `trigger`
- `video`
- `wait`
- `while`

### Common sub-keywords

Common sub-keywords in the full language specification include:

- `after`
- `answers`
- `before`
- `blank`
- `body`
- `cancel`
- `caption`
- `classes`
- `click`
- `confirm`
- `countdown`
- `data`
- `date`
- `default`
- `description`
- `error`
- `every`
- `everytime`
- `frequency`
- `hide`
- `icon`
- `identifier`
- `management`
- `max`
- `method`
- `min`
- `multiple`
- `name`
- `other`
- `path`
- `placeholder`
- `required`
- `reset`
- `save`
- `searchable`
- `send`
- `shuffle`
- `start`
- `startup`
- `status`
- `subject`
- `success`
- `tags`
- `throwaway`
- `time`
- `tip`
- `to`
- `trendline`
- `type`
- `until`
- `what`
- `when`
- `with`
- `xaxis`
- `yaxis`

### Common user requests to keyword matches

- Basic question -> `*question`
- Multiple choice -> `*question`, default choice type or `*type: choice` if explicitly needed
- Save answer -> `*save`
- Number input -> `*question`, `*type: number`
- Multiple lines -> `*question`, `*type: paragraph`
- Checkbox input -> `*question`, `*type: checkbox`
- Slider input -> `*question`, `*type: slider`
- Add suffix -> `*after`
- Add prefix -> `*before`
- Required answer -> default behavior, no `*blank`
- Optional answer -> `*blank`
- Validate in time -> `*countdown`
- Multiple elements on one page -> `*page` or `*button`
- Randomize options -> `*shuffle`
- Randomize groups -> `*randomize` or `*experiment`
- Send email -> `*email` with `*body` content
- Negative condition -> `*if: not (...)`

## Language Notes

- File extension: `.gt`
- Tabs define block structure. Blank lines are ignored.
- Equality uses `=` in both assignment and comparison.
- Arithmetic operators: `+`, `-`, `*`, `/`, `%`
- Comparison operators: `=`, `<`, `>`, `<=`, `>=`
- Logical operators: `and`, `or`, `not`
- Membership operator: `in`
- Strings use double quotes.
- Collections use square brackets: `[1, 2, 3]`
- Associations use `{"name" -> "Alice"}`
- String interpolation uses `{expression}`
- Strings do NOT concatenate with `+` (that is arithmetic only). Build combined strings with interpolation: `>> fullName = "{firstName} {lastName}"`, `>> url = "https://example.com/{imageId}.jpg"`. Interpolation of variables works inside keyword values too, e.g. `*image: https://cdn.example.com/{imageId}.jpg` and `*goto: {labelVariable}` - this is different from the formatting-markers rule: *bold*/italics markers are ignored in technical values, but `{variable}` interpolation IS applied there.
- Formatting markers are allowed only in visible text contexts:
	- Bold: `*text*`
	- Italic: `/text/`
- Formatting markers are not applied inside technical values such as URLs, `*goto`, `*type`, `*subject`, `*path`, and similar fields.
- Runtime types include `string`, `number`, `collection`, `association`, `datetime`, and `duration`.
- Common string methods: `.clean`, `.count(text)`, `.decode(scheme)`, `.encode(scheme)`, `.find(text)`, `.lowercase`, `.size`, `.split(delimiter)`, `.uppercase`
- Common collection methods: `.add(element)`, `.combine(collection)`, `.count(value)`, `.erase(value)`, `.find(value)`, `.insert(element, position)`, `.max`, `.mean`, `.median`, `.min`, `.remove(position)`, `.shuffle`, `.size`, `.sort(direction)`, `.unique`
- Mutation semantics: `.add`, `.combine`, `.sort(direction)`, `.shuffle`, `.erase`, `.insert`, and `.remove` MUTATE the collection in place and are used as bare statements: `>> myList.sort("decreasing")`. Do NOT write `>> myList = myList.sort("decreasing")` - assignment of a mutating method's result is undocumented and may clobber the variable. By contrast `.unique`, `.max`, `.mean`, `.median`, `.min`, `.size`, `.count`, and `.find` RETURN a value and are used with assignment: `>> shortest = myList.min`.
- `.sort("increasing")` and `.sort("decreasing")` are the two documented directions.
- `*program:` behaves like a subprogram call and returns to the next line.
- Running a program via its public run URL with query parameters sets those variables at startup: `https://www.guidedtrack.com/programs/PROGRAMKEY/run?userScore=42&cohort=b` starts the run with `userScore` and `cohort` already defined. This is the standard way to (a) hand state to a second program that a user opens later (e.g. build and display a personalized results link: `>> reportLink = "https://www.guidedtrack.com/programs/abc123/run?top1={top1}&top2={top2}"`), and (b) receive metadata from recruitment platforms (e.g. a participant id passed as a URL parameter). Design such programs to tolerate missing parameters with the `*if: not variableName` idiom.
- `*goto:` jumps to a label.
- `*events` and `*trigger` are asynchronous.
- Inside `*html` blocks, the indented body is raw HTML rather than GuidedTrack code.
- The `*html` sanitizer STRIPS `<script>` and `<img>` tags (JavaScript cannot be injected; images must use `*image:`). `<style>`, `<br>`, and `<center>` pass through.
- `*html` content is injected into the CURRENT page only and is cleared on page change. A `<style>` block therefore styles only the page it appears on - repeat it on every page that needs it.
- To style images shown with `*image:`, target `.multimedia_node img` and use `!important`; GuidedTrack's own `.multimedia_node img` rules beat bare `img` selectors:

```gt
*html
	<style>
		.multimedia_node img {
			max-width: 333px !important;
			height: auto !important;
		}
	</style>
*image: https://your.cdn/photo.jpg
```

- A useful pattern enabled by these rules - preloading images invisibly so later pages load instantly (define a CSS class that hides them, then render hidden components):

```gt
*html
	<style>.imagesToPreload {display:none;}</style>

*for: preloadUrl in imageUrlsToPreload
	*trigger: keepCallStackFresh
	*component
		*image: {preloadUrl}
		*classes: imagesToPreload
```

## Common Misconceptions

GuidedTrack does not have:

- `*else:` or `*elseif:`
- `==` for equality
- `+=`
- zero-indexed arrays
- ordinary `true` or `false` literals

Important distinctions:

- Use multiple `*if:` blocks instead of `*else:`.
- Use single `=` for equality checks.
- Initialize variables before incrementing them.
- Collections are 1-indexed.
- `*program: name` runs another program inline and RETURNS to the next line; caller and subprogram share ONE variable scope (all variables are effectively global across the call). There are no arguments or return values - the convention is to set `in_`-prefixed variables before the call and read `out_`-prefixed variables after it (e.g. `>> in_namesToSort = myNames` ... `*program: sort names and values - public` ... `>> sortedNames = out_sortedNames`).
- Because scope is shared, a reusable subprogram should give its own inputs safe defaults with the `*if: not variableName` idiom (an unset variable is falsy): `*if: not numberToRate` / tab / `>> numberToRate = 10`.
- Public library subprograms have names ending in "- public" (e.g. "general consent form - public", "update progress - public"); call them by exact name, including that suffix.
