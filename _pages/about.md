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

<div class="resume">

<p class="resume-lead">Trustworthy scientific LLM agent researcher. First author of DREAMS.</p>

<a class="resume-pdf-btn" href="{{ '/assets/html/Ziqi_Wang_Resume_Mat.pdf' | relative_url }}" target="_blank">Open full resume (PDF) ↗</a>

<h3 class="resume-section">Education</h3>

<div class="resume-entry">
  <div class="resume-date">2021 – 2026</div>
  <div class="resume-content">
    <div class="resume-title">Ph.D., Mechanical Engineering and Scientific Computing</div>
    <div class="resume-sub">Carnegie Mellon University and University of Michigan</div>
    <div class="resume-detail">Advisor: Prof. Venkatasubramanian Viswanathan</div>
  </div>
</div>

<div class="resume-entry">
  <div class="resume-date">2016 – 2021</div>
  <div class="resume-content">
    <div class="resume-title">B.S., Mechanical Engineering and B.S., Computer Science</div>
    <div class="resume-sub">University of Michigan and Florida Institute of Technology</div>
  </div>
</div>

<h3 class="resume-section">Research experience</h3>

<div class="resume-entry">
  <div class="resume-date">arXiv 2025</div>
  <div class="resume-content">
    <div class="resume-title">DREAMS <span class="resume-tag">first author</span></div>
    <div class="resume-sub">Fully autonomous LLM agent that runs expert-level materials simulations</div>
    <ul>
      <li>Reached human-expert accuracy on multi-step autonomous workflows: reproduced a long-debated surface-chemistry benchmark (CO/Pt(111)) to within 0.2% and matched expert structures on all 27 systems of a crystal benchmark, where a single-agent baseline failed every run and a multi-agent baseline missed by 389%.</li>
      <li>Canvas: a shared communication-and-memory scheme with report-based history compression for long-horizon runs; benchmarked against single-agent and multi-agent baselines on accuracy, success rate, and token cost, using up to roughly 3x fewer tokens on long-horizon tasks.</li>
      <li>Deterministic and LLM-based safety guards validate every parameter against customizable rule levels and user-defined sensitive parameters; tuned on an adversarial harness of 145 scenarios from real DFT runs, best judge at 1 false positive and 3 false negatives (n=145) across five frontier LLMs.</li>
      <li>Full traceability: every result registered with its inputs and outputs, verified by reference ID, and linked to its sources through a provenance DAG, each claim backed by an ID-annotated reasoning chain, enabling recursive debugging of long runs.</li>
    </ul>
  </div>
</div>

<div class="resume-entry">
  <div class="resume-date">In preparation</div>
  <div class="resume-content">
    <div class="resume-title">DREAMS-OER</div>
    <div class="resume-sub">Autonomous agent for open-ended catalyst discovery over a roughly 380,000-material space</div>
    <ul>
      <li>Scaled the agent from a single fixed task to open-ended search over roughly 380,000 candidate materials (Google DeepMind's GNoME set, DFT via VASP), choosing what to study next (material, surface, site, three intermediates) from accumulated evidence rather than a fixed plan. Demonstrated on behavioral runs; production screening in progress.</li>
      <li>Grounded decisions in retrieval-augmented (RAG) literature results and a multi-level relational experiment log that enables efficient information retrieval and progress tracking across hundreds of partial, interdependent studies.</li>
      <li>Gave the agent live time- and compute-budget awareness to submit work opportunistically, keep the cluster busy while reasoning, and re-plan under pressure, completing a bounded 7-hour study without overrunning.</li>
      <li>Caught workers specification-gaming a queue-occupancy target, submitting jobs to satisfy the metric without analyzing results; fixed it structurally with a disposition gate making engagement with every completed result a verifiable predicate before rest, plus a runaway-loop guard.</li>
    </ul>
  </div>
</div>

<div class="resume-entry">
  <div class="resume-date">In preparation</div>
  <div class="resume-content">
    <div class="resume-title">Alloy Thermodynamics and Phase Behavior with Machine-Learning Interatomic Potentials</div>
    <ul>
      <li>Built a committee-based active-learning loop to fine-tune universal MLIPs (MACE, NequIP, GRACE, MatterSim) into low-cost surrogates for the Li-Mg alloy system, cutting prediction error by roughly 5x versus off-the-shelf models; stress-weighted training gave the largest gains on hard second-derivative properties.</li>
      <li>Ran LAMMPS molecular dynamics melting and solidification campaigns with MLIPs and classical potentials to locate melting temperatures across composition for binary and ternary lithium alloys.</li>
      <li>Mapped solid-phase boundaries of binary alloys with canonical and semi-grand canonical Monte Carlo, using cluster-expansion models and LAMMPS with MLIPs and classical potentials.</li>
      <li>Computed free energies by thermodynamic integration to an Einstein crystal; constructed phase diagrams via thermodynamic integration, metadynamics, heat capacity, and moving-interface methods.</li>
      <li>Quantified non-linear excess effects in stiffness and atomic-transport barriers across alloy composition.</li>
    </ul>
  </div>
</div>

<h3 class="resume-section">Publications and preprints</h3>

<div class="resume-entry">
  <div class="resume-date">2025</div>
  <div class="resume-content">
    <div class="resume-detail">Z. Wang, H. Huang, H. Zhao, C. Xu, S. Zhu, J. Janssen, V. Viswanathan. "DREAMS: Density Functional Theory Based Research Engine for Agentic Materials Simulation." <a href="https://arxiv.org/abs/2507.14267">arXiv:2507.14267</a> (2025).</div>
  </div>
</div>
<div class="resume-entry">
  <div class="resume-date">2025</div>
  <div class="resume-content">
    <div class="resume-detail">A. M. Yao, S.-J. Kim, V. D. Veeraraghavan, Z. Wang, S. Gasilov, A. Kastengren, Y.-M. Chiang, P. Fenter, V. Viswanathan. "Effective Li-Ion Transport Quantification in Composite Cathodes for All-Solid-State Batteries via Multiscale Modeling and Experiments." Chem. Mater. 37, 9260-9267 (2025).</div>
  </div>
</div>

<h3 class="resume-section">Technical skills</h3>

<div class="resume-entry">
  <div class="resume-date resume-date--label">Agentic AI / LLM</div>
  <div class="resume-content"><div class="resume-detail">Multi-agent system design, harness design, SKILL design, guardrail and provenance systems, agent evaluation and benchmarking, RAG, LangGraph, LangChain</div></div>
</div>
<div class="resume-entry">
  <div class="resume-date resume-date--label">Machine learning</div>
  <div class="resume-content"><div class="resume-detail">PyTorch, machine-learning interatomic potentials (MACE, NequIP, MatterSim, GRACE, SchNet), graph neural networks, active learning</div></div>
</div>
<div class="resume-entry">
  <div class="resume-date resume-date--label">Atomistic simulation</div>
  <div class="resume-content"><div class="resume-detail">DFT (Quantum ESPRESSO, VASP), ASE, molecular dynamics (LAMMPS), Monte Carlo, cluster expansion, thermodynamic integration, nudged elastic band</div></div>
</div>
<div class="resume-entry">
  <div class="resume-date resume-date--label">Computing</div>
  <div class="resume-content"><div class="resume-detail">Python, C++, HPC / SLURM, Git</div></div>
</div>

<h3 class="resume-section">Honors and awards</h3>

<div class="resume-entry">
  <div class="resume-date">2026</div>
  <div class="resume-content"><div class="resume-detail">Selected for the Anthropic AI for Science Program</div></div>
</div>
<div class="resume-entry">
  <div class="resume-date">2019 – 2021</div>
  <div class="resume-content"><div class="resume-detail">University Honors, University of Michigan</div></div>
</div>

</div>

<style>
/* Scroll-index navigation: smooth jumps that clear the fixed navbar */
html { scroll-behavior: smooth; }
#projects, #resume { scroll-margin-top: 5rem; }

/* About-page section headers (Projects, Resume) and resume subsections,
   styled consistently. Colors come from the existing theme variables. */
#projects, #resume,
.resume-section {
  text-transform: uppercase;
  letter-spacing: 0.06em;
  font-weight: 700;
  color: var(--global-theme-color);
  border-bottom: 1px solid var(--global-divider-color);
  padding-bottom: 0.35rem;
}
#projects, #resume { font-size: 1.2rem; margin-top: 2.4rem; }
.resume-section { font-size: 1rem; margin: 1.9rem 0 1.1rem; }

/* Resume: two-column entries (date/label left, content right) */
.resume-lead { color: var(--global-text-color-light); font-style: italic; margin-bottom: 0.7rem; }
.resume-pdf-btn {
  display: inline-block; margin: 0.2rem 0 1.6rem; padding: 8px 18px;
  border: 1px solid var(--global-divider-color); border-radius: 6px;
  text-decoration: none; font-weight: 500; color: var(--global-text-color);
  transition: color .15s, border-color .15s;
}
.resume-pdf-btn:hover { border-color: var(--global-theme-color); color: var(--global-theme-color); }
.resume-entry { display: grid; grid-template-columns: 8.5rem 1fr; gap: 1.2rem; margin-bottom: 1.15rem; }
.resume-date { font-size: 0.85rem; color: var(--global-text-color-light); padding-top: 0.15rem; }
.resume-date--label { color: var(--global-text-color); font-weight: 600; }
.resume-title { font-weight: 600; color: var(--global-text-color); }
.resume-sub { font-style: italic; color: var(--global-text-color); margin-top: 0.1rem; }
.resume-detail { color: var(--global-text-color-light); }
.resume-content ul { margin: 0.5rem 0 0; padding-left: 1.1rem; }
.resume-content li { margin-bottom: 0.3rem; color: var(--global-text-color-light); }
.resume-tag {
  font-size: 0.68rem; text-transform: uppercase; letter-spacing: 0.05em; font-weight: 600;
  color: var(--global-theme-color); border: 1px solid var(--global-divider-color);
  border-radius: 4px; padding: 1px 6px; margin-left: 8px; vertical-align: middle;
}
@media (max-width: 576px) {
  .resume-entry { grid-template-columns: 1fr; gap: 0.15rem; }
  .resume-date { padding-top: 0; }
}
</style>

<script>
// Scroll-spy: highlight the navbar tab for the section currently in view,
// and let the tabs act as a scroll index on the About page.
(function () {
  var items = Array.prototype.slice.call(document.querySelectorAll('#navbar [data-scrollspy]'));
  if (!items.length) return;
  var targets = items.map(function (li) {
    var key = li.getAttribute('data-scrollspy');
    return { li: li, el: key === '_top' ? null : document.getElementById(key), top: key === '_top' };
  });
  // Only run where the sections exist (the About / home page).
  if (!targets.some(function (t) { return t.el; })) return;
  var navH = 80;
  function spy() {
    var y = window.scrollY + navH + 4;
    var active = targets[0];
    targets.forEach(function (t) {
      var top = t.top ? 0 : (t.el.getBoundingClientRect().top + window.scrollY);
      if (y >= top) active = t;
    });
    if (window.innerHeight + window.scrollY >= document.documentElement.scrollHeight - 2) {
      active = targets[targets.length - 1];
    }
    targets.forEach(function (t) { t.li.classList.toggle('active', t === active); });
  }
  window.addEventListener('scroll', spy, { passive: true });
  window.addEventListener('resize', spy);
  spy();
})();
</script>
