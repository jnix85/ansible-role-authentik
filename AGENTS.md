# Project Context: ansible-role-authentik

**Type:** Ansible Role / Infrastructure Automation  
**Target:** Debian 12 (Bookworm), Debian 13 (Trixie), Ubuntu  
**Service:** [authentik](https://goauthentik.io) SSO / Identity Provider via Docker Compose

---

## Overview

Deploys authentik server and worker stack with PostgreSQL backend using Docker Compose.
Optionally configures nginx reverse proxy with TLS termination.
Secrets are generated once on the host and stored in the environment file.

---

## Repository Layout

```
.
├── AGENTS.md                         # AI source of truth
├── CLAUDE.md -> AGENTS.md            # Symlink for Claude Code
├── README.md                         # Human documentation
├── HANDOFF.md                        # Session state & notes
├── ansible.cfg                       # Inventory and SSH settings (wires shared-inventory)
├── requirements.yml                  # Collections (community.docker, etc.)
├── inventory/
│   ├── hosts.yml                     # authentik_servers group
│   └── group_vars/                   # all/ and authentik_servers/
├── playbooks/
│   └── site.yml                      # Main playbook
└── roles/
    └── authentik/                    # Role implementation
```

---

## Common Commands

```bash
# Syntax check
ansible-playbook -i inventory/hosts.yml playbooks/site.yml --syntax-check

# Dry-run
ansible-playbook -i inventory/hosts.yml playbooks/site.yml --check --diff
```
