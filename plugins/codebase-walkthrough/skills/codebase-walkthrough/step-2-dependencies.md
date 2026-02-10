# Step 2: Dependencies

**Goal:** Map all frameworks, libraries, and external systems this codebase relies on.

## 2.1 Frameworks & Libraries

Read dependency manifests (`go.mod`, `package.json`, `Cargo.toml`, `requirements.txt`, `pom.xml`, `build.gradle`, etc.) and categorize:

| Category             | What to look for                                          |
| -------------------- | --------------------------------------------------------- |
| RPC Framework        | gRPC, Thrift, Kitex, Dubbo, tRPC                          |
| HTTP Framework       | Gin, Echo, Fiber, Express, FastAPI, Spring MVC, Actix     |
| Dependency Injection | Wire, Dig/Fx, Spring, Guice, tsyringe                     |
| ORM / Data Access    | GORM, SQLAlchemy, Prisma, MyBatis, TypeORM, Diesel        |
| Task Scheduling      | Cron libraries, Temporal, Celery, Bull                    |
| Observability        | OpenTelemetry, Prometheus client, Sentry, Datadog         |
| Configuration        | Viper, dotenv, consul client, Apollo client               |
| Serialization        | Protobuf, JSON libraries, msgpack, Avro                   |
| Auth / Security      | JWT libraries, OAuth, RBAC, CASBIN                        |
| Testing              | Test frameworks, mocking libraries, fixture tools         |
| Logging              | Structured logging libraries (zap, logrus, winston, slog) |
| Validation           | Input validation libraries (validator, joi, zod)          |

## 2.2 External Systems

Search for connection strings, client initializations, driver imports, and infrastructure config to identify:

| Category           | Examples                                        |
| ------------------ | ----------------------------------------------- |
| Relational DB      | MySQL, PostgreSQL, SQLite, SQL Server           |
| Document DB        | MongoDB, DynamoDB, CouchDB                      |
| Cache              | Redis, Memcached                                |
| Message Queue      | Kafka, RabbitMQ, Pulsar, NSQ, SQS               |
| Object Storage     | S3, OSS, MinIO, GCS                             |
| Search Engine      | Elasticsearch, OpenSearch, Meilisearch          |
| LLM / AI Service   | OpenAI, Anthropic, local model inference        |
| Service Discovery  | Consul, Nacos, etcd, Zookeeper                  |
| Config Center      | Apollo, Nacos config, Spring Cloud Config       |
| Monitoring Backend | Jaeger, Prometheus, Grafana, Datadog            |
| Other Services     | HTTP/RPC calls to internal or external services |

## 2.3 Output

```markdown
## Dependencies

### Frameworks & Libraries

| Category | Library    | Purpose                        |
| -------- | ---------- | ------------------------------ |
| HTTP     | gin v1.9   | Request routing and middleware |
| ORM      | gorm v1.25 | Database access                |
| ...      | ...        | ...                            |

### External Systems

| System          | Type          | Usage                        |
| --------------- | ------------- | ---------------------------- |
| MySQL (primary) | Relational DB | Core data storage            |
| Redis           | Cache         | Session cache, rate limiting |
| Kafka           | Message Queue | Event publishing             |
| user-service    | RPC (gRPC)    | User profile lookups         |
| ...             | ...           | ...                          |

### Dependency Diagram

<ASCII diagram showing how the application connects to external systems>
```
