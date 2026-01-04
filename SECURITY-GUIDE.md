# Security Best Practices for Server Credentials

## Why NOT to Commit Passwords to GitHub

### Risks of Storing Credentials in Git

1. **Permanent Record**
   - Once committed, it's in git history forever
   - Even deleting from current version doesn't remove from history
   - Anyone with repo access can see entire history

2. **Public Repository Exposure**
   - Anyone can clone public GitHub repos
   - Bots automatically scan for hardcoded credentials
   - Credentials become searchable on the internet

3. **Account Compromise**
   - Attackers can use credentials to access your servers
   - SSH keys can be used for unauthorized access
   - Database credentials can expose sensitive data

4. **Compliance Issues**
   - Violates security standards (SOC 2, ISO 27001, PCI DSS)
   - Can breach data protection regulations (GDPR, CCPA)
   - May violate customer trust agreements

### Real-World Examples

- GitHub automatically revokes any exposed credentials
- Many companies discovered credentials in git and had security breaches
- AWS keys in public repos are exploited within minutes

---

## Recommended Approach (What We Did)

### ✅ Local Credentials File

**What we created:**
- `credentials.local.md` - Template with placeholders
- `.gitignore` - Prevents accidental commits

**How to use:**
1. Fill in YOUR actual credentials locally
2. Keep file on your machine only
3. Share credentials through secure channels (password manager, encrypted email)
4. Never commit to GitHub

### Public Configuration Files

**What we stored in GitHub:**
- Nginx configuration files
- Installation guides
- Troubleshooting documentation
- Architecture diagrams
- All NON-SENSITIVE information

**What we excluded from GitHub:**
- Passwords and API keys
- SSH private keys
- Database credentials
- Firewall access credentials
- Any sensitive configuration

---

## Your Server Information Structure

### Locally (credentials.local.md - NOT in git)

```
ERP Server (192.168.20.10)
├── SSH Username: [YOUR_USERNAME]
├── SSH Password: [YOUR_PASSWORD]
├── SSH Key Path: /path/to/key
├── Database: tradifyerp_user / [DB_PASSWORD]
├── App User: [APP_USER]
└── App Password: [APP_PASSWORD]

Pristine Planet (192.168.20.20)
├── SSH Username: [YOUR_USERNAME]
├── SSH Password: [YOUR_PASSWORD]
├── SSH Key Path: /path/to/key
└── App Directory: /var/www/pristine-planet/

Proxy Server (192.168.20.21)
├── SSH Username: root
├── SSH Password: proxy
├── Certbot Email: admin@tradifyglobal.com
└── Certificate Path: /etc/letsencrypt/live/

Firewall (192.168.1.1)
├── Admin Username: [ADMIN_USER]
└── Admin Password: [ADMIN_PASSWORD]

GoDaddy DNS
├── Username: [GODADDY_USER]
└── Password: [GODADDY_PASSWORD]
```

### On GitHub (Public - No Secrets)

```
PristinePlanet-NGNIX-Proxy/
├── nginx-config/
│   ├── erp.tradifyglobal.com.conf
│   └── pristineplanet.eco.conf
├── NGINX-README.md
├── SETUP-GUIDE.md
├── SSL-COMPLETE-SUCCESS.md
├── .gitignore (Prevents credential leaks)
└── Other documentation
```

---

## How to Fill in Your Credentials

### Step 1: Create Local Template

The `credentials.local.md` file is already created with placeholders:

```bash
cat credentials.local.md
```

### Step 2: Replace Placeholders

Edit the file and replace:
- `[YOUR_USERNAME]` → your actual SSH username
- `[YOUR_PASSWORD]` → your SSH password
- `[SSH_PUBLIC_KEY_PATH]` → /home/you/.ssh/id_rsa.pub
- `[SSH_PRIVATE_KEY_PATH]` → /home/you/.ssh/id_rsa
- `[DB_PASSWORD]` → PostgreSQL password
- And all other placeholders

### Step 3: Keep It Secure

```bash
# Set proper permissions (readable by you only)
chmod 600 credentials.local.md

# Optional: Use a password manager instead
# Services like 1Password, LastPass, Vault, etc. are better for storing credentials
```

### Step 4: NEVER Commit It

```bash
# Verify it's in .gitignore
cat .gitignore | grep credentials

# Double-check before committing anything
git status
# Should NOT show credentials.local.md
```

---

## Accessing Your Servers

Once you've filled in `credentials.local.md`, use it to SSH:

### ERP Server
```bash
# From credentials.local.md:
# Username: [FILLED_IN]
# Password: [FILLED_IN]
# IP: 192.168.20.10

ssh username@192.168.20.10
# Or with key:
ssh -i /path/to/private/key username@192.168.20.10
```

### Pristine Planet Server
```bash
# Username: [FILLED_IN]
# Password: [FILLED_IN]
# IP: 192.168.20.20

ssh username@192.168.20.20
```

### Reverse Proxy Server
```bash
# Username: root
# Password: proxy (CHANGE THIS!)
# IP: 192.168.20.21

ssh root@192.168.20.21
# Or with your preferred SSH method
```

---

## GitHub Repository

**Public Repository:**
```
https://github.com/tradifyglobal/PristinePlanet-NGNIX-Proxy.git
```

**Safe to Share:** YES
- Contains only public configuration files
- No credentials or secrets
- Documentation and guides only

**NOT Shared Here:**
- passwords
- SSH private keys
- API credentials
- Database passwords
- Firewall credentials

---

## Regular Security Maintenance

### Monthly
- Check for compromised credentials
- Review access logs
- Update SSH key permissions

### Quarterly  
- Rotate passwords
- Review firewall rules
- Audit DNS configuration

### Annually
- Update SSH keys
- Change all default passwords
- Review certificate expiration

---

## Emergency: If Credentials Are Leaked

1. **Immediately change** the exposed password
2. **Restart services** to invalidate any sessions
3. **Review logs** for unauthorized access
4. **Monitor accounts** for suspicious activity
5. **Rotate keys** if SSH keys were exposed

**SSH Key Compromise:**
```bash
# Generate new key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Update authorized_keys on servers
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Remove old key from authorized_keys
nano ~/.ssh/authorized_keys
```

---

## Summary

| What | Location | Shared? | Contains |
|------|----------|---------|----------|
| **GitHub Repo** | Public | YES | Config files, docs |
| **credentials.local.md** | Local Only | NO | Passwords, SSH creds |
| **nginx-config/** | GitHub | YES | Nginx server blocks |
| **SETUP-GUIDE.md** | GitHub | YES | Instructions |
| **Password Manager** | Secure | NO | Master credentials |

---

✅ **Your setup is now secure and properly organized!**

The GitHub repository contains everything needed for deployment, while sensitive credentials remain safe on your local machine.
