---
layout: post
title: "Using SQL PIVOT and UNPIVOT"
date: 2012-11-03 23:22
comments: true
categories:
 - Database
 - SQL
---

I learned about a pair of SQL keywords I didn't know existed before. I suppose I shouldn't have been so surprised
this ability existed in SQL since I knew it existed within Excel spreadsheets, but it really saved the day this
last week. I'm speaking of the `PIVOT` and `UNPIVOT` SQL keywords.

This last week I needed to track down some data inconsistencies in our database by joining a few view model tables.
These tables are populated by event handlers that denormalize the relevant data from events into a view model
table that serves the needs of a specific part of the UI. In one particular view model table, it made sense to
arrange the data points specific to each US state as columns on a single row. This allowed for a grid-based UI that
enables batch operations on many rows at one time. A second view model table was intended for a different use case
and was designed so that the state-specific data points became a one-to-many relationship across multiple rows and
only two columns for the key-value pair of the US state and its associated data point value. These were two of the
tables I needed to join together during my database inquisition.

At this point I figured I was stuck, as I couldn't see a sensible way to join two tables designed so contrary to
each other for the purposes of joining. I tried to think of any ways to dynamically join tables on a column name
in one table specified by the value in the other table's row. I knew I was heading nowhere fast, so I asked our
DBA for some ideas. He informed me of the `PIVOT` and `UNPIVOT` keywords in SQL as one option to solve the problem.
We reasoned that even though it may not be the most efficient feature, the likelihood of it proving to be a
performance bottleneck would be essentially non-existent (given that the entire size of the data set was relatively
small, on the magnitude of hundreds or thousands of rows instead of millions of rows). And so I gave it a go.

I will now describe an example scenario in order to illustrate how I used the `UNPIVOT` feature to join two tables
that didn't seem at all compatible. The goal will be to join a table representing what US states each product is
available for sale within to a table representing how many of each product are being sold in each state. We will
call the first table `ProductAvailability` and give it an integer identity column named `Id`, an integer column
named `ProductId`, and then 52 `bit` columns indicating the availability status of the product in each of the 50 US
states, the District of Columbia, and Puerto Rico. These columns will be named by their two letter abbreviations of
`AK` through `WY`. The second table we will call `ProductSales` and it will also have an integer identity column
named `Id` and an integer column named `ProductId`. In addition it will have a string column (specifically
`char(2)`) named `StateCode` and an integer column named `TotalSales`.

The unpivoting of the `ProductAvailability` table to prepare it for joining can be done like so:
<pre class="brush: sql">
    SELECT
      Unpivoted.ProductId,
      Unpivoted.StateCode,
      Unpivoted.IsAvailable
    FROM ProductAvailability
    UNPIVOT
    (
      IsAvailable FOR StateCode
      IN (AK,AL,AR,AZ,CA,CO,CT,
          DC,DE,FL,GA,HI,IA,ID,
          IL,IN,KS,KY,LA,MA,MD,
          ME,MI,MN,MO,MS,MT,NC,
          ND,NE,NH,NJ,NM,NV,NY,
          OH,OK,OR,PA,PR,RI,SC,
          SD,TN,TX,UT,VA,VT,WA,
          WI,WV,WY)
    ) AS Unpivoted
</pre>

The interesting part is obviously the `UNPIVOT` clause. Here we are declaring a list of column names to unpivot and
also naming two new columns in which to place the old column's name and value (`StateCode` and `IsAvailable`
respectively). The result is that instead of having one row of 54 columns for each product, we now have 52 rows of
three columns to describe the exact same availability of the product. But with this second schema, we can more
easily join with the `ProductSales` table.

Here is an example of such a query where we are trying to find the products which have no sales in a state for
which it is marked as available for sale:
<pre class="brush: sql">
    SELECT
      ProductStateAvailability.ProductId,
      ProductStateAvailability.StateCode
    FROM
    (
      SELECT
        Unpivoted.ProductId,
        Unpivoted.StateCode,
        Unpivoted.IsAvailable
      FROM ProductAvailability
      UNPIVOT
      (
        IsAvailable FOR StateCode
        IN (AK,AL,AR,AZ,CA,CO,CT,
            DC,DE,FL,GA,HI,IA,ID,
            IL,IN,KS,KY,LA,MA,MD,
            ME,MI,MN,MO,MS,MT,NC,
            ND,NE,NH,NJ,NM,NV,NY,
            OH,OK,OR,PA,PR,RI,SC,
            SD,TN,TX,UT,VA,VT,WA,
            WI,WV,WY)
      ) AS Unpivoted
      WHERE Unpivoted.IsAvailable = 1
    ) AS ProductStateAvailability
    INNER JOIN ProductSales
      ON ProductSales.StateCode = ProductStateAvailability.StateCode
      AND ProductSales.ProductId = ProductStateAvailability.ProductId
    WHERE ProductSales.TotalSales = 0
</pre>

I hope you find the `PIVOT` and `UNPIVOT` SQL keywords just as helpful as I did if you ever find yourself in a bind
with two tables that are seemingly impossible to join.
