---
title: "Multiple apps on one server using kamal"
date: 2024-01-31T15:17:00+02:00
draft: false
categories:
  - Learning
---

Kamal's effectiveness is undeniable. Its integration with power-efficient bare-metal servers, such as those available on Hetzner.com, elevates its appeal, allowing for the simultaneous deployment of multiple Rails applications (or applications developed with other frameworks) on a single server. This setup can be further enhanced with the inclusion of additional components like PostgreSQL or Redis.

It's important to note that while each service in this configuration operates within its own namespace, Traefik does not share this characteristic; only one Traefik service can be included in our setup. In my scenario, I've designated the Prograils service, which powers the [prograils.com](https://prograils.com) website, as the primary service. Below is the complete configuration for this setup:

```yaml
service: prograils

image: prograils/prograils

servers:
  web:
    hosts:
      - 46.4.64.80
    options:
      network: "prograils"
    labels:
      traefik.http.routers.prograils-web-production.rule: Host(`prograils.com`)
  worker:
    traefik: false
    hosts:
      - 46.4.64.80
    cmd: bin/sidekiq
    options:
      network: "prograils"

# Credentials for your image host.
registry:
  username: mlitwiniuk
  password:
    - MRSK_REGISTRY_PASSWORD

env:
  clear:
    HOST: 'prograils.com'
    DB_HOST: 'prograils-db'
    DB_PORT: 5432
    REDIS_URL: 'redis://prograils-redis:6379/0'
    RAILS_LOG_TO_STDOUT: true
    RAILS_SERVE_STATIC_FILES: false
  secret:
    - POSTGRES_PASSWORD
    - SECRET_KEY_BASE
    - APPSIGNAL_PUSH_API_KEY
    - APPSIGNAL_APP_NAME

accessories:
  db:
    image: postgres:15
    port: 5433:5432
    host: 46.4.64.80
    env:
      clear:
        POSTGRES_USER: 'prograils'
        POSTGRES_DB: 'prograils_production'
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data
    options:
      network: "prograils"
  redis:
    image: redis:7.0
    host: 46.4.64.80
    port: 6380:6379
    directories:
      - data:/data
    options:
      network: "prograils"

traefik:
  options:
    network: "prograils"

healthcheck:
  max_attempts: 10
```


There are few non-standard configuration options in here:

1. Web server has additional label: ```traefik.http.routers.prograils-web-production.rule: Host(`prograils.com`)``` - it's to inform traefik where to direct requests for prograils.com domain. Also note the service name suffix (`-production`) - I'm deploying all my apps with destination parameter
2. Postgres and redis use standard port internally, but custom one externally - it's for me to be able to connect to them to ie. make a backup. And having non-custom port ensures me, that there won't be any conflicts between same services: `port: 5433:5432`
3. All services, accessories and traefik have an additional option with network defined - this is for those containers to be in the same "virtual" docker network and one could access the other via the name, ie. `prograils-db` and not the IP address (which is subject of change upon machine restart):
   ```yaml
   options:
     network: "prograils"
   ```
4. Kamal won't create docker network during the deploy process, but actually it's pretty easy to automate it through hooks, ie. like the one by [@tikal](https://github.com/tikal "{rel='nofollow' target='_blank'}") here: [Docker networking and accessories · Issue #41 · basecamp/kamal · GitHub](https://github.com/basecamp/kamal/issues/41#issuecomment-1789223148 "{rel='nofollow' target='_blank'}") - put it in `.kamal/hooks/pre-deploy` and don't forget to make it executable.
   ```bash
   #!/usr/bin/env bash

   REMOTE_HOST="user@prograils.com"
   NETWORK_NAME="prograils"
   # SSH into the remote host and execute Docker commands
   ssh $REMOTE_HOST << EOF
    # Check if the Docker network already exists
    if ! docker network inspect "$NETWORK_NAME" &>/dev/null; then
        # If it doesn't exist, create it
        docker network create "$NETWORK_NAME"
        echo "Created Docker network: $NETWORK_NAME"
    else
        echo "Docker network $NETWORK_NAME already exists, skipping creation."
    fi
   EOF


Config of the second app (it's staging version of [humadroid.io](https://humadroid.io) ) is very similar:


```yaml
service: humadroid

image: mlitwiniuk/humadroid

servers:
  web:
    hosts:
      - 46.4.64.80
    options:
      network: "humadroid"
    labels:
      traefik.http.routers.humadroid-web-production.rule: HostRegexp(`humadroid.dev`, `{subdomain:[a-z0-9]+}.humadroid.dev`)
  worker:
    traefik: false
    hosts:
      - 46.4.64.80
    cmd: bin/sidekiq -C config/sidekiq.yml
    options:
      network: "humadroid"

registry:
  username: mlitwiniuk
  password:
    - MRSK_REGISTRY_PASSWORD

env:
  clear:
    HOST: 'humadroid.dev'
    RAILS_LOG_TO_STDOUT: true
    RAILS_SERVE_STATIC_FILES: true
    DB_HOST: 'humadroid-db'
    DB_PORT: 5432
    REDIS_URL: 'redis://humadroid-redis:6379/0'
  secret:
    - POSTGRES_PASSWORD
    - SECRET_KEY_BASE

accessories:
  db:
    image: postgres:15
    port: 5434:5432
    host: 46.4.64.80
    env:
      clear:
        POSTGRES_USER: 'humadroid'
        POSTGRES_DB: 'humadroid_production'
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data
    options:
      network: "humadroid"
  redis:
    image: redis:7.0
    host: 46.4.64.80
    port: 6381:6379
    directories:
      - data:/data
    options:
      network: "humadroid"

healthcheck:
  max_attempts: 10
```

Note that there is no explicit Traefik configuration mentioned here. Traefik automatically identifies how to do routing through the labels assigned to the web server. All servers and supplementary services are incorporated into a newly created, separate Docker network, with accessory services configured to expose distinct ports. Essentially, that's all there is to it. The only remaining step is to link Traefik to this new network (named "humadroid" in this case). This connection is established through a post-deploy hook, which is similar in nature to a pre-deploy hook. This hook should be placed in .kamal/hooks/post-deploy and must be made executable.

```bash
#!/usr/bin/env bash

REMOTE_HOST="-p 2122 prograils@prograils.com"
NETWORK_NAME="humadroid"

# SSH into the remote host and execute Docker commands
ssh $REMOTE_HOST << EOF
    # Check if the Docker network already exists
    if ! docker network inspect "$NETWORK_NAME" 2>/dev/null | grep traefik; then
        # If it doesn't exist, create it
        docker network connect "$NETWORK_NAME" traefik
        echo "Connected traefik to docker network: $NETWORK_NAME"
    else
        echo "Traefik already connected to docker network $NETWORK_NAME ."
    fi
EOF
```

And voila, both apps should be working from the same machine, all contenerized and deplyed thanks to kamal.
