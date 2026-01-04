# INFRASTRUCTURE CONFIGURATION GUIDE - FOR LOCAL USE ONLY

This document is stored locally and is **NOT** committed to GitHub (see .gitignore).
This ensures sensitive credentials remain secure while maintaining organized documentation.

## Using This Setup

### Step 1: Fill in Your Credentials
1. Open `credentials.local.md` (stored locally on your machine)
2. Replace all `[PLACEHOLDER]` values with actual credentials:
   - Server usernames and passwords
   - SSH key paths
   - Database credentials
   - API keys and secrets

### Step 2: Keep It Secure
- **Never** commit credentials to GitHub
- Store `credentials.local.md` locally only
- Share credentials through secure channels only
- Use strong, unique passwords

### Step 3: Access Your Servers
Once credentials are filled in, use them to SSH into servers:

```bash
# ERP Server
ssh -i /path/to/key username@192.168.20.10

# Pristine Planet Server
ssh -i /path/to/key username@192.168.20.20

# Reverse Proxy
ssh -o StrictHostKeyChecking=no root@192.168.20.21
# Default password: proxy (CHANGE THIS)
```

### Step 4: Manage Proxy Configuration
```bash
# From proxy server
ssh root@192.168.20.21

# View active configuration
cat /etc/nginx/sites-enabled/erp.tradifyglobal.com
cat /etc/nginx/sites-enabled/pristineplanet.eco

# Update from GitHub
git clone https://github.com/tradifyglobal/PristinePlanet-NGNIX-Proxy.git
cp PristinePlanet-NGNIX-Proxy/nginx-config/*.conf /etc/nginx/sites-available/
nginx -t && systemctl reload nginx
```

## Server Quick Reference

| Server | IP | Service | Port | Domain |
|--------|----|---------|----|--------|
| ERP Backend | 192.168.20.10 | Django + Gunicorn | 443 HTTPS | erp.tradifyglobal.com |
| Pristine Planet | 192.168.20.20 | React + Vite | 80 HTTP | pristineplanet.eco |
| Nginx Proxy | 192.168.20.21 | Nginx Reverse Proxy | 80/443 | Both domains |
| Firewall | 192.168.1.1 | Fortigate | Web Admin | N/A |

## Emergency Access Procedures

### If You Lose SSH Access
1. Contact your infrastructure provider
2. Request console/KVM access
3. Boot into recovery mode if needed
4. Reset SSH keys or passwords

### If Database is Down
1. Check database server status on 192.168.20.11
2. Verify connectivity from ERP server (192.168.20.10)
3. Check PostgreSQL service: `systemctl status postgresql`
4. Review logs: `/var/log/postgresql/`

### If Nginx Proxy is Down
1. SSH to proxy server: `ssh root@192.168.20.21`
2. Check status: `systemctl status nginx`
3. Restart if stopped: `systemctl start nginx`
4. Check configuration: `nginx -t`
5. Review logs: `/var/log/nginx/error.log`

## Common Commands

### Check Services
```bash
# On ERP server
ssh username@192.168.20.10
systemctl status gunicorn  # Django app
systemctl status nginx     # Web server
ps aux | grep python

# On Pristine Planet
ssh username@192.168.20.20
systemctl status nginx
ps aux | grep node

# On Proxy
ssh root@192.168.20.21
systemctl status nginx
ss -tulpn | grep nginx
```

### View Logs
```bash
# Proxy access logs
ssh root@192.168.20.21
tail -f /var/log/nginx/erp.tradifyglobal.com.access.log
tail -f /var/log/nginx/pristineplanet.eco.access.log

# Proxy errors
tail -f /var/log/nginx/error.log

# ERP Django logs
ssh username@192.168.20.10
journalctl -u gunicorn -f
```

### Certificate Management
```bash
# Check certificate expiration
ssh root@192.168.20.21
certbot certificates

# Renew certificates
certbot renew --dry-run
certbot renew

# View certificate details
openssl x509 -in /etc/letsencrypt/live/erp.tradifyglobal.com/fullchain.pem -text -noout
```

## Security Best Practices

1. **Rotate Passwords Regularly**
   - Change SSH passwords every 90 days
   - Change database passwords annually
   - Update API keys as needed

2. **Backup Credentials**
   - Keep encrypted backup of credentials.local.md
   - Store in password manager (LastPass, 1Password, etc.)
   - Never email or message passwords

3. **SSH Key Management**
   - Use strong SSH keys (4096-bit RSA or ed25519)
   - Keep private keys secure on your local machine
   - Never commit private keys to GitHub

4. **Firewall Configuration**
   - Only allow SSH from known IP addresses
   - Use strong passwords or SSH keys only
   - Consider VPN for remote access

5. **Database Security**
   - Use strong database passwords
   - Limit database access to application servers only
   - Enable SSL for database connections if possible

## Deployment Workflow

### Updating Proxy Configuration
```bash
# 1. Make changes to nginx configs locally
# 2. Test locally or in staging
# 3. Commit to GitHub
git add nginx-config/
git commit -m "Update proxy configuration"
git push origin main

# 4. On proxy server, pull and deploy
ssh root@192.168.20.21
cd /opt/nginx-config
git pull origin main
cp nginx-config/*.conf /etc/nginx/sites-available/
nginx -t
systemctl reload nginx
```

### Updating Application Code
```bash
# ERP Updates
ssh username@192.168.20.10
cd /var/www/tradifyerp
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
systemctl restart gunicorn

# Pristine Planet Updates
ssh username@192.168.20.20
cd /var/www/pristine-planet
git pull origin Development
npm install
npm run build
# Nginx will automatically serve updated dist/
```

## Monitoring & Alerts

Set up monitoring for:
- Disk space on all servers
- Memory usage
- Certificate expiration (30 days before)
- Service availability
- Error rate on applications

## Contact & Support

- Primary Admin: [YOUR NAME/EMAIL]
- Secondary Admin: [SECONDARY CONTACT]
- Hosting Support: [PROVIDER CONTACT]
- DNS Support: GoDaddy support portal

---

**Remember:** This guide references `credentials.local.md` which is stored locally and never committed to git. Always keep credentials separate from code.
