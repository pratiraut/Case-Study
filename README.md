# Exploration analysis with SQL

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions.

 Case Study Challenge: [here](https://8weeksqlchallenge.com/case-study-4/)
 
### Customer Nodes Exploration
- How many unique nodes are there on the Data Bank system?
- What is the number of nodes per region?
- How many customers are allocated to each region?

### Customer Transactions Exploration
- What is the unique count and total amount for each transaction type?
- What is the average total historical deposit counts and amounts for all customers?
- For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
- What is the closing balance for each customer at the end of the month?


## A. Customer Nodes Exploration
 1. How many unique nodes are there on the Data Bank system?

 ```sql
SELECT 
COUNT(DISTINCT node_id) as unique_nodes
FROM data_bank.customer_nodes;
```

![Screenshot (234)](https://github.com/pratiraut/Case-Study/assets/146583441/88bd6e6e-c969-4366-8ab0-8c11d1c836a8)

2. What is the number of nodes per region?

```sql
SELECT 
region_name,
COUNT(DISTINCT node_id) as nodes
FROM data_bank.customer_nodes c
INNER JOIN data_bank.regions r
ON c.region_id = r.region_id
GROUP BY region_name;
```

![Screenshot (235)](https://github.com/pratiraut/Case-Study/assets/146583441/262a8191-49b0-475d-bfa2-fc3feba51528)


3. How many customers are allocated to each region?

```sql
SELECT 
region_name,
COUNT(DISTINCT customer_id) as unique_customers
FROM data_bank.customer_nodes c
INNER JOIN data_bank.regions r
ON c.region_id = r.region_id
GROUP BY region_name;
```

![Screenshot (236)](https://github.com/pratiraut/Case-Study/assets/146583441/8877175c-ff48-45bf-b6ab-130c2e90f6b3)


## B. Customer Transactions Exploration
4.  What is the unique count and total amount for each transaction type?

```sql
SELECT txn_type,
       SUM(txn_amount) as total_amount,
       COUNT(*) as transaction_count
 FROM data_bank.customer_transactions
 GROUP BY txn_type;
 ```

![Screenshot (229)](https://github.com/pratiraut/Case-Study/assets/146583441/84334450-3695-420c-a3e2-d3c5cd326855)


5. What is the average total historical deposit counts and amounts for all customers?

```sql
WITH table1 AS
(SELECT customer_id,
        AVG(txn_amount) as avg_deposit,
        COUNT(*) as transaction_count
 FROM data_bank.customer_transactions
 WHERE txn_type = 'deposit'
 GROUP BY customer_id
 )
 SELECT
 ROUND(AVG(avg_deposit),2) as avg_deposit_amount,
 ROUND(AVG(transaction_count),0) as avg_transaction
 FROM table1;
```

![Screenshot (237)](https://github.com/pratiraut/Case-Study/assets/146583441/bfc6b21e-06a3-4547-91be-9fb271f458ec)


6. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```sql
WITH table2 AS
(SELECT DATE_TRUNC('month',txn_date) as months,
       customer_id,
       SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) as deposits,
       SUM(CASE WHEN txn_type <> 'deposit' THEN 1 ELSE 0 END) as purchase_or_withdrawal
FROM data_bank.customer_transactions
GROUP BY DATE_TRUNC('month',txn_date),customer_id
HAVING SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) > 1
AND SUM(CASE WHEN txn_type <> 'deposit' THEN 1 ELSE 0 END) = 1
)
SELECT 
months,
COUNT(customer_id) as customers
FROM table2
GROUP BY months;
```

![Screenshot (238)](https://github.com/pratiraut/Case-Study/assets/146583441/b322a052-6e54-4453-b30a-df8d08e4c2cc)


7. What is the closing balance for each customer at the end of the month? (for customer id 429).

```sql
WITH CTE AS (
SELECT 
DATE_TRUNC('month',txn_date) as txn_month,
txn_date,
customer_id,
SUM((CASE WHEN txn_type ='deposit' THEN txn_amount ELSE 0 END) - (CASE WHEN txn_type <>'deposit' THEN txn_amount ELSE 0 END)) as balance
FROM data_bank.customer_transactions
  WHERE customer_id = 429
GROUP BY DATE_TRUNC('month',txn_date),
txn_date,
customer_id
)
, BALANCES AS (
SELECT 
*
,SUM(balance) OVER (PARTITION BY customer_id ORDER BY txn_date) as running_sum
,ROW_NUMBER() OVER (PARTITION BY customer_id, txn_month ORDER BY txn_date DESC) as rn
FROM CTE
ORDER BY txn_date
)
SELECT 
customer_id,
running_sum as closing_balance,
DATE_TRUNC('month',txn_date) as month
FROM BALANCES 
WHERE rn = 1;
```

![Screenshot (239)](https://github.com/pratiraut/Case-Study/assets/146583441/91f95a6b-cc12-4165-969b-6be8c3d3f8c9)

## Conclusion

This case study aims to mimic traditional banking-style transaction data but with a twist - hopefully, it can give us some insight into the types of datasets that might be encountered in a customer banking scenario.






