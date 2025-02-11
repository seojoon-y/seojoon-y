# Run locally

## SSH

`ssh-copy-id root@IP.IP.IP.IP`

OR

`vim ~/.ssh/config`

```
HOST 137.184.105.149
  User root
  Port 22
  IdentitiesOnly yes
  IdentityFile ~/.ssh/dododoio_digitalocean
```

`ssh root@IP:IP:IP:IP`

# Run remotely

## Node

```
sudo apt update && curl -sL https://deb.nodesource.com/setup_23.x | sudo -E bash - && sleep 1 && sudo apt install nodejs -y
```

## Firewall (UFW)

```
sudo apt install -y ufw && ufw enable && ufw status && ufw allow ssh && ufw allow http && ufw allow https
```

## Node Process Manager (PM2)

```
npm install -g pm2 && pm2 status
```

## Reverse Proxy (Nginx)

```
sudo apt install -y nginx

sudo vim /etc/nginx/sites-available/default
```

```
server_name staging-api.onionfist.com;
	location /hooks/ {
                proxy_pass http://localhost:9000/hooks/;
	}
	location / {
                proxy_pass http://localhost:4000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_redirect off;
    }
```

`sudo nginx -t && sudo service nginx restart`

## Cloudflare - DNS
Simply add the A record to DNS.

## Certbot - SSL
Ubuntu:

`sudo snap install core; sudo snap refresh core && sudo snap install --classic certbot && sudo ln -s /snap/bin/certbot /usr/bin/certbot && sudo certbot --nginx`

Or in case of Debian:

```
sudo apt update && sudo apt install -y snapd
sudo snap install core && sudo snap refresh core && sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot && sudo certbot --nginx
```

## Clone github using deploy key

```
ssh-keygen -t rsa && cat ~/.ssh/id_rsa.pub
```

--> And copy this into Github deploy key, with only read access.

```
touch ~/.ssh/config && chmod 600 ~/.ssh/config && vim ~/.ssh/config
```

```
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa
```

```
sudo apt install -y git && git clone git@github.com:onionfist/icerepo.git
```

```
cd icerepo && npm install --verbose
```

## Webhook

### In github:

Payload URL: `https://iceparty-game-server-na-1.onionfist.com/hooks/main-hook`

Content type: `application/json`

Secret: `SOME_SUPER_SECRET_PASSWORD`

### In VM:
`vim redeploy.sh`

Option 1:
```
#!/bin/sh

# 1. Fetch the latest code from remote
git reset --hard
git pull -f origin production

# 2. Install dependencies
npm install

# 3. (Optional) Build step that compiles code, bundles assets, etc.
npm run build

# 4. Restart application
pm2 restart api-server --update-env
```

Option 2: (could be better for ENV variable refresh)
```
#!/bin/sh

# 1. Fetch the latest code from remote
git reset --hard
git pull -f origin production
npm install
pm2 kill
pm2 flush
pm2 start ecosystem.config.cjs --update-env
```


Might have to add `npm uninstall sharp && npm install --platform=linux --arch=x64 sharp`

`chmod +x redeploy.sh`

`sudo apt install webhook`

`vim ~/hooks.json`

```
[
  {
    "id": "main-hook",
    "execute-command": "/root/redeploy.sh",
    "command-working-directory": "/root/icerepo",
    "response-message": "Delivered",
    "trigger-rule": {
      "and": [
        {
          "match": {
            "type": "payload-hmac-sha256",
            "secret": "SOME_SUPER_SECRET_PASSWORD",
            "parameter": {
              "source": "header",
              "name": "X-Hub-Signature-256"
            }
          }
        },
        {
          "match": {
            "type": "value",
            "value": "refs/heads/production",
            "parameter": {
              "source": "payload",
              "name": "ref"
            }
          }
        }
      ]
    }
  }
]
```

For testing: `webhook -hooks hooks.json -hotreload -verbose -http-methods post` and then make sure to `reboot`

### Webhook server

`cd /etc/systemd/system/`

`vim webhook.service`

```
[Unit]
Description= Github Webhook
Documentation=https://github.com/seojoon-y/seojoon-y/blob/main/deploy.md
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5
ExecStart=webhook -verbose -hotreload -hooks /root/hooks.json -port 9000 -ip "127.0.0.1" -http-methods post

[Install]
WantedBy=multi-user.target
```

`chmod 0644 webhook.service`

`reboot`

`cd /etc/systemd/system/`

`sudo systemctl start webhook.service`

`sudo systemctl status webhook.service`

`cd`

Start pm2: `cd icerepo && pm2 start` OR `pm2 start --only api-server`

Auto-start pm2 on reboot: `pm2 startup`

You are all set!

Useful info:
* Webhook
    * https://maximorlov.com/automated-deployments-from-github-with-webhook/#configure-webhook-endpoint
    * https://www.youtube.com/watch?v=hkuoX--XlxE


# Set environment variable:

`sudo vim /etc/environment`

Update pm2 after changing env variables: `pm2 restart all --update-env`

A more reliable way to fix it is:
`pm2 kill`
`pm2 start --only app-id`

# Makefile

```
deploy:
	make echo-deploy-start
	git checkout production
	git merge development
	git push
	git checkout development
	make echo-deploy-end

echo-deploy-start:
	@echo "\033[92m -- DEPLOY STARTING -- \033[0m"

echo-deploy-end:
	@echo "\033[92m -- DEPLOY FINISHED -- \033[0m"

```

# Log too big
`du -h --max-depth=1`

`pm2 flush` and `reboot`

`pm2 install pm2-logrotate`

# Problem: Cloudflare Redirct 301 many times

Check SSL/TLS Settings in Cloudflare:

Go to SSL/TLS in your Cloudflare dashboard.

Ensure the SSL mode is set to Full or Full (Strict). Avoid using Flexible, as it can cause redirect loops.

# psql
sudo apt install -y postgresql-client

