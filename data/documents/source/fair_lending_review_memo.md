# Fair-Lending Review Memorandum

> **SYNTHETIC INTERNAL DOCUMENT**  
> Created for the DSBA 6171 applied machine learning project. It is not legal advice and must not be used for real lending decisions.

| Document control | Value |
|---|---|
| Document ID | FIN-COMP-001 |
| Document type | Internal Compliance Memorandum |
| Owner | Fair-Lending Compliance and Model Risk Management |
| Effective date | January 1, 2026 |
| Version | 2 |
| Authority status | APPROVED |
| Jurisdiction | United States |
| Confidentiality | Restricted Internal |
| Access group | Fair-Lending Compliance, Model Risk, Internal Audit, and authorized Legal personnel |
| Related law and guidance | Equal Credit Opportunity Act and Regulation B, including 12 CFR Part 1002 |
| Related documents | FIN-POL-001; FIN-PROC-001; FIN-RULE-001; FIN-POL-001-OLD |

**TO:** Consumer Lending Operations, Credit Risk Management, Model Risk Management, and Internal Audit  
**FROM:** Fair-Lending Compliance and Model Risk Management  
**SUBJECT:** Operational Fair-Lending Review of Personal-Loan Decisions

## 1. Purpose

This memorandum establishes a repeatable process for identifying, documenting, and escalating potential differences in personal-loan treatment among applicants who are similarly situated under the same product and underwriting rules.

The review supports the business question:

> Were lending decisions applied consistently across applicants, without unexplained differences among protected demographic groups?

The process is designed for compliance monitoring after or independently from the credit decision. A statistical difference is a screening signal, not by itself proof of unlawful discrimination. Compliance and Legal personnel must evaluate the facts, applicable law, data quality, and legitimate documented business explanations.

## 2. Scope

The review covers personal-loan applications with a recorded outcome of exactly `Approved`, `Denied`, or `Under Review`. It includes:

- Automated eligibility and risk decisions.
- Manual-review referrals.
- Manual overrides of automated recommendations.
- Approved amount and term differences.
- Adverse-action reason codes and notices.
- Product, policy-version, reviewer, time-period, and regional patterns.

The review excludes marketing, servicing, collections, and mortgage-lending analyses unless those functions are added through a separately approved procedure.

## 3. Protected-basis and monitoring attributes

### 3.1 Attributes considered when legally available

The Equal Credit Opportunity Act and Regulation B prohibit discrimination on specified bases. For this synthetic academic review, the monitoring design should be able to evaluate the following categories when collection and use are legally permitted and the data are reliable:

| Monitoring category | Recommended analytical field | Permitted compliance use | Credit-decision use |
|---|---|---|---|
| Race | `race_category` | Compare outcomes and treatment patterns | Prohibited |
| Ethnicity | `ethnicity_category` | Compare outcomes and treatment patterns | Prohibited |
| Sex | `sex_category` | Compare outcomes and treatment patterns | Prohibited |
| Age | `age_at_application` and `age_band` | Monitor age-based differences, subject to applicable legal exceptions | Must not be used adversely except as legally permitted |
| Marital status | `marital_status_category` | Monitor treatment differences | Prohibited as an approval criterion |
| National origin | `national_origin_category`, only if lawfully available | Compliance monitoring only | Prohibited |
| Religion | Not recommended for this project unless specifically authorized and necessary | Compliance monitoring only if lawfully available | Prohibited |
| Public-assistance income | `public_assistance_income_flag` | Confirm lawful income is evaluated consistently | Source may not be used to disadvantage the applicant |
| Good-faith exercise of consumer-credit rights | `consumer_credit_rights_flag`, only if lawfully documented | Investigate potential retaliation or differential treatment | Prohibited |

Because collection rules can differ by credit product and purpose, real institutions must obtain Legal and Privacy approval before collecting or retaining protected-basis information. For this academic project, use synthetic demographic values only.

### 3.2 Strict separation from underwriting

Protected or monitoring attributes must never be included in:

- Product eligibility criteria.
- Credit-risk tier or `risk_status` assignment.
- Approval, denial, or `Under Review` decision rules.
- Pricing, approved amount, or approved term calculations.
- Model-training features used to predict creditworthiness or recommend an outcome.
- Manual-review screens visible to an underwriter before the decision is complete.

The fields may be used only in a restricted, post-decision compliance dataset. Underwriters and automated decision services must not have access to that dataset.

Age must be calculated as of `application_date`; current age must not be substituted. Region and state may be used for documented licensing or product-availability requirements, but must not serve as proxies for protected characteristics or creditworthiness.

## 4. Required data architecture

### 4.1 Restricted monitoring table

Create a separate restricted dataset, such as `fair_lending_monitoring.csv`, linked by `application_id`. Do not place monitoring attributes directly in the operational underwriting feature set.

| Field | Type | Required for analysis | Definition and control |
|---|---|---:|---|
| `application_id` | VARCHAR(50) | Yes | Primary key and foreign key to `loan_applications.application_id` |
| `race_category` | VARCHAR(50) | Recommended | Synthetic or lawfully collected monitoring category; include `Not Available` |
| `ethnicity_category` | VARCHAR(50) | Recommended | Synthetic or lawfully collected monitoring category; include `Not Available` |
| `sex_category` | VARCHAR(30) | Recommended | Synthetic or lawfully collected monitoring category; include `Not Available` |
| `age_at_application` | INT | Yes | Derived from `birth_date` and `application_date` using the applicant's age on the application date |
| `age_band` | VARCHAR(20) | Yes | Stable bands defined before analysis; retain continuous age for sensitivity checks |
| `marital_status_category` | VARCHAR(30) | Optional | Use only if legally available and approved for monitoring |
| `public_assistance_income_flag` | BOOLEAN | Recommended | Indicates that qualifying income includes public-assistance income; not a measure of creditworthiness |
| `monitoring_source` | VARCHAR(30) | Yes | `Synthetic`, `Applicant Provided`, or another approved source |
| `monitoring_as_of_date` | DATE | Yes | Date the monitoring record was established |
| `restricted_access_flag` | BOOLEAN | Yes | Must be `TRUE` for every monitoring record |

Missing protected-basis data must be reported as `Not Available`; it must not be guessed from name, address, ZIP code, photographs, language, or another proxy unless a separately approved compliance methodology explicitly authorizes an estimation method.

### 4.2 Decision and process fields

The following operational fields are also required to test consistency and explain individual differences:

| Dataset | Required field | Current status | Why it is needed |
|---|---|---|---|
| `applicants.csv` | `income` | Available | Compare verified repayment capacity |
| `applicants.csv` | `employment_duration` | Available | Compare employment stability under the same policy rule |
| `applicants.csv` | `credit_score` and `credit_risk_tier` | Available | Compare credit risk using the policy-effective score and tier |
| `applicants.csv` | `birth_date` and `application_date` | Available | Derive age at application |
| `applicants.csv` | `region` and `state` | Available | Monitor geographic patterns and documented availability rules |
| `credit_pull_events.csv` | `dti_ratio`, `credit_score`, `risk_status`, `event_date`, `active_flag`, and `pull_status` | Available | Reconstruct the credit information used at decision time |
| `loan_applications.csv` | `decision_status` | Available | Analyze exactly `Approved`, `Denied`, or `Under Review` |
| `loan_applications.csv` | `requested_amount` | **Missing—add** | Match applicants with similar requested exposure; `approved_amount` is not a substitute |
| `loan_applications.csv` | `requested_term_length` | **Missing—add** | Compare applications requesting similar terms; `approved_term_length` is not a substitute |
| `loan_applications.csv` | `automated_decision_status` | **Missing—add** | Establish the pre-override recommendation |
| `loan_applications.csv` | `override_flag` | **Missing—add** | Identify decisions changed by a person |
| `loan_applications.csv` | `override_reason_code` and `override_notes` | **Missing—add** | Determine whether an override has a legitimate documented explanation |
| `loan_applications.csv` | `reviewer_id` and `override_timestamp` | **Missing—add** | Analyze reviewer patterns and establish sequence |
| `loan_applications.csv` | `adverse_action_notice_date` | **Missing—add** | Test whether required adverse-action documentation was produced timely |
| `loan_applications.csv` | `adverse_action_reason_codes` | **Missing—add** | Compare communicated reasons with the actual decision rules |
| `loan_applications.csv` | `doc_id`, `policy_doc_id`, and `rule_id_triggered` | Available | Trace the decision to supporting rules and policy version |

Until the missing fields are added, the project may describe the fairness-review design, but it cannot fully test override consistency, adverse-action documentation, or similarly situated treatment.

## 5. Identifying similarly situated applicants

### 5.1 Required comparison sequence

Applicants may be compared only after the analyst confirms that they were evaluated under materially comparable circumstances. Construct comparison groups in this order:

1. Use the same `product_code`.
2. Use the same underwriting-policy and product-rule versions effective on `application_date`.
3. Use the same decision channel and stage, such as automated decision, initial manual review, or override review.
4. Use the latest active, successful credit-pull event available before `decision_date`.
5. Match or control for legitimate underwriting factors listed in Section 5.2.
6. Compare protected-group outcomes only after the above conditions are satisfied.

Applications evaluated under different policy versions or products must not be pooled without explicit controls for those differences.

### 5.2 Legitimate matching variables

| Factor | Preferred comparison rule | Reason |
|---|---|---|
| Product | Exact same `product_code` | Products have different limits and risk thresholds |
| Policy period | Exact same `policy_doc_id` and effective version | Prevents old and new policies from being mixed |
| Current credit risk | Same `risk_status`; use continuous score as a sensitivity control | Aligns applicants on current credit risk |
| Credit score | Same policy band; matched-pair review should generally be within 20 score points | Reduces material score differences within a band |
| DTI | Same five-percentage-point band; matched pair generally within 5 points | Aligns repayment burden |
| Income | Same band or within 10% for matched-pair review | Aligns repayment capacity |
| Requested amount | Same product-specific band or within 10% | Aligns requested exposure |
| Requested term | Same term band or exact requested term when sample size permits | Aligns repayment duration |
| Employment duration | Same policy band | Aligns employment-stability treatment |
| Application completeness | Same completeness status | Prevents incomplete files from being compared with complete files |
| Application period | Same month or quarter | Controls for operational and economic timing |
| Region/state | Control only for documented lawful availability or licensing rules | Prevents unsupported geographic explanations |

These thresholds are screening parameters for the synthetic project. They must not be treated as legal standards or as proof that applicants are identical.

### 5.3 Matching methods

Use at least two complementary methods:

- **Cohort comparison:** group applications by product, policy version, risk status, DTI band, income band, amount band, term band, and time period; then compare outcomes across monitoring groups.
- **Matched-pair review:** identify applicants from different monitoring groups with closely comparable legitimate risk characteristics, then examine any difference in decision, amount, term, override, or reason code.
- **Adjusted statistical model:** when the sample is sufficient, estimate decision or treatment differences after controlling only for legitimate, documented underwriting variables. Protected attributes may be included only as monitoring indicators, not as operational decision features.

The analyst must preserve the query, matching rules, model specification, exclusions, and data snapshot so the review can be reproduced.

## 6. Fair-lending metrics

Calculate metrics separately by product, policy version, time period, and relevant monitoring group. Do not rely only on a portfolio-wide average.

| Metric | Calculation | Review purpose |
|---|---|---|
| Approval rate | Approved applications ÷ completed applications | Detect outcome differences |
| Denial rate | Denied applications ÷ completed applications | Detect adverse-outcome differences |
| Manual-review rate | `Under Review` referrals ÷ eligible applications | Detect unequal escalation burden |
| Override rate | Applications with `override_flag = TRUE` ÷ reviewed applications | Detect inconsistent human intervention |
| Favorable override rate | Overrides changing Denied/Under Review to Approved ÷ all overrides | Detect unequal access to exceptions |
| Unfavorable override rate | Overrides changing Approved to Denied/Under Review ÷ all overrides | Detect unequal negative intervention |
| Average approved amount ratio | Group average approved amount ÷ reference-group average | Detect differences in credit extended |
| Average term difference | Group average approved term minus reference-group average | Detect term differences |
| Missing-reason rate | Adverse decisions without complete reason codes ÷ adverse decisions | Detect explanation failures |
| Policy mismatch rate | Applications using an incorrect, superseded, or future policy ÷ reviewed applications | Detect governance failures |

Every report must show group counts and missing-data rates. Small or highly incomplete groups must be flagged; percentages without denominators must not be reported.

## 7. Escalation triggers

The following thresholds are internal screening rules for the synthetic project. They trigger investigation; they do not establish a legal violation or safe harbor.

| Trigger | Severity | Required action |
|---|---|---|
| Any evidence that a protected attribute influenced approval, denial, pricing, amount, term, risk status, or product selection | Critical | Stop affected automated use; preserve evidence; notify Fair-Lending Compliance and Legal immediately |
| Two closely matched applicants receive different decisions without a documented legitimate factor | High | Conduct case-level review and reviewer interview; validate source data and policy application |
| Protected-group approval-rate gap of at least 10 percentage points within a comparable cohort | High | Perform adjusted analysis and matched-file review |
| Protected-group selection-rate ratio below 0.80 within a comparable cohort | High | Perform further analysis; do not treat the ratio as proof or a legal safe harbor |
| Manual-review or override-rate gap of at least 5 percentage points, with at least 30 applications in each compared group | Medium/High | Review reason codes, reviewer patterns, and matched files |
| Repeated unexplained overrides by one reviewer, region, product, or group | High | Escalate to Compliance and management; consider suspending override authority pending review |
| Any adverse decision missing required specific reason codes or supporting evidence | High | Correct the record if permitted; review similar files; assess notice obligations |
| More than 2% of adverse decisions have incomplete or inconsistent reason-code records in a review period | High | Open a systemic documentation review |
| Any current application supported by a future, superseded, draft, or expired policy | High | Reconstruct the decision using the correct effective version and assess applicant impact |
| Geographic outcome difference of at least 10 percentage points after controlling for product and risk factors | Medium/High | Test for documented availability rules, data quality, and potential proxy effects |
| Monitoring data missing for more than 20% of the relevant review population | Data limitation | Do not issue a conclusive fairness finding; document the limitation and remediation plan |

Analysts may escalate a concern below a numeric threshold when the conduct is serious, repeated, intentional, or supported by individual-file evidence.

## 8. Review procedure

### 8.1 Define the review population

1. Set the review start and end dates.
2. Select the product codes and policy versions in scope.
3. Join applications, applicant characteristics, credit-pull events, product rules, and the restricted monitoring table using documented keys.
4. Retain only data available as of each application's `decision_date`.
5. Report excluded records and exclusion reasons.

### 8.2 Validate data

1. Confirm unique primary keys and valid foreign keys.
2. Confirm exact controlled values for decision status, risk status, product code, and reason code.
3. Recalculate age at application and DTI where source components are available.
4. Check for missing, duplicate, future-dated, inactive, or failed credit-pull records.
5. Confirm the policy and product-rule versions were effective on the application date.
6. Measure protected-attribute missingness separately by outcome, product, and region.

### 8.3 Analyze treatment

1. Calculate the metrics in Section 6.
2. Apply the cohort and matched-pair methods in Section 5.
3. Examine automated outcomes separately from manual outcomes.
4. Review favorable and unfavorable overrides separately.
5. Compare actual reason codes with the rules that triggered the decision.
6. Determine whether observed differences have a documented, consistently applied, legitimate underwriting explanation.

### 8.4 Perform file-level review

For every escalated case or sample, preserve:

- `applicant_id` and `application_id`.
- Application and decision timestamps.
- Product code and product-rule version.
- Policy document ID, version, status, and effective date.
- Score, risk status, income, employment duration, DTI, requested amount, and requested term used at decision time.
- Original automated outcome and final outcome.
- Override flag, reviewer, reason, timestamp, and supporting evidence.
- Triggered rule and adverse-action reason codes.
- Monitoring-group attributes in a separately secured review record.
- Comparison-group definition and matched-file identifiers.

### 8.5 Resolve and document

1. Classify each alert as explained, data error, process error, policy exception, model concern, or potential compliance concern.
2. Record the evidence supporting the classification.
3. Assign an owner, remediation action, and due date.
4. Determine whether other applicants were affected by the same issue.
5. Validate remediation and obtain Compliance closure approval.
6. Retain the analysis snapshot, code, output, and approval record for audit.

## 9. Manual-override controls

An override is valid only when:

- The reviewer is authorized for the product and override type.
- The original automated decision is preserved.
- The final status uses exactly `Approved`, `Denied`, or `Under Review`.
- A permitted override reason code and specific narrative explanation are recorded.
- Supporting evidence is attached or traceable.
- The cited policy section and effective document version allow the exception.
- The override is completed before the final decision timestamp.

Unexplained overrides and generic explanations such as “manager approval,” “relationship,” or “special case” are insufficient and require escalation.

## 10. Adverse-action documentation review

For every denied application, the reviewer must determine whether:

- The recorded reasons are specific and consistent with the factors actually considered.
- The reason codes map to the applicable underwriting or product rule.
- The source data support the reasons.
- The correct policy version was used.
- The notice date and required delivery evidence are present.
- A protected or monitoring attribute was not used as a reason.

A generic reason code, missing reason, unsupported reason, or reason inconsistent with the decision record requires high-severity review.

## 11. Roles and access controls

| Role | Permitted responsibility | Prohibited activity |
|---|---|---|
| Underwriter | Apply approved underwriting and product rules | View or use protected monitoring attributes |
| Automated decision service | Use approved legitimate risk variables | Access protected monitoring data or use proxy variables |
| Fair-Lending Compliance | Access restricted monitoring data and conduct post-decision analysis | Alter lending outcomes without documented authority |
| Model Risk Management | Test model design, performance, and group-level impacts | Insert protected attributes into production underwriting features |
| Internal Audit | Independently test controls and evidence | Direct operational lending decisions |
| Data Engineering | Maintain approved joins, access controls, lineage, and versioned snapshots | Expose monitoring fields to underwriting systems |

## 12. Reporting requirements

Each fair-lending review report must contain:

- Population dates, products, policy versions, and record counts.
- Data sources, joins, exclusions, missingness, and known limitations.
- Definitions of monitoring groups and the reference group.
- Exact similarly situated criteria.
- Group-level metrics with counts and denominators.
- Statistical or matching methodology and sensitivity tests.
- Case-level findings and documented explanations.
- Escalations, owners, remediation deadlines, and closure status.
- A statement that protected attributes were used only for compliance monitoring.

If required demographic or operational fields are unavailable, the report must say that the business question could not be fully tested. It must not state that no disparity exists merely because the necessary data were missing.

## 13. Version and authority control

The reviewer must record the document ID, version, effective date, authority status, owner, and jurisdiction for every policy or rule used in the review. A `CURRENT` or `APPROVED` document effective on the application date takes precedence over a related `SUPERSEDED`, `DRAFT`, `EXPIRED`, or future-effective document.

Historical documents may be used to reproduce historical decisions, but they must not be presented as current authority. Any conflict between retrieved documents must result in `Under Review` until the controlling document is confirmed.

## 14. Important notice

This memorandum is synthetic and was created for an academic data-engineering project. It does not replace applicable law, official guidance, legal advice, or an institution's approved fair-lending compliance program.

END OF SYNTHETIC INTERNAL DOCUMENT
