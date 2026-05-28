# Grafana, Prometheus and Port Access

## Goal

Access Grafana from outside the server and connect it correctly to Prometheus metrics.

## Initial Problem

Grafana was running in Docker but mapped only to localhost:

```text
6842b37ad0aa   grafana/grafana:latest   "/run.sh"   Up   127.0.0.1:3001->3000/tcp   grafana
```

This means Grafana was only accessible from the server itself, not other machines.

## Fix Concept

The Docker port binding should use `0.0.0.0`, not `127.0.0.1`, if access is needed from another machine in the network.

Example:

```bash
docker run -d \
  --name grafana \
  -p 3001:3000 \
  grafana/grafana:latest
```

This results in:

```text
0.0.0.0:3001->3000/tcp
```

## Prometheus Connection Issue

Grafana showed an error similar to:

```text
Post "http://localhost:9090/api/v1/query_range": dial tcp [::1]:9090: connect: connection refused
```

## Root Cause

Inside a Docker container, `localhost` refers to the Grafana container itself, not the host machine or Prometheus container.

## Correct Approach

Use one of these instead:

### If Prometheus is another Docker container on same Docker network

Use container name:

```text
http://prometheus:9090
```

### If Prometheus is running on host machine

Use host gateway:

```text
http://host.docker.internal:9090
```

On Linux, host gateway may need explicit mapping:

```bash
--add-host=host.docker.internal:host-gateway
```

## Prometheus URL Confusion

Observed:

```text
http://localhost:9090/query
```

works in browser, but Grafana API expects endpoints like:

```text
http://localhost:9090/api/v1/query
http://localhost:9090/api/v1/query_range
```

The `/query` URL is the Prometheus web UI route. Grafana talks to the Prometheus API.

## Lessons Learned

- `127.0.0.1:3001->3000` means only server-local access.
- `0.0.0.0:3001->3000` means LAN access is possible.
- Grafana inside Docker should not use `localhost` to call Prometheus unless Prometheus is inside the same container.
- Grafana requires Prometheus API access, not just the web UI route.

## Suggested Repo Location

```text
homelab-docs/docs/monitoring/grafana-prometheus.md
```
