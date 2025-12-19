Gaming Laptop rtx 3060, 16gb ram, 500gb ssd

username: tokeoadm
Steps Ran:
Installed ubuntu 24.0.1
apt install &% apt upgrade
apt install -y openssh-server
Create ssh-keygen:
ssh-kygen -t "     " -C "project-tokeo3"

Update and Harden SSH
Update sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowUsers tokeoadm
Firewall Update
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable

Update the system (baseline hygiene) - AGAIN
sudo apt update && sudo apt upgrade -y
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades

# ✅ Phase 0 Status — VERIFIED

### 🔐 Remote Access

- SSH key-based auth: **enabled**
    
- Password SSH: **disabled**
    
- Root login: **disabled**
    
- SSH agent caching key: **working**
    

### 🏷️ Identity

- Hostname: `tokeo-node1`
    
- User: `tokeoadm`
    
- Node role: **Tokeo3 Core Node (Phase 0)**
    

### 🧱 Firewall

- UFW: **active**
    
- Default policy: deny inbound / allow outbound
    
- SSH (22/tcp): allowed (IPv4 + IPv6)
    
- Logging: enabled (low)
    

## Security Framework Alignment

- NIST CSF PR.AC-1 — Identity and credential management
    
- NIST CSF PR.PT-4 — Secure communications and control networks




Further Hardening - Brute Force Protection
install fail2ban 
	Edit fail2ban
	sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
	update sshd portion
	[sshd]
	enabled = true
	port = ssh
	logpath = %(sshd_log)s
	backend = systemd
	maxretry = 5
	findtime = 10m
	bantime = 1h
## Intrusion Prevention (Fail2ban)
- Installed and enabled Fail2ban
- SSH jail configured using systemd backend
- Policy:
  - Max retries: 5
  - Find time: 10 minutes
  - Ban time: 1 hour
- Protects against SSH brute-force attacks

### Verification
- fail2ban-client status shows active sshd jail
- **NIST CSF PR.PT-4** (protective technology)
    
- **NIST CSF DE.CM-1** (continuous monitoring)


SYSTEM AND COMMAND AUDITING - AUDITD 
sudo apt install -y auditd audispd-plugins
sudo systemctl enable auditd
- DO NOT EDIT audit.rules directly
- instead make a rules file
-sudo nano /etc/audit/rules.d/tokeo3-baseline.rules
## System Auditing (auditd)
- auditd installed and enabled at boot
- Custom baseline rules applied via /etc/audit/rules.d
- Audits:
  - Identity and authentication files
  - sudoers configuration
  - SSH daemon configuration
  - System time changes
  - Kernel module loading

### Verification
- auditctl confirms rules loaded
- ausearch returns sudo events

Further hardening ------ Time Syncing and Entropy
Install chrony
sudo apt update
sudo apt install -y chrony
sudo systemctl enable --now chrony
Check chrony
chronyc tracking
chronyc sources -v
	You want to see sources listed and not “unsynchronised”.
## Time + Entropy Hardening
- Time sync configured with chrony
- Timezone set (America/Phoenix or UTC)
- Verified synchronization using chronyc
- Verified entropy availability via /proc/sys/kernel/random/entropy_avail
## Entropy Assessment
- Kernel entropy availability observed at ~256 during idle state
- System uses modern Linux kernel with non-blocking /dev/urandom
- No entropy starvation observed during SSH or cryptographic operations
- rng-tools not installed at this stage to avoid unnecessary complexity

