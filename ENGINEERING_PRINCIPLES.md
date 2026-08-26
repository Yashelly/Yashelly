# Engineering Principles

These are the principles I use when building and changing software, especially systems that combine automation, AI agents, external integrations, and real operational consequences.

They are not a framework I try to apply mechanically. They are defaults for how I investigate problems, evaluate risk, work with coding agents, and decide whether a change is actually an improvement.

## 1. Ground truth over assumptions

I do not treat documentation, status fields, model output, or my own memory as ground truth when the real state can be checked directly.

If a claim matters, I prefer to re-derive it from the code, runtime data, logs, Git history, or another reliable source. A system can be internally consistent and still be wrong about what actually happened.

## 2. Root cause over local patches

A failing example is usually the start of an investigation, not the definition of the fix.

Before changing code, I want to know which layer created the bad outcome and whether the same mechanism can affect other cases. If several similar bugs keep appearing, I start questioning the abstraction rather than adding another exception.

## 3. Measure before changing

I try not to justify a system change from one convincing example.

First I want a baseline: how often the problem occurs, what the current false-positive and false-negative behavior looks like, and what population is actually affected. After the change, I want to compare against that baseline rather than rely on a few successful test cases.

## 4. FP and FN are not equally expensive

I do not optimize around raw error count alone.

A false positive that creates an unnecessary review and a false negative that silently loses a valid action may both count as one error, while having completely different consequences. I care about the direction and expected cost of the failure, then bias the system toward the safer side.

## 5. Prefer reversible failures

When two failure modes are possible, I prefer the one that can still be corrected.

A manual review, explicit escalation, safe fallback, or known residual is often better than a silent wrong action that has already affected a user, external system, or irreversible state.

The question is not only "can this fail?" but "what happens after it fails?"

## 6. Unknown is better than fabricated certainty

If a system does not have enough evidence to know something, I would rather make that uncertainty explicit than manufacture a confident answer.

This is especially important around extracted facts, identifiers, routing decisions, money, permissions, and external actions. A visible unknown can be handled. Fabricated certainty contaminates everything downstream.

## 7. Hard constraints upfront, quality ceiling discovered

When I work with coding agents, I define hard boundaries early: what must not break, what is out of scope, which failure modes are unacceptable, and where human approval is required.

I do not always define a fixed quality target upfront.

A target such as 95% can easily become the stopping point even if 99% is achievable with another reasonable iteration. I prefer to investigate and measure first, then let the achievable quality bar emerge from the evidence and trade-offs.

## 8. Adversarial verification before confidence

"It should work" is not a useful confidence level.

After a solution appears correct, I want another pass whose job is to break it: counterexamples, neighboring cases, negative cases, regression cases, and alternative explanations.

If an independent verifier can refute the first solution, the refutation wins.

## 9. Population before anecdote

A real bug matters, but it does not automatically tell me how large the problem is or what the right fix should be.

I prefer to use individual cases to discover a pattern, then inspect the wider population before deciding scope and priority. This helps avoid optimizing a system around unusually memorable examples.

## 10. Minimize blast radius

I prefer changes whose effect can be reasoned about locally.

Additive, scoped, monotonic, feature-gated, or otherwise bounded changes are easier to validate and roll back than broad rewrites. If a fix can preserve unrelated behavior exactly, that is a strong property.

Small blast radius is not about avoiding change. It is about making change easier to prove.

## 11. Explicit input over inference

When a reliable source explicitly provides a value, I do not want a model or heuristic silently replacing it with a guess.

Inference is useful for missing information. It should not casually override known information unless there is strong evidence that the input itself is invalid.

## 12. Explainability is part of correctness

For important automated decisions, I want to be able to reconstruct why an outcome happened.

That means being able to trace the relevant input, the signals or rules that mattered, the processing stage that changed the state, and the final action. If a system produces good aggregate metrics but individual decisions cannot be explained, operating and improving it becomes much harder.

## 13. Treat memory as context, not source of truth

I do not try to remember every implementation detail of a large codebase.

I keep the higher-level model in my head: important invariants, risky boundaries, major decisions, and why they were made. Exact implementation state should be checked when needed.

My memory is useful context. The system itself is the source of truth.

## 14. Externalize implementation state

Implementation history is cheap to store and expensive to keep mentally synchronized.

Git, code, runtime data, tests, and agent context are better places for exact state such as what changed, whether something was already fixed, and how a particular path behaves today.

I would rather spend human attention on judgment, trade-offs, and the relationships between parts of the system.

## 15. Use AI agents for exploration, not only execution

I do not see coding agents only as faster code generators.

They are useful for parallel investigation, tracing unfamiliar paths, comparing hypotheses, generating adversarial cases, implementing bounded changes, and independently verifying another agent's work.

The human role shifts toward setting constraints, maintaining the broader system model, evaluating evidence, and making the decisions that connect local work into a coherent whole.

## 16. Prefer structural signals over growing allowlists

If a solution requires adding another keyword or exception every time a similar bug appears, I start looking for a more structural discriminator.

Stable properties of the input or system are usually more robust than a growing list of literals. Repeated edge cases often indicate that the representation is wrong, not that the list is still incomplete.

## 17. Fix the layer that is actually broken

A bad final result does not mean every upstream representation is wrong.

I prefer to reconstruct the pipeline and identify where a correct value first becomes incorrect. Fixing the wrong layer can solve one example while creating regressions for every other consumer of that state.

## 18. Keep systems simple until complexity proves necessary

I prefer the simplest mechanism that explains the evidence and satisfies the constraints.

When a fix starts accumulating clever exceptions, I would rather reconsider the model than keep patching it. Complexity should pay for itself through measurable improvement, not exist because a more elaborate solution feels sophisticated.

## 19. Separate investigation, implementation, and release

I like clear boundaries between understanding a problem, deciding what to change, implementing it, verifying it, and releasing it.

During investigation, changing the system can destroy evidence or anchor the analysis around the first plausible fix. During release, rollback and human approval matter more than proving that an agent can act autonomously end to end.

## 20. Optimize expected damage, not raw accuracy

Accuracy is useful, but it is not the objective by itself.

The real objective is a system whose mistakes are rare, visible, bounded, and biased toward outcomes the surrounding process can recover from.

That means considering probability, business impact, reversibility, detectability, and downstream effects together rather than treating every error as equivalent.

---

## How this changes the way I work

In practice, these principles usually lead me to the same sequence:

1. Start from a real failure or measurable problem.
2. Reconstruct what actually happened from reliable evidence.
3. Find the layer and mechanism responsible for the outcome.
4. Measure the wider population before choosing scope.
5. Compare failure directions and define the hard safety boundaries.
6. Let the solution space stay open long enough to find the best achievable result.
7. Implement the smallest change that addresses the actual mechanism.
8. Try to break it with adversarial and regression cases.
9. Keep the change explainable, observable, and reversible.
10. Stop when further improvement is no longer worth the added risk or complexity.

## On AI-assisted engineering

The biggest change AI agents made to my workflow is not typing speed. It is how much investigation and local context can be externalized and run in parallel.

I can keep the system-level invariants and decision boundaries in my head while agents work through different local problems. That only works if their output is treated as evidence to review, not authority to trust automatically.

For me, the useful model is:

- **human:** objectives, constraints, system model, risk judgment, final decisions;
- **agents:** investigation, local context, implementation, counterexamples, verification;
- **code / Git / runtime data:** factual source of truth.

That separation lets me use agents aggressively without giving up technical control.
