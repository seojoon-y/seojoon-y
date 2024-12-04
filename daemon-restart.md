```
cd /etc/systemd/system/
```

# Restart & clear logs

```
sudo systemctl restart webhook.service && sudo systemctl daemon-reload && sudo systemctl stop systemd-journald && sudo rm -rf /var/log/journal/* /run/log/journal/* && sudo systemctl start systemd-journald
```

# See logs

```
sudo journalctl -u webhook.service
```


# Give it access to ENV
```
[Service]
EnvironmentFile=/etc/environment
```
