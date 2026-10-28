# Automated Server Security Hardening Tool

**Production-ready Bash script automating initial server hardening and infrastructure security deployment for Debian and Ubuntu Linux distributions.** This project completely transforms a default, unprotected cloud server instance into a highly secure, production-hardened environment within minutes.

---

## 🛠️ Configuration Variables / Настройка переменных

Before running the script, open `security_setup.sh` and edit the variable block at the very top of the file.

> ⚠️ **Security Notice / Важное примечание:** Insecure passwordless `sudo` (`NOPASSWD`) has been completely removed to prevent unauthorized root escalation if a user account is compromised. The script now securely provisions a standard administrative account. You **must write a plain-text password** in the `USER_PASSWORD` variable. The script will automatically intercept this string and encrypt it using the secure SHA-512 hashing algorithm (`openssl passwd -6`) directly during the account initialization phase. Your new user will safely prompt for this exact password when running terminal `sudo` tasks.

| Parameter Variable | Required Input Example | Operational Purpose |
| - | - | - |
| `NEW_USER` | `"cooladmin"` | Name of the secure sudo-enabled management account. |
| `USER_PASSWORD` | `"SuperPassword123!"` | **[REQUIRED]** The secure plain-text password used to authorize administrative `sudo` commands. |
| `SSH_PORT` | `"2222"` | The target custom TCP port where the SSH daemon will accept connections. |
| `SSH_PUBLIC_KEY` | `"ssh-ed25519 AAA..."` | The raw public cryptographic key sequence generated on your local client machine. |
| `TRUSTED_IP` | `"95.21.34.12"` | Optional: A trusted static IP address exempted from automated ban lists (leave empty `""` if none). |

---

## 🛡️ Full Shield Stack / Из чего состоит защита

This tool provisions a 5-layer automated security stack designed for solid, multi-tiered server protection without unnecessary DevOps complexity:

* **Intrusion Prevention (Fail2Ban + SSH Keys):** Remote password authentication over SSH is completely disabled (`PasswordAuthentication no`). It is mathematically impossible to brute-force your way into the server via remote dictionary or password-guessing attacks. Concurrently, the Fail2Ban engine actively monitors the modern systemd journal backend. If any rogue automated bot triggers repeated access failures on your custom SSH port, Fail2Ban injects an immediate firewall rule at the kernel level to block the source IP for 1 hour.
* **Network Perimeter Shield (UFW Firewall):** Deploys Uncomplicated Firewall (UFW) configured with a strict **Default-Deny Inbound** posture. Every single entry port to your server is completely locked down and invisible from the public internet, except for your explicitly approved custom `SSH_PORT`.
* **Obscurity Layer (Port Rotation):** Mitigates 99% of global automated bot scanning noise by relocating the SSH service away from the heavily targeted default port 22 to your obscure custom port.
* **Identity Separation (Root Lock):** Direct root logins are fully terminated (`PermitRootLogin no`). Administrative control is shifted exclusively to your new isolated user account via the secure `sudo` group mechanism.
* **Log Tracing Evacuation (Command History Wipe):** Eliminates internal footprint vectors. The script concludes by immediately invoking `history -c && history -w` inside the active root shell. This guarantees your raw `USER_PASSWORD` variable is instantly wiped from current shell memory and never written to the `.bash_history` log file where it could be exposed.

---

## 📈 Architectural Workflow / Схема движения трафика

```
Incoming Network Request / Входящий запрос из интернета
             |
             v
UFW Firewall  -------> [ BLOCKED / ЗАКРЫТО ] (Filters out unauthorized inbound traffic)
             |
             v (Approved Custom Port / Разрешенный порт)
Fail2Ban Daemon  -----> [ BANNED / БАН ] (Monitors systemd journal / Bans attacking IPs)
             |
             v (If IP is clean and safe / Если IP чистый)
SSH Daemon  --------> [ SSH-Key Verification ] (Root login and passwords are DISABLED)
             |
             v
Secure User Shell / Доступ разрешен!
```

---

## 🚀 Installation & Deployment Guide / Инструкция по запуску

### Phase 1: Generate an SSH Key Pair on Local Workstation
If you do not have an active secure shell key on your host client machine, generate a modern elliptic curve algorithm pair using your local terminal window (Command Prompt, PowerShell, or macOS Terminal):
```bash
ssh-keygen -t ed25519 -C "deployment_key"
```
Press Enter to accept default storage paths. To extract the public key string needed for the script variable, display the output and copy it entirely:
```bash
cat ~/.ssh/id_ed25519.pub
```

### Phase 2: Upload and Configure Script on Server
1. Establish a temporary connection to your fresh cloud server as the root user.
2. Initialize an empty script file using a command-line editor:
   ```bash
   nano security_setup.sh
   ```
3. Copy the script source code completely and paste it inside the editor window.
4. Modify the `NEW_USER`, `USER_PASSWORD`, `SSH_PORT`, and `SSH_PUBLIC_KEY` variables to mirror your real target values.
5. Save changes and close the text editor: Press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

### Phase 3: Script Execution
Grant explicit execution permissions to your file and run the script under the root context:
```bash
chmod +x security_setup.sh
sudo ./security_setup.sh
```

### Phase 4: Post-Hardening Verification Procedure
**Critical Warning:** Keep your current root terminal session open! Do not close this window until connection validation finishes successfully.

1. Launch a completely new, separate terminal window instance on your local computer workstation.
2. Attempt an authenticated connection using the newly defined port and user attributes:
   ```bash
   ssh -p 2222 cooladmin@YOUR_SERVER_IP
   ```
3. If configured correctly, your system will establish access instantly via your SSH key without prompting for an account password.
4. Run a verification check to ensure administrative operations function correctly:
   ```bash
   sudo apt update
   ```
5. The system will ask for a password. Type the exact password you specified in the `USER_PASSWORD` variable at the top of the script. Once verified, you can close all active terminal instances safely.

---

## 🔧 Administration & Maintenance / Обслуживание системы

### Network Firewall Management
To manually open ports for incoming traffic later on (e.g., exposing a web service), use the following syntax tools:
```bash
# Expose a custom container application port
sudo ufw allow 8080/tcp comment 'Custom Web Service'

# Re-read configurations and check active ports
sudo ufw status verbose
```

### Monitoring Security Bans
To monitor real-time intruder events blocked by your automated intrusion handler, use these diagnostic utilities:
```bash
# Query active status metrics for the SSH daemon jail
sudo fail2ban-client status sshd

# Manually remove an accidental ban from a trusted IP address
sudo fail2ban-client set sshd unbanip 192.168.1.100
```
