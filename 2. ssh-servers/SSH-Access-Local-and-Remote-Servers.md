# SSH Access: Local VMs and Remote Servers

This guide explains how to connect from Windows to Linux using SSH:

1. On the same local network.
2. Through VirtualBox NAT.
3. Across the internet to a server in another country.

## 1. What SSH does

SSH gives you a secure terminal on another computer.

Basic command:

```powershell
ssh USERNAME@SERVER_IP
```

Example:

```powershell
ssh ubuntu@192.168.1.201
```

SSH normally uses TCP port `22`.

## 2. Your current local lab

```text
Router:  192.168.1.1
Windows: 192.168.1.9
CentOS:  192.168.1.200
Ubuntu:  192.168.1.201
Subnet:  192.168.1.0/24
```

All machines are on the same local network.

VirtualBox Bridged mode makes each VM appear as a separate computer connected to your router.

## 3. Step-by-step: Windows to Ubuntu on the same network

### Step 1 — Confirm Ubuntu's IP

Run inside Ubuntu:

```bash
ip -br address
```

Expected:

```text
enp0s8   UP   192.168.1.201/24
```

### Step 2 — Confirm the SSH server

Run inside Ubuntu:

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

Check it:

```bash
sudo systemctl status ssh
```

Press `q` to exit the status screen.

### Step 3 — Test connectivity from Windows

Run in PowerShell:

```powershell
ping 192.168.1.201
```

### Step 4 — Connect from Windows

```powershell
ssh YOUR_UBUNTU_USERNAME@192.168.1.201
```

Example:

```powershell
ssh ubuntuvm@192.168.1.201
```

On the first connection, type:

```text
yes
```

Then enter the Ubuntu user's password.

### Step 5 — Exit the server

```bash
exit
```

## 4. Step-by-step: Windows to CentOS

### Step 1 — Confirm CentOS's IP

```bash
ip -br address
```

Expected bridged address:

```text
enp0s8   UP   192.168.1.200/24
```

### Step 2 — Install and start SSH if required

```bash
sudo dnf install -y openssh-server
sudo systemctl enable --now sshd
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### Step 3 — Connect from Windows

```powershell
ssh YOUR_CENTOS_USERNAME@192.168.1.200
```

## 5. Why local SSH works

```text
Windows 192.168.1.9
        |
        | Local Wi-Fi network
        |
Ubuntu 192.168.1.201
CentOS 192.168.1.200
```

It works because:

- All machines use the same `192.168.1.0/24` subnet.
- Every machine has a unique IP address.
- Bridged mode connects the VMs to the local network.
- The SSH server is running.
- TCP port `22` is allowed.

## 6. If the VM uses only VirtualBox NAT

Example:

```text
Windows: 192.168.1.9
Ubuntu:  10.0.2.15
```

Windows normally cannot connect directly to the guest's NAT address. Configure VirtualBox port forwarding:

### Step 1 — Shut down the VM

Do not save its state.

### Step 2 — Create the forwarding rule

Open:

```text
VirtualBox → VM Settings → Network → Adapter 1 → Advanced → Port Forwarding
```

Add:

| Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
|---|---|---|---:|---|---:|
| SSH | TCP | `127.0.0.1` | `2222` | Leave blank | `22` |

### Step 3 — Start the VM and connect

```powershell
ssh -p 2222 USERNAME@127.0.0.1
```

VirtualBox sends Windows port `2222` to the VM's SSH port `22`.

## 7. Server in another country: recommended method

Suppose:

```text
Your Windows computer: Pakistan
Ubuntu server/laptop:   Canada
```

Their local addresses belong to different private networks. You cannot connect to the Canadian server using its home address such as `192.168.5.20`.

The easiest secure solution is to connect both machines to a private VPN such as Tailscale.

```text
Windows ── encrypted Tailscale network ── Ubuntu in Canada
```

You normally do not need router port forwarding or a public IP.

## 8. Step-by-step: remote SSH using Tailscale

### Step 1 — Prepare SSH on the remote Ubuntu server

Run on the remote Ubuntu machine:

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

### Step 2 — Install Tailscale on remote Ubuntu

Official automated installer:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Connect it:

```bash
sudo tailscale up
```

Open the displayed login URL and authenticate.

### Step 3 — Get the remote Tailscale address

```bash
tailscale ip -4
```

Example result:

```text
100.80.20.5
```

You can also check:

```bash
tailscale status
```

### Step 4 — Install Tailscale on Windows

1. Open <https://tailscale.com/download/windows>.
2. Download and run the Windows installer.
3. Open Tailscale from the system tray.
4. Sign in with the same Tailscale account used for the remote server.

### Step 5 — Test the remote server

From Windows PowerShell:

```powershell
ping 100.80.20.5
```

### Step 6 — Connect using SSH

```powershell
ssh REMOTE_USERNAME@100.80.20.5
```

Example:

```powershell
ssh adnan@100.80.20.5
```

The traffic travels securely across the internet through the private Tailscale network.

## 9. Optional method: public IP and router forwarding

Use this only if you understand the security risk of exposing SSH to the internet.

The remote network must have:

- A reachable public IP address.
- No blocking Carrier-Grade NAT (CGNAT).
- Permission to configure its router.
- A firewall rule allowing SSH.
- SSH keys instead of weak password authentication.

Example forwarding rule on the remote router:

```text
Public-IP:2222 → Remote Ubuntu 192.168.5.20:22
```

Connect from Windows:

```powershell
ssh -p 2222 USERNAME@PUBLIC_IP
```

Example:

```powershell
ssh -p 2222 adnan@203.0.113.10
```

Do not copy the example public IP; use the real public IP of the remote network.

For personal use, prefer the VPN method because it avoids exposing SSH publicly.

## 10. Recommended SSH-key authentication

Passwords can be guessed. SSH keys are safer.

### Step 1 — Generate a key on Windows

```powershell
ssh-keygen -t ed25519
```

Press Enter to accept the default file location. Use a passphrase for better protection.

### Step 2 — Copy the public key to Linux

Display the Windows public key:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

On the Linux server, add that complete single line to:

```text
~/.ssh/authorized_keys
```

Ensure permissions are correct:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Step 3 — Test key login

```powershell
ssh USERNAME@SERVER_IP
```

Do not disable password authentication until key login has been tested successfully in a second terminal.

## 11. Quick troubleshooting

### Ping works but SSH fails

Check SSH on Linux:

```bash
sudo systemctl status ssh
```

For CentOS:

```bash
sudo systemctl status sshd
```

### Connection refused

The machine is reachable, but SSH is not running or port `22` is blocked.

### Connection timed out

Common causes:

- Wrong IP address.
- Machines are not connected by routing or VPN.
- Firewall blocks the traffic.
- Router forwarding is incorrect.
- The remote ISP uses CGNAT.

### Permission denied

The network connection works, but the username, password, or SSH key is incorrect.

### Verify port 22 from Windows

```powershell
Test-NetConnection SERVER_IP -Port 22
```

Example:

```powershell
Test-NetConnection 192.168.1.201 -Port 22
```

## 12. Quick decision guide

| Situation | Best connection method |
|---|---|
| Windows and VM on the same LAN | Bridged IP |
| VM uses only VirtualBox NAT | VirtualBox port forwarding |
| Personal server in another country | Tailscale/private VPN |
| Public production server | Public IP with strict firewall and SSH keys |
| Private cloud server | VPN, bastion host, or managed access service |

## Final checklist

- [ ] The Linux server has a known IP address.
- [ ] Every local machine has a unique IP.
- [ ] `openssh-server` is installed.
- [ ] The SSH service is running.
- [ ] Port `22` is permitted on the intended network path.
- [ ] Same-LAN devices use the bridged IP.
- [ ] Remote-country devices use a VPN such as Tailscale.
- [ ] Public router forwarding is avoided unless necessary.
- [ ] SSH keys are configured for remote access.
- [ ] Root login is not used for normal administration.

## Official references

- Ubuntu OpenSSH Server: <https://ubuntu.com/server/docs/openssh-server/>
- Tailscale Linux installation: <https://tailscale.com/docs/install/linux>
- Tailscale Windows installation: <https://tailscale.com/docs/install/windows>
- Tailscale CLI: <https://tailscale.com/docs/reference/tailscale-cli>

