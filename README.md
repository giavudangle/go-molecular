# Go Microservice Example via MarrioCarion

## Introduction



1. It is _enterprise_, meant to last for years,
1. It allows a team to collaborate efficiently with little friction, and
1. It is as idiomatic as possible.


## Domain Driven Design

This project uses a lot of the ideas introduced by Eric Evans in his book [Domain Driven Design](https://www.domainlanguage.com/), I do encourage reading that book but before I think reading [Domain-Driven Design Distilled](https://smile.amazon.com/Domain-Driven-Design-Distilled-Vaughn-Vernon/dp/0134434420/) makes more sense, also there's a free to download [DDD Reference](https://www.domainlanguage.com/ddd/reference/) available as well.

On YouTube I created [a playlist](https://www.youtube.com/playlist?list=PL7yAAGMOat_GJqfTdM9PBdTRSH7jXs6mI) that includes some of my favorite talks and webinars, feel free to explore that as well.

## Project Structure

Talking specifically about microservices **only**, the structure I like to recommend is the following, everything using `<` and `>` depends on the domain being implemented and the bounded context being defined.

- [ ] `build/`: defines the code used for creating infrastructure as well as docker containers.
  - [ ] `<cloud-providers>/`: define concrete cloud provider.
  - [ ] `<executableN>/`: contains a Dockerfile used for building the binary.
- [ ] `cmd/`
  - [ ] `<primary-server>/`: uses primary database.
  - [ ] `<replica-server>/`: uses readonly databases.
  - [ ] `<binaryN>/`
- [X] `db/`
  - [X] `migrations/`: contains database migrations.
  - [ ] `seeds/`: contains file meant to populate basic database values.
- [ ] `internal/`: defines the _core domain_.
  - [ ] `<datastoreN>/`: a concrete _repository_ used by the domain, for example `postgresql`
  - [ ] `http/`: defines HTTP Handlers.
  - [ ] `service/`: orchestrates use cases and manages transactions.
- [X] `pkg/` public API meant to be imported by other Go package.

There are cases where requiring a new bounded context is needed, in those cases the recommendation would be to
define a package like `internal/<bounded-context>` that then should follow the same structure, for example:

* `internal/<bounded-context>/`
  * `internal/<bounded-context>/<datastoreN>`
  * `internal/<bounded-context>/http`
  * `internal/<bounded-context>/service`

## Tools

```
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@v4.14.1
go install github.com/kyleconroy/sqlc/cmd/sqlc@v1.6.0
go install github.com/maxbrunsfeld/counterfeiter/v6@v6.3.0
go install github.com/deepmap/oapi-codegen/cmd/oapi-codegen@v1.5.1
go install goa.design/model/cmd/mdl@v1.7.6
go install goa.design/model/cmd/stz@v1.7.6
go install github.com/fdaines/spm-go@v0.11.1
```

## Features
- [X] Project Layout 
- [X] Dependency Injection 
- [X] Secure Configuration
  - [X] Using Hashicorp Vault
  - [X] Using AWS SSM
- [ ] Infrastructure as code
- [X] Metrics, Traces and Logging using OpenTelemetry
- [ ] Caching
  - [X] Memcached 
  - [ ] Redis
- [X] Persistent Storage
  - [X] Repository Pattern 
  - [X] Database migrations 
  - [ ] MySQL
  - [X] PostgreSQL
- [ ] REST APIs
  - [X] HTTP Handlers 
  - [X] Custom JSON Types 
  - [ ] Versioning
  - [X] Error Handling 
  - [X] OpenAPI 3 and Swagger-UI
  - [ ] Authorization
- [ ] Events and Messaging
  - [ ] Apache Kafka
  - [ ] RabbitMQ
  - [ ] Redis
- [ ] Testing
  - [X] Type-safe mocks with 
  - [X] Equality with 
  - [X] Integration tests for Datastores with 
  - [X] REST APIs 
- [ ] Containerization using Docker
- [ ] Graceful Shutdown 
- [ ] Search Engine using ElasticSearch
- [ ] Documentation
  - C4 Model
- [ ] Cloud Design Patterns
  * Reliability
    - [ ] Circuit Breaker
- [ ] Whatever else I forgot to include

## More ideas

* [2016: Peter Bourgon's: Repository structure](https://peter.bourgon.org/go-best-practices-2016/#repository-structure)
* [2016: Ben Johnson's: Standard Package Layout](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1)
* [2017: William Kennedy's: Design Philosophy On Packaging](https://www.ardanlabs.com/blog/2017/02/design-philosophy-on-packaging.html)
* [2017: Jaana Dogan's: Style guideline for Go packages](https://rakyll.org/style-packages/)
* [2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0)

## Docker Containers

Please notice in order to run this project locally you need to run a few programs in advance, if you use Docker please refer to the concrete instructions in [`docs/`](docs/) for more details.

There's also a [docker-compose.yml](docker-compose.yml), covered in [Building Microservices In Go: Containerization with Docker](https://youtu.be/u_ayzie9pAQ), however like I mentioned in the video you have to execute `docker-compose` in multiple steps.

Notice that because of the way RabbitMQ and Kafka are being used they are sort of competing with each other, so at the moment we either have to enable Kafka and disable RabbitMQ or the other way around in both the code and the `docker-compose.yml` file, in either case there are Dockerfiles and services defined that cover building and running them.

* Run `docker-compose up`, here both _rest-server_ and _elasticsearch-indexer_ services will fail because the `postgres`, `rabbitmq`, `elasticsearch` and `kafka` services take too long to start.
    * If you're planning to use RabbitMQ, run `docker-compose up rest-server elasticsearch-indexer-rabbitmq`.
    * If you're planning to use Kafka, run `docker-compose up rest-server elasticsearch-indexer-kafka`.
    * If you're planning to use Redis, run `docker-compose up rest-server elasticsearch-indexer-redis`.
* For building the service images you can use:
    * `rest-server` image: `docker-compose build rest-server`.
    * `elasticsearch-indexer-rabbitmq` image: `docker-compose build elasticsearch-indexer-rabbitmq`.
    * `elasticsearch-indexer-kafka` image: `docker-compose build elasticsearch-indexer-kafka`.
    * `elasticsearch-indexer-redis` image: `docker-compose build elasticsearch-indexer-redis`.
* Run `docker-compose run rest-server migrate -path /api/migrations/ -database postgres://user:password@postgres:5432/dbname?sslmode=disable up` to finally have everything working correctly.

## Diagrams

To start a local HTTP server that serves a graphical editor:

```
mdl serve github.com/MarioCarrion/todo-api/internal/doc -dir docs/diagrams/
```

To generate JSON artifact for uploading to [structurizr](https://structurizr.com/):

```
stz gen github.com/MarioCarrion/todo-api/internal/doc
```
