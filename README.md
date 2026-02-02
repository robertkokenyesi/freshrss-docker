# Self-Hosted FreshRSS + Full-Text RSS on Oracle Cloud

This repository contains my personal Docker Compose setup for running **FreshRSS** (a lightweight, self-hosted RSS feed aggregator) alongside a **full-text-rss** feed scraper — all on an Oracle Cloud Always Free Tier Ubuntu VM.

The setup uses an existing Docker network and NGINX reverse proxy (from my main home lab) for SSL (via Let's Encrypt) and custom subdomains.

Live demo (personal instance):  
- FreshRSS: https://freshrss.rcomputing.co.uk  
- Full-Text RSS scraper: (subdomain configured similarly)

## Why This Project?

I wanted a fast, private way to read IT/news feeds without relying on browser clutter or paid services. Inspired by Chris Titus Tech's videos:

- [Why We Don’t Browse the Internet Anymore](https://www.youtube.com/watch?v=nxV0CPNeFxY)
- Related walkthrough: https://www.youtube.com/watch?v=HohZBu_hPZ0

I adapted his approach to use **Docker Compose** (easier backups & management) instead of plain `docker run`.

## Features

- FreshRSS running on port 80 internally (exposed via NGINX proxy)
- Full-text feed scraping for sites without RSS
- Shared Docker network (`rcomputing`) for container communication
- Persistent config via volume mapping
- Environment variables for timezone (Europe/London) and user/group (PUID/PGID 1000)
- Temporary direct port exposure (83/82) during initial setup — later removed after proxy config

## docker-compose.yml Overview (FreshRSS)

```yaml
services:
  freshrss:
    image: lscr.io/linuxserver/freshrss:latest
    container_name: freshrss
    restart: unless-stopped
    ports:
      - '83:80'                  # Temporary during setup; commented out after NGINX proxy
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/London
    volumes:
      - ./:/config
networks:
  default:
    external: true
    name: rcomputing


(The full-text-rss compose file is similar — see separate repo or branch if split.)


Setup Steps (Summary)

Create directory: mkdir freshrss && cd freshrss
Create docker-compose.yml (copy from above or this repo)
Allow temporary port: sudo ufw allow 83
Start: docker compose up -d
Configure subdomain in Cloudflare → point to Oracle VM IP
Add location block in NGINX proxy container → forward to freshrss:80
Test → remove direct port mapping & firewall rule
Repeat similar steps for full-text-rss (port 82 initially)

Prerequisites

Oracle Cloud Always Free Ubuntu VM
Existing Docker + Docker Compose
Shared custom network: docker network create rcomputing (or existing)
NGINX reverse proxy with Let's Encrypt
Cloudflare DNS for subdomains

Related Projects

Main home lab / WordPress + NGINX setup: https://github.com/robertkokenyesi?tab=repositories
Personal site: https://www.rcomputing.co.uk
Blog post with screenshots & detailed walkthrough: https://robertkokenyesi.rcomputing.co.uk/projects/set-up-my-self-hosted-freshrss-server/
