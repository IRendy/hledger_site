# Start tracking lots

hledger 2 can track and check your investment lots.
hledger then works out each lot's cost basis, and picks the lots to sell using your chosen method (FIFO, LIFO, ...),
calculates realised gains (and checks any you have written),
reports holdings with unrealised and realised gains,
and catches mistakes such as selling more than you hold.
Compared to tracking lots by hand with subaccounts, this is much less work once set up, and more reliable.

This page has practical advice for turning on lot tracking in an existing journal.
(If you're moving from hledger 1, see also [Upgrading to hledger 2](upgrading.md#turning-on-lot-tracking).)

## Getting started

Usually you start by adding a `lots` tag to each investment commodity's declaration:
```journal
commodity AAPL  ; lots:
```
Then purchases of that commodity create lots, and sales take from them.
[Lots and capital gains](hledger.md#lots-and-capital-gains) in the manual explains how to write lot entries.

## If you have a long investment history

If you haven't tracked lots before, be aware that recording that full history as lots can be a big job.
In lot accounting, later entries tend to depend on all previous entries;
so every purchase, sale, transfer, split and fee must have the right date, quantity and cost,
or hledger will report lot errors.
(If you have been tracking lots by hand, your entries already have most of the details needed;
see [Upgrading to hledger 2](upgrading.md#if-youve-been-tracking-lots-by-hand).)
But fear not! Here are some ways to divide and conquer:

- **Draft in a separate file.** Copy your investment entries into a new file, eg `investments.journal`,
  add the `lots` tag there, and work on it with `hledger -f investments.journal check`
  and `hledger -f investments.journal holdings --lots`, while your main journal keeps working as before.
  When it's right, replace the original entries with the new ones (or with an [include](hledger.md#include-directive)).
- **Start from a recent date.** Rather than reconstructing your whole history,
  start at the beginning of this year, with an opening entry that acquires the lots you held then,
  with their original acquisition dates and costs (your broker's cost basis records can provide these):
  ```journal
  2026-01-01 opening lots
      assets:stocks      10 AAPL {2024-03-15, $50}
      assets:stocks       5 AAPL {2025-06-02, $62}
      equity:opening/closing balances
  ```
  If you keep a file per year, these replace the stock postings in this year's opening balances entry.
  If you keep everything in one file, there's no such entry to replace,
  so either draft the full history, or first split off this year's entries with [close](hledger.md#close).
- **Do one commodity or account at a time.** Add the `lots` tag to one commodity,
  and get it checking cleanly before moving to the next.
  Accounts where cost basis doesn't matter, like tax-sheltered retirement accounts,
  can be left out with a `lots: NONE` tag on the account declaration.
- **Compare with your broker.** `hledger holdings --lots` shows the lots hledger has worked out;
  check them against your broker's records, and add balance assertions to catch drift.
  Their figures may be rounded differently in different places;
  see [Reconciling](hledger.md#reconciling) for tips.
- **Troubleshoot.** See [Troubleshooting lots](hledger.md#troubleshooting-lots) in the manual.
  `hledger print -a` is the most useful tool, showing how hledger has read each entry.
- **Keep other reports working.** While lot records are incomplete, you can skip lot processing with `--ignore-lots`.
