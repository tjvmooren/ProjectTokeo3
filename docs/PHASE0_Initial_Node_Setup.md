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
### Time Synchronization Verification
- Chrony synchronized to stratum-2 NTP sources
- System clock offset measured in microseconds
- RMS offset ~1 ms
- Leap status normal
- Multiple upstream sources available for redundancy
Chrony was deployed and validated as the system time synchronization service.
The node is synchronized to stratum-2 NTP sources with microsecond-level drift and millisecond-level RMS offset. Multiple upstream sources are available, and leap status is normal, ensuring reliable timestamps for logs, cryptographic operations, and incident timelines.

Last step? Add in Snapshot
dpkg --get-selections > ~/tokeo3_phase0_packages.list
chekkkk
wc -l ~/tokeo3_phase0_packages.list
create the snapshot directory 
mkdir -p ~/tokeo3_snapshots/phase0
Copy critical configs:
`sudo cp -a \ /etc/ssh \ /etc/ufw \ /etc/fail2ban \ /etc/audit \ /etc/chrony \ /etc/systemd \ /etc/hostname \ /etc/hosts \ ~/tokeo3_snapshots/phase0/`
Fix ownership so your user can read them:
`sudo chown -R tokeoadm:tokeoadm ~/tokeo3_snapshots`
Verify:
`ls ~/tokeo3_snapshots/phase0`
Capture system metadata (lightweight but valuable)
`uname -a > ~/tokeo3_snapshots/phase0/system_info.txt lsb_release -a >> ~/tokeo3_snapshots/phase0/system_info.txt timedatectl >> ~/tokeo3_snapshots/phase0/system_info.txt`
rollback .tar.gz file
tar czvf ~/tokeo3_phase0_snapshot.tar.gz -C ~/tokeo3_snapshots phase0
Rollback concept :
1. Fresh Ubuntu install (same LTS)
2. Restore packages:
3. `dpkg --set-selections < tokeo3_phase0_packages.list apt-get dselect-upgrade`
4. Restore configs from snapshot archive
5. Restart services (SSH, UFW, fail2ban, auditd, chrony)
6. Verify access and logs
# Phase 0 — Snapshot & Rollback Strategy

## Objective
Establish a recoverable baseline for Project Tokeo3 prior to Phase 1 changes.

## Snapshot Method
- Filesystem-based snapshot (bare-metal friendly)
- Package manifest capture
- Critical configuration backups
- Manual restore playbook

## Artifacts Captured
- Installed package list
- SSH, UFW, Fail2ban, auditd, Chrony configs
- System metadata (kernel, OS, time config)

## Rollback Concept
1. Reinstall base OS (same Ubuntu LTS)
2. Restore package selections
3. Restore configuration files
4. Restart and verify services

