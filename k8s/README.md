# Kubernetes / Helm – Fail2Ban (in host mode)

Kubernetes and Helm support for Fail2Ban in **host mode**: one pod per node (DaemonSet), host network, host iptables. Config is **embedded in the chart** (from `config/`); no need to copy or mount fail2ban-data on nodes.

## Requirements

- Kubernetes cluster, Helm 3
- Nodes must allow privileged containers and capabilities NET_ADMIN, NET_RAW, SYS_NICE

## Quick start

```bash
helm install fail2ban ./k8s/helm/fail2ban --namespace fail2ban --create-namespace
```

No `--set` needed. The chart ships with the repo’s jail, filter, and cron configs.

## Configuration (values.yaml)

| Value | Description |
|-------|-------------|
| `image.repository` / `image.tag` | Fail2Ban image (default: `crazymax/fail2ban:latest`) |
| `hostLogPath` | Host path for application logs fail2ban monitors (mounted read-only at `containerLogPath`). Default: `/var/log` |
| `containerLogPath` | Path inside the container for logs (default: `/logs`). Jail `logpath` in the chart uses this, e.g. `/logs/*.log`. |
| `resources` | CPU/memory limits/requests |

Config (jail.d, filter.d, fail2ban.local, cron-reload) lives in the chart under `config/` and is deployed as ConfigMaps.

## Host mode and volumes

- **DaemonSet** with `hostNetwork: true`, `hostPID: true`; privileged + NET_ADMIN, NET_RAW, SYS_NICE.
- **ConfigMaps** (from chart `config/`): jail.d, filter.d, fail2ban.local, cron-reload → mounted into the container.
- **Logs**: `hostLogPath` → `containerLogPath` (read-only); host `/var/log` → `/var/log` (for fail2ban log and cron).

## After install

Fail2Ban runs as a DaemonSet (one pod per node). Config is read from the chart’s ConfigMaps. Log path: host path is mounted at `/logs` in the container.

To list pods and run commands:

```bash
kubectl get pods -n fail2ban

kubectl exec -n fail2ban -it <pod-name> -- fail2ban-client status
kubectl exec -n fail2ban -it <pod-name> -- fail2ban-client status custom-ban
kubectl exec -n fail2ban -it <pod-name> -- fail2ban-client reload
```

## Uninstall

```bash
helm uninstall fail2ban -n fail2ban
```

Uninstall does not remove iptables rules added by Fail2Ban; clean those on the nodes if needed.
