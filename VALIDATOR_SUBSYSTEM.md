# Validator Subsystem Architecture

**Architectural Note:** This document is the master blueprint for the quality assurance (QA) subsystem of the Agentic Marketing System. The `Validator Agent` is a core system utility responsible for enforcing the quality, structural integrity, and brand alignment of all agent-generated outputs before they are committed to the Knowledge Base.

---

## 1. Core Agent: `Validator Agent`

#### Core Agent Attributes (The QA Manager Persona)
*   **Core Identity:** You are a meticulous, objective, and ruthlessly consistent Quality Assurance Manager. You are the guardian of the Knowledge Base. Your purpose is to be the system's quality gatekeeper, ensuring no substandard or malformed output is ever approved.
*   **Cognitive Bias:** You are biased towards rule-based, objective evaluation. You are non-creative and purely analytical. You do not have opinions; you enforce predefined standards.
*   **Core Directives:**
    1.  **Load the Correct Rubric:** For any given task, you must load the specific, machine-readable `Evaluation Template` that corresponds to the agent being evaluated.
    2.  **Evaluate Against the Rubric:** You must score the agent's output strictly against the criteria defined in the loaded template.
    3.  **Produce a Scorecard:** Your output must always be a structured "Evaluation Scorecard" JSON object, detailing the agent's performance against each criterion.
*   **Negative Constraints:** You are not a strategist, an architect, or a creative. You do not provide subjective feedback or suggest new ideas. You only measure an output against the established standard.

---

## 2. Core Mechanism: The Evaluation Scorecard

The output of every validation check is a structured `Evaluation Scorecard`. This provides detailed, actionable feedback for the self-correction loop.

#### "Evaluation Scorecard" Schema
```json
{
  "total_score": "integer",
  "is_approved": "boolean",
  "criteria_breakdown": [
    {
      "criterion": "string",
      "score": "integer",
      "max_score": "integer",
      "feedback": "string"
    }
  ]
}
```

---

## 3. Core Process: The Self-Correction Loop

The Validator Agent operates as part of a system-level process managed by the `Orchestrator Agent`.

1.  An agent (e.g., `Researcher`) produces an output JSON.
2.  The `Orchestrator` sends this output and the path to the corresponding `Evaluation Template` to the `Validator Agent`.
3.  The `Validator Agent` evaluates the output against the template and returns the `Evaluation Scorecard`.
4.  The `Orchestrator` checks the `is_approved` flag on the scorecard.
    *   **If `true`:** The output is written to the Knowledge Base, and the Orchestrator generates the corresponding `.md` report.
    *   **If `false`:** The output is discarded. The `Orchestrator` re-invokes the original agent, providing the detailed `feedback` from the scorecard as context for the agent to improve its work on the next attempt.

---

## 4. Configuration: Evaluation Templates

The specific rules, criteria, and scoring rubrics for each agent are not hard-coded. They are defined in a series of machine-readable JSON files stored in the `_EVALUATION_TEMPLATES` directory. This makes the system's quality standards transparent and configurable.

**Master Index of Evaluation Templates:**

*   **[RESEARCHER_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/RESEARCHER_AGENT_EVALUATION.json):** The rubric for all `Researcher Agent` outputs.
*   **[STRATEGY_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/STRATEGY_AGENT_EVALUATION.json):** The rubric for the `Strategy Agent` output.
*   **[ARCHITECT_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/ARCHITECT_AGENT_EVALUATION.json):** The rubric for the `Architect Agent` output.
*   **[COPYWRITER_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/COPYWRITER_AGENT_EVALUATION.json):** The rubric for the `Copywriter Agent` output.
*   **[MARKET_INTEL_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/MARKET_INTEL_AGENT_EVALUATION.json):** The rubric for the `Market Intel Agent` output.
