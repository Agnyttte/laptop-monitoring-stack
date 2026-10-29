# Laptop Infrastructure Monitoring Stack

A small, working infrastructure monitoring stack built with **Docker Compose**, **Prometheus**, **node_exporter**, **Grafana** and **Alertmanager**. It collects host metrics from a Windows 11 laptop, shows them on live dashboards and raises alerts when thresholds are crossed.

Built for the PFD200 Professional Development module (FdSc Computing, NCG), September–October 2026, as practice for a Junior IT Infrastructure Engineer role.

![Architecture](screenshots/architecture.png)

## What it does

| Component | Port | Role |
|---|---|---|
| node_exporter | 9100 | Reads host metrics: CPU, memory, disk, uptime |
| Prometheus | 9090 | Scrapes node_exporter every 15 s, stores metrics, evaluates alert rules |
| Alertmanager | 9093 | Receives firing alerts from Prometheus |
| Grafana | 3000 | Dashboards built on PromQL queries |

## How to run it

Requirements: Docker Desktop (on Windows, with the WSL2 backend enabled).

```bash
git clone https://github.com/Agnyttte/laptop-monitoring-stack.git
cd laptop-monitoring-stack
docker compose up -d
```

Then open:
- Grafana: http://localhost:3000 (change the admin password on first login)
- Prometheus targets: http://localhost:9090/targets (node_exporter should show **UP**)
- Alertmanager: http://localhost:9093

In Grafana, add Prometheus as a data source (`http://prometheus:9090`), then import the four dashboard `.json` files from this repository (Dashboards → New → Import).

## Alert rules

| Alert | Condition | Must last |
|---|---|---|
| HighCPUUsage | CPU above 80% | 2 minutes |
| HighMemoryUsage | Memory above 85% | 2 minutes |
| LowDiskSpace | C: drive above 90% used | 5 minutes |

Alerting was tested by temporarily lowering the memory threshold: the alert went from inactive to pending to firing in about 90 seconds and reached Alertmanager.

## Screenshots

| | |
|---|---|
| ![CPU](screenshots/cpu-dashboard.png) | ![Memory](screenshots/memory-dashboard.png) |
| ![Disk](screenshots/disk-dashboard.png) | ![Uptime](screenshots/uptime-dashboard.png) |
| ![Targets](screenshots/prometheus-targets.png) | ![Alert firing](screenshots/alert-firing.png) |

## Known limitations

- node_exporter runs inside the WSL2 virtual machine, so CPU and memory figures describe that VM, not the whole Windows host. For a real Windows server, use **windows_exporter**.
- Alertmanager has no notification channel yet; alerts are visible in its web interface only.
- Prometheus, node_exporter and Alertmanager have no authentication, and `docker-compose.yml` publishes every port on all network interfaces. On a shared network, bind them to the local machine only, e.g. `"127.0.0.1:9090:9090"`.
- Grafana has no persistent volume; dashboards are kept as exported JSON so they can be re-imported.

## Security rules for anyone using this stack

1. **Your login is yours.** Sign in to Grafana with your own account. Never share the admin login.
2. **Check before you silence.** Investigate an alert before silencing it, and record the reason in a ticket.
3. **No secrets in Git.** Keep passwords in `.env` (already ignored). Run `git status` before every push.

## Further documents

- [Industry research presentation](docs/Industry_Research_Presentation.pptx)
- [Job advert log](docs/Industry_Research_Advert_Log.xlsx)

## Future work

- Combine the four dashboards into one
- Add a persistent volume for Prometheus history
- Add an email or Microsoft Teams notification channel
