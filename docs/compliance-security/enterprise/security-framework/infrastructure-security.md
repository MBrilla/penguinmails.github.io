---
title: "Infrastructure Security"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Infrastructure Security

Network, server, and SSL/TLS configuration for secure infrastructure.

## Network Security

### Firewall Configuration

```bash
# UFW Firewall Rules
ufw default deny incoming
ufw default allow outgoing

# SSH access (specific IPs only)
ufw allow from 192.168.1.0/24 to any port 22

# HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Application ports (internal only)
ufw allow from 10.0.0.0/8 to any port 3000
ufw allow from 10.0.0.0/8 to any port 5432
ufw allow from 10.0.0.0/8 to any port 6379
```

### VPN Access

- **Team Access**: VPN required for infrastructure management
- **Database Access**: VPN-only access to production databases
- **Monitoring**: VPN access to monitoring dashboards

## Server Security

### VPS Security Hardening

```bash
# Disable root SSH login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Disable password authentication (key-based only)
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Install fail2ban
apt-get install fail2ban
systemctl enable fail2ban

# Update system packages
apt-get update && apt-get upgrade -y
```

### SSL/TLS Configuration

```nginx
# Nginx SSL Configuration
server {
    listen 443 ssl http2;
    server_name penguinmails.com;

    ssl_certificate /etc/letsencrypt/live/penguinmails.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/penguinmails.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
}
```

---

[← Back to Security Framework](./README)
