# Run locally

## SSH

`nano ~/.ssh/config`

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

`sudo apt update && curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -`

`apt install nodejs && apt install npm`

## UFW

`ufw enable && ufw status && ufw allow ssh && ufw allow http && ufw allow https`

## PM2

`npm install -g pm2 && pm2 status`

## Nginx

`sudo apt install nginx`

`sudo nano /etc/nginx/sites-available/default`

```
server_name [SOMETHING eg. www].onionfist.com;
    location /hooks/ {
        proxy_pass http://localhost:9000/hooks/;
    }
	location / {
                proxy_pass http://localhost:5000;
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

## Cloudflare - DNS
Simply add the A record to DNS.

## Certbot - SSL
`sudo snap install core; sudo snap refresh core && sudo snap install --classic certbot && sudo ln -s /snap/bin/certbot /usr/bin/certbot && sudo certbot --nginx`

## Clone github using deploy key

`ssh-keygen -t rsa`

`cat ~/.ssh/id_rsa.pub` --> And copy this into Github deploy key, with only read access.

`touch ~/.ssh/config`

`chmod 600 ~/.ssh/config`

`nano ~/.ssh/config`

```
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa
```

`git clone git@github.com:onionfist/icerepo.git`

## Webhook

### In github:
Payload URL: `https://iceparty-game-server-na-1.onionfist.com/hooks/nodejs-app`
Content type: `application/json`
Secret: `SOME_SUPER_SECRET_PASSWORD`

### In VM:

`sudo apt install webhook`

`nano ~/hooks.json`

```
[
  {
    "id": "nodejs-app",
    "execute-command": "/root/redeploy-nodejs-app.sh",
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

For testing: `webhook -hooks hooks.json -hotreload -verbose -http-methods post`

### Webhook server

`cd /etc/systemd/system/`

`nano webhook.service`

```
[Unit]
Description= Github Webhook
Documentation=https://github.com/adnanh/webhook
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


solved by replacing /etc/webhook.conf with the content of hooks.json :-)



`sudo systemctl start webhook.service`

`sudo systemctl status webhook.service`

Useful info:
* Webhook
    * https://maximorlov.com/automated-deployments-from-github-with-webhook/#configure-webhook-endpoint
    * https://www.youtube.com/watch?v=hkuoX--XlxE