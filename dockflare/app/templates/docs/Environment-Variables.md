# Environment Variables

This document lists environment variables supported by DockFlare, their defaults, and what they control. You can set these in your `.env` file, Docker Compose `environment:` block, or directly when running the container.

> Note: Some configuration items (CF credentials and other sensitive values) are migrated into the encrypted DockFlare configuration store during initial setup; variables set in the environment can be imported during pre-flight setup.

---

## Credentials / Bootstrapping

- **CF_API_TOKEN**
  - Default: (none)
  - Description: Cloudflare API token used to interact with Cloudflare (optional during initial import; recommended to set during setup or via the UI). When present in the environment during pre-flight, DockFlare will enable importing configuration from env.
  - Where used: Pre-flight import, `app/main.py`, Cloudflare API calls.

- **CF_ACCOUNT_ID**
  - Default: (none)
  - Description: Cloudflare account ID used for API calls when performing account-scoped operations.

- **CF_ZONE_ID**
  - Default: (none)
  - Description: Optional default zone ID used when no zone can be inferred from hostnames.

- **DOCKFLARE_API_KEY**
  - Default: (none)
  - Description: Master API key for DockFlare (used for agent enrollment and management).
  - Where used: `app/config.py` (exposed to app as `MASTER_API_KEY`), `app/main.py` pre-flight import.

---

## Managed cloudflared / Tunnel behavior

- **USE_EXTERNAL_CLOUDFLARED**
  - Default: `false`
  - Type: boolean
  - Description: If `true`, DockFlare will not manage an embedded `cloudflared` container and will expect an external tunnel to be provided. When `false`, DockFlare manages a `cloudflared` Docker container.
  - Where used: `app/config.py`, `app/core/tunnel_manager.py`.

- **EXTERNAL_TUNNEL_ID**
  - Default: (none)
  - Description: When using external mode, identifies the Cloudflare Tunnel ID to use.

- **CLOUDFLARED_NETWORK_NAME**
  - Default: `cloudflare-net`
  - Description: Docker network name that the managed `cloudflared` container is attached to. Must exist or be creatable by Docker. To change where the managed agent runs, set this value.
  - Example: `CLOUDFLARED_NETWORK_NAME=my-network`
  - Where used: `app/config.py`, `app/core/tunnel_manager.py` (container creation and network checks).

- **CLOUDFLARED_CONTAINER_NAME**
  - Default: `cloudflared-agent-dockflare-tunnel` (derived from `TUNNEL_NAME`)
  - Description: Name used for the managed `cloudflared` container.

- **CLOUDFLARED_IMAGE**
  - Default: `cloudflare/cloudflared:latest`
  - Description: Docker image used when DockFlare creates the managed agent. (Currently a constant in `config.py`.)

- **CLOUDFLARED_METRICS_PORT**
  - Default: (none)
  - Description: If set to a valid port number, enables Prometheus metrics on the managed `cloudflared` agent and exposes the port on that TCP port.

---

## Server & HTTP / Waitress

- **LOG_LEVEL**
  - Default: `WARNING`
  - Description: Controls DockFlare logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`).

- **WAITRESS_HOST**
  - Default: `0.0.0.0`
  - Description: Host/interface Waitress binds to.

- **WAITRESS_PORT**
  - Default: `5000`
  - Description: Port Waitress listens on.

- **WAITRESS_THREADS**, **WAITRESS_CONNECTION_LIMIT**, **WAITRESS_BACKLOG**, **WAITRESS_CHANNEL_TIMEOUT**
  - Defaults: `128`, `256`, `2048`, `360` respectively
  - Description: Waitress tuning knobs exposed as environment variables and validated as integers by `app/config.py`.

---

## Redis & Caching

- **REDIS_URL**
  - Default: (none)
  - Description: If set (for example `redis://redis:6379/0`), DockFlare will use Redis-backed caching instead of in-memory cache. Used by `app/core/cache.py`.

- **REDIS_DB_INDEX**
  - Default: `0`
  - Description: Index (0-15) used when constructing Redis URL for caching separation. Useful when sharing a single Redis instance with other services.

- **DNS_RECORDS_CACHE_TIMEOUT**
  - Default: `300` (seconds)
  - Description: TTL for cached DNS records (seconds).

- **CACHE_REFRESH_INTERVAL**
  - Default: `3600` (seconds)
  - Description: Interval for periodic cache refresh.

- **ENABLE_PERIODIC_CACHE_REFRESH**
  - Default: `true`
  - Description: Enable/disable scheduled cache refresh worker.

- **CACHE_ENABLED**
  - Default: `true`
  - Description: Global switching for caching. If `false`, caching is disabled.

---

## Agent / Multi-Server settings

- **AGENT_API_PREFIX**
  - Default: `/api/v2/agents`
  - Description: URL prefix for agent API endpoints on the Master.

- **AGENT_HEARTBEAT_TIMEOUT**
  - Default: `60` (seconds)
  - Description: Seconds without heartbeat before an agent is considered offline.

- **AGENT_ENROLLMENT_REQUIRED**
  - Default: `true`
  - Type: boolean
  - Description: If `true`, agents must be enrolled manually via the Master UI.

- **AGENT_KEY_STORAGE_PATH**
  - Default: (none)
  - Description: Optional path for storing agent API key metadata (defaults to encrypted config storage if unset).

- **AGENT_COMMAND_POLL_INTERVAL**
  - Default: `10` (seconds)
  - Description: Recommended agent polling interval for commands.

---

## Tuning & Maintenance

- **CLEANUP_INTERVAL_SECONDS**
  - Default: `60` (seconds)
  - Description: Interval at which the cleanup background task runs to remove expired rules.

- **AGENT_STATUS_UPDATE_INTERVAL_SECONDS**
  - Default: `30` (seconds)
  - Description: Periodic interval for the agent status updater thread.

- **MAX_CONCURRENT_DNS_OPS**
  - Default: `5`
  - Description: Maximum number of concurrent DNS operations performed against Cloudflare.

- **RECONCILIATION_BATCH_SIZE**
  - Default: `5`
  - Description: Batch size used during reconciliation of containers / rules.

- **SCAN_ALL_NETWORKS**
  - Default: `false`
  - Type: boolean
  - Description: If `true`, DockFlare will scan containers across all Docker networks instead of the default behavior.

- **AUTO_RESTORE_AGENT_RULES**
  - Default: `true`
  - Description: Master will attempt to restore rules reported by Agents if they are missing.

- **AUTO_RESTORE_COOLDOWN_SECONDS**
  - Default: `60` (seconds)
  - Description: Cooldown between automatic restore operations per agent.

- **MAX_CF_UPDATE_RETRIES**, **CF_UPDATE_RETRY_DELAY**, **CF_UPDATE_BACKOFF_FACTOR**
  - Use: retry behavior for Cloudflare API updates (constants in `config.py`).

---

## Feature Flags & Misc

- **LABEL_PREFIX**
  - Default: `dockflare.` (or overridden by `LABEL_PREFIX` env var)
  - Description: Prefix for Docker labels DockFlare consumes. Set `LABEL_PREFIX` to customize.

- **STATE_FILE_PATH**
  - Default: `/app/data/state.json`
  - Description: Path used to store DockFlare runtime state on disk.

- **USE_REUSABLE_POLICIES**
  - Default: `true`
  - Description: Toggle whether reusable Access Group templates are used.

- **SYNC_ALL_CLOUDFLARE_POLICIES**
  - Default: `false`
  - Description: If `true`, DockFlare will import/sync all Cloudflare policies (can be noisy for large accounts).

- **HOSTNAME**
  - Default: (container-provided)
  - Description: When running in Docker, DockFlare attempts to detect its container `HOSTNAME` and read a `dockflare.hostname` label to auto-configure `DOCKFLARE_PUBLIC_HOSTNAME`.

---

## Build-time / Dockerfile args

- **DOCKFLARE_UID**, **DOCKFLARE_GID**
  - Default: `65532`
  - Description: Build-time args used in `docker build` to set the runtime UID/GID for the `dockflare` system user. See `dockflare/Dockerfile`.

---

## Examples

Docker Compose environment snippet:

```yaml
environment:
  - REDIS_URL=redis://redis:6379/0
  - REDIS_DB_INDEX=0
  - DOCKER_HOST=tcp://docker-socket-proxy:2375
  - LOG_LEVEL=ERROR
  - CLOUDFLARED_NETWORK_NAME=cf_net
```

---
