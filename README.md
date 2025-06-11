Dump data from Elasticsearch or Opensearch to parquet files, one file per index. 

A columnar dataframe is built in memory using [Polars](https://docs.pola.rs/), then written out to parquet with zstd compression.

Nested fields are represented as Structs, unles `--flatten` is provided, in which case fields are flattened into the top-level by combining field names with underscores. Flattening is recommended when working with multiple indices that have dynamic mapping, as columns can then be merged across files - different structs cannot easily be merged.

# Requirements

Developed with Python 3.12 with:

- opensearch-py==2.8.0
- polars==1.21.0
- requests==2.32.3

## Nix (recommended)

A `flake.nix` is included - run `nix develop` to enter a shell with all dependendencies. 
With `direnv` installed run `direnv allow` to have it load the environment for you when you enter the directory.

## Pip

    pip install -r requirements.txt

# Usage

```
usage: dump-es-parquet [-h] [--es ES] [--size SIZE] [--timeout TIMEOUT] [--flatten] [--query QUERY] [--max-partition-size-mb MAX_PARTITION_SIZE_MB] [--debug] [--quiet] index

Dump documents to parquet files

positional arguments:
  index                 source index pattern

options:
  -h, --help            show this help message and exit
  --es ES               source cluster address
  --size SIZE           Record batch size (default 500)
  --timeout TIMEOUT     Elasticsearch read timeout in seconds (default 60)
  --flatten             Flatten nested data into top level, otherwise use structs
  --query QUERY         Query string to filter results
  --max-partition-size-mb MAX_PARTITION_SIZE_MB
                        Maximum in-memory size of partition dataframe in megabytes (default 1000). Note that the file size will be smaller due to compression
  --debug               Enable debug logging
  --quiet               Disable most logging (ignored if --debug specified)
```

# Examples

This will read all records from the `my-data` index, in batches of 500, and write them to a parquet file named `my-data.parquet`:

    dump-es-parquet --es https://example.com:9200 my-data

You can also dump all indices matching a pattern; each index will get its own file:

    dump-es-parquet --es https://example.com:9200 'my-data-*'

If you then want to analyze the data in DuckDB, for instance:

```sql
CREATE TABLE mydata AS SELECT * FROM read_parquet('my-data-*.parquet', union_by_name=true);
```
