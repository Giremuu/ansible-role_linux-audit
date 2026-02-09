# Linux Security Audit (ANSSI‑inspired)

[![CI](https://github.com/Giremuu/ansible-role_linux-audit/actions/workflows/ci.yml/badge.svg)](https://github.com/Giremuu/ansible-role_linux-audit/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Start:** November 2025
**Version :** N°1

Host‑based **Linux security posture audit** built with **Ansible**.
Audit‑only (no remediation), focused on **hardening, exposure, and hygiene**, inspired by **ANSSI** best practices for **Debian and Ubuntu**

---

## Overview

The audit runs read‑only checks on inventory hosts and produces a **machine‑readable JSON report**.

**No configuration is modified.**

---

## What is checked

### SSH security

* Root login disabled (Prevents direct brute-force attacks against the most privileged account you know)
* Password authentication disabled (SSH keys preferred because brute-force too)
* Protocol 2 only (Protocol 1 is vulnerable and obsolete (OpenSSH use Protocol 2 by default))

### Firewall

* UFW active (INFO if inactive; another firewall may be used)
* firewalld active (systemd) (INFO if inactive; another firewall may be used)
* nftables active (systemd) (INFO if inactive; another firewall may be used)
* **Global firewall status** (OK if at least one is active)

### Users & passwords

* Accounts with empty password field (`/etc/shadow`)
* Password expiration policy (`PASS_MAX_DAYS` vs `max_password_age_days`)
* `sudo` installed and admin group present (`sudo` or `wheel`)
* Root account password locked (For brute-force attacks on the root account and enforces controlled privilege escalation via sudo)

### System updates

* Pending package upgrades
* Automatic security updates (`unattended-upgrades`)
* System uptime vs `max_uptime_days` in defaults/main.yml

### Network & filesystem

* Listening ports inventory
* Telnet disabled
* IPv6 status
* World‑writable files in system directories

### Web servers (Not tested)

* Apache / NGINX detection
* HTTP / HTTPS ports exposed
* HTTPS present when a web service is exposed
* Web server version disclosure (basic hardening)

### Databases (Not tested)

* MariaDB / MySQL / PostgreSQL detection
* Database ports exposed
* PostgreSQL `listen_addresses` scope

---

## Output

A **JSON report** generated on the control machine:

```
/tmp/security_audit_<hostname>_<date>.json
```

Contains:

* Summary (OK / WARN / FAIL / INFO)
* Global score
* Per‑control results
* Evidence and remediation guidande

You can change the path or desactivate this in "roles/linux_security_audit/defaults/main.yml" plus you can create Grafana's dashboard with their reports.

---

## How to run

### Prerequisites

* Target host: **Debian or Ubuntu**, powered on and reachable via a **SSH user**
* Ansible control node on the **same network**
* Valid **Ansible inventory**
* Credentials stored in **Ansible Vault**

### Run the audit

```
ansible-playbook playbooks/audit_linux.yml --ask-vault-pass
```

Tags are available to run specific control groups.

---

## SSH first‑connection issue

One of my common error:

```
[ERROR]: Failed to connect to the host via ssh:
ssh_askpass: exec(/usr/bin/ssh-askpass): No such file or directory
Host key verification failed
```

Fix: connect **once manually** to each host using a SSH key to accept the host key.

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_*
ssh user@host
```

---

## Credentials & scaling

* Per‑host Ansible Vault
* Secrets stored in:

```
host_vars/<hostname>/vault.yml
```

* Each host can have its own `ansible_become_password`

---

## Project structure

* `ansible.cfg` – default configuration
* `hosts.yml` – host definitions
* `host_vars/<hostname>/*.yml` – host‑specific variables with vault's files for our secrets

---

## Roadmap

**V1**

* Core ANSSI‑style controls
* JSON reporting
* CI linting

**V2**

* More checks for sure I think
* Developp audit_profile maybe like bastion for example

**One days maybe**

* Same for Windows ?
* Molecule scenarios maybe ??
* A role to integrate JSON reports to Grafana ???

---

**Audit‑only. No remediation.**

---

## License
MIT - see [LICENSE](LICENSE) file.


## Author

Giremuu - [GitHub Profile](https://github.com/Giremuu)
