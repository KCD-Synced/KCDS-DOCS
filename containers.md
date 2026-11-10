# Containers

The KCDS server is also a Docker image, built and published with every release on the GitHub
container registry:

```text
ghcr.io/kcd-synced/kcds-server:<version>
ghcr.io/kcd-synced/kcds-server:latest
```

Use the version tag your players' clients have, for example `1.0.4`: a client and a server of
different versions refuse each other when joining. The image runs the same Linux server as the
server package, with Node.js for the resources, and starts it with `start-server.sh`.

## Running it

```text
docker run -d --name kcds \
  -p 7777:7777/udp -p 7777:7777/tcp \
  --log-opt max-size=20m --log-opt max-file=5 \
  -e KCDS_SERVER_NAME="My server" \
  -v "$PWD/resources:/opt/kcds/resources" \
  ghcr.io/kcd-synced/kcds-server:1.0.4
```

- **Ports:** players connect over UDP 7777, and the game downloads your resources' files over TCP
  7777. Publish both, and open and forward both on the host as for the server package.
- **Resources:** mount your `resources` folder over `/opt/kcds/resources`. The image's own folder is
  empty.
- **Settings:** set them with the `KCDS_SERVER_*` variables (`-e`), or mount your `server.json` at
  `/opt/kcds/server.json`. A variable wins over `server.json`. Every setting and its variable is
  listed in [Running a server](/docs/server#settings-server-json). In the container the server
  listens on every interface unless the settings name one.
- **KCDS-Admin:** set `adminPort` and `adminPassword` (or their `KCDS_ADMIN_*` variables) and
  publish the admin port too, for example `-p 127.0.0.1:30121:30121`. Bind it to the host's
  loopback and reach it over SSH or a reverse proxy with TLS, as described for KCDS-Admin.
- **Restarts:** a restart from KCDS-Admin starts the server again inside the container. `docker stop`
  stops it cleanly.
- **Logs:** the server's log is the container's output, `docker logs kcds`. Docker's default log
  driver keeps all of it, so a server running for weeks can fill the disk: the `--log-opt` options
  above keep at most 5 files of 20 MB. In Docker Compose, put the same under `logging:`
  (`driver: json-file`, `options: {max-size: 20m, max-file: "5"}`), or set it for every container
  in Docker's `daemon.json`. To also have a log file, set `KCDS_LOG_FILE`, for example
  `-e KCDS_LOG_FILE=/opt/kcds/logs/server.log` with a volume on `/opt/kcds/logs`; it keeps
  `KCDS_LOG_FILES` (5) files of `KCDS_LOG_MAX_MB` (20) megabytes, as for the
  [server package](/docs/server#starting-it).

## Updating

The image is the update: pull the new version's tag and start a new container from it with the
same mounts and settings. KCDS-Admin's one-click server update is for the server package, not for
a container.

```text
docker pull ghcr.io/kcd-synced/kcds-server:<new version>
docker stop kcds && docker rm kcds
docker run -d --name kcds ... ghcr.io/kcd-synced/kcds-server:<new version>
```

From 1.0.4 the image carries GitHub's build provenance, as the downloads do: `gh attestation verify
oci://ghcr.io/kcd-synced/kcds-server:<version> --repo KCD-Synced/KCDS` shows that the release workflow of
the public repository built it.

The server writes its own data (crash reports, backups, bans and admin accounts) inside the
container under `/opt/kcds`. To keep it across a new container, keep that data on a volume or in
your mounted folders, as your setup needs.
