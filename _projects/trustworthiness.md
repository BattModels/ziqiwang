---
layout: page
title: Transparency, Provenance, and Trustworthiness
description: provenance records, fabrication-proof verification, and cost accounting that make agent reasoning inspectable
importance: 2
img: assets/img/thumbnails/trustworthiness.png
related_publications: false
---

A plausible report is not evidence of performed analysis. Left unguarded, a language-model agent fails in specific ways: it writes a final number out of nowhere, invents an intermediate input value, derives a result in its own head instead of running the tool, ignores a tool that returned an error and reports success anyway, or reuses an earlier result after it has stopped being valid. For [DREAMS]({{ '/projects/dreams/' | relative_url }}), which runs materials simulations autonomously, any of these is disqualifying: a fabricated value that reaches a report wastes experiments and discredits everything else the system produced.

This page covers the systems that prevent it: every value carries the reasoning and references needed to check it, only tool outputs become citable results, and a report judge re-verifies every reported number against its source. These provenance and anti-fabrication systems are current development, not part of the published paper.

## Provenance: every result has a history

**Every value DREAMS reports can be traced backward to the simulations that produced it.**

The graph below is the provenance record of one DREAMS run, drawn from the canvas, the shared memory where agents record every intermediate result. Each node is a task, a message, or a result. Each arrow shows what was derived from what. Only a result produced by a tool becomes an artifact with a unique reference id, and each artifact records the tool that made it, its full arguments, its output, a description, the context of the call, and the reasoning behind every parameter. An agent cannot mint a reference by hand; only running a tool creates one. The two green nodes are the run's starting inputs; the two gold nodes are the final lattice-constant calculation and the report built from it. Drag to pan and scroll to zoom; the Fit button shows the whole graph.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_provenance_dag.html' | relative_url }}" frameborder="0" height="620px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_provenance_dag.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The graph is a snapshot of the canvas provenance DAG at step 29 of a run, exported by `dag_visualizer.py` and rendered with vis-network. Graph data is embedded in the HTML; no server is involved after page load.
{% enddetails %}

## Enforced reasoning and context: no parameter can be set without a stated why, under a stated context

**No value the agent commits is a bare number. Every tool call carries a per-parameter reason and a context, and any sensitive parameter must cite the earlier result it came from.**

Provenance records where a value came from; enforced reasoning records why. Every tool call must fill two fields, which the agent's tools require by their type signature rather than by request.

<div class="reason-card">
  <div class="reason-field">
    <div class="reason-name">reasons <span class="reason-type">Dict[str, str]</span></div>
    <div class="reason-desc">Per-parameter rationale. For each parameter, two to three sentences covering:</div>
    <ul class="reason-list">
      <li><span class="reason-key">the role</span> it plays in the current study: being varied now, fixed at a converged value from a prior convergence test, inherited from an upstream tool, or held fixed by design while a sub-study is in progress.</li>
      <li><span class="reason-key">the value</span>: how it was chosen, what evidence supports it, and the expected effect on the output.</li>
    </ul>
  </div>
  <div class="reason-field">
    <div class="reason-name">context <span class="reason-type">str</span></div>
    <div class="reason-desc">One to two sentences naming which study or exploration this call is part of (a convergence test for ecutwfc, a production run for adsorption energy, a sensitivity sweep, a one-off check) and why the tool is being called.</div>
  </div>
</div>

On top of these, a human-defined set of sensitive parameters, the ones that actually move the answer, must be set with a reference to the earlier result that produced the value. That reference is checked against both the existence of the source and the equality of the value. An agent can forget to record a reason and be stopped before it proceeds; it cannot invent one that passes the check.

## Verification: the agent cannot fabricate results

**Every claim in the report below was checked against the simulation record that produced it, parameter by parameter, by a separate judge agent.**

Two things make a report checkable. Every claimed number must carry a reference to the tool result that produced it, and the supervisor names in advance which results must be claimed, so the agent cannot quietly drop an inconvenient one. Numbers read from raw output files go through guarded math and extraction tools that require an evidence snippet and check the value against it, which stops the agent from computing or rounding a number in its head.

Once a report exists, a judge agent re-verifies it recursively, one parameter at a time. For each sensitive parameter it first confirms, deterministically, that the cited source exists and its value matches. Then it weighs whether that source is a sensible origin at all: a reference is rejected when it is cross-wired (for example a k-point setting pinned to an energy-cutoff convergence result), when a production run leaves a sensitive parameter with no source, or when a rationale claims a sub-study the context does not support. The judge is deliberately strict; in development it flagged real mistakes, including a value referenced to an input file instead of the output file that computed it. Its own findings become a judging report, which is itself verifiable.

Below is the verification view for one DREAMS report, the lattice constant of body-centered cubic lithium. The graph is the report's value-flow graph, with the final calculation pre-selected. Click any node to inspect it, open the Claims tab to see each checked claim, and the per-parameter checks show what was verified.

<div class="l-page">
  <iframe src="{{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}" frameborder="0" height="720px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The view is exported by `verify_dag_visualizer.py`. The page loads its verification state from a separate JSON file at view time, the same fetch pattern planned for near-live status updates on this site. The judge classifies each sensitive parameter by its role (referenced from an upstream result, held without a source, or intentionally varied) and applies a matching rule to each, so an atypical value is accepted when the recorded context justifies it and rejected when it does not. Tabs expose node detail, the claim list with claim-to-source coherence checks, and the event log of the verification run.
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

<style>
/* Reason/context schema card in the enforced-reasoning section.
   Colors come from the existing theme variables so it tracks light/dark. */
.reason-card {
  border: 1px solid var(--global-divider-color);
  border-radius: 8px;
  overflow: hidden;
  margin: 1.2rem 0;
  background: var(--global-card-bg-color);
}
.reason-field { padding: 16px 18px; }
.reason-field + .reason-field { border-top: 1px solid var(--global-divider-color); }
.reason-name {
  font-family: var(--global-code-font, monospace);
  font-weight: 700;
  color: var(--global-theme-color);
  font-size: 1rem;
  margin-bottom: 0.35rem;
}
.reason-type {
  font-family: var(--global-code-font, monospace);
  font-weight: 400;
  font-size: 0.8rem;
  color: var(--global-text-color-light);
  margin-left: 0.4rem;
}
.reason-desc { color: var(--global-text-color); font-size: 0.94rem; }
.reason-list { margin: 0.5rem 0 0; padding-left: 1.1rem; }
.reason-list li { margin-bottom: 0.35rem; color: var(--global-text-color-light); font-size: 0.92rem; }
.reason-key {
  font-weight: 600;
  color: var(--global-text-color);
  text-transform: uppercase;
  letter-spacing: 0.03em;
  font-size: 0.8rem;
  margin-right: 0.25rem;
}
</style>
