---
title: "Server Configuration Requirements"
description: "Implementation requirements, software stack, and network security for email servers"
last_modified_date: "2025-11-10"
level: "2"
persona: "Technical Teams, DevOps Engineers"
---

# Server Configuration Requirements

Implementation requirements, software stack specifications, and network security configuration for email infrastructure deployment.

## Minimum Server Specifications

Server requirements scale with email volume. Self-hosted infrastructure requires proper resource allocation for reliable delivery.

### 1K-10K emails/month

```
CPU: 1 vCPU (2.0+ GHz)
RAM: 1-2GB
Storage: 25-50GB SSD
Bandwidth: 1-2TB/month
OS: Ubuntu 20.04+ LTS or CentOS 8+
```

### 10K-100K emails/month

```
CPU: 1-2 vCPUs (2.4+ GHz)
RAM: 2-4GB
Storage: 50-80GB SSD
Bandwidth: 2-4TB/month
OS: Ubuntu 20.04+ LTS or CentOS 8+
```

### 100K-1M emails/month

```
CPU: 2-4 vCPUs (3.0+ GHz)
RAM: 4-8GB
Storage: 100-160GB SSD
Bandwidth: 4-6TB/month
OS: Ubuntu 20.04+ LTS or CentOS 8+
```

### 1M+ emails/month

```
CPU: 4-8+ vCPUs (3.2+ GHz)
RAM: 8-16GB+ SSD
Storage: 160-320GB+ SSD
Bandwidth: 6-8TB+ SSD
OS: Ubuntu 20.04+ LTS or CentOS 8+
Multiple servers for load balancing
```

## Software Stack Requirements

### Core Email Server Software

- **Postfix**: SMTP server with advanced configuration
- **Dovecot**: IMAP/POP3 server for mailbox access
- **SpamAssassin**: Spam filtering and virus protection
- **ClamAV**: Antivirus scanning engine
- **Amavisd-new**: Content filter integration

### Supporting Software

- **PostgreSQL**: Database for user management (with NileDB multi-tenancy)
- **Apache/Nginx**: Web server for administration
- **Roundcube/SquirrelMail**: Webmail interface
- **Fail2ban**: Intrusion prevention system
- **Logwatch/Swatch**: Log monitoring and analysis

## Network and Security Configuration

### Firewall Requirements

Essential ports for email servers:

| Service | Port | Direction | Notes |
|---------|------|-----------|-------|
| **SMTP** | 25 | Outbound | Standard SMTP |
| **SMTP SSL** | 465 | Outbound | Encrypted submission |
| **SMTP TLS** | 587 | Outbound | Submission with STARTTLS |
| **IMAP** | 143 | Inbound | Standard IMAP |
| **IMAP SSL** | 993 | Inbound | Encrypted IMAP |
| **POP3** | 110 | Inbound | Standard POP3 |
| **POP3 SSL** | 995 | Inbound | Encrypted POP3 |
| **HTTP** | 80 | Inbound | Webmail |
| **HTTPS** | 443 | Inbound | Encrypted webmail |
| **SSH** | 22 | Inbound | Restricted access |

### DNS Configuration Requirements

Proper DNS records are essential for deliverability and authentication.

#### SPF Record

Required for all sending domains:

```
v=spf1 include:_spf.google.com include:sendgrid.net ~all
```

#### DKIM Record

Required for deliverability:

```
default._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA..."
```

#### DMARC Record

Recommended for policy enforcement:

```
_dmarc.example.com. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
```

## Backup and Disaster Recovery

### Backup Strategy

Critical components requiring backup:

- **Configuration Files**: Postfix, Dovecot, DNS settings
- **User Data**: Mailbox contents, user preferences
- **Database Backups**: User accounts, message metadata
- **SSL Certificates**: Domain validation and keys
- **Custom Scripts**: Automation and monitoring tools

### Backup Implementation

```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/email/$DATE"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup Postfix configuration
tar -czf $BACKUP_DIR/postfix.tar.gz /etc/postfix

# Backup Dovecot configuration
tar -czf $BACKUP_DIR/dovecot.tar.gz /etc/dovecot

# Backup user mailboxes
rsync -av /var/mail/ $BACKUP_DIR/mailboxes/

# Upload to cloud storage
aws s3 sync $BACKUP_DIR s3://email-backups/$DATE/
```

### Disaster Recovery Plan

1. **Primary Server Failure**: Automatic failover to backup server
2. **IP Blacklisting**: Emergency IP rotation procedures
3. **Domain Compromised**: Emergency domain shutdown and re-verification
4. **Data Loss Recovery**: Point-in-time restore from backups
5. **Full System Recovery**: Complete infrastructure rebuild procedures

## Related Documentation

- [Technical Infrastructure Overview](/docs/business/implementation/technical-infrastructure/overview) - Summary and navigation
- [IP Management](/docs/business/implementation/technical-infrastructure/ip-management) - Reputation and allocation
- [Performance Optimization](/docs/business/implementation/technical-infrastructure/performance-optimization) - Tuning guidelines
