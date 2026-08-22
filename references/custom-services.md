# Custom Services (server-side backend)

A **custom service** is server-side JavaScript plus a database, hosted inside GuidedTrack. It is how a program gets state that outlives a single run: quotas, cross-run averages, logins, role-based permissions, or piping one participant's answers between surveys.

A custom service is a collection of **routes**. Each route is an HTTP method plus a path (`GET /greeting`, `POST /registrations`) and holds a JavaScript handler. Routes read and write **tables** through the `guidedtrack-db` library, which is preloaded in every route.

**Custom services are one of the two things `*service:` can call, and the rarer one.** `*service:` is a long-standing core keyword whose usual job is calling a third-party HTTP API; custom services are a newer addition that lets you be the API. The keyword, its sub-keywords, and how a service gets registered are documented once for both cases in [complete_guide.md](complete_guide.md), under "Services and HTTP requests" — read that first. This file covers only what is specific to custom services: building the routes, and the database behind them.

Read it when a task needs data shared across runs or participants.

Source: [Custom Services](https://docs.guidedtrack.com/manual/advanced-options/custom-services/) and the [`guidedtrack` library API](https://docs.guidedtrack.com/api/#the-guidedtrack-library-within-custom-services). Every claim below is cited to those pages or to the two use-case walkthroughs linked at the end; nothing here is inferred.

## What the agent can and cannot do

**Custom services are configured in the browser, not in `.gt` files.** Creating a service, adding routes, editing route JavaScript, creating tables, and connecting a service to a program all happen on guidedtrack.com. There is no `gt` CLI subcommand and no documented API for any of it.

So an agent's job is usually: write the route JavaScript and the `*service:` block, then give the user click-by-click instructions for pasting them in. Do not tell the user a service is "set up" after only writing code.

## Setup, in the order it must happen

1. **Create the service** — home page → **Custom Services** tab → **New custom service** → name it. The name is arbitrary, like a program name. ([Step 1](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#step-1-set-up-the-custom-service))
2. **Create a table** (if the service stores data) — the service's **Tables** page → new table. When creating one, GuidedTrack offers to generate common routes; checking **"Create new records"** generates a working `POST /<table>` insert route. ([Quota example, step 3](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/implementing-quotas/#step-3-create-the-registrations-table-and-save-route))
3. **Create routes** — **New route**, choose method and path, then edit the handler code. ([Step 2](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#step-2-create-a-route))
4. **Connect the service to the program** — the *program's* Settings → **Services** tab → **Add internal service** → pick the service from the dropdown. ([Step 4](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#step-4-connect-the-custom-service-to-a-program))

> **"Add internal service", not "Add external service."** The Services tab has both buttons. External is for third-party APIs; custom services are internal. A program that has not been connected cannot resolve its `*service:` blocks — a silent-looking failure with no clue in the code.

Each program that calls the service needs its own connection. Duplicating a program does not necessarily carry it over — check.

## Calling a route from a program

The call syntax is the ordinary `*service:` block — see "Services and HTTP requests" in [complete_guide.md](complete_guide.md) for the keyword, its sub-keywords, and the result-block rules. Nothing about the call changes because the service is internal.

```gt
*service: Greeter
	*method: GET
	*path: /greeting
	*success
		>> greeting = it["message"]
	*error
		error: {it}

The greeting we received from our custom service: {greeting}
```

Three things are worth knowing when the service on the other end is your own:

- `*path:` is the route's path exactly as you defined it, query string included: `*path: /person?id={uid}`. The handler reads those with `event.queryStringParameters`.
- `*send:` keys become **table column names** when the handler passes the parsed body straight to `.insert()`, so `*send: { "name" -> participantName }` creates a `name` column. The handler reads it via `JSON.parse(event.body)`.
- `{it.text}` renders a whole collection or association, which is handy when a route returns a list of records.

Sending a nested association works, which is what makes search selectors possible from GT:

```gt
*service: Custom DB
	*method: POST
	*path: /people/search
	*send: { "last_name" -> { "$regex" -> "D.*" }}
	*success
		Found!

		{it.text}
	*error
		Error: {it}
```

`*service`, `*path`, `*method`, `*send`, `*success`, and `*error` all appear in the keyword inventory in [complete_guide.md](complete_guide.md#keyword-inventory); the syntax above is their documented usage.

## Writing a route handler

Every route is an `async` function named `handler` that takes `event`. Two hard requirements ([Step 3](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#step-3-edit-the-code-to-do-what-the-route-needs-to-do)):

1. **It must finish within 10 seconds.** The server kills longer requests. Chaining several external API calls inside one route is the usual way to hit this.
2. **It must return a response object** — `statusCode` (an HTTP status) plus `body` (the payload).

```js
export const handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: "Hi there, you're speaking to your custom service!"
    })
  }
};
```

Request data arrives in two places:

- `JSON.parse(event.body)` — the association sent by `*send:` (POST/PUT/PATCH).
- `event.queryStringParameters.id` — query-string values from `*path: /person?id={uid}` (any method).

Otherwise it is ordinary JavaScript, with no documented limits beyond the 10-second budget.

## The `guidedtrack-db` library

Preloaded into every route; the import line is generated for you. ([API reference](https://docs.guidedtrack.com/api/#the-guidedtrack-library-within-custom-services))

```js
import guidedtrack from "guidedtrack-db";
```

`guidedtrack.table(tableName)` opens a table and returns a `Table` object.

### Table methods

| Method | Purpose |
|---|---|
| `.insert(data)` | Write a record. `data` is a JSON object; its keys become table columns. |
| `.find(id)` | Load one record by its `_id`. |
| `.search(selector)` | Load every record matching a [Mango query selector](https://docs.couchdb.org/en/stable/ddocs/mango.html#selectors). |
| `.update(id, data)` | Overwrite the record's columns with `data`. |
| `.delete(id)` | Remove the record. |
| `.count()` | Number of records in the table. |

`.insert`, `.find`, and `.search` are specified in the API reference; `.update`, `.delete`, and `.count` come from documented worked examples ([updating](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#2-paste-in-the-code-to-update-a-record-in-people), [deleting](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#2-paste-in-the-code-to-delete-a-record-in-people), [counting](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/implementing-quotas/#step-5-create-a-route-to-check-registration-count)).

Every inserted record automatically gets:

- `_id` — the record's unique ID, randomly generated unless the inserted data supplies one. It is what `.find(id)`, `.update(id, …)`, and `.delete(id)` take.
- `created_at` — insertion timestamp.

([Insert walkthrough](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#3-use-the-insert-route-within-a-guidedtrack-program))

### The `Operation` object

Table methods do not hit the database immediately — they return an `Operation`, which you configure and then execute. Executing is what `await` is for.

**Executing:**

- `.response()` — wraps the result in a Lambda response: `{ statusCode: 200, body: JSON.stringify(result) }`. Use it when the route's only job is to hand the result back to the program.
- `.values()` / `.value()` — **these two are equivalent**; both return the result body as a JSON object. Use them when the handler needs the data itself before responding.

**Configuring a pending `.search()`** (these apply to search only):

- `.columns("last_name", "age")` — return just these fields.
- `.skip(n)` — skip the first n matches.
- `.limit(n)` — return at most n.
- `.sort({"age": "desc"}, {"last_name": "asc"})` — one JSON object per sort field, value `"asc"` or `"desc"`; multiple objects sort by multiple fields.

> `.sort()` here is the database's, and takes `"asc"`/`"desc"`. It is unrelated to the GuidedTrack language's `collection.sort(direction)`, which takes `"increasing"`/`"decreasing"`. Do not carry one vocabulary into the other.

Chained:

```js
const result = await guidedtrack.table("participants")
  .search({ "first_name": "John" })
  .columns("last_name", "age")
  .sort({ "age": "desc" })
  .limit(100)
  .values();
```

### Aggregating

The documented aggregate is `.latest(n).average(column)`, used to average a column over the n most recent records:

```js
const incomeAverage = await guidedtrack.table("participants").latest(250).average("income");
```

([Average-income example](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/comparing-users-results-against-others/#step-4-create-a-route-to-retrieve-the-average-income)) — `.latest` and `.average` appear in that worked example but not in the API reference's method list, so treat this exact form as the safe one and compute anything more elaborate in JavaScript over `.values()`.

## Mango selectors

`.search(selector)` takes a CouchDB Mango selector, so the selector can be built in the program and sent as an association ([search walkthrough](https://docs.guidedtrack.com/manual/advanced-options/custom-services/#3-use-the-search-route-within-a-guidedtrack-program)):

```js
{ "first_name": "John" }            // exact match
{ "last_name": { "$regex": "D.*" } } // last name starts with D
```

Full operator list: [Mango query documentation](https://docs.couchdb.org/en/stable/ddocs/mango.html#selectors).

A route that forwards `event.body` straight into `.search()` lets anyone who finds the URL query the whole table with any criteria. The docs flag this explicitly: validate that the incoming query is one your programs would actually issue.

## Worked example: enforcing a quota

Accept the first 30 registrations, then close. Condensed from [Workshop Registration with Limited Spots](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/implementing-quotas/).

**Route `GET /registrations/count`** — how many so far:

```js
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
  const count = await guidedtrack.table("registrations").count();

  return {
    statusCode: 200,
    body: count
  };
};
```

**Route `POST /registrations`** — save one, refusing if full. The check is repeated here because a participant who calls the route directly never runs the program's check:

```js
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
  const max_registrations = 30;
  const count = await guidedtrack.table("registrations").count();

  if (count >= max_registrations)
    return {
      statusCode: 403,
      body: JSON.stringify({ error: "This workshop has already reached the maximum number of enrollments!" })
    }

  const data_sent = JSON.parse(event.body);
  return await guidedtrack.table("registrations").insert(data_sent).response();
};
```

**Program** — check the count up front, then save and handle a late rejection:

```gt
>> registration_count = 0
>> max_registrations = 30

*service: Design Workshop Registration Manager
	*path: /registrations/count
	*method: GET
	*success
		>> registration_count = it
	*error
		You got an error: {it}

*if: registration_count >= max_registrations
	We're sorry, but registration for this workshop is now closed.
	*goto: end

*question: What is your name?
	*save: name

*question: What is your email?
	*save: email

>> registration_error = "N/A"

*service: Design Workshop Registration Manager
	*path: /registrations
	*method: POST
	*send: { "name" -> name, "email" -> email }
	*success
		Successfully saved your registration!
	*error
		>> registration_error = it["error"]
		You got an error: {it}

*if: not registration_error = "N/A"
	*goto: end

*label: end
```

Two patterns worth reusing: a route that returns a bare number puts it in `body` unwrapped (`body: count`), and `it` in `*success` is then that number directly. And because a route can fail *after* the participant has filled the form, capture the error into a variable and branch on it rather than assuming `*error` ended the run — it does not.

## Pitfalls

- **The program was never connected to the service.** Settings → Services → Add internal service. Nothing in the `.gt` file reveals this.
- **The service name in `*service:` does not match the dashboard name.** It is exact, spaces and case included.
- **A route silently exceeded 10 seconds.** Suspect this whenever a route that works on a small table starts failing on a large one.
- **The route returned no `statusCode`/`body` object.** Every path through the handler needs one, including early returns.
- **`*error` does not stop the program.** Set a flag inside it and check the flag afterward.
- **A route returned a JSON boolean.** `true`/`false` do not survive into GuidedTrack as booleans: a route returning `has_gem: true` left `*if: it["has_gem"]` unexecuted (verified 2026-08-22). Return `1`/`0` and test `*if: it["flag"] = 1`. The mirror-image failure is just as silent - a returned `0` PASSES a bare `*if:`, because `*if:` tests definedness rather than truth.
- **`*goto` indented under `*error` or `*success` will not compile.** GuidedTrack rejects the program: "The keyword *error cannot have *goto indented underneath it". Set a flag inside the block and branch at column 0 after the `*service` call - which is what you need anyway, since `*error` does not stop the program.
- **Two features of one route reading the same request key.** A route that both chose an "ideal" record and returned a scored list read `min_age`/`max_age` for both, so sending an age range silently shrank the scored list from 430 items to 31 while leaving it unfiltered in every other respect. Prefix keys per feature (`score_min_age` vs `min_age`).
- **Sensitive routes are public URLs.** Any quota, permission, or eligibility rule that matters must be enforced inside the route, not only in the program.

## Further reading

- [Custom Services](https://docs.guidedtrack.com/manual/advanced-options/custom-services/) — setup, routes, tables, insert/find/search/update/delete walkthroughs.
- [The `guidedtrack` library within custom services](https://docs.guidedtrack.com/api/#the-guidedtrack-library-within-custom-services) — the method reference.
- [Implementing quotas](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/implementing-quotas/) — full quota walkthrough, including securing the route.
- [Comparing a user's results against others](https://docs.guidedtrack.com/manual/advanced-options/custom-services-use-cases/comparing-users-results-against-others/) — averages and `*chart` reporting.
