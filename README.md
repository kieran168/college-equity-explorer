# College Equity Explorer

**Where do low-income students actually graduate — and can they afford it?**
An interactive tool and data pipeline built on the U.S. Department of Education's College Scorecard, covering **2,423** U.S. colleges (1,602 four-year and 821 community colleges).

🔗 **Live tool:** _add your GitHub Pages link here once deployed_
📓 **Analysis:** [`college_scorecard_pipeline.ipynb`](./college_scorecard_pipeline.ipynb)

> _Built by [Your Name] — data analyst working in college access. Replace this line and the nonprofit references below with whatever framing you're comfortable making public._

---

## Why this exists

A college's raw graduation rate mostly measures **who it admits**, not how well it serves them. Selective schools that enroll wealthy, well-prepared students post high graduation rates almost automatically — which tells you little about how a *low-income, first-generation* student would fare there.

For an organization focused on college access for low-income students, the useful question isn't *"what's the graduation rate?"* It's *"among schools that actually serve students like ours, which ones graduate them — affordably?"*

This project answers that by pulling the federal data, disaggregating graduation by income (Pell-grant status), pairing it with what low-income families actually pay, and surfacing it in a tool an advisor can use in a meeting with a student.

## What it does

- **Explore** every four-year college and community college in the Scorecard, sortable and filterable by state, sector, institution level, cost, and low-income graduation rate.
- **Fit score** — a single, plain-language signal (Strong / Good / Fair / Caution) that combines how well a school graduates low-income students with how affordable it is, so an advisor doesn't have to mentally weigh six columns. A school that graduates fewer than ~35% of its low-income students is always flagged **Caution**, regardless of price — because a degree you don't finish isn't a bargain.
- **Compare a shortlist** — star the schools a student is weighing and view them side by side, to pressure-test a choice or surface stronger alternatives.
- **Top picks** that update as you filter — e.g., the strongest-fit schools for low-income students in a given state.

## The data

| | |
|---|---|
| **Source** | U.S. Department of Education [College Scorecard API](https://collegescorecard.ed.gov/data/) |
| **Coverage** | 2,423 currently-operating, predominantly-undergraduate institutions |
| **Published** | June 2026 |
| **Key metrics** | low-income (Pell) graduation rate · net price for families earning under \$30k · overall graduation rate · the equity gap between them |

**A caveat worth stating up front:** graduation rates describe students who *entered* college several years ago — completion takes 6+ years to measure. The data file is current; the student experience it captures is not "as of today." This is true of every graduation-rate source, but it's the kind of thing a careful analyst names rather than hides.

## How it's built

**Extract** — Python (`requests`) against the Scorecard REST API. Handles pagination (the API caps results at 100/page), and uses a field-discovery step that *probes the live API for one school and reads the real field names off the response* rather than guessing them from documentation.

**Transform** — `pandas`. The non-obvious work:
- **Merging two Pell-graduation fields.** Community colleges and four-year schools report their low-income completion into *different* API fields (a 2-year degree is measured on a different clock). Naively filtering on the four-year field silently drops every community college; the pipeline pulls both and coalesces them.
- **Coalescing net price by income** into a single column populated for every sector.
- **Computing the equity gap** (overall grad rate − low-income grad rate) as the per-school fairness signal.
- **Handling real-world data quirks** honestly: suppressed values, open-admission schools with no admission rate, and negative net prices (where aid exceeds cost for the lowest-income students — a real and good thing, not a bug).

**Deliver** — the cleaned dataset is exported to JSON and embedded in a single **self-contained HTML file** (vanilla JS, no build step) that opens in any browser and deploys to GitHub Pages as-is.

**Stack:** Python · pandas · requests · vanilla HTML/CSS/JS

## Key findings

- **Raw graduation rates are misleading for equity questions.** Disaggregating by Pell status changes the picture substantially: schools with similar headline rates can serve low-income students very differently.
- **Community colleges vs. four-year schools** graduate low-income students at meaningfully different rates — _insert the exact averages from the comparison table in section 6 of the notebook here._
- **The for-profit pattern is real and documented.** Some for-profit institutions enroll overwhelmingly low-income (Pell) students and graduate very few of them; nationally, fewer than ~1 in 5 Pell students complete a bachelor's at for-profits, versus roughly half at public and private nonprofit four-year schools. _(Sources: U.S. Dept. of Education; Third Way; The Education Trust.)_
- **The equity gap needs context.** A *small* gap at a *low-graduating* school is not good news — it just means everyone graduates at a low rate. The tool's Fit score encodes this by treating absolute low-income graduation as the floor, not the gap alone.

## Honest limitations

- **Suppression bias.** The low-income graduation rate is withheld for schools with too few Pell students in the cohort, so ~17% of schools drop out — disproportionately those serving few low-income students.
- **Cohort timing.** Graduation figures reflect students who entered years ago (see above).
- **Two-year vs. four-year metrics aren't perfectly comparable** — different completion clocks.
- **The Fit score is a heuristic, not a verdict.** It can't see a school's programs, location, or support services, and it weights graduation and affordability with chosen (documented) weights.
- **Open-admission imputation.** Schools with no reported admission rate (mostly community colleges) are treated as admitting ~everyone for filtering purposes.

## Future directions

- **Embed in advising workflows** (e.g., alongside school records in a CRM) so advisors see this data in context during student meetings.
- **"Better alternatives" recommender** — given a school a student has chosen, surface higher-graduating, affordable options in the same state.
- **Student-specific views** that adjust framing based on a student's financial-aid profile.
- **Within-level statistical modeling** of what predicts low-income completion, run separately for two-year and four-year schools to keep the comparison clean.

## Repository structure

```
college-equity-explorer/
├── README.md
├── index.html                       # the self-contained tool (deploys to GitHub Pages)
├── college_scorecard_pipeline.ipynb # the extract -> clean -> analyze notebook
└── data/
    └── schools.json                 # cleaned dataset (2,423 schools)
```

## Run it yourself

The tool needs no setup — open `index.html` in any browser, or visit the live link above.

To regenerate the data, open the notebook in Google Colab, add a free [data.gov API key](https://api.data.gov/signup/) as a Colab secret named `SCORECARD_API_KEY`, and run all cells. The final cell exports `schools.json`.
