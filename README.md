# Brute Force Attack Prevention System

**Author:** Lokesh

This project demonstrates a layered approach to prevent SSH brute-force attacks on Kali Linux using Fail2Ban, UFW, TCP Wrappers, and Google Authenticator (2FA).

## Files
- `Network Security PROJECT.pdf` — College project report
- `BRUTE FORCE ATTACK PREVENTION SYSTEM IN KALI LINUX.pdf` — Research paper
- `scripts/` — (optional) sample configs and setup scripts

## Quick setup (Debian/Kali)
See project PDFs for full details. Key commands:
```bash
sudo apt update
sudo apt install fail2ban ufw libpam-google-authenticator -y
# configure /etc/fail2ban/jail.local, /etc/hosts.allow, /etc/hosts.deny
# run `google-authenticator` per user and enable PAM for ssh
