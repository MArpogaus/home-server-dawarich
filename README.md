# home-server-dawarich

Dawarich, a location history, in a rootless Podman pod, with an Ansible role
that deploys it. The phone reports its position; Dawarich stores it in
PostGIS and draws it on a map.

| Container | Job | Default memory ceiling |
|---|---|---|
| dawarich-db | PostgreSQL with PostGIS | 512M |
| dawarich-redis | Job queue and cache | 128M |
| dawarich-app | Rails web app and API on the loopback port | 1G |
| dawarich-sidekiq | Background jobs: imports, stats, reverse geocoding | 1G |

## Configuration

The service follows the configuration interface in
`home-server-template/README.md`, "Configuration interface".

| Variable | Default | Controls |
|---|---|---|
| `dawarich_service_db_password` | required | Password of the database user `dawarich` |
| `dawarich_service_secret_key_base` | required | Rails secret that signs the sessions: `openssl rand -hex 64` |
| `dawarich_service_hostname` | empty | The public hostname; without it the app answers on `127.0.0.1` alone |
| `dawarich_service_config` | `{}` | Dawarich's environment, merged over `dawarich_service_config_defaults` |
| `dawarich_service_memory` | `{}` | Memory ceilings per container |
| `dawarich_service_*_image` | see `defaults/main.yml` | The images |

The role keeps the database and Redis addresses, the database user and name,
`RAILS_ENV`, `APPLICATION_HOSTS` and `APPLICATION_PROTOCOL`; the config
cannot change them. Dawarich's own variables are in its documentation,
"Environment variables".

## Specifics

- The app's entrypoint creates the database and runs the migrations on every
  start. Sidekiq starts after the app.
- The app speaks HTTP; TLS ends at the proxy.
- The first login is `demo@dawarich.app` with the password `safepassword`. Change
  both at once.

## Role contract

The contract is in `home-server-template/README.md`.

## LLM coding tools

LLM-based coding tools write most of the code and documentation of this
project. The maintainer sets the goals and the design, reviews every change and
is responsible for it. Each change runs on a VM before it reaches a host.

## License

MIT
