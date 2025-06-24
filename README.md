# fail2ban-docker

- [fail2ban-docker](#fail2ban-docker)
  - [Purpose](#purpose)
  - [fail2ban-data Folder Structure](#fail2ban-data-folder-structure)
  - [Passing Logs as Volumes](#passing-logs-as-volumes)
  - [Docker - How Run?](#docker---how-run)
  - [Docker - How Stop?](#docker---how-stop)
  - [Docker - How interact with Fail2Ban Cli?](#docker---how-interact-with-fail2ban-cli)
  - [NOTE](#note)

## Purpose

This project provides a containerized environment for running Fail2Ban, a popular intrusion prevention software framework that protects servers from ddos attacks.
The setup is designed to be easily configurable and extensible, allowing users to monitor custom log files and automate maintenance tasks using cron jobs.

## fail2ban-data Folder Structure

- **fail2ban.local**: File used to override settings on `/etc/fail2ban/fail2ban.conf`;
- **jail.d/**: Directory containing custom action scripts for Fail2Ban, allowing you to specify how bans are enforced (e.g., firewall rules, notifications) and matching with our custom filters and logs;
- **filter.d/**: Directory for custom filter definitions, which determine the patterns Fail2Ban looks for in log files to detect malicious activity.
- **cron-reload/**: Contains cron job definitions for automating tasks such as Fail2Ban reload, Fail2Ban restarts, or other maintenance scripts. At this phase, a cron is added to reload Fail2Ban every hour.
  - This cron jobs are important if we are dealing with dynamic logs, where the filename is not static and we had log rotation, with different names, and we need to reload Fail2Ban, to add the new log files to the monitoring list.

## Passing Logs as Volumes

To monitor your own log files with Fail2Ban inside the container, you should mount your log directory as a volume.
Please adjust the `docker-compose.yml` file.

## Docker - How Run?

```bash
docker compose up -d
```

## Docker - How Stop?

```bash
docker compose down
```

## Docker - How interact with Fail2Ban Cli?

```bash
docker exec -it fail2ban fail2ban-client <command>
docker exec -it fail2ban fail2ban-client status
docker exec -it fail2ban fail2ban-client status custom-ban
docker exec -it fail2ban fail2ban-client reload
```

## NOTE

In production environments, it is recommended to use this in host mode, so that Fail2Ban can directly interact with the host's firewall.
