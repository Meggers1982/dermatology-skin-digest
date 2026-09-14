# Dermatology & Skin Science Research Digest

A GitHub Actions workflow that searches curated dermatology, allergy and immunology, and biochemistry relevant to skin journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** - Sends a short email with study count and a dashboard link

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as Allure, Byrdie, Vogue beauty, Self, Well+Good, Healthline, Prevention, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Dermatology & Skin Science Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Dermatology | 53 | 2 (chunks 1-2) |
| Allergy & Immunology | 139 | 2 (chunks 1-2) |
| Biochemistry | 188 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The category CSVs in `data/` are now hand-maintained and are the source of truth. `scripts/extract_journals.py` originally generated them from a spreadsheet that no longer exists, so re-running it would wipe hand-added rows.

## Journal list audit (2026-09-14)

Pulled OpenAlex's top sources for this digest's subject areas over the prior year, diffed them against the CSVs by ISSN and title, and kept only titles that PubMed indexes with at least 20 articles in the last 12 months. Every row searches PubMed with no topic filter, so each added journal's full weekly output enters the digest.

Added (9):

- **Dermatology:** Clinical, Cosmetic and Investigational Dermatology (~460 PubMed articles/yr), Dermatology and Therapy (~410), JAAD International (~280), International Journal of Women's Dermatology (~80), Annals of Dermatology (~60)
- **Allergy & Immunology:** Frontiers in Allergy (~265), Allergologia et Immunopathologia (~140; the NLM record is tagged Spanish, but all of its PubMed articles from the last 12 months are in English), Allergy, Asthma & Clinical Immunology (~70), Asia Pacific Allergy (~50)

Left out:

- **Not in PubMed, or no articles there in the last 12 months:** Annales de Dermatologie et de Vénéréologie - FMC (190 subject-area articles/yr), Revue française d'allergologie (156), Cosmetics (MDPI, 115), Allergo Journal (79), Allergo Journal International, and Russian Journal of Clinical Dermatology and Venereology. Several smaller regional dermatology titles were also left out for this reason.
- **Too few PubMed articles:** SKIN: The Journal of Cutaneous Medicine (303 articles/yr in OpenAlex, but only 1 in PubMed) and Aerobiologia (2)
- **Mega-journals:** none of the candidates published more than 1,000 articles/yr

## Manual Trigger

Go to **Actions -> Dermatology & Skin Science Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Dermatology`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

## Repo Structure

```text
.github/
  workflows/
    dermatology-skin-digest.yml
scripts/
  dermatology_skin_digest.py
  merge_results.py
  extract_journals.py
data/
  Dermatology.csv
  Allergy & Immunology.csv
  Biochemistry.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for skincare science, skin conditions, cosmetic dermatology, and skin health journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
