# UBUNTU-SRV — Web Application Target

## Overview

Ubuntu Server hosting DVWA (Damn Vulnerable Web Application) via Docker.
Used as the target for web attack scenarios (SQL injection, brute force, XSS).
Status: Complete
IP: 192.168.10.30 | Segment: LAN (vboxnet0)

---

## VM Configuration

| Setting | Value |
| --- | --- |
| OS | Ubuntu Server 24.04 LTS |
| RAM | 2GB |
| CPU | 1 core |
| Storage | 30GB |
| Adapter 1 | Host-only vboxnet0 |

---

## Step 1 — Install Ubuntu Server

Booted from Ubuntu Server 24.04 ISO. Same install process as SPLUNK VM.
- Hostname: ubuntu-srv
- Username: flamy
- OpenSSH server: installed during setup

---

## Step 2 — Set static IP

Used tee to avoid YAML indentation issues with nano:

```bash
sudo tee /etc/netplan/00-installer-config.yaml << 'YAML'
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.10.30/24
      nameservers:
        addresses:
          - 192.168.10.10
YAML
sudo chmod 600 /etc/netplan/00-installer-config.yaml
sudo netplan apply
```

DNS points to DC01 (192.168.10.10) for flamurlab.local resolution.
Result: enp0s3 UP with inet 192.168.10.30/24 confirmed.

---

## Step 3 — Enable SSH

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Connected from Ubuntu host: `ssh flamy@192.168.10.30`
All remaining steps performed via SSH.

---

## Step 4 — Install Docker

Temporarily added NAT as Adapter 2 for internet access:

```bash
sudo ip link set enp0s8 up
sudo networkctl up enp0s8
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker flamy
```

Logged out and back in for group membership. Removed NAT adapter after install.
Result: Docker version 29.1.3

---

## Step 5 — Deploy DVWA via docker-compose

DVWA requires a separate MariaDB container — single container image does not
include MySQL. Created docker-compose.yml:

```bash
mkdir ~/dvwa
cat > ~/dvwa/docker-compose.yml << 'YAML'
services:
  dvwa:
    image: ghcr.io/digininja/dvwa:latest
    environment:
      - DB_SERVER=db
    depends_on:
      - db
    ports:
      - "80:80"
    restart: unless-stopped

  db:
    image: mariadb:10.11
    environment:
      - MYSQL_ROOT_PASSWORD=dvwa
      - MYSQL_DATABASE=dvwa
      - MYSQL_USER=dvwa
      - MYSQL_PASSWORD=dvwa
    restart: unless-stopped
YAML
cd ~/dvwa
sudo docker-compose up -d
```

DVWA's config uses password `p@ssw0rd` by default (not `dvwa`). Fixed with:

```bash
sudo docker-compose exec db mariadb -u root -pdvwa -e "DROP USER IF EXISTS 'dvwa'@'%'; CREATE USER 'dvwa'@'%' IDENTIFIED BY 'p@ssw0rd'; GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'%'; FLUSH PRIVILEGES;"
```

Result: Both containers running. DVWA accessible at http://192.168.10.30

---

## Step 6 — Configure DVWA

Opened http://192.168.10.30/setup.php in browser.
Clicked Create / Reset Database.
Logged in: admin / password
Set security level to Low via DVWA Security page.

---

## Step 7 — Configure rsyslog forwarding to Splunk

```bash
sudo apt install -y rsyslog
sudo tee /etc/rsyslog.d/50-splunk.conf << 'CONF'
*.* @192.168.10.50:514
CONF
sudo systemctl restart rsyslog
```

Note: Splunk receives syslog on 192.168.10.50 (its LAN interface on vboxnet0),
not 192.168.30.10 (management interface).

---

## Notes

- DVWA docker-compose starts automatically on VM boot (restart: unless-stopped)
- To restart DVWA: `cd ~/dvwa && sudo docker-compose restart`
- To check container status: `sudo docker-compose ps`
- MariaDB credentials: root/dvwa, app user: dvwa/p@ssw0rd
