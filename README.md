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
raspitoune_webserver/
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

dont forget to run every command from `raspitoune_webserver/` !

## Routing

yoav.ch                → nginx → /var/www/website

schmoundcloud.yoav.ch  → nginx → /var/www/schmoundcloud

## Create Cloudflare tunnel

to connect raspitoune and Cloudflare, we first need to login and that will generate a certificate file (cert.pem)
in order to give Docker the ability to write the file in `./cloudflared`, we do:
```
chmod 777 cloudflared
```
(maybe there is a smarter way to just give write permissions to the "docker" user we created in the 1st step)
and then we can run:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:home/nonroot/.cloudflared \
  cloudflare/cloudflared tunnel login

```
because Cloudflare will create the .pem file in `/home/nonroot/.cloudflared` (in the Docker container)
that command will show a link to Cloudflare and, in the browser, we need to login to Cloudflare and choose to which domain the tunnel is routed

then we can create the tunnel:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared tunnel create mysite
```
it should output something like "Created tunnel mysite with id xxxx" and create a `cloudflared/mysite.json` file
(the .json file can also have the id, xxxx, of the tunnel as the filename i.e. `xxxx.json`)

### create Cloudflare DNS route

run this once:
```
docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel route dns mysite yoav.ch
```
should output "Added CNAME yoav.ch which will route to this tunnel tunnelID=xxxx"

```
docker run -it --rm \
  -v $(pwd)/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared tunnel route dns mysite schmoundcloud.yoav.ch
```
=> "Added CNAME schmoundcloud.yoav.ch which will route to this tunnel tunnelID=xxxx"
and we should see both DNS records in Cloudflare as CNAME → tunnel.

we then change the `config.yml` accordingly with the .json filename

(https://raspberrytips.com/cloudflare-selfhosted-website/ was really useful for these steps!)

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