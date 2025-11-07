# Docker tips and tricks

## How to run

The steps are fairly simple:

1. Create a directory for your tt-rss installation. Do the rest in there.
1. Get the `.env` file and edit it to suite your needs.
1. Make sure you change all the password using something like `pwgen` to generate long and
   complex ones.
1. Get the `docker-compose.yml` file and edit it to suite your needs.
1. Run `docker-compose up [-d]`. The `-d` stands for detached so you can run it in the
   background.
1. [Optional] Run `lazydocker` so you can always see what is happening.
1. [Optional] Run lots of docker commands so you see what is happening.

## Lazydocker

[Lazydocker](https://github.com/jesseduffield/lazydocker) is a nice little terminal UI for
both docker and docker-compose, written in Go with the `gocui` library. This means that
you can run it in an ssh session to see what your docker compose installation is doing —
or not.

## GAh! I want to start over!

⚠️This will remove everything!

- Purging All Unused or Dangling Images, Containers, Volumes, and Networks:
  `docker system prune -a`.
- Remove all images: `docker rmi $(docker images -a -q)`
- Remove all containers: `docker rm $(docker ps -a -f status=exited -q)`
- Remove all volumes: `docker volume prune`

Those are nuclear options and will remove **everything** so use with caution.

## Backups

Better be safe than sorry. **This is not optional**: make sure you either run the backup
container, or have another backup strategy in place.

Additionally, you should download your data in `OPML` regularity.
