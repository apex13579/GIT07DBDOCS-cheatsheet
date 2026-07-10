| camp | purpose | why it matters |


### Systemctl

#### Service Control

| Command | What It Does |
|---|---|
| `systemctl status <service>` | Running state, recent logs, PID |
| `systemctl start <service>` | Start immediately |
| `systemctl stop <service>` | Stop immediately |
| `systemctl restart <service>` | Stop then start — reloads config |
| `systemctl enable <service>` | Auto-start on boot |
| `systemctl disable --now <service>` | Disable AND stop in one command |
| `systemctl list-units --type=service` | List all active services |
| `systemctl daemon-reload` | Reload systemd after editing unit files |

#### Logs via journald

| Command | What It Does |
|---|---|
| `journalctl -u <service> -f` | Follow live logs for a specific service |
| `journalctl -u <service> --since "1h ago"` | Logs from the past hour |
| `journalctl -p err -b` | Only errors since last boot |
| `journalctl --vacuum-time=7d` | Delete logs older than 7 days |
