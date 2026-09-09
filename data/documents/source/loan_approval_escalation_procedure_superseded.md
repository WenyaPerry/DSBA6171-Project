# SYNTHETIC INTERNAL DOCUMENT

## Loan Approval and Escalation Procedure

**Document ID:** FIN-PROC-001
**Document Type:** Procedure
**Organization:** Meridian Financial Services (Synthetic)
**Owner:** Consumer Lending Operations
**Effective Date:** January 1, 2025
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
