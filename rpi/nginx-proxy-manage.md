# Nginx Proxy Manager

Install [docker](docker.md)

```sh
mkdir -p ~/nginx/data
mkdir -p ~/nginx/letsencrypt
```

```sh
vim nginx docker-compose.yml
```

```yml, ~/nginx/docker-compose.yml
version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    # Uncomment the next line if you uncomment anything in the section
    # environment:
      # Uncomment this if you want to change the location of
      # the SQLite DB file within the container
      # DB_SQLITE_FILE: "/data/database.sqlite"

      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

## Fail2ban

### Install

```sh
sudo apt install fail2ban
```

### Configuration

```sh, /etc/fail2ban/filter.d/nginx-http-proxy.conf
[INCLUDES]

[Definition]

failregex = ^<HOST>.+" (4\d\d|3\d\d) (\d\d\d|\d) .+$
            ^.+ 4\d\d \d\d\d - .+ \[Client <HOST>\] \[Length .+\] ".+" .+$
```

```sh, /etc/fail2ban/action.d/nginx-http-proxy.conf
[Definition]

actionban = iptables -I DOCKER-USER -m string --algo bm --string 'X-Forwarded-For: <ip>' -j DROP

actionunban = iptables -D DOCKER-USER -m string --algo bm --string 'X-Forwarded-For: <ip>' -j DROP
```

```sh, /etc/fail2ban/jail.local
[nginx-http-proxy]
enabled = true
port = http,https
filter = nginx-http-proxy
logpath = ...
bantime = 1800
findtime = 10
maxretry = 3
action = nginx-http-proxy
```

### Restart fail2ban service

```sh
sudo service fail2ban restart
```

### Check jail status for specific `filter.d`

```sh
sudo fail2ban-client status nginx-http-proxy
```

### Unban IP

```sh
sudo fail2ban-client set nginx-http-proxy unbanip <IP>
```