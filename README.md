# DSBA 6171: [Domain Name] Data & Knowledge Microcosm

## 1. Team Information & Roles
* **Team Number / Name:** [e.g., Team 01 - FinTech]
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
* `file_1.csv` - [Brief description, row count, primary key]
* `file_2.csv` - [Brief description, row count, primary key]
* `file_3.csv` - [Brief description, row count, primary key & foreign keys]

### Knowledge Corpus (`data/documents/source/`)
* Contains **[X]** total documents (**[Y]** PDFs, **[Z]** TXT/MD files).
* Includes structural challenges (tables, hierarchical headers) and version state diversity (e.g., Current vs. Superseded).

---

## 4. Structured Signal to Knowledge Linkage
* **Signal Example 1:** [Structured Field / Event] -> [Policy Document ID]
  * *Why it matters:* [Explanation of business impact]
* **Signal Example 2:** [Structured Field / Event] -> [Policy Document ID]
  * *Why it matters:* [Explanation of business impact]

---

## 5. Controlled Quality Issues & Risk Matrix Summary
##Option 1
| Quality Problem | Affected Layer | AI Impact | Business Consequence | Future Control |
| :--- | :--- | :--- | :--- | :--- |
| A duplicate credit-pull event is intentionally included for one applicant. | Ingestion  | The AI may use the same credit event twice or retrieve conflicting credit information. | Incorrect risk assessment | Primary-key validation and deduplication gate |
| A superseded underwriting policy is incorrectly labeled as current. | Knowledge Corpus | Hallucination on superseded rules | Compliance failure | Authority filter |
| One loan application contains a missing or invalid product_code. | Data quality / Relational join | The AI cannot connect the application to the correct loan product and eligibility rules. | Partial analytics | Foreign-key validation and schema contract |

##Option 2 Mortgage
| Quality Problem | Affected Layer | AI Impact | Business Consequence | Future Control |
| :--- | :--- | :--- | :--- | :--- |
| One appraisal photograph is intentionally assigned to the wrong property_id. | Ingestion / Vector Store | The AI may associate another property’s condition with the mortgage application under review. | Incorrect collateral assessment, valuation conclusion, or escalation decision | Document-image manifest, checksum validation, and property-ID verification |
| A superseded underwriting policy is incorrectly labeled as current. | Knowledge Corpus | Hallucination on superseded rules | Compliance failure | Authority filter |
| One appraisal record contains a missing or invalid appraisal_id or application_id. | Data quality / Relational join | The AI cannot connect the mortgage application to its appraisal report, valuation table, and property photographs. | Unsupported collateral decision or incomplete audit evidence | Foreign-key validation, schema contract, and evidence-completeness check |


## 6. How to Run & Environment Setup
1. Clone this repository to your local machine.
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
