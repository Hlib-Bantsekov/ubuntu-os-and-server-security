# Ubuntu OS & Server Security Labs

![OS](https://img.shields.io/badge/OS-Ubuntu%20Server-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Service](https://img.shields.io/badge/Service-BIND9%20DNS-2C3E50?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Linux%20Server%20Security-27AE60?style=flat-square)

A collection of hands-on Linux system administration and server hardening labs built and verified in **Ubuntu Server** virtual environments. Each lab focuses on practical network service deployment, access control enforcement, log auditing, and packet-level troubleshooting. This repository is organized by progressively more advanced topics, covering replicated DNS infrastructure, hidden-primary hardening with AppArmor, and Wireshark traffic inspection.

> **Work in Progress:** This repository is maintained alongside ongoing coursework. Additional completed labs covering subsequent Linux administration and server security topics are being documented and uploaded incrementally as time permits.

## Lab Environment & Topology

All lab nodes operate inside an isolated internal virtual network (`192.168.187.0/24`):

| Node Role | Operating System | Static IP | Function & Traffic Flow |
| :--- | :--- | :--- | :--- |
| **Hidden Primary DNS** | Ubuntu Server | `192.168.187.50` | Authoritative master for `student.co.uk`; excluded from public `NS` records, confined by AppArmor, and logging queries to `/var/log/named/query.log`. |
| **Secondary DNS (`ns1`)** | Ubuntu Server | `192.168.187.51` | Client-facing slave server pulling zone data from `.50` via `rndc retransfer` / AXFR. |
| **Secondary DNS (`ns2`)** | Ubuntu Server | `192.168.187.52` | Redundant client-facing slave server pulling zone data from `.50` via `rndc retransfer` / AXFR. |
| **Admin / Test Client** | Ubuntu Desktop | `192.168.187.128` | Queries `ns1` and `ns2` over `UDP/53` using `nslookup` and captures DNS traffic in Wireshark. |

## Lab Modules

| # | Module | Configured Features & Services |
| :--- | :--- | :--- |
| 01 | [BIND9 Primary & Secondary DNS](./01-bind9-primary-secondary-dns) | BIND9 (`named`), Primary/Slave Zone Replication, `dnssec-validation no`, `rndc retransfer`, `named-checkconf` |
| 02 | [DNS Hardening, AppArmor & Query Logging](./02-bind9-hardening-apparmor-logging) | Hidden Primary Architecture, `NS`/`A` Zone Records, `chown` Directory Permissions, AppArmor (`usr.sbin.named`), BIND9 Query Logging |
| 03 | [Wireshark DNS Traffic Analysis](./03-wireshark-dns-traffic-analysis) | Wireshark Packet Capture (`.pcapng`), `dns` Display Filter, UDP Port 53 Dissection, Plaintext Stream Auditing |

## Repository Layout

Every lab directory follows a uniform structure for quick review:

```text
|-- 0X-lab-name/
|   |-- configs/        # BIND9 configuration files, zone records, and AppArmor rules (Modules 01–02)
|   |-- captures/       # Wireshark packet capture files in .pcapng format (Module 03)
|   |-- screenshots/    # Terminal verification outputs, syntax checks, and Wireshark captures
|   \-- README.md       # Lab objectives, configuration breakdown, and verification results
```

## Verification & Troubleshooting Approach

Every configuration change and network flow was validated end-to-end using native Linux diagnostic utilities and packet inspection:

* **Configuration Syntax Validation:** `sudo named-checkconf` to catch missing braces, semicolons, or improperly commented `forwarders` directives before reloading `bind9`.
* **Daemon & AppArmor Policy Status:** `sudo systemctl status bind9` and `sudo systemctl status apparmor` to verify active service states and mandatory access control profile reloads.
* **Zone Synchronization & Client Resolution:** `sudo rndc retransfer student.co.uk` to force manual slave zone transfers (when `notify no;` is configured on the primary node), followed by `nslookup -type=ns` and `nslookup -type=A` queries from Ubuntu Desktop.
* **Directory Permissions & Audit Logs:** `sudo chown bind:bind /var/log/named` to enforce least-privilege ownership and `sudo tail -n 15 /var/log/named/query.log` to confirm real-time DNS query recording.
* **Packet-Level Traffic Auditing:** Wireshark with the `dns` display filter on `UDP/53` to dissect `A`, `CNAME`, and `NS` responses and inspect unencrypted DNS payloads via `Follow -> UDP Stream`.

## How to Reproduce

1. **Provision Virtual Machines:** Deploy three **Ubuntu Server** VMs on a shared internal subnet (`192.168.187.50` as Hidden Primary DNS; `192.168.187.51` and `192.168.187.52` as Secondary DNS nodes) alongside an **Ubuntu Desktop** client VM.
2. **Deploy BIND9 & Hardening Rules:** Install BIND9 (`sudo apt update && sudo apt install bind9 bind9utils -y`), copy the configuration files from `01-.../configs/` and `02-.../configs/` into `/etc/bind/` and `/etc/apparmor.d/usr.sbin.named`, and create `/var/log/named` owned by `bind:bind`.
3. **Validate & Synchronize Zones:** Verify syntax with `sudo named-checkconf`, reload `apparmor` and `bind9` via `systemctl`, and pull the zone onto the secondary servers using `sudo rndc retransfer student.co.uk`.
4. **Test Resolution & Capture Packets:** Run `nslookup` queries from Ubuntu Desktop, verify logged entries in `/var/log/named/query.log`, and capture live `UDP/53` traffic in Wireshark (`sudo wireshark`) using the `dns` display filter.
