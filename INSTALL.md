# Install

This folder finds B2B leads for you: companies in your area, filtered, with a real
email address where one can be found. It produces a spreadsheet. It never contacts
anyone, and it never sends anything.

Two guides below. **Denmark first, because that is the only country it works in
today.** If you are somewhere else, skip to the second part.

---

# Part 1: In Denmark

## What you need first

- **Claude Code**, installed and signed in
- **Python 3.9 or newer** (`python3 --version`)
- About **two hours**, most of it waiting on the company register

## 1. Install

From inside this folder:

```bash
python3 -m pip install -r requirements.txt
python3 scripts/doctor.py
```

Three small packages. `doctor.py` checks everything and tells you what is missing.
It will report failures on a fresh copy, which is correct: nothing is filled in yet.

Nothing installs globally. Everything lives inside this folder, including the
Claude Code commands. Delete the folder and it is gone.

## 2. Tell it who you are

Open this folder in Claude Code and run:

```
/new-company
```

It interviews you for about ten minutes: what you sell, who decides on it inside a
target company, your area, your sender details, and which kinds of company are worth
most to you. Then it writes the configuration.

**One answer matters more than the rest.** It will ask where a single agreement
reaches many people at once. That answer becomes the first sector it searches, and
most days it never reaches the bottom of the list. Think about it before you answer,
and correct what it proposes.

## 3. Get the company data

The engine reads company data you download yourself. It is the only manual step and
the one everything else waits on, so do it now rather than later.

1. Go to **datacvr.virk.dk** and choose **Udvidet søgning**
2. Apply the filters written in `config/export_filter.md`
3. Export the result and save the `.xlsx` file into `data/exports/`

Split it into several exports if the site limits the row count. The engine reads
every file in that folder and merges them.

Then check what you have:

```bash
python3 scripts/check_sectors.py
python3 scripts/runway.py --per-sector
```

The second command shows how many companies each sector has and how long they last.
A sector showing zero means you have not downloaded it yet, not that the market is
empty.

## 4. Write your email

Two files in `templates/`, one plain text and one HTML, saying the same thing.
Recipients see one or the other depending on their mail program, so keep them in step.

**Keep the P.S. at the bottom.** It is a one line opt out, and under Danish
markedsføringsloven §10 it is what makes each mail an individual approach rather
than a mass send. Three lines, and the cheapest legal protection you have.

## 5. First run

```bash
python3 scripts/run_daily.py --count 5
```

Five, not twenty. Read the sheet it produces. Two questions only:

- Are these the companies you want?
- Does that email address really belong to that company?

If the answer to the first is no, your sector order is wrong. Go back to step 2 and
fix it before running again. Every company delivered is permanently spent, so a bad
run at full size wastes real inventory.

## 6. Make it daily

```bash
./make-routines.sh --at 06:00
```

This writes two scheduled task descriptions. Install the **lead** one. The drafting
one is deliberately switched off: look at a week of sheets before letting anything
write into a mailbox.

## What you get, every morning

| File | What it is |
|---|---|
| `..._leads.xlsx` | The sheet. Read it, mark Yes or No in the status column. |
| `..._mailmerge.csv` | One row per company, ready for your mail merge tool. |
| `..._drafts.json` | Pre written emails, if you connect Gmail to Claude Code. |

## Three things worth knowing

**It refuses companies on purpose.** Advertising protected companies, insolvent
ones, anyone you have already contacted, and any address that does not plausibly
belong to the company it is listed under. That refusal is the point of the tool. If
you relax a filter to get more volume, you have a list you could have bought cheaper.

**Twenty a day is a floor, not a promise.** It depends entirely on how much register
data you downloaded. Run `runway.py --per-sector` monthly and export more before you
run dry.

**Companies it fails on are not thrown away.** If no address can be found, the
company waits in `data/retry_pool.csv` and comes back later rather than being used
up. That is why the pool lasts.

## When something looks wrong

```bash
python3 scripts/doctor.py
```

Run it first, every time. It checks the interpreter, the packages, every config
file, whether your templates are still placeholders, whether register data exists,
your keys, and whether `seen.csv` is carrying somebody else's history.

---

# Part 2: Outside Denmark

**Read this before installing anything.** The engine does not work outside Denmark
yet, and two of the reasons are legal rather than technical.

## The two questions that come before any code

**1. Is cold B2B email lawful where you are?**

Denmark allows it, with a visible opt out. Germany effectively does not: under UWG
§7 unsolicited B2B email requires prior consent, and the opt out line that works in
Denmark does not help there. Other countries sit somewhere between.

This is a go or no go. Find out before spending a day on setup.

**2. Does your country have a register of companies that refuse marketing?**

Denmark does. Every company can register advertising protection, the register
publishes that flag, and this engine treats it as an absolute filter. It is the
single most important safeguard in the whole pipeline.

If your country has no equivalent, that safeguard has nothing to work with. The
engine will still run, but the compliance story it is built around does not hold,
and you should decide consciously what replaces it.

## What is technically missing

The engine reads Danish company data by its Danish column names, uses the Danish
company number as its identifier, and uses Danish postcodes as its geography. Those
are not settings. They are wired into the core.

Adding a country means writing:

- a reader for that country's company register file or API
- the identifier concept, whatever replaces the Danish CVR number
- the geography, whatever replaces a postcode range
- the industry classification for that country
- the contact page words the enrichment looks for, in that language

Roughly a week of work for the first country, less for the ones after it, because
most of it is building the seam rather than the country.

`scripts/registry/README.md` describes what a country adapter must provide.
`python3 scripts/doctor.py` will refuse cleanly and name the countries that exist.

## What already works anywhere

Not everything is Danish. These transfer unchanged:

- the email attribution check, which decides whether an address really belongs to
  the company it is listed under
- the retry pool, so companies you fail on are held rather than burned
- the permanent record of who has already been contacted
- the eleven territory framework the lead scout searches: registers, memberships,
  rankings, hiring, press, tenders, events, property, social, adjacencies,
  communities

That framework is the transferable part. The Danish source list is one instance of
it, and the same eleven territories apply in any country.

## The cheapest first country

Sweden and Norway. Bolagsverket and Brønnøysund are close analogues to the Danish
register, business culture is similar, and marketing law is comparable. Spain and
Greece are considerably more expensive, because open bulk company data is not the
norm there and the cost model changes from free to paid.

---

Questions about the Danish setup: read `SETUP.md`, which goes deeper on each config
file. Questions about adding a country: `scripts/registry/README.md`.
