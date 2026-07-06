---
layout: page
title: Transparency, Provenance, and Trustworthiness
description: provenance records, fabrication-proof verification, and cost accounting that make agent reasoning inspectable
importance: 2
related_publications: false
---

A plausible report is not evidence of performed analysis. Language-model agents can emit a number that looks right while the calculation behind it was skipped or invented. For [DREAMS]({{ '/projects/dreams/' | relative_url }}), which runs materials simulations autonomously, that failure mode is disqualifying: a fabricated result that survives into a report wastes experiments and erodes trust in everything else the system produced.

This page covers the systems that address it: provenance records that give every value a traceable history, a safety guard and report judge that check every claim against that history, and the token accounting that shows what verification costs. These provenance and anti-fabrication systems are current development, not part of the published paper.

## Provenance: every result has a history

**Every value DREAMS reports can be traced backward to the simulations that produced it.**

The graph below is the provenance record of one DREAMS run, drawn from the canvas, the shared memory where agents record every intermediate result. Each node is a task, a message, or a result. Each arrow shows what was derived from what. The two green nodes are the run's starting inputs; the two gold nodes are the final lattice-constant calculation and the report built from it. Drag to pan and scroll to zoom; the Fit button shows the whole graph.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_provenance_dag.html' | relative_url }}" frameborder="0" height="620px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_provenance_dag.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The graph is a snapshot of the canvas provenance DAG at step 29 of a run, exported by `dag_visualizer.py` and rendered with vis-network. Graph data is embedded in the HTML; no server is involved after page load.
{% enddetails %}

## Verification: the agent cannot fabricate results

**Every claim in the report below was checked against the simulation record that produced it.**

One observed failure mode is the identity-math expression: a calculation that returns its input unchanged while reading as analysis. DREAMS counters this with two mechanisms. The safety guard inspects the expressions agents execute and blocks fabrication, so the agent cannot fabricate results. The report-judge agent then checks the final report claim by claim: each claim must trace to a source in the canvas, following a value-flow graph from reported numbers back to raw simulation outputs.

Below is the verification view for one DREAMS report, the lattice constant of body-centered cubic lithium. The graph is the report's value-flow graph, with the final calculation pre-selected. Click any node to inspect it, open the Claims tab to see each checked claim, and the per-parameter checks show what was verified.

<div class="l-page">
  <iframe src="{{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}" frameborder="0" height="720px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The view is exported by `verify_dag_visualizer.py`. The page loads its verification state from a separate JSON file at view time, the same fetch pattern planned for near-live status updates on this site. Tabs expose node detail, the claim list with claim-to-source coherence checks, and the event log of the verification run.
{% enddetails %}

## Cost: what running and verifying agents costs

**Token accounting per tool and per step shows where an agent run spends its budget, including the judge's share.**

Tokens are the unit of work and cost for language-model agents. The explorer below breaks down one DREAMS run by tool and by step, separately for input and output tokens, with the judge tracked as its own trace. In the run shown, the judge used 515,887 of 4,990,911 total tokens. The same accounting underlies the published efficiency comparison: on the CO/Pt(111) challenge, DREAMS used 2 to 3x fewer tokens than the strongest baseline framework while reaching a correct result.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_judge_token_analysis.html' | relative_url }}" frameborder="0" height="1000px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_judge_token_analysis.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The explorer is exported by `build_token_html.py` and rendered with Plotly. Charts cover per-tool input and output token totals and per-step cumulative usage, with a slider to walk through the session step by step. Data is embedded in the HTML.
{% enddetails %}
