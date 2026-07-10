---
layout: page
title: Transparency, Provenance, and Trustworthiness
description: provenance records, fabrication-proof verification, and cost accounting that make agent reasoning inspectable
importance: 2
img: assets/img/thumbnails/provance_trust_tn.png
related_publications: false
---

A plausible report is not evidence of performed analysis. Left unguarded, a language-model agent fails in specific ways: it writes a final number out of nowhere, invents an intermediate input value, derives a result in its own head instead of running the tool, ignores a tool that returned an error and reports success anyway, or reuses an earlier result after it has stopped being valid. For [DREAMS]({{ '/projects/dreams/' | relative_url }}), which runs materials simulations autonomously, any of these is disqualifying: a fabricated value that reaches a report wastes experiments and discredits everything else the system produced.

The defense is a chain. The agent must state why it sets each value and in what context; every tool result is stored as an artifact that bundles that reasoning with the value and a unique reference; those artifacts link into a full provenance history; and a judge re-verifies every reported number against that history. These provenance and anti-fabrication systems are current development, not part of the published paper.

## Enforced reasoning and context: no parameter can be set without a stated why, under a stated context

**No value the agent commits is a bare number. Every tool call carries a per-parameter reason and a context, and any sensitive parameter must cite the earlier result it came from.**

The chain starts before anything is saved. Every tool call must fill two fields, which the agent's tools require by their type signature rather than by request.

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

Requiring this before anything runs buys two things:

1. Stating the context and reason up front heads off trivial mistakes the agent would otherwise make.
2. The recorded context and reasons are what the semantic judge later uses to check that each setting is scientifically sound.

## What a registered artifact contains

**When a tool runs, its result is stored as an artifact that bundles the value with everything needed to check it, and is stamped with a unique reference.**

Every time a tool produces a result, that result is registered as a single artifact, and only a tool call can create one. The artifact records the value together with how it was produced and why.

<div class="reason-card">
  <div class="reason-field">
    <div class="reason-name">register_tool_output(...) <span class="reason-type">returns a reference id</span></div>
    <div class="reason-desc">One tool result becomes one artifact, recording:</div>
    <ul class="artifact-fields">
      <li><span class="af-key">tool_name</span> which tool produced the result</li>
      <li><span class="af-key">args</span> the full argument settings it ran with</li>
      <li><span class="af-key">value</span> the result itself</li>
      <li><span class="af-key">description</span> what the result is</li>
      <li><span class="af-key">context</span> which study or exploration the call was part of</li>
      <li><span class="af-key">reasons</span> the per-parameter rationale from the previous section</li>
      <li><span class="af-key">parent_result_ids</span> references to the artifacts this one was derived from</li>
      <li><span class="af-key">metadata</span> anything else attached to the result</li>
    </ul>
  </div>
</div>

The call returns a unique reference id. That id is the only way to cite this result later, whether as the source for a sensitive parameter or as a claim in a report, and it is what the provenance history in the next section is built from.

A human can mark a set of parameters as sensitive, the ones that actually move the answer. Each of those must then be set with a reference to the earlier result that produced its value, and that reference is checked against both the existence of the source and the equality of the value. The agent can omit a reference and be stopped before it proceeds; it cannot fabricate one that passes both checks.

## Provenance: every result has a history

**Every value DREAMS reports can be traced backward to the simulations that produced it.**

Because each artifact records the references it was derived from, the artifacts link into a graph. The graph below is the provenance record of one DREAMS run, drawn from the canvas, the shared memory where agents record every intermediate result. Each node is an artifact; each arrow follows a parent reference, showing what was derived from what. The two green nodes are the run's starting inputs; the two gold nodes are the final lattice-constant calculation and the report built from it. Drag to pan and scroll to zoom; the Fit button shows the whole graph.

<div class="l-page">
  <iframe src="{{ '/assets/html/dreams_provenance_dag.html' | relative_url }}" frameborder="0" height="620px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/dreams_provenance_dag.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The graph is a snapshot of the canvas provenance DAG at step 29 of a run, exported by `dag_visualizer.py` and rendered with vis-network. Each node is an artifact registered when a tool produced a result; edges follow the parent references recorded on each artifact. Graph data is embedded in the HTML; no server is involved after page load.
{% enddetails %}

## Verification: the agent cannot fabricate results

**Every claim in the report below was checked against the simulation record that produced it, parameter by parameter, by a separate judge agent.**

A complete history is only useful if something checks it. Two things make a report checkable. Every claimed number must carry a reference to the tool result that produced it, and the supervisor names in advance which results must be claimed, so the agent cannot quietly drop an inconvenient one. Numbers read from raw output files go through guarded math and extraction tools that require an evidence snippet and check the value against it, which stops the agent from computing or rounding a number in its head.

Once a report exists, a judge agent re-verifies it recursively, one parameter at a time. For each sensitive parameter it first confirms, deterministically, that the cited source exists and its value matches. Then it weighs whether that source is a sensible origin at all: a reference is rejected when it is cross-wired (for example a k-point setting pinned to an energy-cutoff convergence result), when a production run leaves a sensitive parameter with no source, or when a rationale claims a sub-study the context does not support. The judge is deliberately strict; in development it flagged real mistakes, including a value referenced to an input file instead of the output file that computed it. Its own findings become a judging report, which is itself verifiable.

Below is the verification view for one DREAMS report, the lattice constant of body-centered cubic lithium. The graph is the report's value-flow graph, with the final calculation pre-selected. Click any node to inspect it, open the Claims tab to see each checked claim, and the per-parameter checks show what was verified.

<div class="l-page">
  <iframe src="{{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}" frameborder="0" height="720px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/DREAMS_verify_BCC_Li_Lattice_Constant_Final_Report.html' | relative_url }}){:target="\_blank"}

{% details Technical details %}
The view is exported by `verify_dag_visualizer.py`. The page loads its verification state from a separate JSON file at view time, the same fetch pattern planned for near-live status updates on this site. The judge classifies each sensitive parameter by its role (referenced from an upstream result, held without a source, or intentionally varied) and applies a matching rule to each, so an atypical value is accepted when the recorded context justifies it and rejected when it does not. Tabs expose node detail, the claim list with claim-to-source coherence checks, and the event log of the verification run.
{% enddetails %}

## Adversarial tuning of the judge

**A judge is only worth having if it is hard to fool, so it is stress-tested against cases built to slip a fabrication past it, and the two ways it can fail are measured, not assumed.**

An automated judge that is easy to trick gives false confidence, which is worse than no judge at all. To find out how good this one really is, it is run against a bench of about 150 realistic cases. Roughly half are legitimate results that *should* pass. The rest are deliberate fakes, each written to look convincing: a number that appears out of nowhere, a value quietly "laundered" through a calculator step to disguise that it was invented, a setting borrowed from an unrelated calculation, a chemically wrong value wrapped in confident wording, or a justification that claims a shortcut the situation does not allow. The fakes are written as carefully as the real cases, so the judge is tested on the *deception itself* rather than on sloppy writing that would give it away.

Two kinds of mistake are scored, and one is never traded for the other: approving a fake, the costly error, and flagging a legitimate result, which wastes real work. Because these AI judges are slightly random, every case is run several times, across five different AI models, and every disagreement is read by hand to see whether the judge was wrong or the test case was. Often it was the test case: the judge had caught a subtle flaw introduced by accident, which itself builds confidence.

The result, for the strongest of the five models:

<div class="l-body">
  <img src="{{ '/assets/img/trustworthy/judge_confusion.svg' | relative_url }}"
       alt="Confusion matrix of the strongest judge on 145 test cases: of 83 flawed or fabricated cases it flagged 80 and 3 slipped through as approvals; of 62 legitimate cases it approved 61 and raised 1 false alarm."
       style="display:block; margin:1.4rem auto; width:100%; max-width:720px;" />
</div>

Of 83 traps, 80 were caught; the three that slipped through were borderline judgment calls rather than clear fabrications. Of 62 legitimate results, 61 were correctly approved, with a single cautious false alarm. This is what turns "the judge seems to work" into a measured result that can be defended and improved.

{% details Technical details %}
The bench is 145 scenarios across the twelve result-producing tools, grounded in real runs (CO/Pt(111), plus a second chemistry so the judge is not tuned to one system's idioms) and balanced between should-pass and should-fail. Each scenario carries a study-situated rationale, so the judge is tested on the injected defect rather than on rationale-poverty. Every scenario is run at N = 3 samples per model across five models to average out stochasticity, scored as a confusion matrix of intended verdict versus the judge's majority verdict, and tracked as two rates: false positives (a should-pass flagged) and false negatives (a should-fail approved). "Flagged" counts both a hard failure and a softer warning. Tuning is gated on this bench: a wording change to a rule is kept only if it fixes its target cases without regressing the clean ones, cross-validated so a fix found on one model must also hold on the others. The figure shows the best-balanced model (1 false alarm, 3 misses); a stricter model missed only 1 trap but raised 10 false alarms, which is why both error types are reported rather than a single accuracy number.
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
/* Reason/context and artifact schema cards. Colors come from the existing
   theme variables so they track light and dark. */
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
.artifact-fields { margin: 0.6rem 0 0; padding-left: 0; list-style: none; }
.artifact-fields li {
  margin-bottom: 0.35rem;
  color: var(--global-text-color-light);
  font-size: 0.92rem;
  display: grid;
  grid-template-columns: 11rem 1fr;
  gap: 0.6rem;
}
.af-key {
  font-family: var(--global-code-font, monospace);
  color: var(--global-theme-color);
  font-size: 0.85rem;
}
@media (max-width: 576px) {
  .artifact-fields li { grid-template-columns: 1fr; gap: 0.1rem; }
}
</style>
