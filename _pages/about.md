---
layout: about
title: about
permalink: /
subtitle: PhD candidate, Mechanical Engineering and Scientific Computing, University of Michigan

profile:
  align: right
  image:
  image_circular: false # crops the image to make it circular
  more_info:

selected_papers: false # includes a list of papers marked as "selected={true}"; re-enable once papers.bib has entries
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

I work at the intersection of computational materials science and agentic AI systems. I build multi-agent systems that run the physics simulations behind materials discovery autonomously on supercomputers. Because a correct answer can hide fabricated reasoning, I also build the verification systems that catch agents fabricating results. I was selected for Anthropic's AI for Science program. I am seeking applied research scientist roles and graduate in summer 2026.

**Headline results**

- **0.2% error** on adsorption-energy differences between preferred sites, using the same exchange-correlation functional (CO/Pt(111))
- **Below 1% error** on lattice-constant calculations across 27 elemental bulk materials spanning 3 crystal structures (Sol27LC)
- **Up to 3x token reduction** for the lightweight DREAMS framework compared to baseline frameworks
- Worker error rate and judge success rate are currently under benchmarking
- DREAMS-OER catalyst exploration is currently running; result analysis is preliminary

## Projects

<div class="projects">
  <div class="row row-cols-1 row-cols-md-2">
  {% assign sorted_projects = site.projects | sort: "importance" %}
  {% for project in sorted_projects %}
    {% include projects.liquid %}
  {% endfor %}
  </div>
</div>

## Resume

Trustworthy scientific LLM agent researcher. First author of DREAMS.

<a href="{{ '/assets/html/Ziqi_Wang_Resume_2026.pdf' | relative_url }}" target="_blank" style="display:inline-block; margin:0.4rem 0 1rem; padding:8px 18px; border:1px solid var(--global-divider-color); border-radius:6px; text-decoration:none; font-weight:500;">Open full resume (PDF) ↗</a>

**Education**

- **Ph.D., Mechanical Engineering and Scientific Computing**, Carnegie Mellon University and University of Michigan, 2021 to 2026 (expected). Advisor: Prof. Venkatasubramanian Viswanathan.
- **B.S., Mechanical Engineering** and **B.S., Computer Science**, University of Michigan and Florida Institute of Technology, 2016 to 2021.

**Research experience**

**DREAMS** (first author, arXiv 2025). Fully autonomous LLM agent that runs expert-level materials simulations.

- Reached human-expert accuracy on multi-step autonomous workflows: reproduced a long-debated surface-chemistry benchmark (CO/Pt(111)) to within 0.2% and matched expert structures on all 27 systems of a crystal benchmark, where a single-agent baseline failed every run and a multi-agent baseline missed by 389%.
- Canvas: a shared communication-and-memory scheme for long-horizon, complex runs, with report-based history compression suited to scientific workflows, giving finer control over agent context than general-purpose agent frameworks.
- Deterministic and LLM-based safety guards that vouch for each result's credibility, validating every parameter setting against customizable rule levels (R1, R2, ...) and user-defined sensitive parameters.
- Full traceability and white-box transparency: every result is registered with its inputs and outputs, verified by reference ID (ref_id), and linked to its sources through a provenance DAG, with each claim backed by a context-aware reasoning chain annotated with those IDs.
- Enables recursive, deep debugging of long, multi-step runs.
- Benchmarked against representative single-agent and multi-agent frameworks, reporting accuracy, success rate, and token usage (cost), using up to roughly 3x fewer tokens than the baseline framework on long-horizon tasks.

**DREAMS-OER** (manuscript in preparation). Autonomous agent for open-ended catalyst discovery over a roughly 380,000-material space.

- Scaled the agent from a single fixed task to open-ended search over roughly 380,000 candidate materials (Google DeepMind's GNoME set), choosing what to study next (material, surface, site, three intermediates) from accumulated evidence rather than a fixed plan. Demonstrated on behavioral runs; production screening in progress.
- Built a multi-level relational experiment log, enabling efficient information retrieval and progress tracking across hundreds of partial, interdependent studies.
- Gave the agent live time- and compute-budget awareness to submit work opportunistically, keep the cluster busy while reasoning, and re-plan under pressure, completing a bounded 7-hour study without overrunning.
- Hardened it against scale-only failure modes: a disposition gate enforcing genuine engagement with each result (removing a queue-occupancy gaming pattern), a runaway-loop guard, retrieval-augmented literature grounding, and required hypothesis and limitation logging, including a "too credulous" failure now being addressed.

**Non-linear Material-Property Prediction with Machine-Learning Interatomic Potentials** (manuscript in preparation).

- Built a committee-based active-learning loop to fine-tune several universal ML interatomic potentials (MACE, NequIP, GRACE, MatterSim) into accurate, low-cost surrogates for the Li-Mg alloy system, cutting prediction error on target physical properties by roughly 5x versus off-the-shelf models.
- Showed that weighting training toward higher-order (stress) information, not just energy, most improves accuracy on hard second-derivative properties, with the gain transferring across multiple properties.
- Mapped how alloying produces non-linear excess effects in stiffness and atomic-transport barriers across composition, quantifying behavior that cannot be inferred from the pure end-members alone.

**Publications and preprints**

- Z. Wang, H. Huang, H. Zhao, C. Xu, S. Zhu, J. Janssen, V. Viswanathan. "DREAMS: Density Functional Theory Based Research Engine for Agentic Materials Simulation." [arXiv:2507.14267](https://arxiv.org/abs/2507.14267) (2025).

**Technical skills**

- **Agentic AI / LLM**: multi-agent system design, harness design, SKILL design, guardrail and provenance systems, agent evaluation and benchmarking, LangGraph, LangChain.
- **Machine learning**: PyTorch, machine-learning interatomic potentials (MACE, NequIP, MatterSim, GRACE, SchNet), graph neural networks, active learning.
- **Atomistic simulation**: DFT (Quantum ESPRESSO), ASE, molecular dynamics (LAMMPS), Monte Carlo, nudged elastic band, Bayesian error estimation (BEEF).
- **Computing**: Python, C++, HPC / SLURM, Git.

**Honors and awards**

- Selected for the Anthropic AI for Science Program (2026).
- University Honors, University of Michigan (2019, 2020, 2021).
