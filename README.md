# ansible-playbook-security

[![Linter](https://github.com/thbe/ansible-playbook-security/actions/workflows/linter.yml/badge.svg)](https://github.com/thbe/ansible-playbook-security/actions/workflows/linter.yml)

Security auditing and compliance playbooks for RHEL hosts. Each playbook runs a
scanner on the target and collects the results back to the control node under
`results/`, so findings can be reviewed and archived centrally.

This repository is normally consumed as the `playbooks/security` git submodule of
[ansible-main](https://github.com/thbe/ansible-main), but it can also be used
standalone.

## Table of Contents

- [Requirements](#requirements)
- [Playbooks](#playbooks)
- [Results layout](#results-layout)
- [Usage](#usage)
- [License](#license)
- [Author](#author)

## Requirements

- Ansible 2.14+ (core) with the collections:
  - `ansible.posix`
  - `community.general`
- Scanner tooling installed on the targets as required by each playbook:
  `aide`, `lynis`, `rkhunter`, and `openscap-scanner` + `scap-security-guide`
  (for the CIS remediation generator).
- Privilege escalation (`become`) available on the targets (set per task).
- All playbooks target the `all` host group.

## Playbooks

| Playbook                      | Tool          | Description                                                                                   |
| ----------------------------- | ------------- | --------------------------------------------------------------------------------------------- |
| `check_aide.yml`              | AIDE          | Run an AIDE file-integrity check and store the report locally.                                |
| `check_file_permissions.yml`  | `find`        | List world-writable files across the filesystem (excluding known safe entries).               |
| `check_lynis.yml`             | Lynis         | Run a quick Lynis system audit and fetch `lynis.log` + `lynis-report.dat`.                     |
| `check_rkhunter.yml`          | rkhunter      | Run an rkhunter scan and fetch its log.                                                        |
| `create_cis-playbook.yml`     | OpenSCAP      | Generate an Ansible CIS remediation playbook from the SCAP Security Guide (RHEL 8 and RHEL 9). |

## Results layout

Results are written to the control node. Create the target directories (or
symlink them to a central archive) before running:

```text
results/
  aide/          # <hostname>_check.txt
  permissions/   # <hostname>_check.txt
  lynis/         # per-host lynis.log and lynis-report.dat
  rkhunter/      # per-host rkhunter.log
  cis/           # <hostname>_rhelN_cis_remediation_playbook.yml
```

The repository ships `results/.gitkeep`; actual result files are intended to be
kept out of version control.

## Usage

```shell
mkdir -p results/{aide,permissions,lynis,rkhunter,cis}

# Run a Lynis audit across the fleet
ansible-playbook -i inventories/prod/hosts.yml check_lynis.yml

# Generate CIS remediation playbooks for a single host
ansible-playbook -i inventories/prod/hosts.yml create_cis-playbook.yml --limit web01

# File integrity check on the database group
ansible-playbook -i inventories/prod/hosts.yml check_aide.yml --limit dbservers
```

## License

GPL-3.0-only

## Author

Thomas Bendler - [https://www.thbe.org/](https://www.thbe.org/)
