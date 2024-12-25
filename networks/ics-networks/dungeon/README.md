# Dungeon Network Guide

![Dungeon Network Banner](https://quasarstaking.ai/assets/dungeon-banner.png)

Welcome to the Dungeon Network guide - an ICS (Inter-Chain Security) chain designed for learning and testing in the Cosmos ecosystem.

## Table of Contents
- [Network Overview](#network-overview)
- [Getting Started](#getting-started)
- [Opt-In Process](#opt-in-process)
- [Validator Setup](#validator-setup)
- [Operation Guide](#operation-guide)
- [Resource Management](#resource-management)

## Network Overview 🌟

### What is Dungeon Network?
Dungeon is an ICS consumer chain that:
- Leverages Cosmos Hub security
- Provides learning environment
- Offers additional rewards
- Tests new features

### Key Details
```
Network Information:
- Chain ID: dungeon-1
- Token: DGN
- Minimum Commission: 0.05udgn
- Block Time: ~6 seconds
- ICS Protocol: v1
```

## Getting Started 🚀

### Prerequisites
- Active Cosmos Hub validator
- Good standing (no slashing history)
- Technical infrastructure ready
- Resource availability

Detailed in [setup.md](./setup.md)
 
### Monitoring Setup
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'dungeon'
    static_configs:
      - targets: ['localhost:26660']
    metrics_path: '/metrics'
```

### Alert Configuration
```yaml
# alerts.yml
groups:
  - name: dungeon
    rules:
      - alert: LowCommission
        expr: validator_commission < 0.05
        for: 5m
```

## Best Practices 💡

### Security
✅ Use dedicated sentry nodes
✅ Implement proper firewall rules
✅ Regular security updates
✅ Monitor system logs

### Performance
✅ Keep commission at/above 0.05udgn
✅ Maintain high uptime
✅ Regular backups
✅ Resource optimization

## Troubleshooting 🔍

### Common Issues

#### 1. Commission Too Low
```bash
# Update commission
gaiad tx staking edit-validator \
  --commission-rate "0.05" \
  --from validator \
  --chain-id dungeon-1
```

#### 2. Sync Issues
```bash
# Check status
gaiad status

# Reset if needed
gaiad unsafe-reset-all
```

## Support & Resources 🤝

### Official Channels
- Discord: [Join](https://discord.gg/tZW4xf3c2D)
- Telegram: [@quasarstakingeng](https://t.me/quasarstakingeng)
- Technical Support: [@whtech_support](https://t.me/whtech_support)

## FAQ ❓

### Why 0.05udgn minimum commission?
To ensure network sustainability and prevent commission wars.

### Can I change my commission?
Yes, but it must stay at or above 0.05udgn.

### What happens if commission is too low?
Risk of penalties and removal from active set.

---

*Maintained by Quasar - Leading Dungeon Network Validators*

💡 **Pro Tip**: Always verify your commission settings after network upgrades or validator restarts!