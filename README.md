This SQL script extracts and aggregates loan payment data to provide insights into customer payment behavior. It consists of three main components:

Loan Data Extraction

Retrieves distinct loan details, including client type, loan status, and payment classification.
Extracts key date/time attributes (e.g., closed hour, day of the week).
Payment Aggregation

Computes the total promised-to-pay (PTP) amounts from loan payment data.
Ensures accurate aggregation of payments made within the promise period.
Final Data Aggregation

Joins loan details with aggregated payment data.
Summarizes loan charges, outstanding balances, and PTP amounts by due date.
