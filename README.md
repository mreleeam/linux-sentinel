# Sentinel

**v1.0**

A small Linux monitoring agent with a web dashboard.

The agent is a C program that watches CPU, memory, swap, disk, network, and
processes, and writes what it sees to a JSON file. The dashboard is a Django
app that reads that file and shows it in a browser. Both run as systemd
services and start on boot.

I built this as a learning project, so the design favors simplicity over
features. No database for metrics, no external services, no agents to install
on other machines. Just one binary, one file, one dashboard.


<img width="1317" height="915" alt="Sentinel" src="https://github.com/user-attachments/assets/9e4b2cf3-a922-4cfb-a254-b6cf6efae12e" />


---

## How it works

```
┌────────────────────┐        ┌──────────────────────┐        ┌────────────────────┐
│   sentinel-agent   │ writes │  /var/lib/sentinel/  │ reads  │  Django dashboard  │
│   (C, systemd)     │ ─────► │     metrics.json     │ ─────► │(your-host-ip:8000) │
└────────────────────┘        └──────────────────────┘        └────────────────────┘
```

---

## Dependencies

Install commands per distribution:

```bash
# Debian / Ubuntu
sudo apt install build-essential cmake python3 python3-venv python3-pip ca-certificates

# Fedora / RHEL / CentOS
sudo dnf install gcc make cmake python3 python3-pip ca-certificates

# Arch
sudo pacman -S base-devel cmake python python-pip ca-certificates

# openSUSE
sudo zypper install gcc make cmake python3 python3-pip ca-certificates
```
---

## Installation
```bash
git clone https://github.com/mreleeam/linux-sentinel.git
cd linux-sentinel
./install.sh
```
---

## Using it

Open the dashboard:

```
http://localhost:8000
```
From another machine:

```
http://<your-host-ip>:8000
```
Log in with the credentials you entered during installation.

To check that both services are running:

```bash
systemctl status sentinel-agent.service sentinel-web.service
```

### Firewall

The installer doesn't touch your firewall. If you want to reach the dashboard
from another machine, open port `8000` yourself:

```bash
sudo ufw allow 8000/tcp
```

---

## Configuration

### The agent

The agent's configuration lives at `/etc/sentinel/config.ini`. It's copied
from `agent/config.ini` on the first install.

The agent's working directory under systemd is `/var/lib/sentinel/`, which is
also where it writes its state and metrics files. That directory is created
by the installer.

### The dashboard

Django reads its settings from `/etc/sentinel/web.env`, which contains:

- `DJANGO_SECRET_KEY`, generated on first install and kept on reinstall.
- `DJANGO_DEBUG`, always `False`.
- `DJANGO_ALLOWED_HOSTS`, a comma-separated list.

If you need to change `ALLOWED_HOSTS`, edit this file and restart the web
service:

```bash
sudo systemctl restart sentinel-web.service
```

### Your login

The username is stored at `/etc/sentinel/dashboard-user`. The password is
hashed in `web/db.sqlite3`. To change either, re-run `./install.sh` and enter
new values.

---

## Uninstalling

```bash
./install.sh -r
```
