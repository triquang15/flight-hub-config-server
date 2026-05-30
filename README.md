# FlightHub Config Server Properties

This repository should not contain production database credentials. Service YAML files use Spring placeholders, and the running service resolves them from environment variables or platform secrets.

## Production Secret Flow

1. Store real values in your deploy platform secret manager, for example Docker Compose `.env`, Kubernetes `Secret`, Render/Railway/Fly/EC2 environment variables, or Vault.
2. Inject the variables into each microservice container. The variables must exist in the client service process, not only in the config-server process.
3. Keep `JPA_DDL_AUTO=validate` in production. Use `JPA_DDL_AUTO=update` only for local/dev environments.

## Required Variables

Use `.env.prod.example` as the template. Each service needs its own datasource URL, and all services can share `FLIGHTHUB_DATASOURCE_USERNAME` and `FLIGHTHUB_DATASOURCE_PASSWORD` when the same DB user is used.

Per-service credentials are also supported:

```bash
USER_DATASOURCE_USERNAME=...
USER_DATASOURCE_PASSWORD=...
AIRLINE_CORE_DATASOURCE_USERNAME=...
AIRLINE_CORE_DATASOURCE_PASSWORD=...
```

If per-service credentials are not set, the config falls back to the shared `FLIGHTHUB_DATASOURCE_USERNAME` and `FLIGHTHUB_DATASOURCE_PASSWORD`.

## Docker Compose Example

```yaml
services:
  airline-core-service:
    env_file:
      - .env.prod
```

## Security Note

If real credentials were ever committed or pasted into chat/logs, rotate the database password in Neon and redeploy with the new secret.
