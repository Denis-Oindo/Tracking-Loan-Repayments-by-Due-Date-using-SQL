This SQL script extracts and aggregates loan payment data to provide insights into customer payment behavior. 
It consists of three main components:

1. Loan Data Extraction
Retrieves distinct loan details, including client type, loan status, and payment classification.
Extracts key date/time attributes (e.g., closed hour, day of the week).

2. Payment Aggregation
Computes the total promised-to-pay (PTP) amounts from loan payment data.
Ensures accurate aggregation of payments made within the promise period.

3. Final Data Aggregation
Joins loan details with aggregated payment data.
Summarizes loan charges, outstanding balances, and PTP amounts by due date.

This analysis is useful for loan portfolio analysis, collections strategy optimization, and customer repayment behavior tracking.

![image](https://github.com/user-attachments/assets/f8457880-523c-4863-921b-1bfc8c8e2aee)
