# Changelog

All notable changes to this dataset are recorded here. The format is based on
Keep a Changelog. Releases are dated; consumers should pin to a tagged release
rather than tracking `main`.

## [1.0.0] - 2026-09-16

### Added
- Initial public release of the AWS service end-of-support lifecycle dataset.
- Lifecycle dates for Amazon EKS, RDS, Aurora, Lambda, ElastiCache, OpenSearch
  Service, MSK, Amazon MQ, and DocumentDB.
- Per-version lifecycle status, end of standard support, Extended Support start
  and end, post-deprecation and post-extended-support behavior, a confidence
  level, and an official `sourceUrl` on every entry.
- JSON schema (`data/schema.json`) and a worked example that joins the companion
  pricing dataset on `serviceCode` and `engine`.
