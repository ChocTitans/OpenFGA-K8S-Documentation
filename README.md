# OpenFGA

A high-performance and flexible authorization/permission engine built for developers and inspired by [Google Zanzibar](https://research.google/pubs/pub48190/).

OpenFGA is designed to make it easy for developers to model their application permissions and add and integrate fine-grained authorization into their applications.

It allows in-memory data storage for quick development, as well as pluggable database modules. It currently supports PostgreSQL 14 and MySQL 8.

It offers an [HTTP API](https://openfga.dev/api/service) and a [gRPC API](https://buf.build/openfga/api/file/main:openfga/v1/openfga_service.proto). It has SDKs for [Node.js/JavaScript](https://www.npmjs.com/package/@openfga/sdk), [GoLang](https://github.com/openfga/go-sdk), [Python](https://github.com/openfga/python-sdk) and [.NET](https://www.nuget.org/packages/OpenFga.Sdk). Look in our [Community section](https://github.com/openfga/community#community-projects) for third-party SDKs and tools.

## Getting Started

The following section aims to explian how to deploy OpenFGA as service in Docker Swarm . For further infos look at theofficial [documentation](https://openfga.dev/) for in-depth information.

### Setup and Installation

> ℹ️ The following sections setup an OpenFGA server using the default configuration values. These are for rapid development and not for a production environment. Data written to an OpenFGA instance using the default configuration with the memory storage engine will *not* persist after the service is stopped.
>
> For more information on how to configure the OpenFGA server, take a look at the official documentation on [Configuring OpenFGA](https://openfga.dev/docs/getting-started/setup-openfga#configuring-the-server) or our [Production Checklist](https://openfga.dev/docs/getting-started/setup-openfga#production-checklist).

#### Docker

OpenFGA is available on [Dockerhub](https://hub.docker.com/r/openfga/openfga), so you can quickly start it using the in-memory datastore by running the following commands:

```bash
docker pull openfga/openfga
docker run -p 8080:8080 -p 3000:3000 openfga/openfga run
```

#### Docker Compose

[`docker-compose.yml`](./docker-compose.yml) provides an example of how to launch OpenFGA with Postgres using `docker compose`.

1. First, either use the `docker-compose.yml`in this repo, or use the one that OpenFGA suggests with the following command:

   ```bash
   curl -LO https://openfga.dev/docker-compose.yaml
   ```

2. Then, run the following command:

   ```bash
   docker stack deploy -c docker-compose.yml OpenFGA
   ```
