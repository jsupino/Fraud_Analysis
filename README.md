## Credit Card Fraud Detection & Analysis ##

## Overview ##
This project simulates and analyzes credit card fraud using a randomized Python-generated dataset, SQL-based fraud detection rules, and PowerBI visualizations. It focuses on identifying anomalous patterns of fradulent activity in credit card transactions based on amount, time, and location.

## Dataset Generation ##
A dataset of 1,000 randomized credit card transactions was created using Python and saved as *transaction_dataset.csv*. The dataset includes the following fields:
- transaction_id (INT): Auto-incremented identifier
- user_id (INT): Randomizer user reference
- date (DATETIME): Random timestamp
- amount (DOUBLE): Transaction amount
- category (TEXT): Merchant category from a pre-defined list
- location (TEXT): Random popular U.S. cities from a pre-defined list

## SQL Schema & Data Import ##
- A schema named *fraud_detection* was created in MySQL workbench.
  
        USE fraud_detection;
  
- The CSV file was imported using the Table Data Import Wizard.
Two additional columns were added to the *transactions* table:
- IsFraud (BOOLEAN): based on the following rules before
- FraudReason (VARCHAR)

## Fraud Detection Rules ##
The following rules were applied to flag transactions as fraud:

1. Hihgh-Value Transactions (>$10,000)

        UPDATE transactions
        SET IsFraud = 1,
        	FraudReason = "Over $10,000"
        WHERE amount > 10000.00;

3. Suspicious Time Window (Between 2 AM - 5 AM)
   
        UPDATE transactions
        SET IsFraud = 1,
        	FraudReason = "Between 2 and 5 AM"
        WHERE HOUR(date) between 2 and 4;

5. Geographic Anomalies (More than 6 unique locations per user)
   
        UPDATE transactions
        SET IsFraud = 1,
        	FraudReason = "Multiple Locations"
        WHERE user_id IN (
        	SELECT user_id
            FROM (
        		select user_id
                FROM transactions
                GROUP BY user_id
                HAVING COUNT(DISTINCT location) > 6
        	) AS temp
        );

7. Transaction Velocity (>= 5 transactions per hour)
   
          UPDATE transactions
          SET IsFraud = 1,
          	FraudReason = "5+ Transactions/Hour"
          WHERE (user_id, DATE(date), HOUR(date)) IN (
              SELECT user_id, DATE(date), HOUR(date)
              FROM (
                  SELECT user_id, DATE(date), HOUR(date)
                  FROM transactions
                  GROUP BY user_id, DATE(date), HOUR(date)
                  HAVING COUNT(*) >= 5
              ) AS suspicious
          );

To view final dataset:

          SELECT * FROM transactions;

## Power BI Analysis ##
PowerBI was used to visualize and derive insights from the processed data:
- Donut Chart: Percentage of fraudulent vs. legitimate transactions, segmented by fraud reason
- Top 10 Locations Bar Chart: Displays the 10 U.S. cities with the most fraud, broken down by merchant category
- Total Fraud Cases by Reason Bar Chart: Number of fraudulent cases per rule
- Line Chart: Timeline of fradulent spending by fraud reason to detect patterns over time

**High-Value Fraud**
- Transactions over $10,000

**Time-Based Fraud**
- A notable amount of fraudulent transactions occurred between the hours of 2 AM and 5 AM with a total of 11%. This showed a high potenial card misue during off-hours.

**Geographic Anomalies**
- users with transactions from more than six unique U.S. cities were flagged, highlighting potential compromised accounts or travel-related fraud.

**Rapid Transaction Frequency**
- Many users flagged under the 5+ transactions/hour rule showed possible bot or automated fraud behavior.

## Files Included ##
transaction_dataset_creation.py - Python script to create transaction_dataset.csv
transaction_dataset.csv - Simulated credit card transaction dataset from the Python script
transactions_updated.csv - updated transaction dataset with the fraud labels and reasons
fraud_dashboard: PowerBI dashboard
fraud_dashboard.pdf - PowerBI dashboard as a PDF

## Conclusion ##
This project demonstrates how rule-based SQL detection and interactive dashboarding in Power BI can be used to identify and interpret suspicious activity in financial data. It serves as a foundation for future enhancements using machine learning, anomaly detection, or real-time fraud monitoring systems.'
