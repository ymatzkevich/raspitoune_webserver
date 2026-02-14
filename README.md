# Hosting yoav.ch on raspitoune

## Download Docker

run once:
```
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```
and 
```
sudo apt install -y docker-compose-plugin
```

## Structure
```
mysite/
├── docker-compose.yml
├── nginx/
│   ├── nginx.conf
│   └── site.conf
├── website/
│   ├── index.html
│   └── ...
├── schmoundcloud/
│   └── ...
└── cloudflared/
    └── config.yml
```

## Routing

yoav.ch                → nginx → /var/www/website

schmoundcloud.yoav.ch  → nginx → /var/www/schmoundcloud

## Create Cloudflare tunnel
run this once:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel login

```
then:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel create mysite
```

### create Cloudflare DNS route

run this once:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel route dns mysite yoav.ch

docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel route dns mysite schmoundcloud.yoav.ch

```
that should create a `cloudflared/mysite.json` file and we should see both DNS records in Cloudflare as CNAME → tunnel.

## Fail2Ban 

Fail2Ban should stay on the host, not in Docker, because it needs access to system logs and iptables.

run:
```
sudo apt install fail2ban
```
enable jail:
```
sudo nano /etc/fail2ban/jail.local
```
in there, something like:
```
[sshd]
enabled = true
port = ssh
maxretry = 5
bantime = 1h
```

restart:
```
sudo systemctl restart fail2ban
```

check:
```
sudo fail2ban-client status sshd
```

## Start up everything

run:
```
docker compose up -d
```