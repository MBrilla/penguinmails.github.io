---
title: "Analysis & Reporting"
description: "Reputation analysis and reporting"
last_modified_date: "2025-12-04"
level: "3"
keywords: "analysis, reporting, insights"
---

# Analysis & Reporting

This document covers reputation analysis and reporting infrastructure.

## Database Schema

```sql
-- Reputation alerts table
CREATE TABLE reputation_alerts (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  acknowledged_at TIMESTAMP,
  resolved_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reputation_alerts_tenant ON reputation_alerts(tenant_id, created_at);
CREATE INDEX idx_reputation_alerts_status ON reputation_alerts(status);

-- Reputation history (daily snapshots)
CREATE TABLE reputation_history (
  id UUID PRIMARY KEY,
  domain_id UUID NOT NULL REFERENCES domains(id),

  -- Date
  snapshot_date DATE NOT NULL,

  -- Scores
  health_score INTEGER,
  sender_score INTEGER,

  -- Metrics
  inbox_rate DECIMAL(5,2),
  spam_rate DECIMAL(5,2),
  bounce_rate DECIMAL(5,2),
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),

  -- Blacklists
  blacklist_count INTEGER DEFAULT 0,

  created_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(domain_id, snapshot_date)
);

CREATE INDEX idx_reputation_history_domain ON reputation_history(domain_id, snapshot_date);

```

### Reputation Monitoring Service

```typescript
interface ReputationMetrics {
  healthScore: number;
  senderScore: number;
  inboxRate: number;
  spamRate: number;
  bounceRate: number;
  openRate: number;
  clickRate: number;
}

class ReputationMonitoringService {
  async calculateDomainHealth(domainId: string): Promise<ReputationMetrics> {
    const domain = await db.domains.findById(domainId);

    // Get recent campaign metrics
    const campaigns = await db.campaigns.findAll({
      where: {
        domainId,
        sentAt: { [Op.gte]: subDays(new Date(), 30) },
      },
    });

    // Calculate aggregate metrics
    const totalSent = campaigns.reduce((sum, c) => sum + c.sentCount, 0);
    const totalDelivered = campaigns.reduce((sum, c) => sum + c.deliveredCount, 0);
    const totalOpens = campaigns.reduce((sum, c) => sum + c.openCount, 0);
    const totalClicks = campaigns.reduce((sum, c) => sum + c.clickCount, 0);
    const totalBounces = campaigns.reduce((sum, c) => sum + c.bounceCount, 0);
    const totalSpam = campaigns.reduce((sum, c) => sum + c.spamCount, 0);

    const inboxRate = totalSent > 0 ? (totalDelivered / totalSent) * 100 : 0;
    const bounceRate = totalSent > 0 ? (totalBounces / totalSent) * 100 : 0;
    const spamRate = totalSent > 0 ? (totalSpam / totalSent) * 100 : 0;
    const openRate = totalDelivered > 0 ? (totalOpens / totalDelivered) * 100 : 0;
    const clickRate = totalDelivered > 0 ? (totalClicks / totalDelivered) * 100 : 0;

    // Calculate health score
    const healthScore = this.calculateHealthScore({
      inboxRate,
      bounceRate,
      spamRate,
      openRate,
      clickRate,
    });

    // Calculate sender score
    const senderScore = await this.calculateSenderScore(domainId);

    // Save metrics
    await db.domainReputation.create({
      domainId,
      healthScore,
      senderScore,
      inboxRate,
      spamRate,
      bounceRate,
      openRate,
      clickRate,
      measuredAt: new Date(),
    });

    return {
      healthScore,
      senderScore,
      inboxRate,
      spamRate,
      bounceRate,
      openRate,
      clickRate,
    };
  }

  private calculateHealthScore(metrics: {
    inboxRate: number;
    bounceRate: number;
    spamRate: number;
    openRate: number;
    clickRate: number;
  }): number {
    let score = 100;

    // Penalize for poor deliverability
    score -= (100 - metrics.inboxRate) * 0.5;  // Max -50 points
    score -= metrics.bounceRate * 5;  // -5 points per 1% bounce
    score -= metrics.spamRate * 10;  // -10 points per 1% spam

    // Reward for engagement
    score += Math.min(metrics.openRate * 0.5, 20);  // Max +20 points
    score += Math.min(metrics.clickRate * 2, 10);  // Max +10 points

    return Math.max(0, Math.min(100, Math.floor(score)));
  }

  private async calculateSenderScore(domainId: string): Promise<number> {
    let score = 0;

    // Infrastructure (30 points)
    const authScore = await this.checkAuthentication(domainId);
    score += authScore;

    // Volume & Consistency (25 points)
    const volumeScore = await this.checkSendingPattern(domainId);
    score += volumeScore;

    // Engagement (25 points)
    const engagementScore = await this.checkEngagement(domainId);
    score += engagementScore;

    // Complaints (20 points)
    const complaintsScore = await this.checkComplaints(domainId);
    score += complaintsScore;

    return Math.min(100, score);
  }

  async checkBlacklists(
    type: 'domain' | 'ip',
    value: string,
    tenantId: string
  ): Promise<{ listed: string[]; clean: string[] }> {
    const blacklists = this.getBlacklistsToCheck(type);
    const listed: string[] = [];
    const clean: string[] = [];

    for (const blacklist of blacklists) {
      const isListed = await this.queryBlacklist(blacklist, value);

      await db.blacklistChecks.create({
        checkType: type,
        checkValue: value,
        tenantId,
        blacklistName: blacklist.name,
        blacklistCategory: blacklist.category,
        isListed,
        listedSince: isListed ? new Date() : null,
        checkedAt: new Date(),
      });

      if (isListed) {
        listed.push(blacklist.name);

        // Create alert
        await this.createBlacklistAlert(tenantId, blacklist.name, value);
      } else {
        clean.push(blacklist.name);
      }
    }

    return { listed, clean };
  }

  private getBlacklistsToCheck(type: 'domain' | 'ip'): any[] {
    const ipBlacklists = [
      { name: 'Spamhaus ZEN', category: 'ip', dnsbl: 'zen.spamhaus.org' },
      { name: 'Spamcop', category: 'ip', dnsbl: 'bl.spamcop.net' },
      { name: 'SORBS', category: 'ip', dnsbl: 'dnsbl.sorbs.net' },
      { name: 'Barracuda', category: 'ip', dnsbl: 'b.barracudacentral.org' },
      // ... more IP blacklists
    ];

    const domainBlacklists = [
      { name: 'Spamhaus DBL', category: 'domain', dnsbl: 'dbl.spamhaus.org' },
      { name: 'SURBL', category: 'domain', dnsbl: 'multi.surbl.org' },
      { name: 'URIBL', category: 'domain', dnsbl: 'multi.uribl.com' },
      // ... more domain blacklists
    ];

    return type === 'ip' ? ipBlacklists : domainBlacklists;
  }

  private async queryBlacklist(blacklist: any, value: string): Promise<boolean> {
    try {
      // Reverse IP for DNSBL query
      const query = blacklist.category === 'ip'
        ? this.reverseIP(value) + '.' + blacklist.dnsbl
        : value + '.' + blacklist.dnsbl;

      const result = await dns.resolve4(query);
      return result.length > 0;
    } catch (error) {
      // NXDOMAIN means not listed
      return false;
    }
  }

  private reverseIP(ip: string): string {
    return ip.split('.').reverse().join('.');
  }

  private async createBlacklistAlert(
    tenantId: string,
    blacklistName: string,
    value: string
  ): Promise<void> {
    await db.reputationAlerts.create({
      tenantId,
      alertType: 'blacklist_detection',
      severity: 'critical',
      title: `Blacklist Detection: ${blacklistName}`,
      description: `${value} has been detected on ${blacklistName}`,
      alertData: {
        blacklist: blacklistName,
        value,
      },
      recommendedActions: [
        'Pause sending immediately',
        'Review recent campaigns',
