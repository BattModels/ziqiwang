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
2. **Project: DREAMS** (results page, decided 2026-07-06): architecture, canvas communication, Sol27LC (27/27, <1% average error), CO/Pt(111) (0.2%, self-correction), BEEF-vdW uncertainty, framework benchmark vs MDCrow/ChemGraph. Paper figures live in `assets/img/dreams/` as fig1-fig6 (SITE numbering per Ziqi's files; maps to updated paper figs 2,7,3,4,5,6). Tables 1-5 on site = paper tables 3-7. Reference paper: `tmp-images/DREAMS__Density_Functional_Theory_Based_Research_Engine_for_Agentic_Materials_Simulation.pdf` (updated over the arXiv v1).
3. **Project: Transparency, Provenance, and Trustworthiness** (`_projects/trustworthiness.md`): purely provenance/verification; hosts the three interactives (provenance DAG, safety-guard verification report, token usage explorer). The "up to 3x token reduction" headline is grounded in the paper: DREAMS used 2-3x fewer tokens than ChemGraph on CO/Pt(111) (paper Table 7 section).
4. **Project: DREAMS-OER** (`_projects/dreams-oer.md`, built 2026-07-06 from `tmp-images/dreams oer claude guide.pptx`): autonomous OER catalyst screening over ~381k GNoME candidates. Story is agent-durability over a 7-hour, hundreds-of-jobs campaign (six requirements from deck slide 7), not just screening results. Sections: problem/pathway, why-hard, workflow, relational experiment log, enforced reasoning, time/resource awareness, safety guards + judge, traceable execution (449-node OER DAG screenshot), early analysis (volcano, scaling, UMAPs, element occurrence), acknowledged limitations, phase-2 live-panel placeholder. Figures in `assets/img/dreams-oer/` extracted from the pptx (low-res UMAPs are placeholders; Ziqi supplies hi-res in phase 2). Slide-1 electrolysis GIF is 3rd-party (Wu et al. 2021), excluded. Deck numbers are all Ziqi-sanctioned. OER token-analysis HTML still pending upload to `assets/html/`. Every screening result carries the "production screening in progress" caveat.
5. **Project: MLIP**: machine-learned interatomic potential training and benchmarking for alloy systems (shorter page). Not built yet; no project tile until it exists.

Structural change 2026-07-06 (consolidated onto About; supersedes the separate-pages plan above):
- **Nav is About only.** Everything lives on the About page (`_pages/about.md`): intro + headline results, project tiles (looped from `_projects` via the `projects.liquid` card include, ordered by `importance`), and an inline Resume section.
- **Project tiles** use placeholder thumbnails in `assets/img/thumbnails/{dreams,trustworthiness,dreams-oer}.png` (generated with matplotlib; Ziqi replaces with real images later). Each project's `img:` front-matter points at its placeholder.
- **Publications page removed** (was empty; resume covers publications). `_bibliography/papers.bib` stays empty.
- **CV page removed** (`_pages/cv.md` deleted; no more `al_folio_cv`/RenderCV). The resume is the PDF at `assets/html/Ziqi_Wang_Resume_2026.pdf`, opened via a button on the About Resume section (target=_blank, browser renders it), plus an inline text resume transcribed from that PDF. When the PDF is updated, re-transcribe the inline section to match.
- The old projects index (`_pages/projects.md`) still exists but is `nav: false`.

## Narrative spine

Every DREAMS-related page follows this arc, in this order: materials discovery needs DFT → DFT is hard to automate → DREAMS automates it → correct answers can hide fabricated reasoning → provenance and verification fix that. The safety guard and judge content is the differentiator; frame it as "verification systems that catch agent fabrication," legible to any AI hiring manager regardless of materials background.

## Layered disclosure pattern (mandatory for every visualization)

Each embedded figure or interactive asset gets, in order:
1. A one-sentence claim stating what the reader should conclude.
2. A short plain-language paragraph (no jargon a non-specialist would not know).
3. The interactive or static asset itself.
4. A collapsible "technical details" section or link for domain experts.

## Headline numbers (use these; do not invent or round others)

Refined wording provided by Ziqi 2026-07-06 (supersedes the older bare numbers):

- 0.2% error on adsorption-energy differences between preferred sites, using the same exchange-correlation functional (CO/Pt(111)). Not "0.2% accuracy".
- Below 1% error on lattice-constant calculations across 27 elemental bulk materials spanning 3 crystal structures (Sol27LC). This is the accurate framing of the old "27/27".
- Up to 3x token reduction for the lightweight DREAMS framework compared to baseline frameworks.
- Worker error rate and judge success rate are currently under benchmarking (state as in progress, no number yet).
- DREAMS-OER catalyst exploration is currently running; result analysis is preliminary.
- 5x MLIP error reduction still holds for the MLIP page (roughly 5x lower prediction error vs off-the-shelf models, Li-Mg alloy system), but is not on the About headline list.

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

## Live-update architecture (decided 2026-07-06)

GitHub Pages is static and the HPC live view is not publicly reachable, so phase 2 "near-live" updates use snapshot-push: a cron job on the cluster exports JSON snapshots and figures, pushes them to this repo, the GitHub Action rebuilds, and page JavaScript fetch()es the JSON at view time. No separately hosted backend. Phase 1's "split large embedded data into JSON + fetch()" rule exists to prepare this.

## Roadmap (one session each; do not run ahead)

Session 1 (done 2026-07-06): git identity check, local preview via ruby/3.3.7 module, baseurl/url fix, demo content purge, first push.

**Session 2: identity + About page.**
- Fill `_config.yml` identity fields: name, description, GitHub, LinkedIn, Google Scholar, email. Ask Ziqi for each handle; do not guess.
- About page: positioning statement plus the four headline numbers from this file, visible without scrolling. Positioning to draft against: PhD candidate at the intersection of computational materials science and agentic AI systems; builds multi-agent systems for autonomous scientific computing with verification that catches agent fabrication; selected for Anthropic's AI for Science program; targeting applied research scientist roles. Draft, show Ziqi, iterate. Style rules in this file are strict.
- Disable unused sections (teaching, books, blog/news) in config unless Ziqi says otherwise. (Ziqi decided 2026-07-06: hide everything except About, Projects, Publications, CV; no profile photo for now.)

**Session 3: DREAMS project page.**
- Follow the narrative spine and layered disclosure pattern from this file for every asset.
- Assets to embed (Ziqi must copy them into `assets/html/` from elsewhere on the cluster; ask for paths): provenance DAG HTML (from dag_visualizer.py, vis-network), verification/safety-guard DAG HTML (from verify_dag_visualizer.py), judge token analysis HTML (Plotly, from build_token_html.py).
- Check each file's size before committing (100 MB hard limit; large embedded data should be split to JSON + fetch()).
- Prose covers: hierarchical planning supervisor + DFT/HPC worker agents; canvas (shared provenance-tracked memory); report-judge agent; value-flow graph; safety guard catching fabrication (e.g., identity-math expressions); claim-to-source coherence checks. Approved terminology: canvas, report-judge agent, value-flow graph, BEEF-vdW, Sol27LC, CO/Pt(111) puzzle. The safety guard/judge material is the page's centerpiece per this file.
- Paper link: arXiv:2507.14267, Ziqi first author.

**Session 4: DREAMS-OER project page.**
- Assets: exploration-range figures, UMAP analysis (featurize_umap.py outputs; UMAP fit on full design space with promising subset projected via .transform()), coverage figures, OER token analysis HTML.
- Every screening result carries the "production screening in progress" caveat per this file.
- Context for prose: screening ~381,000 GNoME candidates; disposition gate preventing specification gaming (agents submitting jobs to satisfy queue metrics without analyzing results).
- Leave an explicit placeholder slot (a marked section, no code) for the phase 2 near-live status panel.

**Session 5: MLIP page + publications + CV.**
- MLIP page (short): MLIP fine-tuning and benchmarking for alloy systems (broader than lithium; see framing rules), 5x error reduction headline, thermal-expansion validation of BCC Li (100-400 K) and HCP Mg (100-700 K) across MACE and GRACE model families. Ask Ziqi which figures to include; do not fabricate findings.
- Publications: Ziqi supplies BibTeX for `_bibliography/papers.bib`; ask for it.
- CV: `layout: cv` via al_folio_cv gem (RenderCV YAML or JSONResume). Content mirrors the one-page resume; ask Ziqi to paste resume content rather than reconstructing it.

**Session 6: polish + review packet.**
- Cross-page consistency pass against the style rules in this file.
- Verify all iframes load, all layers of the disclosure pattern present on every asset.
- The 60-second test: a recruiter reading only the About headline numbers and each asset's layer-1 claim sentence must get the complete story without opening any technical details.
- Produce a short review checklist for Venkat listing every dataset, figure, and candidate-level result exposed, so hide/keep decisions are easy. After approval, making the repo public is what enables Pages (free plan); verify the production build then.
