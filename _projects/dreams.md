---
layout: page
title: DREAMS
description: a hierarchical multi-agent AI framework that runs density functional theory simulations end to end, validated at human expert accuracy on standard benchmarks
importance: 1
img: assets/img/thumbnails/dreams.png
related_publications: false
---

New materials for batteries and catalysts are screened by simulation before anyone synthesizes them. The workhorse method is density functional theory (DFT). DFT is hard to automate: each simulation needs settings that experts tune by hand, a single study can require hundreds of runs, and those runs must be mutually comparable. To compare N calculations, all N have to be run with compatible settings; otherwise the comparison, and any result built on it, is invalid.

DREAMS (DFT-based Research Engine for Agentic Materials Simulation) automates this work. It is described in the DREAMS paper ([arXiv:2507.14267](https://arxiv.org/abs/2507.14267)), where I am first author. This page covers the framework and its benchmark results.

(Automation at this level creates a second problem: an agent can report a correct-looking answer built on reasoning it never performed. The provenance records, verification systems, and safety guard that address this are covered on the [Transparency, Provenance, and Trustworthiness]({{ '/projects/trustworthiness/' | relative_url }}) page.)

## One supervisor plans, specialized agents execute

**DREAMS splits DFT work across a planning supervisor and specialized worker agents, so no single model carries the whole problem.**

The supervisor turns a research objective into a task list and revises it as results arrive. A DFT agent builds crystal structures, runs convergence tests, prepares calculations, and analyzes and post-processes the results. An HPC agent allocates compute resources and submits and monitors jobs on the supercomputer. When a calculation fails, a dedicated convergence agent diagnoses it from the input and output files. Every agent reads and writes one shared memory, the canvas.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig1.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 1: The DREAMS architecture. (a) Planning supervisor. (b) DFT agent and its tools. (c) HPC agent and its tools. (d) Convergence agent. (e) HPC cluster. (f) Canvas, the shared memory." %}

{% details Technical details %}
DREAMS is implemented as a hierarchical multi-agent system on the LangGraph framework using the Claude 3.7 Sonnet API. Worker agents are ReAct agents whose tools interface through the Atomic Simulation Environment (ASE), preventing direct file manipulation. Structure generation for the adsorption benchmark uses the AutoCat library, constraining structures to physically meaningful configurations. A batch modification tool generates Quantum ESPRESSO convergence-test scripts from a template, reducing script generation from O(N) to O(1) manual effort.
{% enddetails %}

## A shared memory keeps the agents consistent

**All agents communicate through the canvas, a structured shared memory, instead of passing text to each other.**

The figure shows one full communication cycle: the user's request goes to the supervisor, the supervisor checks the canvas and assigns a task, and the worker agent reads what it needs from the canvas and calls its tools. The tools themselves also read from and write to the canvas, looking up prior information and storing new data and results before handing control back to the agent, which then writes its own results back. Keeping every value in its original form, rather than as text in a conversation, prevents copy errors and lost context during long runs.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig2.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 2: Communication flow among the user, supervisor, worker agents, tools, and the canvas." %}

The canvas records where every value came from, under which settings, and under which conditions, which is what makes provenance tracking and verification possible. That story continues on the [trust page]({{ '/projects/trustworthiness/' | relative_url }}).

{% details Technical details %}
The canvas is a centralized dictionary holding all variables in native format, with three exposed operations: inspection (list all keys), reading (by key, with suggestions on invalid keys), and writing (descriptive key required; overwriting an existing key requires confirmation). Keys can be marked read-only, protected, or format-restricted. Access control and transparent logging keep multi-agent communication traceable.
{% enddetails %}

## Benchmark 1: all 27 crystal structures correct (Sol27LC)

**DREAMS reproduced 27 of 27 crystal structures on the Sol27LC lattice-constant benchmark, with average errors below 1% of human DFT expert results.**

Sol27LC contains 27 elemental crystals with varying structures. For each one, DREAMS planned the workflow, chose its own simulation settings, submitted the calculations, and calculated the lattice constant, end to end, without human intervention.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig3.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 3: End-to-end execution log of the Sol27LC benchmark. Each step is designed by the supervisor and assigned to a worker agent." %}

| Structure | Systems | Correct structures | MAPE | k-point range | ecutwfc |
| --- | --- | --- | --- | --- | --- |
| BCC | Li, Na, K, ... | 11/11 | 0.36% | 8-16 | 40-70 |
| FCC | Rh, Ir, ... | 12/12 | 0.51% | 8-18 | 40-70 |
| DIA | C, Si, Ge, ... | 4/4 | 1.00% | 6-8 | 40-70 |

<div class="caption">Table 1: Correct structures generated by DREAMS, mean absolute percentage error (MAPE) against human DFT expert results, and the parameter ranges DREAMS chose, across the Sol27LC benchmark.</div>

**DREAMS picks its own simulation settings by systematic convergence testing, the same way an expert would.**

Every DFT setting trades cost against accuracy. Set it too fine and the calculation becomes too slow to ever finish; set it too coarse and the result cannot be trusted. Before running production calculations, DREAMS varies each numerical setting until the result stops changing, then locks in the point where the answer is stable at the lowest cost that still preserves accuracy. This is the step that normally consumes expert time and is skipped by naive automation.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig4.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 4: Convergence tests for the two main DFT parameters. The agent selects final parameters against a 1 meV/atom accuracy threshold." %}

{% details Technical details %}
The plane-wave energy cutoff (ecutwfc) is converged first while holding k-point spacing fixed at 0.1 per Angstrom; the k-point mesh sampling is then converged at the chosen cutoff of 120 Ry. Energy differences are computed against a reference obtained with strict settings (ecutwfc = 120 Ry, k-spacing = 0.1 per Angstrom), with a 1 meV/atom convergence threshold decided by the convergence agent. Converging the two parameters separately follows common practice in the DFT community.
{% enddetails %}

## Benchmark 2: the CO/Pt(111) puzzle to 0.2%

**On the CO/Pt(111) adsorption puzzle, DREAMS reproduced the human expert answer to 0.2%, recovering from seven failed calculations on its own.**

CO/Pt(111) asks which surface site a carbon monoxide molecule prefers on a platinum surface. It is a long-standing benchmark, chosen both for its sensitivity to calculation settings and for its industrial relevance: CO on platinum sits at the center of automotive catalytic converters and fuel-cell electrodes, and pinning down the preferred site has been a persistent test for simulation methods. Mid-run, DREAMS noticed a missing calculation script and revised its own plan; when seven of eight production jobs failed, the convergence agent diagnosed the failures and DREAMS resubmitted corrected calculations until all runs converged. Crucially, whenever it changed a setting on one calculation, it applied the same change to every related calculation. This matters because the final adsorption energy is a difference between separate calculations; if those calculations do not all share the same settings, the difference is meaningless and the entire run is invalid.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig5.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 5: End-to-end execution log of the CO/Pt(111) study. The supervisor modifies the plan mid-run and invokes the convergence agent to resolve failures." %}

| Supercell | Coverage | Functional | Agent team | Human expert | Literature |
| --- | --- | --- | --- | --- | --- |
| 2x2 | 1/4 ML | PBE | 0.1078 eV | 0.108 eV | 0.10-0.24 eV |
| 2x2 | 1/4 ML | LDA | 0.318 eV | 0.320 eV | 0.32-0.45 eV |

<div class="caption">Table 2: Adsorption-energy difference between the ontop and FCC sites calculated by DREAMS, a human expert, and literature values, for two DFT functionals.</div>

**The convergence agent recovers failed calculations the way an experienced practitioner would, which a fixed pipeline cannot do.**

When a calculation fails to converge, a traditional automated pipeline has no recourse: it cannot reason about why the failure happened. DREAMS' convergence agent reads the failed run's input and output files and proposes specific parameter changes, each with a justification grounded in DFT practice. The suggestions below are from the CO/Pt(111) run.

| Parameter | Suggestion | Reason |
| --- | --- | --- |
| ecutwfc | increase to 80.0 | higher cutoff needed for ultrasoft pseudopotentials with transition metals |
| degauss | increase to 0.03 | helps with metallic systems |
| mixing_beta | add with value 0.3 | the current value of 0.7 is too high for this complex system with CO adsorption, leading to charge sloshing |
| mixing_mode | local-TF | a more sophisticated scheme like local-TF can help with difficult convergence cases |
| electron_maxstep | 300 | the calculation stopped at 200 iterations but was still making progress, so more iterations might help it converge |

<div class="caption">Table 3: Representative convergence-agent suggestions and rationales.</div>

Each of these is a judgment an expert makes from experience, not a rule a pipeline can hard-code: recognizing charge sloshing from a mixing factor set too high, or seeing that a run was still improving when it hit its iteration limit. Automating this recovery is what lets DREAMS finish long, failure-prone studies without a person watching.

## Quantifying the uncertainty of the physics itself

**DREAMS quantified how much the CO/Pt(111) answer depends on the chosen physics approximation, and the site preference held across the whole uncertainty range.**

Different DFT approximations (functionals) give different results. DREAMS ran a Bayesian ensemble analysis (BEEF-vdW) that samples thousands of plausible functionals, producing a distribution of answers instead of a single value. The agent's distribution agrees with the human expert's, and both keep the same site preference.

{% include figure.liquid loading="eager" path="assets/img/dreams/fig6.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Figure 6: (a) Representative structures DREAMS can generate. (b) Configurations used for the BEEF ensemble analysis. (c) Distribution of adsorption-energy differences: the human expert ensemble mean is -0.13 eV and DREAMS produces -0.12 eV, each with a standard deviation of 0.01 eV." %}

{% details Technical details: human validation through bond length and charge transfer analysis %}
To confirm that DREAMS reached the right answer for the right physical reason, we (the authors) analyzed the adsorbed structures by hand as an independent check on the agent. Bader charge and bond-length analysis across all three functionals supports the FCC-site preference DREAMS predicted. For every adsorbed CO, the C-O bond lengthens relative to the free molecule, because electrons transfer from the platinum surface into the antibonding orbitals of the C-O bond and weaken it. The FCC site draws 0.15 to 0.18 more electron transfer than the ontop site, and correspondingly shows the longer C-O bond; the larger transfer means a stronger CO-Pt interaction, which is why CO binds more favorably at the FCC site. This is the back-donation mechanism, described for CO on transition metals as early as 1964 and long argued to favor the FCC site. That DREAMS recovered the same preference with no challenge-specific tuning is what makes the result convincing.

| Functional | Ontop d(C-O) | Ontop charge | FCC d(C-O) | FCC charge | CO molecule d(C-O) |
| --- | --- | --- | --- | --- | --- |
| LDA | 1.15 A | 0.02 e- | 1.18 A | 0.18 e- | 1.13 A |
| PBE | 1.16 A | 0.03 e- | 1.19 A | 0.21 e- | 1.14 A |
| BEEF-vdW | 1.15 A | 0.08 e- | 1.18 A | 0.23 e- | 1.13 A |

<div class="caption">Table 4: Bond length and charge transfer at different CO adsorption sites on Pt(111).</div>
{% enddetails %}

## Against other agentic frameworks

**On the harder CO/Pt(111) challenge, DREAMS completed 100% of steps at 0.2% error while baseline frameworks stalled or produced a 389% error, and it used 2 to 3x fewer tokens than the strongest baseline.**

| Challenge | Agent | Steps attempted | Steps succeeded | MAPE | STD |
| --- | --- | --- | --- | --- | --- |
| Sol27LC, BCC lithium | MDCrow | 100% | 100% | 0.55% | 0 A |
| | ChemGraph | 100% | 100% | 0.55% | 0 A |
| | DREAMS | 100% | 100% | 0.55% | 0 A |
| CO/Pt(111), PBE | MDCrow | 48% | 29% | - | - |
| | ChemGraph | 90% | 57% | 389% | 0.390 eV |
| | DREAMS | 100% | 100% | 0.2% | 0.001 eV |

<div class="caption">Table 5: Benchmarking agentic systems on both challenges: fraction of workflow steps attempted and succeeded, mean absolute percentage error of the final result, and standard deviation across independent runs.</div>

The two baselines represent different agent designs. MDCrow is an early single-agent framework, built on LangChain, that reasons and calls tools in one sequential loop, which limits how well it can coordinate multi-step dependencies. ChemGraph is a graph-based multi-agent framework, built on LangGraph, with modular coordination, but unlike DREAMS it has no specialized worker agents for domain-specific subtasks, no persistent communication layer for exchange between agents and tools, and no way to adapt its plan from intermediate results. To keep the comparison fair, both baselines were given the same DFT toolset as DREAMS, without the canvas, and only their prompts were adjusted to the DFT setting while preserving each framework's original design (for example, "You are a molecular dynamics expert" became "You are a DFT expert").

Each framework was run several times on both tasks, so the comparison measures reliability as well as accuracy, using four metrics: how many workflow steps it attempted, how many it completed successfully, the error of its final number, and the spread across repeated runs.

All three handle the short Sol27LC workflow. The differences appear on the long-horizon problem: MDCrow loses context and misuses tools; ChemGraph completes most steps but repeats calculations and overwrites intermediate files, producing an invalid answer. DREAMS keeps full context through the canvas and hierarchical communication, and its lower token usage comes from shared context and clear handoffs rather than repeated work.

## What keeps the answers honest

A correct number means little if the reasoning behind it cannot be checked. Beyond reaching the right answer, DREAMS keeps its entire process transparent and traceable, and it judges every result strictly before that result is trusted. The provenance records, verification systems, and the safety guard that make this possible have their own page: [Transparency, Provenance, and Trustworthiness]({{ '/projects/trustworthiness/' | relative_url }}).
