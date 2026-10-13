# AI usage tracking

Here are some notes on tracking AI usage, currently just from the hledger project.

## hledger project

The project's [AI policy](AI.md) requires tracking our AI usage.

`just ai-help` in the hledger repo shows some related Justfile scripts.

### Measuring usage

In the hledger project we have mostly used claude. Here are some ways to measure claude usage:

- If you use API pricing, count monthly output tokens from the Analytics > Usage report at
  <https://platform.claude.com/usage>.

- If you use a monthly plan, report and log monthly usage with [ccusage](https://ccusage.com),
  which collects data from the claude code chat logs on your machine (this works for API usage also).
  Report and log regularly, because it sees only nonexpired chat logs, and by default they expire after 30 days.
  You can configure claude code to retain logs longer, eg for a year or more.

### Logging usage

From the start of Q3, we log our AI usage (measured with ccusage, eg) in every AI-assisted git commit,
with an "AI usage: ..." line in a standard format.

Each month I summarise the usage estimates from commit messages to `doc/ai/aicommits.csv`,
using `just ai-commits-csv` and `tools/aicommits`.
And I import a new entry from there to a permanent hledger journal,
[doc/ai/ai.journal](https://github.com/hledgerorg/hledger/blob/main/doc/ai/ai.journal)

Whenever there's significant usage that didn't get logged in a commit,
we add that manually to ai.journal also.

### Reports

WIP. Here's an project AI usage report as of 2026-10-11: <https://groups.google.com/g/hledger/c/q8IiWpbljmM>

