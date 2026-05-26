# Activating White-Box Process-Level Diagnostics in Compound AI Systems: Execution Tracing, Cognitive Metrication, and Trajectory Alignment

## The Transition from Black-Box Outcomes to Process-Level Diagnostics

Evaluating autonomous large language model (LLM) agents has historically relied on black-box, outcome-only metrics.1 These paradigms assess an agentic system by comparing its final generated output against a static, human-annotated reference target, summarizing performance as a binary pass/fail scalar.1 While simple to execute, outcome-only testing fails to scale with the complexity of multi-step, tool-augmented systems.3 This operational limitation is driven by two primary architectural vulnerabilities:

First, a correct final output can mask severe upstream logic failures, a phenomenon known as logical divergence, reward hacking, or rote memorization.1 For example, in a complex genomic sequence analysis task, an agent might construct an execution pipeline that completely ignores batch effects or implements an incorrect normalization function.1 Despite these structural errors, the system may still generate a visually plausible plot and hit the correct cluster count purely by chance, dataset bias, or data contamination.1 Conversely, a highly rigorous, scientifically sound trajectory might be penalized with a binary score of zero if the final output exhibits a negligible formatting mismatch or a minor floating-point deviation.1

Second, outcome-level benchmarks assume a singular ground-truth execution pathway, penalizing alternative but equally valid strategies.1 In real-world environments, a user request can often be resolved through multiple distinct sequence paths.3 A rigid, outcome-centric evaluator marks these creative or alternative workflows as incorrect, stifling the development of agentic adaptability and tool selection.3

Process-level diagnostics address these limitations by shifting the focus from final outputs to the entire execution trajectory, which is logged as a sequence of thought-action-result triples.2 This white-box approach allows system developers to inspect the logic, tool invocations, and environmental feedback that collectively shape the agent's path.4

| **Evaluation Paradigm** | **Focus of Assessment** | **Treatment of Alternate Paths** | **Major Failure Mode** | **Optimal Use Case** |
| --- | --- | --- | --- | --- |
| **Outcome-Level (Black-Box)** 1 | Final system output matched against a static reference.1 | Penalizes valid alternative methodologies.1 | Susceptible to logical divergence, reward hacking, and memorization.1 | Simple, low-step tasks with deterministic outcomes.3 |
| **Process-Level (White-Box)** 1 | Complete trajectory (thoughts, tool calls, and states).2 | Validates alternative paths against task-specific rubrics.1 | High latency and evaluation token cost.6 | Long-horizon, multi-step tool-use and enterprise agents.8 |

To scale process-level evaluation without the high cost of manual human annotation, frameworks like **TRACE** provide reference-free evaluation for tool-augmented LLMs.3 TRACE assesses whether an agent has followed a logically sound sequence of operations without depending on a single ground-truth path.3 By avoiding direct comparison with a static trace, TRACE accommodates multiple valid problem-solving strategies and mitigates the performance degradation commonly observed when LLM-as-a-judge models process long, complex context windows.3

Similarly, **BiomniBench-DA** utilizes task-specific, expert-designed rubrics to grade the full execution trajectory of biomedical agents, providing the fine-grained diagnostics that outcome-only scoring cannot provide.1

## Tracing Infrastructures and Multi-Granular Observability Topologies

Diagnosing long-horizon agent trajectories requires tracing infrastructures that can decompose an execution history into structured, assessable units.7 Distributed tracing frameworks, such as OpenTelemetry and the OpenInference convention, capture agent executions as a hierarchical tree of spans, where each span represents a specific unit of activity.6

A comprehensive evaluation framework utilizes this hierarchy to pair bottom-up span-level evaluation with top-down agent-level diagnostics.7

                 
                                 │
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
                                  
         │                                               │
  ┌──────┴──────┐                                 ┌──────┴──────┐
  ▼             ▼                                 ▼             ▼
                     

Within this hierarchy, each specialized span type is evaluated against a targeted metric taxonomy to identify and localize failures 6:

- **Root Spans:** Capture the entire multi-agent or single-agent execution from the initial user query to the final response, monitoring end-to-end latency and overall query cost.6

- **Agent Spans:** Monitor individual agent processes, tracking metric dimensions such as Plan Efficiency, Tool Coverage, and Tool Abuse (e.g., assessing if the agent uses tools efficiently without repeated calls and adapts properly after errors).6

- **LLM Spans:** Monitor individual foundation model calls, checking for Instruction Following (conformance to system instructions), Logical Integrity (soundness of context understanding and logical consistency), and Avoidance.6

- **Tool Spans:** Validate external tool or API invocations, tracking Tool Completeness (fulfillment of intended parameters) and system-level error codes.6

- **Retriever Spans:** Measure retrieval context quality by comparing the retrieved document embeddings against the input query.6

Deploying this multi-granular topology on trace-annotated benchmarks like **TRAIL** (which contains 841 span-level errors under a 20+ category taxonomy across GAIA and SWE-bench traces) yields massive performance improvements.7 Decomposing the evaluation task into independent span assessments rather than relying on a monolithic LLM judge resolves context-window limitations and "lost-in-the-middle" attention loss.7

| **Benchmark Dataset** | **Evaluation Framework** | **Category F1​ Score Gain** | **Localization Accuracy** | **Joint Localization-Categorization Accuracy** |
| --- | --- | --- | --- | --- |
| **GAIA** 7 | Holistic Decomposed (Decompresses leaf spans bottom-up).7 | +26.0% Relative 10 | 1.25x Increase 10 | 2.58x Increase 10 |
| **SWE-bench** 7 | Holistic Decomposed (Decompresses leaf spans bottom-up).7 | +38.0% Relative 10 | 3.50x Increase 10 | 12.50x Increase 10 |

These results demonstrate that the same frontier foundation model achieves multiple times higher localization accuracy when wrapped in a decomposed, span-aware evaluation framework compared to its performance as a monolithic judge over the full trace.11 This confirms that the evaluation topology, rather than the raw model capability, is the primary performance bottleneck.11

## Abductive Assertion Verification and Execution-Driven Mutation Engines

As autonomous agents transition from editing simple code snippets to managing repository-level software projects, they face significant cognitive stressors.12 Repository-level operations require agents to track state mutations across complex file systems, traverse deep call chains, and manage multi-source dependencies.12

To evaluate these capabilities without data leakage or memorization errors, the **RepoReason** benchmark introduces **Abductive Assertion Verification** paired with an **Execution-Driven Mutation Engine**.12 This framework extracts authentic unit-test assertions from mainstream repositories (such as sympy, toolz, and jinja2) to serve as semantic anchors.13 The values of these assertions are masked (e.g., assert len(cache) == <mask_value>), forcing the agent to perform abductive logical inference.13 It must reconstruct the preceding execution history across multi-module call graphs to deduce the verified state.13

To eliminate memorization shortcuts, the execution-driven mutation engine treats the test environment as a Semantic Oracle.13 It applies visual mutations (such as renaming variables and reordering imports) and semantic mutations (such as perturbing input variables) while keeping the underlying logic intact.13 By re-executing the mutated repository inside a sandbox, the Semantic Oracle injects print-statement probes and applies a Deterministic Value Protocol to programmatically filter out unstable or non-deterministic test cases.13

To select complex, high-density test cases, the benchmark applies a weighted scoring system based on captured runtime traces, calculating a complexity score for each test function 14:

where  represents the count of unique files,  denotes the count of functions,  represents the number of call operations, and  measures the call stack depth.14

Using dynamic program slicing, RepoReason isolates the minimal causal computational subgraph that directly influences each assertion, calculating three orthogonal cognitive metrics to pinpoint agent failures 12:

- **Effective Sliced Volume (ESV):** Measures the reading load as the cumulative source code volume (in lines of code) of all causal functions in the slice, normalized to a 600 LoC baseline.12

- **Mutation Chain Length (MCL):** Quantifies the simulation depth as the sum of all executed statements in the slice, weighted by execution frequency and normalized to 100 steps.12

- **Dependency Fan-In (DFI):** Captures the integration width as the number of independent external inputs feeding into the slice, normalized by 20.12

| **Evaluated Model** | **Overall Accuracy (%)** | **Easy Cases (%)** | **Medium Cases (%)** | **Hard Cases (%)** | **Correlation with ESV (Pearson ρ)** | **Correlation with MCL (Pearson ρ)** | **Correlation with DFI (Pearson ρ)** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Claude-4.5-Sonnet** 14 | 66.98 | 80.27 | 66.79 | 47.39 | -0.161 | -0.122 | -0.225 |
| **DeepSeek-v3.1-Terminus** 14 | 60.96 | 74.03 | 60.02 | 41.71 | -0.152 | -0.122 | -0.157 |
| **GPT-5.2** 14 | 56.86 | 75.92 | 53.94 | 30.85 | -0.188 | -0.158 | -0.234 |
| **Kimi-K2** 14 | 54.74 | 69.38 | 51.79 | 35.66 | -0.130 | -0.117 | -0.195 |
| **Qwen3-Coder-480B** 14 | 50.56 | 61.35 | 49.16 | 35.50 | -0.148 | -0.154 | -0.196 |

This diagnostic analysis reveals two distinct failure modes 12:

- **Comprehension and Simulation Cliffs:** Model accuracy drops sharply when the reading load exceeds 600 LoC () or the simulation path exceeds 100 execution steps ().12

- **The Aggregation Deficit:** The strongest negative correlation across all tested models is with , showing that integration width is the primary cognitive bottleneck in repository-level tasks.12 As the number of external inputs to synthesize increases, models struggle to integrate multiple independent facts into a cohesive conclusion.12

## Trajectory Alignment, Decision Calibration, and Long-Horizon Stability

When deployed in highly regulated enterprise environments (such as insurance claims or loan qualification), long-horizon agents operate under strict regulatory and operational constraints.8 Evaluating these systems requires multidimensional alignment frameworks that can diagnose decision-making quality over long context windows.8

The **Four-Axis Decision Alignment** framework decomposes long-horizon agent behavior into four orthogonal, measurable dimensions 8:

- **Factual Precision (FRP):** Measures the proportion of required, ground-truth facts (e.g., exact financial values or dates) preserved exactly across the trajectory.8

- **Reasoning Coherence (RCS):** Evaluates the logical consistency between cited facts, applied regulatory rules, and the final decision.8

- **Compliance Reconstruction (CRR):** Verifies if the agent's output satisfies regulatory standards, such as reconstructing the exact statutory provisions required for adverse action notices.8

- **Calibrated Abstention (CAR):** Measures the agent's ability to decline to make a decision when presented with incomplete or ambiguous evidence, opting instead to flag the case for human review.8

Empirical studies on **LongHorizon-Bench (LHB)** expose critical trade-offs across these alignment axes.8 For instance, pure retrieval-only architectures frequently collapse on Factual Precision, while schema-anchored memory architectures maintain high precision but pay a substantial "scaffolding tax" that degrades processing efficiency.8

| **Budget Context** | **Evaluation Metric** | **Stateless Decision Memory (DPM)** | **Plain Summarization Baseline** | **Performance Delta (Δ)** | **Statistical Significance (p-value)** | **Cohen's h Effect Size** |
| --- | --- | --- | --- | --- | --- | --- |
| **Tight Budget** 19 | Factual Precision () 17 | 0.907 | 0.392 | +0.515 | 0.001 | +1.17 |
| **Tight Budget** 19 | Reasoning Coherence () 17 | 0.800 | 0.267 | +0.533 | 0.003 | +1.13 |
| **Tight Budget** 19 | End-to-End Accuracy () 17 | 1.000 | 0.500 | +0.500 | 0.065 | +1.57 |
| **Tight Budget** 19 | Compliance Reconstruction () 17 | 0.900 | 0.400 | +0.500 | 0.066 | +1.13 |
| **Moderate Budget** 19 | Factual Precision () 17 | 0.907 | 0.950 | -0.043 | 0.255 | -0.17 |
| **Moderate Budget** 19 | Reasoning Coherence () 17 | 0.800 | 0.750 | +0.050 | 0.669 | +0.12 |
| **Moderate Budget** 19 | End-to-End Accuracy () 17 | 1.000 | 1.000 | 0.000 | 1.000 | 0.00 |
| **Moderate Budget** 19 | Compliance Reconstruction () 17 | 1.000 | 1.000 | 0.000 | 1.000 | 0.00 |

These evaluations show that plain summarization with fact-preservation prompts remains a highly competitive baseline under moderate and loose memory budgets, but collapses under tight budgets where specialized stateless memory architectures like DPM maintain stable performance.18

In open-ended dialogic environments, long-term alignment is monitored via the **Intent Drift Score (IDS)**.20 Intent drift is a trajectory-level instability where multi-turn agents gradually diverge from the user's target objective, even when individual execution steps appear correct.20 Grounded in rate-distortion and stability theories, IDS integrates semantic, structural, and temporal signals into a prefix-monotone, linear-time computable metric, enabling real-time monitoring of drift across million-token context windows.20

This process-level focus is further validated by domain-specific benchmarks like **BiomniBench-DA**, which evaluates biomedical data-analysis agents using expert-designed rubrics across six dimensions 1:

| **BiomniBench-DA Dimension** | **Score Range (% of available points)** | **Diagnostic Assessment** |
| --- | --- | --- |
| **Source Reliability** 22 | 88 - 98 | **Strong:** Agents reliably locate and cite real primary sources.1 |
| **Data Handling** 22 | 58 - 78 | **Adequate:** Satisfactory loading, cleaning, and normalization.22 |
| **Statistical Rigor** 22 | 45 - 74 | **Variable:** Minor flaws in pipeline execution and batch correction.2 |
| **Biological Interpretation** 22 | 45 - 74 | **Variable:** Lapses in drawing accurate disease and genetic conclusions.22 |
| **Scientific Logic** 22 | 45 - 74 | **Variable:** Gaps in maintaining a logically supported argument.1 |
| **Method Selection** 22 | 44 - 67 | **Weakest:** High failure rate in selecting appropriate statistical tests.22 |

This dimension-level analysis shows that while agents are highly capable of sourcing and citing real documents, they struggle with method selection, often choosing incorrect statistical tests that invalidate their biological interpretations.22 Crucially, switching the underlying agent harness (such as Terminus-2, Claude Code, or Codex CLI) shifts performance by up to 13.5 points—a margin larger than the gap between successive foundation model generations.2 This highlights that system-level scaffolding is just as critical to process alignment as the raw model capabilities.2

## Trajectory-Aware Mutation, Evolutionary Red-Teaming, and Multi-Agent Environments

Evaluating and securing tool-calling agents operating via protocols like the Model Context Protocol (MCP) requires dynamic, execution-driven evaluation architectures.24 Static datasets are vulnerable to memorization and fail to capture the stateful, non-linear interactions of live environments.4 Consequently, modern evaluation frameworks employ trajectory-aware mutation and evolutionary algorithms to proactively discover system vulnerabilities.24

A leading paradigm in this domain is **T-MAP (Trajectory-aware MAP-Elites)**, an evolutionary algorithm designed for red-teaming tool-calling agents.24 Rather than evaluating text-level safety alignment, T-MAP measures success by whether a critical operational vulnerability is realized through actual, multi-step tool executions in the external environment.24 To efficiently navigate the agent's vulnerability landscape, T-MAP maintains a multi-dimensional archive of attack prompts, optimizing them through a four-step iterative cycle guided by execution feedback 24:

- **Cross-Diagnosis (Prompt-Level Mutation):** An LLM Analyst compares successful and failed attack trajectories.24 It extracts success factors from parent trajectories and failure causes from unsuccessful runs, allowing the prompt mutator to inherit effective behavioral framing while discarding logical paths that led to execution failures.24

- **Tool Call Graph (TCG) Optimization (Action-Level Mutation):** Beyond individual trajectories, the system constructs a directed graph representing sequential tool transitions 24:

Here,  represents the set of available tools, and  represents the transition edges between sequential tool calls.24 The mapping function  registers edge-level transition outcomes into a metadata space 24:

where  and  count transition successes and failures, and  and  store the semantic reasons for those outcomes.24 By querying this graph, the prompt mutator bypasses fragile transition sequences, generating prompts that guide the agent along highly stable, multi-step execution paths.24

This evolutionary approach shares structural characteristics with defensive frameworks like **SE-Agent**, which optimizes agent performance through trajectory self-evolution.29 SE-Agent improves upon historical trajectories using three core operations: revision (refining paths via self-reflection), recombination (performing crossovers by fusing successful segments from different trajectories), and refinement (optimizing parameters using reward signals).29

These dynamic methodologies are crucial for evaluation platforms like **SaaSBench** and **Emergence World**.26 SaaSBench evaluates agents across heterogeneous enterprise SaaS environments (spanning multiple databases, programming languages, and frameworks) using a dependency-aware hybrid evaluation paradigm backed by DAG-based validation test networks.30 Emergence World steps outside standard benchmarking to run multi-agent populations continuously for weeks in shared environments.26 This long-horizon execution exposes macroscopic system dynamics—such as behavioral drift, governance collapse, and cross-model contamination—that remain invisible to short-term, static testing.26

These multi-agent dynamics have been formalized using **dynamic social choice theory**, which models long-term alignment under recursive retraining (where models train on their own outputs) as an interaction between Model Owners (filtering outputs to be learned) and Public Users (determining which outputs are retained).31 Using Bradley-Terry preference structures, this analysis reveals three long-term convergence regimes 31:

- **Consensus Collapse:** Complete preference convergence that can lead to model collapse if the diversity of inputs is not maintained.31

- **Compromise on Shared Optima:** Steady-state convergence on mutually acceptable output regions.31

- **Asymmetric Refinement:** Path-dependent optimization that heavily favors the incentives of the dominant feedback loop.31

## Diagnostic Tooling and Architectural Trade-offs

Deploying white-box diagnostics in production requires robust observability pipelines.32 The modern tooling landscape has evolved from simple logging systems to advanced observability platforms that support distributed tracing, prompt management, and automated evaluation.32

| **Platform** | **Core Licensing** | **OpenTelemetry Native** | **Core Process Evaluation Strength** | **Optimal Use Case** |
| --- | --- | --- | --- | --- |
| **LangSmith** 32 | Proprietary / Commercial SaaS 32 | No (Requires specialized SDK wrappers) 34 | Renders detailed non-linear execution trees and offers annotation queues for collaborative human validation of trace runs.4 | Teams utilizing the LangChain or LangGraph stack who require native debugging workflows.32 |
| **Langfuse** 32 | MIT Open Source 32 | Yes 35 | Combines low-overhead tracing, robust prompt version management, and automated evaluation metrics within a single self-hosted platform.32 | Teams seeking complete data sovereignty and a unified interface for tracing and prompt management.32 |
| **Arize Phoenix** 32 | Elastic License 2.0 32 | Yes (Designed natively around OTLP standards) 33 | Features OpenInference support and automated, research-backed evaluators for context and planning diagnostics.33 | ML platforms requiring standards-based tracing and deep integration with existing monitoring infrastructure.33 |
| **TruLens** 32 | MIT Open Source 32 | Yes (Supports standard OpenTelemetry backends) 32 | Focuses heavily on the RAG Triad framework, evaluating context relevance, groundedness, and answer relevance across steps.32 | Evaluation-centric teams seeking to identify localized hallucination and retrieval bottlenecks.33 |

## Strategic Recommendations for Next-Generation Compound AI Systems

Based on the empirical findings of process-level evaluation research, system developers should adopt the following architectural guidelines:

- **Deploy Decomposed Observability Trees:** Rather than using monolithic LLM-as-a-judge models over complete logs, parse trajectories into hierarchical span structures.7 Evaluate leaf spans (such as Tool or LLM calls) independently to prevent context loss and improve error localization.7

- **Mitigate Aggregation Deficits with Structured Memory:** Since integration width () represents the primary bottleneck in repository-level tasks, design explicit memory consolidation layers (such as Stateless Decision Memory or schema-anchored stores) to help agents synthesize parallel inputs.12

- **Prioritize Scaffolding and Harness Optimization:** Because the choice of agent harness can impact performance by up to 13.5 points (surpassing the gains of model generational upgrades), focus engineering efforts on scaffolding design, step-level validation guards, and deterministic planning structures.2

- **Integrate Calibrated Abstention Filters:** For high-stakes, regulated decisioning pipelines, construct explicit abstention filters.8 This allows agents to safely hand off ambiguous or incomplete cases to human operators rather than forcing a decision and risking non-compliance.8

#### Works cited

- BiomniBench: Process-level Evaluation of LLM Agents for Real-world Biomedical Research, accessed May 21, 2026, [https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1](https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1)

- BiomniBench: Process-level Evaluation of LLM Agents for Real-world Biomedical Research - bioRxiv, accessed May 21, 2026, [https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full.pdf](https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full.pdf)

- Beyond the Final Answer: Evaluating the Reasoning Trajectories of Tool-Augmented Agents, accessed May 21, 2026, [https://arxiv.org/html/2510.02837v2](https://arxiv.org/html/2510.02837v2)

- Best LLM Evaluation Tools for AI Agents in 2026 - Confident AI, accessed May 21, 2026, [https://www.confident-ai.com/knowledge-base/compare/best-llm-evaluation-tools-for-ai-agents](https://www.confident-ai.com/knowledge-base/compare/best-llm-evaluation-tools-for-ai-agents)

- Understanding Software Engineering Agents: A Study of Thought-Action-Result Trajectories, accessed May 21, 2026, [https://software-lab.org/publications/ase2025_trajectories.pdf](https://software-lab.org/publications/ase2025_trajectories.pdf)

- How to Trace and Debug Multi-Agent Systems in 2026: A Production Guide with traceAI, OpenTelemetry, and Span-Level Evals - Future AGI, accessed May 21, 2026, [https://futureagi.com/blog/trace-debug-multi-agent-systems-observability-guide/](https://futureagi.com/blog/trace-debug-multi-agent-systems-observability-guide/)

- (PDF) Holistic Evaluation and Failure Diagnosis of AI Agents - ResearchGate, accessed May 21, 2026, [https://www.researchgate.net/publication/404891613_Holistic_Evaluation_and_Failure_Diagnosis_of_AI_Agents](https://www.researchgate.net/publication/404891613_Holistic_Evaluation_and_Failure_Diagnosis_of_AI_Agents)

- Four-Axis Decision Alignment for Long-Horizon Enterprise AI Agents - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2604.19457v1](https://arxiv.org/html/2604.19457v1)

- BiomniBench: Process-level Evaluation of LLM Agents for Real-world Biomedical Research, accessed May 21, 2026, [https://www.researchgate.net/publication/404882712_BiomniBench_Process-level_Evaluation_of_LLM_Agents_for_Real-world_Biomedical_Research](https://www.researchgate.net/publication/404882712_BiomniBench_Process-level_Evaluation_of_LLM_Agents_for_Real-world_Biomedical_Research)

- Holistic Evaluation and Failure Diagnosis of AI Agents - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2605.14865v1](https://arxiv.org/html/2605.14865v1)

- Shir Chorev's research works - ResearchGate, accessed May 21, 2026, [http://www.researchgate.net/scientific-contributions/Shir-Chorev-2217111943](http://www.researchgate.net/scientific-contributions/Shir-Chorev-2217111943)

- From Laboratory to Real-World Applications: Benchmarking Agentic Code Reasoning at the Repository Level - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2601.03731v3](https://arxiv.org/html/2601.03731v3)

- RepoReason: Repository-Level Code Reasoning - Emergent Mind, accessed May 21, 2026, [https://www.emergentmind.com/topics/reporeason](https://www.emergentmind.com/topics/reporeason)

- From Laboratory to Real-World Applications: Benchmarking Agentic Code Reasoning at the Repository Level - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2601.03731v2](https://arxiv.org/html/2601.03731v2)

- (PDF) From Laboratory to Real-World Applications: Benchmarking Agentic Code Reasoning at the Repository Level - ResearchGate, accessed May 21, 2026, [https://www.researchgate.net/publication/399559390_From_Laboratory_to_Real-World_Applications_Benchmarking_Agentic_Code_Reasoning_at_the_Repository_Level](https://www.researchgate.net/publication/399559390_From_Laboratory_to_Real-World_Applications_Benchmarking_Agentic_Code_Reasoning_at_the_Repository_Level)

- From Laboratory to Real-World Applications: Benchmarking Agentic Code Reasoning at the Repository Level - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2601.03731v1](https://arxiv.org/html/2601.03731v1)

- vasundras/decision-alignment-long-horizon-agents - GitHub, accessed May 21, 2026, [https://github.com/vasundras/decision-alignment-long-horizon-agents](https://github.com/vasundras/decision-alignment-long-horizon-agents)

- [2604.19457] Four-Axis Decision Alignment for Long-Horizon Enterprise AI Agents - arXiv, accessed May 21, 2026, [https://arxiv.org/abs/2604.19457](https://arxiv.org/abs/2604.19457)

- Stateless Decision Memory for Enterprise AI Agents - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2604.20158v1](https://arxiv.org/html/2604.20158v1)

- Towards Trajectory-Level Alignment: Detecting Intent Drift in Long-Horizon LLM Dialogues - NeurIPS 2026, accessed May 21, 2026, [https://neurips.cc/virtual/2025/128062](https://neurips.cc/virtual/2025/128062)

- BiomniBench: Process-level Evaluation of LLM Agents for Real-world Biomedical Research, accessed May 21, 2026, [https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full-text](https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full-text)

- BiomniBench: How Do We Know When AI Agents Do Good Biomedical Science? - Sekbio, accessed May 21, 2026, [https://www.sekbio.com/blog-9-bioomnibench-ai-evaluation.html](https://www.sekbio.com/blog-9-bioomnibench-ai-evaluation.html)

- BiomniBench: Process-level Evaluation of LLM Agents for Real-world Biomedical Research, accessed May 21, 2026, [https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full](https://www.biorxiv.org/content/10.64898/2026.05.12.724604v1.full)

- T-MAP: Red-Teaming LLM Agents with Trajectory-aware Evolutionary Search - arXiv, accessed May 21, 2026, [https://arxiv.org/html/2603.22341v1](https://arxiv.org/html/2603.22341v1)

- Paper page - T-MAP: Red-Teaming LLM Agents with Trajectory-aware Evolutionary Search, accessed May 21, 2026, [https://huggingface.co/papers/2603.22341](https://huggingface.co/papers/2603.22341)

- EMERGENCE WORLD: A Laboratory for Evaluating Long-horizon Agent Autonomy, accessed May 21, 2026, [https://www.emergence.ai/blog/emergence-world-a-laboratory-for-evaluating-long-horizon-agent-autonomy](https://www.emergence.ai/blog/emergence-world-a-laboratory-for-evaluating-long-horizon-agent-autonomy)

- T-MAP: Red-Teaming LLM Agents with Trajectory-aware Evolutionary Search - arXiv, accessed May 21, 2026, [https://arxiv.org/pdf/2603.22341](https://arxiv.org/pdf/2603.22341)

- T-MAP: RED-TEAMING LLM AGENTS - OpenReview, accessed May 21, 2026, [https://openreview.net/pdf/ed0ffc5282349ad1563f9efb3c0fa41c9992c6fb.pdf](https://openreview.net/pdf/ed0ffc5282349ad1563f9efb3c0fa41c9992c6fb.pdf)

- Self-Evolution Trajectory Optimization in Multi-Step Reasoning with LLM-Based Agents | OpenReview, accessed May 21, 2026, [https://openreview.net/forum?id=isATAFP71B¬eId=ZHJCVGyBGR](https://openreview.net/forum?id=isATAFP71B&noteId=ZHJCVGyBGR)

- (PDF) SaaSBench: Exploring the Boundaries of Coding Agents in Long-Horizon Enterprise SaaS Engineering - ResearchGate, accessed May 21, 2026, [https://www.researchgate.net/publication/404990560_SaaSBench_Exploring_the_Boundaries_of_Coding_Agents_in_Long-Horizon_Enterprise_SaaS_Engineering](https://www.researchgate.net/publication/404990560_SaaSBench_Exploring_the_Boundaries_of_Coding_Agents_in_Long-Horizon_Enterprise_SaaS_Engineering)

- A Theory of Long-Horizon Alignment Through Recursive Curation - AAAI Publications, accessed May 21, 2026, [https://ojs.aaai.org/index.php/AAAI/article/view/41070/45031](https://ojs.aaai.org/index.php/AAAI/article/view/41070/45031)

- 8 LLM Observability Tools to Monitor & Evaluate AI Agents - LangChain, accessed May 21, 2026, [https://www.langchain.com/articles/llm-observability-tools](https://www.langchain.com/articles/llm-observability-tools)

- LLM Observability Tools for Reliable AI Applications - MachineLearningMastery.com, accessed May 21, 2026, [https://machinelearningmastery.com/llm-observability-tools-for-reliable-ai-applications/](https://machinelearningmastery.com/llm-observability-tools-for-reliable-ai-applications/)

- 10 Best LLM Observability Tools to Track AI Agents in 2026 (Complete Guide) - GoGloby, accessed May 21, 2026, [https://gogloby.com/insights/best-observability-tools-to-track-ai-agents/](https://gogloby.com/insights/best-observability-tools-to-track-ai-agents/)

- Best LLM Observability Tools in 2026 - Firecrawl, accessed May 21, 2026, [https://www.firecrawl.dev/blog/best-llm-observability-tools](https://www.firecrawl.dev/blog/best-llm-observability-tools)

- Top 5 Agent Evaluation Tools in 2026 - MLflow, accessed May 21, 2026, [https://mlflow.org/top-5-agent-evaluation-frameworks/](https://mlflow.org/top-5-agent-evaluation-frameworks/)