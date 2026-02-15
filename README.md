# Vathsalya Project - Cloudflare Tunnel Setup Guide

This guide documents how to run the Vathsalya project locally and deploy it to a custom domain using Cloudflare Tunnel.

## Project Overview

This is a static website project with HTML, CSS, and JavaScript files serving content for the Vathsalya organization.

## Prerequisites

- Python 3.x (for local development server)
- Cloudflare account
- Domain registered with GoDaddy (or any registrar)
- Domain added to Cloudflare with nameservers configured

## Setup Instructions

### 1. Running the Project Locally

Start a local Python HTTP server to serve the static files:

```powershell
python -m http.server 8000
```

The site will be available at: http://localhost:8000/

### 2. Installing Cloudflare Tunnel (cloudflared)

Download the cloudflared executable directly:

```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri "https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe" -OutFile "cloudflared.exe"
```

Verify the installation:

```powershell
.\cloudflared.exe --version
```

### 3. Authenticate with Cloudflare

Log in to your Cloudflare account (this will open a browser):

```powershell
.\cloudflared.exe tunnel login
```

This creates a certificate file at: `C:\Users\<username>\.cloudflared\cert.pem`

### 4. Create a Tunnel

Create a named tunnel for your project:

```powershell
.\cloudflared.exe tunnel create vathsalya-project
```

This will:
- Create a tunnel with a unique ID
- Save credentials to: `C:\Users\<username>\.cloudflared\<tunnel-id>.json`

**Tunnel ID for this project:** `58148887-ea2c-4422-914a-2e6ab3a136eb`

### 5. Configure the Tunnel

Create a `config.yml` file in the project directory with the following content:

```yaml
tunnel: 58148887-ea2c-4422-914a-2e6ab3a136eb
credentials-file: C:\Users\shail\.cloudflared\58148887-ea2c-4422-914a-2e6ab3a136eb.json

ingress:
  - hostname: vathsalyasevasamsthe.com
    service: http://localhost:8000
  - hostname: www.vathsalyasevasamsthe.com
    service: http://localhost:8000
  - service: http_status:404
```

### 6. Configure DNS Routes

Create DNS CNAME records that route traffic to your tunnel:

```powershell
.\cloudflared.exe tunnel route dns vathsalya-project vathsalyasevasamsthe.com
.\cloudflared.exe tunnel route dns vathsalya-project www.vathsalyasevasamsthe.com
```

### 7. Configure Cloudflare DNS (CRITICAL STEP)

Before the tunnel will work, you must configure your domain's nameservers and DNS records in Cloudflare:

#### Option A: Add Domain to Cloudflare (Recommended)

1. **Add your domain to Cloudflare**:
   - Go to https://dash.cloudflare.com/
   - Click "Add a Site"
   - Enter your domain: vathsalyasevasamsthe.com
   - Select the Free plan
   - Click "Continue"

2. **Update nameservers at GoDaddy**:
   - Cloudflare will provide you with 2 nameservers (e.g., `anna.ns.cloudflare.com` and `bob.ns.cloudflare.com`)
   - Log into your GoDaddy account
   - Go to your domain settings
   - Find "Nameservers" section and click "Change"
   - Select "Custom nameservers"
   - Enter the Cloudflare nameservers provided
   - Save changes
   - **Note**: This can take 24-48 hours to fully propagate

3. **Configure DNS in Cloudflare**:
   - Once Cloudflare detects your domain (check status in dashboard)
   - Go to DNS > Records
   - Delete any existing A/AAAA records for root domain and www subdomain
   - The CNAME records should already be created from step 6
   - Verify they point to: `58148887-ea2c-4422-914a-2e6ab3a136eb.cfargotunnel.com`

#### Option B: Update DNS Manually (If domain already in Cloudflare)

If your domain is already added to Cloudflare:

1. Log into Cloudflare Dashboard: https://dash.cloudflare.com/
2. Select your domain: vathsalyasevasamsthe.com
3. Go to DNS > Records
4. Find the CNAME records created in step 6:
   - `vathsalyasevasamsthe.com` should point to `58148887-ea2c-4422-914a-2e6ab3a136eb.cfargotunnel.com`
   - `www.vathsalyasevasamsthe.com` should point to `58148887-ea2c-4422-914a-2e6ab3a136eb.cfargotunnel.com`
5. If they point to a different tunnel ID, update them to use the correct tunnel ID
6. Delete any A or AAAA records that might be conflicting

### 8. Start the Tunnel

Run the tunnel to connect your local server to your domain:

```powershell
.\cloudflared.exe tunnel --config config.yml run vathsalya-project
```

### 9. Verify Your Website is Live

Test that your domain is serving the correct content:

```powershell
# Test the domain
curl https://vathsalyasevasamsthe.com
```

Or simply open your browser and visit:
- https://vathsalyasevasamsthe.com
- https://www.vathsalyasevasamsthe.com

You should see your local project content, not GoDaddy's default page.

## Access Your Website

Once the tunnel is running and DNS is properly configured, your website is accessible at:
- https://vathsalyasevasamsthe.com
- https://www.vathsalyasevasamsthe.com

## File Structure

```
Vathsalya/
├── about.html
├── contact.html
├── donate.html
├── gallery.html
├── index.html
├── programs.html
├── script.js
├── styles.css
├── volunteer.html
├── config.yml                  # Cloudflare Tunnel configuration
├── cloudflared.exe            # Cloudflare Tunnel client
└── README.md                  # This file
```

## Important Notes

1. **Keep the local server running**: The Python HTTP server must be running on port 8000 for the tunnel to work.
2. **Keep the tunnel running**: The cloudflared tunnel command must be running to maintain the connection.
3. **Certificate security**: Never share your `.cloudflared` directory or the tunnel credentials file.
4. **Nameservers**: Ensure your domain's nameservers are pointed to Cloudflare in your GoDaddy account.
5. **DNS Configuration**: The tunnel will only work after you've properly configured your domain's nameservers to point to Cloudflare and the DNS records are set correctly.
6. **Tunnel ID**: The tunnel ID in config.yml must match the credentials file and the tunnel you're running.

## Stopping the Services

To stop the local server:
- Press `Ctrl+C` in the terminal running the Python HTTP server

To stop the tunnel:
- Press `Ctrl+C` in the terminal running cloudflared

## Troubleshooting

### Tunnel not connecting
- Verify the local server is running on port 8000
- Check that the tunnel ID in config.yml matches your tunnel
- Ensure DNS records are properly configured in Cloudflare dashboard

### Domain not accessible or showing wrong content

**If you see GoDaddy's default page instead of your project:**

1. **Check nameservers**: Your domain's nameservers must point to Cloudflare
   - Log into GoDaddy
   - Go to domain settings
   - Verify nameservers are Cloudflare's (not GoDaddy's default)
   
2. **Verify DNS records in Cloudflare**:
   - Go to https://dash.cloudflare.com/
   - Select your domain
   - Go to DNS > Records
   - Ensure CNAME records point to: `58148887-ea2c-4422-914a-2e6ab3a136eb.cfargotunnel.com`
   - Delete any A/AAAA records pointing to GoDaddy's servers

3. **Check tunnel is running**:
   ```powershell
   Get-Process cloudflared -ErrorAction SilentlyContinue
   ```

4. **Verify tunnel configuration**:
   ```powershell
   .\cloudflared.exe tunnel list
   .\cloudflared.exe tunnel info vathsalya-project
   ```

5. **Wait for DNS propagation**: Changes can take 5-60 minutes to propagate globally

**If you have multiple tunnels:**
- Check which tunnel your DNS is routed to
- Delete or stop unused tunnels to avoid confusion
- Ensure config.yml uses the correct tunnel ID and credentials file

### Certificate warnings on Windows
The warning "cloudflared does not support loading the system root certificate pool on Windows" is normal and does not affect functionality.

## Additional Resources

- [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [Cloudflare Dashboard](https://dash.cloudflare.com/)
- [GitHub: cloudflared](https://github.com/cloudflare/cloudflared)

## Support

For issues specific to this project, contact the development team.
For Cloudflare-specific issues, refer to [Cloudflare Support](https://support.cloudflare.com/).
