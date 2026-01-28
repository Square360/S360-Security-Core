# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-01-28

### Added
- Initial release of S360 Security Core module
- 10 preconfigured autoban rules for common attack patterns
- PHP vulnerability scan protection
- WordPress path scanning protection
- Joomla admin panel access protection
- Environment file (.env) access protection
- Git directory access protection
- Database admin tools (phpMyAdmin, Adminer) protection
- Backup file theft protection
- XML-RPC brute force protection
- Webshell upload protection
- Config file access protection
- IP range banning support via Advanced Ban module
- Automated installation via Drupal recipes
- Comprehensive status report integration
- Cron monitoring for security checks
- Full Drupal 10 and 11 compatibility
- PHP 8.1, 8.2, and 8.3 support

### Security
- Critical file protection with single-attempt ban threshold
- IP range banning to block entire malicious networks
- Rules apply only to anonymous users to avoid disrupting authenticated access

[Unreleased]: https://packages.square360.com
[1.0.0]: https://packages.square360.com
