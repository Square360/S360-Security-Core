# S360 Autoban Security Recipe

This recipe installs and configures comprehensive autoban security rules for Drupal websites.

## What This Recipe Does

1. **Installs Required Modules**:
   - Advanced Ban (advban) - Provides IP range banning
   - Autoban - Automated IP banning based on watchdog logs
   - Autoban Advanced Ban integration
   - Autoban Database Log integration

2. **Installs Security Configurations**:
   - 10 autoban rules targeting common attack patterns
   - Configures ban thresholds and time windows
   - Sets up IP range banning for malicious networks

3. **Applies Security Rules**:
   - PHP file scanning protection
   - WordPress vulnerability scanning protection
   - Joomla admin panel protection
   - Environment file (.env) protection
   - Git directory exposure protection
   - Database admin tool (phpMyAdmin) protection
   - Backup file theft protection
   - XML-RPC brute force protection
   - Webshell upload protection
   - Configuration file access protection

## Installation

### Using Drush

```bash
drush recipe web/modules/contrib/s360_security_core/recipes/autoban_security
```

### Using Drupal Core (Drupal 10.3+)

```bash
php web/core/scripts/drupal recipe web/modules/contrib/s360_security_core/recipes/autoban_security
```

## Post-Installation

After applying this recipe:

1. **Verify Rules are Active**:
   Visit `/admin/config/people/autoban` to see all configured rules

2. **Check Ban Activity**:
   ```bash
   drush watchdog:show --type=autoban --count=50
   ```

3. **View Banned IPs**:
   Visit `/admin/config/people/advban` or:
   ```bash
   drush sqlq "SELECT * FROM advban_ip_range ORDER BY created DESC LIMIT 20"
   ```

4. **Configure Cron**:
   Ensure cron is running regularly for autoban checks:
   ```bash
   drush cron
   ```

## Security Rules Overview

| Rule | Pattern | Threshold | Severity |
|------|---------|-----------|----------|
| php_scan | `*.php` | 2 | High |
| wordpress_scan | `/wp-*` | 2 | High |
| joomla_scan | `/administrator` | 2 | High |
| env_file_scan | `*.env*` | 1 | Critical |
| git_scan | `/.git*` | 1 | Critical |
| phpmyadmin_scan | `phpmyadmin\|pma\|adminer` | 2 | High |
| backup_scan | `*.sql\|*.zip\|*.bak` | 2 | High |
| xmlrpc_attack | `xmlrpc.php` | 1 | High |
| shell_scan | `shell\|c99\|backdoor*.php` | 1 | Critical |
| config_scan | `config*.php` | 2 | High |

## Maintenance

- Rules run automatically on cron
- Banned IPs expire based on advban expiry settings (default: varies by rule)
- Rules can be disabled individually at `/admin/config/people/autoban`
- Whitelist trusted IPs in autoban settings to prevent false positives

## Troubleshooting

**Issue**: Legitimate IPs getting banned
- **Solution**: Add to whitelist at `/admin/config/people/autoban/settings`

**Issue**: Rules not triggering
- **Solution**: Check that cron is running: `drush cron`

**Issue**: Too many false positives
- **Solution**: Increase thresholds for specific rules at `/admin/config/people/autoban`

## Related Documentation

- [Autoban Module Documentation](https://www.drupal.org/docs/contributed-modules/autoban)
- [Advanced Ban Module Documentation](https://www.drupal.org/docs/contributed-modules/advanced-ban)
- [CBEY Security Module README](../../README.md)
