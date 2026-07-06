# CLAUDE.md

## What this repo is

Personal academic-to-industry website for Ziqi, PhD candidate in Mechanical Engineering and Scientific Computing (University of Michigan / CMU), graduating summer 2026. Built on the al-folio v1.x Jekyll starter, hosted on GitHub Pages. Primary purpose: support a job search for applied research scientist roles at big tech companies, with a hard deadline of October 15, 2026. The primary reader is a hiring manager or recruiter with 60 to 90 seconds of attention, not a domain expert.

This repo was created from the al-folio template. `CLAUDE.upstream.md`, `AGENTS.md`, `.github/copilot-instructions.md`, and `docs/BOUNDARIES.md` are upstream maintainer docs; consult them for framework internals only. Their PR-routing rules, sibling-repo paths (`~/Documents/dev/al-org/`), CI style contracts, and Docker workflow do not apply here.

## Theme architecture (al-folio v1, read before editing anything)

- This repo is a thin starter. It owns only config, content (`_pages`, `_posts`, `_projects`, `_news`, `_bibliography`), and assets. All layouts, includes, Sass, Liquid tags/filters, and feature JS live in versioned gems (`al_folio_core` is the hub; feature gems include `al_charts`, `al_img_tools`, `al_search`, `al_folio_cv`, etc.).
- Do not go looking for `_layouts/` or `_includes/` files to edit; they are not in this repo. Customize via `_config.yml` and page front matter first.
- If a gem-owned file must be shadowed locally, register the override with `bundle exec al-folio upgrade overrides diff <path>` then `overrides accept <path>`, and commit `.al-folio-overrides.yml`. Never shadow silently.
- Features fail silently: a feature renders only when its gem is in BOTH the `Gemfile` and `_config.yml`'s `plugins:` list AND its config flag or front-matter key is on. A missing gem or flag emits an empty string, no error. When an embed or feature does not appear, check this chain first.
- `Gemfile` (pinned versions) and `_config.yml` `plugins:` must stay in sync; adding or removing a plugin means editing both.
- Do not remove the v1 config contract keys (`al_folio.api_version`, `style_engine`, `tailwind.*`, `distill.*`); builds warn and the upgrade audit blocks on them.
- baseurl: this is a user site (`<username>.github.io`), so `baseurl` in `_config.yml` must be EMPTY. The upstream docs' `localhost:4000/al-folio/` path applies to the demo repo only. If assets 404 locally or in production, check baseurl first.
- `al_charts` bundles Plotly (plus Mermaid, Chart.js, ECharts, Vega) gated by per-page `chart.*` front matter. Known upgrade path for token-analysis plots; phase 1 still uses iframes (see Technical constraints).

## Site structure (do not add pages beyond these without asking)

1. **About (landing page)**: positioning statement plus headline numbers, visible without scrolling.
2. **Project: DREAMS**: multi-agent DFT framework, safety guard, judge, provenance DAG, token analysis.
3. **Project: DREAMS-OER**: autonomous catalyst screening (~381k GNoME candidates), exploration analysis, UMAP, coverage figures. A near-live status panel is planned for phase 2; do not build it yet, but do not design against it.
4. **Project: MLIP**: machine-learned interatomic potential training and benchmarking for alloy systems (shorter page).
5. **Publications**: generated from `_bibliography/papers.bib`. DREAMS paper is arXiv:2507.14267.
6. **CV**: uses the `al_folio_cv` gem (`layout: cv`, RenderCV YAML or JSONResume), mirroring the one-page resume.

## Narrative spine

Every DREAMS-related page follows this arc, in this order: materials discovery needs DFT → DFT is hard to automate → DREAMS automates it → correct answers can hide fabricated reasoning → provenance and verification fix that. The safety guard and judge content is the differentiator; frame it as "verification systems that catch agent fabrication," legible to any AI hiring manager regardless of materials background.

## Layered disclosure pattern (mandatory for every visualization)

Each embedded figure or interactive asset gets, in order:
1. A one-sentence claim stating what the reader should conclude.
2. A short plain-language paragraph (no jargon a non-specialist would not know).
3. The interactive or static asset itself.
4. A collapsible "technical details" section or link for domain experts.

## Headline numbers (use these; do not invent or round others)

- 0.2% accuracy on CO/Pt(111)
- 27/27 crystal structures (Sol27LC)
- Up to 3x token reduction
- 5x MLIP error reduction

## Factual framing rules (these correct past drafting errors; follow strictly)

- Provenance and anti-fabrication features are current development, not shipped product.
- MLIP work covers broader alloy systems, not only lithium.
- DREAMS-OER production screening is still in progress; state this explicitly wherever screening results appear.
- The agent "cannot fabricate results" is the approved framing for the safety guard.
- Do not merge unrelated content into the same sentence.

## Writing style (strict)

- Formal register, terse, technically precise.
- No em-dashes.
- Never use the words "natural" or "precisely."
- No inflated verbs (e.g., "revolutionize," "unlock," "supercharge").
- No vague qualitative claims; every claim of improvement carries a number or is cut.
- No jargon inaccessible to non-specialists in layers 1 and 2 of the disclosure pattern; jargon is acceptable inside "technical details" only.

## Technical constraints

- Development happens on an HPC cluster login node over VS Code Remote-SSH. No Docker; no root access. Do not suggest Docker or sudo.
- Local preview, in priority order:
  1. User-space Ruby if available (`module avail ruby`, then `bundle config set --local path vendor/bundle`, `bundle install`, `bundle exec jekyll serve --livereload --port <high uncommon port>`). VS Code auto-forwards the port. Remember baseurl is empty, so pages serve at the root path.
  2. Apptainer with al-folio's Docker image (`apptainer pull docker://amirpourmand/al-folio:latest`), bind-mounting the repo; set GEM_HOME to a writable bound path if gem paths conflict. Note the upstream image serves with the demo baseurl; override the serve command.
  3. No preview: push to a branch and let the GitHub Action build. Acceptable for content edits only.
- Keep `vendor/`, `.sif` files, and Apptainer caches out of git (add to .gitignore).
- The Jekyll server is a long-lived process on a shared login node; keep it lightweight, stop it when done, and pick a nonstandard high port.
- Before the first commit, verify git identity in this repo uses Ziqi's personal email, not a cluster or institutional default (`git config user.email`).
- Interactive assets (Plotly token analysis, vis-network DAG HTML, UMAP plots) live in `assets/html/` and are embedded via iframe or linked as standalone pages. Use them as-is for phase 1; do not regenerate or restyle them. Migrating Plotly plots to `al_charts` front-matter rendering is a possible later improvement, not phase 1. The "Live Preview" VS Code extension can render standalone HTML files directly for quick checks; it cannot render the Jekyll site.
- Watch file sizes: GitHub hard limit is 100 MB per file. If an asset with embedded data is large, split the data into a separate JSON loaded client-side with fetch(); this also prepares the phase 2 snapshot pipeline.
- Deployment is the al-folio GitHub Action on push to main; never add a manual deploy step or use `bin/deploy`.

## Privacy

The repo stays private until Ziqi's advisor (Venkat) reviews the content, especially candidate-level DREAMS-OER results. All data is included for now; hiding decisions come from that review. Never make the repo public or enable public Pages without an explicit instruction from Ziqi.

## Working norms

- Ask before structural changes (new pages, nav changes, theme swaps).
- Commit in small, described steps; never force-push.
- When adding content about results, if a number or claim is not in this file or provided by Ziqi in the session, ask rather than infer.
- This is a shared login node: no long-running background processes beyond the preview server, no heavy builds.
