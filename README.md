# Server Setup Documentation

> Polished README for the `server-setup-doc` repository.

**Live demo / source:** [https://ronakmer.github.io/server-setup-doc/](https://ronakmer.github.io/server-setup-doc/)

---

## Overview

This repository provides a concise, practical guide to provisioning a Linux server and deploying a Django project using Docker, Nginx and Certbot. It covers essential steps from initial package updates and user setup to SSH hardening, firewall configuration, Docker and Django deployment, and obtaining SSL certificates.

## Quick Links

* Update & Upgrade
* User Setup
* SSH Configuration
* Firewall (UFW)
* Timezone (Asia/Kolkata)
* Security Hardening
* Nginx Setup
* Docker Setup
* Django Project (Dockerized)
* SSL Certificate (Certbot)

---

## Prerequisites

* An Ubuntu server (or Debian-based) with sudo access
* Domain name pointed to your server
* Local machine with SSH key for deployment
* Git installed locally

---

## Quick Start (recommended)

1. Update and upgrade the server packages:

```bash
sudo apt update && sudo apt upgrade -y
```

2. Create a deploy user and configure SSH keys (replace `ubuntu` and `your_email@example.com` as needed).

3. Install Docker and add the deploy user to the `docker` group.

4. Clone your project to `/opt/_code` (or any path) and use the provided `build.sh` to build and run the services via `docker compose`.

5. Configure Nginx with the included site template and obtain an SSL certificate with Certbot.

---

## Configuration placeholders you must replace

* `GIT_REPO_URL` — URL of the Git repository to deploy
* `DJANGO_PROJECT_NAME` — Django project Python package name (used for `settings.py` and WSGI entry)
* `VERSION` — Python base image tag for Dockerfile (e.g. `3.11`)
* `domain_name.in` — your domain (used in Nginx server block and Certbot)

---

## Example files (templates)

### build.sh

```bash
#!/usr/bin/env bash
set -e

rm -rf _code
git clone -b master GIT_REPO_URL _code
sed -i "s/DEBUG = True/DEBUG = False/" _code/DJANGO_PROJECT_NAME/settings.py
docker compose up --build -d
```

Make executable: `chmod +x build.sh`

### Dockerfile (example)

```dockerfile
FROM python:VERSION
WORKDIR /app
ENV PYTHONUNBUFFERED 1
COPY _code/requirements.txt .
RUN pip install -r requirements.txt
COPY _code/ /app
CMD gunicorn --bind 0.0.0.0:8000 DJANGO_PROJECT_NAME.wsgi:application
```

### docker-compose.yml (example)

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "127.0.0.1:10001:8000"
    volumes:
      - ./media:/app/media
      - ./db.sqlite3:/app/db.sqlite3
    restart: always
```

### Nginx site template (example)

```nginx
server {
    server_name domain_name.in;

    location /media/ {
        alias /app/media/;
    }

    location / {
        proxy_pass http://127.0.0.1:10001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Place the config into `/etc/nginx/sites-available/domain_name.conf` and symlink it to `/etc/nginx/sites-enabled/`.

---

## SSL: Get certificate via Certbot

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d domain_name.in
```

---

## Recommended hardening steps

* Disable root login and password authentication for SSH (use key-based auth).
* Configure `ufw` to limit SSH and allow only necessary ports (80, 443).
* Set server timezone: `sudo timedatectl set-timezone Asia/Kolkata`.
* Keep packages updated and schedule security updates.

---

## Troubleshooting

* If `docker compose up` fails, check `/var/log/syslog`, `docker logs` for the service, and verify volumes/permissions.
* Nginx `502` errors: ensure Gunicorn is running and reachable on the proxied port.
* Certbot renewal issues: test with `sudo certbot renew --dry-run`.

---

## Contributing

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit changes and open a PR

---

## License

MIT

---

## Contact

If you want changes or a more opinionated deployment (systemd units, Ansible playbook, or multi-service Docker stacks), open an issue or contact me.

*Generated from the live demo server docs.*
