```
cd /etc/systemd/system/
```

# how to restart systemd webhook.service and the daemon

```
sudo systemctl restart webhook.service && sudo systemctl daemon-reload
```

# See logs

```
sudo journalctl -u webhook.service
```


# Clear logs

```
sudo systemctl stop systemd-journald && sudo rm -rf /var/log/journal/* && sudo systemctl start systemd-journald

```

# Give it access to ENV
```
[Service]
EnvironmentFile=/etc/environment
```
