# Preserve intent and revise a standard

Find the user decision, current requirement, external contract, or independently worked example that establishes the expected outcome. Code and tests show current behavior. Proposals, archives and previous implementations supply context. For maintenance, preserve observable behavior. When two plausible outcomes affect users differently and the requirement leaves the choice open, show the smallest distinguishing example and ask.

A useful comment explains a demonstrated constraint near the code that depends on it. Put a durable product decision and its rationale where the project keeps approved requirements. Link to that decision when a later maintainer might reasonably reverse it.

## Turn a lesson into a change

Record the task, the decision that went wrong, and the instruction that would have led to a better choice. Decide where the correction belongs. A deterministic source rule fits Abacus. A contract involving timing, persisted state or ownership needs an executable example in the app. An app-specific preference belongs with that app. Generalize the smallest part that transfers to another project.

To evaluate an instruction, give a model the pre-fix source and user-visible symptom. In a fresh context repeat with the revised guidance, using the same model and task. Withhold the fix and regression explanation. Compare the observed behavior, preservation of adjacent cases and unrelated edits. One trial supplies a candidate lesson. Repeat on another case before calling it a general improvement.
