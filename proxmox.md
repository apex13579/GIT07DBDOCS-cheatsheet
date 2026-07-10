| camp | purpose | why it matters |


#### VM Management

| Command | What It Does |
|---|---|
| `qm list` | List all VMs and their status |
| `qm start <vmid>` | Start a VM by ID |
| `qm shutdown <vmid>` | Graceful shutdown via guest agent |
| `qm stop <vmid>` | Force stop — hard power off |
| `qm config <vmid>` | Full VM config — cores, RAM, disks, network |
| `qm set <vmid> --memory 4096` | Change RAM allocation in MB |
| `qm terminal <vmid>` | Attach to VM serial console |

#### LXC Containers

| Command | What It Does |
|---|---|
| `pct list` | List all LXC containers |
| `pct start <ctid>` | Start an LXC container |
| `pct enter <ctid>` | Open a shell inside the container |
| `pct config <ctid>` | Show container config |
| `pct set <ctid> --memory 512` | Update container memory limit |

#### Storage & Backup

| Command | What It Does |
|---|---|
| `pvesm status` | All storage backends and usage |
| `vzdump <vmid> --storage <pool>` | Backup a VM or container |
| `qmrestore <backup> <vmid>` | Restore a VM from a vzdump backup |

#### Node Info

| Command | What It Does |
|---|---|
| `pveversion` | Show Proxmox VE version |
| `pve-firewall status` | Check Proxmox firewall state |

---

