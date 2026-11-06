# Superagent Runbook

How to start, stop and check the Docker container that runs dsh-superagent.
Run the commands below on the Docker host. The container is named
`dsh-superagent` (confirm with `docker ps`), and the superagent panel is served
at <http://127.0.0.1:3082>.

## Check

```sh
docker ps --filter name=dsh-superagent
```

For the service view, run `status` inside the container:

```sh
docker exec dsh-superagent /src/dsh-superagent/scripts/dsh-superagent --profile superagent status
```

Status exits `0` when everything is healthy, `1` when the service is down or
reports a runtime error (a restart may fix it), `2` when a component such as the
GitHub source is failing (a restart does not fix it), and `3` when the service
waits for setup. Failing parts and missing setup are printed under `errors` and
`setup`.

Read the logs when the panel does not answer or `status` reports a problem:

```sh
docker exec dsh-superagent tail -n 50 /var/log/dsh-superagent-boot.log
docker exec dsh-superagent sh -c 'ls -t /src/dsh-superagent/.artifacts/*.log | head -n 1'
```

## Start

```sh
docker start dsh-superagent
```

The container entrypoint starts the agent service automatically. If the
container is already running but the service is not, start it directly:

```sh
docker exec dsh-superagent /src/dsh-superagent/scripts/dsh-superagent --profile superagent start
```

Then run `status` again and open <http://127.0.0.1:3082>. If the container no
longer exists, recreate it with the deployment's Compose file
(`docker compose up -d`).

## Stop

Stop the service first so running tickets shut down gracefully, then stop the
container:

```sh
docker exec dsh-superagent /src/dsh-superagent/scripts/dsh-superagent --profile superagent stop
docker stop dsh-superagent
```

The service `stop` waits up to five minutes for executors to finish and never
force-kills processes.

## Restart

Restart the agent service without recreating the container:

```sh
docker exec dsh-superagent /src/dsh-superagent/scripts/dsh-superagent --profile superagent restart
```

Restart the whole container with `docker restart dsh-superagent`.
