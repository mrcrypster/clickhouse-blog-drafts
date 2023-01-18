# Importing and exporting data in any format in Clickhouse

ClickHouse supports most of the known text and binary data formats. This allows easy integration into almost any working data pipeline and leveraging on ClickHouse benefits.

## Standard text formats

CSV is one of the most popular formats to store data due to its simplicity. Importing and exporting CSV data is easy with the [CSV](https://clickhouse.com/docs/en/interfaces/formats/#csv) format:

```bash
clickhouse-client -q "INSERT INTO some_table FORMAT CSV" < data.csv
```

In many cases, CSV files are broken, poorly encoded, and have custom delimiters or even line separators. ClickHouse provides ways to handle any of that cases.

To process CSVs with custom delimiters (`;` in our example), we have to set the following option:

```sql
SET format_csv_delimiter = ';';
```

### Importing data from broken or custom CSV files

In cases CSV is encoded in a non-standard way or just invalid, we can use CustomSeparated format to customize escaping rules and delimiters:

```sql
SET format_custom_field_delimiter = '|';
SET format_custom_row_between_delimiter = ';';
SET format_custom_escaping_rule = 'JSON';
```

We've used JSON [escaping rule](https://clickhouse.com/docs/en/operations/settings/settings/#format_custom_escaping_rule), custom value delimiter (`|`), and line separator (`;`) here. After the settings are changed, we can continue with import:

```sql
INSERT INTO some_table FROM INFILE 'custom.csv'
FORMAT CustomSeparated
```

Another popular text data format is TSV, which is also supported by ClickHouse with the [TabSeparated](https://clickhouse.com/docs/en/interfaces/formats/#tabseparated) format:


```sql
clickhouse-client -q "INSERT INTO some_table FORMAT TabSeparated" < data.tsv
```

Explore more capabilities when working with CSV formats family in ClickHouse, including rows skipping, controlling Null values, automatic compression, and more.

### JSON data

ClickHouse can work with almost any JSON data, be that an array of values, an object of objects, or separate JSON objects.

For example, a logging app can write logs as a JSON object per line. This case can be addressed using [JSONEachRow](https://clickhouse.com/docs/en/interfaces/formats/#jsoneachrow) format in ClickHouse:

```bash
clickhouse-client - q "INSERT INTO sometable FORMAT JSONEachRow" < access.log
```

Explore how JSON data of different forms can be loaded to ClickHouse as well as exported. Worth mentioning that ClickHouse also supports the BSON format used by MongoDB.

## Regular expressions for custom text formats

Besides standard formats, like CSV or JSON, ClickHouse also supports data import based on regular expressions. In this case, the `Regexp` format should be used together with the `format_regexp` option containing regular expression with capture groups (treated as table columns):

```sql
INSERT INTO some_log FROM INFILE 'custom.txt'
SETTINGS
  format_regexp = '([0-9]+?) \[(.+?)\] \- "(.+)"'
FORMAT Regexp
```

This query can be used to load the following example file:

```
121201 [notice] - "Started service"
121202 [error] - "Configuration file not found"
122203 [warning] - "Creating default configuration file"
```

Read more on how to use Regexp format for data ingesting. 

Another option to process custom text formats is to use a Template format. But the Template format is even more powerful in terms of exporting data because it allows rendering query results into high-level formats, like HTML.

## Native and binary formats

ClickHouse has it's own native format that can be used to import and export data. It's more efficient than text formats regarding processing speed and space usage. Native format is useful for transferring data between ClickHouse servers when they don't have a direct connection with each other. For example, to transfer data from a ClickHouse server to ClickHouse Cloud:

```bash
clickhouse-client -q "SELECT * FROM some_table FORMAT Native" | \
clickhouse-client --host some.aws.clickhouse.cloud --secure \
--port 9440 --password 12345 \
-q "INSERT INTO some_table FORMAT Native"
```

Binary formats are usually more efficient and safe than text formats but are limited in support. ClickHouse has RowBinary format for general binary cases, and RawBLOB is used with, but not limited to, files. Additionally, ClickHouse supports popular serialization formats like Protocol Buffers, Cap’n Proto and Message Pack.

## Parquet and other Apache formats
Apache has multiple data storage and serialization formats that are popular in Hadoop environments. ClickHouse can work with all of them, including Parquet.

We can import data from a Parquet file:

```sql
clickhouse-client -q "INSERT INTO some_table FORMAT Parquet" < data.parquet
```

By using a [`file()`](https://clickhouse.com/docs/en/sql-reference/table-functions/file/) function, we can explore data before actually loading it into a table:

```sql
SELECT *
FROM file('data.parquet', Parquet)
LIMIT 3;

┌─path──────────────────────┬─date───────┬─hits─┐
│ Akiba_Hebrew_Academy      │ 2017-08-01 │  241 │
│ Aegithina_tiphia          │ 2018-02-01 │   34 │
│ 1971-72_Utah_Stars_season │ 2016-10-01 │    1 │
└───────────────────────────┴────────────┴──────┘
```

We can also export data to a Parquet file:

```sql
clickhouse-client -q "SELECT * FROM some_table FORMAT Parquet" > file.parquet
```

Find out more about other supported Apache formats, such as Avro, Arrow, and ORC.


## SQL dumps
Though SQL dumps are inefficient in storing and transferring data, ClickHouse supports loading data from MySQL dumps and creating SQL dumps for Mysql, PostgreSQL, and other databases.

To create an SQL dump, the `SQLInsert` format should be used:

```sql
SET output_format_sql_insert_table_name = 'some_data';
SET output_format_sql_insert_include_column_names = 0;
SELECT * FROM some_table
INTO OUTFILE 'dump.sql'
FORMAT SQLInsert;
```

This will create the `dump.sql` file with an SQL values dump in it. It will use `some_data` as a table name and skip columns declaration. It can then be fed to other DBMS:

```bash
psql < dump.sql 
```

ClickHouse also supports importing data from MySQL dumps using `MySQLDump` format:

```sql
cat mysql-dump.sql | clickhouse-client -q "INSERT INTO some_data FORMAT MySQLDump"
```


## Null format for performance testing
There's a special [Null](https://clickhouse.com/docs/en/interfaces/formats/#null) data format that will not print anything but wait for the query to execute:

```sql
SELECT *
FROM bit_table
LIMIT 100000
FORMAT `Null`

0 rows in set. Elapsed: 1.112 sec. Processed 131.07 thousand rows, 167.57 MB (117.86 thousand rows/s., 150.68 MB/s.)
```

ClickHouse server still returns data to the client, but it's not printed. This makes the `Null` format useful for testing query performance which returns too much data to fit into the terminal.

## Prettifying command line
By default, ClickHouse uses [PrettyCompact](https://clickhouse.com/docs/en/interfaces/formats/#prettycompact) format for the command line client. It outputs data in blocks sometimes (as soon as results are returned):

```
┌─id──────────────────────────┬─gender─┬─birth_year─┐
│ B7mkoLYIZEdoPkfCRTKtYg_0000 │ female │ 1951       │
└─────────────────────────────┴────────┴────────────┘
┌─id──────────────────────────┬─gender─┬─birth_year─┐
│ nzHXUMnmjspwV4JxL-KqzQ_0000 │ female │ 1956       │
└─────────────────────────────┴────────┴────────────┘
┌─id──────────────────────────┬─gender─┬─birth_year─┐
│ 5cs05UbDttZBFBE6tPpjUg_0000 │ male   │ 1989       │
│ 5cs05UbDttZBFBE6tPpjUg_0000 │ male   │ 1989       │
└─────────────────────────────┴────────┴────────────┘
...
```

We can use [PrettyCompactMonoBlock](https://clickhouse.com/docs/en/interfaces/formats/#prettycompactmonoblock) to ask ClickHouse to output results as a single table:

```sql
SELECT * FROM some_table
FORMAT PrettyCompactMonoBlock;

┌─id──────────────────────────┬─gender─┬─birth_year─┐
│ B7mkoLYIZEdoPkfCRTKtYg_0000 │ female │ 1951       │
│ nzHXUMnmjspwV4JxL-KqzQ_0000 │ female │ 1956       │
│ 5cs05UbDttZBFBE6tPpjUg_0000 │ male   │ 1989       │
│ 5cs05UbDttZBFBE6tPpjUg_0000 │ male   │ 1989       │
│ mcXoSkxAk-1xGpSiqnCB1Q_0000 │ female │ 1961       │
└─────────────────────────────┴────────┴────────────┘
```

Less compact but easier to perceive is the [Pretty](https://clickhouse.com/docs/en/interfaces/formats/#pretty) format (also [PrettyMonoBlock](https://clickhouse.com/docs/en/interfaces/formats/#prettymonoblock) is available for single table output):

```sql
SELECT FROM some_table
FORMAT PrettyMonoBlock;

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ path                           ┃ hits ┃ bar(hits, 0, 1500, 25)  ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Bangor_City_Forest             │   34 │ ▌                       │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Alireza_Afzal                  │   24 │ ▍                       │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Akhaura-Laksam-Chittagong_Line │   30 │ ▌                       │
├────────────────────────────────┼──────┼─────────────────────────┤
│ 1973_National_500              │   80 │ █▎                      │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Attachment                     │ 1356 │ ██████████████████████▌ │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Kellett_Strait                 │    5 │                         │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Ajarani_River                  │   30 │ ▌                       │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Akbarabad,_Khomeyn             │    8 │ ▏                       │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Adriaan_Theodoor_Peperzak      │   88 │ █▍                      │
├────────────────────────────────┼──────┼─────────────────────────┤
│ Alucita_dryogramma             │    1 │                         │
└────────────────────────────────┴──────┴─────────────────────────┘

```

Finally, we can ask ClickHouse to get rid of the table grid with the [PrettySpace](https://clickhouse.com/docs/en/interfaces/formats/#prettyspace) format (and [PrettySpaceMonoBlock](https://clickhouse.com/docs/en/interfaces/formats/#prettyspacemonoblock)):

```sql
SELECT FROM some_table
FORMAT PrettySpace;

 path                             hits   bar(hits, 0, 1500, 25) 

 Bangor_City_Forest                 34   ▌                       
 Alireza_Afzal                      24   ▍                       
 Akhaura-Laksam-Chittagong_Line     30   ▌                       
 1973_National_500                  80   █▎                      
 Attachment                       1356   ██████████████████████▌ 
 Kellett_Strait                      5                           
 Ajarani_River                      30   ▌                       
 Akbarabad,_Khomeyn                  8   ▏                       
 Adriaan_Theodoor_Peperzak          88   █▍                      
 Alucita_dryogramma                  1       
```

## Summary

ClickHouse provides tools for all imaginable formats - standard text, binary or custom ones. Explore in-depth data formats in the official documentation. Consider using [clickhouse-local](https://clickhouse.com/blog/extracting-converting-querying-local-files-with-sql-clickhouse-local), which is a powerful portable tool to query, convert and transform data from local files.