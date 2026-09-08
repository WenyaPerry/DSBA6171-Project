# SYNTHETIC INTERNAL DOCUMENT

## Loan Approval and Escalation Procedure

**Document ID:** FIN-PROC-001
**Document Type:** Procedure
**Organization:** Meridian Financial Services (Synthetic)
**Owner:** Consumer Lending Operations
**Effective Date:** January 1, 2026
**Version:** 1.0
**Authority Status:** APPROVED
**Jurisdiction/Region:** United States
**Confidentiality:** Internal
**Access Group:** Lending Operations, Risk, Compliance, and Audit

---

# 1. Purpose

This procedure establishes the workflow for reviewing, approving, declining, or escalating loan applications.

The procedure is intended to ensure that loan decisions are based on the current underwriting policy and applicable loan-product requirements.

---

# 2. Application Intake

When an application is received, the system should assign a unique applicant and application identifier.

The system should verify that the required application fields are populated.

Required information includes:

* Applicant identifier
* Income
* Employment duration
* Credit or risk tier
* Region
* Requested loan amount
* Requested loan product

Applications with missing required information should be placed into **MANUAL_REVIEW** status.

---

# 3. Applicant Verification

The reviewer or automated system should verify:

1. Applicant information is internally consistent.
2. Income information is available.
3. Employment duration is available.
4. Credit information is available.
5. The requested loan amount is valid.
6. The selected product exists in the current product catalog.

If verification fails, the application should not proceed directly to approval.

---

# 4. Credit Risk Assessment

The system should identify the applicant's current credit or risk tier.

Credit-related events should be reviewed when available.

Examples of events requiring attention include:

* Credit-tier change
* Credit-score change
* Failed credit pull
* Credit-pull status marked incomplete
* Conflicting credit information

A material change in credit information may require reassessment of the underwriting decision.

---

# 5. Product Eligibility Review

The system must identify the loan product requested by the applicant.

The system should retrieve the current product eligibility rules using the product code.

The following characteristics should be compared:

| Applicant Attribute | Product Requirement       |
| ------------------- | ------------------------- |
| Credit Tier         | Minimum eligible tier     |
| Annual Income       | Minimum income            |
| Requested Amount    | Maximum loan amount       |
| Debt-to-Income      | Maximum permitted ratio   |
| Region              | Eligible region           |
| Review Level        | Standard or manual review |

The current product rules must be used for the decision.

---

# 6. Underwriting Review

The system or reviewer should compare the applicant's information with the current underwriting policy.

The reviewer should determine whether the application:

* Meets all automatic approval requirements.
* Fails one or more requirements.
* Requires additional information.
* Requires compliance review.
* Requires manual underwriting.

## 6.1 Decision Reason-Code Table

The following rule IDs provide standardized, traceable reasons for approving, denying, or escalating an application. The primary applicable rule ID must be recorded in `loan_applications.rule_id_triggered`.

| Rule ID | Description | Applicable Outcome | Evidence Required | Manual Review Required |
|---|---|---|---|---|
| UW-APP-001 | All automatic approval requirements are satisfied | Approved | Complete applicant, credit-pull, product, and application records | No |
| UW-AGE-001 | Applicant is below the minimum age of 18 | Denied | `birth_date` and `application_date` | No |
| UW-INC-001 | Annual income is below the automatic approval threshold | Under Review | `applicants.income` and supporting income documentation | Yes |
| UW-EMP-001 | Employment duration is below 12 months | Under Review | `employment_duration`, `employment_status`, and supporting documentation | Yes |
| UW-CRD-001 | Credit risk does not satisfy the selected product criteria | Denied | Credit score, risk status, product code, and product risk criteria | No |
| UW-CRD-002 | Credit score, credit tier, or risk status is inconsistent | Under Review | Applicant profile and credit-pull records | Yes |
| UW-CRD-003 | Credit pull failed or remains pending | Under Review | `pull_status`, `failure_code`, and `provider_request_id` | Yes |
| UW-CRD-004 | Credit-pull evidence is missing, inactive, duplicated, or conflicting | Under Review | Credit-pull event history and active status | Yes |
| UW-DTI-001 | DTI exceeds the selected product maximum | Under Review | `dti_ratio` and `max_dti_score` | Yes |
| UW-AMT-001 | Requested amount exceeds the selected product maximum | Under Review | Requested amount and `loan_products.max_amount` | Yes |
| UW-TRM-001 | Requested term is outside the permitted product range | Under Review | Requested term, `min_term_month`, and `max_term_month` | Yes |
| UW-DATA-001 | Birth date or application date is missing or invalid | Under Review | Missing-field or validation report | Yes |
| UW-DATA-002 | Income information is missing, invalid, or unsupported | Under Review | Income field and verification evidence | Yes |
| UW-DATA-003 | Product credit-risk criteria are missing or ambiguous | Under Review | Product record and applicable eligibility rules | Yes |
| UW-DATA-004 | DTI information or the product DTI limit is missing or invalid | Under Review | Credit-pull and product records | Yes |
| UW-DATA-005 | Requested amount or requested term is unavailable | Under Review | Application record and missing-field report | Yes |
| UW-FL-001 | Potential fair-lending inconsistency or prohibited-factor concern | Under Review and Compliance Escalation | Comparison results, policy references, and audit evidence | Yes |

When multiple rules apply, `rule_id_triggered` must contain the rule that principally determined the outcome. Supporting rule IDs should be preserved in an audit log or application-rule bridge table when available.

A denied application must use a specific and accurate reason that reflects the factor actually evaluated. A generic or approximate reason must not be substituted merely because it is the closest available code.
---

# 7. Decision Categories

## 7.1 Automatic Approval

An application may receive an **APPROVED** decision when:

* Required information is present.
* The applicant meets the product eligibility requirements.
* The applicant satisfies current underwriting requirements.
* No escalation condition is present.

## 7.2 Automatic Decline

An application may receive a **DECLINED** decision when an applicable underwriting requirement clearly prevents approval and no manual-review condition requires further evaluation.

The decision must be traceable to the applicable rule.

## 7.3 Manual Review

An application should receive a **MANUAL_REVIEW** status when:

* Required information is missing.
* The application exceeds an automated threshold.
* The requested amount exceeds $50,000.
* Credit information contains a material inconsistency.
* The product requires manual review.
* Current and historical documents appear to conflict.
* A compliance concern is identified.

---

# 8. Escalation

## 8.1 High-Value Applications

Applications above $50,000 should be escalated to a lending reviewer.

## 8.2 Compliance Concerns

Applications involving potential compliance concerns should be escalated to the Compliance team.

## 8.3 Policy Conflicts

When two documents provide conflicting underwriting requirements, the reviewer must verify:

1. Effective date.
2. Version.
3. Authority status.

The document marked **CURRENT** should be used for the current decision unless a documented exception applies.

## 8.4 Missing Documentation

Applications with missing information should remain in manual review until the required information is obtained or the application is otherwise resolved according to applicable procedures.

---

# 9. Final Decision Documentation

Every finalized application should contain:

* Final decision
* Decision date
* Applicable product
* Applicable underwriting policy version
* Primary decision reason
* Manual-review indicator, when applicable
* Escalation reason, when applicable

The final decision should be traceable to the evidence used during review.

---

# 10. Audit Review

The Lending Operations team should periodically review loan decisions for:

* Incorrect application of policy versions
* Missing decision documentation
* Unresolved manual reviews
* Product eligibility inconsistencies
* Credit-event inconsistencies
* Potential fair-lending concerns

**Document Status:** APPROVED
