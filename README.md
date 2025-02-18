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

WITH loan_data AS (
    -- Code 1: Logic to select loan details
    SELECT DISTINCT ON (id) 
        id, 
        client_id, 
        loan_status, 
        issued_date, 
        issued_at, 
        CASE 
            WHEN loan_sequence_nr = 1 THEN 'New' 
            ELSE 'Repeated' 
        END AS client_type, 
        CASE 
            WHEN loan_status = 'PAID' THEN 'Paid' 
            WHEN loan_status = 'ACTIVE' AND COALESCE(derived_paid_total, 0) = 0 THEN 'Not_paid' 
            WHEN loan_status = 'ACTIVE' AND COALESCE(derived_paid_total, 0) > 0 THEN 'Partially_paid' 
            ELSE 'Other' 
        END AS payment_status, 
        closed_date, 
        original_closed_at, 
        EXTRACT(HOUR FROM original_closed_at) AS closed_hour, 
        TRIM(TO_CHAR(original_closed_at, 'Day')) AS closed_day_of_week, 
        due_date, 
        derived_principal_disbursed, 
        derived_charged_total, 
        derived_outstanding_total, 
        COALESCE(derived_paid_total, 0) AS derived_paid_total, 
        derived_past_due_total 
    FROM platform.loan 
    ORDER BY id, issued_date DESC
),
ptp_aggregated AS (
    -- Code 2: Logic to calculate payment amounts
    SELECT 
        sub.loan_id, 
        SUM(sub.amount_paid) AS Ptp_amount 
    FROM (
        SELECT 
            lp.loan_id, 
            CASE 
                WHEN SUM(CASE 
                    WHEN pa.payment_type = 'INCOMING' 
                        AND pa.value_date BETWEEN lp.promise_date AND lp.due_date 
                        AND lp.promise_status IN ('ACTIVE', 'PARTIALLY_PAID', 'PAID') 
                    THEN CASE WHEN pa.amount > lp.promise_amount THEN lp.promise_amount ELSE pa.amount END 
                    ELSE 0 
                END) > lp.promise_amount 
                THEN lp.promise_amount 
                ELSE SUM(CASE 
                    WHEN pa.payment_type = 'INCOMING' 
                        AND pa.value_date BETWEEN lp.promise_date AND lp.due_date 
                        AND lp.promise_status IN ('ACTIVE', 'PARTIALLY_PAID', 'PAID') 
                    THEN CASE WHEN pa.amount > lp.promise_amount THEN lp.promise_amount ELSE pa.amount END 
                    ELSE 0 
                END) 
            END AS amount_paid 
        FROM platform.loan_promise_to_pay lp 
        LEFT JOIN platform.payment_allocation pa 
            ON lp.loan_id = pa.loan_id 
        WHERE lp.promise_status IN ('ACTIVE', 'PARTIALLY_PAID', 'PAID') 
        GROUP BY lp.created_at, lp.updated_at, lp.created_by, lp.loan_id, lp.promise_amount, lp.promise_date, lp.due_date
    ) sub 
    GROUP BY sub.loan_id
)
-- Final SELECT to join loan data with aggregated payment data and SUM the totals by due_date
SELECT 
    ld.due_date, 
    SUM(ld.derived_charged_total) AS total_derived_charged, 
    SUM(ld.derived_paid_total) AS total_derived_paid, 
    SUM(ld.derived_outstanding_total) AS total_derived_outstanding, 
    SUM(pa.Ptp_amount) AS total_ptp_amount 
FROM loan_data ld 
LEFT JOIN ptp_aggregated pa 
    ON ld.id = pa.loan_id 
WHERE ld.due_date BETWEEN COALESCE('2025-01-01', ld.due_date) 
                     AND COALESCE('2025-01-31', ld.due_date) 
GROUP BY ld.due_date 
ORDER BY ld.due_date;

![image](https://github.com/user-attachments/assets/f8457880-523c-4863-921b-1bfc8c8e2aee)
