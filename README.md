# Ubuntu OS & Server Security Labs

![OS](https://img.shields.io/badge/OS-Ubuntu%20Server-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Service](https://img.shields.io/badge/Service-BIND9%20DNS-2C3E50?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Linux%20Server%20Security-27AE60?style=flat-square)

A collection of hands-on Linux system administration and server hardening labs built and verified in **Ubuntu Server** virtual environments. Each lab focuses on practical network service deployment, access control enforcement, log auditing, and CLI-based troubleshooting. This repository is organized by progressively more advanced topics, covering everything from replicated DNS infrastructure to hidden-primary hardening and AppArmor confinement.

> **Work in Progress:** This repository is maintained alongside ongoing coursework. Additional completed labs covering subsequent Linux administration and server security topics are being documented and uploaded incrementally as time permits.

## Lab Modules

| # | Module | Configured Features & Services |
| :--- | :--- | :--- |
| 01 | [BIND9 Primary & Secondary DNS](./01-bind9-primary-secondary-dns) | BIND9 (`named`), Primary/Slave Zone Replication, `dnssec-validation no`, `rndc retransfer`, `named-checkconf` |
| 02 | [DNS Hardening, AppArmor & Query Logging](./02-bind9-hardening-apparmor-logging) | Hidden Primary Architecture, `NS`/`A` Zone Records, `chown` Directory Permissions, AppArmor (`usr.sbin.named`), BIND9 Query Logging |

## Repository Layout

Every lab directory follows a uniform structure for quick review:

```text
|-- 0X-lab-name/
|   |-- configs/        # BIND9 configuration files, zone records, and AppArmor profile rules
|   |-- screenshots/    # Terminal verification outputs and syntax debugging captures
|   \-- README.md       # Lab objectives, configuration breakdown, and test results
```

## Verification & Troubleshooting Approach

All server configurations were validated end-to-end using native Linux diagnostic utilities rather than relying solely on daemon restarts:

* **Configuration Syntax Validation:** `sudo named-checkconf` (catching missing braces, semicolons, and uncommented `forwarders` directives prior to reloading `bind9`)
* **Daemon & Mandatory Access Control Status:** `sudo systemctl status bind9` and `sudo systemctl status apparmor`
* **Zone Synchronization & Client Resolution:** `sudo rndc retransfer student.co.uk` (manual slave zone pull when `notify no;` is configured on Primary) and `nslookup -type=ns` / `nslookup -type=A` queries from an Ubuntu Desktop client
* **Ownership & Audit Log Verification:** `sudo chown bind:bind /var/log/named` and `sudo cat /var/log/named/query.log` to confirm real-time query capture

## How to Reproduce

1. Deploy three **Ubuntu Server** virtual machines on an internal subnet (`192.168.187.50` as Primary DNS; `192.168.187.51` and `192.168.187.52` as Secondary DNS servers) alongside an **Ubuntu Desktop** client VM.
2. Install BIND9 (`sudo apt update && sudo apt install bind9 bind9utils -y`) and copy the files from each module's `configs/` directory into `/etc/bind/` (and `/etc/apparmor.d/usr.sbin.named` for Module 02).
3. Validate syntax with `sudo named-checkconf` and reload `apparmor` and `bind9` via `systemctl`.
