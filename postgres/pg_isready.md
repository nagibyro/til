# pg_isready
A common task when automating acceptance test pipelines is the need to spin up
test databases and services (usually in docker). But there is a difference between
when the container starts, or the db process starts, and the server being ready for use.

For Postgres I usually had a script like
```bash
function wait_for_db {
    local retries=5

    local service_up=1

    until docker-compose exec postgres psql -h localhost -U <user> -d <db> -c "select 1" > /dev/null 2>&1 || [ $retries -eq 0 ]; do
        ((retries--)) || true
        echo "Waiting for postgres server, $retries remaining attempts..."
        sleep 1
    done

    if [ ${retries} -eq 0 ]; then
        echo "DB took too long to start...exiting"
        exit 1
    else
        echo "DB Ready..."
    fi
}
```

which works fine but this can be simplified for postgres using the `pg_isready` utility which
is a part of the postgres cli utilities for most distros. Adding this to my above script increases readability --
making it clearer what the line is doing.


:rotating_light: Warning: `pg_isready` only checks for TCP connection
readiness. Important note most of the Docker containers for postgres start,
create a new user & database then **restart** so `pg_isready` is not a great tool
for handling a case where you need to wait for your DB to ready in say an integration test suite

```bash
function wait_for_db {
    local retries=5

    local service_up=1

    until docker-compose exec pg_isready -h postgres-host -p 5432 -U postgres || [ $retries -eq 0 ]; do
        ((retries--)) || true
        echo "Waiting for postgres server, $retries remaining attempts..."
        sleep 1
    done

    if [ ${retries} -eq 0 ]; then
        echo "DB took too long to start...exiting"
        exit 1
    else
        echo "DB Ready..."
    fi
}
```
