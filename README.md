# Ansible NFS Server

> An Ansible role that installs and configures an **NFS server** on Ubuntu — it installs
> `nfs-kernel-server`, creates the exported directory, and renders `/etc/exports` from your
> inventory.

[![Ansible Role](https://img.shields.io/badge/Ansible-role-EE0000.svg?logo=ansible&logoColor=white)](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
[![Platform: Ubuntu 22](https://img.shields.io/badge/platform-Ubuntu%2022-E95420.svg?logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Overview

This role stands up an NFS server on a Debian/Ubuntu host. Given a directory to share and a
list of client addresses, it:

1. Installs the `nfs-kernel-server` package.
2. Creates the export directory (owned by `nobody:nogroup`, mode `0755`).
3. Renders `/etc/exports` from a template, one export line per client.
4. Restarts (and enables) the NFS server whenever any of the above changes, via a handler.

It pairs with the companion [`ansible-nfs-client`](https://github.com/Tatendaz/ansible-nfs-client)
role, which mounts the share on the client side using systemd units.

## Features

- Idempotent install + configuration of `nfs-kernel-server`.
- Export directory created automatically if it doesn't exist.
- Per-client export lines generated from a simple list of IPs/CIDRs.
- Export options `rw,sync,no_subtree_check`.
- Service restart driven by a handler (only restarts when configuration actually changes).

## Requirements

- **Target OS:** Ubuntu 22 (Debian-family; uses the `apt` module and `nfs-kernel-server`).
- **Ansible:** 2.1+ (`min_ansible_version` in `meta/main.yml`).
- **Privileges:** the play must run with `become: yes` (root) to install packages and write
  `/etc/exports`.

## Role variables

Defined in [`defaults/main.yml`](defaults/main.yml) — override them in your inventory, group/host
vars, or playbook:

| Variable | Default | Description |
|---|---|---|
| `folder_to_share` | `/nfs/example` | Absolute path of the directory to export. Created if missing. |
| `client_ip_list` | `["10.128.0.31"]` | List of client IPs/CIDRs allowed to mount the share. One `/etc/exports` line is generated per entry. |

The generated export line for each client looks like:

```
{{ folder_to_share }} {{ client }}(rw,sync,no_subtree_check)
```

(The export options are currently fixed in [`templates/exports.j2`](templates/exports.j2).)

## Example playbook

```yaml
- hosts: nfs_servers
  become: yes
  roles:
    - role: ansible-nfs-server
      vars:
        folder_to_share: "/srv/share"
        client_ip_list:
          - "10.128.0.31"
          - "10.128.0.32"
          - "192.168.1.0/24"
```

Minimal form (using the defaults):

```yaml
- hosts: all
  become: yes
  roles:
    - ansible-nfs-server
```

## Project structure

```
.
├── defaults/main.yml        # folder_to_share, client_ip_list (the role's public API)
├── vars/main.yml            # internal vars (empty)
├── tasks/
│   ├── main.yml             # orchestrates install → create → configuration
│   ├── install.yml          # apt install nfs-kernel-server
│   ├── create.yml           # create the export directory
│   ├── configuration.yml    # render /etc/exports from the template
│   └── restart.yml          # (standalone restart task; main flow uses the handler)
├── handlers/main.yml        # "Restart service" → restart + enable nfs-kernel-server
├── templates/exports.j2     # generates /etc/exports from client_ip_list
├── meta/main.yml            # Galaxy metadata
└── tests/                   # inventory + test.yml for manual runs
```

## Testing

Point [`tests/inventory`](tests/inventory) at a reachable Ubuntu 22 host, then run the test
playbook:

```sh
ansible-playbook -i tests/inventory tests/test.yml
```

Recommended local checks before committing:

```sh
ansible-lint
yamllint .
ansible-playbook --syntax-check -i tests/inventory tests/test.yml
```

## License

[MIT](LICENSE) © Tatenda Zhou.
