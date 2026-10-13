# AI usage accounting

Notes on AI usage tracking, currently just from the hledger project.

## hledger

[AI.md](AI.md) describes the project's AI policy.

`just ai-help` shows some related Justfile scripts.

### Measuring AI usage

In 2026 Q1, for the initial lots work in the first 2.0 preview release (1.99.1),
I (SM) started using AI assistance. (The first known use of AI in the hledger project.)

I used Anthropic's API pricing, and recorded token counts and costs
from <https://platform.claude.com/usage>. Here's my estimate/recollection of those numbers.
Note these include both input and output tokens; later data counts output tokens only.

- 2026-01: 133 Mt,   $85
- 2026-02: 598 Mt,  $551
- 2026-03: 299 Mt,  $256
- Total:    ~1 Gt, ~$900

In this time I spent about 150h working with AI (estimated market value ~$10k-30k).

In Q2 I received and started using a 6-month Anthropic FOSS credit for a Claude Max plan (worth about $1200).
I tracked my usage with [ccusage](https://ccusage.com).

ccusage shows data from the claude code chat logs on your machine (for both API and monthly plan usage).
Note it sees only the chat logs still present on the machine, and by default these expire after 30 days.
I tried to gather data regularly, while my logs were intact, but for me it still showed numbers
much lower than the ones from platform.claude.com above, I'm not sure why.
Later I saw you can configure claude code to retain logs longer, eg for a year or more.

From the start of Q3, I and a few other developers using AI haved logged our AI usage (measured with ccusage, eg) in git commits,
adding a standard "AI usage: ..." line.

Each month I summarise and import my and others' usage estimates from the git commits,
to [doc/ai/ai.journal](https://github.com/hledgerorg/hledger/blob/main/doc/ai/ai.journal)
When there's significant usage not logged in a commit, I add that manually to ai.journal also.

## Reports

WIP.
Here's a project AI usage report from 2026-10-11: <https://groups.google.com/g/hledger/c/q8IiWpbljmM>
