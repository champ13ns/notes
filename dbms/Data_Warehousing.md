
                    DATA WAREHOUSE
                          |
       -----------------------------------------
       |               |             |         |
    Subject         Integrated      Time       Non-
    Oriented                       Variant    Volatile
       |               |             |         |
 Business          Multiple       Historical  Stable/
 subjects          sources          data      read-heavy


Data warehousing -> A large centeral repository containing historical data collected from different sources, mainly used for:
                    Analysis, Reporting, Desicion making, Buisness intelligence.
                    Father -> Bill Inmon.

                    Data warehouse is defined using these characteristics
                    a. Subject-Oriented. -> A data warehouse is organized around major subjects of an organization.

                    b. Integrated. -> Data collected from differenct sources is converted into a consitent format.

                    c. Time-Variant -> Warehouse must contain a historical time dimension.

                    d. Non-Volatile -> A Data Warehouse is called non-volatile because data stored there is primarily used for reading and analysis rather than continuous transactional modification.
                    In a normal banking database:

                        INSERT
                        UPDATE
                        DELETE
                        INSERT
                        UPDATE
                        DELETE

                        happen continuously.

                        In a warehouse, conceptually:

                        Load historical data
                                ↓
                        Keep it
                                ↓
                        Analyse it
                        
                        Do not interpret non-volatile as: Data can NEVER change That is too absolute.
            Operational DB → frequent modifications
            Data Warehouse → primarily historical/read-oriented


Data reaches the warehouse through.

1. ETL -> Extract, Transfrom, Load.
Operational Systems
       |
       |---- ATM DB
       |---- UPI DB
       |---- Loan DB
       |---- Credit Card DB
       |---- Branch DB
       |
       ↓
    EXTRACT
       ↓
   TRANSFORM
       ↓
      LOAD
       ↓
+-------------------+
|   DATA WAREHOUSE  |
+-------------------+
       |
       ↓
 Analysis / Reports
       |
       ↓
Management Decisions

OLTP vs OLAP — Master Table
Feature	                    OLTP	OLAP
Full form	                Online Transaction Processing	Online Analytical Processing
Purpose	                    Daily operations	            Analysis/reporting
Data	                    Current, detailed	            Historical, often summarized
Operations	                INSERT/UPDATE/DELETE	        Mostly complex SELECT/read
Queries	                    Short/simple	                Complex analytical
Users	                    Customers, clerks, applications	    Analysts, managers
Example	                        ATM withdrawal	                5-year ATM trend
Design tendency	                Normalized	O                   ften denormalized
Main concern	                Fast transactions	            Fast analysis



What database is normally used for a Data Warehouse?

Traditionally, Data Warehouses are largely based on the relational model.
So yes, you will commonly see:
Tables
Rows
Columns
SQL

But they are specially optimized for analytics.

Modern examples include:

Snowflake
Google BigQuery
Amazon Redshift
Azure Synapse
Teradata

These aren't just ordinary application databases like a typical MySQL instance. They are designed to efficiently scan and aggregate huge amounts of data.

For example:

SELECT state, SUM(total_sales)
FROM sales
WHERE year BETWEEN 2021 AND 2026
GROUP BY state;

A warehouse may need to scan billions of records for a query like this.


Data Mining -> 

DATA MINING
    |
    |--- Classification -> In classificatin categories are predefined, classification predicts a category.  (fraud/not fraud)
    |
    |--- Clustering -> In clustering, categroes are not predefined, the system/algorithm itself discovers categories/groups.
    |
    |--- Association -> Simple associating mutliple data items from there history. Ex bread , butter/
    |
    └--- Regression -> Instead of predicting a category, we predict a numerical value.

Will the customer default -> Yes/No (Classification)
Customer next month expense -> 20000 (Regression)



             DATA MINING
                  |
     -----------------------------
     |          |        |       |
Classification Cluster Association Regression
     |          |        |       |
Category      Groups   Together  Number
     |          |        |       |
Fraud/      Customer Bread →   Predict
Genuine     segments  Butter    price


Classification = predefined categories

Clustering = similar groups, no predefined labels

Association = items/events occurring together

Regression = numerical prediction


