# Zabbix on Docker with Jenkins Deployment

This repository contains all files required to deploy **Zabbix** using **Docker Compose** automatically through **Jenkins Pipeline**.

## 📦 Files Included
- `.env` — Environment variables for Docker Compose.
- `docker-compose.yml` — Defines Zabbix services (Postgres, Server, Web, Agent).
- `Jenkinsfile` — Automates cloning and deployment in Jenkins.
- `.gitignore` — Keeps repo clean.

## 🚀 Manual Usage
If you want to deploy manually (without Jenkins):
```bash
docker compose pull
docker compose up -d
