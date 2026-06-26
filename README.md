# ntswell-infra

Ansible repository for bootstrapping infrastructure nodes — Rocky Linux 9 VMs and Proxmox VE bare-metal hosts.

---

## Requirements

Install the required Ansible collections before running any playbook:

```bash
ansible-galaxy collection install community.general ansible.posix
```

---

## Inventory

Edit `inventory/hosts.yaml` to reflect your actual hostnames and IPs before running anything.

```
inventory/
├── hosts.yaml               # Host definitions grouped by rocky_vms / proxmox_hosts
└── group_vars/all/vars.yaml # Shared variables (e.g. timezone)
```

---

## Roles

| Role | Target OS | Description |
|---|---|---|
| `bootstrap_rocky` | Rocky Linux 9 | Common packages, SELinux, SSH, timezone, Docker |
| `bootstrap_proxmox` | Proxmox VE (Debian) | Common packages, lid switch, beep suppression |

---

## Usage

Bootstrap all Rocky Linux VMs:
```bash
ansible-playbook playbooks/bootstrap-rocky.yaml
```

Bootstrap all Proxmox hosts:
```bash
ansible-playbook playbooks/bootstrap-proxmox.yaml
```

Run everything at once:
```bash
ansible-playbook site.yaml
```

Target a specific host:
```bash
ansible-playbook playbooks/bootstrap-rocky.yaml --limit rocky-node-01
```

Dry run (check mode):
```bash
ansible-playbook site.yaml --check
```

## A la carte tasks
### swap
```bash
ansible-playbook site.yaml --tags swap --limit proxmox_hosts
ansible-playbook site.yaml --tags swap --limit pve-node-01
ansible-playbook site.yaml --tags swap --limit pve-node-01 -e "swap_size=16G"
```
