# Personal Loan Approval and Escalation Procedure

> **SYNTHETIC INTERNAL DOCUMENT**  
> Created for an academic data-engineering project. This is not an actual lending procedure or legal advice.

| Document Metadata | Value |
|---|---|
| Document ID | FIN-PROC-001 |
| Document Type | Personal Loan Approval and Escalation Procedure |
| Organization | Meridian Financial Services (Synthetic) |
| Owner | Consumer Lending Operations |
| Effective Date | January 1, 2026 |
| Version | 2.0 |
| Authority Status | APPROVED |
| Jurisdiction/Region | United States |
| Confidentiality | Internal |
| Access Group | Lending Operations, Credit Risk, Compliance, and Internal Audit |

## 1. Purpose

This procedure defines the operational steps for reviewing a personal lending application and assigning one of the decision statuses permitted by the structured data:

- `Approved`
- `Denied`
- `Under Review`

It is designed to answer the following primary business question:

> Should a personal loan application be approved, denied, or escalated for manual review based on the applicant's credit risk, the selected loan product, and the underwriting policies in effect on the application date?

It also supports the traceability question:

> Can the decision be explained and supported by accurate application data, valid reason codes, and the correct version of the underwriting and fair-lending policies?

The underwriting policy defines the applicable thresholds and decision rules. This procedure defines how employees and automated systems locate the evidence, validate it, apply the correct rules, record the outcome, and escalate exceptions.

## 2. Scope

This procedure applies only to the personal lending products defined in `loan_products.csv`:

| Product Code | Product Description | Procedure Treatment |
|---|---|---|
| `MNT-01` | Term/installment personal loan | Review amount, credit risk, DTI, and requested term |
| `REV-02` | Revolving personal credit | Review credit limit, credit risk, and DTI; a fixed term may not apply |

The process does not cover mortgages, auto loans, student loans, home-equity products, or small-business loans. A product code other than `MNT-01` or `REV-02` must be assigned `Under Review` and routed outside this procedure.

## 3. Roles and Responsibilities

| Role | Responsibility |
|---|---|
| Automated underwriting process | Performs joins, required-field checks, score and DTI comparisons, policy-version selection, and preliminary decision logic |
| Loan underwriting officer | Reviews exceptions, missing evidence, threshold escalations, and requested terms |
| Credit-risk manager | Resolves risk-rule ambiguity, approves permitted overrides, and monitors product-level risk |
| Fair-lending compliance reviewer | Investigates potential inconsistent treatment, prohibited-factor concerns, and unsupported adverse-action reasons |
| Internal audit | Tests data lineage, policy-version accuracy, evidence retention, reason-code validity, and control performance |

No employee may approve their own undocumented exception or replace a system-generated reason with a more convenient but inaccurate reason code.

## 4. Structured Dataset Confirmation

### 4.1 Required Decision Variables

The following table confirms whether each variable requested by the business questions exists in the supplied structured-data dictionary.

| Decision Variable | Exists? | Dataset and Field | Confirmed Values or Use | Procedure Conclusion |
|---|---|---|---|---|
| Income | Yes | `applicants.income` | Gross annual income in USD | Use for the income rule in the applicable underwriting policy |
| Employment duration | Yes | `applicants.employment_duration` | Months with the current employer | Use for the employment-history rule |
| Employment status | Yes | `applicants.employment_status` | `employed`, `self-employed`, or `retired` | Use for documentation routing; do not deny solely because of the category |
| Baseline credit score | Yes | `applicants.credit_score` | Score at profile creation, expected range 300-850 | Use for profile comparison or preliminary screening only |
| Current credit score | Yes, conditionally | `credit_pull_events.credit_score` | Bureau score from a credit-pull event; blank is allowed when the pull fails | Use the qualifying credit-pull score for the final decision |
| Applicant credit tier | Yes | `applicants.credit_risk_tier` | `Excellent`, `Good`, `Fair`, or `Subprime` | Compare with the score-derived tier as a consistency check |
| Current underwriting risk status | Yes | `credit_pull_events.risk_status` | `Low Risk`, `Moderate Risk`, `High Risk`, `Very High Risk`, or `Review` | Compare with `loan_products.risk_status_criteria` |
| Requested amount | **No** | No field in the supplied dictionary | `approved_amount` is an output and is not the original request | Automatic amount-limit testing is not reliable until `requested_amount` is added |
| DTI ratio | Yes | `credit_pull_events.dti_ratio` | Percentage at the time of the credit pull | Compare with `loan_products.max_dti_score` |
| Region | Yes | `applicants.region` and `applicants.state` | Five market regions and two-letter state | Use for routing and fair-lending monitoring; no product-region eligibility field currently exists |
| Requested term | **No** | No field in the supplied dictionary | `approved_term_length` is an output | Add `requested_term_length` if term eligibility will be tested |
| Application date | Yes, but stored at applicant level | `applicants.application_date` | Date and time the applicant record was submitted | Move or duplicate this field in `loan_applications.csv` if an applicant can submit multiple applications |
| Decision status | Yes | `loan_applications.decision_status` | `Approved`, `Denied`, or `Under Review` | These exact values must be used throughout this procedure |

### 4.2 Product-Code Confirmation

The only valid product codes defined in `loan_products.csv` are:

- `MNT-01` for term/installment personal loans.
- `REV-02` for revolving personal credit.

The procedure must not use unrelated codes such as `PL-100`, `PL-200`, or product names as substitutes for `product_code`.

### 4.3 Risk-Tier Confirmation

The structured datasets contain two related but different classifications:

| Dataset Field | Allowed Values | Meaning |
|---|---|---|
| `applicants.credit_risk_tier` | `Excellent`, `Good`, `Fair`, `Subprime` | Applicant profile classification |
| `credit_pull_events.risk_status` | `Low Risk`, `Moderate Risk`, `High Risk`, `Very High Risk`, `Review` | Underwriting status assigned to the credit-pull evidence |

These names must not be mixed. The current underwriting policy supplies the score ranges that map the profile tier to the credit-pull risk status. If the stored tier, calculated tier, and risk status disagree, the application must be assigned `Under Review`.

### 4.4 Decision-Status Confirmation

Only these exact case-sensitive business values may be stored in `loan_applications.decision_status`:

| Valid Status | Use |
|---|---|
| `Approved` | All automatic requirements are satisfied and no escalation condition exists |
| `Denied` | An explicit policy rule prevents approval and the specific principal reason is documented |
| `Under Review` | Data, evidence, policy selection, or compliance concerns require human review |

Do not store `APPROVED`, `DECLINED`, `MANUAL_REVIEW`, `Escalated`, or `Compliance Escalation` in `decision_status`. Escalation is a workflow action; the application status remains `Under Review` until a valid final decision is recorded.

## 5. Required Record Relationships

Before reviewing eligibility, the system must validate the following relationships:

| From | To | Join Condition | Control |
|---|---|---|---|
| `loan_applications.csv` | `applicants.csv` | `loan_applications.applicant_id = applicants.applicant_id` | Exactly one applicant must match the application |
| `loan_applications.csv` | `loan_products.csv` | `loan_applications.product_code = loan_products.product_code` | Exactly one valid product must match |
| `loan_applications.csv` | `credit_pull_events.csv` | Match both `applicant_id` and `product_code` | At least one relevant event should exist; the qualifying event is selected in Step 4 |
| `loan_applications.policy_doc_id` | Document corpus metadata | Match the selected policy document ID | The selected policy must cover the application date |
| `loan_applications.doc_id` | Document corpus metadata | Match the principal supporting source document | The document must exist and be retrievable |
| `loan_applications.rule_id_triggered` | Approved reason-code catalog | Match one valid rule ID | The code must exist in Section 9 of this procedure |

A missing or duplicate parent record, orphaned foreign key, or invalid document reference must stop automated approval and place the application `Under Review`.

## 6. End-to-End Review Process

### Step 1: Identify the Application

1. Retrieve the `loan_applications.csv` record using `application_id`.
2. Confirm that `application_id`, `applicant_id`, and `product_code` are populated.
3. Confirm that `application_id` is unique.
4. Join the application to exactly one record in `applicants.csv` using `applicant_id`.
5. Join the application to exactly one record in `loan_products.csv` using `product_code`.
6. Confirm that `product_code` is `MNT-01` or `REV-02`.

If a required identifier is missing, a join fails, or the product is outside scope, stop automated processing and assign `Under Review` using the applicable data or product exception code.

### Step 2: Establish the Application Date

1. Retrieve `applicants.application_date`.
2. Confirm that it contains a valid date and time.
3. Confirm that `decision_date` is not earlier than `application_date`.
4. Confirm that only one application is associated with the applicant if the applicant-level date is being used.

Because `application_date` is currently stored in `applicants.csv`, the date is ambiguous if one applicant can submit more than one application. Until `loan_applications.application_date` is added, any applicant with multiple applications must be assigned `Under Review` under `UW-POL-001` so a reviewer can establish the correct application date.

### Step 3: Select the Applicable Policy Version

1. Search the document metadata for underwriting policies whose effective period contains the application date.
2. Verify the candidate policy's `doc_id`, title, version, effective date, end date when applicable, and authority status.
3. Select `FIN-POL-001-OLD`, Version 1.0, only for applications dated January 1, 2024 through December 31, 2025.
4. Select `FIN-POL-001`, Version 2.0, for applications dated on or after January 1, 2026.
5. Do not select a document marked `DRAFT`, `EXPIRED`, or outside its effective period.
6. A `SUPERSEDED` policy may be used only for a historical application whose date falls within that policy's historical effective period.
7. Record the selected policy document ID in `loan_applications.policy_doc_id`.

If no policy matches, more than one policy matches, the application date is unreliable, or the stored `policy_doc_id` conflicts with the date, stop the automated review and assign `Under Review` using `UW-POL-001` or `UW-POL-002`.

### Step 4: Select and Validate the Credit-Pull Event

1. Filter `credit_pull_events.csv` to records matching the application's `applicant_id` and `product_code`.
2. Exclude records where `active_flag` is not `TRUE`.
3. Review `pull_status`:
   - `SUCCESS` requires a credit score from 300 through 850 and a blank `failure_code`.
   - `FAILED` requires a populated `failure_code` and a blank credit score.
   - `PENDING` cannot support a final automated decision.
4. Confirm that `provider_request_id` is populated and unique.
5. Confirm that `bureau_name` is `EQUIFAX`, `EXPERIAN`, or `TRANSUNION`.
6. Confirm that `event_date` is not before the established application date and is not after `decision_date`.
7. When multiple valid successful pulls exist, select the most recent qualifying `event_date`.
8. Preserve `credit_pull_event_id` and `provider_request_id` in the audit evidence so the score can be traced to the bureau request.

A failed or pending pull requires `Under Review` under `UW-CRD-003`. Missing, inactive, duplicated, out-of-window, or conflicting credit evidence requires `Under Review` under `UW-CRD-004`.

### Step 5: Validate the Score, Tier, and Risk Status

1. Use `credit_pull_events.credit_score` from the qualifying event as the final decision score.
2. Apply the score ranges in the policy version selected in Step 3.
3. Calculate the expected applicant tier and expected underwriting risk status.
4. Compare the calculated applicant tier with `applicants.credit_risk_tier`.
5. Compare the calculated underwriting status with `credit_pull_events.risk_status`.
6. Review `credit_score_tier_change` and confirm it uses one of the allowed values: `No_Change`, `Upgraded`, `Downgraded`, or `Not_Applicable`.
7. Escalate a material downgrade occurring before the decision.

Do not use the baseline `applicants.credit_score` instead of the qualifying event score for a final Version 2.0 decision. A mismatch among score, tier, and status requires `Under Review` under `UW-CRD-002`.

### Step 6: Apply Product Eligibility Rules

1. Retrieve the selected product using `loan_applications.product_code`.
2. Confirm that the matching product record includes `risk_status_criteria`, `max_amount`, `product_name`, `min_term_month`, `max_term_month`, and `max_dti_score`.
3. Compare the applicant's current `risk_status` with `loan_products.risk_status_criteria`.
4. If the text in `risk_status_criteria` is missing, contradictory, or cannot be interpreted consistently, do not guess; assign `Under Review` under `UW-DATA-003`.
5. Apply the rules from the selected underwriting policy and the current product eligibility document.

`loan_products.base_rate` may support pricing or disclosure, but the supplied application dataset does not contain an approved rate. Base rate therefore must not be used as evidence that the eligibility review was completed.

### Step 7: Review Income and Employment

1. Retrieve `applicants.income`, `employment_duration`, and `employment_status`.
2. Validate that income is numeric, nonnegative, and within a plausible range.
3. Apply the income threshold in the policy version selected in Step 3.
4. Validate that employment duration is a nonnegative number of months.
5. Apply the employment-duration standard from the selected policy version.
6. Route self-employed and retired applicants for the documentation required by policy without automatically treating their status as unfavorable.

Below-threshold income requires `Under Review` under `UW-INC-001`. Short employment history requires `Under Review` under `UW-EMP-001`. Missing or unsupported income requires `UW-DATA-002`.

### Step 8: Review DTI

1. Retrieve `credit_pull_events.dti_ratio` from the qualifying credit-pull event.
2. Retrieve `loan_products.max_dti_score` for the selected product.
3. Confirm that both fields contain valid percentages.
4. Compare the applicant DTI with the product maximum.
5. Preserve both values in the review evidence.

DTI above the product maximum requires `Under Review` under `UW-DTI-001`. Missing or invalid DTI evidence requires `Under Review` under `UW-DATA-004`. A high DTI ratio does not automatically produce `Denied` unless the applicable policy expressly requires denial.

### Step 9: Review Requested Amount and Term

The business question requires comparison of the requested amount with `loan_products.max_amount`, but the supplied dictionary has no `requested_amount` field. It also has no `requested_term_length` field.

Until those fields are added:

1. Do not treat `approved_amount` as the requested amount.
2. Do not treat `approved_term_length` as the requested term.
3. Obtain the requested values from controlled application evidence when available.
4. Record the evidence document through `doc_id`.
5. Assign `Under Review` under `UW-DATA-005`; a fully automated approval is not supportable from the current structured data alone.

After the dictionary and dataset are updated:

1. Compare `loan_applications.requested_amount` with `loan_products.max_amount`.
2. For `MNT-01`, compare `requested_term_length` with `min_term_month` and `max_term_month`.
3. For `REV-02`, confirm whether a fixed term applies. If it does not, store zero in `approved_term_length` as defined by the dictionary.
4. A request exceeding `max_amount` requires `Under Review` under `UW-AMT-001`.
5. A term outside the permitted range requires `Under Review` under `UW-TRM-001`.

### Step 10: Use Region Correctly

1. Validate that `applicants.region` is one of `Northeast`, `Southeast`, `Midwest`, `Southwest`, or `West`.
2. Validate that `applicants.state` is a two-letter state abbreviation.
3. Use region and state for operational routing, portfolio monitoring, and fair-lending analysis.
4. Do not deny or disadvantage an applicant based solely on region or state.
5. Do not apply a product-region restriction unless that restriction is documented, legally reviewed, consistently applied, and represented in the product data.

The current `loan_products.csv` dictionary does not contain an eligible-region or eligible-state field. Therefore, region currently exists as applicant data but is not a supported product eligibility rule.

### Step 11: Determine the Preliminary Outcome

Apply the following order of precedence:

1. **Data or policy uncertainty:** Assign `Under Review` when required data, evidence, document references, or policy selection is missing, invalid, or conflicting.
2. **Compliance concern:** Assign `Under Review` and route to Compliance when a prohibited-factor, inconsistent-treatment, override, or reason-code concern exists.
3. **Explicit denial rule:** Assign `Denied` only when the selected policy explicitly prevents approval and the supporting evidence is complete.
4. **Automatic approval:** Assign `Approved` only when every required check passes and no review condition remains.

The system must not change `Under Review` to `Denied` merely because information is missing. Missing information requires investigation, not an unsupported adverse decision.

### Step 12: Record the Decision and Evidence

Populate `loan_applications.csv` as follows:

| Field | Required Procedure |
|---|---|
| `application_id` | Preserve the unique application identifier |
| `applicant_id` | Preserve the validated applicant foreign key |
| `product_code` | Store exactly `MNT-01` or `REV-02` |
| `decision_date` | Record the date and time the outcome was entered |
| `decision_status` | Store exactly `Approved`, `Denied`, or `Under Review` |
| `approved_amount` | Store the approved amount; store zero when no amount is approved |
| `approved_term_length` | Store the approved term; store zero when no term is approved or applicable |
| `doc_id` | Store the principal source document supporting the outcome |
| `policy_doc_id` | Store the policy version selected from the application date |
| `rule_id_triggered` | Store the primary valid reason code from Section 9 |

Before completing the record, verify that `doc_id`, `policy_doc_id`, and `rule_id_triggered` resolve to real records in the document metadata or approved rule catalog.

## 7. Decision Requirements

### 7.1 Approved

An application may be recorded as `Approved` only when:

1. All required identifiers and joins are valid.
2. The correct policy was selected using the application date.
3. The product code is valid and the product record is complete.
4. The qualifying credit pull is active, successful, current, product-matched, and traceable.
5. Credit score, tier, and risk status are consistent.
6. Income and employment satisfy the applicable automatic requirements.
7. DTI is within the selected product limit.
8. Requested amount and term are known and within product limits.
9. No manual-review or compliance concern remains.
10. Supporting document and reason-code references are valid.

Record `UW-APP-001` as the primary reason code.

Under the currently supplied dictionary, fully automated approval is blocked because requested amount and requested term are not stored. This limitation should be corrected before demonstrating automatic approval logic.

### 7.2 Denied

An application may be recorded as `Denied` only when:

1. An explicit rule in the applicable policy requires denial.
2. The evidence supporting that rule is complete and valid.
3. The policy version was effective on the application date.
4. The reason code accurately identifies the principal factor actually used.
5. The decision is not based on missing data, a failed pull, or an unresolved policy conflict.

The current policy permits denial under `UW-AGE-001` and `UW-CRD-001` when their conditions are fully established. Other exceptions in this procedure generally require `Under Review` rather than automatic denial.

### 7.3 Under Review

Assign `Under Review` when:

- Required data or evidence is missing.
- A primary or foreign key fails validation.
- The correct application date or policy version cannot be established.
- A credit pull is failed, pending, inactive, duplicated, or inconsistent.
- Score, tier, and risk status do not agree.
- Income, employment duration, DTI, amount, or term requires judgment.
- Product criteria are missing or ambiguous.
- A manual override is proposed.
- A fair-lending or compliance concern is identified.
- The decision cannot be explained using valid data, documents, and reason codes.

`Under Review` is both the data status and the control that prevents unsupported approval or denial.

## 8. Manual Review and Escalation Workflow

### 8.1 Lending Operations Review

The underwriting officer must:

1. Confirm the triggering rule and missing or conflicting evidence.
2. Review the applicant, application, product, and credit-pull records.
3. Obtain permitted supporting documents when required.
4. Reperform the relevant comparison using the correct policy version.
5. Record the reviewer, review date, evidence examined, and resolution in an audit record.
6. Leave the application `Under Review` until all required evidence is resolved.

### 8.2 Credit-Risk Escalation

Escalate to the credit-risk manager when:

- `risk_status_criteria` is unclear or conflicts with the underwriting policy.
- A requested amount or term requires an authorized exception.
- Credit evidence materially changed before the decision.
- A manual override of an automated result is proposed.
- Multiple rules produce conflicting outcomes.

### 8.3 Compliance Escalation

Escalate to fair-lending Compliance when:

- Similarly situated applicants appear to have received different treatment without a documented business reason.
- A prohibited characteristic or potential proxy appears to have influenced the decision.
- Region or state appears to have been used as an unsupported eligibility rule.
- A reason code does not match the factor actually evaluated.
- A manual override lacks specific evidence or an identifiable reviewer.
- A current application was evaluated using a superseded policy.

The application remains `Under Review` during the compliance investigation.

### 8.4 Override Control

An override record should preserve:

- Original automated outcome.
- Final human outcome.
- Primary and supporting reason codes.
- Specific justification.
- Evidence reviewed.
- Reviewer identifier.
- Approval authority.
- Review and approval dates.
- Applicable policy and product-rule versions.

These fields do not exist in the supplied datasets. If overrides will be analyzed, create a separate decision-event or override table rather than overwriting the original automated outcome.

## 9. Decision Reason-Code Catalog

The primary applicable code must be stored in `loan_applications.rule_id_triggered`.

| Rule ID | Description | Applicable Outcome | Required Evidence | Review Required |
|---|---|---|---|---|
| `UW-APP-001` | All automatic approval requirements are satisfied | Approved | Complete applicant, application, product, credit-pull, policy, and source-document records | No |
| `UW-AGE-001` | Applicant is below the policy's minimum legal age | Denied | `birth_date`, application date, and applicable policy | No |
| `UW-INC-001` | Income is below the automatic approval threshold | Under Review | `applicants.income` and income evidence | Yes |
| `UW-EMP-001` | Employment duration is below the automatic standard | Under Review | `employment_duration`, `employment_status`, and supporting evidence | Yes |
| `UW-CRD-001` | Credit risk does not satisfy the selected product criteria | Denied | Qualifying score, risk status, product criteria, and policy | No |
| `UW-CRD-002` | Score, applicant tier, or risk status is inconsistent | Under Review | Applicant profile and qualifying credit-pull event | Yes |
| `UW-CRD-003` | Credit pull failed or remains pending | Under Review | `pull_status`, `failure_code`, and `provider_request_id` | Yes |
| `UW-CRD-004` | Credit evidence is missing, inactive, duplicated, out of date, or conflicting | Under Review | Credit-pull event history and validation results | Yes |
| `UW-DTI-001` | DTI exceeds the selected product maximum | Under Review | `dti_ratio`, `max_dti_score`, and product code | Yes |
| `UW-AMT-001` | Requested amount exceeds the product maximum | Under Review | Requested amount, `max_amount`, and application evidence | Yes |
| `UW-TRM-001` | Requested term is outside the product range | Under Review | Requested term, `min_term_month`, and `max_term_month` | Yes |
| `UW-DATA-001` | Birth date or application date is missing or invalid | Under Review | Field-validation report | Yes |
| `UW-DATA-002` | Income is missing, invalid, or unsupported | Under Review | Income field and verification evidence | Yes |
| `UW-DATA-003` | Product risk criteria are missing, ambiguous, or inconsistent | Under Review | Product record and eligibility-rule document | Yes |
| `UW-DATA-004` | DTI or the product DTI limit is missing or invalid | Under Review | Credit-pull and product records | Yes |
| `UW-DATA-005` | Requested amount or requested term is unavailable | Under Review | Application record and missing-field report | Yes |
| `UW-POL-001` | Applicable policy cannot be determined from the application date | Under Review | Application date and policy metadata | Yes |
| `UW-POL-002` | Stored policy is superseded, outside its effective period, or inconsistent with the application date | Under Review | Stored policy ID, effective dates, version, and authority status | Yes |
| `UW-DOC-001` | Supporting document reference is missing or cannot be resolved | Under Review | `doc_id` and document metadata | Yes |
| `UW-REF-001` | Applicant, product, or document relationship fails validation | Under Review | Join and referential-integrity results | Yes |
| `UW-FL-001` | Potential fair-lending inconsistency or prohibited-factor concern | Under Review with Compliance escalation | Comparison evidence, policy references, and audit record | Yes |

If multiple rules apply, `rule_id_triggered` must contain the rule that principally determined or escalated the outcome. Supporting codes should be retained in an audit log or application-rule bridge table.

## 10. Fair-Lending Review Process

### 10.1 Separate Decisioning from Compliance Testing

Protected demographic characteristics must not be inputs to the approval, denial, amount, term, or pricing decision, except that `birth_date` may be used to confirm legal capacity under the minimum-age rule.

Compliance personnel may use controlled demographic data to test whether similarly situated applicants received consistent treatment. Compliance-only data must be access-restricted and logically separated from automated underwriting inputs.

### 10.2 Define Similarly Situated Applicants

When performing a consistency review, compare applicants with materially similar legitimate underwriting characteristics, including:

- Same product code.
- Same policy version and effective period.
- Similar credit-score or risk-status range.
- Similar income range.
- Similar DTI range.
- Similar requested amount and term, once those fields are available.
- Similar employment circumstances when relevant to the applicable rule.

Differences in outcomes should be investigated when these legitimate factors do not explain the difference.

### 10.3 Current Fair-Lending Data Limitation

The supplied dictionary contains `birth_date`, `region`, and `state`, but it does not contain most protected-group attributes needed for a complete fair-lending analysis. Region and state are not substitutes for race, ethnicity, sex, marital status, religion, national origin, public-assistance status, or other protected characteristics.

Therefore, the current structured datasets can support:

- Age-related legal-capacity testing.
- Geographic outcome monitoring.
- Review of consistent rule application using legitimate financial variables.

They cannot, by themselves, fully answer whether decisions were consistent across all protected demographic groups. The project must either add a separately governed compliance dataset or clearly disclose this analytical limitation.

## 11. Explainability and Traceability Test

Before finalizing any decision, the reviewer or system must be able to answer all of the following:

| Traceability Question | Required Evidence |
|---|---|
| Which applicant and application were reviewed? | `applicant_id` and `application_id` |
| Which product was selected? | `product_code` matching `MNT-01` or `REV-02` |
| Which credit evidence was used? | `credit_pull_event_id`, `event_date`, `bureau_name`, and `provider_request_id` |
| Which policy governed the decision? | Application date plus `policy_doc_id`, version, effective dates, and authority status |
| Which product requirements were applied? | Product record plus the current eligibility-rule document |
| Which rule principally determined the outcome? | Valid `rule_id_triggered` from Section 9 |
| Which source document supports the decision? | Resolvable `doc_id` |
| Can the calculations be reproduced? | Stored source values for score, risk status, income, employment duration, DTI, requested amount, and term |
| Was the final status valid? | Exactly `Approved`, `Denied`, or `Under Review` |
| Was a fairness or override concern resolved? | Compliance or override audit evidence, when applicable |

If any required answer cannot be supported, the application must remain `Under Review`.

## 12. Data Quality and Control Checks

### 12.1 Required Checks Before Automated Approval

- Primary keys are populated and unique.
- Foreign keys resolve to exactly one valid parent record.
- Product code is `MNT-01` or `REV-02`.
- Application and decision dates are valid and logically ordered.
- Policy version matches the application date.
- Credit score is between 300 and 850 when `pull_status = SUCCESS`.
- `failure_code` is populated when `pull_status = FAILED`.
- `provider_request_id` is populated and unique.
- DTI and product DTI limit are valid percentages.
- Income and employment duration are valid numeric values.
- Requested amount and requested term are available and within product limits.
- Decision status and reason code use approved values.
- `doc_id` and `policy_doc_id` resolve to available corpus documents.

### 12.2 Prohibited Shortcuts

- Do not use `approved_amount` as requested amount.
- Do not use `approved_term_length` as requested term.
- Do not use the applicant's baseline score when Version 2.0 requires a qualifying credit pull.
- Do not select policy by decision date alone.
- Do not convert missing information directly into a denial.
- Do not use region, state, housing status, or employment category as a proxy for a protected characteristic.
- Do not choose the closest reason code when it does not describe the actual decision factor.
- Do not overwrite the original automated result when recording a manual override.

## 13. Required Data-Model Improvements

| Priority | Recommended Change | Reason |
|---|---|---|
| Critical | Add `application_date` to `loan_applications.csv` | Policy selection must occur at the application level, especially when one applicant may submit multiple applications |
| Critical | Add `requested_amount` to `loan_applications.csv` | Required to answer the business question and compare the request with `loan_products.max_amount` |
| High | Add `requested_term_length` to `loan_applications.csv` | Required to test term eligibility for `MNT-01` |
| High | Use structured values or columns for product risk eligibility instead of relying only on `risk_status_criteria` text | Prevents inconsistent parsing and makes automated decisions reproducible |
| High | Add an application-rule bridge or audit table | Preserves multiple triggered rules while `rule_id_triggered` stores only the primary code |
| High | Add a decision-event or override table | Preserves the original automated decision, human override, reviewer, evidence, and timestamps |
| Conditional | Add a separately governed fair-lending audit dataset | Needed to test outcomes across protected groups without exposing those attributes to underwriting models |
| Optional | Add an approved-rate field if pricing fairness will be analyzed | `base_rate` exists, but no application-level approved rate is currently stored |

Until the critical changes are completed, the project should describe its approval output as a demonstration of the review workflow rather than claim that every approval requirement is fully supported by structured data.

## 14. Periodic Quality Review

Consumer Lending Operations and Internal Audit should periodically test samples for:

- Incorrect applicant, product, or credit-event joins.
- Use of the wrong policy version.
- Missing or invalid document references.
- Invalid or vague reason codes.
- Unsupported approvals or denials.
- Unresolved `Under Review` applications.
- Manual overrides without preserved evidence.
- Differences in treatment not explained by legitimate underwriting factors.
- Use of region or state as an unsupported decision rule.
- Data corrections that overwrite original evidence.

Findings should identify the affected `application_id`, responsible control, severity, corrective action, owner, and due date.

## 15. Related Documents

- `FIN-POL-001` - Internal Personal Loan Underwriting Policy, Version 2.0, CURRENT
- `FIN-POL-001-OLD` - Internal Personal Loan Underwriting Policy, Version 1.0, SUPERSEDED
- `FIN-RULE-001` - Personal Loan Product Eligibility Rules, Version 2026.1
- `FIN-COMP-001` - Fair-Lending Review Memorandum
- 12 CFR Part 1002 - Equal Credit Opportunity Act (Regulation B)
- 12 CFR 1026.18 - Regulation Z Content of Disclosures for Closed-End Credit
- CFPB Circular 2023-03 - Adverse Action Notifications and Complex Credit Models

---

**Document Status:** APPROVED  
**End of Synthetic Internal Document**
