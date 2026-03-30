# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-03-30

### Added
- Initial Claude Code plugins marketplace with catalog structure
- `plugin-creator` plugin — scaffold new plugins from `drafts/` specs via `/plugin-creator:new-plugin`
- `uvify` plugin — transform Python scripts into self-contained uv-executable scripts (PEP 723) via `/uvify:adapt`
- Marketplace registry (`marketplace.json`) for plugin discovery

### Changed
- Replaced placeholder username with actual `ruslan-rv-ua` repository references
- Enabled `plugin-dev` plugin in project settings for plugin development workflow

[0.1.0]: https://github.com/ruslan-rv-ua/claude-plugins/releases/tag/v0.1.0
