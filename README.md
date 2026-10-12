# Evaluating an Enterprise AI Support Assistant
### A Risk-Based Evaluation Framework for Groundedness, Correctness, and Instruction Following

## Overview

Enterprise AI assistants must do more than generate plausible answers. They must use approved information, handle uncertainty appropriately, respect confidentiality boundaries, and follow prescribed workflows.

This project explores how a structured evaluation process can identify and reduce response failures in an enterprise support assistant through:

- Risk-based test design
- A 10-case golden evaluation dataset
- Human evaluation and LLM-as-a-Judge
- Failure-pattern analysis
- Targeted prompt refinement
- Before-and-after evaluation
- Adversarial testing and regression checks

**Project status:** Portfolio evaluation prototype. Not a production deployment.

**Organisation:** Amexa IT, a hypothetical enterprise IT services organisation.

## 1. The Problem

The assistant was designed to answer internal questions about project status, approval stages, budget alignment, release timelines, and resource-related information using a supplied source of truth.

The initial prompt instructed the assistant to answer using only the provided information and remain concise. However, these broad instructions did not sufficiently specify how the assistant should handle missing information, mandatory redirection, or attempts to override its instructions.

The product question was:

**Can targeted prompt refinements improve the assistant's responses against explicit, testable quality criteria?**

The evaluation focused on five key risk areas:

- Unsupported factual claims and hallucinations
- Missing or ambiguous information
- Confidentiality and information boundaries
- Incorrect handling of resource and JIRA-related queries
- Adversarial instructions attempting to override established rules

## 2. Evaluation Design

### Golden dataset

I created an initial golden dataset of 10 test cases covering representative enterprise-support scenarios.

The dataset was intentionally small to establish an initial evaluation baseline, validate the scoring rubric, and identify recurring failure patterns before expanding test coverage. It is not intended to represent the full range of production queries.

Each test case included a question, expected behaviour, response, and evaluation criteria.

### Evaluation metrics

Each response was assessed against three binary metrics.

| Metric | Evaluation question |
|---|---|
| Correctness | Does the response satisfy the expected behaviour for the test case? |
| Groundedness | Are all factual claims supported by the supplied information? |
| Instruction Following | Does the assistant follow the required rules without guessing or inventing information? |

Each metric was scored as `1` for a pass or `0` for a fail.

A test case was considered a pass only when all three metrics passed.

This strict criterion recognises that an answer can sound plausible while still violating a critical requirement.

### Evaluation approach

Both versions were assessed using human evaluation and LLM-as-a-Judge against the same three quality dimensions.

The process followed four stages:

1. **Establish a baseline:** Evaluate the initial assistant responses against the golden dataset.
2. **Diagnose failures:** Identify recurring issues in correctness, grounding, and instruction following.
3. **Refine instructions:** Add explicit handling rules for missing information, confidentiality, redirection, and unsupported assumptions.
4. **Re-evaluate:** Assess the revised assistant against the same test scenarios and rubric.

## 3. Baseline Findings: V1

The initial evaluation exposed several weaknesses.

### Failure 1: Treating missing information as evidence

When asked whether new projects were under discussion, the assistant stated that no new projects were identified and implied that no additional projects had been confirmed.

**Why it matters:** Missing information does not establish that something does not exist.

**Required behaviour:** Acknowledge the information gap without making an unsupported positive or negative claim.

### Failure 2: Ignoring mandatory redirection

For questions about resource joining dates and FTE requirements, the assistant explained that the data was unavailable but did not consistently follow the prescribed HR redirection. A JIRA-related attendance query also failed to follow the required Team Lead redirection.

**Why it matters:** A safe response is not necessarily a correct response if it ignores the intended workflow.

**Required behaviour:** Apply the designated fallback or redirection for the relevant query category.

### Failure 3: Introducing unnecessary or unsupported details

Some responses included additional project information, release dates, or tables that were not required to answer the question.

**Why it matters:** Extra detail increases the opportunity for unsupported claims and makes answers less concise.

**Required behaviour:** Provide only relevant, supported information and distinguish known facts from unknowns.

### Failure 4: Inconsistent handling of adversarial instructions

The evaluation also examined whether the assistant would follow attempts to override its instructions or disclose confidential information.

**Why it matters:** Enterprise assistants must preserve their intended behaviour even when users request restricted information or attempt to change the assistant's rules.

**Required behaviour:** Maintain the established constraints and avoid disclosing information that is unavailable or restricted.

## 4. Improvements Implemented: V2

Based on the observed failure patterns, I refined the assistant's instructions to make the expected behaviour more explicit.

| Improvement | Intended effect |
|---|---|
| Source-of-truth enforcement | Restrict factual answers to the supplied information |
| Explicit uncertainty handling | Use a defined fallback when information is unavailable |
| Mandatory redirection | Route resource and FTE queries to HR and unavailable JIRA-related queries to Team Leads |
| Assumption correction | Politely correct incorrect assumptions rather than accepting them |
| Confidentiality boundaries | Avoid unsupported disclosure of confidential or employee-related information |
| Instruction robustness | Resist attempts to override established instructions |
| Concise responses | Reduce unnecessary explanations and unrelated details |

The objective was not simply to make responses shorter. It was to make the expected behaviour more consistent, measurable, and testable.

## 5. Evaluation Results: V1 vs. V2

The workbook records human and LLM-as-a-Judge scores for both evaluation rounds.

The table below reports the mean of the two evaluators' scores for each quality dimension.

| Quality dimension | V1 | V2 | Change |
|---|---:|---:|---:|
| Correctness | 80% | 90% | +10 percentage points |
| Groundedness | 84.5% | 100% | +15.5 percentage points |
| Instruction Following | 60% | 94.5% | +34.5 percentage points |
| Composite quality score | 74.83% | 94.83% | +20 percentage points |

**How to interpret these results**

The composite quality score is the arithmetic mean of the three quality-dimension scores. It is not the proportion of test cases that passed all three metrics.

The strongest improvement was in instruction following, consistent with the addition of explicit fallback, redirection, and uncertainty-handling rules.

Groundedness also improved in the recorded evaluation, while correctness improved more modestly.

These results describe performance on the initial 10-case evaluation set only. They do not establish statistical significance, generalised reliability, or production readiness.

### Evaluator interpretation

Human evaluation provides a direct review of responses against the defined rubric, while LLM-as-a-Judge offers a complementary assessment.

Neither should automatically be treated as ground truth. The evaluation process would benefit from documented adjudication of disagreements, further rubric validation, and additional test coverage.

## 6. Adversarial Testing and Regression Checks

After the second evaluation, I tested additional prompts designed to expose potential weaknesses beyond the original evaluation scenarios.

Examples included:

- Requests for confidential compensation information
- Questions about undisclosed clients
- Attempts to override the assistant's instructions
- Requests for current external information, such as the temperature in Mumbai

The external-information test exposed a regression: the assistant initially generated a current temperature even though the task required it to answer only from the supplied context. An additional check confirmed the need for a fallback response when the answer was unavailable in the approved information.

This highlighted an important distinction:

**Passing a fixed evaluation set does not guarantee that an assistant will behave correctly on new or adversarial inputs.**

Newly discovered failures should be converted into regression tests so that subsequent prompt changes can be checked against previously observed issues.

## 7. Proposed Release Readiness Criteria

For a real enterprise deployment, aggregate quality scores alone would not be sufficient.

The following are proposed criteria for future validation, not evidence that the prototype is production-ready.

- **Critical-risk handling:** Investigate and remediate unauthorised disclosure and successful instruction-override attempts.
- **Quality thresholds:** Define acceptance thresholds for correctness, groundedness, and instruction following before release evaluation.
- **Regression testing:** Re-run the golden dataset after material prompt or system changes.
- **Evaluator validation:** Review human–judge disagreements and validate the scoring rubric.
- **Dataset expansion:** Add test cases based on newly discovered failure modes and representative user queries.
- **Operational validation:** Evaluate access controls, monitoring, and behaviour under representative operating conditions.

A release decision should depend on meeting defined acceptance criteria and resolving critical risks, not merely achieving a high aggregate score.

## 8. Limitations and Next Steps

This project is an initial evaluation prototype with several limitations:

- The golden dataset contains only 10 test cases.
- The results are specific to the scenarios evaluated.
- The evaluation does not establish statistical significance or generalised model reliability.
- Human and LLM-judge agreement requires further validation.
- The scoring rubric and calculation process should be independently audited before the results are used for formal release decisions.
- No production deployment, automated CI/CD evaluation pipeline, or live monitoring system is claimed.

### Next steps

1. Expand the dataset using risk-based prioritisation and observed failure patterns.
2. Add worked evaluation examples showing the input, expected behaviour, response, individual scores, and scoring rationale.
3. Document the LLM-as-a-Judge rubric and validate its agreement with human reviewers.
4. Automate repeatable evaluation and regression checks.
5. Test more complex multi-part questions, conflicting source information, and adversarial scenarios.
6. Define measurable release gates for critical risks and quality dimensions.

## 9. Key Learnings

- AI product quality requires explicit acceptance criteria, not subjective impressions of answer quality.
- Groundedness and correctness are related but distinct: a response can avoid unsupported claims and still fail to answer the intended question.
- Missing information should trigger the correct fallback behaviour rather than speculation.
- Domain-specific redirection is part of the product's expected behaviour, not merely a conversational preference.
- LLM-as-a-Judge can support evaluation, but its outputs and scoring criteria require validation.
- Small golden datasets are useful starting points, but they must expand as new failure modes emerge.
- Prompt refinement should be evaluated through repeatable tests rather than anecdotal improvements.
- Evaluation results must be reported with transparent calculation methods and limitations.

## Conclusion

This project demonstrates an iterative approach to enterprise AI quality: define expected behaviour, evaluate representative risk scenarios, diagnose failure patterns, refine instructions, and re-evaluate against explicit criteria.

The central product lesson is that an enterprise AI assistant should not be judged only by whether its answers sound plausible. It should be evaluated on whether it uses approved information, handles uncertainty appropriately, respects confidentiality boundaries, and follows the intended workflow.

**Project note:** Amexa IT is a hypothetical organisation created for this portfolio case study. All results are from a controlled evaluation prototype and should be interpreted within the scope and limitations described above.
