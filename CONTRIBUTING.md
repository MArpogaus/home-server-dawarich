# Contributing

The branch flow, the hooks, the releases and the house style are in
`home-server/CONTRIBUTING.md`. The rules a service follows are in
`home-server-template/CONTRIBUTING.md`.

## Tags

A tag names the Dawarich version that the release deploys, such as `1.15.2`. A
later release on the same version adds a counter: `1.15.2-1`, `1.15.2-2`.

## Checks in this repository

- Hooks: the basics, ansible-lint and commitizen.
- Ansible variables are `<role>_*`.
- Renovate updates the container image tags in the role defaults, through the
  preset that `.github/renovate.json` extends.
- `home-server` checks this repository out at `services/dawarich`. Work on it
  there. `home-server/CONTRIBUTING.md` says how a change here reaches the pin.
