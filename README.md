# College Equity Explorer

**Where do Pell-eligible (lower-income) students actually graduate, and can they afford it?**
An interactive tool and data pipeline built on the U.S. Department of Education's College Scorecard, covering **2,423** U.S. colleges (1,602 four-year and 821 two-year schools).

**Live tool:** https://kieran168.github.io/college-equity-explorer/
**Analysis:** [`college_scorecard_pipeline.ipynb`](./college_scorecard_pipeline.ipynb)

> _Built by Kieran Yuen, a data analyst working in college access. Replace this line and the nonprofit references below with whatever framing you are comfortable making public._

---

## Why this exists

A college's raw graduation rate mostly measures **who it admits**, not how well it serves them. Selective schools that enroll wealthy, well-prepared students post high graduation rates almost automatically, which tells you little about how a lower-income, first-generation student would fare there.

For an organization focused on college access for lower-income students, the useful question is not "what is the graduation rate?" It is "among schools that actually serve students like ours, which ones graduate them, affordably?"

This project answers that by pulling the federal data, breaking graduation out by income (Pell Grant status), pairing it with what lower-income families actually pay, and surfacing it in a tool an advisor can use in a meeting with a student.

## What it does

- **Explore** every four-year college and two-year school in the Scorecard, sortable and filterable by state, sector, institution level, cost, and Pell graduation rate.
- **Start with one state, then add more.** A counselor begins by picking a state (the focused entry point), then adds neighboring states as needed, since students often weigh schools across state lines.
- **Fit score.** A single, plain-language signal (Strong, Good, Fair, Caution) that combines how well a school graduates Pell-eligible students with how affordable it is, so an advisor does not have to weigh six columns in their head. A school that graduates fewer than about 35% of its Pell-eligible students is always flagged Caution, regardless of price, because a degree you do not finish is not a bargain.
- **Compare a shortlist.** Star the schools a student is weighing and view them side by side, to pressure-test a choice or surface stronger alternatives.
- **Top five highlights** per state: the schools with the highest graduation rate for Pell-eligible students.

## The data

| | |
|---|---|
| **Source** | U.S. Department of Education [College Scorecard API](https://collegescorecard.ed.gov/data/) |
| **Coverage** | 2,423 currently operating, predominantly undergraduate institutions |
| **Published** | June 2026 |
| **Key metrics** | Pell (lower-income) graduation rate, net price for families earning under $30k, overall graduation rate, and the equity gap between them |

**Pell as the measure of "lower-income."** The Scorecard reports outcomes for Pell Grant recipients, the standard federal marker for students with significant financial need. Counselors and families typically know whether a student is Pell-eligible, which makes it a more concrete lens than the vaguer phrase "low-income."

**A caveat worth stating up front:** graduation rates describe students who entered college several years ago, since completion takes six or more years to measure. The data file is current, but the student experience it captures is not "as of today." This is true of every graduation-rate source, but it is the kind of thing a careful analyst names rather than hides.

## How it is built

**Extract.** Python (`requests`) against the Scorecard REST API. Handles pagination (the API caps results at 100 per page), and uses a field-discovery step that probes the live API for one school and reads the real field names off the response rather than guessing them from documentation.

**Transform.** `pandas`. The non-obvious work:
- **Merging two Pell-graduation fields.** Two-year and four-year schools report their Pell completion into different API fields, because a two-year degree is measured on a different clock. Naively filtering on the four-year field silently drops every two-year school, so the pipeline pulls both and coalesces them.
- **Coalescing net price by income** into a single column populated for every sector.
- **Computing the equity gap** (overall grad rate minus Pell grad rate) as the per-school fairness signal.
- **Handling real-world data quirks** honestly: suppressed values, open-admission schools with no admission rate, and negative net prices (where aid exceeds cost for the lowest-income students, which is a real and good thing, not a bug).

**Deliver.** The cleaned dataset is exported to JSON and embedded in a single self-contained HTML file (vanilla JavaScript, no build step) that opens in any browser and deploys to GitHub Pages as is.

**Stack:** Python, pandas, requests, vanilla HTML, CSS, and JavaScript.

## Key findings

- **Raw graduation rates are misleading for equity questions.** Breaking the numbers out by Pell status changes the picture substantially: schools with similar headline rates can serve lower-income students very differently.
- **Two-year and four-year schools** graduate Pell-eligible students at meaningfully different rates. _Insert the exact averages from the comparison table in section 6 of the notebook here._
- **The for-profit pattern is real and documented.** Some for-profit institutions enroll overwhelmingly Pell-eligible students and graduate very few of them. Nationally, fewer than about one in five Pell students complete a bachelor's degree at for-profits, versus roughly half at public and private nonprofit four-year schools. (Sources: U.S. Department of Education, Third Way, The Education Trust.)
- **The equity gap needs context.** A small gap at a low-graduating school is not good news, because it just means everyone graduates at a low rate. The Fit score encodes this by treating absolute Pell graduation as the floor, not the gap alone.

## Honest limitations

- **Suppression bias.** The Pell graduation rate is withheld for schools with too few Pell students in the cohort, so about 17% of schools drop out, disproportionately those serving few lower-income students.
- **Cohort timing.** Graduation figures reflect students who entered years ago (see above).
- **Two-year and four-year metrics are not perfectly comparable**, since they use different completion clocks.
- **The Fit score is a heuristic, not a verdict.** It cannot see a school's programs, location, or support services, and it weights graduation and affordability with chosen, documented weights.
- **Open-admission imputation.** Schools with no reported admission rate (mostly two-year schools) are treated as admitting nearly everyone for filtering purposes.

## Future directions

- **Distance and nearby schools.** The Scorecard includes each school's city, ZIP, and latitude and longitude, so a "show me schools within X miles" feature is feasible. It would let an advisor recommend strong alternatives close to a student's home.
- **Embed in advising workflows** (for example, alongside school records in a CRM) so advisors see this data in context during student meetings.
- **A "better alternatives" recommender** that, given a school a student has chosen, surfaces higher-graduating, affordable options nearby.
- **Within-level statistical modeling** of what predicts Pell completion, run separately for two-year and four-year schools to keep the comparison clean.

## Repository structure

```
college-equity-explorer/
  README.md
  index.html                          the self-contained tool (deploys to GitHub Pages)
  college_scorecard_pipeline.ipynb    the extract, clean, and analyze notebook
  data/
    schools.json                      cleaned dataset (2,423 schools)
```

## Run it yourself

The tool needs no setup. Open `index.html` in any browser, or visit the live link above.

To regenerate the data, open the notebook in Google Colab, add a free [data.gov API key](https://api.data.gov/signup/) as a Colab secret named `SCORECARD_API_KEY`, and run all cells. The final cell exports `schools.json`.
