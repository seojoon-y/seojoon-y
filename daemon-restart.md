# how to restart systemd webhook.service and the daemon

```
sudo systemctl restart webhook.service
sudo systemctl daemon-reload
```

# Error debug

```
sudo journalctl -u webhook.service
```
