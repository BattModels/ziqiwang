---
layout: page
title: DREAMS-OER
description: an autonomous agent that screens a materials database for oxygen-evolution catalysts over a multi-hour campaign, tracking hundreds of interdependent simulations without losing the thread
importance: 4
related_publications: false
---

Green hydrogen from water electrolysis is limited by the oxygen evolution reaction (OER), the slow half of splitting water. Better OER catalysts lower the energy the reaction wastes. DREAMS-OER extends the [DREAMS]({{ '/projects/dreams/' | relative_url }}) framework from single calculations to a full screening campaign: given Google DeepMind's GNoME database of roughly 381,000 predicted materials, find which catalyst, at which crystal facet, with which surface termination, and on which adsorption site gives the lowest reaction barrier, subject to stability, cost, and toxicity constraints.

Production screening is still in progress. Every result below is from development runs and is stated as such.

## Why this is hard for an agent

**A screening campaign runs for hours across hundreds of interdependent calculations, and a language-model agent has to stay coherent the whole way.**

The reaction proceeds through four proton-coupled electron transfer steps, and the catalyst's quality is set by the largest free-energy jump among them. Evaluating one candidate means relaxing its bulk structure, choosing facets and terminations, placing adsorbates on surface sites, and computing the free energies of the O, OH, and OOH intermediates, with later steps branching on earlier results.

To do this autonomously, the agent has to satisfy six requirements at once:

1. Track hundreds of calculations across different systems, terminations, and sites simultaneously, with dependencies between them.
2. Record why it is doing each thing, and not forget hypotheses it proposed earlier.
3. Optimize both its compute budget and its time budget.
4. Ground its decisions in the literature.
5. Neither terminate prematurely nor run away and exhaust its budget.
6. Keep every DFT calculation valid, with no hallucinated inputs.

Each mechanism below addresses one or more of these. The workflow the agent follows repeats a site-selection, relaxation, free-energy, and branch-decision loop for each of the three adsorbates.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/workflow.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The screening workflow: literature search, candidate filtering, bulk relaxation, facet and termination selection, surface relaxation, then a per-adsorbate loop over sites, relaxation, free energy, and a branch decision, before combining results into the best picks." %}

{% details Technical details: physics assumptions %}
The backend DFT calculator uses GGA+U with U parameters from the Materials Project, and Materials Project standard DFT settings. Custodian handles DFT errors. Adsorption energies are computed by the backend. Preferred termination is determined by a coordination-based approach. The 4-electron reaction pathway is assumed. Structural anomalies are marked as failures. An unexpected result is trusted during runtime and criticized during reporting.
{% enddetails %}

## Architecture

**DREAMS-OER keeps the DREAMS supervisor-and-worker design and adds an experiment log, an arXiv agent, safety guards, and a report judge.**

A planning supervisor assigns tasks to a DFT agent and an OER-focused HPC agent, each wrapped in safety guards, with a convergence agent for failed calculations. Two additions carry the campaign: an arXiv agent supplies literature grounding for decisions, and everything the agents do is recorded to the canvas and an experiment log. A report-judge agent reviews the final answer against the original request.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/architecture.png" class="img-fluid rounded z-depth-1" zoomable=true caption="DREAMS-OER architecture. (a) Planning supervisor. (b) DFT agent and tools, behind safety guards. (c) OER and HPC agent with an arXiv agent. (d) Convergence agent. (f) Canvas and experiment log. A report-judge agent checks the final report against the user request." %}

What is new relative to DREAMS: a structured experiment log with live HPC status; enforced reasoning and hypothesis claims; time awareness; traceable data flow; extended safety guards over a hard-coded science backend; an arXiv agent; and reporting, judging, and summarizing. Each is covered below.

## The experiment log: one memory for the whole campaign

**A relational experiment log tracks every calculation, its reason, and its result, so the agent always knows the state of the campaign.**

Instead of holding job state in a conversation that scrolls away, DREAMS-OER writes it to a structured log the agent can query, filter, and sort. Every row carries the reason it was created, which is what lets the agent revisit its own decisions hours later.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/explog_query.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The agent querying the experiment log for bulk relaxation jobs, with a stated reason for the query. Each job carries its candidate, type, HPC job ID, and status." %}

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/explog_summary.png" class="img-fluid rounded z-depth-1" zoomable=true caption="A rolled-up status summary from the log: per candidate, how many bulk, surface, O adsorption, and OH adsorption jobs have finished." %}

## Enforced reasoning: no action without a stated why

**Before any consequential action, the agent must state its reason and the hypothesis it is testing.**

The prompts that drive the agent require a justification at every decision point: why this candidate, why this termination, why this adsorption site, and for each simulation parameter, why that value and what effect it is expected to have. This makes the agent's behavior legible after the fact and produces a hypothesis-driven trail rather than a black box.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/reasoning_chain.png" class="img-fluid rounded z-depth-1" zoomable=true caption="An example reasoning chain: literature points to Ir and Ru, an initial test finds a Y-Ir compound exceptional, the agent forms an optimization plan, finds the database sparse for that family, hypothesizes a scandium analog by periodic-table reasoning, tests it, and records that the hypothesis failed and a new direction is needed." %}

## Time and resource awareness

**The agent knows how much of its time budget remains and adapts its plan to protect the final report.**

Every step is timestamped and the agent on duty is reminded of elapsed versus total time, with a watch and an alarm as explicit tools. When time runs short, it makes deliberate tradeoffs rather than stopping abruptly or overrunning.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/time_check.png" class="img-fluid rounded z-depth-1" zoomable=true caption="A time-check decision recorded to the canvas: with about 11 minutes left, the agent chooses to skip a third screening round, finish the current round opportunistically, run only selective OH calculations, and reserve time to generate its final report." %}

## Safety guards and the final judge

**Repeated identical tool calls are cut off, references are checked for existence, and a judge blocks the agent from stopping before the task is done.**

The safety systems from DREAMS are extended for the longer campaign. Tool results are logged and any referenced artifact is verified to exist. Repeated calls to the same tool with the same input and output are warned and then terminated, which stops runaway loops. When the supervisor decides to end, a report-judge agent checks the result against the original user request, guarding against premature termination. Intermediate reports trigger a summarization of the full conversation history so context does not overflow.

## Fully traceable execution

**Every value the agent produced can be traced back through the calculations that produced it, at campaign scale.**

The provenance record for one OER run spans hundreds of nodes: literature searches, database analyses, candidate log entries, and the DFT jobs that follow from them. It is the same provenance idea as the [DREAMS trust page]({{ '/projects/trustworthiness/' | relative_url }}), scaled to a multi-hour screening campaign.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/provenance_dag.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Provenance graph of one OER run: 449 nodes and 702 edges across 35 roots. Selecting a node shows its stated reasons, for example a candidate filter set to a 0.2 eV/atom decomposition threshold with a literature reference." %}

## Early analysis

The following are from development runs. Production screening is still in progress, so these show the kind of analysis DREAMS-OER produces rather than final screening conclusions.

**The catalysts DREAMS-OER explored can be placed on the standard activity map for OER.**

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/volcano.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Explored candidates on the OER activity map: computed O-site free energy against an optimal window (left), and the candidates placed on the two-dimensional overpotential volcano under the scaling relation and the ideal case (right)." %}

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/scaling_relations.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Scaling relation among the adsorbate free energies for the explored candidates." %}

**Dimensionality reduction shows how much of the design space the agent covered, in composition and in structure.**

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/umap_composition.png" class="img-fluid rounded z-depth-1" zoomable=true caption="UMAP projection of explored candidates by composition (magpie features)." %}

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/element_occurrence.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Element occurrence across two screening rounds, showing which elements the explored candidates contain." %}

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/umap_structural.png" class="img-fluid rounded z-depth-1" zoomable=true caption="UMAP projection of explored candidates by structure (Coulomb matrix features)." %}

**Token accounting shows where a multi-hour campaign spends its budget, across communication and other tools.**

Tokens are the unit of work and cost for the agent. The explorer below breaks down one OER run by tool and by step, separately for input and output tokens, with a slider to walk through the 656-step session.

<div class="l-page">
  <iframe src="{{ '/assets/html/oer_token_usage.html' | relative_url }}" frameborder="0" height="1300px" width="100%" style="border: 1px solid var(--global-divider-color); border-radius: 6px;"></iframe>
</div>

[Open full screen]({{ '/assets/html/oer_token_usage.html' | relative_url }}){:target="\_blank"}

{% details Technical details: acknowledged limitations of the development runs %}
DREAMS-OER records its own limitations. Computational: PBE+U may miss some electronic-structure effects; the G(OOH) = G(OH) + 3.2 eV scaling relation may not hold for all sites; coordination-based termination ranking is a heuristic, not a surface-energy calculation; no explicit solvation; only thermodynamic overpotentials, no kinetic barriers. Experimental: no experimental validation; Pourbaix stability does not guarantee long-term stability; real surfaces may reconstruct; bulk calculations may miss nanoparticle effects. Data quality in the runs shown: one candidate with the best O-site free energy had all its OH jobs fail; 20 of 56 O jobs failed or were unrecoverable (a 36% failure rate); only 1 to 3 sites per termination and 1 to 2 terminations per candidate were sampled. Hypothesis testing: only 30 candidates tested, in unbalanced element groups, with confounding mixed-metal effects and pre-filtering by Pourbaix stability.
{% enddetails %}

## Live status (phase 2)

<div class="l-body" style="border: 1px dashed var(--global-divider-color); border-radius: 6px; padding: 24px; text-align: center; color: var(--global-text-color-light);">
A near-live status panel for the running screening campaign is planned here. It will read snapshot data pushed from the HPC cluster.
</div>
