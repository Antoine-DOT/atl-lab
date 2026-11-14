# Security model

## Six rules

1. **Closed by default.** The firewall drops anything that is not explicitly
   allowed, between zones as well as from the Internet.
2. **One way in.** Only the reverse proxy faces the Internet, and it holds no
   application or data.
3. **Separate zones.** Public, applications, core, dev, observation and home
   Wi-Fi each have their own network.
4. **Only what is needed.** Each access does one thing. A deploy key can only
   deploy. No agent ships to production without my approval.
5. **Observe without touching.** Monitoring never comes with the right to
   change anything.
6. **Always reversible.** Releases are kept, machines are backed up nightly,
   and releases and network cutovers roll back automatically when a check
   fails.

## How access is scoped

There are no general-purpose shells for automation. Each machine-to-machine
access is a dedicated account whose SSH key is pinned to a single forced
command, often also to a single source address. The command validates its
input and refuses anything else.

| Channel | What it allows | What it cannot do |
| --- | --- | --- |
| Site deployment | Upload a release archive, which is checked and switched in atomically | Open a shell, choose a rollback target, touch anything outside the site |
| Staging deployment | `deploy`, `status`, `remove`, `rollback` for a registered project | Publish ports, mount host paths, run privileged containers, reach the Docker socket |
| Staging controller update | Install an exact commit of the controller, or roll back | Run any other command; the updater cannot update itself |
| Observer queries | A fixed list of read operations, validated on the Observer side | Pick a destination, a command, a path or a credential |
| Changes through Observer | Install a bundle, only with a manifest I signed | Sign anything: the signing key stays on my Mac |
| Malun production | Inspect and operate the app within the runbooks | Release or roll back without my approval |

Firewall changes follow the same pattern: a dedicated tool applies a rule
tied to a ticket, and a human confirms each transaction. It cannot touch NAT.
Riskier cutovers, like moving the public entry point, used a reviewed,
hash-pinned script with a ten-minute automatic rollback, cancelled only after
the checks passed.

## What agents can never touch

Without an explicit decision from me, agents do not touch the hypervisor, the
firewall, DNS, the VPN, the core services, certificates, secret rotation, or
external providers. They never print a secret.

## Prompt injection

The agents on the dev machine read untrusted content (web pages, issues,
dependencies) and have Internet access. So they are treated as executors that
could be manipulated, and their permissions are sized for that:

- settings that would let an agent change lab infrastructure on its own only
  exist on my Mac, where I drive the session in real time;
- on the dev machine, agents get a description of the lab, not broader
  permissions;
- any remaining wide access (an old admin key, for example) is tracked as
  debt to remove, not as a convenience to keep.

## Secrets

- Secrets never go into Git, deployment bundles or the shared memory. Notes
  only say where a secret is stored.
- Staging projects read their secrets from root-owned files on the staging
  host, never from the bundle.
- Staging has its own secrets, separate from production.
