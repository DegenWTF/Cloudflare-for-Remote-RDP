Cloudflared RDP — Full Documentation

Cloudflared Tunnel for Remote RDP — Full Documentation

---

Overview

This setup allows you to securely connect to a Windows PC’s Remote Desktop (RDP) over the internet without exposing your home IP or opening ports in the router. The tunnel runs via Cloudflare’s network and can be run as a persistent Windows service.

Key benefits:

No port forwarding needed

Encrypted traffic

Persistent service (starts on boot)

Optional Cloudflare Access for authentication and MFA

---

# > Prerequisites

Windows 10/11 (Home or Pro; RDP enabled)

Cloudflare account with a registered domain (e.g., exclesior.xyz)

Administrative access to Windows PC

Internet connection

---

# Step 1 — Install cloudflared

## Option A — Using PowerShell

1. Open PowerShell as Administrator.

2. Run:

iwr -useb https://developers.cloudflare.com/cloudflared/install.sh | iex

## Option B — Manual MSI

1. Download from Cloudflare: Cloudflared Windows

2. Run the .msi installer.

Verify installation:

cloudflared --version

---

# Step 2 — Login to Cloudflare

1. Run:

cloudflared tunnel login

2. Your browser opens → log into Cloudflare and select your domain.

3. This creates cert.pem in:

C:\Users\<YourUser>\.cloudflared\

---

# Step 3 — Create a Tunnel

## 3a — Create tunnel

cloudflared tunnel create myremote

Note the Tunnel ID and .json credentials file generated.

Example: <TUNNEL-ID>.json

## 3b — Map DNS name

cloudflared tunnel route dns myremote rdp.darkmonarch.xyz

## 3c — Create config.yml

File path:

C:\Users\<YourUser>\.cloudflared\config.yml

## Content (replace placeholders):

tunnel: <TUNNEL-ID> # from tunnel create output
credentials-file: C:\Users\<YourUser>\.cloudflared\<TUNNEL-ID>.json
logfile: C:\Users\Boss\.cloudflared\cloudflared.log

ingress:

- hostname: rdp.exclesior.com
  service: tcp://localhost:3389

# catch-all deny (recommended)

- service: http_status:404

## Notes:

logfile: enables logging for the service

Indentation must be 2 spaces per level

hostname must match your domain/subdomain

---

# Step 4 — Run Tunnel Interactively (Optional for testing)

cloudflared tunnel run <TUNNEL-ID>

Output shows successful connection: Registered tunnel connection ...

Press Ctrl + C to stop the interactive run

---

# Step 5 — Install Tunnel as Windows Service

1. Stop any running tunnel process (Ctrl + C + Task Manager → end cloudflared.exe if needed)

2. Uninstall old service (if stuck):

cloudflared service uninstall

3. Install service:

cloudflared service install

4. Start service:

Start-Service cloudflared

5. Check status:

Get-Service cloudflared

Status should show Running ✅

---

# Step 6 — Verify Logs

Tail logs to see tunnel activity:

Get-Content "C:\Users\Boss\.cloudflared\cloudflared.log" -Wait

You should see Registered tunnel connection ... messages

---

# Step 7 — Connect From Remote PC

7a — Install cloudflared on remote PC

7b — Start RDP Access

cloudflared access rdp --hostname rdp.darkmonarch.xyz

Opens browser for Cloudflare login if Access is enabled

Starts a local listener on localhost:3389

7c — Connect via Windows RDP

Open Remote Desktop Connection

Connect to:

localhost:3389

Use your host PC’s Windows username/password

---

# Step 8 — Optional: Cloudflare Access Security

Enable Cloudflare Access for the hostname: rdp.darkmonarch.xyz

Enforce:

Login via email / SSO

Multi-factor authentication (MFA)

This ensures only authorized users can connect

---

# Step 9 — Tips & Troubleshooting

If the tunnel service fails to start:

Ensure no foreground cloudflared process is running

Check config.yml indentation & Tunnel ID

Use Event Viewer: eventvwr.msc → Windows Logs → Application → Cloudflared

Logs help debug connection issues

Restart service after editing config.yml:

Restart-Service cloudflared

Foreground mode (cloudflared tunnel run) is useful for testing before using service

---

# Step 10 — Summary

Tunnel setup is persistent & secure

No open ports on your router

Can connect remotely via RDP using cloudflared access rdp

Fully logged and manageable via Windows service and log file

---

✅ You now have a reusable, step-by-step Cloudflare Tunnel setup for remote RDP.
