# DSBA 6171: [Domain Name] Data & Knowledge Microcosm

## 1. Team Information & Roles
* **Team Number / Name:** Team 01 - FinTech
* **Team Lead / Liaison:** Marie Vrablic
* **Data & Ingestion Lead:** Aswani Karaturi
* **Database & Analytics Lead:** Marie Vrablic
* **Knowledge & Retrieval Lead:** Sofia Martinez
* **Quality & Governance Lead:** Wenya Perry
* **Domain & Strategy Support:** Bobby Deasy
---

## 2. Business Context & Problem Framing
* **Assigned Domain:** FinTech: Credit & Loan Review 
* **Business Function / Process:** Personal Loan Approval and Fair-Lending Review
* **Target Stakeholders:** Loan underwriting officers, credit-risk managers, fair-lending compliance auditors, and internal audit teams
* **Primary Decision Question:** Should a personal loan application be approved, denied, or escalated for manual review based on the applicant’s credit risk, the selected loan product, and the underwriting policies in effect on the application date?
* **Supporting Business Questions:**
  1. Does the applicant satisfy the applicable eligibility requirements, including minimum credit score, income, debt-to-income ratio, and requested-loan limits?
  2. Was the lending decision applied consistently across applicants, without unexplained differences among protected demographic groups?
  3. Can the decision be explained and supported by accurate application data, valid reason codes, and the correct version of the underwriting and fair-lending policies?

## 3. Microcosm Assets Overview

### Structured Operational Datasets (`data/structured/raw/`)
* applicants.csv - Stores applicant demographic, employment, income, and credit-risk information. Contains 150 rows. Primary key: applicant_id.
* loan_products.csv - Stores loan-product terms, pricing baselines, risk criteria, maximum amounts, and DTI limits. Contains 2 rows. Primary key: product_code.
* loan_applications.csv - Stores loan applications and their underwriting decisions, approved amounts, terms, and supporting policy references. Contains 150 rows. Primary key: application_id. Foreign keys: applicant_id and product_code.
* credit_pull_events.csv - Stores credit-bureau inquiry events, credit scores, DTI ratios, risk classifications, and pull outcomes. Contains 200 rows. Primary key: credit_pull_event_id. Foreign keys: applicant_id and product_code.
  
### Knowledge Corpus (`data/documents/source/`)
* Contains **[X]** total documents (**[Y]** PDFs, **[Z]** TXT/MD files).
* Includes structural challenges (tables, hierarchical headers) and version state diversity (e.g., Current vs. Superseded).

---

## 4. Structured Signal to Knowledge Linkage
## Product DTI Eligibility

| Linkage component | Project-specific mapping |
|---|---|
| **Business question** | Does the applicant satisfy the DTI eligibility requirement for the selected personal-loan product? |
| **Example application** | The application selects product `MNT-01`. The applicant’s qualifying DTI is `39.00%`, while the product’s maximum permitted DTI is `36.50%`. |
| **Structured signal** | `loan_applications.application_id = AP-10039`<br><br>`loan_applications.product_code = MNT-01`<br><br>`credit_pull_events.dti_ratio = 39.24` provides the applicant’s qualifying DTI.<br><br>`loan_products.product_code = MNT-01` connects the application to its product rules.<br><br>`loan_products.max_dti_score = 36.50` provides the product limit.<br><br>`loan_applications.policy_doc_id = FIN-POL-001` identifies the underwriting policy.<br><br>`loan_applications.doc_id = FIN-RULE-001` identifies the product-rules document.<br><br>`loan_applications.rule_id_triggered = UW-DTI-001` records the specific rule responsible for escalation. |
| **Knowledge assets linked** | `underwriting_policy_2026.md` — Section 7, **Debt-to-Income Review**.<br><br>`loan_product_eligibility_rules.md` — Section 3, **Current Product Eligibility Table**.<br><br>`loan_approval_escalation_procedure.md` — Step 8, **Review DTI**, and Section 9, **Decision Reason-Code Catalog**. |
| **Current product and policy language needed** | The current product table establishes `36.50%` as the maximum DTI for `MNT-01`.<br><br>The current underwriting policy requires the system to compare `credit_pull_events.dti_ratio` with `loan_products.max_dti_score` for the selected product.<br><br>- If DTI is at or below the product maximum, the review continues.<br>- If DTI exceeds the product maximum, the application must be assigned `Under Review` with rule ID `UW-DTI-001`.<br>- If either DTI value is missing or invalid, the application must be assigned `Under Review` with rule ID `UW-DATA-004`.<br>- Exceeding the DTI limit does not automatically produce `Denied` unless the applicable current policy expressly requires denial. |
| **Rule evaluation** | `39.00 > 36.50`. Therefore, the applicant exceeds the maximum DTI permitted for `MNT-01`. |
| **Required output** | `decision_status = Under Review`<br><br>`rule_id_triggered = UW-DTI-001` |
| **Why the linkage matters** | This linkage enables the AI system to determine whether the loan decision is consistent with the current product rules and underwriting policy. It connects the applicant’s structured DTI value and product code to the precise rule governing the decision, produces the correct escalation result, and preserves evidence explaining why the application was placed `Under Review`. |

## Fair Lending Consistency Review

| Linkage component | Project-specific mapping |
|---|---|
| **Business question** | Were personal-loan decisions applied consistently and in accordance with the underwriting rules in effect on the application date, without unexplained differences among protected demographic groups? |
| **Applications reviewed** | **Application 1:** `application_id = LA-10063`; `applicant_id = AP-10063`.<br><br>**Application 2:** `application_id = LA-10143`; `applicant_id = AP-10143`. |
| **Structured signal** | Both applications have `product_code = MNT-01` and `policy_doc_id = FIN-POL-001`.<br><br>**AP-10063:** credit score `551`; income `$117,213.00`; employment duration `171 months`; reported DTI approximately `28.05%`; approved amount `$22,000`; approved term `72 months`; decision `Approved`; recorded rule `UW-APP-001`.<br><br>**AP-10143:** credit score `550`; income `$72,553.66`; employment duration `141 months`; DTI `36.96%`; approved amount `$19,000`; approved term `36 months`; decision `Approved`; recorded rule `UW-APP-001`. |
| **Structured joins** | `loan_applications.applicant_id` → `applicants.applicant_id`.<br><br>`loan_applications.product_code` → `loan_products.product_code`.<br><br>`credit_pull_events.applicant_id` → `applicants.applicant_id`.<br><br>`credit_pull_events.product_code` → `loan_products.product_code`.<br><br>The selected credit event must be active, successful, product-matched, and available before the decision date. |
| **Knowledge assets linked** | `underwriting_policy_2026.md` — Section 6.1, **Credit Score and Risk Mapping**; Section 6.2, **Product Risk Eligibility**; Section 7, **Debt-to-Income Review**; Section 8, **Amount and Term Review**; Section 9, **Decision Outcomes**; and Section 11, **Fair-Lending Controls**.<br><br>`loan_product_eligibility_rules.md` — Section 2.1, **Interpretation of risk_status_criteria**, and Section 3, **Current Product Eligibility Table**.<br><br>`loan_approval_escalation_procedure.md` — Steps 5–9, Section 9, **Decision Reason-Code Catalog**, and Section 10, **Fair-Lending Review Process**.<br><br>`fair_lending_review_memo.md` — Section 5, **Identifying Similarly Situated Applicants**, and Section 7, **Escalation Triggers**. |
| **Current policy language needed** | Under `FIN-POL-001`, credit scores from `300–579` map to **Subprime** and **Very High Risk**.<br><br>The `MNT-01` product permits applicants no riskier than **Moderate Risk** for normal eligibility. A risk status exceeding the product threshold requires `Denied` with rule ID `UW-CRD-001`.<br><br>`MNT-01` permits terms from `24–60 months` and has a maximum DTI of `36.50%`. A term outside the permitted range requires `Under Review` with `UW-TRM-001`. DTI above the product maximum requires `Under Review` with `UW-DTI-001`. |
| **AP-10063 rule evaluation** | Credit score `551` maps to **Very High Risk**, which exceeds the `MNT-01` maximum of **Moderate Risk**. The approved term of `72 months` also exceeds the product maximum of `60 months`. Therefore, `Approved` with `UW-APP-001` is inconsistent with the current policy. |
| **AP-10143 rule evaluation** | Credit score `550` maps to **Very High Risk**, which exceeds the `MNT-01` risk limit. DTI of `36.96%` also exceeds the product maximum of `36.50%`. Therefore, `Approved` with `UW-APP-001` is inconsistent with the current policy. |
| **Fair-lending interpretation** | These two records do not yet demonstrate different treatment because both decisions are `Approved`. They demonstrate possible policy-application or data-quality failures. |
| **Why the linkage matters** | This linkage allows the AI system to determine whether the recorded approvals are consistent with the current product and underwriting rules. It also identifies records that require compliance screening before making a fair-lending conclusion. The system can trace each structured value to the controlling policy language and distinguish a policy violation from an actual protected-group treatment difference. |
---

## 5. Controlled Quality Issues 


## 6. Business Risk Matrix Summary
| Quality Problem | Affected Layer | AI Impact | Business Consequence | Future Control |
|---|---|---|---|---|
| Policy version is selected using `decision_date` instead of `application_date`. For example, AP-10016 and AP-10031 applied in 2025 but were initially linked to the 2026 policy because their decisions occurred in 2026. | Knowledge Linkage / Temporal Logic | The AI retrieves underwriting requirements that were not in effect when the applicants applied. | Incorrect decisions, failed historical reconstruction, and compliance risk. | Select the policy using `application_date` and documented effective dates. Link 2024–2025 applications to `FIN-POL-001-OLD` and applications from 2026 onward to `FIN-POL-001`. |
| A credit-pull event occurring after `decision_date` is used as decision evidence. The identified credit pulls for AP-10016 and AP-10031 occurred after their recorded decisions. | Event Data / Temporal Retrieval | The AI uses future information that was unavailable when the original decision was made. | Data leakage, incorrect risk assessment, and an unsupported audit trail. | Require `application_date <= event_date <= decision_date`. Use only active, successful, applicant-matched, and product-matched credit pulls available before the decision. |
| `requested_amount` and `requested_term_length` are missing, while approved values may be treated as application inputs. | Schema / Decision Logic | The AI cannot determine whether the applicant’s original request satisfied the product amount and term limits. | False approvals and incomplete decision explanations. | Add `requested_amount` and `requested_term_length` to `loan_applications.csv`. Never substitute `approved_amount` or `approved_term_length` for the requested values. |
| `doc_id`, `policy_doc_id`, or `rule_id_triggered` does not match the identifiers used in the policy corpus. | Knowledge Linkage / Metadata | The AI cannot retrieve the exact document, version, and rule supporting the decision. | Failed traceability, weak explanations, and audit failure. | Maintain a controlled document-and-rule registry. Validate every ID against document metadata and preserve the policy version effective on the application date. |
| Protected-group monitoring attributes are unavailable or are mixed into underwriting inputs. | Fair-Lending Analytics / Access Control | The AI either cannot test fair-lending consistency or may improperly use protected information during decisioning. | The fairness business question cannot be tested reliably, and inappropriate use could create compliance exposure. | Store synthetic or lawfully available monitoring attributes in a restricted `fair_lending_monitoring.csv`. Use them only for post-decision compliance testing, never for approval, denial, pricing, amount, term, or risk classification. |


## 7. How to Run & Environment Setup
1. Clone this repository to your local machine.
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
