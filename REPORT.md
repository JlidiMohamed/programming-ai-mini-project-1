# Mini-project 1 — Service Ticket Data Pipeline

## Diagnosis
Each row of `service_tickets.csv` is a ticket *observation*, so one `ticket_id` may appear several times. The raw file has 84 rows. I distinguished values that are missing in the source from values that are present but invalid (e.g. negative minutes, fractional ratings, dates outside October 2026).

## Rules
1. The raw CSV is never modified; the pipeline works on a copy.
2. Only exact duplicate rows are removed (3 extra copies). Every occurrence of an ID that is still repeated with conflicting data is rejected, as are rows with a missing or malformed ID (not `T` + 3 digits) or a missing, unparseable or out-of-October-2026 date. All reasons are kept in `rejection_reason`; each row is counted once. Rejected rows: 4 (conflicting_id: 2, missing_id: 1, unparseable_date: 1).
3. `channel` and `priority` are stripped and lowercased; missing or unrecognised values become `unknown` with flags (unknown channels: 2, unknown priorities: 2).
4. Invalid or missing `handling_minutes` are filled with the median of valid retained durations (59.0) and flagged: 3 rows. Zero and fractional values are valid. If no valid duration exists, the code raises an error.
5. `satisfaction` is valid only as an integer 1-5; otherwise it becomes missing and is flagged (`satisfaction_unavailable`: 3). It is never imputed, because that would invent customer opinions.
6. `handling_zscore` uses mean 59.590 and population standard deviation 30.595 (ddof=0; constant column gives zeros). The feature table holds the ID, the z-score and four one-hot channel columns; the rating is not a feature.

## Checks
Row balance: 84 = 3 duplicates + 4 rejected + 77 retained. Retained IDs are well formed and unique, dates fall in October 2026, durations are finite and nonnegative, categories are known or `unknown`, ratings are integers 1-5 or missing, feature IDs match the clean table with one channel indicator per row, and all files were read back successfully. Edge cases (conflicting IDs, multiple reasons, constant duration, no valid duration) are tested in section 7.

## Limitations
- Rejecting every row of a conflicting ID removes tickets that may be valid, which shrinks the sample and may bias it.
- Median imputation shrinks the variance of durations, and missing ratings may not be random, so observed satisfaction may not represent all tickets. The data are also synthetic.

## Run instructions
Open `project_1_student.ipynb` in Google Colab, upload `service_tickets.csv` in the Files panel, choose Runtime > Restart and run all. Outputs are written to `outputs/`. Locally: `pip install numpy pandas jupyter`, then run the notebook from the repository root.

## Assistance used
Describe the assistance you used here (e.g. an AI assistant helped draft the code structure; I reviewed, ran and can explain every step).
