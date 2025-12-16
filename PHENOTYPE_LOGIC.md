# Phenotype Definitions & ICD Code Logic

## 1. File Structure: `pheno_defs.tsv`

The `pheno_defs.tsv` file is the primary configuration file for defining case and control groups. It reconciles ICD-9 and ICD-10 codes into a single actionable list for each phenotype.

### Column Dictionary

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `trait_description` | String | The unique identifier/name for the phenotype (e.g., *Atrial fibrillation*, *Asthma*). |
| `phecode` | Float/String | The primary PheCode group associated with the phenotype. |
| `sex` | String | Demographic filter. **Critical for analysis.**<br>Values: `Male`, `Female`, `Both`.<br>*Example:* Breast Cancer is limited to `Female`. |
| `Umbrella_ICD9_case_inclusion` | String | **Logic-based list.** Contains ICD-9 codes using the [Umbrella Notation](#2-methodology-umbrella-code-logic) for defining cases. |
| `Umbrella_ICD10_case_inclusion` | String | **Logic-based list.** Contains ICD-10 codes using the [Umbrella Notation](#2-methodology-umbrella-code-logic) for defining cases. |
| `full_list_ICD9_case_inclusion` | String | **Explicit list.** Comma-separated string of *every* specific ICD-9 code for cases. Use this for direct SQL matching. |
| `full_list_ICD10_case_inclusion` | String | **Explicit list.** Comma-separated string of *every* specific ICD-10 code for cases. |
| `Umbrella_ICD9_control_exclusion` | String | **Logic-based list.** Codes that must be excluded from the control group (ICD-9). |
| `Umbrella_ICD10_control_exclusion` | String | **Logic-based list.** Codes that must be excluded from the control group (ICD-10). |
| `full_list_ICD9_control_exclusion` | String | **Explicit list.** Full expansion of excluded ICD-9 codes for controls. |
| `full_list_ICD10_control_exclusion` | String | **Explicit list.** Full expansion of excluded ICD-10 codes for controls. |

---

## 2. Methodology: Umbrella Code Logic

To resolve ambiguity between "Parent" codes and specific "Child" codes, this dataset uses a **Trailing Dot Notation** within the `Umbrella_` columns.

### A. Umbrella Codes (Parents)
* **Syntax:** Code ends with a dot `.` (e.g., `493.` or `I48.`).
* **Meaning:** Represents the **highest level** of a category.
* **Action:** When parsing, you must extract this code **AND** all its sub-codes.
* **Example:**
    * Input: `493.` (Asthma)
    * Output: Captures `493`, `493.0`, `493.01`, `493.1`, `493.9`...

### B. Singleton Codes (Subcodes)
* **Syntax:** Code has **NO** trailing dot (e.g., `427.31` or `272.0`).
* **Meaning:** Represents a specific subcode that is the "highest level" of itself for the purposes of this study.
* **Action:** Extract **ONLY** this exact code. Do not recursively grab children.
* **Example:**
    * Input: `427.31` (Atrial Fibrillation).
    * Output: Captures only `427.31`. It does *not* capture `427.3` or `427`.

### C. Visual Example

| Phenotype | Umbrella Column (Logic) | Full List Column (Expanded) |
| :--- | :--- | :--- |
| **Asthma** | `493.` | `493, 493.0, 493.00, 493.01, ...` |
| **AFib** | `427.31` | `427.31` |
| **Coronary Artery Disease** | `414.0., 411.81` | `414.0, 414.00, 414.01...` (expanded `414.0.`)<br>`411.81` (kept as singleton) |

---

## 3. Reference Files: `ICD9/10_definitions.tsv`

These files are strictly for lookup and validation. They map the raw codes back to their human-readable descriptions and PheCodes.

### Column Dictionary

| Column Name | Description |
| :--- | :--- |
| `ICD9_diagnosis_code` | The specific ICD-9 code (Primary Key). |
| `trait_description` | Official medical description (e.g., *Extrinsic asthma with status asthmaticus*). |
| `phecode` | The associated PheCode (e.g., `495.2`). |
| `Inclusion/Exclusion` | Indicates if this code is used as an `Inclusion` criteria (Case) or `Exclusion` criteria (Control) for the mapped phenotype. |
| `phenotype` | The broad phenotype group (e.g., *Asthma*). |

*(Note: `ICD10_definitions.tsv` follows the exact same structure).*

---

## 4. Implementation Notes for Analysts

### Data Loading
* **Always load ICD columns as strings.**
* Many codes contain leading zeros (e.g., `070.`). Loading them as numeric types will truncate the zero (becoming `70.`), rendering the code invalid.

### Constructing Cohorts
1.  **Filter by Sex:** Before matching codes, filter your subject list by the `sex` column in `pheno_defs.tsv`.
    * *Example:* If `trait_description` == "Prostate Cancer", remove all Female subjects.
2.  **Match Cases:** Use the `full_list_ICD...` columns. A subject is a **Case** if they have **at least one** code from the case inclusion list.
3.  **Match Controls:** A subject is a **Control** if:
    * They have **NO** codes from the case inclusion list.
    * They have **NO** codes from the `full_list_..._control_exclusion` column.