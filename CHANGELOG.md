# Changelog

All notable changes to `filament-database-mail` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
## [2.0.0] - 2025-06-15
### Upgraded
- Upgraded filament to v4
- Upgraded tailwind to v4

### Changed
- MailTemplateResource canActivate & canActivateAny to getActivateAuthorizationResponse & getActivateAnyAuthorizationResponse

## [1.2.1] - 2025-06-05
### Fixed
- Fixed MailTemplateResource cluster error on package:discover command

## [1.2.0] - 2025-05-31
### Added
- Added support for hidden properties
- Added option to set a MailTemplateResource cluster from plugin configuration

## [1.1.3] - 2025-05-08
### Fixed
- Fixed MailTemplateResource pages resource error if plugin is registered on non default panel

## [1.1.2] - 2025-04-15
### Changed
- Extracted MailTemplateResource table columns and filters to separate methods

## [1.1.1] - 2025-04-01
### Fixed
- Fixed MailTemplateResource pages to load resource from plugin

## [1.1.0] - 2025-03-08
### Added
- Added authorization to activate/deactivate mail template actions

## [1.0.0] - 2025-03-08
### Added
- Added initial version of Filament Database Mail

[unreleased]: https://github.com/martinpetricko/filament-database-mail/compare/2.0.0...HEAD
[2.0.0]: https://github.com/martinpetricko/filament-database-mail/compare/1.2.1...2.0.0
[1.2.1]: https://github.com/martinpetricko/filament-database-mail/compare/1.2.0...1.2.1
[1.2.0]: https://github.com/martinpetricko/filament-database-mail/compare/1.1.3...1.2.0
[1.1.3]: https://github.com/martinpetricko/filament-database-mail/compare/1.1.2...1.1.3
[1.1.2]: https://github.com/martinpetricko/filament-database-mail/compare/1.1.1...1.1.2
[1.1.1]: https://github.com/martinpetricko/filament-database-mail/compare/1.1.0...1.1.1
[1.1.0]: https://github.com/martinpetricko/filament-database-mail/compare/1.0.0...1.1.0
[1.0.0]: https://github.com/martinpetricko/filament-database-mail/releases/tag/1.0.0
