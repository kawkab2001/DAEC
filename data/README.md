# Data

Place the three dataset CSV files here before running the notebooks:

| File | Domain | Rows |
|------|--------|------|
| `apple_privacy_hallucination_10000.csv` | Apple Privacy Policy | 10,000 |
| `x_privacy_policy_10000.csv` | X (Twitter) Privacy Policy | 10,000 |
| `meta_human_rights_policy_10000.csv` | Meta Human Rights Policy | 10,000 |

## Schema

Each CSV must contain the following 13 columns:

| Column | Type | Description |
|--------|------|-------------|
| `id` | str | Unique row identifier |
| `document` | str | Source document name |
| `document_version` | str | Document version tag |
| `topic` | str | Policy topic category |
| `question` | str | Question the claim answers |
| `context` | str | Passage from the source document |
| `claim` | str | Claim to be verified |
| `label` | str | Ground truth: `SUPPORTED`, `UNSUPPORTED`, `CONTRADICTED`, or `HALLUCINATED` |
| `evidence` | str | Supporting evidence span |
| `source_lines` | str | Line references in the source document |
| `generation_type` | str | How the sample was generated |
| `difficulty` | str | Difficulty tier |
| `variant_id` | str | Variant identifier for grouping |

## Notes

- CSV files are excluded from version control via `.gitignore` due to their size.
- If using Git LFS, track them with: `git lfs track "data/*.csv"`
- The notebooks default to loading files directly from this directory; update `DATA_PATH` / `DATASETS` at the top of each notebook if your paths differ.
