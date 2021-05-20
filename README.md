# Unofficial Chia-Dashboard-UI Docker container

https://github.com/felixbrucker/chia-dashboard-ui

Based on Linuxserver.io Alpine Nginx container. 
* Requries existing MongoDB setup - https://hub.docker.com/_/mongo
* Requires at least one oauth provider - Google/Gitlab/Discord supported
* Requires Chia-Dashboard-Core - https://github.com/edifus/docker-chia-dashboard-core 


## Instructions

```
git clone https://github.com/edifus/docker-chia-dashboard-ui.git
cd docker-chia-dashboard
````

### Dashboard Core

* Modify `.env.tempalte` and save as `.env` with your own values, requires at least one oauth provider.
* `PUID` - Set UserID to run container | `PGID` - Set GroupID to run container

### Dashboard UI

* Modify `dockerfile/config.ts.template` and save as `dockerfile/config.ts` with your own values, requires at least one oauth provider.
* `PUID` and `PGID` will apply to this container as well

### Run containers

```
docker-compose up -d
```

## Dashboard UI

* Access WebUI at http://\<docker-host-ip\>


## Reverse Proxy

* Recommend running these services behind reverse proxy with SSL if hosting over the internet.

### Example Traefik v2

```
version: '3.8'

services:
  chia-dashboard-core:
    image: ghcr.io/edifus/chia-dashboard-core:latest
    container_name: chia-dashboard-core
    env_file:
      - .env
    volumes:
      - type: bind
        source: /etc/localtime
        target: /etc/localtime
        read_only: true
    labels:
      traefik.enable: 'true'
      traefik.http.routers.chia-dashboard-api-http.entrypoints: http
      traefik.http.routers.chia-dashboard-api-http.rule: Host(`chia-dashboard-api.domain.tld`)
      traefik.http.middlewares.redirect-https.redirectscheme.scheme: https
      traefik.http.middlewares.redirect-https.redirectscheme.permanent: 'true'
      traefik.http.routers.chia-dashboard-api-http.middlewares: redirect-https
      traefik.http.routers.chia-dashboard-api-http.service: chia-dashboard-api
      traefik.http.routers.chia-dashboard-api-https.entrypoints: https
      traefik.http.routers.chia-dashboard-api-https.rule: Host(`chia-dashboard-api.domain.tld`)
      traefik.http.routers.chia-dashboard-api-https.tls: 'true'
      traefik.http.routers.chia-dashboard-api-https.service: chia-dashboard-api
      traefik.http.services.chia-dashboard-api.loadbalancer.server.port: 5000
    restart: unless-stopped
  chia-dashboard-ui:
    build: ./dockerfile
    image: chia-dashboard-ui
    container_name: chia-dashboard-ui
    env_file:
      - .env
    volumes:
      - type: bind
        source: /etc/localtime
        target: /etc/localtime
        read_only: true
      - type: bind
        source: /path/to/config
        target: /config
    labels:
      traefik.enable: 'true'
      traefik.http.routers.chia-dashboard-http.entrypoints: http
      traefik.http.routers.chia-dashboard-http.rule: Host(`chia-dashboard.domain.tld`)
      traefik.http.routers.chia-dashboard-http.middlewares: redirect-https
      traefik.http.routers.chia-dashboard-http.service: chia-dashboard
      traefik.http.routers.chia-dashboard-https.entrypoints: https
      traefik.http.routers.chia-dashboard-https.rule: Host(`chia-dashboard.domain.tld`)
      traefik.http.routers.chia-dashboard-https.tls: 'true'
      traefik.http.routers.chia-dashboard-https.service: chia-dashboard
      traefik.http.services.chia-dashboard.loadbalancer.server.port: 80
    restart: unless-stopped
```

### WebUI

* Access Dashboard at https://chia-dashboard.domain.tld
* Direct Satellites to https://chia-dashboard-api.domain.tld
* `config.ts` use `export const apiBaseUrl = 'https://chia-dashboard-api.domain.tld/api';`

### Satellite Configuration Example

```
chiaConfigDirectory: /home/user/.chia/mainnet
apiKey: f2d3a4ed-6480-4ae8-b130-06fd1845b440
excludedServices:
  - wallet
  - fullNode
chiaDashboardCoreUrl: https://chia-dashboard-api.domain.tld
```
