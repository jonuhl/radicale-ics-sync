# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.1.0] - 2026-06-18

### Added
- Initial release
- Sync external ICS feeds into Radicale collections on a configurable interval
- Filter events by include/exclude regex patterns matched against SUMMARY
- Preserve local edits to upstream events until the upstream event itself changes
- Leave local-only events untouched
- Delete events that disappear from the upstream feed or are filtered out
- Support multiple feeds per collection and multiple collections