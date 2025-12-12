---
layout: default
title: Installation Guide
nav_order: 2
---

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Overview

The main (and recommended) way to run tt-rss is under Docker.

Docker images for <https://github.com/tt-rss/tt-rss> are being built (for `linux/amd64` and `linux/arm64`) and published
([via GitHub Actions](https://github.com/tt-rss/tt-rss/actions/workflows/publish.yml)) to:
* Docker Hub (as [supahgreg/tt-rss](https://hub.docker.com/r/supahgreg/tt-rss/) and [supahgreg/tt-rss-web-nginx](https://hub.docker.com/r/supahgreg/tt-rss-web-nginx/))
* GitHub Container Registry (as [ghcr.io/tt-rss/tt-rss](https://github.com/orgs/tt-rss/packages/container/package/tt-rss)
  and [ghcr.io/tt-rss/tt-rss-web-nginx](https://github.com/orgs/tt-rss/packages/container/package/tt-rss-web-nginx))

{: .warning }
> Podman is not Docker. Please don't report issues related to running tt-rss when using Podman or Podman Compose.

This setup uses PostgreSQL and runs tt-rss using several containers as outlined below.
Consider using an external [Patroni cluster](https://patroni.readthedocs.io/en/latest/) instead of a single `db` container in "production" deployments.

## TL;DR

Place [`.env`](#env) and [`docker-compose.yml`](#docker-composeyml) (contents below) together in a directory, edit `.env` as you see fit, and run `docker compose up -d`.

## In more detail

1. Create a directory for your tt-rss installation. Do the remaining steps in there.
2. Create a `.env` file using [`.env`](#env) as a starting point; edit it to suit your needs (e.g. adjusting `HTTP_PORT`).
   * Consider changing password/secret environment variables to something you're comfortable with (e.g. `pwgen`-generated values).
3. Create a `docker-compose.yml` file using [`docker-compose.yml`](#docker-composeyml) as a starting point; edit it to suit your needs
   (e.g. enabling the `backups` container, using the `ghcr.io` images, using a newer `postgres` image, etc.).
4. Run [`docker compose up -d`](https://docs.docker.com/reference/cli/docker/compose/up/) to bring up the environment.
   * Note that the `-d` will result in the containers running in the background, which is generally what you want.
5. Review containers logs and states.  Some typical ways this may be done include:
   * Running commands like [`docker compose ps`](https://docs.docker.com/reference/cli/docker/compose/ps/) and [`docker compose logs`](https://docs.docker.com/reference/cli/docker/compose/logs/)
   * Using a third-party tool like [`lazydocker`](https://github.com/jesseduffield/lazydocker) (a terminal UI for Docker and Docker Compose).
6. Access tt-rss in your browser.
   * The URL to use depends upon how you set things up, but assuming you kept `HTTP_PORT=127.0.0.1:8280` in your `.env` file and are on the same system
     as tt-rss, you'd use <http://127.0.0.1:8280/tt-rss>.
7. Log in as `admin` or (if you enabled the related environment variables) the auto-created user.
   * See comments in [`.env`](#env) regarding the password(s).

### .env

```ini
# Put any local modifications here.

# Run FPM under this UID/GID.
# OWNER_UID=1000
# OWNER_GID=1000

# FPM settings.
#PHP_WORKER_MAX_CHILDREN=5
#PHP_WORKER_MEMORY_LIMIT=256M

# ADMIN_USER_* settings are applied on every startup.

# Set admin user password to this value. If not set, random password
# will be generated on startup, look for it in the 'app' container logs.
#ADMIN_USER_PASS=

# Sets admin user access level to this value. Valid values:
# -2 - forbidden to login
# -1 - readonly
#  0 - default user
# 10 - admin
#ADMIN_USER_ACCESS_LEVEL=

# Auto create another user (in addition to built-in admin) unless it already exists.
#AUTO_CREATE_USER=
#AUTO_CREATE_USER_PASS=
#AUTO_CREATE_USER_ACCESS_LEVEL=0

# Default database credentials.
TTRSS_DB_USER=postgres
TTRSS_DB_NAME=postgres
TTRSS_DB_PASS=password

# You can customize other config.php defines by setting overrides here.
# See tt-rss/.docker/app/Dockerfile for a complete list.

# You probably shouldn't disable auth_internal unless you know what you're doing.
# TTRSS_PLUGINS=auth_internal,auth_remote
# TTRSS_SINGLE_USER_MODE=true
# TTRSS_SESSION_COOKIE_LIFETIME=2592000
# TTRSS_FORCE_ARTICLE_PURGE=30
# ...

# Bind exposed port to 127.0.0.1 to run behind reverse proxy on the same host.
# If you plan to expose the container, remove "127.0.0.1:".
HTTP_PORT=127.0.0.1:8280
#HTTP_PORT=8280
```

### docker-compose.yml

{: .warning }
> See [this FAQ entry](#i-got-the-updated-docker-compose-file-above-and-now-my-database-keeps-restarting)
> if you're upgrading between PostgreSQL major versions (e.g. 15 to 17).

{: .warning }
> Regarding PostgreSQL 18:
> * Support for backing up PostgreSQL 18 databases via the `backups` container image was introduced on 2025-12-04.
> * PostgreSQL 18 (and newer) Docker images have changed the volume from `/var/lib/postgresql/data` to `/var/lib/postgresql`.
>   The example below includes a commented-out volume mapping that demonstrates this.
>   * See <https://hub.docker.com/_/postgres> and <https://github.com/docker-library/postgres/pull/1259> for more info.

```yaml
services:
  db:
    image: postgres:17-alpine
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - POSTGRES_USER=${TTRSS_DB_USER}
      - POSTGRES_PASSWORD=${TTRSS_DB_PASS}
      - POSTGRES_DB=${TTRSS_DB_NAME}
    volumes:
      - db:/var/lib/postgresql/data
      # or, if 18+
      # - db:/var/lib/postgresql

  app:
    image: supahgreg/tt-rss:latest
    # or
    # image: ghcr.io/tt-rss/tt-rss:latest
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - app:/var/www/html
      - ./config.d:/opt/tt-rss/config.d:ro
    depends_on:
      - db

#  optional, makes weekly backups of your install
#  backups:
#    image: supahgreg/tt-rss:latest
#    # or
#    # image: ghcr.io/tt-rss/tt-rss:latest
#    restart: unless-stopped
#    env_file:
#      - .env
#    volumes:
#      - backups:/backups
#      - app:/var/www/html
#    depends_on:
#      - db
#    command: /opt/tt-rss/dcron.sh -f

  updater:
    image: supahgreg/tt-rss:latest
    # or
    # image: ghcr.io/tt-rss/tt-rss:latest
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - app:/var/www/html
      - ./config.d:/opt/tt-rss/config.d:ro
    depends_on:
      - app
    command: /opt/tt-rss/updater.sh

  web-nginx:
    image: supahgreg/tt-rss-web-nginx:latest
    # or
    # image: ghcr.io/tt-rss/tt-rss-web-nginx:latest
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - ${HTTP_PORT}:80
    volumes:
      - app:/var/www/html:ro
    depends_on:
      - app

volumes:
  db:
  app:
  backups:
```

## FAQ

### How do I update tt-rss?

When you see that tt-rss needs an update, just pull the latest image/code.  How this is done depends upon your environment, but typical ways to accomplish this are:
* Docker (generally in the directory where you placed `docker-compose.yml`): `docker compose pull && docker compose up -d`
* Git (from your tt-rss directory): `git pull`

If a tt-rss database upgrade is required you'll be redirected to a special screen in the UI.

### Your Docker images won't run on X

If you're using an OS or architecture that isn't currently supported (i.e. something other than `linux/amd64` and `linux/arm64`),
you'll likely need to build your own Docker images by using an override and running `docker compose build`.

```yaml
# docker-compose.override.yml
services:
  app:
    image: supahgreg/tt-rss:latest
    # or
    # image: ghcr.io/tt-rss/tt-rss:latest
    build:
      dockerfile: .docker/app/Dockerfile
      context: https://github.com/tt-rss/tt-rss.git
      args:
        BUILDKIT_CONTEXT_KEEP_GIT_DIR: 1

  web-nginx:
    image: supahgreg/tt-rss-web-nginx:latest
    # or
    # image: ghcr.io/tt-rss/tt-rss-web-nginx:latest
    build:
      dockerfile: .docker/web-nginx/Dockerfile
      context: https://github.com/tt-rss/tt-rss.git
```

`BUILDKIT_CONTEXT_KEEP_GIT_DIR` build argument is needed to display tt-rss version info properly.
If that doesn't work for you (no BuildKit?) you'll have to resort to terrible hacks.

{: .warning }
> Self-built images are not necessarily supported (i.e. best effort and/or community support).

### I got the updated Docker Compose file above and now my database keeps restarting

We'll use the following error message as an example of what you might see in the logs:

`Error message: The data directory was initialized by PostgreSQL version 12, which is not compatible with this version 15.4.`

Official PostgreSQL containers have no support for migrating data between major versions.
Using the aforementioned example, you could do one of the following:

1. Replace `postgres:15-alpine` with `postgres:12-alpine` in `docker-compose.yml` (or use `docker-compose.override.yml`, see below) and keep using PG 12
2. Use [this DB container](https://github.com/pgautoupgrade/docker-pgautoupgrade) which would automatically upgrade the database
3. Migrate the data manually using `pg_dump` and `pg_restore` (somewhat complicated if you haven't done it before)

### I'm using docker-compose.override.yml and now I'm getting schema update (and other) strange issues

You've might've changed something related to `/var/www/html/tt-rss` in `docker-compose.yml`.

Your Docker setup is messed up for some reason, so tt-rss can't update itself to the persistent storage location on startup
(this is just an example of one issue, there could be many others).

Consider undoing any recent changes, searching for error messages, etc.

### How do I make it run without /tt-rss/ in the URL, i.e. at website root?

Set the following variables in `.env`:

```ini
APP_WEB_ROOT=/var/www/html/tt-rss
APP_BASE=
```

Don't forget to remove `/tt-rss/` from `TTRSS_SELF_URL_PATH` (if you have it set).

### How do I apply configuration options?

There are two sets of options you can change through the environment: those specific to tt-rss (which are prefixed with `TTRSS_`) and those affecting container behavior.

#### Options specific to tt-rss

For example, to set tt-rss global option `SELF_URL_PATH`, add the following to `.env`:

```ini
TTRSS_SELF_URL_PATH=https://example.com/tt-rss
```

Don't use quotes around values. Note the prefix (`TTRSS_`) before the value.

Look at [Global-Config](Global-Config.md) for more information.

#### Container options

Some options, but not all, are mentioned in `.env-dist`. You can see all available options in the [Dockerfile](https://github.com/tt-rss/tt-rss/blob/main/.docker/app/Dockerfile).

### How do I customize the YAML without committing my changes to git?

You can use [docker-compose.override.yml](https://docs.docker.com/compose/extends/). For example, customize `db` to use a different `postgres` image:

```yml
# docker-compose.override.yml
services:
  db:
    image: postgres:17-alpine
```

### I'm trying to run CLI tt-rss scripts inside the container and they complain about root

In your Docker Compose directory, run something like one of the examples below.
Check <https://github.com/tt-rss/tt-rss/blob/main/.docker/app/Dockerfile> for the latest image's PHP version.

```sh
docker compose exec --user app app php84 /var/www/html/tt-rss/update.php --help

#                           ^   ^
#                           |   |
#                           |   +- service (container) name
#                           +----- run as user
```

or

```sh
docker compose exec app sudo -Eu app php84 /var/www/html/tt-rss/update.php --help
```

or

```sh
docker exec -it <container_id> sudo -Eu app php84 /var/www/html/tt-rss/update.php --help
```

Note: `sudo -E` is needed to keep environment variables.

### How do I add plugins and themes?

{: .note }
> First party plugins can be added using plugin installer in `Preferences` &rarr; `Plugins`.

By default, tt-rss code is stored on a persistent Docker volume (`app`). You can find
its location like this:

```sh
docker volume inspect ttrss-docker_app | grep Mountpoint
```

Alternatively, you can mount any host directory as ``/var/www/html`` by updating ``docker-compose.yml``, i.e.:

```yml
volumes:
      - app:/var/www/html
```

Replace with:

```yml
volumes:
      - /opt/tt-rss:/var/www/html
```

Copy and/or git clone any third party plugins into ``plugins.local`` as usual.

### I'm running into 502 errors and/or other connectivity issues

First, check that all containers are running:

```text
$ docker compose ps
                   Name                                 Command               State           Ports
------------------------------------------------------------------------------------------------------------
ttrss-docker-demo_app_1_f49351cb24ed         /bin/sh -c /startup.sh           Up      9000/tcp
ttrss-docker-demo_backups_1_8d2aa404e31a     /dcron.sh -f                     Up      9000/tcp
ttrss-docker-demo_db_1_fc1a842fe245          docker-entrypoint.sh postgres    Up      5432/tcp
ttrss-docker-demo_updater_1_b7fcc8f20419     /updater.sh                      Up      9000/tcp
ttrss-docker-demo_web-nginx_1_fcef07eb5c55   /docker-entrypoint.sh ngin ...   Up      127.0.0.1:8280->80/tcp
```

Then, ensure that frontend (`web-nginx` or-`web`) container is up and can contact FPM (`app`) container:

```text
$ docker compose exec web-nginx ping app
PING app (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.144 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.128 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.206 ms
^C
--- app ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.128/0.159/0.206 ms
```

Containers communicate via DNS names assigned by Docker based on service names defined in `docker-compose.yml`. This means that services (specifically `app`) and Docker DNS service should be functional.

Similar issues may be also caused by Docker `iptables` functionality either being disabled or conflicting with `nftables`.

### I want to rename `app` (FPM) container

You can but you'll need to pass `APP_UPSTREAM` environment variable to the `web-nginx` container with its new name.

### How do I put this container behind a reverse proxy?

- Don't forget to pass `X-Forwarded-Proto` to the container if you're using HTTPS, otherwise tt-rss would generate plain HTTP URLs.
- Upstream address and port are set using `HTTP_PORT` in `.env`:

```ini
HTTP_PORT=127.0.0.1:8280
```

#### nginx example

```nginx
location /tt-rss/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://127.0.0.1:8280/tt-rss/;
    break;
}
```

If you run into problems with global PHP-to-FPM handler taking priority over proxied location, define the tt-rss location like this so it takes higher priority:

```nginx
location ^~ /tt-rss/ {
   ....
}
```

If you want to pass an entire nginx virtual host to tt-rss:

```nginx
server {
   server_name rss.example.com;

   ...

   location / {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://127.0.0.1:8280/;
      break;
   }
}
```

Note that `proxy_pass` in this example points to container website root.

#### Apache example

```apache
<IfModule mod_proxy.c>
    <Location /tt-rss>
      ProxyPreserveHost On
      ProxyPass        http://localhost:8280/tt-rss
      ProxyPassReverse http://localhost:8280/tt-rss
      RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
    </Location>
  </IfModule>
```

### I have internal web services tt-rss is complaining about (URL is invalid, loopback address, disallowed ports)

Put your local services on the same Docker network with tt-rss, then access them by service (i.e. host) names, i.e. `http://rss-bridge/`.

```yml
services:
   rss-bridge:
....
networks:
  default:
    external:
      name: ttrss-docker_default
```

If your service uses a non-standard (i.e. not `80` or `443`) port, make an internal reverse proxy sidecar container for it.

### Backup and restore

It's highly recommended that you back up your data.  If your tt-rss is particularly important to you, also consider validating the backup artifacts by
using them to restore to a parallel environment.

If you have the `backups` container enabled, the default configuration takes automatic backups (database, local plugins, etc.) once a week to a separate storage volume.

{: .note }
> The `backups` container is included as a safety net for people who wouldn't otherwise bother with backups.
> For a more robust database backup/restore solution, consider setting up something like [WAL-G](https://github.com/wal-g/wal-g).

#### Manually taking a backup

To run [`.docker/app/backup.sh`](https://github.com/tt-rss/tt-rss/blob/main/.docker/app/backup.sh) (the backup script that executes weekly):

`docker compose exec backups /etc/periodic/weekly/backup`

Alternatively, if you want to initiate backups from the host (or if you're using PostgreSQL 18+, currently incompatible with the `backup` container) you can do something like this:

```sh
source .env
docker compose exec \
  -e PGPASSWORD="$TTRSS_DB_PASS" \
  db \
  /bin/bash \
  -c "export PGPASSWORD=$TTRSS_DB_PASS \
    && pg_dump -U $TTRSS_DB_USER $TTRSS_DB_NAME" \
  | gzip -9 > backup.sql.gz
```

#### Restoring backups

The process to restore the database from a `backups` container backup might look like this:

1. Enter `backups` container shell: `docker compose exec backups /bin/sh`
2. Inside the container, locate and choose the backup file: `ls -t /backups/*.sql.gz`
3. Clear database (**THIS WOULD DELETE EVERYTHING IN THE DB**): `psql -h db -U $TTRSS_DB_USER $TTRSS_DB_NAME -e -c "drop schema public cascade; create schema public"`
3. Restore the backup: `zcat /backups/ttrss-backup-yyyymmdd.sql.gz | psql -h db -U $TTRSS_DB_USER $TTRSS_DB_NAME`

#### OPML

As an additional, **but incomplete**, form of backup, you might also wish to periodically export a subset of your tt-rss user data as OPML.
This may be done in `Preferences --> Feeds --> OPML` or via the CLI with a command like `update.php --opml-export:USERNAME:FILENAME`.

{: .warning }
> OPML exports are not a complete backup.
> They only include information about one user's categories, feeds (configuration only), and (optionally) tt-rss settings (e.g. preferences, labels, filters).

### How do I use custom certificates?

You need to mount custom certificates into the *app* and *updater* containers like this:

```yml
volumes:
    ....
    ./ca1.crt:/usr/local/share/ca-certificates/ca1.crt:ro
    ./ca2.crt:/usr/local/share/ca-certificates/ca2.crt:ro
    ....
```

Don't forget to restart the containers.

### How do I run these images on Kubernetes?

You'll need to set several mandatory environment values to the container running the `web-nginx` image:

1. `APP_UPSTREAM` should point to the fully-qualified DNS service name provided by the app (FPM) container/pod
2. `RESOLVER` should be set to `kube-dns.kube-system.svc.cluster.local`

### Where's the Helm chart?

There isn't an official Helm chart.

### I'm using Podman, and

We neither test against nor support Podman.
