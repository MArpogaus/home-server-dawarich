# home-server-dawarich

Dawarich, a location history, in a rootless Podman pod, with an Ansible role
that deploys it. The phone reports its position; Dawarich stores it in
PostGIS and draws it on a map.

| Container | Job | Default memory ceiling |
|---|---|---|
| dawarich-db | PostgreSQL with PostGIS | 512M |
| dawarich-redis | Job queue and cache | 128M |
| dawarich-app | Rails web app and API on the loopback port | 1G |
| dawarich-sidekiq | Background jobs, such as imports and stats | 1G |

## Configuration

The service follows the configuration interface in
`home-server-template/README.md`, "Configuration interface".

| Variable | Default | Controls |
|---|---|---|
| `dawarich_service_db_password` | required | Password of the database user `dawarich`, in hex: `openssl rand -hex 32` |
| `dawarich_service_secret_key_base` | required | Rails secret that signs the sessions and derives the key for stored API keys; keep it after the first start: `openssl rand -hex 64` |
| `dawarich_service_hostname` | empty | The public hostname; without it the app answers on `127.0.0.1` alone |
| `dawarich_service_config` | `{}` | Dawarich's environment, merged over `dawarich_service_config_defaults` |
| `dawarich_service_memory` | `{}` | Memory ceilings per container |
| `dawarich_service_cpu` | `{}` | CPU quotas per container, such as `{<container>: 50%}` |
| `dawarich_service_*_image` | see `defaults/main.yml` | The images |

The role keeps the database and Redis addresses, the database user and name,
`RAILS_ENV`, `APPLICATION_HOSTS` and `APPLICATION_PROTOCOL`; the config
cannot change them. It must not set `DATABASE_URL`, which the entrypoints read
before all of them. Dawarich's own variables are in its documentation,
"Environment variables".

## Specifics

- The app's entrypoint creates the database and runs the migrations on every
  start. Sidekiq starts after the app.
- The app speaks HTTP; TLS ends at the proxy.
- The first start creates the admin `demo@dawarich.app` with the password
  `safepassword`. Change both through an SSH tunnel to the loopback port
  before the hostname and the proxy site put the app on the internet.
- The database takes `dawarich_service_db_password` at its first start only.
- Reverse geocoding needs a provider, such as `PHOTON_API_HOST`. A provider
  key goes on the admin page `/admin/settings`, which stores it encrypted.
- Two-factor login and mail need secrets in the config, which carries none, so
  they stay off.

## Proxy site and phones

The entry in `bunker_service_sites` of `home-server`:

```yaml
dawarich:
  options:
    REVERSE_PROXY_WS: true
    # Imports upload with PUT, and the map edits with PATCH and DELETE.
    ALLOWED_METHODS: "GET|POST|HEAD|PUT|PATCH|DELETE"
    MAX_CLIENT_SIZE: 1G
    # ModSecurity reads the first 128K of a JSON upload and passes the rest.
    MODSECURITY_SEC_REQUEST_BODY_LIMIT_ACTION: ProcessPartial
    # CRS rule 953100 finds PHP error text in Dawarich's pages and cuts them off.
    CUSTOM_CONF_MODSEC_DAWARICH: SecRuleRemoveById 953100
```

A phone reports to `https://<hostname>/api/v1/<endpoint>?api_key=<key>`; the
key is on the account page, `/users/edit`.

| App | Endpoint | Body |
|---|---|---|
| OwnTracks (HTTP mode) | `owntracks/points` | OwnTracks JSON |
| Traccar Client, or any app with a custom URL | `traccar/points` | Form POST: `lat`, `lon`, `timestamp` (Unix seconds), optional `accuracy`, `altitude`, `speed`, `bearing`, `batt` |

## Role contract

The contract is in `home-server-template/README.md`.

## LLM coding tools

LLM-based coding tools write most of the code and documentation of this
project. The maintainer sets the goals and the design, reviews every change and
is responsible for it. Each change runs on a VM before it reaches a host.

## License

MIT
