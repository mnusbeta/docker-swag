# SWAG SETUP

# Docker stack / .yaml config file:
-----

```yaml

services:
  swag:
    image: lscr.io/linuxserver/swag:latest
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Budapest
      - URL=your.domain #replace
      - SUBDOMAINS=qbittorrent,sonarr,radarr,portainer,plex,overseerr #subdomains
      - VALIDATION=dns #or http
      - DNSPLUGIN=cloudflare
      - EMAIL=your@email.com #replace
      - DOCKER_MODS=linuxserver/mods:swag-cloudflare-real-ip
    volumes:
      - /data/swag/config:/config #location for config
    ports:
      - 443:443
    restart: unless-stopped

```

```bash
docker compose up
```

-----

# Cloudflare setup
-----

On the cloudflare site:
- Account - domain name - Get your API token - Create token - copy api token


```bash
nano /data/swag/config/dns-conf/cloudflare.ini
```

```bash
dns_cloudflare_api_token = yourapikeyhere
```

```bash
chmod 600 /data/swag/config/dns-conf/cloudflare.ini
```

```bash
ls -lah /data/swag/config/dns-conf/cloudflare.ini
```

```bash
chown 1000:1000 /data/swag/config/dns-conf/cloudflare.ini
```

```bash
docker restart swag
```

Check logs, if everything is ok, you can move forward:

```bash
docker logs -f swag
```

-----

# OPNsense / pfSense config

Services - Unbound DNS - Host Overrides - Add

- Host: subdomain
- Domain: domain name
- Type: A
- IP address: 192.168.x.x

Save - Restart service

- or edit host files on your device

-----

# Proxy config

```bash
nano /data/swag/config/nginx/proxy.conf
```
```bash
  # Proxy Header Settings

  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto https;
```

Save

# Add subdomain configs to SWAG

Some default ports:

- qbittorrent - 8082
- sonarr - 8989
- radarr - 7878
- portainer - 9443
- plex - 32400
- prowlarr - 9696
- overseerr - 5055


Example: 

```bash
nano /data/swag/config/nginx/proxy-confs/radarr.subdomain.conf
```

```php
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name radarr.your.domain;

    include /config/nginx/ssl.conf;

    location / {
        proxy_pass http://192.168.x.x:7878/;
        include /config/nginx/proxy.conf;
    }
}
```

Save
