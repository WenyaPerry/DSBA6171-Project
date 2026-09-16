This combined section explains how data structure, point-in-time joins, schema expectations, semantic contracts, and governance metadata affect AI-assisted underwriting decisions.

# Data Modeling & AI Context Integrity 

The table below summarizes an example of an AI failure scenario.

| **Requirement**                      | **What Teacher Wants**                                                                                                                                                                                                                                                                                                                                           |
|--------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Choose one concrete scenario         | Scenario: A credit-pull event occurring after the loan decision is incorrectly used as decision evidence for the original decision. For example, AP-10016 has a loan decision date February 17, 2026, but its recorded credit pull event occurred June 26, 2026.                                                                                                 |
| Pick one issue type                  | Wrong Point in Time Value. The AI may use information that was not available when the loan decision was made.                                                                                                                                                                                                                                                    |
| Explain the issue in business terms  | A loan decision should only be evaluated using information available at the time of the decision. If a later credit event is joined to the application without a date restriction, the applicant's historical financial context becomes inaccurate.                                                                                                              |
| Show what AI receives                | The AI could receive AP-10016's February 17, 2026 loan decision together with a June 26, 2026 credit event showing a 792 credit score, 46.38% DTI, and Low Risk status. The June information was not available when the February decision occurred.                                                                                                              |
| Explain AI's possible interpretation | The AI could responsibly interpret the later credit information as evidence supporting the original loan decision. It might use the 792 credit score or Low Risk status to explain the approval even though those values were unavailable at decision time.                                                                                                      |
| Explain the business consequence     | The organization could produce an inaccurate loan-decision explanation or risk assessment. It could also create data leakage, make historical decisions impossible to reconstruct accurately, and create problems during compliance, fair-lending, or audit reviews.                                                                                             |
| Give an architectural control        | Add temporal validation to the data pipeline so that only credit pull events satisfying event_date \<= decision_date can be used as decision evidence. The event must also match the applicant and product and have a valid successful status. If no valid event exists before the decision, the application should be flagged rather than using a future event. |

# Schema Semantic Contract and AI Context Quality

## A. Schema Expectations Technical Structure

### Required Fields Data Types Keys Nullability and Allowed Values

The table below defines the expected structure for the credit pull event data used by the AI context.

| **Column**               | **Data Type**  | **Required** | **Key** | **Allowed Values**                             |
|--------------------------|----------------|--------------|---------|------------------------------------------------|
| credit_pull_event_id     | string         | required     | PK      |                                                |
| applicant_id             | string         | required     | FK      |                                                |
| product_code             | string         | required     | FK      |                                                |
| event_date               | date/timestamp | required     |         | standard format                                |
| dti_ratio                | numeric        | required     |         | non-negative, unit must be percentage          |
| credit_score             | integer        |              |         | valid range 300-850 when pull_status = SUCCESS |
| credit_score_tier_change | categorical    |              |         | allowed values defined by policy               |
| pull_status              | categorical    | required     |         | allowed values: SUCCESS, FAILED, PENDING       |
| bureau_name              | categorical    |              |         | allowed values: EQUIFAX, TRANSUNION, EXPERIAN  |
| failure_code             | string         | nullable     |         | required when pull_status = FAILED             |
| provider_request_id      | string         | required     |         | traceable                                      |
| risk_status              | categorical    | required     |         | must match policy tiers                        |

### Schema Version Identifier

| schema_version | credit_pull_events_v1.0                                                  |
|----------------|--------------------------------------------------------------------------|
| last_updated   | 2026-09-13                                                               |
| change_log     | tracks changes to allowed values, bureau list, or risk tier definitions. |

## B. Semantic Contract Business Meaning

### 1. Business Definition and Grain

- Each row represents one credit-bureau pull event for a specific applicant and product.

- It is a point-in-time snapshot of credit risk used for underwriting.

### 2. Units of Measure

| dti_ratio    | Must be a percentage (e.g., 35 = 35%).               |
|--------------|------------------------------------------------------|
| credit_score | Uses standard 300-850 scale.                         |
| risk_status  | Must follow the active policy version at event_date. |

### 3. Point-in-Time and As-Of Meaning

- All values represent credit information available at event_date, not later updates.

- AI must treat the record as the credit context used at decision time, not a historical average.

### 4. Source of Truth and Freshness

| Source                | credit bureau API + provider logs.                                               |
|-----------------------|----------------------------------------------------------------------------------|
| Freshness expectation | The latest successful pull (Credit score) before decision_date is the valid one. |

### 5. Contract Version Information

Consumers must preserve:

- Event-date correctness

- Policy-aligned risk tiers

- Pull_status meaning (FAILED does not mean low credit score)

- Bureau identity (different bureaus may produce different scores)

- Canonical DTI units

# Required Semantic Failure Example

## Schema Valid but Semantically Wrong Record

The table below shows a record that passes technical validation but uses the wrong DTI unit.

| credit_pull_event_id     | EV-55201   |
|--------------------------|------------|
| applicant_id             | AP-10088   |
| product_code             | MNT-01     |
| event_date               | 2025-04-10 |
| dti_ratio                | 0.42       |
| credit_score             | 710        |
| credit_score_tier_change | No_Change  |
| pull_status              | SUCCESS    |
| bureau_name              | TRANSUNION |
| failure_code             | NULL       |
| provider_request_id      | PR-99881   |
| risk_status              | Low Risk   |

## Why It Passes Schema but Fails Semantics

- dti_ratio = 0.42

- Schema: numeric, non-negative -\> valid

- Semantic: DTI must be percentage, but this value is in ratio form (0.42 = 42%).

- AI interprets 0.42% instead of 42%.

**What AI receives:** Applicant appears to have extremely low debt burden.

## AI's Possible Misinterpretation

- Applicant has excellent repayment capacity

- Applicant qualifies for higher loan amounts

- Applicant belongs to a low-risk tier

- Application should be approved or auto-approved

## Business Consequence

- Incorrect approval

- Wrong pricing

- Failed audit trail

- Fair-lending mismatch (DTI is a major eligibility factor)

## Correct Canonical Representation

The table below shows the same record using the correct canonical DTI percentage.

| credit_pull_event_id     | EV-55201   |
|--------------------------|------------|
| applicant_id             | AP-10088   |
| product_code             | MNT-01     |
| event_date               | 2025-04-10 |
| dti_ratio                | 42         |
| credit_score             | 710        |
| credit_score_tier_change | No_Change  |
| pull_status              | SUCCESS    |
| bureau_name              | TRANSUNION |
| failure_code             | NULL       |
| provider_request_id      | PR-99881   |
| risk_status              | Low Risk   |

# AI Context and Quality Controls

## Control Table 3 Required Controls

The table below summarizes the main controls that protect AI context quality.

| **Rule / Expectation**                          | **What Goes Wrong**                          | **What AI Receives**                     | **Possible AI Impact**              | **Control / Response**                            |
|-------------------------------------------------|----------------------------------------------|------------------------------------------|-------------------------------------|---------------------------------------------------|
| DTI must be percentage                          | Source sends ratio instead of percent        | 0.42 instead of 42                       | Applicant appears extremely low-DTI | Semantic contract + unit normalization            |
| Use latest successful pull before decision_date | Pipeline selects a failed or older pull      | AI sees outdated or invalid credit score | Wrong risk tier -\> wrong decision  | Point-in-time selection rule + pull_status filter |
| Risk_status must match active policy version    | Risk tier derived using wrong policy version | AI sees mismatched risk classification   | Incorrect approval/denial reasoning | Policy-version tagging + validation               |

# Governance Metadata and AI Lineage

## Data Owner and Source System of Record

The table below identifies the data owner and systems of record used to support AI-assisted underwriting decisions.

| **Item**               | **Description**                                       |
|------------------------|-------------------------------------------------------|
| Data Owner             | Consumer Lending Risk Management                      |
| applicants.csv         | Applicant profile and application information         |
| credit_pull_events.csv | Credit bureau results and underwriting evidence       |
| loan_products.csv      | Product eligibility requirements                      |
| loan_applications.csv  | Final underwriting decision and supporting references |

Together, these structured datasets provide the information needed to evaluate personal loan applications and support AI-assisted underwriting decisions.

## Classification and Access Considerations

The datasets contain applicant financial information and internal underwriting records. They should be used only by authorized lending, credit risk, compliance, and internal audit personnel. Fair-lending monitoring information should remain separate from the operational underwriting data so that protected attributes are not used during automated lending decisions.

## Freshness and Point-in-Time Requirements

The AI system must evaluate each application using the latest successful credit-pull event available before the decision date. The underwriting policy version must be selected based on the application date, and historical records should remain unchanged so previous decisions can be reproduced and audited.

## Important Contextual Metadata

The table below lists metadata fields that preserve business context and decision traceability.

| **Metadata Field**  | **Purpose**                                                                             |
|---------------------|-----------------------------------------------------------------------------------------|
| Application date    | Determines which underwriting policy version applies.                                   |
| Event date          | Indicates when the credit pull occurred.                                                |
| Product code        | Identifies the selected loan product and applicable eligibility rules.                  |
| policy_doc_id       | Records the underwriting policy version used for the decision.                          |
| rule_id_triggered   | Records the primary rule that approved, denied, or placed the application Under Review. |
| provider_request_id | Links the decision to the original credit bureau request.                               |
| pull_status         | Indicates whether the credit pull was SUCCESS, FAILED, or PENDING.                      |

These metadata preserve the business context required to interpret every underwriting decision consistently and provide complete decision traceability.
