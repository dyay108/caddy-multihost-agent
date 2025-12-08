# Developer Notes

> See [README.md](README.md) for user documentation.

IMPORTANT: Developed on Windows, must work on Linux.

## Test Hosts (SSH passwordless)

| Host | IP | Role | SSH |
|------|-----|------|-----|
| host1 | 192.168.0.96 | Server (caddy-docker-proxy) | `ssh root@192.168.0.96` |
| host2 | 192.168.0.98 | Agent | `ssh root@192.168.0.98` |
| host3 | 192.168.0.99 | Agent | `ssh root@192.168.0.99` |

## Production Hosts

| Host | IP | Role | SSH |
|------|-----|------|-----|
| prod-host1 | 192.168.0.112 | Server | `ssh root@192.168.0.112` |
| prod-host2 | 192.168.0.235 | Agent | `ssh root@192.168.0.235` |
| prod-host3 | 192.168.0.212 | Agent | `ssh root@192.168.0.212` |

## Test Infrastructure

Files in `tests/`:
- `docker-compose-prod-server.yml` - Server config (host1)
- `docker-compose-prod-agent2.yml` - Agent config (host2)
- `docker-compose-prod-agent3.yml` - Agent config (host3)
- `deploy-hosts.sh` - Deploy to test hosts
- `cleanup-hosts.sh` - Cleanup test hosts
- `test_all.py` - Test suite

## Deployment

```bash
# Build and deploy all hosts
./tests/deploy-hosts.sh --build

# Deploy single host
./tests/deploy-hosts.sh host1 --build

# Cleanup
./tests/cleanup-hosts.sh
./tests/cleanup-hosts.sh --volumes
```

## Testing

```bash
# All tests
python tests/test_all.py

# Unit only (no hosts needed)
python tests/test_all.py --unit

# Integration only (requires deployed hosts)
python tests/test_all.py --integration
```

## Test Domains

| Pattern | TLS | Notes |
|---------|-----|-------|
| `*.lacny.me` | Cloudflare DNS | Real wildcard cert |
| `*.lan` | Internal CA | Self-signed |
| `*.server.lan` | Internal CA | Server mode testing |

## Quick Debug

```bash
# Check Caddy config
curl -s http://192.168.0.96:2020/config/ | head -c 500

# Agent logs
ssh root@192.168.0.98 "docker logs caddy-agent-remote --tail 20"
ssh root@192.168.0.99 "docker logs caddy-agent-remote3 --tail 20"

# Test route
curl -sk --resolve test-remote.lan:443:192.168.0.96 https://test-remote.lan
```

## Architecture Notes

- Admin API exposed on `:2020` (proxies to internal `:2019`)
- `caddy-docker-proxy` handles `caddy.*` labels on host1
- `caddy-agent-server` on host1 uses `agent.*` prefix for testing
- Remote agents push to `:2020`, not `:2019`
