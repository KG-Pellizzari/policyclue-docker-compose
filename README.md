# PolicyClue – Docker Compose

Reference deployment for running PolicyClue on a single host with Docker
Compose. Full deployment, operations, and SSO documentation lives at
**[docs.policyclue.com/deploy/deployment/](https://docs.policyclue.com/deploy/deployment/)**.

## Quick start

1. Install Docker Engine and Docker Compose on the host.
2. Clone this repository.
3. Copy `.env.example` to `.env` and fill in the secrets (DB password, Elastic
   password, `PCLUE_SECRET`, mail and Entra credentials as needed).
4. Sign in to the image registry - credentials are provided by PolicyClue:
   ```bash
   docker login registry.khost.ch
   ```
5. Start the stack:
   ```bash
   docker compose up -d
   ```
   Database migrations run automatically on first boot.
6. The portal is reachable on port `80` of the host (Traefik). Put it behind
   your own TLS terminator, or enable the ACME override below.

## Files in this repo

| File | Purpose |
|---|---|
| `docker-compose.yml` | The base stack: Postgres, Redis, Elasticsearch, the API + workers, the webapp, and Traefik. |
| `.env.example` | Template for `.env`. Copy and fill in. |
| `docker-compose.override.template` | Template for `docker-compose.override.yml`. Copy and uncomment the includes you want. |
| `docker-compose-acme.yml` | Optional override: Traefik with HTTPS via static certificates. |
| `docker-compose-ssl.yml` | Optional override: Traefik with HTTPS via Let's Encrypt. |
| `docker-compose-pgadmin.yml` | Optional override: pgAdmin on port `8081` for database inspection. |
| `docker-compose-ollama.yml` | Optional override: self-hosted Ollama container for on-prem LLM inference. |

## How the override files work

> **Always use a `docker-compose.override.yml` instead of editing
> `docker-compose.yml` directly.** New PolicyClue releases regularly add or
> change services in the base file; if you've modified it in place, every
> upgrade turns into a manual merge. Keeping your local changes in an
> override file lets you `git pull` future updates cleanly.

Compose merges files listed under the top-level `include:` key. The base
`docker-compose.yml` ships with all overrides commented out:

```yaml
include:
# - docker-compose-acme.yml
# - docker-compose-pgadmin.yml
# - docker-compose-ollama.yml
```

To enable them, copy `docker-compose.override.template` to
`docker-compose.override.yml` and uncomment the includes there. Compose
auto-loads any file named `docker-compose.override.yml`, and it stays out
of version control:

```yaml
# docker-compose.override.yml
include:
  - docker-compose-acme.yml
  - docker-compose-ollama.yml
```

You can also add ad-hoc service tweaks (extra env vars, port bindings,
resource limits, additional containers) to the same override file - they
merge into the base services on top of the base file.

### Override details

- **`docker-compose-acme.yml`** - switches Traefik to HTTPS with automatic
  Let's Encrypt certificates via the HTTP-01 challenge. Requires `ACME_EMAIL`
  in `.env` and that the host is reachable on TCP/80 and TCP/443 from the
  internet. Set `ACME_CASERVER` to use the Let's Encrypt staging endpoint or
  a private ACME server. Certificates are persisted in a `traefik` volume.

- **`docker-compose-pgadmin.yml`** - runs pgAdmin on port `8081` for direct
  database access. Default credentials are `admin@policyclue.com` / `admin` -
  change them or restrict access to localhost before using in any shared
  environment.

- **`docker-compose-ollama.yml`** - adds an Ollama container on the backend
  network for on-prem LLM inference. After enabling it, pull the model once:
  ```bash
  docker compose exec ollama ollama pull qwen2.5:14b
  ```
  Then set `OVERRIDE_LLM_BASE_URL=http://ollama:11434/v1` and
  `OVERRIDE_LLM_MODEL=qwen2.5:14b` in `.env` to route LLM calls to this
  local Ollama instead of the central PolicyClue services gateway. Uncomment
  the GPU `deploy:` block in the file for NVIDIA acceleration. See the
  Services Gateway / LLM sections in the
  [official deployment docs](https://docs.policyclue.com/deploy/deployment/)
  for tuning and alternative backends (OpenAI, LiteLLM, Azure).

## Internet-restricted environments

If outbound traffic from the host is filtered, see the
[Network Allow-List](https://docs.policyclue.com/deploy/network/) page in
the official documentation. It lists every external hostname the stack
contacts, grouped by feature, and shows the minimal allow-list when
optional modules are disabled.

## Further reading

- **Deployment, SSO, M365 webhook setup, BAS, LLM gateway** -
  [docs.policyclue.com/deploy/deployment/](https://docs.policyclue.com/deploy/deployment/)
- **Operations & upgrades** -
  [docs.policyclue.com/deploy/operations/](https://docs.policyclue.com/deploy/operations/)
- **Troubleshooting** -
  [docs.policyclue.com/deploy/troubleshooting/](https://docs.policyclue.com/deploy/troubleshooting/)
