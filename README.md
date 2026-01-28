# S360 Security Core Module

This module provides comprehensive security configurations including automated IP banning rules to protect against common web attacks.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Monitoring](#monitoring)
- [Composer Installation](#composer-installation)
- [Troubleshooting](#troubleshooting)
- [Maintainers](#maintainers)

## Features

### Autoban Rules

The module includes 10 preconfigured autoban rules that automatically ban IPs attempting common attack patterns:

1. **PHP Scan** - Blocks attempts to access arbitrary `.php` files (2 attempts)
2. **WordPress Scan** - Blocks WordPress path scanning `/wp-*` (2 attempts)
3. **Joomla Scan** - Blocks Joomla admin panel access `/administrator` (2 attempts)
4. **Environment File Scan** - Blocks `.env` file access attempts (1 attempt) - CRITICAL
5. **Git Directory Scan** - Blocks `.git` directory access (1 attempt) - CRITICAL
6. **Database Admin Tools** - Blocks phpMyAdmin, Adminer attempts (2 attempts)
7. **Backup File Scan** - Blocks backup file theft attempts (2 attempts)
8. Requirements
### System Requirements

- PHP 8.1, 8.2, or 8.3
- Drupal 10.x or 11.x

### Module Dependencies
This module requires the following modules:

- [Autoban](https://www.drupal.org/project/autoban) (^1.0)
- [Advanced Ban](https://www.drupal.org/project/advban) (^1.0)

## **XML-RPC Attack** - Blocks XML-RPC brute force attempts (1 attempt)
9. **Shell/Webshell** - Blocks webshell upload attempts (1 attempt) - CRITICAL
10. **Config File Scan** - Blocks config file access attempts (2 attempts)

### Ban Thresholds

- **Critical threats** (`.env`, `.git`, shells, xmlrpc): Ban after 1 attempt
- **High threats**: Ban after 2 attempts
- All rules use IP range banning to block entire malicious subnets
- Rules only apply to anonymous users (authenticated users are not affected)

## Installation

> **⚠️ IMPORTANT:** After enabling this module, you must run the included recipe to activate the security rules. Simply enabling the module does not automatically configure the autoban rules.

### Option 1: Using Recipe (Recommended)

Install via the included recipe that handles all dependencies and configurations:

```bash
# Using Lando + Drush
lando drush recipe /app/web/modules/contrib/s360_security_core/recipes/autoban_security/

# Or using Drush (without Lando)
drush recipe recipes/autoban_security

# Or using Drupal core's recipe command (Drupal 10.3+)
php core/scripts/drupal recipe recipes/autoban_security
```

### Option 2: Manual Installation

```bash
# Enable the module
drush en s360_security_core -y

# Import configurations
drush config:import --partial --source=modules/contrib/s360_security_core/config/install
```

## Dependencies

- `autoban` - Main autoban module
- `autoban_advban` - Advanced ban integration
- `autoban_dblog` - Database log integration
- `advban` - Advanced ban module for range banning

## Configuration

After installation, you can manage autoban rules at:
- `/admin/config/people/autoban`

View banned IPs at:
- `/admin/config/people/advban`

## Monitoring

Check autoban activity in watchdog logs:

```bash
drush watchdog:show --type=autoban --count=50
```

## Maintenance

The module runs autoban checks on cron. Ensure cron is running regularly for optimal protection.

## Composer Installation

This module is available via the Square360 private Composer repository.

### Prerequisites

Add the Square360 repository to your project's `composer.json`:

```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.square360.com"
    }
  ]
}
```

### Installation

```bash
composer require square360/s360_security_core
drush en s360_security_core -y
```

**Important:** After enabling, run the recipe to activate security rules:

```bash
# Using Lando
lando drush recipe /app/web/modules/contrib/s360_security_core/recipes/autoban_security/

# Without Lando
drush recipe web/modules/contrib/s360_security_core/recipes/autoban_security
```

## Troubleshooting

### Autoban rules not working

1. Check that cron is running regularly
2. Verify autoban and advban modules are enabled
3. Check the status report at `/admin/reports/status`
4. Review recent log messages at `/admin/reports/dblog`

### False positives

If legitimate users are being banned:

1. Review the banned IPs at `/admin/config/people/advban`
2. Unban the IP if needed
3. Consider adjusting the threshold for the rule at `/admin/config/people/autoban`
4. Add the IP to the whitelist in autoban settings

## Future Enhancements

### Smart Ban Duration & Escalation (Planned)

**Status:** Feasibility analysis complete - Implementation planned for future release

**Goal:** Implement intelligent ban duration with escalation to permanent bans for repeat offenders.

#### Proposed Behavior

1. **First Offense**: Random ban duration between 5-25 days
   - Prevents attackers from knowing exact unban time
   - Reduces predictability of security measures
   - Adds variability to deter automated retry attempts

2. **Repeat Offenses**: Permanent ban (no expiry)
   - Any IP banned more than once (across any autoban rule) receives permanent ban
   - Escalation based on historical violations stored in `advban_ip` table
   - Applies to all autoban-triggered bans, not manual bans

#### Implementation Strategy

**Approach:** Event Subscriber Pattern (Option 3)

This approach was selected for:
- **Drupal 11 Compatibility**: Event-driven architecture aligns with Drupal's modern patterns
- **Non-invasive**: No contrib module patches required
- **Maintainability**: Clean separation of concerns
- **Testability**: Easy to unit test ban logic independently
- **Configurability**: Can expose settings via admin UI

#### Technical Architecture

**Integration Point:**
- Event subscriber in s360_security_core module
- Intercepts ban events before `AdvbanIpManager::banIp()` execution
- Query ban history: `SELECT COUNT(*) FROM advban_ip WHERE ip = :ip AND reason LIKE 'autoban:%'`

**Ban Duration Logic:**
```
IF previous_ban_count = 0:
  expiry_date = current_time + random(432000, 2160000)  // 5-25 days in seconds
ELSE:
  expiry_date = 0  // Permanent ban (0 = never expires)
```

**Data Points Available:**
- IP address from ban request
- Autoban rule ID from metadata (`reason` field stores `autoban:{rule_id}`)
- Historical ban count from `advban_ip` table
- Ban timestamps for tracking repeat offenses

#### Database Schema

The `advban_ip` table provides all necessary fields:
- `ip` (varchar) - IP address being banned
- `ip_end` (varchar) - For range bans
- `expiry_date` (int) - Unix timestamp (0 = permanent)
- `reason` (text) - Stores `autoban:{rule_id}` for tracking

#### Edge Cases Handled

1. **Expired Bans**: Count toward history (prevents reset gaming)
2. Maintainers

- **Square360** - [https://square360.com](https://square360.com)

## **Multiple Rules**: Same IP across different rules counts as repeat offense
3. **IP Ranges**: Range bans handled separately from single IP bans
4. **Whitelist**: Autoban whitelist checked before ban logic runs
5. **Manual Bans**: Only autoban-triggered bans (`reason` contains 'autoban:') count toward escalation

#### Benefits

- **Unpredictable**: Random duration prevents attackers from timing retry attempts
- **Escalating**: Permanent bans for persistent threats
- **Auditable**: All decisions logged to watchdog
- **Reversible**: Admins can manually unban if needed
- **Performant**: Single database query per ban, minimal overhead

#### Configuration Options (Future)

Potential admin settings:
- Minimum ban duration (default: 5 days)
- Maximum ban duration (default: 25 days)
- Offense threshold for permanent ban (default: 2nd offense)
- Include/exclude specific rules from escalation
- Reset history after X days of clean behavior

#### Implementation Estimate

- **Complexity**: Moderate
- **Development Time**: 2-3 hours
- **Testing Requirements**: Requires simulated attacks to verify escalation
- **Dependencies**: None (uses existing autoban/advban APIs)

**Feasibility Rating:** 9/10 - Highly feasible with existing architecture

---

## Security Best Practices

This module provides automated protection against common attacks, but should be part of a comprehensive security strategy including:

- Regular Drupal core and contrib module updates
- Strong password policies
- HTTPS enforcement
- Web application firewall (WAF)
- Regular security audits
- Backup and disaster recovery plans

## License

Same as Drupal core (GPL-2.0-or-later)
