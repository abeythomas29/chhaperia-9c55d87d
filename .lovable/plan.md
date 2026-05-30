## Goal
Make the 36 Head Production form reliably show slitting entries as selectable sources.

## Plan
1. Update the 36 Head source loader in `src/pages/slitting/Head36Entry.tsx` to match the defensive fetch pattern already used elsewhere.
   - First try the richer select.
   - If the live backend rejects `gsm`, retry without that column instead of leaving the list empty.

2. Stop over-restricting the source list in 36 Head.
   - Remove the `slitting_manager_id = current user` filter so the source dropdown can use the same visible slitting pool the app already allows authenticated users to read.
   - Increase/remove the hard `limit(50)` so recent-but-not-top-50 entries do not disappear.

3. Preserve the source details panel even when `gsm` is unavailable.
   - Keep thickness from the row.
   - Derive GSM from notes when the column is missing, so the form still shows source info and saves consistent 36 Head entries.

4. Validate in the live preview.
   - Confirm the dropdown populates after opening 36 Head.
   - Confirm a newly created slitting entry becomes selectable as a source.
   - Confirm no browser 400 error remains for the source-loading request.

## Technical notes
- This is a frontend-only fix.
- I will not use backend schema tools for this because your app is intentionally pointed at the separate live backend, and the bug is in the query logic inside the UI.