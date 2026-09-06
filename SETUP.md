# Setting up a company

## Installing on a machine that has never run this

Three steps, then check.

```bash
python3 -m pip install -r requirements.txt   # requests, beautifulsoup4, openpyxl
python3 scripts/doctor.py                    # what is missing, and what to do
```

Open the folder in Claude Code and its four slash commands and three agents come
with it: `/new-company`, `/leads`, `/draft`, `/scout`. They live in `.claude/` inside
this folder, so nothing has to be installed globally and nothing on the host machine
is touched.

`doctor.py` is the thing to run whenever a run does something surprising. It checks
the interpreter, the packages, every config file, whether the outreach templates are
still placeholders, whether any register data exists, the keys, and whether
`seen.csv` is carrying somebody else's history. It fixes nothing and says what to do
about each failure.

**What it cannot check:** whether cold B2B email is lawful in the target market, and
whether that market has an advertising-protection register. Both are country
questions, both are load-bearing, and `DK` is the only adapter that exists.

---

This folder is the machine with no company in it. Copy it, fill in six things, and
it runs. Nothing in `scripts/` should ever need editing — if you find yourself in
there to change a company detail, something belongs in config that isn't.

The fastest route is `./new-company.sh "Company Name"`, which copies this folder,
carries the lookup caches over from an existing instance, interviews you about what
the company sells, and writes most of the below. This file is what it is doing, and
what to check afterwards.

## The six things

### 1. `company.json` — who you are

Name, sender identity, own domains, subject line, greetings. Every field marked
`FILL IN`.

**`own_domains` matters more than it looks.** Your own group's entities turn up in
the register pool like anyone else's. List every domain you own, including sister
companies and holding entities, or the pipeline will cheerfully draft a cold email
to your own office.

**`user_agent` is not a place to be anonymous.** It goes to every site you scrape
and to the registry APIs. Convention is `Company purpose - contact-email`, so anyone
reading their server logs can reach a human.

### 2. `templates/outreach_*.txt` and `.html` — what you send

Two files saying the same thing, because recipients see one or the other depending
on their mail client. Keep them in step.

**Keep the P.S. opt-out.** Under Danish markedsføringsloven §10 — and most European
equivalents — a visible, effortless opt-out is what makes each mail an individual
approach rather than a mass send. It is the cheapest legal protection in the whole
pipeline and it is three lines.

Naming is `outreach_<language>.txt`, matching `outreach.language` in `company.json`.

### 3. `config/settings.json` — where and how big

- `country` — which adapter `scripts/registry/` loads. `DK` is the only one
  implemented; see `scripts/registry/README.md` to add another.
- `postnummer_from` / `postnummer_to` / `extra_postnumre` — the target geography.
- `min_employees` — the size floor. **Single source of truth.** Never hardcode a
  number in a script, agent file or playbook.
- `leads_per_day` — a floor, not a per-sector target.
- `enrich.contact_page_hints` and the mailbox prefixes — language-specific. A
  non-Danish setup edits these, not code. Swedish would add `om-oss`, `medarbetare`.

### 4. `config/sectors.json` — who you sell to

**The part people get wrong.** This is the targeting strategy, not a formality. The
runner works down the list in order and most runs never reach the bottom, so the
first sector is the one that matters.

`new-company.sh` generates this from an interview about what the company sells and
who decides. **Review what it writes.** The example left in the file is from a spa
selling employee-benefit agreements, which put unions and associations first because
one agreement there reaches tens of thousands of members. Your leverage sector is
almost certainly somewhere else.

`config/reference/db25-6digit.csv` holds the 738 official Danish DB25 codes. Validate
with `python3 scripts/check_sectors.py` — it reports invalid codes and, separately, valid
codes you have no export data for. The second list is a download list, not a fault.

### 5. `agents/user.md` — who the scout is working for

The buyer profile, the offer, and what disqualifies a company. The scout scores
channels against this, so vagueness here produces vague channels. An honest "we don't
know yet" is better than a guess that later gets treated as settled.

### 6. `config/secrets.json` — the keys

Copy `secrets.example.json` and fill in. Gitignored, and never part of the template.
`new-company.sh` copies the real file across from an existing instance.

## What you do NOT copy between companies

| Do carry over | Never carry over |
|---|---|
| `data/employees.csv`, `websearch.csv`, `wikidata.csv` — lookup caches, expensive in registry quota, and facts about companies rather than about you | `data/seen.csv` — who *you* already contacted |
| `data/exports/` and any register exports — raw public data, reusable | `data/do_not_contact.csv` — *your* opt-outs |
| The rejected rows in `channels-tried.md` — a site's robots.txt doesn't change per company | `data/state.json` — position in *your* rotation |
| | `data/*.xlsx` — finished lead sheets |

Copying `seen.csv` by accident is the quiet failure: the new company skips every
company the old one contacted, and its first weeks produce almost nothing while
looking like they ran fine.

## Then

```bash
python3 scripts/run_daily.py
```

Exit codes and the daily routine are in [GUIDE.md](GUIDE.md). Data provenance and
config tuning are in [README.md](README.md). The rules that never bend are in
[CLAUDE.md](CLAUDE.md) — read that one before changing a filter.

## Scheduling

The two routines are machine-level, live in `~/.claude/scheduled-tasks/`, and hardcode
an absolute path — so a copied folder has no schedule until its own pair is generated.
`new-company.sh` writes them without installing them.

**Stagger them.** The lead run spends registry lookup quota, and two companies firing
at the same minute compete for it. Leave at least an hour between instances.

**The drafts routine starts disabled.** Look at the first week's sheets before drafts
accumulate against a sector rotation nothing has validated yet. Enabling is one command.

## One company per folder

Not one folder with a company switch. The scout, the ledger, the sector rotation and
the contact history are all per-company, and the moment they share a folder the
`seen.csv` logic stops meaning anything.
