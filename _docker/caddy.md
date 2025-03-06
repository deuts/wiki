---
title: Caddy
---
# Caddy
## Basic `compose.yml` file
```yml
services:
  caddy:
    image: caddy:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - ./data:/data
      - ./config:/config
```