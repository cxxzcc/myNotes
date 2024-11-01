

Mediated schema


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

