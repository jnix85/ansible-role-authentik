# Handoff — ansible-role-authentik

Status as of 2026-09-10. This role is written but **not yet run against real
infrastructure** — no VM/LXC has been provisioned or targeted, no play has
executed, and this directory is not (yet) a git repository.

## What exists

A standalone Ansible role deploying [authentik](https://goauthentik.io)
(self-hosted SSO / identity provider) via Docker Compose, modeled directly
on the existing `ansible-role-infisical` layout in this same
`~/src/infra/ansible/` tree (same class of app: Compose + Postgres, optional
nginx/TLS, secrets generated once on the host).

```
.
├── ansible.cfg
├── requirements.yml            # community.docker, community.general
├── inventory/
│   ├── hosts.yml                 # authentik_servers group — placeholder IP 10.1.0.52
│   └── group_vars/
│       ├── all/main.yml          # domain, vm_fqdn
│       └── authentik_servers/main.yml   # version, dir, public fqdn, acme_cert_*
├── playbooks/site.yml          # references common/docker/acme_cert + authentik
├── roles/authentik/
│   ├── defaults/main.yml
│   ├── handlers/main.yml       # nginx reload
│   ├── meta/main.yml
│   ├── tasks/{main,deploy,nginx}.yml
│   └── templates/{env.j2, docker-compose.yml.j2, nginx-site.conf.j2}
├── README.md
└── LICENSE / .gitignore
```

Compose stack deployed: `postgresql` + `server` + `worker` only — confirmed
against the current official `docs.goauthentik.io/compose.yml` that Redis is
no longer required as of authentik 2026.8.1 (the version pinned in
defaults). `PG_PASS` and `AUTHENTIK_SECRET_KEY` are generated once on first
run into `/opt/authentik/.env` (root-only, `0600`) and never regenerated.

## What's NOT done / open items

1. **Not a git repo yet.** `git status` fails here — no `.git` directory.
   Decide whether to `git init` this directory standalone or fold it into
   an existing monorepo before committing anything.
2. **Never run.** No `ansible-playbook` execution has happened — not even a
   syntax-check against a real inventory. `ansible`/`ansible-playbook` are
   not installed in this shell environment, so only YAML-parse and
   hand-review validation were done (see prior session).
3. **Sibling roles not vendored.** `playbooks/site.yml` calls `common`,
   `docker`, and `acme_cert` — roles that live in the `proxmox-ansible-roles`
   monorepo and were *not* copied in here, matching how `ansible-role-infisical`
   also leaves them out. You need to either point `roles_path` at that
   monorepo, `ansible-galaxy role install` them, or pre-bake the equivalent
   (Docker Engine installed, base packages, a TLS cert at the paths
   `roles/authentik` expects) before this play will run end-to-end.
4. **`authentik_manage_nginx` defaults to `false`.** TLS termination via the
   role's nginx task is untested; flip it on and populate
   `acme_cert_fullchain_file` / `acme_cert_key_file` to exercise it.
5. **Placeholder inventory values.** `inventory/hosts.yml` has a made-up
   host (`authentik1` @ `10.1.0.52`) and `authentik_public_fqdn` in
   `group_vars/authentik_servers/main.yml` is not a real domain — both need
   real values before running against production.
6. **Stray local files in this directory** (not part of the role, ignore or
   clean up before committing): `proxmox-jobs.sqlite3`,
   `proxmox-jobs.sqlite3-shm`, `proxmox-jobs.sqlite3-wal` — these look like
   local state from the `proxmox-mcp-plus` MCP tool, not role artifacts.

## Related but separate: shared PostgreSQL cluster (not started)

User has asked (2026-09-07) for a **shared PostgreSQL cluster**: LXC
containers running Postgres across a few Proxmox nodes, meant to eventually
replace the pattern where each app role (authentik, infisical, netbox,
keycloak) runs its own bundled Postgres container. Explicitly deferred —
"we'll get to that later." Not designed, not scaffolded. When picked up:

- Likely a new standalone role/dir (e.g. `ansible-role-postgresql-cluster`),
  following the same layout convention as `ansible-role-infisical` /
  `ansible-role-authentik`.
- Will need decisions this session did NOT make: which Proxmox nodes, how
  many LXCs, replication topology (single primary + replicas? Patroni/etcd
  for HA?), and whether existing roles like this one get switched from
  bundled Postgres to pointing at the shared cluster (would touch
  `roles/authentik/templates/docker-compose.yml.j2` and `defaults/main.yml`
  here specifically — remove the `postgresql` service, add
  `authentik_pg_host` pointing off-box).

## Suggested next steps (in order)

1. Fill in real inventory values (host IP, domain/FQDN).
2. Decide how to satisfy the `common`/`docker`/`acme_cert` role dependency.
3. Install `ansible` + collections from `requirements.yml` somewhere
   runnable, then `ansible-playbook playbooks/site.yml --syntax-check` at
   minimum, ideally a real run against a test VM/LXC.
4. `git init` (or fold into the right monorepo) and commit.
5. Only after that's stable — start the shared-Postgres-cluster design.
