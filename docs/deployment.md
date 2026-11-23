# YoungAi deployment on DGX Spark

Updated 2026-11-23. This documents the deployment on this machine and the prepared restoration release.

## Current setup

The model server runs CUDA on the DGX Spark using the DeepSeek V4.1 Flash 1017 GGUF. The current process uses the restoration sidecar from the prepared release, the existing DeepSeek 1017 engram shards, and the vision GGUF. It listens only on `127.0.0.1:8001` with a 110,000 MB memory budget. Apache listens on `0.0.0.0:8000`, requires the existing bearer token, and proxies requests to the model server. Keep the token in `$HOME/youngai/proxy/apache.conf`; do not put it in shell history or this document.

Model, sidecars, engram shards and runtime state live under `$HOME/youngai`. Release commands and logs live under `$HOME/youngai/releases/restore-20261123`. The model log is `logs/restore.log`; watchdog output is `logs/watchdog-process.log` and `logs/watchdog.log`.

## Bring the current model up

This command checks the staged files and runtime settings, stops any already running model process tracked by the deployment, starts the restoration model, waits for readiness, checks authenticated proxy access, and makes a short chat request:

```bash
$HOME/youngai/releases/restore-20261123/deploy.sh
```

It can take a few minutes to load; the startup check allows up to 15 minutes. The API remains at `http://<spark-host>:8000/v1` and uses the existing bearer token. Check readiness without starting or stopping anything with:

```bash
$HOME/youngai/releases/restore-20261123/deploy.sh --check
```

## Bring the model down

Stop the model backend and its watchdog while leaving Apache running:

```bash
$HOME/youngai/releases/restore-20261123/stop.sh
```

Afterward port 8001 should be closed. Port 8000 remains open and returns an upstream error for model requests until the backend is started again. This command validates process identity before signalling it and removes the backend PID/state files.

## Restore the previous coding-sidecar deployment

To run the previous server binary and coding sidecar instead of the restoration model:

```bash
$HOME/youngai/releases/restore-20261123/rollback.sh
```

This starts a model. To stop it, use `stop.sh` above. Apache is independent of the model process and should only be stopped separately when intentionally taking the proxy offline.
