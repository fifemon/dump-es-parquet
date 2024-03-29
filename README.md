Tools for working with Landscape data offline.

# Requirements

Developed with Python 3.11 with:

- opensearch-py==2.4.2
- polars==0.20.15
- requests==2.31.0

## Nix (recommended)

A `flake.nix` is included - run `nix develop` to enter a shell with all dependendencies. 
With `direnv` installed run `direnv allow` to have it load the environment for you when you enter the directory.

## Pip

    pip install -r requirements.txt

# dump-es-parquet

Dump data from Elasticsearch or Opensearch to parquet files, one file per index. 

Nested fields are represented as Structs, unles `--flatten` is provided, in which case fields are flattened into the top-level by separating field names with underscores. Flattening is recommended when working with multiple indices that have dynamic mapping, as columns can then be merged across files - different structs cannot easily be merged.

## Usage

This will read all records from the `my-data` index, in batches of 500, and write them to a parquet file named `my-data.parquet`:

    dump-es-parquet --es https://example.com:9200 my-data

You can also dump all indices matching a pattern; each index will get its own file:

    dump-es-parquet --es https://example.com:9200 'my-data-*'