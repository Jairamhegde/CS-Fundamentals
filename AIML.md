## RAG(Retrieval Augmented Reality)
It helps LLMs answer questions using external, potentially private or updated information that may not be present in their training data.

How RAG works

![[Pasted image 20261024004636.png]]

### What are the limitations of RAG?

- Poor retrieval leads to poor answers.
    
- Missing or outdated documents can produce incorrect results.
    
- Relevant context can exceed the model's context window.
    
- RAG reduces hallucinations but does not eliminate them.

## Agentic AI
Agentic AI is AI that can work toward a goal by deciding what steps to take, using tools, observing results, and adjusting its actions until it completes the task or reaches a limitation.
![[Pasted image 20261024012102.png|700]]

`observe -> act -> evaluate -> adapt`


### Agentic AI System Design – Interview Guide (Summary)
> Source: [The Complete Agentic AI System Design Interview Guide (2026)](https://atul4u.medium.com/the-complete-agentic-ai-system-design-interview-guide-2026-f95d0cfeb7cf)

---

### I. Core Concepts & Judgment

#### 1. What makes an AI system truly agentic and what does not qualify?
An agentic system autonomously decides what to do, when to do it and how to adapt based on environmental feedback — all in service of a goal it pursues over multiple steps.
- **Goal-directed autonomy** – receives a high-level objective and determines its own path to achieve it.
- **Environmental interaction** – agents observe, act and adapt; the feedback loop is essential.
- **Temporal extension** – maintains goals and context across multiple steps, not single request-response pairs.
- **What doesn't qualify:** RAG pipelines, single-turn function calling, workflow automation with hardcoded paths, chatbots with personality.
- **Nuance:** "agentic" is a spectrum, not a binary — where to place a system on it is a design choice.

#### 2. When is an agentic architecture the wrong solution?
When the task is well-defined, deterministic and the cost of agent failures exceeds the value of agent flexibility.
- **The problem is actually a workflow** – if you can draw a flowchart, use Temporal, Airflow, or a state machine.
- **Failures are catastrophic and irreversible** – financial, medical, legal; agents hallucinate and misuse tools.
- **Latency requirements are strict** – each reasoning step might take 1–3 seconds.
- **The task requires perfect accuracy** – agents are probabilistic; use them as assistants, not primary actors.
- **You can't define success** – without termination conditions the agent stops too early or runs forever.
- **Red flag:** choosing agents because they're exciting, not because the problem requires autonomy.

#### 3. How do you define and enforce agent autonomy boundaries?
Through explicit permission systems, action classification, budget constraints and human approval gates — all enforced outside the LLM, not by prompting.
- **Layer 1: Action classification** – read-only, reversible-write, irreversible-write, external-communication.
- **Layer 2: Resource budgets** – hard limits on API calls, tokens, money, time, enforced in the orchestrator.
- **Layer 3: Scope constraints** – the agent literally cannot call a tool it doesn't have access to.
- **Layer 4: Approval gates** – high-risk actions route to human review; the agent proposes, a human disposes.
- **What doesn't work:** asking the LLM to respect boundaries via system prompts — boundaries must be structural.
- **Pattern:** a policy engine between the LLM's proposed actions and actual execution.

#### 4. What are the essential components of an agent beyond an LLM?
An orchestrator, tool interface layer, memory systems, policy/guardrails engine and observability infrastructure. The LLM is maybe 20% of a production agent system.
- **Orchestrator / Control Loop** – manages observe → think → act → observe, retries, timeouts, termination.
- **Tool Interface Layer** – standardized schemas, sandboxing, result parsing, error handling.
- **Memory Systems** – working (current task), episodic (past executions), semantic (long-term knowledge).
- **Policy & Guardrails Engine** – enforces boundaries, validates actions, routes approvals.
- **State Management** – checkpointing, resumption after failure, state serialization.
- **Observability Stack** – logging, tracing, metrics of every decision and tool call.
- **Human Interface** – approval workflows, intervention mechanisms, feedback channels.
- *The LLM is the brain, but brains don't survive without bodies.*

#### 5. How do you prevent agents from over-reasoning or over-planning?
Step limits, confidence thresholds, action-bias in prompts and detecting planning loops programmatically.
- **Hard step limits** – enforced in the orchestrator, not suggested in the prompt.
- **Action-biased prompting** – "Take the simplest action that makes progress."
- **Confidence thresholds with defaults** – if it can't decide after N seconds, take a default action or ask.
- **Loop detection** – track semantic similarity of recent thoughts; interrupt and force a decision.
- **Decomposition limits** – "Book a flight" shouldn't decompose into 47 subtasks.

#### 6. How do you explain agentic systems to non-technical stakeholders?
Use the "capable intern" analogy, emphasize the observe-think-act loop, be honest about uncertainty and failure modes and focus on business outcomes.
- **Executives** – an extremely capable intern that needs supervision, clear boundaries and no keys on day one.
- **Product managers** – a loop of observe, decide, act, observe; powerful flexibility, challenging unpredictability.
- **Risk/compliance** – like supervising a contractor: set boundaries, monitor outcomes, intervene when needed.
- **Never** oversell capabilities or hide failure modes.

---

### II. Agent Architecture & Control Plane

#### 7. Walk through a production-ready agent architecture.
Request intake → context assembly → LLM reasoning → action validation → sandboxed execution → result processing → state update → loop or terminate. With observability at every stage.
![[Pasted image 20261025175422.png]]

- **Separation of concerns** – the LLM reasons; the orchestrator controls; the policy engine governs; the sandbox executes.
- **Fail-safe defaults** – every timeout, every limit defaults to the safe option.
- **Complete observability** – every stage emits traces, logs and metrics.
- **Stateless orchestrator** – state lives in external storage; the orchestrator can crash and resume.

#### 8. What logic belongs in the orchestrator vs the LLM?
Orchestrator handles control flow, enforcement and infrastructure concerns. LLM handles reasoning, planning and decision-making within the boundaries the orchestrator enforces.

| Orchestrator                           | LLM                                      |
| -------------------------------------- | ---------------------------------------- |
| Loop control, timeouts, budgets        | Understanding the goal, planning         |
| State persistence & recovery           | Selecting tool, generating arguments     |
| Tool dispatch, error handling, retries | Interpreting results                     |
| Approval routing, observability        | Deciding if task is complete, edge cases |

- **Key principle:** anything that must be guaranteed → orchestrator; anything that requires judgment → LLM.
- **Anti-pattern:** putting control flow in prompts — leads to infinite loops and runaway costs.

#### 9. How do you design a safe and debuggable agent loop?
Explicit state machines, comprehensive logging at decision points, reproducible execution and circuit breakers at multiple levels.
- **State machine clarity** – PLANNING, EXECUTING, WAITING_FOR_APPROVAL, PROCESSING_RESULT, TERMINATED.
- **Decision point logging** – log inputs the LLM saw, the output and how it was interpreted.
- **Reproducibility** – same state snapshot + same LLM call (temperature=0) → same action.
- **Circuit breakers** – iteration limit, time limit, cost limit, error limit.
- **Graceful degradation** – emit status, save state, notify for human review; don't crash silently.
- **Debugging workflow** – find failed run → see what agent saw → understand why → replay with modified inputs.
#### 10. How do you implement termination conditions in long-running agents?
Combine goal-completion detection, iteration limits, time limits and explicit "I'm stuck" recognition — with the orchestrator as the final authority.
- **Layer 1: LLM self-assessment** – separate step: "Is the task complete? Why or why not?"
- **Layer 2: Programmatic verification** – check the result exists; don't trust the agent's claim.
- **Layer 3: Progress detection** – no meaningful environment change in N steps → clarify or terminate.
- **Layer 4: Hard limits** – caps on iterations, time and cost, regardless of what the agent thinks.
- **Layer 5: Stuck detection** – repeated errors, circular reasoning, same tool calls → human review.
- *Most insidious failure: an agent that thinks it's making progress but isn't.*

#### 11. Stateless vs stateful agents — tradeoffs and use cases?
Stateless agents are simpler, scalable and easier to debug. Stateful agents are necessary for long-running tasks, learning and complex context.

| | Stateless | Stateful |
|---|---|---|
| Advantages | Horizontal scale, simple to debug, fault-tolerant | Multi-session tasks, learns, richer context |
| Disadvantages | Context window limits, no learning | State corruption, complex scaling, memory pollution |
| Use when | Single-session tasks, scale matters | Long-running tasks, personalization |

- **Preferred pattern:** stateless execution layer with external state storage.

#### 12. How do you version and roll back agent behavior?
Version the complete configuration: prompts, tool schemas, policies, model version. Maintain rollback capability for each component. Test changes against behavioral benchmarks before deployment.
```yaml
agent-v23:
  prompts: prompts-v12
  tools: tools-v8
  policies: policies-v15
  model: llama3.3:70b
  orchestrator: orchestrator-v6
```
- **Rollback** – keep previous N versions deployable, traffic splitting, instant rollback, backward-compatible state.
- **Behavioral testing** – reference tasks, resource bounds, policy constraints, regressions.
- **Lesson:** model provider updates can change behavior — always pin model versions.

---

### III. Planning, Reasoning & Goal Decomposition

#### 13. How do agents decompose high-level goals into executable steps?
Through recursive decomposition: break the goal into subgoals, break subgoals into actionable steps, execute and adapt. The key is knowing when to stop decomposing and start acting.
- **Process:** goal interpretation → subgoal identification → action planning → dependency analysis → execution with adaptation.
- **Good decomposition:** verifiable subgoals, atomic actions, shallow-enough plans, acknowledged uncertainty.
- **Example (competitor pricing report):** identify competitors → extract pricing → structure data → generate report → verify.

#### 14. Chain-of-thought vs tree-of-thought vs graph planning — when would you use each?
Chain-of-thought for linear problems with clear progression. Tree-of-thought when you need to explore alternatives. Graph planning for complex problems with dependencies and constraints.

| Approach | Use when | Example |
|---|---|---|
| Chain-of-thought | Linear progression, latency matters | Debugging, procedures, arithmetic |
| Tree-of-thought | Multiple approaches, backtracking needed | Strategy selection, puzzles |
| Graph planning | Complex dependencies & constraints | Travel planning, scheduling |

- **Guidance:** start with CoT, escalate to ToT on bad paths, use graph planning only for genuine constraint satisfaction.

#### 15. How do you detect and stop infinite planning loops?
Track reasoning state similarity, enforce step limits, detect repetitive patterns programmatically and require periodic action or termination.
- **Detection:** similarity tracking, pattern matching, progress metrics, state hashing.
- **Stopping:** soft interrupt (inject a message), hard interrupt (stop & escalate), forced action.
- **Prevention:** action-biased prompting, reasonable step limits, clear guidance on when to ask for help.

#### 16. How do you handle partial observability or missing information?
Agents should recognize uncertainty, seek information when available, make reasonable assumptions when not and expose their assumptions to users.
- **Hierarchy:** use tools → ask clarifying questions → make explicit assumptions → express uncertainty.
- **Patterns:** uncertainty propagation, assumption logging, assumption validation checkpoints, graceful degradation.
- *Calibrated uncertainty is a feature, not a weakness.*

#### 17. How do agents decide a task is "done"?
By evaluating whether success criteria are met, verifying outputs where possible and confirming with users when verification isn't possible programmatically.
- **Framework:** explicit success criteria, self-assessment with evidence, programmatic verification, user confirmation, negative case handling.
- **Failure modes:** declaring victory without verifying, stopping at first plausible result, ambiguous goal, over-optimizing.

#### 18. What planning failures are hardest to detect in production?
Silent wrong answers, slow drift from objectives, over-optimization of proxy metrics and confidently wrong assumptions that propagate through the plan.
- **Silent wrong answers** – detect via sampling audits, output validation, user feedback.
- **Goal drift** – periodically re-ground against original objectives.
- **Assumption propagation** – explicit assumption tracking and validation checkpoints.
- **Hidden dependencies** – environmental variation in testing, runtime verification.
- **Local optima** – compare against baselines, periodic re-planning from scratch.

---

### IV. Tool Use & Action Execution

#### 19. How do agents decide which tool to use?
Through a combination of semantic matching (which tools are relevant?), capability reasoning (which tools can accomplish the goal?) and constraint checking (which tools are permitted?).
- **Process:** tool discovery → relevance filtering → capability reasoning → constraint checking → selection & argument generation.
- **Tool descriptions matter enormously** – bad descriptions cause hallucinated tool use.
- **Fewer tools is better** – curate tools per context.
- **Fallback handling** & **tool composition** – recognize no-fit cases; plan multi-tool sequences.

#### 20. How do you design tool schemas that reduce hallucinated actions?
Explicit types, enumerated options, clear descriptions, required fields, examples of valid usage and validation at the schema level.
```json
// Bad
{ "status": "string" }
// Good
{ "status": { "enum": ["pending", "approved", "rejected", "cancelled"] } }
```
- Use enums over strings, require rather than assume, constrain formats, add descriptions & examples.
- Validate before execution, document failure modes, test with adversarial prompts.

#### 21. How do you sandbox tool execution safely?
Defense in depth: isolated execution environments, capability restrictions, resource limits, output validation and fail-safe defaults.
- **Isolation:** process isolation, container isolation, network whitelisting, credential scoping.
- **Resource limits:** CPU/memory, timeouts, rate limiting, I/O limits.
- **Output validation:** match schemas, sanitize before next LLM call, handle error states.
- **Fail-safe defaults:** fail closed; missing permissions = cannot execute; timeout = termination.

#### 22. How do you handle tool failures, retries and idempotency?
Classify failures by type, implement intelligent retry with backoff, ensure idempotent operations where possible and maintain operation logs for recovery.
- **Failures:** transient (retry with backoff), permanent (don't retry), partial (hardest).
- **Retry:** exponential backoff with jitter, max retry count, retry only transient failures.
- **Idempotency:** idempotency keys, check before creating, "ensure state X" rather than "apply change Y".
- **Recovery:** log every tool call, checkpoint before risky operations, compensating transactions.

#### 23. What are the biggest security risks with tool-using agents?
Prompt injection through tool outputs, privilege escalation via tool chains, data exfiltration, unintended actions from hallucinated tools and confused deputy attacks.
- **Prompt injection** – mark tool results as data, not instructions.
- **Privilege escalation** – principle of least privilege; monitor tool combinations.
- **Data exfiltration** – data classification; restrict data flow between tool categories.
- **Hallucinated tools** – strict schema validation; no dynamic tool generation.
- **Confused deputy** – validate actions align with original user intent.

#### 24. How do you control cost explosions from tool calls?
Hard budget limits, per-operation cost tracking, tiered approval for expensive operations, monitoring with automatic circuit breakers and cost-aware tool selection.
```python
class BudgetTracker:
    def request_operation(self, operation, estimated_cost):
        if self.spent + estimated_cost > self.limit:
            raise BudgetExhausted()
    def record_cost(self, actual_cost):
        self.spent += actual_cost
```
- **Budgets:** session, per-user, per-operation.
- **Tiered approval:** low-cost free, medium-cost warning, high-cost human approval.
- **Monitoring:** real-time dashboards, spend-rate alerts, automatic shutdown.
- *Always assume loops will run longer than expected.*

---

### V. Memory Systems & Context Management

#### 25. What types of memory do agentic systems need?
Working memory (current task context), episodic memory (past experiences), semantic memory (learned knowledge) and procedural memory (learned skills/patterns).
- **Working** – high fidelity, limited capacity, cleared between sessions.
- **Episodic** – time-indexed, personal to user/session, queryable by similarity.
- **Semantic** – declarative facts like "The user prefers detailed explanations."
- **Procedural** – how-to knowledge that emerges from successful episodes.
- **Considerations:** not all systems need all types; cold-start problem; memory pollution.

#### 26. How do you design long-term memory without polluting it?
Selective storage, quality filtering, decay mechanisms, validation before retrieval and user control over memory contents.
- **Selective storage** – only confirmed facts, verified patterns, user preferences, summaries.
- **Quality filtering** – verify accuracy, confidence threshold, filter contradictions.
- **Decay** – recency weighting, confidence decay, usage-based retention.
- **Validation at retrieval** – check relevance, consistency, allow override by current info.
- **User control** – see, correct, delete, reset, opt out.

#### 27. When should memory be retrieved vs ignored?
Retrieve when past context would improve the current response. Ignore when it would bias toward outdated patterns or when the current context is sufficient.
- **Retrieve when** – user references the past, preferences matter, context insufficient.
- **Ignore when** – context is sufficient, risk of outdated bias, user wants a fresh start.
- **Strategy:** relevance threshold, recency, source weighting, favor current context on contradiction.
- **Anti-pattern:** retrieving memory on every turn regardless of need.

#### 28. How do embeddings help — and where do they fail?
Embeddings enable semantic search over memory and tools, finding relevant information based on meaning rather than keywords. They fail on precision requirements, negation, recency and multi-hop reasoning.
- **Help:** semantic similarity, scalable search, cross-lingual matching, fuzzy matching.
- **Fail:** precision ("2024 Q3 report"), negation, temporal reasoning, multi-hop reasoning, specific values/IDs.
- **Compensate:** keyword filters, metadata filtering, structured queries, multiple retrieval strategies with fusion.

#### 29. How do you delete or correct agent memory safely?
Soft deletion with audit trails, propagation checking to find derived memories, user confirmation and gradual rollout of corrections.
- **Deletion:** soft delete first, audit trail, propagation analysis, user notification.
- **Correction:** don't overwrite silently, version rather than replace, lower confidence until reconfirmed.
- **Bulk:** gradual rollout, consistency checks, backup and recovery.
- **User-initiated:** clear interface, confirmation step, honor requests promptly (compliance).

---

### VI. Multi-Agent Systems

#### 30. When is multi-agent architecture better than single-agent?
When tasks require genuinely distinct capabilities, when separation improves reliability, when parallel execution is valuable, or when adversarial setups improve quality.
- **Good reasons:** distinct capabilities, reliability through separation, parallel execution, generator/critic, separation of concerns.
- **Bad reasons:** it seems cool, task is actually sequential, to avoid improving prompts.
- **Decision:** can one agent do it? is the limit fundamental? would agents be independent? is coordination worth it?

#### 31. How do agents coordinate without conflicting actions?
Through shared state with concurrency control, message passing with clear protocols, resource locking, conflict detection and resolution and centralized coordination where necessary.
- **Shared state with locking** – simple, but contention and deadlocks.
- **Message passing** – cleaner, more complex implementation.
- **Centralized coordinator** – clear control, single point of failure.
- **Event sourcing** – great audit trail, eventual consistency challenges.
- **Conflicts:** prevention (partition work), detection, resolution (priority/timestamp/escalation).
- **Advice:** start with a central coordinator with explicit turn-taking.

#### 32. What emergent behaviors have you seen in multi-agent systems?
Unexpected cooperation patterns, gaming of evaluation metrics, information hoarding, cascade failures and occasionally genuinely creative solutions that no single agent would produce.
- **Positive:** complementary specialization, error correction, creative solutions.
- **Negative:** metric gaming, information silos, infinite hand-off loops, cascade failures, adversarial dynamics.
- **Manage:** monitor system-level outcomes, adversarial testing, system-wide circuit breakers, regular audits.

#### 33. How do you debug failures across interacting agents?
Distributed tracing with correlation IDs, comprehensive logging at interaction boundaries, replay capability and root cause analysis tools that span agents.
- **Infrastructure:** correlation IDs, interaction logging, state snapshots, causal ordering.
- **Workflow:** identify failure → trace backward → find divergence → root cause → verify with replay.
- **Tooling:** unified log viewer, timeline visualization, correlation-ID filter, diff view, replay.
- *If you can't debug it, you can't run it in production.*

---

### VII. Evaluation, Safety & Reliability

#### 34. How do you evaluate long-horizon agent performance?
Through task completion benchmarks, step efficiency metrics, intermediate checkpoint evaluation, trajectory analysis and comparative evaluation against baselines.
- **Dimensions:** task completion, efficiency, trajectory quality, intermediate milestones, robustness.
- **Methodology:** benchmark suites, A/B testing, human evaluation, failure analysis.
- **Challenges:** few samples per budget, many valid solutions, changing environments, human eval doesn't scale.

#### 35. What metrics matter beyond task success?
Efficiency (steps, cost, time), safety (boundary violations, risky actions), reliability (consistency, failure rate), user experience (satisfaction, intervention rate) and alignment (goal adherence, unexpected behaviors).
- **Efficiency** – tokens, tool calls, wall-clock time, dollar cost; is the agent earning its complexity?
- **Safety** – boundary violations, risky actions proposed, interventions, near-misses.
- **Reliability** – consistency, failure rate, recovery rate, degradation over long conversations.
- **User experience** – satisfaction, abandonment, retry rate, time to value.
- **Alignment & operational** – goal adherence, policy compliance, latency, error rates, availability.

#### 36. How do you detect goal drift or misalignment?
Through explicit goal tracking, periodic re-grounding, divergence metrics, behavioral bounds checking and user feedback integration.
- **Detection:** goal restating, re-grounding prompts, semantic divergence metrics, action distribution monitoring, behavioral bounds, user feedback.
- **Drift patterns:** proxy optimization, scope creep, local minima, mode collapse.
- *Watch for slow changes that accumulate over time.*

#### 37. How do you implement human-in-the-loop controls?
Through approval gates for risky actions, escalation paths for uncertainty, override capabilities, meaningful notification/context for reviewers and feedback integration.
- **Approval gates** – classify risk, present context, approve/reject/modify/escalate, timeout → safe default.
- **Escalation** – uncertainty, anomaly and threshold triggers.
- **Override** – stop execution anytime, correct state and resume.
- **Notification** – don't cry wolf, provide context, make decisions easy.
- **Anti-patterns:** approving everything, approval for low-risk actions, notifications without context.

#### 38. What are the most dangerous failure modes of agentic AI?
Confident wrong actions at scale, goal misalignment with real consequences, security breaches through tool chains, runaway costs and silent failures that compound over time.
- **Confident wrong actions at scale** – calibrated confidence; batch risky actions for review.
- **Goal misalignment** – conservative autonomy for real-world actions.
- **Security breaches** – defense in depth; assume the agent will be manipulated.
- **Runaway costs** – hard budget limits, circuit breakers.
- **Silent failures** – automated quality checks, sampling audits, trend monitoring.
- **Reputation damage** – content filtering, human review for external-facing outputs.

---

### VIII. Scaling, Production & Taste

#### 39. What bottlenecks limit agent scalability in production?
LLM latency and throughput, context window limitations, state management overhead, tool execution bottlenecks and coordination costs in multi-agent systems.
- **LLM** – 1–3+ s per step, rate limits, context size → caching, smaller models, batching.
- **State management** – retrieval latency, serialization, consistency → lazy loading, partitioning.
- **Tool execution** – external API limits, sequential dependencies, sandbox overhead → caching, parallelism.
- **Coordination** – communication, lock contention, consensus → partition work, eventual consistency.
- **Operational** – observability overhead; human approval becomes a bottleneck at scale.

#### 40. What tradeoffs do most teams get wrong when building agents?
Autonomy vs control, capability vs reliability, sophistication vs debuggability and speed-to-market vs production readiness.
- **Autonomy vs control** – start with minimal autonomy, expand based on demonstrated reliability.
- **Capability vs reliability** – prefer agents that do less but do it reliably.
- **Sophistication vs debuggability** – simpler architectures with clear reasoning traces.
- **Speed-to-market vs production readiness** – observability and safety from day one.
- **Building vs buying** – use existing frameworks; build custom only where required.
- **Prompt engineering vs architecture** – prompts can't fix bad tool designs or missing components.

---

### How These Questions Are Used in Interviews
- Pick 3–5 questions and go deep; drill into failure modes, tradeoffs and war stories; expect architecture diagrams.
- **Looking for:** ✅ production experience ✅ safety awareness ✅ strong system design taste ✅ honest uncertainty.

> *The best answers aren't the most confident ones — they show you understand the problem deeply enough to know where the hard parts are.*
