# Internal Personal Loan Underwriting Policy

> **SYNTHETIC INTERNAL DOCUMENT**  
> Created for an academic data-engineering project. This is not an actual lending policy or legal advice.

| Document Metadata | Value |
|---|---|
| Document ID | FIN-POL-001 |
| Document Type | Internal Personal Loan Underwriting Policy |
| Organization | Meridian Financial Services (Synthetic) |
| Owner | Consumer Lending Risk Management |
| Effective Date | January 1, 2026 |
| Version | 3.0 |
| Authority Status | CURRENT |
| Jurisdiction/Region | United States |
| Confidentiality | Internal |
| Access Group | Lending, Risk, Compliance, and Internal Audit |

## 1. Purpose

This policy establishes consistent standards for approving, denying, or escalating personal lending applications. It connects each decision to the applicant data, credit-pull evidence, selected product, triggered rule, and policy version used on the application date.

The policy supports automated underwriting and documented human review while preserving fair-lending controls and decision traceability.

## 2. Scope

This policy applies only to consumer personal lending products identified in `loan_products.csv`:

| Product Code | Product Category | Policy Scope | Primary Disclosure Framework |
|---|---|---|---|
| MNT-01 | Term/installment personal loan | In scope | Regulation Z closed-end credit rules, including 12 CFR 1026.18 |
| REV-02 | Revolving personal line of credit | In scope | Regulation Z open-end credit rules; 12 CFR 1026.18 does not apply |

This policy does not cover mortgages, auto loans, student loans, home-equity products, or small-business loans. An application for a product outside this scope must be assigned `Under Review` and routed to the appropriate underwriting procedure.

## 3. Governing Data and Source Priority

The underwriting decision must use structured records whose identifiers link across the following datasets:

1. `applicants.csv` provides applicant identity, income, employment, baseline credit profile, residence, and application date.
2. `credit_pull_events.csv` provides product-specific credit-pull results, DTI, bureau score, risk status, and provider tracking evidence.
3. `loan_products.csv` provides the selected product's credit-risk criteria, amount limit, rate, and permitted term range.
4. `loan_applications.csv` records the outcome, approved terms, supporting document, policy version, and primary rule that determined or escalated the decision.

For a final underwriting decision, the current credit evidence is the latest active credit-pull record that:

- Matches `applicant_id` and `product_code` on the application.
- Has `active_flag = TRUE`.
- Has `pull_status = SUCCESS`.
- Occurred no earlier than the application date and no later than the decision date.

The profile-level `applicants.credit_score` may support initial screening, but it must not replace a valid product-specific credit pull for the final decision. A missing, failed, pending, inactive, or conflicting credit pull requires `Under Review`.

## 4. Structured Data Dependencies

| Policy Requirement | Dataset Field(s) | Underwriting Use |
|---|---|---|
| Applicant identity | `applicants.applicant_id` | Links the applicant, credit pull, and application |
| Application timing | `applicants.application_date`; `loan_applications.decision_date` | Selects the applicable policy and eligible credit-pull window |
| Minimum age | `applicants.birth_date`; `applicants.application_date` | Confirms the applicant is at least 18 on the application date |
| Annual income | `applicants.income` | Applies the general minimum-income rule |
| Employment history | `applicants.employment_duration`; `applicants.employment_status` | Determines automatic eligibility or manual review |
| Current credit score | `credit_pull_events.credit_score` | Assigns the current underwriting risk category |
| Credit-pull validity | `pull_status`; `failure_code`; `active_flag`; `provider_request_id`; `bureau_name` | Confirms that the credit evidence is complete and traceable |
| Debt-to-income ratio | `credit_pull_events.dti_ratio`; `loan_products.max_dti_score` | Compares applicant DTI with the selected product limit |
| Product selection | `loan_applications.product_code`; `loan_products.product_code` | Applies the correct product rules |
| Credit-risk eligibility | `credit_pull_events.risk_status`; `loan_products.risk_status_criteria` | Compares current risk status with product eligibility criteria |
| Requested amount | **Field missing from supplied dictionary**; `loan_products.max_amount` | Required to compare the requested principal or limit with the product maximum |
| Requested term | **Field missing from supplied dictionary**; `min_term_month`; `max_term_month` | Required to compare the requested term with the permitted range |
| Decision traceability | `decision_status`; `doc_id`; `policy_doc_id`; `rule_id_triggered` | Records the outcome and its supporting authority |

`approved_amount` and `approved_term_length` are outputs of underwriting. They must not be substituted for missing requested values because doing so would hide whether the original request exceeded a product limit.

## 5. Applicant Eligibility Rules

### 5.1 Minimum Age

The applicant must be at least 18 years old on `application_date`.

- If the calculated age is at least 18, continue the review.
- If the calculated age is below 18, assign `Denied` with rule ID `UW-AGE-001`.
- If `birth_date` or `application_date` is missing or invalid, assign `Under Review` with rule ID `UW-DATA-001`.

Age beyond the legal-capacity check must not be used to disadvantage an applicant.

### 5.2 Minimum Income

The applicant must have annual income of at least **$40,000** for automatic approval eligibility.

- If `income` is at least $40,000, continue the review.
- If `income` is below $40,000, assign `Under Review` with rule ID `UW-INC-001`.
- If income is missing, invalid, or cannot be supported when verification is required, assign `Under Review` with rule ID `UW-DATA-002`.

Income below the automatic threshold does not by itself require an automatic denial.

### 5.3 Employment History

The applicant should have at least **12 months** with the current employer for automatic approval eligibility.

- If `employment_duration` is at least 12 months, continue the review.
- If it is less than 12 months, assign `Under Review` with rule ID `UW-EMP-001`.
- `employment_status` must be evaluated consistently. Self-employed and retired applicants must not be denied solely because of status; appropriate income documentation may be requested under the applicable procedure.

## 6. Credit Risk Assessment

### 6.1 Credit Score and Risk Mapping

The decision score must come from the qualifying record in `credit_pull_events.credit_score`. The system must use the following Version 2.0 mapping:

| Credit Score | Applicant Credit Tier | Credit-Pull Risk Status |
|---:|---|---|
| 750-850 | Excellent | Low Risk |
| 670-749 | Good | Moderate Risk |
| 580-669 | Fair | High Risk |
| 300-579 | Subprime | Very High Risk |
| Missing, invalid, or conflicting | Not determinable | Review |

The calculated category must agree with `credit_pull_events.risk_status` and, when used for comparison, `applicants.credit_risk_tier`. A disagreement requires `Under Review` with rule ID `UW-CRD-002`.

### 6.2 Product Risk Eligibility

The current `credit_pull_events.risk_status` must satisfy `loan_products.risk_status_criteria` for the selected `product_code`.

- If the risk criteria are satisfied, continue the review.
- If the criteria explicitly prohibit approval, assign `Denied` with rule ID `UW-CRD-001`.
- If the criteria are ambiguous, missing, or cannot be parsed reliably, assign `Under Review` with rule ID `UW-DATA-003`.

### 6.3 Credit-Pull Exceptions

Assign `Under Review` when:

- `pull_status` is `FAILED` or `PENDING`.
- A failed pull lacks `failure_code`.
- A successful pull does not contain a valid score from 300 through 850.
- `provider_request_id` is missing or duplicated.
- No active credit pull matches the applicant and selected product.
- A material downgrade occurred after the application but before the decision.

Use rule ID `UW-CRD-003` for a failed or pending pull and `UW-CRD-004` for missing or conflicting credit-pull evidence.

## 7. Debt-to-Income Review

Compare `credit_pull_events.dti_ratio` with `loan_products.max_dti_score` for the selected product.

- If DTI is at or below the product maximum, continue the review.
- If DTI exceeds the product maximum, assign `Under Review` with rule ID `UW-DTI-001`.
- If either value is missing or outside its valid range, assign `Under Review` with rule ID `UW-DATA-004`.

A high DTI ratio does not by itself produce an automatic denial under this policy.

## 8. Amount and Term Review

The requested principal or credit limit must not exceed `loan_products.max_amount`, and the requested term must fall between `min_term_month` and `max_term_month` when a term applies.

The supplied data dictionary does not contain `requested_amount` or `requested_term_length` in `loan_applications.csv`. Until those fields are added and populated:

- The system must not infer requested values from `approved_amount` or `approved_term_length`.
- The system cannot issue a fully automated approval.
- The application must be assigned `Under Review` with rule ID `UW-DATA-005`.

After the requested fields are added:

- A request within the product amount and term limits may continue through automated review.
- A request exceeding `max_amount` must be assigned `Under Review` with rule ID `UW-AMT-001`.
- A requested term outside the permitted range must be assigned `Under Review` with rule ID `UW-TRM-001`.

## 9. Decision Outcomes

The value stored in `loan_applications.decision_status` must use exactly one of the values defined in the data dictionary:

| Decision Status | Policy Meaning |
|---|---|
| Approved | All automatic approval requirements are satisfied, all required evidence is valid, and no escalation rule is triggered |
| Denied | A policy rule explicitly requires denial and the principal reason is documented |
| Under Review | Information is missing, a threshold requires judgment, evidence conflicts, a fairness concern exists, or a manual-review rule is triggered |

An automated `Approved` decision is permitted only when:

1. All required structured fields are complete and valid.
2. The applicant satisfies the age and automatic eligibility rules.
3. The selected product exists in `loan_products.csv` and is in scope.
4. The current credit pull is successful, active, product-matched, and traceable.
5. The credit-risk criteria are satisfied.
6. DTI is within the product maximum.
7. The requested amount and term are within product limits.
8. No manual-review or compliance escalation rule applies.

When all eight conditions are satisfied, record `UW-APP-001` as the primary approval rule.

## 10. Decision Reason and Evidence Requirements

Every decision must be traceable through `loan_applications`:

| Field | Required Entry |
|---|---|
| `decision_status` | Approved, Denied, or Under Review |
| `decision_date` | Date and time the decision was recorded |
| `policy_doc_id` | `FIN-POL-001` for decisions governed by this version |
| `doc_id` | Identifier for the principal supporting rule, procedure, or authoritative source document |
| `rule_id_triggered` | Primary machine-readable rule that approved, denied, or escalated the application |
| `approved_amount` | Approved principal or credit limit; zero when no amount is approved |
| `approved_term_length` | Approved term in months; zero when no term is approved or no term applies |

If multiple rules apply, `rule_id_triggered` must contain the rule that principally determined the outcome. Supporting rules should be preserved in an audit log or a separate application-rule bridge table when available.

A denied application must use a specific and accurate principal reason that reflects the factor actually evaluated. A generic, approximate, or unrelated reason must not be substituted merely because it is the closest available label.

## 11. Fair-Lending Controls

### 11.1 Prohibited Use

Except for the minimum-age legal-capacity check, protected demographic characteristics must not be used as approval, denial, pricing, amount, or term criteria.

`region`, `state`, `housing_status`, and employment information must not be used as substitutes or proxies for prohibited characteristics. Any geographic restriction must be legally reviewed, product-specific, documented, and applied consistently.

### 11.2 Consistent Treatment

Similarly situated applicants must be evaluated under the same policy version and product rules. Differences in outcome, pricing, approved amount, term, documentation requests, or manual overrides must be supported by legitimate and documented underwriting factors.

### 11.3 Compliance Review

Assign `Under Review` and refer the case to Compliance with rule ID `UW-FL-001` when:

- Similar applicants appear to have received materially different treatment without a documented underwriting reason.
- A manual override lacks evidence or an identifiable approving reviewer.
- A prohibited factor or potential proxy appears to have influenced the decision.
- The decision cannot be supported by the structured data and cited policy documents.
- An adverse decision reason does not match the factor actually considered.

Protected-group attributes collected for compliance testing must be access-restricted and kept separate from automated underwriting inputs.

The supplied data dictionary includes age through `birth_date`, but it does not include most other protected-group attributes needed for a complete fair-lending comparison. The project must either use a separately governed compliance dataset or disclose that the available structured data cannot fully test consistency across all protected demographic groups.

## 12. Manual Review and Overrides

Manual reviewers must document all identified rule failures, validate the supporting data, and apply the current policy and product rules.

For every override, the audit record should preserve:

- Original decision.
- Final decision.
- Primary and supporting rule IDs.
- Specific reason for the override.
- Evidence reviewed.
- Reviewer identifier.
- Review date.
- Applicable policy and product-rule versions.

The supplied `loan_applications.csv` structure does not include override-specific fields. If overrides are part of the project analysis, add an override or decision-event table rather than overwriting the original automated decision.

## 13. Policy Version and Application-Date Control

The underwriting system must select the policy version effective on `applicants.application_date`, not merely the version current on `loan_applications.decision_date`.

For applications dated on or after January 1, 2026, the system must record:

```text
policy_doc_id = FIN-POL-001
```

Historical applications must retain the policy identifier and rule version actually in effect on their application dates. A document marked `SUPERSEDED`, `DRAFT`, or `EXPIRED` must not govern a current application unless an authorized exception is documented.

## 14. Data Quality and Audit Controls

The system must prevent an automated approval when:

- A required primary or foreign key is missing.
- `applicant_id` or `product_code` fails referential-integrity checks.
- Dates occur in an impossible order.
- Numeric values are outside their valid ranges.
- Decision fields use values outside the data dictionary.
- `policy_doc_id`, `doc_id`, or `rule_id_triggered` is missing.
- A source record required for the decision is inactive, failed, pending, duplicated, or inconsistent.

Data corrections must preserve the original value, corrected value, correction date, and responsible party in an auditable history when available.

## 15. Related Documents

- `FIN-RULE-001` - Personal Loan Product Eligibility Rules, Version 2026.1
- `FIN-PROC-001` - Personal Loan Approval and Escalation Procedure
- `FIN-COMP-001` - Fair-Lending Review Memorandum
- `FIN-POL-001-OLD` - Internal Personal Loan Underwriting Policy, Version 1.0, SUPERSEDED
- 12 CFR Part 1002 - Equal Credit Opportunity Act (Regulation B)
- 12 CFR 1026.18 - Regulation Z Content of Disclosures for Closed-End Credit
- CFPB Circular 2023-03 - Adverse Action Notifications and Complex Credit Models

## 16. Version History

| Version | Status | Effective Period | Summary |
|---|---|---|---|
| 1.0 | SUPERSEDED | Historical period before January 1, 2026 | Prior personal-loan underwriting requirements |
| 2.0 | CURRENT | January 1, 2026 onward | Personal-loan-only scope, structured-data mapping, traceability rules, and fair-lending controls |

---

**Document Status:** CURRENT  
**End of Synthetic Internal Document**
