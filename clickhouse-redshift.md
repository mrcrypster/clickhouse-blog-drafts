# Migrating from RedShift to ClickHouse
### [alternatively ->] Integration ClickHouse into the running RedShift infrastructure

Amazon Redshift is a cloud data warehousing solution focused on providing analytics capabilities for structured and semi-structured data at scale. Being a part of AWS infrastructure it is a widely used and feature-powerful solution.

## Why switch to ClickHouse

There's growing number of cases engineers decide to choose ClickHouse over RedShift. Let's discover reasons behind that and ways to get benefits of ClickHouse in situations RedShift doesn't meet requirements.

### Query Performance

Increasing number of ClickHouse users shares how ClickHouse helped them increase query performance, especially the ones critical to real time. ClickHouse implement the most modern approaches to providing best performance across OLTP landscape.

### Data Ingestion Latency

Getting a lot of data into the database and being able to execute queries with the lowest possible latency is another reason many engineers choose ClickHouse.

### Cost reduction

ClickHouse is not only fast, but provides the best-in-class solution for storage cost efficiency. Utilizing unique approaches in encoding and compressing data, it demonstrates great disk efficiency (and security?).

### There's more

Reasons to switch to ClickHouse are not limited to its great performance and efficiency. Powerful aggregating capabilities, enormous set of integrations and supported libraries, bult-in ETL capabilities, transparent OLTP integrations, local and cloud as well as structured and semi-structured data supported, integrated machine learning and more.

## Quickly getting ClickHouse to fix RedShift drawbacks

First and most important. You don't have to fully migrate from RedShift to ClickHouse in a single step to get all the benefits. Wide range of options are available to integrate and migrate gradually making it days or even hours to start getting benefits of ClickHouse in the running RedShift setup.

## Copying data from Redshift to ClickHouse

RS-> export to CSV/TSV/... [S3?] -> import -> CH
CH <- RS table directly

## Migrating tables

Create table -> types matching
Load data <- from table engine or S3 file

### Migrate entire database?

Materialized PostgreSQL

## Summary and further reading

- Cloud
- Data ingesting and formats
- Performance tuning
- Data formats, integrations and client libraries
