# AI usage accounting

Here are some notes on tracking AI usage, currently just from the hledger project.

## hledger project

The project's [AI policy](AI.md) requires tracking our AI usage.

`just ai-help` in the hledger repo shows some related Justfile scripts.

### Measuring usage

In 2026 Q1, I (SM) started exploring AI assistance for the hledger project,
beginning with the initial lots work for the first 2.0 preview release (1.99.1).
I used Anthropic's API pricing, and recorded token counts and costs
from <https://platform.claude.com/usage>.

In Q2 I received and started using a 6-month Anthropic FOSS credit for a Claude Max plan (worth about $1200).
I tracked my usage with [ccusage](https://ccusage.com), which collects data from the claude code chat logs
on your machine (for both API and monthly plan usage).
Note it sees only nonexpired chat logs, and by default they expire after 30 days.
I tried to gather data regularly, while my logs were intact.
Later I saw you can configure claude code to retain logs longer, eg for a year or more.

(I compared reports from ccusage with the platform usage, and they did not agree very well; I'm not sure why.)

From the start of Q3, I and a few other developers using AI haved logged our AI usage (measured with ccusage, eg) in git commits,
adding a standard "AI usage: ..." line.

Each month I summarise and import my and others' usage estimates from the git commits,
to [doc/ai/ai.journal](https://github.com/hledgerorg/hledger/blob/main/doc/ai/ai.journal)
When there's significant usage not logged in a commit, I add that manually to ai.journal also.

See [doc/ai/commodities.journal](https://github.com/hledgerorg/hledger/blob/main/doc/ai/commodities.journal)
for some notes on estimating and conversion to other units.

### Reports

Here's my estimate/recollection of my Q1 usage, as reported by platform.claude.com.
Note these numbers include both input and output tokens; later I switched to counting output tokens only.

- 2026-01: 133 Mt,   $85
- 2026-02: 598 Mt,  $551
- 2026-03: 299 Mt,  $256
- Total:    ~1 Gt, ~$900

In this time I spent about 150h working with AI (estimated market value ~$10k-30k).

Here's a more complete project AI usage report for 2026, as of 2026-10-11: <https://groups.google.com/g/hledger/c/q8IiWpbljmM>

