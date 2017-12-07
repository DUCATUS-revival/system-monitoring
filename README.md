# Infrastructure Monitoring

## What it monitors

- docker events
- docker logs
- Auth logs
- System stats
- Application specific tests (TBD)

## Installing

### Docker Events Monitor

```bash
DOCKER_COMPOSE_DIR=location_of_service \
LOGGING_URL=logging.ducatus.io \
LOGGING_PORT=9090 \
./templater.sh docker-event-logging.service > /etc/systemd/system/docker-event-logging.service
systemctl start docker-event-logging.service
```

NOTE: I didn't create templater.sh

### Docker logs

```bash
LOGGING_URL=logging.ducatus.io \
LOGGING_PORT=9095 \
./templater.sh daemon.json > /etc/docker/daemon.json
```

Must restart the docker service. Also this solution will only start logging on new containers, not existing ones. Be sure to recreate any containers

### Auth Logs

```bash
DOCKER_COMPOSE_DIR=location_of_service \
LOGGING_URL=logging.ducatus.io \
LOGGING_PORT=9096 \
./templater.sh auth-log-logging.service > /etc/systemd/system/auth-log-logging.service
systemctl start auth-log-logging.service
```

## Docker Events

Reports the following:

- Container creation
- Container restarts
- Container stops/start
- New image pulls

Command:

```bash
docker-compose events --json
```

Output:

```json
{"service": "nginx", "time": "2017-12-06T16:30:05.403209", "action": "kill", "attributes": {"image": "nginx", "name": "ducatuscorewalletservice_nginx_1"}, "type": "container", "id": "7a4571f65cda750427692b85c71fc12f91918513515fd619d589b5d29e92b517"}
{"service": "nginx", "time": "2017-12-06T16:30:05.576882", "action": "die", "attributes": {"image": "nginx", "name": "ducatuscorewalletservice_nginx_1"}, "type": "container", "id": "7a4571f65cda750427692b85c71fc12f91918513515fd619d589b5d29e92b517"}
{"service": "nginx", "time": "2017-12-06T16:30:05.791829", "action": "stop", "attributes": {"image": "nginx", "name": "ducatuscorewalletservice_nginx_1"}, "type": "container", "id": "7a4571f65cda750427692b85c71fc12f91918513515fd619d589b5d29e92b517"}
{"service": "nginx", "time": "2017-12-06T16:30:07.205826", "action": "start", "attributes": {"image": "nginx", "name": "ducatuscorewalletservice_nginx_1"}, "type": "container", "id": "7a4571f65cda750427692b85c71fc12f91918513515fd619d589b5d29e92b517"}
{"service": "nginx", "time": "2017-12-06T16:30:07.205885", "action": "restart", "attributes": {"image": "nginx", "name": "ducatuscorewalletservice_nginx_1"}, "type": "container", "id": "7a4571f65cda750427692b85c71fc12f91918513515fd619d589b5d29e92b517"}

```

## Docker Logs

Reports the following:

- The container's foreground process's standard output and error

Command:

```bash
docker-compose logs -f --no-color -t
```

Switch explanation:

- `-f`: follow the logs
- `--no-color`: Remove the color so exported logs don't have color codes
- `-t`: Include timestamps

Output:

```
nginx_1    | 2017-10-20T18:50:06.978096559Z 70.123.60.56 - - [20/Oct/2017:18:50:06 +0000] "GET /bws/api/v2/wallets/?includeExtendedInfo=0&twoStep=1&r=93330 HTTP/1.1" 200 716 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36" "-"
nginx_1    | 2017-10-20T18:50:07.004851072Z 70.123.60.56 - - [20/Oct/2017:18:50:07 +0000] "GET /bws/api/v1/notifications/?timeSpan=21600&r=67847 HTTP/1.1" 200 35854 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36" "-"
nginx_1    | 2017-10-20T18:50:10.902654502Z 70.123.60.56 - - [20/Oct/2017:18:50:10 +0000] "GET /bws/api/v1/notifications/?notificationId=015085253925280000&r=98297 HTTP/1.1" 200 258 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36" "-"
```

## SSH Logs

Reports the following:

- Login attempts via SSH

Command:

```bash
tail -f /var/log/auth.log
```

Output:

```
Dec  6 03:11:19 dws sshd[30415]: Connection closed by 45.21.1.61 port 61355 [preauth]
Dec  6 03:17:01 dws CRON[30761]: pam_unix(cron:session): session opened for user root by (uid=0)
Dec  6 03:17:01 dws CRON[30761]: pam_unix(cron:session): session closed for user root
Dec  6 03:30:25 dws sshd[30779]: Accepted publickey for noah from 45.21.1.61 port 61613 ssh2: RSA SHA256:c35/iLHkqHqOraEHin0OVJuCX3MhaRJlhHqG0NtTMAk
Dec  6 03:30:25 dws sshd[30779]: pam_unix(sshd:session): session opened for user noah by (uid=0)
Dec  6 03:30:25 dws systemd-logind[1193]: New session 1459 of user noah.
```

This outputs more than SSH attempts, but can be filtered out at log aggregation

## System Stats

Reports the following:

- Filesystem stats
  - Available space
  - Total  space
- Uptime
  - CPU Load

### Filesystem Stats

Command:

```bash
df -h
```

Output:

```
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           396M   26M  370M   7% /run
/dev/xvda1       30G  6.2G   23G  22% /
tmpfs           2.0G  824K  2.0G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
none             30G  6.2G   23G  22% /var/lib/docker/aufs/mnt/65e0dd48dc7b3830aba7878a70752e0a215304594df18cd14b44c17d3cf545ae
```

This outputs docker filesystems, which let's us know how much space a given container is taking up, GROK rules will need to be in place in the log aggregation tool to allow easy correlation.

### Uptime

Command:

```bash
uptime
```

Output:

```
16:57:57 up 54 days, 17:59,  5 users,  load average: 0.06, 0.15, 0.18
```

## Application specific tests (TBD)

Reports the following for specific endpoints:

- Response time
- Response code
- Response body

We'll probably use a tool like Sentry to monitor this

# Logging Infrastructure

We'll use [Graylog](https://www.graylog.org/) for log aggregation and correlation. Graylog is basically a pretty version of the ELK stack, it handles clustering, triggers, log collection, and  visualizations. The ELK stack can do all of these things, but it's more labor intensive, doesn't have user management for free, and doesn't look nearly as good.

Each log type will have their own specific GROK rules to ensure proper parsing.

Every AWS EC2 and physical instance will report their logs to this system.
