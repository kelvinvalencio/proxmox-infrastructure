# proxmox-infrastructure

Bootstrap, manage proxmox host and the VMs.

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
| `bootstrap_rocky` | Rocky Linux 9 | Common packages, security patching, kernel hardening, SELinux, SSH, timezone, Docker |
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

## Security hardening (Rocky)

The `bootstrap_rocky` role keeps hosts patched against the growing stream of
kernel CVEs and shrinks the kernel attack surface. All of it is toggle-able via
variables in `roles/bootstrap_rocky/defaults/main.yaml`.

| Variable | Default | Purpose |
|---|---|---|
| `security_auto_updates` | `true` | Install and enable `dnf-automatic` (unattended patching) |
| `security_upgrade_type` | `security` | `security` or `default` (full upgrade) |
| `security_auto_reboot` | `never` | `never` or `when-needed` (reboot to activate new kernels) |
| `security_baseline_upgrade` | `true` | Apply a security upgrade during bootstrap |
| `kernel_hardening` | `true` | Deploy hardening sysctls + blacklist rarely-used modules |
| `kernel_blacklist_modules` | see defaults | Modules blocked from loading (uncommon FS/net protocols) |

Notes:
- Kernel CVE fixes only take effect after a reboot. `security_auto_reboot` is
  `never` by default so reboots stay in your control — reboot in a maintenance
  window, or set it to `when-needed`.
- Module blacklisting fully applies after the next reboot.
- SELinux is intentionally left disabled, as before.

## A la carte tasks
### swap
```bash
ansible-playbook site.yaml --tags swap --limit proxmox_hosts
ansible-playbook site.yaml --tags swap --limit pve-node-01
ansible-playbook site.yaml --tags swap --limit pve-node-01 -e "swap_size=16G"
```

### battery
```bash
ansible-playbook site.yaml --tags battery --limit proxmox_hosts
ansible-playbook site.yaml --tags battery --limit pve-node-01
ansible-playbook site.yaml --tags battery --limit pve-node-01 -e "charging_limit_low=50 charging_limit_high=80"
```

### security
Apply security patching and kernel hardening to already-running Rocky VMs
without re-running the full bootstrap:
```bash
ansible-playbook site.yaml --tags security --limit rocky_vms
ansible-playbook site.yaml --tags security --limit rocky-node-01
```
Run just one half with the granular tags:
```bash
ansible-playbook site.yaml --tags patching --limit rocky_vms
ansible-playbook site.yaml --tags hardening --limit rocky_vms
```
