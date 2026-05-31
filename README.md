# Advanced Deep Learning (AI4203) — Midterm Exam Results Site

Static site for students to look up their midterm exam results by entering their university ID.

## Structure

```
site/
├── index.html          ← Landing page (student ID lookup)
├── students.json       ← 41 students metadata (id, name, score, tier)
└── reports/            ← One HTML EN report per student
    ├── 443002036.html
    ├── ...
    └── 444015439.html
```

## How it works

1. Student enters their university ID on `index.html`
2. JavaScript looks them up in `students.json`
3. On success: shows summary card with grade/tier, plus a link to the full report
4. Full report is a single self-contained HTML file with base64-embedded answer images

## Deploy to GitHub Pages

1. Create a new repo or push to existing repo
2. Copy the contents of this `site/` directory to the repo root (or `docs/`)
3. Go to repo Settings → Pages → Source: `main` branch, `/` or `/docs`
4. Wait a minute, then visit `https://<your-username>.github.io/<your-repo>/`

## Rebuild

To regenerate the site after re-grading any student, run from the project root:

```bash
uv run python build_site.py
```

This copies all `Grading_Report_*_EN.html` files from `output/section*/.../` into `site/reports/{student_id}.html` and rebuilds `students.json`.

## Instructor

د. احمد بن حسن الهندي — جامعة أم القرى
