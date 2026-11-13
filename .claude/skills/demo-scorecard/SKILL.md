---
name: demo-scorecard
description: Turn a sales demo transcript into a KPI scorecard dashboard showing good and bad points, a 9-area score, buyer question handling, feature coverage and prioritised improvements. Use when the user supplies a demo or sales call transcript (Fathom, Gong, Teams, Zoom or similar) and wants it reviewed, scored or turned into a dashboard, or types /demo-scorecard.
---

# Demo Scorecard

Turn one demo transcript into a published dashboard. The layout is fixed in `template.html` so every demo looks the same and can be compared. Only the data and a few lines of text change.

Files in this folder:
- `template.html` - the dashboard page. Placeholders look like `{{NAME}}`.
- `example-assessment.json` - a finished example (K&S GL Connect demo, 6 November). Match this shape exactly.

House style for everything you write (data, page text, reply): plain UK business English, no em dashes (use a hyphen), no filler praise. Tag key claims in the reply with [Certain], [Likely] or [Guessing].

If no transcript is in the message, ask for one and stop.

## 1. Extract the data

Build `assessment.json` with these keys, matching the example file's field names and types:

- `meeting`: `{title, date, duration_s}`. Duration is the last timestamp in seconds.
- `talk`: one row per speaker `{speaker, org, side, role, seconds}`. `side` is exactly `"Seller"` or `"Buyer"`. Seconds = sum of the time from each of their timestamps to the next one. Fix lines that are clearly given to the wrong speaker (for example, an answer listed under the person who asked) and note which ones in METHOD_NOTE. Work it out with a script, not by hand.
- `phases`: 6 to 12 rows `{start, end, start_s, end_s, type, label}`. `type` is exactly `Demo`, `Q&A`, `Discovery` or `Close`. Phases cover the whole call with no gaps.
- `questions`: every real buyer question (not "does that make sense?" or other confirmations). `{time, seconds, asked_by, question, answer, quality, note}`. `time` is unique (add `:01` style seconds if two collide). `answer` is exactly one of `Shown live`, `Verbal - clear`, `Verbal - unclear`, `Roadmap only`, `Failed live`, `Not answered`. `quality` is `Strong`, `Adequate` or `Weak`.
- `moments`: 8 to 12 good and 8 to 12 bad `{id, time, seconds, type, title, detail}`. `type` is `Good` or `Bad`. Ids are `g1..`, `b1..`, except the moment the buyer's main reason for buying first came out, which has id `driver` (normally type Bad if it came out late, Good if in the first quarter of the call). There must be exactly one `driver`.
- `features`: `{feature, status}`, status `Shown live`, `Failed live` or `Told, not shown`.
- `scores`: exactly these 9 areas, in this order, `{dimension, score, confidence, evidence}`: Preparation and agenda; Discovery; Fit to the audience; Demo execution; Handling questions; Differentiation; Team coordination; Close and next steps; Product capability shown. `score` is a whole number 1 to 5 (3 = acceptable). `confidence` is `Certain`, `Likely` or `Guessing`. Evidence is one sentence that cites timestamps.
- `improvements`: 6 to 10 rows `{priority, area, happened, instead}` in priority order (1 = would have changed the outcome most). `instead` must be specific enough to act on in the next call.
- `actions`: every agreed next step `{action, owner, due, status}`. `due` is `null` when no date was agreed. Status is `Open`, `Done` or `Declined for now`.

Scoring rules (apply them, don't soften them):
- No agenda or discovery before the screen share caps Preparation and agenda at 2.
- Each live failure lowers Demo execution by 1 from a starting 4.
- Sellers contradicting each other caps Team coordination at 3.
- No dated next step caps Close and next steps at 3.
- Check facts the seller stated (limits, release dates, figures). Record any that were wrong or guessed as a Bad moment.
- Count apologies and hedges from the seller. If 5 or more, include an improvement for it.

Never invent content that isn't in the transcript. If something can't be told from the transcript, say so in the evidence and tag it Guessing.

## 2. Fill the template

Copy `template.html` and replace every placeholder. Keep the text short, plain and specific to this call:

| Placeholder | What goes in |
|---|---|
| `{{CUSTOMER}}` | Buyer company name. Escape `&` as `&amp;`. |
| `{{PRODUCT}}` | Product demonstrated. |
| `{{SELLER_ORG}}` | Selling company name. |
| `{{DATE}}` | Call date, e.g. `6 November`. |
| `{{DURATION_MIN}}` | Length in whole minutes. |
| `{{LEDE_OPENER}}` | One short sentence of overall verdict, e.g. "The product landed better than the demo." |
| `{{FAILURES_NOTE}}` | What failed live, or "No live failures." |
| `{{DRIVER_NOTE}}` | The buying driver in one short sentence. |
| `{{ACTIONS_NOTE}}` | Decision timeline if stated, else "No decision timeline was agreed." |
| `{{TIMELINE_INSIGHT}}` | One sentence on how the time was spent. |
| `{{COVERAGE_INSIGHT}}` | One sentence on what was told but not shown. |
| `{{RECOMMENDED_FOLLOWUP}}` | The 3 or 4 follow-ups to do now, as one sentence. |
| `{{METHOD_NOTE}}` | Source, how talk time was estimated, which speaker lines were reassigned, that scores are a reviewer's judgement, and what can't be seen in a transcript. |

Afterwards check that no `{{` is left, there are no em dashes in the page or the JSON, and the script passes `node --check`.

## 3. Publish

1. `Artifact` with `action: "quickstart"`, `intent: "other"`, then create a new dashboard from the Data type it names, with title `{{PRODUCT}} Demo Scorecard - {{CUSTOMER}}` and `auto_open: "after_first_write"`. Follow the instructions that call returns.
2. Upload `assessment.json` to that artifact with `asset: true` and note the returned `/_blob/<id>` url.
3. With `ArtifactData` `batch`, write:
   - `dash/meta`: `{title, lang: "en-GB"}`
   - `datasets/<id>` for each of `talk`, `phases`, `scores`, `questions`, `moments`, `features`, `improvements`, `actions`: `{title, description, source: {kind: "file", url: <blob url>, name: "assessment.json", format: "json"}, updated: {at, by: "claude"}, fixed: true, edit: {via: "claude", at}}`. End each description with "Reviewer assessment of the <date> transcript."
   - `files/index.html`: `{text: <filled template>, edit}`
4. Read `data/users/me` for `check` and `runs`. Fix any fault they report. If nothing has judged the page yet, say so in the reply.

## 4. Reply

Give the link, then:
- headline figures with [Certain]/[Likely]/[Guessing] tags: overall score, seller talk share, questions proven live, live failures, when the driver surfaced, dated next steps
- the top 3 good points and top 3 bad points
- the top 5 improvements
- anything in the transcript that was unclear or that you had to guess
- that the page is private until shared from its Share menu, and that anyone it is shared with can read the attached assessment file
