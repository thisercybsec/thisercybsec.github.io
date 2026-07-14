# Security Portfolio — Chirpy Setup

## 1. Get this into your GitHub Pages repo

1. Create a new repo named exactly `YOURUSERNAME.github.io`
2. Copy every file/folder in this scaffold into that repo (keep the
   folder structure exactly as-is — Jekyll relies on the `_` prefixed
   folder names)
3. Edit `_config.yml`: replace `YOURUSERNAME`, your display name, and
   tagline
4. Push to GitHub
5. In the repo: **Settings → Pages → Source → Deploy from a branch →
   `main`** (Chirpy's GitHub Action-based build also works if you copy
   `.github/workflows/pages-deploy.yml` from the
   [chirpy-starter repo](https://github.com/cotes2046/chirpy-starter) —
   recommended, since it handles the Jekyll build for you with no local
   Ruby install required)
6. Add a square image at `assets/img/avatar.png` for your sidebar photo

Local preview (optional, requires Ruby + Bundler installed on your machine):

```bash
bundle install
bundle exec jekyll serve
```

Not required to publish — GitHub's own Pages build (or the Chirpy GitHub
Action) does this for you on every push. Local preview is only useful if
you want to check formatting before pushing.

## 2. Where your existing DRP files go

Your six source files map to posts like this. **Part 1 is already fully
built** in `_posts/2026-07-13-mroc-drp-01-company-overview.md` as a worked
example — use it as the pattern for the rest.

| Source file | Becomes | Status |
|---|---|---|
| `Midwest_Regional_Outpatient_Clinic-intro.docx` | Part 1: Company Overview | ✅ Done |
| `Critcal_Functions_Spreadsheet.xlsx` | Part 2: Critical Business Functions | To do |
| `Risk_Reg.xlsx` | Part 3: Risk Register Summary | To do |
| `Midwest_Regional_Outpatient_Clinic_strategy.docx` | Part 4: Risk Response Strategies (R-001–R-006) | To do |
| `MROC_DRP.docx` (Pre/Post-Event Mitigations sections) | Part 5: Mitigation & Recovery Procedures | To do |
| `MROC_DRP_Test_Schedule.xlsx` + `MROC_DRP.docx` (Test Plan section) | Part 6: Testing & Maintenance | To do |

`MROC_DRP.docx` is your master document and contains everything (including
the Glossary) — the standalone files are the same content broken out by
module. Use whichever is easier to pull from per section.

### Turning a source file into a post

**From a .docx:**
```bash
pandoc -t markdown "Midwest_Regional_Outpatient_Clinic_strategy.docx" -o strategy.md
```
Open `strategy.md`, clean up the formatting (pandoc's markdown is a rough
draft — headings, bullets, and bold usually need tidying), then paste the
cleaned content into a new post file following the same front-matter
pattern as Part 1.

**From an .xlsx (e.g., the Risk Register):**
Recreate the table in Markdown directly — Chirpy renders tables cleanly
with no special config needed:

```markdown
| Risk ID | Function | Likelihood | Severity | Owner |
|---|---|---|---|---|
| R-001 | Facility / All Critical Functions | Low | High | Clinic Manager |
| R-002 | EHR System Access | High | High | IT Coordinator |
```

For the Test Schedule matrix specifically, a simplified Markdown table
(Risk ID, Test Type, Responsible Party, Frequency) reads better on the web
than trying to reproduce the full month-by-month checkbox grid.

### New post naming pattern

Keep dates sequential so they read top-to-bottom as parts 1–6:

```
_posts/2026-07-13-mroc-drp-01-company-overview.md      ✅ done
_posts/2026-07-14-mroc-drp-02-critical-functions.md
_posts/2026-07-15-mroc-drp-03-risk-register.md
_posts/2026-07-16-mroc-drp-04-risk-response-strategies.md
_posts/2026-07-17-mroc-drp-05-mitigations.md
_posts/2026-07-18-mroc-drp-06-testing-maintenance.md
```

Same `categories: [Projects, Disaster Recovery]` on all six so Chirpy
groups them together in the category archive automatically.

## 3. Where your CTF writeups go

`_posts/2026-07-13-htb-forge-writeup.md` is your reusable template.
For every new writeup:

1. Duplicate the file, rename it `YYYY-MM-DD-platform-boxname-writeup.md`
2. Make a matching screenshot folder: `assets/img/posts/boxname/`
3. Update the front matter (`title`, `date`, `tags`)
4. Fill in the sections in order as you go

## 4. Folder structure reference

```
.
├── _config.yml
├── Gemfile
├── README.md
├── _tabs/
│   └── about.md
├── _posts/
│   ├── 2026-07-13-mroc-drp-01-company-overview.md
│   └── 2026-07-13-htb-forge-writeup.md
└── assets/
    └── img/
        ├── avatar.png            (add your own)
        └── posts/
            ├── mroc-drp-01/      (screenshots/diagrams for DRP Part 1, if any)
            └── htb-forge/        (screenshots for that writeup)
```
