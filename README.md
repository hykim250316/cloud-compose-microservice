# Dockerised web app and microservice

A small student-records app built up in six steps, each one a Docker Compose stack
that adds a piece of infrastructure to the previous one. Done as an individual
project for the Cloud Computing module of my MSc at University College Dublin
(October 2026).

Each `taskN/` folder is a complete, runnable stack. `task6/` is the final result.

| Step | What it adds |
|---|---|
| task1 | Flask web app in a container (non-root user, health check, `docker compose up`) |
| task2 | FastAPI service behind the web app, started only when the API is healthy |
| task3 | Postgres with a seed script, a named volume and pgweb for browsing the data; separate `frontend` / `backend` networks |
| task4 | Redis cache in front of the student list, with a cache-info endpoint and a way to clear it |
| task5 | Prometheus and Grafana, with Postgres, Redis and blackbox exporters and a provisioned dashboard |
| task6 | Traefik reverse proxy: everything behind one port (`/app`, `/api`, `/pgweb`, `/prometheus`, `/grafana`) |

## Run the final stack

```bash
cd task6
cp .env.example .env        # then change the passwords
docker compose up --build
```

Then open:

- http://localhost/app — the web app
- http://localhost/api/docs — FastAPI docs
- http://localhost/pgweb — database browser
- http://localhost/prometheus and http://localhost/grafana — monitoring

Only Traefik publishes a port on the host. The database and Redis are reachable
only on the internal `backend` network.

## Things I had to work out

- Health checks that use what is already in the image (`python -c "urllib.request..."`, `pg_isready`, `redis-cli ping`) instead of adding curl everywhere.
- `depends_on` with `condition: service_healthy` so the web app does not start before the API, and the API not before Postgres and Redis.
- Running Prometheus and Grafana under a path prefix behind Traefik (`--web.route-prefix`, `GF_SERVER_SERVE_FROM_SUB_PATH`).
- A tiny WSGI middleware in the Flask app so it works under `/app` without changing every route.

## Notes

Secrets live in `.env` (ignored by git). `.env.example` shows the variables each
step needs. The seed data in `task*/init-scripts/01-init.sql` is made up.
