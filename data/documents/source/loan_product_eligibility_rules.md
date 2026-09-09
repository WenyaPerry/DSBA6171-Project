# Personal Loan Product Eligibility Rules

> **SYNTHETIC INTERNAL DOCUMENT**  
> Created for the DSBA 6171 applied machine learning project. It is not an actual lending policy and must not be used for real credit decisions.

| Document control | Value |
|---|---|
| Document ID | FIN-RULE-001 |
| Document type | Personal Loan Product Eligibility Rules |
| Owner | Consumer Lending Risk Management |
| Effective date | January 1, 2026 |
| Version | 2 |
| Authority status | APPROVED |
| Jurisdiction | United States |
| Related policy | FIN-POL-001, Internal Personal Loan Underwriting Policy, Version 2026.1 |
| Related procedure | FIN-PROC-001, Loan Approval and Escalation Procedure |
| Structured source | `loan_products.csv` |

## 1. Purpose and scope

This document defines product-level eligibility limits for Meridian Financial Services personal-loan products. It supports decisions to approve, deny, or place an application under review based on the selected product, the applicant's current credit risk, and the policy version effective on the application date.

Only consumer personal-loan products are in scope. Auto loans, mortgages, student loans, and small-business loans are excluded.

## 2. Separation of policy and product rules

The underwriting policy owns the universal rules for calculating debt-to-income ratio, validating income and employment, mapping credit scores to risk statuses, handling missing or conflicting data, and selecting the policy version by effective date.

The product table owns only product-specific limits: the highest permissible risk status, base rate, maximum amount, permitted term range, and maximum DTI.

The `risk_status_criteria` field must contain one controlled risk-status value for each product. It must not repeat numerical credit-score ranges. Those score bands belong in the underwriting policy.

### 2.1 Interpretation of `risk_status_criteria`

`risk_status_criteria` is the highest risk status eligible for normal product consideration. A lower-risk status also satisfies the criterion.

| Stored value | Eligible current `risk_status` values | Automatic eligibility for `Very High Risk` or `Review` |
|---|---|---|
| Low Risk | Low Risk | No |
| Moderate Risk | Low Risk; Moderate Risk | No |
| High Risk | Low Risk; Moderate Risk; High Risk | No |

`Very High Risk` is not an automatic-eligibility threshold for any product. A `Review` value is an operational exception status, not a credit-risk grade; it always produces `Under Review` until resolved.

## 3. Current product eligibility table

The values below are synthetic. The column names and value formats are aligned to `loan_products.csv`.

| product_code | risk_status_criteria | base_rate | max_amount | product_name | min_term_month | max_term_month | max_dti_score |
|---|---|---:|---:|---|---:|---:|---:|
| MNT-01 | Moderate Risk | 0.0550 | 50000.00 | Standard Term Personal Loan | 24 | 60 | 36.50 |
| REV-02 | Low Risk | 0.1250 | 25000.00 | Revolving Personal Line of Credit | 24 | 72 | 40.00 |
| MNT-03 | Moderate Risk | 0.0695 | 60000.00 | Debt Consolidation Personal Loan | 24 | 72 | 38.00 |
| MNT-04 | High Risk | 0.1695 | 10000.00 | Emergency Expense Personal Loan | 6 | 24 | 45.00 |
| MNT-05 | High Risk | 0.0495 | 75000.00 | Secured Personal Loan | 24 | 84 | 42.00 |
| MNT-06 | High Risk | 0.0895 | 30000.00 | Medical Expense Personal Loan | 12 | 60 | 43.00 |
| MNT-07 | Moderate Risk | 0.0795 | 75000.00 | Home Improvement Personal Loan | 24 | 84 | 38.00 |
| MNT-08 | Moderate Risk | 0.0745 | 40000.00 | Professional Development Personal Loan | 12 | 60 | 40.00 |
| MNT-09 | High Risk | 0.1895 | 15000.00 | Credit Builder Personal Loan | 6 | 24 | 45.00 |
| MNT-10 | Low Risk | 0.0450 | 100000.00 | Prime Personal Loan | 24 | 84 | 32.00 |

## 4. Product-specific rationale

### 4.1 MNT-01 — Standard Term Personal Loan

- General-purpose installment loan for applicants no riskier than `Moderate Risk`.
- Retains the existing 5.50% base rate, $50,000 maximum, 24–60 month term, and 36.50% maximum DTI.

### 4.2 REV-02 — Revolving Personal Line of Credit

- Restricted to `Low Risk` because the borrower can draw funds repeatedly up to the credit limit.
- Retains the existing 12.50% base rate, $25,000 limit, and 40.00% maximum DTI.
- The term fields represent the permitted account review or repayment-plan period in the synthetic dataset.

### 4.3 MNT-03 — Debt Consolidation Personal Loan

- Available to applicants no riskier than `Moderate Risk`.
- Allows a longer maximum term and moderately higher amount than the standard product to consolidate multiple debts.
- Approval still requires verified payoff information when debt consolidation is the stated purpose.

### 4.4 MNT-04 — Emergency Expense Personal Loan

- Permits applicants up to `High Risk`, but limits exposure through a $10,000 cap and a 24-month maximum term.
- A higher base rate reflects the broader risk eligibility in this synthetic product design.

### 4.5 MNT-05 — Secured Personal Loan

- Permits applicants up to `High Risk` because acceptable collateral reduces loss exposure.
- Collateral type, ownership, valuation, and lien status must be verified before approval.
- If the structured data does not contain collateral evidence, the application must remain `Under Review`.

### 4.6 MNT-06 — Medical Expense Personal Loan

- Permits applicants up to `High Risk` with a $30,000 cap.
- Medical purpose must not be inferred from protected or health-related information; it must be supplied by the applicant and used only for product administration.

### 4.7 MNT-07 — Home Improvement Personal Loan

- This is an unsecured personal loan, not a mortgage or home-equity product.
- Available to applicants no riskier than `Moderate Risk`, with a 38.00% maximum DTI.

### 4.8 MNT-08 — Professional Development Personal Loan

- Covers applicant-selected training, certification, or career-development expenses.
- It is a personal loan and not a student loan.
- Available to applicants no riskier than `Moderate Risk`.

### 4.9 MNT-09 — Credit Builder Personal Loan

- Designed for applicants up to `High Risk`, with a low amount cap and short permitted term.
- `Very High Risk` still requires review or denial under the underwriting policy; the product name does not override that rule.

### 4.10 MNT-10 — Prime Personal Loan

- Restricted to `Low Risk` applicants.
- Provides the lowest synthetic base rate, largest maximum amount, and strictest DTI threshold.

## 5. Eligibility evaluation sequence

For every application, the decision process must perform these steps in order:

1. Confirm `loan_applications.product_code` matches one active row in `loan_products.csv`.
2. Select the underwriting and product-rule versions effective on `application_date`.
3. Obtain the latest active, successful credit pull available as of the decision time.
4. Validate that its `risk_status` is one of: `Low Risk`, `Moderate Risk`, `High Risk`, `Very High Risk`, or `Review`.
5. Compare the application's current `risk_status` with the product's `risk_status_criteria` using Section 2.1.
6. Compare the current `dti_ratio` with `max_dti_score`.
7. Compare the requested amount—not the final approved amount—with `max_amount`.
8. Compare the requested term—not the final approved term—with `min_term_month` and `max_term_month`.
9. Apply the income, employment-duration, identity, documentation, and other general rules in FIN-POL-001.
10. Record exactly one supported result in `loan_applications.decision_status`: `Approved`, `Denied`, or `Under Review`.
11. Store the supporting `doc_id`, `policy_doc_id`, and `rule_id_triggered` so an auditor can reproduce the result.

No product attribute alone guarantees approval. All applicable underwriting rules must pass.

## 6. Decision and reason-code rules

| Condition | Required decision status | Minimum evidence | Recommended rule or reason code |
|---|---|---|---|
| Product code does not exist or is inactive | Under Review | Submitted product code and product-table lookup | UW-APP-001 |
| Current risk status is `Review` or cannot be validated | Under Review | Credit-pull status, score, and risk-status derivation | UW-CRD-001 |
| Current risk exceeds the product threshold | Denied, unless an authorized policy exception applies | Current risk status and product criterion | UW-CRD-001 |
| DTI exceeds the product maximum | Under Review or Denied as directed by FIN-POL-001 | DTI calculation inputs and product maximum | UW-DTI-001 |
| Requested amount exceeds product maximum | Under Review or Denied as directed by FIN-POL-001 | Requested amount and product maximum | UW-AMT-001 |
| Requested term is outside the product range | Under Review | Requested term and product term limits | UW-TRM-001 |
| Required application field is missing or conflicting | Under Review | Missing-field or conflict record | UW-APP-001 |
| All product and policy rules pass | Approved | Complete application snapshot and rule results | UW-ELG-001 |

The process must never infer an approval from `approved_amount` or `approved_term_length`. Those are recorded outcomes, not application inputs.

## 7. Structured-data alignment and required changes

### 7.1 Confirmed fields

The supplied data dictionary confirms the following decision inputs or evidence fields exist:

- `applicants.csv`: `income`, `employment_duration`, `credit_score`, `credit_risk_tier`, `region`, `state`, and `application_date`.
- `credit_pull_events.csv`: `product_code`, `dti_ratio`, `credit_score`, `pull_status`, `risk_status`, `event_date`, and `active_flag`.
- `loan_applications.csv`: `product_code`, `decision_date`, `decision_status`, `approved_amount`, `approved_term_length`, `doc_id`, `policy_doc_id`, and `rule_id_triggered`.
- `loan_products.csv`: all eight columns used in the product table in Section 3.

### 7.2 Required schema corrections

Before the rules are used for model evaluation or automated decisions, make these schema changes:

- Add `requested_amount DECIMAL(12,2)` to `loan_applications.csv`. The current dictionary contains only `approved_amount`, which cannot prove whether the original request exceeded a product maximum.
- Add `requested_term_length INT` to `loan_applications.csv`. The current dictionary contains only `approved_term_length`, which cannot show whether the requested term was eligible.
- Revise the data-dictionary definition of `loan_products.risk_status_criteria` to: **Highest permissible underwriting `risk_status` for normal product eligibility; lower-risk statuses also qualify. Allowed values: Low Risk, Moderate Risk, or High Risk.**
- Expand the `loan_products.product_code` definition to list MNT-01, REV-02, and MNT-03 through MNT-10, or state that the valid values are maintained in FIN-RULE-001.
- Add a document-version or effective-date mechanism for `loan_products.csv` if historical applications must be reproduced after product rules change.

## 8. Data-quality and fair-lending controls

- Use product codes, risk statuses, and decision statuses exactly as defined in the data dictionary; do not create spelling or capitalization variants.
- Do not use `birth_date`, region, state, or another protected or proxy attribute to make a credit-risk decision unless a documented legal or operational rule specifically requires it.
- Region and state may be used only for documented product availability, licensing, servicing, or compliance requirements—not as a substitute for creditworthiness.
- Apply the same rule sequence and thresholds to similarly situated applicants.
- Log manual overrides, original outcomes, final outcomes, reviewers, timestamps, and reasons.
- Route unexplained outcome differences or inconsistent overrides to fair-lending compliance review.

## 9. Version control

This document is Version 2026.1 and is marked `APPROVED` for the synthetic 2026 corpus. Historical decisions must use the version effective on the application date. Superseded versions must remain available for audit and must not be silently overwritten.

END OF SYNTHETIC INTERNAL DOCUMENT
