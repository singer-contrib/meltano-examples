# Load data into PostgreSQL, pgvector, and CrateDB, using Meltano

A Meltano project exercising a few data loading tasks, defined
within `meltano.yml`.


## Prerequisites

Run a PostgreSQL server with pgvector extension.
```shell
docker run --rm -it --publish=5432:5432 \
  --env "POSTGRES_HOST_AUTH_METHOD=trust" \
  --volume "$(pwd)/init.sql:/docker-entrypoint-initdb.d/init.sql" \
  ankane/pgvector postgres -c log_statement=all
```

In another shell, before running any `psql` command, define the authentication
credentials.
```shell
export PGHOST=localhost
export PGUSER=postgres
```


## Setup

Install project dependencies.
```shell
meltano install
```

Dry-run data source elements (taps), to check whether they work well.
```shell
meltano invoke tap-smoke-test
meltano invoke load-jsonl-array-jsonb
meltano invoke load-jsonl-array-vector
```


## Data Loading

### PostgreSQL
Load data into the `load_jsonl_array_jsonb.array_float` table, converging
the array into a `jsonb[]` column.
```shell
meltano run load-jsonl-array-jsonb target-postgres --full-refresh
psql postgresql://postgres@localhost/ --command '\d load_jsonl_array_jsonb.array_float'
psql postgresql://postgres@localhost/ --command 'SELECT * from load_jsonl_array_jsonb.array_float;'
```

```shell
meltano run tap-smoke-test target-postgres
psql postgresql://postgres@localhost/ --command '\dt tap_smoke_test.*'
```

### PostgreSQL/pgvector
Load data into the `load_jsonl_array_vector.array_float` table, converging
the array into a `vector` column.
```shell
meltano run load-jsonl-array-vector target-postgres --full-refresh
psql postgresql://postgres@localhost/ --command '\d load_jsonl_array_vector.array_float'
psql postgresql://postgres@localhost/ --command 'SELECT * from load_jsonl_array_vector.array_float;'
```


## Troubleshooting

### UndefinedObject: type "vector" does not exist

This error indicates that the pgvector extension has not been installed.
```shell
psycopg2.errors.UndefinedObject: type "vector" does not exist
```

This rig should take care of it by running the corresponding SQL command
on behalf of the `init.sql` file. If you are running your PostgreSQL/pgvector
instance in a different environment, you may need to execute this statement
once.
```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### Increase log level
Meltano is pretty sparse on output. This significantly changes when increasing
the log level. So, to see more output, optionally use:
```shell
meltano --log-level=debug {invoke,run}
```
