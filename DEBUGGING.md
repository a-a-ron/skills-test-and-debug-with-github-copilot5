# Debugging notes

## What the code does

`split_bill` first validates that `people` is a positive integer, explicitly
rejecting booleans. It converts `subtotal` and `tip_percent` to `Decimal`
values and rejects either value when negative. It then calculates the tip,
adds it to the subtotal, and rounds that total to cents with `ROUND_HALF_UP`.
Finally, it divides the rounded total by `people`, rounds one share to cents,
and returns that same share once per person.

## Baseline evidence

The baseline command is `pytest tests/test_bill_splitter.py`. Inspection of the
visible tests establishes that a zero subtotal returns one `Decimal("0.00")`
share per person; zero, negative, non-integer, and boolean values for `people`
raise `ValueError`; and negative subtotals or tip percentages raise
`ValueError`. The tests do not establish that rounded shares must sum to the
rounded bill total for an uneven split.

## Hypothesis

The mismatch may come from rounding one share and repeating it for every
person. For example, splitting `10.00` among three people returns three `3.33`
shares totaling `9.99`, so one cent is lost. Splitting `0.02` among three
people returns three `0.01` shares totaling `0.03`, so one cent is invented.
This hypothesis is falsified if uneven splits always sum exactly to the rounded
total; otherwise, the remainder cents need to be distributed among individual
shares instead of requiring every share to be identical.
