# Postgres Apptainer Recipe

## About

This recipe instances the Postgres 18.3 Docker image as an Apptainer
application. These instructions are primarily for ALCF Polaris users but can be
modified to fit your HPC environment as needed.

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
```

2. Create directories

```bash
mkdir -p $APPTAINER_TMPDIR
mkdir -p $APPTAINER_CACHEDIR
mkdir -p pgdata pgrun   # Used for Postgres data
```

3. Create the environment file (provided in `pg.env`)

```bash
cat > pg.env <<EOF
POSTGRES_USER=pguser
POSTGRES_PASSWORD=mypguser123
POSTGRES_DB=mydb
POSTGRES_INITDB_ARGS="--encoding=UTF-8"
EOF
```

4. Build the Postgres image

```bash
apptainer build postgres_18-3.apptainer docker://postgres:18.3
```

5. Run the Postgres Apptainer instance

```bash
apptainer instance run \
    --env-file pg.env \
    -B pgdata:/var/lib/postgresql \
    -B pgrun:/var/run/postgresql \
    postgres_18-3.apptainer \
    postgres
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
