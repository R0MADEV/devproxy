# devproxy

Local development reverse proxy using [Traefik](https://traefik.io/). Routes multiple Docker projects to clean `.localhost` domains without port conflicts.

## How it works

Each project gets its own Docker subnet (e.g. `10.200.1.0/24`) with fixed IPs. Traefik routes traffic by hostname to those IPs — no exposed ports, no conflicts.

```
browser → my-project.localhost:8090 → Traefik → 10.200.1.10:80
browser → other-project.localhost:8090 → Traefik → 10.200.2.10:80
```

## Requirements

- Docker Desktop

## Quick start

```bash
cd docker
docker compose up -d
```

Traefik dashboard available at `http://localhost:8091`.

## Adding a project

**1. Assign a subnet to your project's `docker-compose.yml`:**

```yaml
services:
  web:
    networks:
      mynetwork:
        ipv4_address: 10.200.1.10

networks:
  mynetwork:
    driver: bridge
    ipam:
      config:
        - subnet: 10.200.1.0/24
```

**2. Add a router and service to `docker/traefik/traefik.yml`:**

```yaml
http:
  routers:
    my-project:
      rule: "Host(`my-project.localhost`)"
      entryPoints:
        - web
      service: my-project

  services:
    my-project:
      loadBalancer:
        servers:
          - url: "http://10.200.1.10:80"
```

Traefik reloads automatically — no restart needed.

## Ports

| Port | Description |
|------|-------------|
| `8090` | HTTP traffic |
| `8091` | Traefik dashboard |

> These ports avoid conflicts if you already have another Traefik instance running on port 80.

## Subnet convention

Use `10.200.x.0/24` and increment `x` per project:

| Project | Subnet |
|---------|--------|
| project-1 | `10.200.1.0/24` |
| project-2 | `10.200.2.0/24` |
| project-3 | `10.200.3.0/24` |
