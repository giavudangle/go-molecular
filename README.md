
## Requirements

1. Go 1.15
1. [`direnv`](https://mariocarrion.com/2020/11/20/golang-go-tool-direnv.html): for _sanboxing_ the required tools used by this project.
1. [PostgreSQL](#postgresql): either a remote instance, local binary or docker container.
1. [Memcached](#memcached): either a remote instance, local binary or docker container.
1. [Load testing data](#load-testing-data)

## Running

Download the required packages:

```
go mod download
```

Then you can run `server.go` with the PostgreSQL and memcached configuration:

```
DATABASE_URL=postgres://user:vudang@localhost:5432/service_db?sslmode=disable \
MEMCACHED=localhost:11211 \
  go run .
```


#### PostgreSQL and PgAdmin4 Docker 

```bash
# start a postgres container
docker run \
  -d \
  -e POSTGRES_HOST_AUTH_METHOD=trust \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=vudang \
  -e POSTGRES_DB=dbname \
  -p 5432:5432 \
  postgres
```

### Reverse Proxying
```bash
# start pgAdmin container
docker pull dpage/pgadmin4
docker run -p 5050:80 \
    -e 'PGADMIN_DEFAULT_EMAIL=vudangdev@gmail.com' \
    -e 'PGADMIN_DEFAULT_PASSWORD=vudang' \
    -d dpage/pgadmin4

# create server 
host_name: your ipV4 address given from $ifconfig or $ipconfig
```

#### Migrations

Make sure the `migrate` tool is installed:

```
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate
```

Then, assuming you used docker, you can run:

```
migrate -path db/migrations/ -database postgres://user:vudang@localhost:5432/service_db?sslmode=disable up
```


### Memcached

```
docker run \
  -d \
  -p 11211:11211 \
  memcached
```

### Load testing data

You can use the tool I build during my [Implementing Complex Pipelines in Go](https://mariocarrion.com/2020/08/27/go-implementing-complex-pipelines-part-5.html) series, install it using:

```
go install github.com/MarioCarrion/complex-pipelines/part5
```

Then run it using:

```
DATABASE_URL="postgres://user:vudang@localhost:5432/service_db?sslmode=disable" part5
```

Sadly, at the moment this tool does not provide any progress feedback; but it should finish after a couple of minutes.