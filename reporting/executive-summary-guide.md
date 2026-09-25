# Executive Summary Guide

## Objective

The executive summary lets a non-technical decision-maker answer four questions:

1. What was tested?
2. What material risk was found?
3. What could happen to the business?
4. What should happen next?

Write it after the detailed findings are complete.

## Recommended Structure

### 1. Overview

State the assessment type, objective, dates, major in-scope systems, and important coverage limitations.

### 2. Overall Result

Describe the security posture without claiming absolute security. Summarize the number and distribution of findings only when the counts help decision-making.

### 3. Business Impact

Explain credible outcomes such as unauthorized transactions, loss of customer data, operational interruption, regulatory exposure, or compromise of privileged functions.

### 4. Remediation Direction

Identify immediate actions and longer-term themes. State whether remediation is primarily targeted fixes, architectural work, or program-level improvement.

## Useful Pattern

> An authorized [assessment type] of [scope] was performed to determine whether [business objective]. Testing identified [key result]. The most significant exposure could allow [credible business impact]. [Material limitation, if any]. Priority should be given to [immediate action], followed by [systemic improvement] and validation through retesting.

## Executive Summary Rules

- Use business language.
- Explain acronyms on first use.
- Mention only findings that materially affect the conclusion.
- State attack chains in outcome-oriented language.
- Include meaningful limitations.
- Do not include payloads or step-by-step exploitation.
- Do not say “secure,” “safe,” or “no vulnerabilities exist.”
- Do not exaggerate beyond the demonstrated or well-supported impact.

## Findings and Recommendations Section

A separate security-stakeholder section can add:

- severity distribution;
- recurring control weaknesses;
- related findings and attack chains;
- remediation dependencies;
- recommended priority order;
- ownership and retest strategy.

Individual ratings remain isolated; this section explains why combinations or systemic weaknesses may change remediation priority.
