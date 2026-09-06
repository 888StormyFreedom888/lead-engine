# Lead Engine

A daily pipeline that finds companies worth contacting, verifies each one, and hands a
person a list to act on. Built for a Copenhagen venue through 2026, then generalised so it
can be installed for another company in another country and sector.

This repository is the documentation. The engine itself is private.

## What it does

Two lanes run on the same machinery.

The first finds companies to sell to. Every weekday it draws from an official registry
export, screens against the configured geography and sectors, confirms each company still
trades and has a reachable address, removes anyone on the legal do-not-market register, and
writes a spreadsheet. A separate step drafts the emails. A person sends them.

The second finds businesses to partner with. It runs weekly and sorts candidates by how
likely they are to accept rather than by how good they look on paper, because a partner only
counts once they say yes. Same verification, same legal filters.

## It never sends anything

No message leaves the system, in either lane, at any level of autonomy. It researches,
verifies and recommends. The operator writes and sends from their own mailbox.

That is a choice, not a missing feature. Nobody should hand a machine their outbound
reputation in week one.

## How it earns more autonomy

Four rungs. The operator promotes, never the system.

| Rung | Runs itself | Operator still does |
|---|---|---|
| 1 | Nothing, manual runs only | Reads every report, decides |
| 2 | The schedule | Approves every candidate |
| 3 | Drafting the approach | Reads the draft, sends it |
| 4 | Choosing who to work up | Sends. Always, at every rung |

Promotion is earned: ten runs in a row with nothing invented, no padded empty result, no
guessed contact recorded as real. One fabricated company resets the count to zero and drops
the lane back to rung 1 the same day. A made up business looks exactly like a real one, so a
single instance destroys the value of every report before it.

## What building it taught

Three things cost the most time, and none of them were the code.

**The legal register.** Every market has a do-not-market list. In Denmark it is
reklamebeskyttelse in the company register. The live API is rate limited per day, the
official portal forbids automated access, the commercial mirror's terms forbid systematic
collection, and search engines serve consent walls to anything automated. What works is a
bulk export taken by hand every few months. It carries the protection flag as a column, so
the check runs offline against a spreadsheet with no quota and no terms problem. Budget for
it in every install.

**Two lanes, one API budget.** The weekday sell-to run spent the daily registry quota before
the partnerships run started, so the partnerships lane could not clear its own legal filter
and parked every candidate it found. The symptom looked like a quota problem. The cause was
that both lanes were reaching for registry exports built for entirely different sectors.

**The market has a size.** Measured across four registry exports covering one sector in
greater Copenhagen:

| | |
|---|---|
| Companies in the sectors | 7,202 |
| On the do-not-market register | 4,939, or 68% |
| Closed | 1,288 |
| Legally approachable | about 2,263 |

Two thirds of that market cannot be contacted at all. A client promised a fresh list every
week forever will be disappointed by month three, and it will look like the system failing
rather than the market running out. Do the arithmetic during scoping and put the number in
the proposal.

## Empty results

A system that manufactures candidates rather than report nothing is worse than one that
reports nothing, because the operator cannot tell the difference until they contact a
company that does not exist. An empty report has to be safe to file, in the wording of the
engagement as much as in the code.

## What is in here

- `GUIDE.md`, the daily operator's guide for both lanes
- `INSTALL.md` and `SETUP.md`, what an install involves and what gets configured per company
- this file

## What is not in here

The pipeline scripts, the message templates, the per company setup tooling, the pilot
playbook and the commercial material. Those sit in a private repository and are available
under an engagement.

## Contact

Morten Storm, ms@yourkeyz.io
