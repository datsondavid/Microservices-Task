# Microservices Task

# Microservices-Task - Dockerized Submission

Node.js microservices containerized with Docker and orchestrated with Docker Compose.

## Services

| Service | Port | What it does |

| user-service | 3000 | Returns a list of users |
| product-service | 3001 | Returns a list of products |
| order-service | 3002 | Create and list orders |
| gateway-service | 3003 | Routes requests to the other three under `/api/*` |

## Folder Structure

```
Microservices-Task/
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
├── screenshots/
└── README.md
```

## Requirements

- Docker (20.10+)
- Docker Compose (v2, comes with Docker Desktop, run as `docker compose`. The older standalone `docker-compose` binary also works)

## Setup

Clone the repo and move into it:

```bash
git clone https://github.com/datsondavid/Microservices-Task.git
cd Microservices-Task
```

Build and start all four services:

```bash
docker compose up --build
```

This builds an image for each service from its own Dockerfile, starts all four containers on a shared network called `microservices-net`, and maps each container's port to the same port on your machine. Since they're on the same network, they can talk to each other using their service name, for example the gateway calls `http://user-service:3000` internally.

Run in the background instead:

```bash
docker compose up --build -d
```

Stop everything:

```bash
docker compose down
```

## Testing each service

Once all four containers are up, hit them from your host machine.

User service:
```bash
curl http://localhost:3000/health
curl http://localhost:3000/users
```

Product service:
```bash
curl http://localhost:3001/health
curl http://localhost:3001/products
```

Order service:
```bash
curl http://localhost:3002/health
curl http://localhost:3002/orders

curl -X POST http://localhost:3002/orders \
  -H "Content-Type: application/json" \
  -d '{"userId": 1, "productId": 2}'
```

Gateway service (this routes to the other three):
```bash
curl http://localhost:3003/health
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders

curl -X POST http://localhost:3003/api/orders \
  -H "Content-Type: application/json" \
  -d '{"userId": 1, "productId": 2}'
```

Any of the GET routes above also work fine in a browser.

To check container status and logs:
```bash
docker compose ps
docker compose logs -f
```

## Screenshots

Building and starting all services:

![docker compose cli](<screenshots/docker compose cli.png>)

All containers running:

![docker compose](<screenshots/docker compose.png>)

Docker Desktop showing all containers healthy:

![docker desktop success](<screenshots/docker desktop success.png>)

User service response:

![curl users](<screenshots/curl users.png>)

Product service response:

![curl products](<screenshots/curl products.png>)

Order service response:

![curl orders](<screenshots/curl orders.png>)

Gateway service response:

![curl api users](<screenshots/curl api users.png>)

Stopping the services:

![docker compose down](<screenshots/docker compose down.png>)
=======

## Troubleshooting

**Port already in use.** Something else on your machine is already using that port. Either stop it, or change the host side of the port mapping in `docker-compose.yml`, e.g. `"3010:3000"`, then run `docker compose up` again.

**Gateway returns an error fetching users/products/orders.** This usually means the gateway can't reach the target service over the internal network. Check that all four containers are running with `docker compose ps`, make sure you didn't rename a service in `docker-compose.yml` since the gateway calls services by name (`user-service`, `product-service`, `order-service`), not `localhost`. Check that specific service's logs with `docker compose logs order-service`.

**Changes to app.js aren't showing up.** Docker Compose caches built images. Rebuild with `docker compose up --build`, or force a clean rebuild with `docker compose build --no-cache` followed by `docker compose up`.

**"docker compose" command not found.** Your Docker install might only have the older standalone binary. Use `docker-compose up --build` instead, or update Docker Desktop.

**Containers exit right after starting.** Run `docker compose logs <service-name>` to see what happened. This setup doesn't use bind mounts, so a fresh `docker compose up --build` should always work.

**Start over from scratch:**
```bash
docker compose down -v
docker system prune -f
docker compose up --build
```
