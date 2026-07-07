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

<a href="{{ '/assets/html/Ziqi_Wang_Resume_2026.pdf' | relative_url }}" target="_blank" style="display:inline-block; margin-bottom:1rem; padding:8px 18px; border:1px solid var(--global-divider-color); border-radius:6px; text-decoration:none; font-weight:500;">Open full resume (PDF) &nearr;</a>

**Education**

- **Ph.D., Mechanical Engineering and Scientific Computing**, Carnegie Mellon University and University of Michigan, 2021 to 2026 (expected). Advisor: Prof. Venkatasubramanian Viswanathan.
- **B.S., Mechanical Engineering** and **B.S., Computer Science**, University of Michigan and Florida Institute of Technology, 2016 to 2021.

**Research experience**

- **DREAMS** (first author, arXiv 2025): a fully autonomous LLM agent that runs expert-level materials simulations, with a shared communication-and-memory canvas, safety guards that vouch for each result, and full provenance-based traceability.
- **DREAMS-OER** (manuscript in preparation): an autonomous agent for open-ended catalyst discovery over a design space of roughly 380,000 materials, with a relational experiment log, time and compute budget awareness, and hardening against scale-only failure modes.
- **Machine-learned interatomic potentials** (manuscript in preparation): a committee-based active-learning loop that fine-tunes universal ML potentials (MACE, NequIP, GRACE, MatterSim) into low-cost surrogates for the Li-Mg alloy system, cutting prediction error on target physical properties by roughly 5x.

**Publications**

- Z. Wang, H. Huang, H. Zhao, C. Xu, S. Zhu, J. Janssen, V. Viswanathan. "DREAMS: Density Functional Theory Based Research Engine for Agentic Materials Simulation." [arXiv:2507.14267](https://arxiv.org/abs/2507.14267) (2025).

**Technical skills**

- **Agentic AI / LLM**: multi-agent system design, harness design, guardrail and provenance systems, agent evaluation and benchmarking, LangGraph, LangChain.
- **Machine learning**: PyTorch, machine-learned interatomic potentials (MACE, NequIP, MatterSim, GRACE, SchNet), graph neural networks, active learning.
- **Atomistic simulation**: DFT (Quantum ESPRESSO), ASE, molecular dynamics (LAMMPS), Monte Carlo, nudged elastic band, Bayesian error estimation (BEEF).
- **Computing**: Python, C++, HPC / SLURM, Git.

**Honors and awards**

- Selected for the Anthropic AI for Science Program (2026).
- University Honors, University of Michigan (2019, 2020, 2021).
