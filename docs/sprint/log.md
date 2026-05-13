# Release Notes / Sprint Log

!!! info "How this is maintained"
    Append a new entry at the top of this file on every documentation
    regeneration. The **Docs Engine** prompt instructs Claude to add
    a dated section summarising what changed in the source repository
    since the last entry.

---

## 2026-05-13 — Docs site scaffolded

- Initial MkDocs Material scaffold created.
- GitHub Actions workflow `.github/workflows/deploy.yml` added to
  auto-publish `main` to GitHub Pages on every push.
- Placeholder pages added for every nav entry (Home, User Guide,
  Technical Docs, QA, Release Notes).
- Source project: [`CE-FranchiseOwner`](https://github.com/adssconsulting/CE-FranchiseOwner)
  — Docs Engine tab in the Admin window now ships the prompt that
  regenerates these pages.
