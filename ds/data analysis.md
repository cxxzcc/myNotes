

Mediated schema

# 1
In this course you have learned how to create SQL views over a database.

a) What is the relationship between views and a mediated schema?

A mediated schema defines a common set of attributes that serve as an abstraction over multiple data sources.

SQL views are used to define the schema mapping between the data sources and the mediated schema.

b) A view can be defined based on other views. What is the purpose of doing this when defining a mediated schema?

It is possible to define local views as wrappers over data sources.

The mediated schema can then be defined based on a set of global views over local views.

Views over views simplify the definition of global views, and also provide an abstraction layer over data source schemas.

c) Explain the concept of query unfolding and describe when it should be applied.

Query unfolding is the process of replacing and expanding a query with the definitions of the views used in that query.

This process can be repeated until the original query is fully expanded into a query that no longer refers to views.
# 2
In this course you have learned and used ETL tools, such as Pentaho Data integration (PDI).

a) ETL tools are used to build materialized or non-materialized views? Justify your answer.

An ETL process consists in extracting data from data sources, transforming the data, and loading (i.e. storing) the output somewhere, for example in a file, in database table, or in a data warehouse.

The output is materialized because it is stored in a persistent way and becomes available without recomputation.

b) Suppose you use PDI to migrate data from an input table to an output table. The input table is 20

GB in size, but you only have 8 GB of RAM. Do you think it will work? Justify your answer.

PDI works in streaming mode, meaning that some records are being processed and written to the output at the same time as more records are being read from the input.

So, in principle, it should not be necessary to have all records in memory at any point in time. It should work.

c) In PDI, there is a dialog to define a database connection. In which other tools have you seen a similar dialog? What was the purpose of defining a database connection in those other tools?

The same dialog was present in Schema Workbench

(to connect to the data warehouse tables that are needed to define the OLAP cube),

in Pentaho Server (to connect to the data warehouse in order to run MDX queries), and

Report Designer (to connect to a database or to a data warehouse in order to run SQL or MDX queries to generate a report).
