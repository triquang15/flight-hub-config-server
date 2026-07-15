# FlightHub Config Server

Centralized Spring Cloud Config repository for the FlightHub microservice
platform.

This repo stores runtime configuration only. Do not commit production secrets,
OAuth client secrets, database passwords, S3 keys, or payment provider keys.
Service YAML files use Spring placeholders and each running service resolves
real values from environment variables or the deployment secret manager.

## Active Config Files

| File | Service | Local port | Notes |
| --- | --- | ---: | --- |
| `application.yml` | Shared defaults | - | Eureka URL, JPA defaults, actuator/Prometheus exposure |
| `api-gateway.yml` | API Gateway | 8080 | CORS, gateway-level runtime settings |
| `airline-core-service.yml` | Airline Core | 8081 | Airlines, aircraft, cabins, airline assets |
| `ancillary-service.yml` | Ancillary | 8082 | Baggage, meals, master ancillaries |
| `booking-service.yml` | Booking | 8083 | Booking, ticketing, passenger records |
| `flight-ops-service.yml` | Flight Ops | 8084 | Flights, schedules, flight instances |
| `location-service.yml` | Location | 8085 | Airports, cities, route search data |
| `payment-service.yml` | Payment | 8086 | Stripe, PayPal payment providers |
| `pricing-service.yml` | Pricing | 8087 | Fares, coupons, fare rules |
| `seat-service.yml` | Seat | 8088 | Seat maps, seat inventory, holds |
| `media-service.yml` | Media | 8089 | Local/S3-ready upload storage |
| `user-service.yml` | User | 8090 | Auth, users, social login metadata |
| `notification-service.yml` | Notification | 8091 | Email/SMS/Kafka notification events |

`subscription-service.yml` was removed because subscription is not part of the
current FlightHub runtime and its old port conflicted with `media-service`.
Reintroduce it only when the service module and gateway routes are restored.

## Local Development

The defaults are local-friendly so services can run from the IDE without
duplicating every setting in each Run Configuration:

- PostgreSQL defaults: `localhost`, user `postgres`, password `12345678`
- Kafka default: `localhost:9092`
- Redis default: `localhost:6379`
- Eureka default: `http://localhost:8761/eureka/`
- JPA default: `ddl-auto=update`
- SQL logging: enabled for debugging

Start order:

1. `service-registry`
2. `config-server`
3. `api-gateway`
4. Domain services
5. `flighthub-web`

Validate that config-server is serving a file:

```bash
curl http://localhost:8888/booking-service/default
curl http://localhost:8888/media-service/default
curl http://localhost:8888/api-gateway/default
```

## Environment Variables

Use the root `FlightHub/.env.local` and `FlightHub/.env.local.example` as the
local template. In production, inject the same keys through Docker/Kubernetes or
your cloud secret manager.

Common variables:

```bash
FLIGHTHUB_DATASOURCE_USERNAME=...
FLIGHTHUB_DATASOURCE_PASSWORD=...
EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=...
SPRING_KAFKA_BOOTSTRAP_SERVERS=...
JPA_DDL_AUTO=validate
```

Service-specific datasource variables override the shared credentials when set:

```bash
USER_DATASOURCE_URL=...
USER_DATASOURCE_USERNAME=...
USER_DATASOURCE_PASSWORD=...

AIRLINE_CORE_DATASOURCE_URL=...
BOOKING_DATASOURCE_URL=...
MEDIA_DATASOURCE_URL=...
NOTIFICATION_DATASOURCE_URL=...
```

Media storage variables:

```bash
MEDIA_STORAGE_PROVIDER=LOCAL
MEDIA_STORAGE_PATH=uploads/media
MEDIA_PUBLIC_BASE_URL=http://localhost:8080

# S3-ready, disabled until MEDIA_STORAGE_PROVIDER=S3
MEDIA_S3_BUCKET=
MEDIA_S3_REGION=us-east-1
MEDIA_S3_PUBLIC_BASE_URL=
MEDIA_S3_ENDPOINT=
MEDIA_S3_PATH_STYLE_ACCESS=false
MEDIA_S3_ACCESS_KEY_ID=
MEDIA_S3_SECRET_ACCESS_KEY=
```

Payment and social login secrets must stay outside Git:

```bash
STRIPE_SECRET_KEY=...
STRIPE_WEBHOOK_SECRET=...
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...
PAYPAL_WEBHOOK_ID=...
GOOGLE_CLIENT_ID=...
FACEBOOK_APP_ID=...
FACEBOOK_APP_SECRET=...
```

## Production Rules

- Keep `JPA_DDL_AUTO=validate` in production.
- Use Flyway migrations for schema changes.
- Store secrets in a secret manager or deploy platform environment variables.
- Keep only placeholders and safe local defaults in this repository.
- Rotate any credential that was ever committed or pasted into logs/chat.

## Docker Compose Example

```yaml
services:
  booking-service:
    env_file:
      - .env.prod
```

Environment variables must exist in the target service process. Setting a
secret only on config-server does not inject it into client services.
