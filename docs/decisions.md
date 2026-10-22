# Decisions

| Choice | Why |
| --- | --- |
| **Light containers rather than VMs** | The server has 16 GB of RAM, already well used. A static site needs 512 MB, not a full VM. |
| **A static site, no database** | Fewer moving parts, fewer weaknesses, and rollback is instant. |
| **A reverse proxy with nothing on it** | If the only exposed machine is compromised, there is no application or data to take. |
| **Separate acting from observing** | The agent that changes something is never the one that confirms it worked. |
| **Signed approvals for changes through Observer** | An agent can prepare a change, but only a key held by a human can authorize it. |
| **Staging that mirrors production** | What I accept is exactly what ships. Bugs show up on a URL I can open on my phone, not in production. |
| **A shared memory rather than hand-offs** | A versioned Obsidian vault replaced a home-made tracking service: one source, readable by me and by the agents. |
| **Debian 12 for new containers** | The installed Proxmox version rejected the newer template. Waiting was cheaper than forcing it. |
| **No off-site backup yet** | A known single point of failure, accepted for now. Services that start earning money will move to a hosting provider rather than stay at home. |

## How it got here

1. **School**: first contact with Proxmox, up to my final-year project.
2. **An old gaming PC** becomes a server, rebuilt several times.
3. **Useful**: the lab moves from tests to services that are actually used.
4. **Operated**: monitoring, controlled deployments, documentation, rollback.
5. **AI agents**: Claude Code, Codex and Observer, assisted administration
   within limits.
