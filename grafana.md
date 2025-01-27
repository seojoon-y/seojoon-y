# Postgres:

I wasted 1 hour cause of this.

`verify-ca`

`TLS/SSL Client Certificate` : BLANK

`TLS/SSL Root Certificate` : FILL IN

`TLS/SSL Client Key` : BLANK

# Systemd

`cd /etc/systemd/system`

`sudo systemctl status grafana-server`

`sudo systemctl restart grafana-server`

`sudo journalctl -u grafana-server -n 100`
