# Secure Debian Web Platform

## Goal
Deploy a mini-infrastructure: a core server + a web server + an admin workstation. Security, web (vhosts/HTTPS/reverse proxy), centralized logs, automated backups.

## Architecture (3 VMs)
- **srv-core**: security baseline + centralized logging + backups  
- **srv-web**: Nginx (vhosts, HTTPS, reverse proxy)  
- **cli-admin**: administration / testing workstation  

## Addressing plan (Host-Only 192.168.56.0/24)
- **srv-core**: 192.168.56.10  
- **srv-web**: 192.168.56.11  
- **cli-admin**: 192.168.56.12  

## Repository contents
- **configs/**: configuration files (ssh, nftables, nginx, rsyslog…)  
- **configs/systemd/**: systemd units (.service/.timer)  
- **scripts/**: bash scripts  
- **docs/proofs/**: proofs (command outputs)  

## Backups (srv-core)
- Script: `scripts/backup-configs.srv-core.sh`  
- systemd: `configs/systemd/backup-configs.service` + `.timer`  
- Restore test: extract `etc/nftables.conf` from the latest archive to `/tmp/restore-test/` (see `docs/proofs/backup-restore-*`)  

## VMs
- **cli-admin**: 192.168.56.12 (admin workstation / tests)  
- **srv-core**: 192.168.56.10 (internal services + logs + backups)  
- **srv-web**: 192.168.56.11 (Nginx front-end HTTPS + reverse proxy)  

## Implemented security (summary)
- **SSH**: keys only (`PasswordAuthentication no`) + fail2ban sshd  
- **Firewall**: nftables default drop policy + minimal port exposure  
- **Web**: self-signed HTTPS + HTTP→HTTPS redirect + security headers  
- **Reverse proxy**: `app.site1.lab` → `srv-core:8080` (internal)  
- **/admin**: BasicAuth + fail2ban nginx-http-auth (automatic ban)  

## Quick tests (proofs in docs/proofs)
- HTTP→HTTPS: `curl -I http://site1.lab`  
- HTTPS OK: `curl -k -I https://site1.lab`  
- Reverse proxy: `curl -k https://app.site1.lab/`  
- /admin without creds: `curl -k -I https://site1.lab/admin/` (401)  
- /admin with creds: `curl -k -u demo:*** https://site1.lab/admin/`  
- Fail2ban ban: after several bad logins → IP banned (see proof)  
