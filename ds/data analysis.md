
# 1

a) What is the relationship between views and a mediated schema?

mediated schema defines a common set of attributes that serve as an abstraction over multiple data sources.

view use to define the schema mapping between the data sources and the mediated schema.

b) A view can be defined based on other views. What is the purpose of doing this when defining a mediated schema?

define local views as wrappers over data sources.

The mediated schema can then be defined based on a set of global views over local views.

Views over views simplify the definition of global views, and also provide an abstraction layer over data source schemas.

c) Explain the concept of query unfolding and describe when it should be applied.

Query unfolding is the process of replacing and expanding a query with the definitions of the views used in that query.

This process can be repeated until the original query is fully expanded into a query that no longer refers to views.
# 2
In this course you have learned and used ETL tools, such as Pentaho Data integration (PDI).

a) ETL tools are used to build materialized or non-materialized views? Justify your answer.

ETL process consists extract transform and load

The output is materialized because it is stored in a persistent way and becomes available without recomputation.

b) Suppose you use PDI to migrate data from an input table to an output table. The input table is 20GB in size, but you only have 8 GB of RAM. Do you think it will work? Justify your answer.

work
PDI works in streaming mode

c) In PDI, there is a dialog to define a database connection. In which other tools have you seen a similar dialog? What was the purpose of defining a database connection in those other tools?

Schema Workbench
(to connect to the data warehouse tables that are needed to define the OLAP cube),

Pentaho Server (to connect to the data warehouse in order to run MDX queries)

Report Designer (to connect to a database/data warehouse in order to run SQL or MDX queries to generate a report).
# 3
Suppose you are building a transformation to detect approximate duplicate records in a database table with customers (first name, last name, e-mail, phone).

a) How can you reduce the number of comparisons that need to be done between those records?

join step with a condition like A.id < B.id.

b) If you had to choose a different string matching technique to compare each field, which techniques would you choose for each field and why?

first names-Jaro/Jaro-Winkler-compare short strings.

last name-Soundex-base on phonetics

e-mail-edit distance - align characters such as @ or dots.

phone-Jaccard, considers transpositions and useful to compare phone numbers with the same digits but in a different order.

c) After you have computed the string matching result for each field, how do you decide if two records are duplicates or not? Please explain.

use a formula with weights (e.g. sim_total = 0.3*sim1 + 0.4*sim2 + 0.3*sim3)

a threshold for that measure.
records with an overall measure above or below the threshold (above if similarity; below if distance) will duplicates.
# 4
In the same customers table as before, suppose that some customers may not have an e-mail or phone(or both), and other customers may have multiple e-mails and multiple phones.

a) How would you use a data profiling tool to discover these anomalies? Please explain.

DataCleaner 
check the number of NULL values
check the string length or word count

# 5
Consider a data warehouse that stores 3-D facts such as “customer C bought product P on date D”.

a) Suppose you have the option of defining a customer hierarchy with two levels (customer, country) or three levels (customer, city, country). What is the impact of this decision on the OLAP operations that you will be able to perform on the data warehouse? Justify your answer.

with city enables OLAP operations like roll-up from customer to city, drill-down from country to city, and slicing or dicing by city.

Without it, these operations are limited to customer and country levels.

b) If the customer dimension has four levels (customer, city, state, country) and the state level can be skipped, what do you call this kind of hierarchy? Also, how would you implement (in the data warehouse schema) this possibility of skipping the state?

ragged hierarchy.

allow that attribute to be NULL (star structure) or
a foreign key between city table and country table (snow structure)

c) If the customer dimension has three levels (customer, city, country), what could make you decide between having a star structure or having a snowflake structure for this dimension?

based on the presence of additional attributes for those levels.

# 6
In the topic of data warehousing, you were introduced to the concept of surrogate keys.

a) Why use a surrogate key instead of the same primary key as in the original database?

primary keys of data sources may be inconsistent or change over time.
have all keys in the data warehouse as integers for faster processing

b) Why do slowly-changing dimensions always require a surrogate key?

In a slowly-changing dimension there may be multiple versions of the same record.

c) When you use a surrogate key instead of the natural key, what do you have to change in the transformation that populates the fact table?

include a database lookup.

That surrogate key can be found in a dimension table
# 7
Once the data warehouse has been created, we need to analyze the data contained therein.

a) What is the purpose of a tool called Pentaho Schema Workbench (PSW)? What is the relationship between PSW and an OLAP tool such as Saiku Analytics?

PSW allow the user to define an OLAP cube
PSW export an XML file that can be imported into an OLAP tool
This OLAP tool will allow querying the cube with MDX.

b) If you use only ROWS and COLUMNS in an MDX query, how can you analyze data in three or more dimensions?

those additional dimensions must be nested either in ROWS or COLUMNS.
CROSSJOIN

c) What is the purpose of the WHERE clause in an MDX query? Give two different examples.

slicing or dicing by selecting.
For example: WHERE Customer.Country.Italy.

specify the measures that should be included in the result.
For example: WHERE Measures.Sales.

d) If a reporting tool can get data either from SQL or from MDX, why not querying the database directly with SQL, instead of querying the data warehouse with MDX?

gather analytical data.
data warehouse using MDX, because it facilitates the analysis of multi-dimensional data
data warehouse contains pre-computed measures




# VIEWS
```sql
CREATE VIEW myview AS SELECT …
DROP VIEW myview

```
Views map data from tables the physical model toa new logical model
Views support logical independence from the physical model



# String matching
* String matching is based on measures
	measures of distance - lower is better, higher is worse
	measures of similarity - higher is better, lower is worse

* sequence-based
	* Edit distance (Levenshtein)
	* Damerau-Levenshtein distance
	* Needleman-Wunsch measure
	* Jaro measure
	* Jaro-Winkler measure
* set-based
	* Jaccard measure
* phonetic
	* Soundex
		Refined Soundex



OLTP
* Focus on the operation of a system
* Large number of short transactions
* Very fast query response
* Performance metric is ‘TransacKons per second’
OLAP
* Focus on analysis of historical data
* Low number of long data load transactions
* Slow response to very complex queries involve aggregations
* Performance metric is ‘Query response time’


Surrogate keys
* Provide independence from keys in the original data sources
* Solve inconsistencies between keys from multiple sources
* Keys as integers to improve efficiency





