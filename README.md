Dump data from Elasticsearch or Opensearch to parquet, json, or csv files, or directly to stdout.
Files are named the same as the index, with a partition number added in case of large datasets, and an 
appropriate extension.

There are two modes of operation, depending on the output requested:

## parquet, ndjson, or csv 

A columnar dataframe is built in memory using [Polars](https://docs.pola.rs/), 
then written out to parquet with zstd compression, ndjson, or csv (compression not yet supported) as appropriate. For large
datasets (controlled with the `--max-partition-size-mb` flag) multiple files will be output, 
with an incremental partition number appended to the index name.

Nested fields are represented as Structs, unles `--flatten` is provided, in which case fields are flattened into the top-level by combining field names with underscores. Flattening is recommended when working with multiple indices that have dynamic mapping, as columns can then be merged across files - different structs cannot easily be merged. Flattening is also required to output to CSV.

## stdout or jsonl

Records are dumped in JSON, one record per line, to stdout or a file as they are received, with up to `--max-partition-rows` (default 1 000 000) records per file.

# Requirements

Developed with Python 3.12 with:

- opensearch-py==2.8.0
- polars==1.36.0
- requests==2.32.5

## Nix (recommended)

A `flake.nix` is included - run `nix develop` to enter a shell with all dependendencies. 
With `direnv` installed run `direnv allow` to have it load the environment for you when you enter the directory.

## Pip

    pip install -r requirements.txt

# Usage

```
usage: dump-es-parquet [-h] [--es ES] [--cert CERT] [--key KEY] [--no-verify-certs] [--capath CAPATH]
                       [--size SIZE] [--sort SORT] [--timeout TIMEOUT]
                       [--output {parquet,ndjson,csv,jsonl,stdout}] [--flatten] [--query QUERY]
                       [--fields FIELDS] [--max-partition-rows MAX_PARTITION_ROWS]
                       [--max-partition-mb MAX_PARTITION_MB] [--no-partition] [--debug] [--quiet]
                       index

Dump documents from Elasticsearch or OpenSearch to stdout or files.

Behavior varies with output format:

    parquet: builds a polars dataframe in-memory, accumulating records until the dataframe reaches the
             specified max partition size, at which point it is written to a parquet file with the index name
             and partition number, which is omitted if the entire results fit into a single partition.
    ndjson:  same as parquet, but written to newline-delimited json files instead.
    csv:     same as parquet, but written to csv files instead.
    stdout:  outputs raw records in JSON format to stdout. Does not attempt to build a dataframe,
             so will work even if the source data has problematic/inconsistent types.
    jsonl:   same as stdout, but outputs records to files, one per request batch.

positional arguments:
  index                 source index pattern

optional arguments:
  -h, --help            show this help message and exit
  --es ES               source cluster address
  --cert CERT           Client x509 certificate
  --key KEY             Client x509 key
  --no-verify-certs     Do not verify x509 certificates
  --capath CAPATH       Path to CA certificates
  --size SIZE           Record batch size (default 500)
  --sort SORT           Comma-separated list of field:direction pairs
  --timeout TIMEOUT     Elasticsearch read timeout in seconds (default 60)
  --output {parquet,ndjson,csv,jsonl,stdout}
                        output format
  --flatten             Flatten nested data into top level, otherwise use structs
  --query QUERY         Query string to filter results
  --fields FIELDS       Comma-separated list of fields to include in the output. Wildcards are supported.
                        Defaults to all fields.
  --max-partition-rows MAX_PARTITION_ROWS
                        Maximum rows in partition
  --max-partition-mb MAX_PARTITION_MB
                        Maximum in-memory size of partition dataframe in megabytes (default 1000). Note that
                        the file size will be smaller due to compression
  --no-partition        Do not partition into files no matter how big the dataset is
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
