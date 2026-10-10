# Upgrading to hledger 2

hledger 2 (the 1.99.x previews, and 2.0 when released) is a major update,
whose biggest new feature is [lot tracking and capital gains reporting](hledger.md#lots-and-capital-gains).
It reads hledger 1 data, and most hledger 1 users can switch to it with no changes.
This page helps you check that, and fix anything that needs it.
It covers the changes since hledger 1.52 that can affect existing setups;
the [release notes](relnotes.md) list all changes.

hledger 1 (1.52.x) continues to receive occasional fixes,
and both versions can be installed side by side.

## Upgrade steps

1. Install hledger 2 (see [Install](install.md)),
   keeping hledger 1 available under another name, eg by renaming its executable to `hledger1`.

2. Make sure your data files are backed up, or committed to version control.

3. Check your data with hledger 2:
   ```cli
   $ hledger check
   ```
   (Add any extra checks you normally use, eg `hledger check -s`.)

4. If you import CSV files, run your usual import command with `--dry-run` added,
   which shows what would be imported without changing anything. Eg:
   ```cli
   $ hledger import --dry-run *.rules
   ```

5. Compare your usual reports with hledger 1's. Eg:
   ```cli
   $ diff <(hledger1 accounts) <(hledger accounts)
   $ diff <(hledger1 bs) <(hledger bs)
   $ diff <(hledger1 is) <(hledger is)
   ```
   (This works in bash or zsh. Elsewhere, save each report to a file and compare the files.)

6. Fix anything found, using the sections below.
   And update any scripts or config which use [changed commands or options](#changed-commands-and-options).

## Errors you might see

These are errors in hledger 2, which hledger 1 accepted. You probably won't see these.

**Lot errors.**
hledger 1 accepted cost basis annotations like `{$50}` but ignored them.
hledger 2 uses them to track lots, and checks them:

- *"no ... lots available for disposal"*:
  a sale (a negative amount with a `{...}` annotation) takes from a lot that the journal never acquires,
  often because the journal starts partway through your investment history.
  Add the missing purchases, or an opening entry which acquires the lots, giving each lot's original date and cost:
  ```journal
  2026-01-01 opening lots
      assets:stocks      10 AAPL {2024-03-15, $50}
      equity:opening/closing balances
  ```
  (A dated annotation like this needs hledger 2; hledger 1 can't read it.)

- *"This acquire posting's cost basis ($50) differs from its transacted cost ($55)"*:
  a purchase like `10 AAPL {$50} @ $55`. These must now agree.
  Usually `@` is what you paid; make the `{}` amount match it, or remove the `{}`.
  If the difference is real (eg a gift with a carried-over cost basis),
  record the cost basis as the cost, and the difference in another posting,
  as in this [gift example](investments.md#gift-received).

- *Account names ending in `{...}`*, like `assets:{savings}`,
  are reserved for [lot subaccounts](hledger.md#lot-subaccounts), and are rejected unless the braces contain a valid lot name.
  Rename such accounts.

If you don't want to deal with these now, you can run hledger with `--ignore-lots` or `-I`.
This skips lot tracking and lot error checking, giving results close to hledger 1's.
(`-I` also skips balance assertions.)

**Decimal marks.**
When a `decimal-mark` directive (or CSV rule) is in effect,
a number using the other decimal mark (eg `1,000.00` when `decimal-mark ,` is declared),
or using the declared mark more than once (eg `1,2,34`),
is now an error. Fix the number, or the directive.

**CSV rules comments.**
In CSV rules files, a line beginning with `*` is no longer a comment, and is reported as an error.
Use `#` or `;` to begin comment lines.

## Report differences

These changes can cause reports to differ, without causing an error.
Comparing reports from both versions, as in the upgrade steps above, will show them.

- **Config file default command.**
  A config file can no longer choose the command to run.
  A bare word at the top of its general section, which hledger 1 used as the default command,
  is now read as a query argument, which quietly filters every report.
  Remove it, and give the command on the command line.

- **Realised gains.**
  If your journal has sales with cost basis annotations, like `-5 AAPL {$50} @ $70`,
  hledger 2 calculates the realised gain and posts it to a gain account (`revenues:gain` by default).
  So your income statement shows the gain, and your balance sheet totals change by the same amount.
  This is usually what you want; `-I` gives hledger 1's results.

- **Aliases and `apply account`.**
  When `alias` directives and `apply account` affect the same entries,
  the aliases now apply first, to account names as written, and the parent account is added afterward (as in Ledger).
  hledger 1 did it the other way round.
  So aliases written with the parent account (`alias business:checking = ...`) no longer match,
  and aliases written without it (`alias checking = ...`) now do.
  Update the aliases if you need to. (`--alias` options are unaffected; they still apply last.)

- **CSV rules.**
  When a directive like `date-format` or `separator` is written more than once,
  the last one now wins (except for `skip`), as documented.
  If you wrote one before an `include` to override the included file's, move it after the `include`.

- **CSV data directory.**
  The `source` rule now looks for bare file names and relative paths in a `data/` directory next to the main journal,
  then in `~/Downloads`. (Paths beginning with `./` or `../` are still relative to the rules file.)
  The `archive` rule now saves files in `data/archive/` there,
  instead of in a `data/` directory next to the rules file.

- **print layout.**
  `print` now aligns amounts by their decimal mark. `--layout=hledger1` gives the old layout.

- **Decimal places.**
  Amounts which hledger infers no longer affect a commodity's display precision,
  so some reports show fewer decimal places.

- **accounts and payees.**
  Query terms which only match transactions, like `date:` or `desc:`, now exclude declared accounts or payees which aren't used.
  And `payees payee:REGEX` now matches declared payees.

- **any: and all: queries.**
  These now also work in posting reports like `register` and `balance`, not just in `print`.

- **commodities --used.**
  This now shows only commodities used in transactions; `--priced` shows those used in P directives.

## Changed commands and options

If your scripts, aliases or config files use these, update them:

- `--tldr` is now `--examples`.
- `stats -1` is now `stats --oneline`.
- `--verbose-tags` is now an option of `print` and `rewrite` only.
- The `demo` and `commands` commands have been removed.
  (`hledger help commands` lists the commands.)
- hledger-web, when listening on a non-local address (set with `--host`), is now read-only by default.
  Add `--allow=edit` to allow editing.

## What's new

Some of the most useful new features in hledger 2:

- **Speed**: [2-3x faster](PERFORMANCE.md) than hledger 1.52.
- **Localisation**: report headings and hledger-ui/hledger-web's UI can be shown in other [languages](hledger.md#languages), with `--lang`
  (currently en, de, zh).
- **Lot tracking and capital gains**: a `lots` tag tracks each purchase's cost basis and calculates gains when you sell,
  with several [cost basis methods](hledger.md#cost-basis-methods) (FIFO, LIFO, HIFO, AVERAGE..),
  and the new [holdings](hledger.md#holdings) report shows your investments, their value and gains.
- **Simpler importing**: the new [get](hledger.md#get) command fetches new transactions and market prices,
  and with a conventional `data/`, `rules/`, `prices/` layout, `hledger import` needs no arguments
  (see [Finding the data](hledger.md#finding-the-data)).
- **CSV rules**: [merge](hledger.md#merge) combines several CSV records into one transaction,
  [functions](hledger.md#functions) transform field values,
  and CSV or rules files can be included in a journal.
- **Aliases**: [command aliases](hledger.md#command-aliases) in your config file,
  and [commodity aliases](hledger.md#commodity-aliases).
- **Find anywhere**: [`::TEXT`](hledger.md#find-query) matches text in any field, as you'd see it in `print` output.
- **Compact transactions list**: the new [transactions](hledger.md#transactions) (tx) command lists transactions one per line.
- **hledger-ui** updates automatically when your files change, by default.
- **hledger-web** gains balance reports, paging and a dark mode.

For all changes, see the [release notes](relnotes.md).

## Turning on lot tracking

Once your data works with hledger 2, you can let it track your investment lots and calculate gains for you.
Usually you just add a `lots` tag to each investment commodity's declaration:
```journal
commodity AAPL  ; lots:
```
See [Lots and capital gains](hledger.md#lots-and-capital-gains).

One thing to watch for: in hledger 1 a sale was often recorded at its original cost, with a written gain posting:
```journal
2026-02-01 sell
    assets:stocks     -5 AAPL @ $50   ; at original cost
    assets:cash      $350
    revenues:gain   $-100
```
With lot tracking, `@` is read as the selling price, so this entry becomes unbalanced.
Write the selling price instead:
```journal
2026-02-01 sell
    assets:stocks     -5 AAPL @ $70
    assets:cash      $350
    revenues:gain   $-100
```
hledger then checks the written gain against its own calculation; or you can leave out the gain posting, and let hledger add it.

## Switching back to hledger 1

Journals written for hledger 2 are mostly readable by hledger 1, with these caveats:

- hledger 1 ignores cost basis annotations.
  An acquisition written as `10 AAA {$50}` means `10 AAA` in hledger 1.
  So you may need to write the transacted cost too: `10 AAA {$50} @ $50`.
- hledger 1 rejects `type: U` account declarations.
  (hledger 2 supports them but doesn't use them.)
- hledger 1 does not infer gain postings, and balances disposals at transacted cost,
  so a disposal entry with an explicit gain posting (like `revenues:gain   $-50`)
  is unbalanced in hledger 1.
  Omit the gain posting (or its amount) instead.
  See [Recording gains](hledger.md#recording-gains).
- hledger 2's `print --lots` output includes lot subaccount names like `assets:stocks:{2026-01-01, $50}`;
  hledger 1 reads these as ordinary subaccounts.
- The `lots` tag on commodity and account declarations is an ordinary tag in hledger 1, and is ignored.

## Keeping data usable with both

To keep the same journal working in both hledger 1 and hledger 2:

- write acquisitions with both cost basis and cost, with the same amount in each: `10 AAA {$50} @ $50`
- write disposals without a gain posting
  (then hledger 2 infers it, and hledger 1 shows no gain)
- don't declare `type: U` accounts (hledger 1 rejects them, hledger 2 doesn't need them)
- avoid account names ending in `{...}`
- after changes, check the journal with both versions, eg `hledger check` and `hledger1 check`
