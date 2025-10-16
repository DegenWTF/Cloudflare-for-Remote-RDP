This repository provides a step-by-step guide to set up a secure Cloudflare Tunnel to access a Windows PC's Remote Desktop (RDP) from anywhere over the internet.

Overview

Securely expose RDP without opening router ports.

Use Cloudflare Access for authentication and MFA.

Run the tunnel as a persistent Windows service.


Prerequisites

Windows 10/11 PC (Home or Pro) with RDP enabled.

Cloudflare account with a registered domain.

Administrative access to the PC.


Step 1: Install cloudflared

Option A: PowerShell

iwr -useb https://developers.cloudflare.com/cloudflared/install.sh | iex

Option B: Manual MSI

1. Download from Cloudflare


2. Run the .msi installer.



Verify installation:

cloudflared --version

Step 2: Login to Cloudflare

cloudflared tunnel login

Opens browser for authentication.

Select your domain.

Downloads cert.pem to C:\Users\<YourUser>\.cloudflared\.


Step 3: Create a Tunnel

3a: Create the tunnel

cloudflared tunnel create myremote

Note the Tunnel ID and JSON credentials file.


3b: Map DNS name

cloudflared tunnel route dns myremote rdp.yourdomain.com

3c: Create config.yml

Path: C:\Users\<YourUser>\.cloudflared\config.yml

tunnel: <TUNNEL-ID>
credentials-file: C:\Users\<YourUser>\.cloudflared\<TUNNEL-ID>.json
logfile: C:\Users\<YourUser>\.cloudflared\cloudflared.log

ingress:
  - hostname: rdp.yourdomain.com
    service: tcp://localhost:3389

  - service: http_status:404

Ensure 2-space indentation.


Step 4: Run Tunnel Interactively (Optional)

cloudflared tunnel run <TUNNEL-ID>

Ctrl + C to stop.


Step 5: Install Tunnel as Windows Service

1. Stop any running foreground tunnel.


2. Uninstall old service (if needed):



cloudflared service uninstall

3. Install service:



cloudflared service install

4. Start service:



Start-Service cloudflared

5. Check status:



Get-Service cloudflared

Step 6: Monitor Logs

Get-Content "C:\Users\<YourUser>\.cloudflared\cloudflared.log" -Wait

Step 7: Connect from Remote PC

1. Install cloudflared.


2. Run:



cloudflared access rdp --hostname rdp.yourdomain.com

3. Open Remote Desktop Connection → connect to localhost:3389.



Step 8: Cloudflare Access Security (Optional)

Enforce login via email/SSO.

Enable MFA.


Step 9: Troubleshooting

Ensure no other cloudflared processes are running.

Check config.yml indentation and Tunnel ID.

Use Event Viewer: Windows Logs → Application → Cloudflared.

Restart service after edits: Restart-Service cloudflared.


Summary

Tunnel runs persistently and securely.

RDP accessible remotely via Cloudflare Access.

No router port forwarding required.

Logs available for monitoring.

![wtf](https://github.com/user-attachments/assets/e0c9971a-f4c1-4592-a93a-8ee46d304d54)
