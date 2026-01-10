# pfSense SSL/TLS Certificate Setup with Let's Encrypt & Cloudflare

Complete guide for setting up trusted SSL certificates on pfSense using ACME and Cloudflare DNS validation.

## Overview

This setup uses: 
- **Let's Encrypt** for free, trusted SSL certificates
- **Cloudflare DNS validation** to prove domain ownership
- **ACME package** in pfSense for automatic certificate management and renewal

## Prerequisites

- pfSense firewall with internet access
- Domain managed by Cloudflare (DNS only, orange cloud OFF)
- Subdomain pointing to pfSense (e.g., `pfsense.yourdomain.com`)

---

## Step 1: Get Cloudflare API Token

1. Log into **Cloudflare Dashboard**
2. Go to **My Profile** → **API Tokens**
3. Click **Create Token**
4. Use **Edit zone DNS** template
5. Configure permissions:
   - **Permissions**: Zone → DNS → Edit
   - **Zone Resources**: Include → Specific zone → `yourdomain.com`
6. Click **Continue to summary** → **Create Token**
7. **Copy the API token** (save it - you won't see it again!)

---

## Step 2: Install ACME Package in pfSense

1. Go to **System** → **Package Manager** → **Available Packages**
2. Search for **"acme"**
3. Click **Install**
4. Confirm installation

---

## Step 3: Configure ACME Account (Production)

1. Go to **Services** → **Acme** → **Account keys**
2. Click **+ Add**
3. Configure:
   - **Name**: `LetsEncrypt-Production`
   - **Description**: Let's Encrypt Production Account
   - **ACME Server**: **Let's Encrypt Production ACME v2** (NOT staging!)
   - **E-Mail Address**: Your email
4. Click **Create new account key**
5. Click **Register ACME account key**
6. Click **Save**

⚠️ **Important**: Make sure you're using **Production**, NOT Staging!

---

## Step 4: Create Certificate

1. Go to **Services** → **Acme** → **Certificates**
2. Click **+ Add**
3. Configure certificate settings:
   - **Name**:  `pfsense-cert` (or any descriptive name)
   - **Description**: pfSense WebGUI Certificate
   - **Status**: ✅ Active (checked)
   - **Acme Account**: Select your production account from Step 3
   - **Private Key**: 384-bit ECDSA (or 2048-bit RSA)
   - **OCSP Must Staple**: Leave unchecked

4. **Domain SAN list** - Click **+ Add**: 
   - **Mode**:  Enabled
   - **Domainname**: `pfsense.yourdomain.com` (your pfSense subdomain)
   - **Method**: **DNS-Cloudflare**
   - **Key**:  Paste your Cloudflare API token from Step 1
   - **Account email**: Your email address

5. Scroll down and click **Save**
6. Click **Issue/Renew** button
7. Wait for success message (should see "Cert success" and "Reload successful")

---

## Step 5: Apply Certificate to WebGUI

1. Go to **System** → **Advanced** → **Admin Access**
2. Under **SSL/TLS Certificate** dropdown, select your new certificate (e.g., `pfsense-cert`)
3. Scroll down to **Alternate Hostnames**
4. Add your domain:  `pfsense.yourdomain.com`
   - This prevents DNS Rebind protection errors
5. Click **Save**
6. Browser will disconnect briefly

---

## Step 6: Verify HTTPS Works

1. Clear browser cache or use incognito/private window
2. Navigate to:  `https://pfsense.yourdomain.com`
3. Should see: 
   - ✅ Secure lock icon
   - ✅ No certificate warnings
   - ✅ Valid certificate from Let's Encrypt

---

## Optional: Apply Certificate to OpenVPN

If you're using OpenVPN:

1. Go to **VPN** → **OpenVPN** → **Servers**
2. Edit your OpenVPN server
3. Under **TLS Configuration**, select your ACME certificate
4. Save and restart OpenVPN service

---

## Troubleshooting

### Certificate shows as "Staging" or not trusted
- **Problem**: Used Let's Encrypt Staging instead of Production
- **Solution**: In Account keys, verify ACME Server is set to "Production ACME v2", then re-issue certificate

### Browser shows "Not Secure" after setup
- **Problem**: Browser cached old certificate
- **Solution**: 
  - Hard refresh:  `Ctrl + Shift + R` (Windows/Linux) or `Cmd + Shift + R` (Mac)
  - Clear site data: `chrome://settings/content/all` (search for your domain)
  - Reboot computer if needed

### "DNS Rebind attack detected" error
- **Problem**:  Hostname not whitelisted
- **Solution**: Add domain to **System** → **Advanced** → **Admin Access** → **Alternate Hostnames**

### Certificate won't issue
- **Problem**: DNS validation failing
- **Check**:
  - Cloudflare API token has correct permissions
  - Domain exists in Cloudflare
  - pfSense can reach Cloudflare API (check firewall rules)

---

## Certificate Renewal

ACME automatically renews certificates every 60 days. You can manually renew anytime:

1. Go to **Services** → **Acme** → **Certificates**
2. Click **Issue/Renew** next to your certificate

---

## Notes

- Certificates are valid for 90 days
- Auto-renewal happens at 60 days
- Let's Encrypt has rate limits (5 certificates per week per domain)
- Keep your Cloudflare API token secure
- DNS-only mode (orange cloud OFF) is required for direct access via VPN

---

## Summary of URLs Used

- **Cloudflare Dashboard**: https://dash.cloudflare.com
- **pfSense WebGUI**: https://pfsense.yourdomain. com (replace with your domain)
- **Let's Encrypt Docs**: https://letsencrypt.org/docs/

---

**Setup Date**: 2026-01-10  
**Domain Used**: pfsense.example.com (update with your domain)
