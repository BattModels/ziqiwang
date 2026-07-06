---
layout: page
title: DREAMS
description: a multi-agent AI system that runs the physics simulations behind materials discovery, with verification that catches agent fabrication
importance: 1
related_publications: false
---

New materials for batteries and catalysts are screened by simulation before anyone synthesizes them. The workhorse method is density functional theory (DFT). DFT is hard to automate: each simulation needs settings that experts tune by hand, and one study can require hundreds of runs.

DREAMS automates this work with a team of AI agents. A hierarchical planning supervisor decomposes a research question into simulation tasks. DFT and HPC worker agents execute those tasks on a supercomputer. Every intermediate result is written to the canvas, a shared memory that records where each value came from.

Two results establish that the automation works: DREAMS reproduced 27 of 27 crystal structures on the Sol27LC benchmark, and reached 0.2% accuracy on the CO/Pt(111) puzzle. The framework is described in the DREAMS paper ([arXiv:2507.14267](https://arxiv.org/abs/2507.14267)), where I am first author.

Automation creates a second problem: an agent can report a correct-looking answer built on reasoning it never performed. The rest of this page covers the systems that make agent reasoning inspectable and the guard that blocks fabrication. These provenance and anti-fabrication systems are current development, not part of the published paper.

## Provenance: every result has a history

**Every value DREAMS reports can be traced backward to the simulations that produced it.**

The graph below is the provenance record of one DREAMS run, drawn from the canvas. Each node is a task, a message, or a result. Each arrow shows what was derived from what. Drag to pan and scroll to zoom.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_provenance_dag.html' | relative_url }}" frameborder="0" height="620px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_provenance_dag.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The graph is a snapshot of the canvas provenance DAG at step 29 of a run, exported by `dag_visualizer.py` and rendered with vis-network. Graph data is embedded in the HTML; no server is involved after page load.
{% enddetails %}

## Verification: the agent cannot fabricate results

**Every claim in the report below was checked against the simulation record that produced it.**

A plausible report is not evidence of performed analysis. Language-model agents can emit a number that looks right while the calculation behind it was skipped or invented. One observed failure mode is the identity-math expression: a calculation that returns its input unchanged while reading as analysis.

DREAMS counters this with two mechanisms. The safety guard inspects the expressions agents execute and blocks fabrication, so the agent cannot fabricate results. The report-judge agent then checks the final report claim by claim: each claim must trace to a source in the canvas, following a value-flow graph from reported numbers back to raw simulation outputs.

Below is the verification view for one DREAMS report, the lattice constant of body-centered cubic lithium. The graph is the report's value-flow graph. Click a node to inspect it, open the Claims tab to see each checked claim, and the per-parameter checks show what was verified.

<div class="l-page">
  <iframe src="{{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}" frameborder="0" height="720px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The view is exported by `verify_dag_visualizer.py`. The page loads its verification state from a separate JSON file at view time, the same fetch pattern planned for near-live status updates on this site. Tabs expose node detail, the claim list with claim-to-source coherence checks, and the event log of the verification run.
{% enddetails %}

## Cost: measuring agent token usage

**Agent token usage in DREAMS was reduced by up to 3x.**

Tokens are the unit of work and cost for language-model agents. The explorer below breaks down the token usage of a DREAMS run by tool and by step, separately for input and output tokens. This measurement is how the reductions were identified and verified.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_judge_token_analysis.html' | relative_url }}" frameborder="0" height="650px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_judge_token_analysis.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The explorer is exported by `build_token_html.py` and rendered with Plotly. Charts cover per-tool input and output token totals and per-step usage over the run. Data is embedded in the HTML.
{% enddetails %}
