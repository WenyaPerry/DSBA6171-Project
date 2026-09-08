# Internal Personal Loan Underwriting Policy

> **SYNTHETIC INTERNAL DOCUMENT - SUPERSEDED**  
> Historical policy retained for academic retrieval, audit, and version-control testing.  
> **Do not use this version for applications dated on or after January 1, 2026.**

| Document Metadata | Value |
|---|---|
| Document ID | FIN-POL-001-OLD |
| Document Type | Internal Personal Loan Underwriting Policy |
| Organization | Meridian Financial Services (Synthetic) |
| Owner | Consumer Lending Risk Management |
| Effective Date | January 1, 2024 |
| End Date | December 31, 2025 |
| Version | 1.0 |
| Authority Status | SUPERSEDED |
| Superseded By | FIN-POL-001, Version 2.0 |
| Jurisdiction/Region | United States |
| Confidentiality | Internal |
| Access Group | Lending, Risk, Compliance, and Internal Audit |

## 1. Purpose

This historical policy established the standards used by Meridian Financial Services to approve, deny, or manually review consumer personal lending applications submitted from January 1, 2024 through December 31, 2025.

It is retained so historical decisions can be evaluated against the rules that were in effect on their application dates. It must not be treated as current policy.

## 2. Scope

Version 1.0 applied to the following consumer personal lending products:

| Product Code | Product Category | Policy Scope |
|---|---|---|
| MNT-01 | Term/installment personal loan | In scope |
| REV-02 | Revolving personal line of credit | In scope |

The policy did not apply to mortgages, auto loans, student loans, home-equity products, or small-business loans.

## 3. Historical Source Data

Under Version 1.0, underwriting used the applicant profile, available credit information, selected product record, and application record.

1. `applicants.csv` supplied applicant identity, income, employment information, profile credit score, credit tier, residence, and application date.
2. `credit_pull_events.csv` supplied available bureau-pull details and DTI information.
3. `loan_products.csv` supplied the selected product's risk criteria, maximum amount, base rate, and term range.
4. `loan_applications.csv` recorded the final decision and approved terms.

Version 1.0 permitted the profile-level `applicants.credit_score` to serve as the primary decision score when a newer successful bureau pull was unavailable. Version 2.0 discontinued this practice and requires a valid, active, product-matched credit-pull event for a final automated decision.

## 4. Historical Applicant Eligibility

### 4.1 Minimum Age

The applicant was required to be at least 18 years old on the application date.

- An applicant under age 18 was assigned `Denied` with historical rule ID `UW1-AGE-001`.
- Missing or invalid age information required `Under Review` with rule ID `UW1-DATA-001`.
- Age beyond the legal-capacity check was not an approval or pricing factor.

### 4.2 Minimum Income

The Version 1.0 minimum annual income for automatic approval eligibility was **$35,000**.

- Income of at least $35,000 satisfied the general income screen.
- Income below $35,000 required `Under Review` with rule ID `UW1-INC-001`.
- Missing or unsupported income required `Under Review` with rule ID `UW1-DATA-002`.

Version 2.0 increased the automatic approval threshold to $40,000.

### 4.3 Employment History

Version 1.0 generally required at least **6 months** with the applicant's current employer for automatic approval eligibility.

- Employment duration of less than 6 months required `Under Review` with rule ID `UW1-EMP-001`.
- Self-employed and retired applicants were eligible for review when permitted income documentation was available.

Version 2.0 increased the general employment-duration standard to 12 months.

## 5. Historical Credit-Risk Rules

### 5.1 Version 1.0 Risk Mapping

| Credit Score | Applicant Credit Tier | Historical Risk Status |
|---:|---|---|
| 720-850 | Excellent | Low Risk |
| 660-719 | Good | Moderate Risk |
| 580-659 | Fair | High Risk |
| 300-579 | Subprime | Very High Risk |
| Missing or invalid | Not determinable | Review |

These ranges applied only during the Version 1.0 effective period. Version 2.0 introduced revised score ranges and must be used for applications dated on or after January 1, 2026.

### 5.2 Product Eligibility

The assigned risk status was compared with the applicable `loan_products.risk_status_criteria` for the selected `product_code`.

- A qualifying risk status allowed the application to continue.
- A risk status expressly prohibited by the applicable product rule resulted in `Denied` with rule ID `UW1-CRD-001`.
- Missing or conflicting credit information resulted in `Under Review` with rule ID `UW1-CRD-002`.

### 5.3 Credit-Pull Exceptions

A failed or pending credit pull required `Under Review` when the available applicant profile could not support a reliable preliminary assessment. The historical escalation code was `UW1-CRD-003`.

Version 1.0 did not consistently require the credit pull to be active and product-matched. Version 2.0 added those controls.

## 6. Historical Debt-to-Income Standard

The applicant's `credit_pull_events.dti_ratio` was compared with `loan_products.max_dti_score` when a product-specific limit was available.

- DTI at or below the product limit satisfied the screen.
- DTI above the product limit required `Under Review` with rule ID `UW1-DTI-001`.
- When a product-specific limit was unavailable, Version 1.0 used a general manual-review threshold of **45%**.

A high DTI ratio did not by itself require automatic denial.

## 7. Historical Amount and Term Review

The requested amount was compared with `loan_products.max_amount`, and the requested term was compared with `min_term_month` and `max_term_month` when applicable.

- A request exceeding the product maximum required `Under Review` with rule ID `UW1-AMT-001`.
- A requested term outside the permitted range required `Under Review` with rule ID `UW1-TRM-001`.
- Applications requesting more than **$40,000** required manual review even when the product permitted a higher amount.

Historical requested amounts and terms may have been retained in supporting application documents rather than dedicated structured fields. `approved_amount` and `approved_term_length` must not be treated as proof of the original requested values.

## 8. Historical Decision Outcomes

| Decision Status | Historical Meaning |
|---|---|
| Approved | The Version 1.0 automatic requirements were satisfied and no manual-review rule was triggered |
| Denied | A Version 1.0 rule explicitly prohibited approval and the principal reason was documented |
| Under Review | Information was missing, a threshold required judgment, or an escalation condition existed |

An approval under Version 1.0 does not establish eligibility under Version 2.0 because the income, employment, credit-score, and credit-pull requirements changed.

## 9. Decision Documentation

Historical decisions governed by this policy should record:

| Field | Historical Entry Requirement |
|---|---|
| `decision_status` | Approved, Denied, or Under Review |
| `decision_date` | Date and time the decision was recorded |
| `policy_doc_id` | `FIN-POL-001-OLD` |
| `doc_id` | Principal supporting policy, rule, or source document |
| `rule_id_triggered` | Primary Version 1.0 rule responsible for the outcome |
| `approved_amount` | Approved amount, or zero when no amount was approved |
| `approved_term_length` | Approved term, or zero when no term was approved or applicable |

The application date determines whether this policy is applicable. The decision date alone must not be used to select a policy version.

## 10. Fair-Lending Controls

Applicants with materially similar credit and financial characteristics were required to be evaluated under the same Version 1.0 rules.

Except for the minimum-age legal-capacity requirement, protected demographic characteristics were prohibited as approval, denial, amount, term, or pricing factors. A potential prohibited-factor concern, unexplained difference in treatment, or unsupported manual override required compliance review under historical rule ID `UW1-FL-001`.

Protected-group data collected for compliance testing was required to remain separate from automated underwriting inputs.

## 11. Manual Review and Overrides

Manual reviewers were required to document the primary reason for review, the evidence considered, the final decision, the reviewer, and the policy version.

An override could not be supported solely by personal judgment. It required a documented, legitimate underwriting reason applied consistently to similarly situated applicants.

## 12. Supersession and Retrieval Rules

This policy was superseded by `FIN-POL-001`, Version 2.0, effective January 1, 2026.

- Use this policy only for applications with `application_date` from January 1, 2024 through December 31, 2025.
- Do not use this policy for a current application.
- Do not use Version 2.0 to retroactively judge whether a historical decision followed the policy in effect at the time.
- When retrieved with Version 2.0, the system must prefer the policy whose effective period contains the application's date.
- If the application date is missing or falls outside the documented periods, assign `Under Review` and obtain the correct policy evidence.

## 13. Material Changes Introduced by Version 2.0

| Policy Area | Version 1.0 - SUPERSEDED | Version 2.0 - CURRENT |
|---|---|---|
| Minimum income for automatic eligibility | $35,000 | $40,000 |
| General employment-duration standard | 6 months | 12 months |
| Excellent/Low Risk starting score | 720 | 750 |
| Good/Moderate Risk starting score | 660 | 670 |
| Primary final-decision score | Applicant profile permitted when newer pull unavailable | Valid active product-matched credit pull required |
| High-amount manual-review threshold | More than $40,000 | Product maximum and current escalation rules |
| Decision traceability | Basic policy and rule reference | Expanded data-source, evidence, and rule traceability |

## 14. Related Documents

- `FIN-POL-001` - Internal Personal Loan Underwriting Policy, Version 2.0, CURRENT
- `FIN-PROC-001` - Personal Loan Approval and Escalation Procedure
- `FIN-COMP-001` - Fair-Lending Review Memorandum
- 12 CFR Part 1002 - Equal Credit Opportunity Act (Regulation B)

---

**Authority Status:** SUPERSEDED  
**Historical Effective Period:** January 1, 2024 through December 31, 2025  
**End of Synthetic Internal Document**
