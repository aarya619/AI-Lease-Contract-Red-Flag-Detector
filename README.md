# AI Lease & Contract Red-Flag Detector

Reads a rental lease, splits it into clauses, flags the ones that are risky for the tenant, ranks them by real-world risk, cites the relevant provision for every flag, and drafts a negotiation email.

**Why I built it:** [write 1-2 lines in your own words: what bothered you about leases or renting]

## How it works

```
lease (txt / PDF / scan / photo)
   -> OCR / text extraction        redflag/ocr.py
   -> LLM clause segmentation      redflag/segment.py   (rule-based fallback if no API key)
   -> risk scoring vs a ruleset    redflag/score.py + rulesets/*.json
   -> ranked report + citations    redflag/report.py
   -> negotiation email draft      redflag/email_draft.py
```

- **Ranked by risk, not volume.** Each flag gets `risk = severity x confidence`. A lock-out clause outranks three minor ones.
- **Every flag cites its source.** Rules live in a JSON ruleset with an act and provision. If a rule isn't in the ruleset, it can't be flagged, so the tool can't invent law.
- **Honest about uncertainty.** Where I haven't confirmed a section number against the official text, the report says so (`section_checked: false`).

## Legal-safety boundary

This is information and drafting help, not legal advice. It never asserts a legal conclusion. The shipped ruleset benchmarks against the **Model Tenancy Act, 2021**, which is a *model* law circulated to states and is not automatically binding everywhere. Before relying on it for a specific state, add a state ruleset (see below) after checking that state's actual law.

## Run it

```bash
git clone https://github.com/aarya619/lease-red-flag-detector.git
cd lease-red-flag-detector
pip install -r requirements.txt

# works offline, no API key needed:
python -m redflag.cli sample/sample_lease.txt --no-llm --email

# with LLM clause segmentation:
cp .env.example .env     # add your ANTHROPIC_API_KEY
python -m redflag.cli sample/sample_lease.txt --email
```

Scanned PDFs and photos need the Tesseract binary installed (`apt install tesseract-ocr`, or the Windows installer).

Run tests: `python -m pytest`

## Add a state ruleset

Copy `rulesets/model_tenancy_india.json`, change the citations to that state's real provisions, and run with `--ruleset rulesets/your_state.json`. Set `"section_checked": true` only after you've read the official text.

## Limitations

- Detection is pattern-based, so unusual wording can be missed. Treat "no flags" as "nothing obvious", not "safe".
- Only the 7 rules in the ruleset are checked.
- OCR quality depends on the scan.
- Not tested on real-world leases beyond [fill in honestly: e.g. the leases from my audits].

## Author

Aarya Kothe | [LinkedIn](https://www.linkedin.com/in/aarya-kothe-3887b62b5/) | [Portfolio](https://aarya-portfolio-theta.vercel.app/)
