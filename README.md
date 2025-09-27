# Brute Force Attack Prevention System in Kali Linux 🔐

**Author:** Lokesh  
**Repository:** Brute-Force-Attack-Prevention  
**Platform:** Kali Linux / Debian-based systems  
**Date:** 2025

---

## 🚀 Project Summary

This project implements a **layered defense system** to protect SSH (and other services) against brute-force and automated login attacks. By combining multiple open-source tools and configuration techniques, the system improves security without significantly affecting legitimate users.

Key components:
- **Fail2Ban** — monitors logs and bans IPs that show malicious patterns (e.g., repeated failed logins).  
- **UFW (Uncomplicated Firewall)** — simple firewall rules and rate-limiting for SSH connections.  
- **TCP Wrappers** — host-based allow/deny controls using `/etc/hosts.allow` and `/etc/hosts.deny`.  
- **Two-Factor Authentication (2FA)** — TOTP (Google Authenticator) to require a second factor at login.  
- **Optional**: username + device-IP verification and real-time email alerts to notify users of suspicious activity.

This repository contains the college project report and the research paper demonstrating the design, implementation, and testing results.

---

## 📂 Repository Contents
```
Brute-Force-Attack-Prevention/
├─ README.md
├─ Network Security PROJECT.pdf
├─ BRUTE FORCE ATTACK PREVENTION SYSTEM IN KALI LINUX.pdf
├─ scripts/
│ ├─ setup_fail2ban.sh
│ ├─ setup_2fa.sh
│ └─ sample_jail.local
├─ docs/
│ └─ diagrams.png
└─ LICENSE
```

**Files included:**  
- `Network Security PROJECT.pdf` — college project report (detailed steps & screenshots).  
- `BRUTE FORCE ATTACK PREVENTION SYSTEM IN KALI LINUX.pdf` — research paper (abstract, literature survey, results).  
- `scripts/` — sample scripts & config snippets to help automate setup (optional).  
- `LICENSE` — project license (MIT by default).

---

## 🛠️ Why this approach?

Brute force attacks are typically automated and rely on repeated login attempts using password lists or simple guesses. No single technique stops every attack, so a multi-layered approach minimizes the chance of success:

- **Fail2Ban** blocks repeating attackers automatically.
- **UFW** reduces the volume of connection attempts and enforces port-level rules.
- **TCP Wrappers** provide an additional host-level allow-list (useful for static environments).
- **2FA** ensures that knowledge of a password alone is not enough to access an account.

---

## ⚙️ Installation & Setup (Full)

> These commands and snippets assume a Debian-based distribution (Kali, Ubuntu). Run as root or prefix `sudo` where needed.

### 1. System update
```bash
sudo apt update && sudo apt upgrade -y
```
2. Install required packages
```bash
sudo apt install fail2ban ufw libpam-google-authenticator -y
```
3. Fail2Ban: basic setup
   1. Copy default configuration to a local file:
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
   2.Edit /etc/fail2ban/jail.local and ensure SSH section is configured:

```ini
Copy code
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 300
```
  3.Restart and enable the service:

```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```
  4.Check Fail2Ban status for sshd:

```bash
sudo fail2ban-client status sshd
```
Notes: Adjust maxretry, bantime, and findtime based on your environment. For production, consider longer ban periods or evolving ban strategies.

4. UFW: firewall and rate-limiting
```bash
# install (if not already installed)
sudo apt install ufw -y

# allow basic ports
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# rate-limit SSH (protects against rapid repeated attempts)
sudo ufw limit ssh

# enable firewall
sudo ufw enable

# verify
sudo ufw status verbose
```
5. TCP Wrappers (optional / host-based allow)
Edit /etc/hosts.allow:

```makefile
sshd: 192.168.1.100, 203.0.113.45
```
Edit /etc/hosts.deny:

```makefile
sshd: ALL
```
Warning: TCP Wrappers only work for services compiled with libwrap support. For many modern setups, rely primarily on UFW/iptables and SSH configuration.

6. Google Authenticator (2FA) setup
   1.Install PAM module (done above with package install).
   2.For each user who needs 2FA, run:

```bash
google-authenticator
```
Follow prompts to set up TOTP, save emergency codes and scan the QR code with an authenticator app.

  3.Update PAM configuration for SSH: edit /etc/pam.d/sshd and add:

```swift
auth required pam_google_authenticator.so
```
   4.Update SSH configuration: edit /etc/ssh/sshd_config

```nginx
ChallengeResponseAuthentication yes
UsePAM yes
PasswordAuthentication yes   # Keep or change depending on policy
```
  5.Restart SSH:

```bash
sudo systemctl restart ssh
```
Important: Test 2FA on a non-critical account first. Keep a separate admin console or IP to avoid locking yourself out.

🧪 How to Test & Validate
Test Fail2Ban
From another machine, attempt SSH with wrong credentials multiple times:

```bash
ssh user@server-ip
# intentionally enter wrong password 3+ times
```
On server, check banned IPs:

```bash
sudo fail2ban-client status sshd
```
Test UFW rate-limiting
Attempt rapid connections and observe UFW blocking/rate-limiting behavior:

```bash
sudo ufw status verbose
```
Test 2FA
Attempt normal SSH login: after password prompt you should be asked for the TOTP code from your authenticator app.

Logs to monitor
/var/log/auth.log — authentication events

/var/log/fail2ban.log — fail2ban actions

🧩 Sample scripts & config snippets
scripts/sample_jail.local

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600   # 1 hour
findtime = 600
```
scripts/setup_fail2ban.sh (example)

```bash
#!/bin/bash
sudo apt update
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
# copy sample_jail.local into /etc/fail2ban/jail.d/custom.conf or append
sudo cp ./scripts/sample_jail.local /etc/fail2ban/jail.d/custom.conf
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```
📌 Best Practices & Tips
Always keep a recovery method: Console access, out-of-band admin, or whitelisted IP to avoid lockout.

Use VPN for remote admin access instead of static IP whitelists when users are mobile.

Tune parameters: maxretry, bantime, findtime should reflect server use and threat level.

Monitor regularly: Keep an eye on logs and banned IP lists. Correlate with other logs if possible.

Backup configs: Keep copies of jail.local, sshd_config and other important files.

Educate users about strong passwords and 2FA usage.

⚠️ Troubleshooting
I locked myself out after enabling 2FA

Access server via console or from a whitelisted IP and remove the pam_google_authenticator line from /etc/pam.d/sshd, restart SSH, then reconfigure carefully.

Fail2Ban not banning IPs

Check logs: /var/log/fail2ban.log and /var/log/auth.log.

Ensure logpath in jail.local points to the correct auth log on your distro.

Confirm the filter (e.g., sshd) matches the log patterns.

UFW rules not applied

Run sudo ufw status verbose to see active rules.

Ensure no other firewall manager (like iptables managed elsewhere) conflicts.

📜 License
This project is provided under the MIT License. See LICENSE file for full terms.

🤝 Contribution & Academic Use
You are welcome to fork this repository and adapt scripts for your environment.

If you reuse code or scripts in academic submissions, please cite this repository and list collaborators/mentors where appropriate.

✉️ Contact
Author: Lokesh
Email: chavalilokesh7@gmail.com
GitHub: https://github.com/lokeshchavali
