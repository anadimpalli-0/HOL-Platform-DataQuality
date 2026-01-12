# Getting Started with Snowflake Data Quality

## Overview

Trusted data is essential for confident decision-making, analytics, and AI. Snowflake’s Data Quality Framework
provides a native, end-to-end approach for ensuring that data is accurate, complete, and reliable throughout its
lifecycle.
This framework combines proactive monitoring with interactive UI for root cause analysis and impact
assessment into a single experience. Built directly on Snowflake, it enables scalable, automated data quality
without requiring external tools or data movement.be more expensive for automatic clustering to maintain than those with an appropriate cardinality.

##Concepts:
-Data Profiling
-System and Custom DMF
-Expectations
-Data Quality UI
-Data metric scan
-Data Lineage
-[Anomaly Detection - PrPr]
-[Notifications - PrPr]

## Steps:
- Step 1 - Environment Setup
- Step 2 - Create sample data with quality issues
- Step 3 - Analyze dataset with profiling statistics
- Step 4 - Write DQ test using DMFs and Expectations
- Step 5 - Review DQ Dashboard
- Step 6 - Set up notification

# Step 1 - Environment Setup

```sql

--Create the dq_tutorial_role role to use throughout the tutorial:

USE ROLE ACCOUNTADMIN;
CREATE ROLE IF NOT EXISTS dq_tutorial_role;
GRANT CREATE DATABASE ON ACCOUNT TO ROLE dq_tutorial_role;
GRANT DATABASE ROLE SNOWFLAKE.USAGE_VIEWER TO ROLE dq_tutorial_role;


--Create a warehouse to query the table that contains the data and grant the USAGE privilege on the role to the dq_tutorial_role role:

CREATE WAREHOUSE IF NOT EXISTS dq_tutorial_wh;
GRANT USAGE ON WAREHOUSE dq_tutorial_wh TO ROLE dq_tutorial_role;

--Confirm the grants to the dq_tutorial_role role:

SHOW GRANTS TO ROLE dq_tutorial_role;

--Establish a role hierarchy and grant the role to a user

GRANT ROLE dq_tutorial_role TO ROLE SYSADMIN;
GRANT ROLE dq_tutorial_role TO USER <replace with your UserID>;


--Create Objects

USE ROLE dq_tutorial_role;

CREATE DATABASE IF NOT EXISTS dq_tutorial_db;
CREATE SCHEMA IF NOT EXISTS sch;

CREATE OR REPLACE TABLE CUSTOMER (
    ID FLOAT,                       
    FIRST_NAME VARCHAR,             
    LAST_NAME VARCHAR,              
    STREET_ADDRESS VARCHAR,         
    STATE VARCHAR,                  
    CITY VARCHAR,                   
    ZIP VARCHAR,                    
    PHONE_NUMBER VARCHAR,           
    EMAIL VARCHAR,                  
    SSN VARCHAR,                    
    BIRTHDATE VARCHAR,              
    JOB VARCHAR,                    
    CREDITCARD VARCHAR,             
    COMPANY VARCHAR,                
    OPTIN VARCHAR                   
);

-- Customer Orders in Bronze with day/month/year columns + PAYMENT_TYPE
CREATE OR REPLACE TABLE CUSTOMER_ORDERS (
    CUSTOMER_ID VARCHAR,            
    ORDER_ID VARCHAR,               
    ORDER_DAY INT,                 
    ORDER_MONTH INT,               
    ORDER_YEAR INT,                
    ORDER_AMOUNT FLOAT,             
    ORDER_TAX FLOAT,                
    ORDER_TOTAL FLOAT,              
    PAYMENT_TYPE VARCHAR  -- e.g., 'CREDIT_CARD', 'PAYPAL', 'BANK_TRANSFER'
);

```

# Step 2 - Create sample dataset with quality issues

Sample data setup - To begin, you'll need a sample dataset to work with. This section will walk you through the
process of setting up sample data for your Snowflake environment.
Consider a data pipeline following a modern data architecture pattern with three distinct layers following the
medallion architecture.
<img src="/images/Medallion_Architecture.png" width="70%">

Bronze Layer: Raw data from multiple sources (databases, JSON, XML files)
Silver Layer: Cleaned and enriched data with AI-powered insights
Gold Layer: Analytics-ready data stored in managed Iceberg tables and regular Snowflake tables.

## Bronze Layer (ingestion)

```sql
INSERT INTO CUSTOMER VALUES
(1, 'John', 'Doe', '123 Elm St', 'CA', 'San Francisco', '94105', '123-456-7890',
'john.doe@email.com', '111-22-3333', '1985-04-12', 'Engineer', '4111111111111111', 'Acme
Inc.', 'Y'),
(2, 'Jane', 'Smith', NULL, 'CA', 'Los Angeles', '90001', NULL, 'jane.smith@email.com',
NULL, '1990/07/25', 'Manager', '5500000000000004', 'Globex', 'N'),
(3, 'Mike', 'Brown', '456 Oak St', 'NV', 'Las Vegas', '89101', '9999999999',
'mike.brown.com', '123-45-6789', 'bad-date', 'Analyst', '4000000000000002', NULL, 'Y'),
(4, 'Anna', 'Lee', '789 Pine St', 'WA', 'Seattle', '98101', '206-555-1234',
'anna.lee@email.com', '222-33-4444', '1988-11-05', 'Designer', '340000000000009', 'Innotech',
'Y');

INSERT INTO CUSTOMER_ORDERS VALUES
    ('1', 'O1', 1, 9, 2025, 100, 8.25, 108.25, 'CREDIT_CARD'),
    ('2', 'O2', 2, 9, 2025, 200, NULL, 200, 'PAYPAL'),
    ('2', 'O3', 3, 9, 2025, 300, 60, 360, 'BANK_TRANSFER'),
    ('3', 'O4', NULL, 9, 2025, -50, 5, -45, 'CREDIT_CARD'),
    ('4', 'O5', 4, 9, 2025, 150, 12, 162, 'PAYPAL'),
    ('1', 'O6', 5, 9, 2025, 250, 20, 270, 'CREDIT_CARD'),
    ('5', NULL, 5, 9, 2025, 250, 20, 270, 'VENMO');

```

## Silver Layer (Cleaned Orders Only)

```sql
CREATE OR REPLACE TABLE SILVER_CUSTOMER_ORDERS AS
SELECT
CUSTOMER_ID,
ORDER_ID,
ORDER_TS,
ORDER_AMOUNT,
ORDER_TAX,
ORDER_TOTAL,
PAYMENT_TYPE
FROM CUSTOMER_ORDERS;
```

## Gold Layer (Aggregated by Customer State and Payment Type)

```sql
CREATE OR REPLACE TABLE GOLD_SALES_SUMMARY AS
SELECT
c.STATE,
o.PAYMENT_TYPE,
COUNT(DISTINCT o.ORDER_ID) AS TOTAL_ORDERS,
SUM(o.ORDER_AMOUNT) AS TOTAL_AMOUNT,
SUM(o.ORDER_TAX) AS TOTAL_TAX,
SUM(o.ORDER_TOTAL) AS TOTAL_REVENUE
FROM SILVER_CUSTOMER_ORDERS o
JOIN CUSTOMER c
ON c.ID = TO_NUMBER(o.CUSTOMER_ID)
WHERE o.ORDER_AMOUNT IS NOT NULL
GROUP BY c.STATE, o.PAYMENT_TYPE;

```
## Our setup looks like the following:
<img src="/images/GoldLayer_Lineage.png" width="70%">

# Step 3 - Analyze dataset with profiling statistics
The first step in the data quality lifecycle is data profiling. Data profiling is the process of analyzing a dataset to
understand its structure, content, and quality. It typically includes gathering statistics such as data types, value
distributions, null counts, and uniqueness to identify patterns and potential data quality issues.

Using Snowsight, you can easily profile a dataset by accessing the Data Quality Tab. This is an important role
in helping you get started with continuous data quality monitoring by laying the groundwork for identifying data
quality rules.
<img src="/images/Profiling.png" width="70%">

# Step 4 - Best practices for Monitoring Data Quality with DMFs and Expectations 
At each layer, we are interested in monitoring different aspects. 

- Bronze: Test source data 
- Silver: Validate transformations
- Gold: Test business logic 

1) Organizations typically take different approaches to monitoring and enforcing data quality. Some focus on Bronze layer to catch issues as early as possible, ensuring that all ingested data meets baseline expectations. 
Yet, bad data might emerge later in the pipeline. 
2) Others monitor Gold layer to prioritize end-user experience by ensuring that curated datasets remain accurate and reliable. However, this makes root-cause analysis more complex.
3) Silver layer to validate transformations and business logic. But this requires ongoing maintenance as transformation evolves.

While each strategy has merit, the most effective approach is hybrid monitoring across all three layers. This ensures comprehensive coverage, early detection of raw data issues, validation of transformations, and protection of business-critical outputs.

Snowflake Data Metric Functions
You can measure the quality of your data in Snowflake by using DMFs. Snowflake provides built-in [system DMFs](https://docs.snowflake.com/en/user-guide/data-quality-system-dmfs#system-dmfs) to measure common metrics without having to define them. 
You can also define your own custom DMFs to meet business-specific requirements.

All DMFs that are set on the table follow a [schedule](https://docs.snowflake.com/en/user-guide/data-quality-working#schedule-the-dmf-to-run) to automate the data quality measurement, and you can define a schedule to trigger DMFs based on time or based on DML events. After you schedule the DMFs to run, Snowflake records the results of the DMF in a dedicated [event table for data metric functions](https://docs.snowflake.com/en/user-guide/data-quality-results).

## DMF Privilage Setup
```sql
GRANT EXECUTE DATA METRIC FUNCTION ON ACCOUNT TO ROLE dq_tutorial_role;

GRANT DATABASE ROLE SNOWFLAKE.DATA_METRIC_USER TO ROLE dq_tutorial_role;

GRANT APPLICATION ROLE SNOWFLAKE.DATA_QUALITY_MONITORING_VIEWER TO ROLE dq_tutorial_role;
```

## Monitoring Bronze Layer
The Bronze layer is the raw, unprocessed storage zone where ingested data is captured in its original format before any cleaning, transformation, or enrichment. It is important to monitor this layer for data quality to ensure the integrity of source data before downstream transformation and analytics. 

The following example associates volume, freshness, and null checks using DMFs and Expectations.

```sql
ALTER TABLE CUSTOMER_ORDERS SET DATA_METRIC_SCHEDULE = '5 minutes';

ALTER TABLE CUSTOMER_ORDERS ADD DATA METRIC FUNCTION 
SNOWFLAKE.CORE.ROW_COUNT ON () Expectation Volume_Check (value > 1),   -- Row count (Volume)
SNOWFLAKE.CORE.FRESHNESS ON () Expectation Freshness_Check (value < 1800), 	-- Freshness
SNOWFLAKE.CORE.NULL_COUNT ON (ORDER_ID) Expectation Null_Check (value = 0);   -- Null count
```
## Monitoring Silver Layer
The Silver layer is the transformation and enrichment zone where raw data from the Bronze layer is cleaned, joined, and reshaped to create intermediate datasets. Monitoring this layer is important to validate that transformations are applied correctly, business rules are enforced, and data anomalies are caught before reaching business-facing datasets. Ensuring quality at this stage helps prevent errors from propagating downstream and supports reliable analytics and reporting.

Create a custom DMF for referential integrity check

```sql
CREATE OR REPLACE DATA METRIC FUNCTION referential_check(
  arg_t1 TABLE (arg_c1 VARCHAR), arg_t2 TABLE (arg_c2 VARCHAR))
RETURNS NUMBER AS
 'SELECT COUNT(*) FROM arg_t1
  WHERE arg_c1 NOT IN (SELECT arg_c2 FROM arg_t2)';
```

## Create a custom DMF that compares row counts between two tables

```sql
CREATE OR REPLACE DATA METRIC FUNCTION volume_check(
  arg_t1 TABLE (arg_c1 VARCHAR), arg_t2 TABLE (arg_c2 VARCHAR))
RETURNS NUMBER AS
  'SELECT ABS(
      (SELECT COUNT(*) FROM arg_t1) - (SELECT COUNT(*) FROM arg_t2)
    )';
  ```
  
  ## Associate Checks 
  ```sql
ALTER TABLE SILVER_CUSTOMER_ORDERS SET DATA_METRIC_SCHEDULE = '5 minutes';

ALTER TABLE SILVER_CUSTOMER_ORDERS ADD DATA METRIC FUNCTION referential_check ON (CUSTOMER_ID, TABLE (dq_tutorial_db.sch.CUSTOMER(ID))) Expectation FK_Check (value=0);

ALTER TABLE SILVER_CUSTOMER_ORDERS ADD DATA METRIC FUNCTION volume_check
    ON (CUSTOMER_ID, TABLE (dq_tutorial_db.sch.CUSTOMER_ORDERS(CUSTOMER_ID))) Expectation NoDiff (value=0);
```

# Monitoring Gold Layer
The Gold layer is the curated, consumption-ready zone where trusted datasets are delivered to dashboards, reports, and machine learning models. Monitoring this layer is critical to protect the end-user experience and ensure that business decisions are made using accurate and complete data. Quality checks at this stage catch any remaining issues before they impact stakeholders, providing confidence in the datasets that drive critical business outcomes.

A common type of check for this layer is Accepted Values. This type of check ensures data consistency, enforces business rules, and prevents invalid values from propagating downstream into analytics, reporting, or machine learning models. It validates that a column contains only a predefined set of allowed values and compares each value in the column against a reference list of valid values. This is particularly useful for categorical columns, such as product items, payment types, order statuses, or region codes. 

For example, in a payment_type column, the accepted values might be ('CREDIT_CARD', 'PAYPAL', 'WIRE_TRANSFER'). The check will flag any rows containing values outside this set, such as CHECK or BITCOIN.

```sql
ALTER TABLE GOLD_SALES_SUMMARY SET DATA_METRIC_SCHEDULE = '5 minutes';

ALTER TABLE GOLD_SALES_SUMMARY
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.ACCEPTED_VALUES
    ON (
      payment_type,
      payment_type -> payment_type IN ('CREDIT_CARD', 'PAYPAL', 'BANK_TRANSFER')
    ) Expectation Payment_Type_Check (value=0);

```

# Step 5 - Review DQ Dashboard
Once the DMF setup is completed, you can review the DMF results by accessing the Quality Tab on Snowsight. Note that the first DMF run has to be completed before results start showing up in the dashboard.

The interactive interface displays DQ metric results, trends, and incidents. 
<img src="/images/DQ_Monitor.png" width="70%">

You can drill down into failed quality checks, view impacted assets, and investigate specific records that violate a data quality check.
<img src="/images/Checks_Drilldown.png" width="70%">
<img src="/images/Checks_Drilldown1.png" width="70%">


# Step 6 - Set up notifications
Finally, you can use these results for Monitoring and leverage Snowflake Alerts to set up notifications based on the DMF results, or even build integration with 3rd party tools.

This feature is currently in Private Preview(PrPr). The setup procedures for notifications will be accessible in the LAB once the feature transitions to Public Preview(PuPr).

# Next Steps
## Grading 
Complete grading before cleaning up the objects created in this Lab

Detailed instruction on Grading instructions can be found [HERE](/config/readme.md)

