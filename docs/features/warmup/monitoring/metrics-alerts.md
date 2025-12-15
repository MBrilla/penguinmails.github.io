---
title: "Metrics & Alerts"
description: "Reputation metrics and alerting"
last_modified_date: "2025-12-04"
level: "3"
keywords: "metrics, alerts, monitoring"
---

# Metrics & Alerts

## Delisting Process

### Delisting Guide: Spamcop

```text
Delisting Guide: Spamcop

Status: Listed
Listed Since: Nov 25, 2025 2:15 PM
Auto-Delist: 24 hours after last report

Steps to Delist:

1. Identify Root Cause
   ☐ Review recent campaigns
   ☐ Check for compromised accounts
   ☐ Verify list quality

2. Fix Issues
   ☐ Remove problematic content
   ☐ Clean email list
   ☐ Secure compromised accounts

3. Request Delisting
   URL: https://www.spamcop.net/bl.shtml

   Information Needed:

   - IP Address: 192.0.2.100

   - Listing ID: ABC123XYZ

   - Explanation of corrective actions

4. Monitor Status
   ☐ Check delisting status (every 4 hours)
   ☐ Verify removal (may take 24-48 hours)

5. Resume Sending
   ☐ Gradually increase volume
   ☐ Monitor reputation closely

[Copy Delisting Template] [Track Progress]

```

### Domain Health Monitoring

#### Health Score Components

```text

Domain Health: penguinmails.com (92/100)

Authentication (25 points): 25/25 âœ“
  SPF: âœ“ Pass (v=spf1 include:_spf.google.com ~all)
  DKIM: âœ“ Pass (2048-bit key)
  DMARC: âœ“ Pass (p=quarantine, pct=100)
  BIMI: âœ“ Configured

Deliverability (30 points): 28/30 âœ“
  Inbox Rate: 94% (28/30)
  Bounce Rate: 1.8% (âœ“)
  Spam Rate: 3% (âœ“)

Engagement (25 points): 22/25 âœ“
  Open Rate: 28% (18/25)
  Click Rate: 4.2% (4/25)

Infrastructure (20 points): 17/20 âœ“
  DNS Configuration: âœ“ Correct
  SSL/TLS: âœ“ Valid
  Reverse DNS: âœ“ Configured
  Blacklists: âœ“ Clean (20/20)
  IP Reputation: 85/100 (-3 points)

```

#### Historical Trends

```text

Reputation Trend: Last 30 Days

Health Score:
  Nov 1:  85 â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘
  Nov 8:  87 â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘
  Nov 15: 89 â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘
  Nov 22: 91 â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘
  Nov 25: 92 â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘

  Trend: ↑ +7 points (30 days)

Inbox Rate:
  Nov 1:  89% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘
  Nov 8:  91% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘
  Nov 15: 92% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘
  Nov 22: 93% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘
  Nov 25: 94% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ

  Trend: ↑ +5% (30 days)

Engagement:
  Open Rate: 24% → 28% (↑ +4%)
  Click Rate: 3.8% → 4.2% (↑ +0.4%)

```

### Recovery Recommendations

#### AI-Powered Insights

```text

Reputation Recovery Plan

Current Status:
  Domain: sales.penguinmails.com
  Health Score: 78/100 (Fair)
  Primary Issue: Elevated bounce rate (4.2%)

Recommended Actions:

Priority 1: Immediate (Next 24 hours)

  1. ☐ Pause all campaigns from this domain
     Impact: Prevent further damage
     Effort: 5 minutes

  2. ☐ Clean email list (remove hard bounces)
     Impact: Reduce bounce rate by ~2%
     Effort: 30 minutes

  3. ☐ Verify list source and acquisition method
     Impact: Identify root cause
     Effort: 1 hour

Priority 2: Short-term (Next 7 days)

  4. ☐ Implement double opt-in for new signups
     Impact: Improve list quality long-term
     Effort: 2 hours

  5. ☐ Re-engage inactive subscribers
     Impact: Improve engagement metrics
     Effort: 4 hours

  6. ☐ Reduce sending volume by 50%
     Impact: Rebuild reputation gradually
     Effort: 15 minutes

Priority 3: Long-term (Next 30 days)

  7. ☐ Implement list hygiene automation
     Impact: Maintain high list quality
     Effort: 8 hours

  8. ☐ Set up engagement-based segmentation
     Impact: Send only to engaged subscribers
     Effort: 4 hours

Expected Outcome:
  Week 1: Health score → 82/100
  Week 2: Health score → 86/100
  Week 4: Health score → 90/100

[Start Recovery Plan] [Customize Plan]

```

#### Automated Remediation

```text

Auto-Remediation Rules

Rule 1: High Bounce Rate
  Trigger: Bounce rate > 5%
  Actions:
    ☑ Pause all sending
    ☑ Send alert to admin
    ☑ Create support ticket
    ☑ Remove hard bounces from list

Rule 2: Blacklist Detection
  Trigger: Listed on any blacklist
  Actions:
    ☑ Pause sending from affected domain/IP
    ☑ Send immediate alert
    ☑ Create delisting ticket
    ☑ Notify deliverability team

Rule 3: Spam Complaints
  Trigger: Spam rate > 0.1%
  Actions:
    ☑ Pause campaign
    ☑ Review content for spam triggers
    ☑ Check list acquisition method
    ☑ Send alert to sender

Rule 4: Low Engagement
  Trigger: Open rate < 10% for 3 consecutive campaigns
  Actions:
    ☐ Pause sending (optional)
    ☑ Send warning to sender
    ☑ Suggest list cleaning
    ☑ Recommend re-engagement campaign

[Edit Rules] [Add Custom Rule]

```

### Competitive Benchmarking

```text

Industry Benchmarks

Your Performance vs Industry Average

Inbox Rate:
  You: 94% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ
  Avg: 85% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘
  Top: 97% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ

  You're 9% above average âœ“

Open Rate:
  You: 28% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘
  Avg: 21% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘
  Top: 35% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘

  You're 7% above average âœ“

Click Rate:
  You: 4.2% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘
  Avg: 2.8% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘
  Top: 6.5% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘

  You're 1.4% above average âœ“

Bounce Rate:
  You: 1.8% â–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘
  Avg: 2.5% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘
  Top: 0.8% â–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘

  You're 0.7% below average âœ“

Industry: SaaS
Region: North America
Company Size: 51-200 employees

```

---

## Level 3: Technical Implementation

### Database Schema

```sql
-- Domain reputation tracking
CREATE TABLE domain_reputation (
  id UUID PRIMARY KEY,
  domain_id UUID NOT NULL REFERENCES domains(id),

  -- Reputation scores
  health_score INTEGER,
  sender_score INTEGER,
  domain_authority INTEGER,

  -- Deliverability metrics
  inbox_rate DECIMAL(5,2),
  spam_rate DECIMAL(5,2),
  bounce_rate DECIMAL(5,2),

  -- Engagement metrics
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),
  reply_rate DECIMAL(5,2),

  -- Authentication
  spf_status VARCHAR(50),
  dkim_status VARCHAR(50),
  dmarc_status VARCHAR(50),

  -- Provider-specific
  gmail_reputation VARCHAR(50),
  outlook_reputation VARCHAR(50),
  yahoo_reputation VARCHAR(50),

  -- Timestamp
  measured_at TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_domain_reputation_domain ON domain_reputation(domain_id, measured_at);

-- Blacklist monitoring
CREATE TABLE blacklist_checks (
  id UUID PRIMARY KEY,

  -- Target
  check_type VARCHAR(50),  -- domain, ip
  check_value VARCHAR(255),  -- domain.com or IP address
  tenant_id UUID REFERENCES tenants(id),

  -- Blacklist
  blacklist_name VARCHAR(255),
  blacklist_category VARCHAR(100),  -- ip, domain, combined

  -- Result
  is_listed BOOLEAN DEFAULT FALSE,
  listed_since TIMESTAMP,
  delisted_at TIMESTAMP,
  listing_reason TEXT,

  -- Metadata
  checked_at TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_blacklist_checks_value ON blacklist_checks(check_value, checked_at);
CREATE INDEX idx_blacklist_checks_listed ON blacklist_checks(is_listed);

-- Reputation alerts
CREATE TABLE reputation_alerts (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Alert details
  alert_type VARCHAR(100),  -- blacklist, bounce_rate, spam_rate, etc.
  severity VARCHAR(50),  -- info, warning, critical

  -- Target
  domain_id UUID REFERENCES domains(id),
  email_account_id UUID REFERENCES email_accounts(id),

  -- Alert data
  title VARCHAR(255),
  description TEXT,
  alert_data JSONB,

  -- Recommendations
  recommended_actions JSONB,

  -- Status
  status VARCHAR(50),  -- active, acknowledged, resolved
  acknowledged_by UUID REFERENCES users(id),
