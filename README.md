# rhel-hardening

CIS Level 1 Server hardening for RHEL 9 and RHEL 10, using the Ansible Lockdown roles. The group_vars files are the reusable baselines: what each OS turns off, and why.

| OS | Role | CIS benchmark | Baseline | Status |
|---|---|---|---|---|
| RHEL 10 | [RHEL10-CIS](https://github.com/ansible-lockdown/RHEL10-CIS) 1.1.2 | v1.0.1 | `sysconfig/group_vars/cis_rhel10_host.yml` | Run and audited on lab_server |
| RHEL 9 | [RHEL9-CIS](https://github.com/ansible-lockdown/RHEL9-CIS) 2.4.2 | v2.0.0 | `sysconfig/group_vars/cis_rhel9_host.yml` | Same decisions as RHEL 10, not run yet |

## Layout

| Path | What |
|---|---|
| `playbooks/main_playbook.yml` | Runs `rhel<major>_cis` on every host in the `cis` group, then checks the host's services still answer |
| `sysconfig/inventory.yml` | Hosts; put each in `cis_rhel10_host` or `cis_rhel9_host` |
| `sysconfig/group_vars/` | The baselines (one per OS) |
| `sysconfig/host_vars/` | Per-host settings, e.g. `verify_urls` checked after hardening |
| `requirements.yml` | Pinned role versions and collections |

## Run

```bash
ansible-galaxy install -r requirements.yml -p roles
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/main_playbook.yml --limit <host> --check
ansible-playbook playbooks/main_playbook.yml --limit <host>
```

## Baseline decisions

- All Level 2 rules off; target is Level 1.
- IP forwarding left on: Podman publishes ports through it.
- `dnf update` (1.2.2.1) off: patching is a separate change.
- `repo_gpgcheck` (1.2.1.3) off: Red Hat repo metadata is not signed.
- SSH only for `frqadmin` and `sysadmin`; faillock locks after 3 bad passwords for 15 min.
- IPv6 off.
- Role audit off: with the current role releases it stops the run (reported upstream).

## New host

- Partitions come from the kickstart; the roles only set mount options on mounts that exist.
- RHEL 9: `/tmp` needs its own mount (`systemctl enable tmp.mount`).
- RHEL 10 with separate `/home`, `/var`, `/var/tmp`, `/var/log`, `/var/log/audit`: remove the 13 mount-option `false` lines from its baseline.
- The host needs a `frqadmin` or `sysadmin` account, or SSH locks you out.
