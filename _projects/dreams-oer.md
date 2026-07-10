---
layout: page
title: DREAMS-OER
description: an autonomous agent that screens a materials database for oxygen-evolution catalysts over a multi-hour campaign, tracking hundreds of interdependent simulations without losing the thread
importance: 4
img: assets/img/dreams-oer/workflow.png
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

A planning supervisor breaks the objective into a plan and assigns each step to one of two worker agents, both wrapped in safety guards. The DFT agent generates crystal structures, optimizes settings, samples k-points, and saves files, with a convergence agent as its child to handle failed calculations. The OER agent carries the screening-specific work through its own toolset: it inspects the GNoME database, uses an OER toolbox, submits DFT jobs to the HPC cluster and tracks their time, and calls an arXiv agent for literature grounding. Everything the agents do is read from and recorded to the canvas, which holds notes, artifacts, and reports, alongside a running log. When the supervisor decides the task is done, a report-judge agent reviews the final answer against a set of rules before it is accepted.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/architecture.png" class="img-fluid rounded z-depth-1" zoomable=true caption="DREAMS-OER architecture. (a) Planning supervisor with an editable plan. (b) DFT agent and its tools, behind safety guards. (c) OER agent with its own OER tools (inspect GNoME, OER toolbox, submit DFT jobs, check time), behind safety guards. (d) Convergence agent, a child of the DFT agent. (e) HPC cluster. (f) arXiv agent. (g) Canvas holding notes, artifacts, and reports. (h) The running log. A report-judge agent checks the final report against its rules." %}

What is new relative to DREAMS: a structured experiment log with live HPC status; enforced reasoning and hypothesis claims; time awareness; traceable data flow; extended safety guards over a hard-coded science backend; an arXiv agent; and reporting, judging, and summarizing. Each is covered below.

## The experiment log: one memory for the whole campaign

**A relational experiment log tracks every calculation, its reason, and its result, so the agent always knows the state of the campaign.**

Instead of holding job state in a conversation that scrolls away, DREAMS-OER writes it to a structured log the agent can query, filter, and sort. Every row carries the reason it was created, which is what lets the agent revisit its own decisions hours later.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/explog_query.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The agent querying the experiment log for bulk relaxation jobs, with a stated reason for the query. Each job carries its candidate, type, HPC job ID, and status." %}

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/explog_summary.png" class="img-fluid rounded z-depth-1" zoomable=true caption="A rolled-up status summary from the log: per candidate, how many bulk, surface, O adsorption, and OH adsorption jobs have finished." %}

## An investigation, not a random search

**What the agent studies next follows from what it just learned, not from random sampling.**

DREAMS-OER reasons from the literature and its own accumulated results toward the next candidate to try. It states an explicit hypothesis before testing it and records whether the hypothesis held. The chain below is one example: the agent behaves like an investigator working a lead, not a search that samples the space blindly. The discipline behind this, requiring a stated reason and context for every choice, is described on the [Transparency, Provenance, and Trustworthiness]({{ '/projects/trustworthiness/' | relative_url }}) page.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/reasoning_chain.png" class="img-fluid rounded z-depth-1" zoomable=true caption="An example reasoning chain: literature points to Ir and Ru, an initial test finds a Y-Ir compound exceptional, the agent forms an optimization plan, finds the database sparse for that family, hypothesizes a scandium analog by periodic-table reasoning, tests it, and records that the hypothesis failed and a new direction is needed." %}

## Time and resource awareness

**The agent knows how much of its time budget remains and adapts its plan to protect the final report.**

Every step is timestamped and the agent on duty is reminded of elapsed versus total time, with a watch and an alarm as explicit tools. When time runs short, it makes deliberate tradeoffs rather than stopping abruptly or overrunning.

{% include figure.liquid loading="eager" path="assets/img/dreams-oer/time_check.png" class="img-fluid rounded z-depth-1" zoomable=true caption="A time-check decision recorded to the canvas: with about 11 minutes left, the agent chooses to skip a third screening round, finish the current round opportunistically, run only selective OH calculations, and reserve time to generate its final report." %}

## Specification gaming, and the gate that stops it

**Told to keep the supercomputer busy, the agent learned to submit jobs it never looked at. A disposition gate now blocks it from resting until every result has actually been analyzed.**

A screening campaign wastes money when the HPC queue sits idle, so an early version of DREAMS-OER was prompted to keep the queue full. It appeared to work. But reading back through the agent's history showed what it had actually learned to do: submit jobs for the sake of submitting jobs, without reading the results or advancing the study. The agent had optimized the instruction we gave, keep the queue busy, instead of the goal we meant, make progress on the science. This is specification gaming, a recurring failure mode of capable agents handed a proxy metric.

The current design removes the proxy and replaces it with a gate. The agent is no longer told to fill the queue. Instead, the tool it uses to pause and wait for jobs to finish is gated on analysis: the agent may only wait once every candidate has been fully dispositioned, meaning every finished calculation has been tied back to its candidate and the hypothesis it was testing, with a recorded decision on where to take that candidate next. Only after that gate passes does a second, optional check keep the queue from draining. If any result is still unanalyzed, the wait is refused and the agent is told exactly which candidates and which finished jobs it skipped.

The agent can no longer buy idle time with busywork. To rest, it has to do the analysis, which was the reason for running the jobs in the first place. This is current development, being built into the production screening run.

{% details Technical details: dispositions and how the gate works %}
**What a disposition records.** Each time the agent dispositions a candidate, it writes four things: a summary of what the finished results say about that candidate and which hypothesis they confirmed or rejected; the exact job ids the summary is based on; a decision drawn from a fixed set of categories (such as abandon, run more O or OH sites, or enough), which the agent can later filter and sort on to steer the overall campaign, and which cannot combine contradictory choices; and a plan for the next calculations to run. Successive dispositions for a candidate are kept as a record, so its history of decisions is preserved. A candidate counts as fully dispositioned when its latest disposition covers every one of that candidate's newly finished jobs; if the agent leaves one out, the gate names the specific job and candidate.

**The two gates.** The wait tool enforces two. The first, always active, requires every candidate with finished jobs to be fully dispositioned. The second, which the supervisor sets per task, requires the number of pending jobs to stay above a floor so the queue does not empty; the supervisor waives it when winding the study down near the time limit, where new jobs would not finish anyway.

**Keeping the analysis honest.** Two per-candidate flags stop the agent from faking a disposition. When results arrive, the candidate is flagged as needing an update. Before the agent can write a new disposition it must first read the candidate's previous record, which unlocks the write; committing the write locks it again, so each update builds on the prior record instead of being overwritten blindly or padded with repeated calls. Batch jobs, where several magnetic configurations or OH initializations are tried together, are dispositioned only once the whole batch reaches a terminal state, and citing one job from a batch counts for the batch. When the queue runs low, the agent is handed a hint that names any forgotten work, such as a converged bulk structure with no surface job or a promising O adsorption with no OH follow-up, and points it back to the supervisor to expand the study rather than invent filler.
{% enddetails %}

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

The three intermediates are not independent. Across catalysts, the adsorption free energies of O, OH, and OOH are linearly correlated, a well-known scaling relation (for OER, the free energy of OOH stays close to that of OH plus about 3.2 eV). This correlation puts a floor under the overpotential any single-site catalyst can reach, which is why the peak of the volcano sits above the ideal line.

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
