# Architecture

One physical Proxmox server at home, split into isolated zones. Each zone
has its own network, and the only way from one zone to another is through
the firewall, which drops anything not explicitly allowed.

## Zones

| Zone | What lives there | Who can reach it |
| --- | --- | --- |
| **Public** | Reverse proxy, static site, staging platform | The Internet reaches the proxy only. Staging answers VPN clients only. |
| **Applications** | Malun, its PostgreSQL database, Keycloak | The proxy, on a single port, to a single machine |
| **Core** | DNS, time, logs, metrics, SSH bastion | The services everything else depends on |
| **Dev & AI** | Development machine, AI agents, shared memory | Where changes are prepared, tested and documented |
| **Observation** | Observer | Reads every zone, can change nothing |
| **Home Wi-Fi** | Phones, laptops, connected devices | A separate network, never trusted |

Most services run in unprivileged LXC containers. Full VMs are kept for what
needs them (the firewall, Observer, the dev machine).

## The path of a request

1. The browser asks for `antoineleflon.com`.
2. The home router forwards ports 80 and 443 to the firewall, and nothing else.
3. The firewall lets this flow through to the reverse proxy. Every allowed
   flow is a rule written for it.
4. The reverse proxy terminates TLS (Let's Encrypt). It holds no application
   and no data.
5. A small container (1 core, 512 MB) serves static files.

The application zone is only reachable from the proxy, on one port. The
identity provider is only exposed through an allowlist of paths: admin
consoles and the endpoints the app does not need return 404 at the proxy.

DNS is split: internal names resolve to internal addresses from inside the
lab and over the VPN, public names resolve normally.

## Environments

| | Dev | Staging | Production |
| --- | --- | --- | --- |
| **Where** | One isolated working copy per change, on the dev machine | A copy of production, VPN only | The service people use |
| **Who** | AI agents write code and run tests | I test as a user, on a laptop and an iPhone | Nobody by hand: a script ships the approved version |
| **Data** | Generated test data | Test data, with its own secrets | Real data, backed up before every release |

Two rules tie them together:

- The version shipped to production is exactly the commit tested in staging.
  Otherwise the release is refused.
- If the final check fails after a release, the previous version comes back
  automatically.

A real case: a release failed its final check right after the switch. It
rolled back on its own, the fix went through staging again, and the release
succeeded the same day.

## How a release ships

The static site is the simplest example:

1. **Commit**: every version maps to a Git commit.
2. **Build**: clean install, lint, build. Refused if anything is uncommitted.
3. **Release**: one folder per commit, never modified afterwards.
4. **Upload**: the deploy key can only push an archive. It opens no shell: the
   receiving account runs a single forced command that checks the archive and
   installs it.
5. **Switch**: a `current` link moves to the new folder in one atomic step.

Rollback means pointing `current` back to the previous folder.

Staging works the same way for containerized apps, with a health check before
any traffic moves. See
[atl-preview-controller](https://github.com/Antoine-DOT/atl-preview-controller).

## Monitoring and backups

- Prometheus and Grafana for metrics, a central log server, and Observer for
  state checks (see [AI agents](ai-agents.md)).
- Every machine is backed up nightly to a dedicated disk.
- Releases are kept on disk, so rolling back never depends on rebuilding.
