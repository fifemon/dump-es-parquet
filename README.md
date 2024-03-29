Dump data from Elasticsearch or Opensearch to parquet files. 

## Requirements

Developed with Python 3.11 with:

- opensearch-py==2.4.2
- polars==0.20.15
- requests==2.31.0

### Nix (recommended)

A `flake.nix` is included - run `nix develop` to enter a shell with all dependendencies. 
With `direnv` installed run `direnv allow` to have it load the environment for you when you enter the directory.

### Pip

    pip install -r requirements.txt

## Usage

This will read all records from the `my-index` index, in batches of 500, and write them to a parquet file named `my-index.parquet`:

    dump-es-parquet --es https://example.com:9200 my-index

