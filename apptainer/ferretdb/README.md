# Postgres Apptainer Recipe

## About

This recipe instances FerretDB Postgres + Document DB
17-0.107.0 Docker image. You will need to download and add the FerretDB binary
to your shell `$PATH` from
[this url](https://github.com/FerretDB/FerretDB/releases/tag/v2.7.0). These
instructions are primarily for ALCF Polaris users but can be modified to fit
your HPC environment as needed.

## Instructions

1. Set environment variables

```bash
# Apptainer locations
export BASE_SCRATCH_DIR=/local/scratch/ # Polaris specific directory
export APPTAINER_TMPDIR=$BASE_SCRATCH_DIR/apptainer-tmpdir
export APPTAINER_CACHEDIR=$BASE_SCRATCH_DIR/apptainer-cachedir

# Sets Apptainer to work in the root directory of the `container`
export APPTAINER_PWD=/

# ALCF Polaris HTTP proxies for downloading files
export HTTP_PROXY=http://proxy.alcf.anl.gov:3128
export HTTPS_PROXY=http://proxy.alcf.anl.gov:3128
export http_proxy=http://proxy.alcf.anl.gov:3128
export https_proxy=http://proxy.alcf.anl.gov:3128

# FerretDB Variable
export FERRETDB_POSTGRESQL_URL=postgres://username:password@localhost:5432/postgres
```

2. Create directories

```bash
mkdir -p $APPTAINER_TMPDIR
mkdir -p $APPTAINER_CACHEDIR
mkdir -p ferretdata ferretrun   # Used for Postgres data
```

3. Create the Postgres environment file (provided in `pg.env`)

```bash
cat > pg.env <<EOF
POSTGRES_USER=username
POSTGRES_PASSWORD=password
POSTGRES_DB=postgres
EOF
```

4. Build the Postgres image

```bash
apptainer build postgres_17.apptainer docker://ghcr.io/ferretdb/postgres-documentdb:17-0.107.0-ferretdb-2.7.0
```

5. Run the Postgres Apptainer instance

```bash
apptainer instance run \
    --env-file pg.env \
    -B ferretdata:/var/lib/postgresql/data \
    -B ferretrun:/var/run/postgresql \
    postgres_17.apptainer \
    ferretdb_postgres
```

6. View the currently running Apptainer instances

```bash
apptainer instance list
```

7. Clean up the environment

```bash
unset http_proxy
unset https_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
```

8. Test the connection via `psql`

```bash
apptainer exec instance://postgres psql -U pguser -d mydb
```
