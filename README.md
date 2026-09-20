# Microservices-Task - Dockerized Submission

This repo contains four Node.js/Express microservices, each containerized with its own
`Dockerfile`, and orchestrated together with `docker-compose.yml`.

| Service          | Port | Description                                  |
|-------------------|------|-----------------------------------------------|
| user-service      | 3000 | Returns a list of users                       |
| product-service   | 3001 | Returns a list of products                    |
| order-service     | 3002 | Create / list orders                          |
| gateway-service   | 3003 | Aggregates the above services under `/api/*`  |

## Folder Structure

```
submission/
├── user-service/
│   ├── Dockerfile
│   ├── app.js
│   └── package.json
├── product-service/
│   ├── Dockerfile
│   ├── app.js
│   └── package.json
├── order-service/
│   ├── Dockerfile
│   ├── app.js
│   └── package.json
├── gateway-service/
│   ├── Dockerfile
│   ├── app.js
│   └── package.json
├── docker-compose.yml
└── README.md
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2 syntax, bundled with
  modern Docker Desktop — run as `docker compose`, or install the standalone
  `docker-compose` binary)

## Setup Instructions

1. Clone this repository:
   ```bash
   git clone https://github.com/datsondavid/Microservices-Task.git
   cd Microservices-Task/submission
   ```

2. Build and start all four services:
   ```bash
   docker compose up --build
   ```
   (or `docker-compose up --build` on older installs)

3. Docker Compose will:
   - Build an image for each service from its own `Dockerfile`
   - Start all four containers on a shared bridge network (`microservices-net`),
     so they can reach each other by service name (e.g. `http://user-service:3000`)
   - Map each container's port to the same port on your host machine

4. To run in the background instead:
   ```bash
   docker compose up --build -d
   ```

5. To stop everything:
   ```bash
   docker compose down
   ```

## How to Test Each Service

Once `docker compose up` reports all four containers as running, test each one from
your host machine:

**User Service**
```bash
curl http://localhost:3000/health
curl http://localhost:3000/users
```

**Product Service**
```bash
curl http://localhost:3001/health
curl http://localhost:3001/products
```

**Order Service**
```bash
curl http://localhost:3002/health
curl http://localhost:3002/orders

# Create an order
curl -X POST http://localhost:3002/orders \
  -H "Content-Type: application/json" \
  -d '{"userId": 1, "productId": 2}'
```

**Gateway Service** (routes to the three services above)
```bash
curl http://localhost:3003/health
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders

# Create an order through the gateway
curl -X POST http://localhost:3003/api/orders \
  -H "Content-Type: application/json" \
  -d '{"userId": 1, "productId": 2}'
```

You can also open any of the `GET` URLs above directly in a browser.

To confirm all four containers are up and healthy:
```bash
docker compose ps
docker compose logs -f
```

## Screenshots

> All the screenshots are present in /screenshots
> - Terminal output of `docker compose up --build` showing all four services starting
> - `docker compose ps` / `docker ps` showing all containers in the `Up` state
> - Browser or `curl` output for `http://localhost:3000/users`,
>   `http://localhost:3001/products`, `http://localhost:3002/orders`, and
>   `http://localhost:3003/api/users`

## Troubleshooting

- **Port already in use** (`Bind for 0.0.0.0:3000 failed: port is already allocated`)
  Something else on your machine is using that port. Stop it, or change the host-side
  port mapping in `docker-compose.yml` (e.g. `"3010:3000"`), then re-run
  `docker compose up`.

- **Gateway returns `{"error": "Error fetching users"}` (or products/orders)**
  The gateway couldn't reach the target service over the internal network. Check:
  - All four containers are running: `docker compose ps`
  - You didn't rename a service in `docker-compose.yml` — the gateway calls services
    by their **container/service name** (`user-service`, `product-service`,
    `order-service`), not `localhost`.
  - Check that service's logs: `docker compose logs order-service`

- **Changes to `app.js` aren't showing up**
  Docker Compose caches built images. Rebuild with:
  ```bash
  docker compose up --build
  ```
  or force a clean rebuild:
  ```bash
  docker compose build --no-cache
  docker compose up
  ```

- **`docker compose` command not found**
  Your Docker install may only support the older standalone `docker-compose` binary.
  Use `docker-compose up --build` instead, or upgrade Docker Desktop / the Docker CLI
  plugin.

- **Containers exit immediately after starting**
  Run `docker compose logs <service-name>` to see the error. A common cause is a
  missing `node_modules` folder if you bind-mounted the source directory over the
  container's `/app` — this setup does not use bind mounts, so a fresh
  `docker compose up --build` should always work.

- **Reset everything and start clean**
  ```bash
  docker compose down -v
  docker system prune -f
  docker compose up --build
  ```
