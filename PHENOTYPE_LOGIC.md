# Phenotype Definitions & ICD Code Logic

## 1. File Structure: `pheno_defs.tsv`

The `pheno_defs.tsv` file is the primary configuration file for defining case and control groups. It reconciles ICD-9 and ICD-10 codes into a single actionable list for each phenotype.

### Column Dictionary

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `trait_description` | String | The unique identifier/name for the phenotype (e.g., *Atrial fibrillation*, *Asthma*). |
| `phecode` | Float/String | The primary PheCode group associated with the phenotype. |
| `sex` | String | Demographic filter. **Critical for analysis.**<br>Values: `Male`, `Female`, `Both`. |
| `Umbrella_ICD9_case_inclusion` | String | **Logic-based list.** Contains ICD-9 codes using the Umbrella Notation for defining cases. |
| `Umbrella_ICD10_case_inclusion` | String | **Logic-based list.** Contains ICD-10 codes using the Umbrella Notation for defining cases. |
| `full_list_ICD9_case_inclusion` | String | **Explicit list.** Comma-separated string of every specific ICD-9 code for cases. |
| `full_list_ICD10_case_inclusion` | String | **Explicit list.** Comma-separated string of every specific ICD-10 code for cases. |
| `Umbrella_ICD9_control_exclusion` | String | **Logic-based list.** Codes that must be excluded from the control group (ICD-9). |
| `Umbrella_ICD10_control_exclusion` | String | **Logic-based list.** Codes that must be excluded from the control group (ICD-10). |
| `full_list_ICD9_control_exclusion` | String | **Explicit list.** Full expansion of excluded ICD-9 codes for controls. |
| `full_list_ICD10_control_exclusion` | String | **Explicit list.** Full expansion of excluded ICD-10 codes for controls. |
| `note` | String | **Phenotype-specific instructions** (e.g., removal of overlapping T1D/T2D participants or additional case-identification requirements). |

---

## 2. Methodology: Umbrella Code Logic

To resolve ambiguity between parent and child codes, this dataset uses **Trailing Dot Notation** in the `Umbrella_` columns.

### A. Umbrella Codes (Parents)
- **Syntax:** Code ends with `.` (e.g., `493.` or `I48.`).
- **Action:** Include the code and all sub-codes.

### B. Singleton Codes (Subcodes)
- **Syntax:** Code has no trailing dot (e.g., `427.31`).
- **Action:** Include only the exact code listed.

### C. Visual Example

| Phenotype | Umbrella Column | Full List Column |
| :--- | :--- | :--- |
| **Asthma** | `493.` | `493, 493.0, 493.01, ...` |
| **AFib** | `427.31` | `427.31` |
| **CAD** | `414.0., 411.81` | Expanded `414.0.` + `411.81` |

---

## 3. Reference Files: `ICD9/10_definitions.tsv`

Lookup tables used for validation and code expansion.  
(ICD-9 and ICD-10 files share the same structure.)

---

## 4. Implementation Notes for Analysts

### Data Loading
- Always load ICD columns as strings (leading zeros must be preserved).

### Constructing Cohorts
1. **Filter by Sex** using the `sex` column.
2. **Identify Cases**
   - A participant is a case if they have ≥2 separate diagnosis code from `full_list_ICD*_case_inclusion`.
   - If this requirement cannot be implemented, document the limitation and notify the team.
3. **Identify Controls**
   - No case inclusion codes.
   - No control exclusion codes.
4. **Review the `note` Column**
   - Contains mandatory phenotype-specific rules.
   - Example: Participants with both T1D and T2D should be removed from analysis.
   - If a note cannot be implemented, explicitly report this.
