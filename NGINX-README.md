# Pristine Planet Nginx Reverse Proxy Configuration

Production-ready Nginx reverse proxy configuration for routing multiple domains to backend services.

## 📋 Overview

This repository contains the Nginx reverse proxy configuration files for the Pristine Planet infrastructure migration from AWS to on-premises ESXi.

**Infrastructure:**
- **Reverse Proxy Server:** 192.168.20.21 (Ubuntu 22.04)
- **ERP Backend:** 192.168.20.10 (Django + Gunicorn, HTTPS)
- **Pristine Planet Frontend:** 192.168.20.20 (React + Vite, HTTP)
- **Public IP:** 184.144.89.206 (via Fortigate NAT)

## 🚀 Quick Start

### Prerequisites
- Ubuntu 22.04 LTS
- Nginx 1.24.0 or higher
- Let's Encrypt certificates (Certbot)
- Root or sudo access

### Installation

1. **Install Nginx and Certbot:**
```bash
apt update
apt install -y nginx certbot python3-certbot-nginx
```

2. **Copy configuration files:**
```bash
cp erp.tradifyglobal.com.conf /etc/nginx/sites-available/
cp pristineplanet.eco.conf /etc/nginx/sites-available/
```

3. **Enable sites:**
```bash
ln -s /etc/nginx/sites-available/erp.tradifyglobal.com /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/pristineplanet.eco /etc/nginx/sites-enabled/
```

4. **Generate SSL certificates:**
```bash
# Create webroot directory for ACME challenge
mkdir -p /var/www/certbot

# Generate certificate
certbot certonly --webroot -w /var/www/certbot \
  --agree-tos --email admin@tradifyglobal.com \
  -d erp.tradifyglobal.com \
  -d pristineplanet.eco
```

5. **Test and reload:**
```bash
# Test configuration syntax
nginx -t

# Reload Nginx
systemctl reload nginx

# Verify Nginx is running
systemctl status nginx
```

## 📁 Directory Structure

```
/etc/nginx/
├── sites-available/
│   ├── erp.tradifyglobal.com        # ERP proxy config
│   └── pristineplanet.eco            # Pristine Planet proxy config
├── sites-enabled/
│   ├── erp.tradifyglobal.com -> ../sites-available/erp.tradifyglobal.com
│   └── pristineplanet.eco -> ../sites-available/pristineplanet.eco
└── ...

/etc/letsencrypt/live/
└── erp.tradifyglobal.com/
    ├── fullchain.pem
    └── privkey.pem

/var/log/nginx/
├── erp.tradifyglobal.com.access.log
├── erp.tradifyglobal.com.error.log
├── pristineplanet.eco.access.log
└── pristineplanet.eco.error.log
```

## 🔧 Configuration Details

### ERP Server (erp.tradifyglobal.com)

**Routes to:** 192.168.20.10:443 (HTTPS)

**Features:**
- HTTP (port 80) redirects to HTTPS
- HTTPS (port 443) with Let's Encrypt certificate
- Proxies to Django backend on port 443
- WebSocket support enabled
- SSL verification disabled for self-signed backend certificates
- Connection timeouts: 60 seconds
- Request/response buffering enabled

### Pristine Planet (pristineplanet.eco)

**Routes to:** 192.168.20.20:80 (HTTP)

**Features:**
- HTTP (port 80) redirects to HTTPS
- HTTPS (port 443) with Let's Encrypt certificate
- Proxies to React/Vite frontend on port 80
- WebSocket support enabled
- Connection timeouts: 60 seconds
- Request/response buffering enabled

## 🔐 SSL/TLS Configuration

### Certificate Details
- **Issuer:** Let's Encrypt
- **Certificate:** /etc/letsencrypt/live/erp.tradifyglobal.com/fullchain.pem
- **Private Key:** /etc/letsencrypt/live/erp.tradifyglobal.com/privkey.pem
- **Protocols:** TLSv1.2 and TLSv1.3
- **Auto-renewal:** Enabled via Certbot

### Certificate Renewal
```bash
# Manual renewal (if needed)
certbot renew --non-interactive --quiet

# Check renewal status
certbot certificates
```

## 📊 Domain Routing

```
┌─────────────────────────────────┐
│  Internet Users                 │
│  184.144.89.206                 │
└─────────────┬───────────────────┘
              │ HTTPS
              ▼
      ┌──────────────────┐
      │ Nginx Proxy      │
      │ 192.168.20.21    │
      └────┬──────────┬──┘
           │          │
    HTTPS  │          │ HTTP
    443    │          │ 80
           ▼          ▼
    192.168.20.10  192.168.20.20
    (ERP)          (React)
```

## 🧪 Testing

### Test HTTPS Connection
```bash
# From local machine
curl https://erp.tradifyglobal.com
curl https://pristineplanet.eco

# With verbose output
curl -v https://erp.tradifyglobal.com

# Check certificate
curl -vI https://erp.tradifyglobal.com 2>&1 | grep -i "certificate"
```

### Verify Backend Connectivity
```bash
# From proxy server
nc -zv 192.168.20.10 443
nc -zv 192.168.20.20 80

# Or with curl
curl -k https://192.168.20.10:443
curl http://192.168.20.20:80
```

### Check Nginx Logs
```bash
# Real-time access logs
tail -f /var/log/nginx/erp.tradifyglobal.com.access.log
tail -f /var/log/nginx/pristineplanet.eco.access.log

# Error logs
tail -f /var/log/nginx/erp.tradifyglobal.com.error.log
tail -f /var/log/nginx/pristineplanet.eco.error.log

# All logs
tail -f /var/log/nginx/*.log
```

## 🔍 Monitoring

### Check Nginx Status
```bash
# Service status
systemctl status nginx

# Process info
ps aux | grep nginx

# Listening ports
ss -tulpn | grep nginx

# Memory usage
top -p $(pgrep -d, nginx)
```

### Certificate Expiration
```bash
# Check when certificate expires
openssl x509 -in /etc/letsencrypt/live/erp.tradifyglobal.com/fullchain.pem -noout -dates

# Or with certbot
certbot certificates
```

## 🚨 Troubleshooting

### Problem: Certificate Not Found
```bash
# Check if certificate exists
ls -la /etc/letsencrypt/live/erp.tradifyglobal.com/

# Generate if missing
certbot certonly --webroot -w /var/www/certbot -d erp.tradifyglobal.com -d pristineplanet.eco
```

### Problem: Backend Connection Refused
```bash
# Check if backend is running
ssh root@192.168.20.10 "systemctl status nginx"
ssh root@192.168.20.20 "systemctl status nginx"

# Test connectivity from proxy
nc -zv 192.168.20.10 443
nc -zv 192.168.20.20 80
```

### Problem: Nginx Configuration Error
```bash
# Test configuration syntax
nginx -t

# Check for detailed errors
nginx -T

# View Nginx error log
tail -f /var/log/nginx/error.log
```

### Problem: Let's Encrypt Validation Fails
```bash
# Check if port 80 is accessible from internet
curl -v http://erp.tradifyglobal.com/.well-known/acme-challenge/test

# Verify Fortigate firewall rules allow port 80 inbound
# Verify DNS points to correct public IP
nslookup erp.tradifyglobal.com
```

## 📝 Header Forwarding

All configurations include these important headers for proper backend communication:

- **Host:** Original hostname from client request
- **X-Real-IP:** Client's real IP address
- **X-Forwarded-For:** Proxy chain (for logging)
- **X-Forwarded-Proto:** Original protocol (HTTP/HTTPS)
- **X-Forwarded-Host:** Original hostname
- **X-Forwarded-Port:** Original port

These headers are essential for:
- Django/Python apps to get correct hostname for ALLOWED_HOSTS
- WebSocket connections
- Client IP logging
- Security headers

## 🔄 Auto-Renewal

Certbot automatically renews certificates 30 days before expiration:

```bash
# Check renewal timer
systemctl list-timers certbot

# View renewal logs
tail -f /var/log/letsencrypt/letsencrypt.log

# Manual test of renewal
certbot renew --dry-run
```

## 📚 References

- [Nginx Proxy Documentation](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Let's Encrypt Certbot](https://certbot.eff.org/)
- [Nginx Best Practices](https://nginx.org/en/docs/)

## 📞 Support

For issues or questions regarding this configuration:
1. Check the troubleshooting section above
2. Review Nginx error logs: `/var/log/nginx/error.log`
3. Check Let's Encrypt logs: `/var/log/letsencrypt/letsencrypt.log`

## 📄 License

This configuration is part of the Pristine Planet infrastructure project.

---

**Status:** Production Ready
**Last Updated:** January 4, 2026
**Nginx Version:** 1.24.0
**Let's Encrypt:** Active with auto-renewal
