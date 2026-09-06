# Daily operator's guide

Quick reference for day-to-day use. For one-time setup, config tuning, and
where each column's data comes from, see [README.md](README.md) instead —
this file doesn't repeat any of that.

## Run the daily pipeline

```bash
python3 scripts/run_daily.py
```

Or `/leads` from Claude Code — same thing, plus a spoken summary.

**Exit codes:**

| Code | Meaning | What to do |
|---|---|---|
| 0 | Success | Sheet written to `data/YYYY-MM-DD_<sector>_leads.xlsx` |
| 2 | Current sector exhausted | Rotation already advanced — run the exact same command again. If the second run also exits 2, the export files hold nothing further; pull a fresh datacvr export |
| 3 | cvrapi.dk headcount quota hit | Rotation is deliberately **not** advanced. Retry tomorrow — nothing to do today |

## Mark a lead yes or no

```bash
python3 scripts/mark_status.py "Company Name" yes
python3 scripts/mark_status.py "Company Name" no
```

Finds the row by company-name match across every `data/*.xlsx` sheet and
flips its Status cell from `Attacked` to `Yes`/`No`.

**What it does NOT do:** it doesn't suppress the company from future runs.
The sheets carry no CVR column, so a "no" only updates that sheet. To
permanently exclude a company from all future exports, add its CVR by hand
to `data/do_not_contact.csv` (columns: `cvr,name,reason,added_on`).

## Send drafts

Use the `/draft` skill — creates Gmail drafts from the most recent lead run
for every recipient with a usable email (real found address or Pipedrive,
never a pattern guess). Always ask before running it; it's not automatic
after a daily run.

## Reading the RUNWAY block

Printed at the end of every `run_daily.py` run:

| Field | Meaning |
|---|---|
| Untouched pool | Companies from the exports not yet looked up |
| Measured pass rate | Fraction of *looked-up* companies that cleared the headcount floor, from observed data |
| Estimated qualifying companies left | Untouched pool × pass rate |
| Runway (working days) | Estimated qualifying companies left ÷ `leads_per_day` |

**Rules of thumb:**
- Runway below ~40 working days → flag it. Options are a fresh, broader
  datacvr export, or lowering `leads_per_day` — never decide this yourself,
  report it and let the operator choose.
- Pass-rate sample under 50 companies → say the estimate is still rough,
  not settled.

## Status column lifecycle

`not yet` → `Attacked` (set automatically when a draft is created) →
`Yes` / `No` (set by hand, via `mark_status.py` above, once a reply comes in
or a decision is made).

---

# Add-on module — partnerships lane

An optional second lane on the same install. The daily pipeline above finds companies to
**sell to**. This one finds businesses to **partner with**, and it is a different product
to the client even though it shares the machinery.

Sell it where the client's growth depends on other people's customers rather than on
outbound volume: venues, clinics, studios, hospitality, anything with a referral or
cross-sell motion. Proven on a Copenhagen venue, first run 2026-09-06.

## What the client gets

A weekly report of businesses worth partnering with, each verified to trade today, to have
a reachable human, and to be legally contactable. Sorted by **willingness to say yes**, not
by how good they look on paper, because a partner only counts once they accept.

No outreach. Ever. The lane researches and recommends; the client's own people make contact
in their own words from their own mailbox. That is a feature to sell, not a limitation:
nobody buys a system that emails on their behalf on day one, and it removes the objection
before it is raised.

## The autonomy ladder is the sales story

Four rungs, and the client stays in control of promotion.

| Rung | What runs itself | What the client still does |
|---|---|---|
| 1 | Nothing. Manual runs only | Reads every report, decides approach or park or drop |
| 2 | The schedule | Approves every candidate |
| 3 | Drafting the approach | Reads the draft, sends it themselves |
| 4 | Choosing who to work up | Sends. Always. At every rung |

**Promotion is earned, not granted:** ten consecutive runs with nothing invented, no padded
zero, no guessed contact recorded as real. One fabricated candidate resets the counter to
zero and drops the lane to rung 1 the same day. A made-up business looks identical to a real
one, so a single instance destroys the value of every report before it.

Clients buy this. It answers "how do I know it isn't making things up" with a mechanism
rather than a promise.

## What you must configure per client

| Item | Why it is per-client |
|---|---|
| Territories | 8 on the first install: adjacent categories, neighbourhoods, hotels, retail, restaurants, new openings, awards lists, trade bodies. Rewrite for the client's market |
| The existing-partners list | The seen-list. Without it the first report proposes people they already work with, and credibility is gone in one run |
| Advertising-protection source | See below. Country-specific and the single hardest part |
| Do-not-contact list | Client's own, plus anyone who has already said no |
| Scoring weights | Fit, reach, willingness, effort. Willingness leads the sort |

## The registry problem, and how to price it

Every market has a legal do-not-market register, and getting at it is where these projects
actually stall. In Denmark it is **reklamebeskyttelse** in CVR. What we learned building it:

- The live API has a **daily quota**, and if the client also runs the sell-to lane, that
  lane spends the quota first every morning. Two lanes, one budget.
- The obvious web sources are closed: the official portal forbids automated access, the
  commercial mirror's terms forbid systematic collection, and general search engines serve
  consent walls and CAPTCHAs to anything automated.
- **The fix is a bulk export, taken by hand from the official registry, refreshed every few
  months.** It carries the protection flag and often an email as columns, so the check runs
  offline against a spreadsheet with no quota and no terms problem.

Scope this explicitly in every proposal. It is an afternoon of setup per sector, it recurs,
and it is the difference between a system that works and one that parks every candidate as
unverifiable.

## Set expectations on market size before you sell

Run the arithmetic during the pilot, not after. For one body-care market in greater
Copenhagen, measured across four registry exports:

| | |
|---|---|
| Companies in the sectors | 7,202 |
| Advertising-protected | 4,939 (68%) |
| Closed | 1,288 |
| **Legally approachable** | **~2,263** |

Two thirds of that market is legally off limits. A client expecting a fresh list every week
forever will be disappointed by month three, and it will look like the system failing rather
than the market being finite.

So: sell a **finite, high-quality pipeline**, not a tap. Say the number in the proposal.
Then a report that returns zero reads as the system being honest, which is the behaviour you
want them to trust, instead of reading as a fault.

## Delivery shape

1. **Scoping.** Territories, the existing-partners list, the registry route for that
   country, and the market-size arithmetic. Half a day, and it is the half that decides
   whether the project is viable.
2. **Registry exports.** By sector, by hand, into the install.
3. **Pilot.** Manual runs at rung 1 until ten come back clean. This is the client learning
   to trust it, and it cannot be shortened.
4. **Handover.** The operator guide, the ladder, and who promotes rungs.

## Two failure modes to write into the contract

**Padding.** An agent that produces candidates to avoid reporting nothing is worse than one
that reports nothing, because the client cannot tell the difference until they contact
someone who does not exist. Zeros must be reportable without penalty, in the wording of the
engagement as well as in the prompt.

**Lane bleed.** If the client runs both lanes, the partner candidates and the sell-to
prospects must never share a source file. Otherwise the outbound engine starts selling to
the businesses the partnership lane is courting, with two contradictory messages to the same
people. Keep the registry exports in separate folders, and say so in the handover.
