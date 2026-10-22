# Migrating from v1 to v2

v2.0 rebuilds this dataset on the GA **AWS Health Version Catalog API**
(`DescribeServiceLifecycle`). The structure changed; this guide maps v1 fields to v2.

**You are not forced to migrate immediately.** Releases are tagged, so if you pinned
to a v1.x tag you are unaffected. v2 is the recommended path forward.

## Top-level structure

| v1 | v2 |
|----|----|
| `services[]`, each with a nested `versions[]` | a flat `entries[]` array, one object per version |

## Per-version field mapping

| v1 field | v2 equivalent |
|----------|---------------|
| `serviceCode` (`aurora`) | `serviceCode` (`rds-aurora`) - the RDS/Aurora split is now explicit |
| Aurora MySQL `version` `"3"` / `"2"` (Aurora major-version labels) | `versionNumber` `"8.0"` / `"5.7"` (the MySQL-compatible version, matching the start of `EngineVersion` such as `8.0.mysql_aurora.3.x`). Aurora MySQL 8.4 already uses this numbering |
| `serviceName` | `title` (human name) plus `service` (raw API service name) |
| `version` | `version` (raw API id, may carry an engine prefix) plus `versionNumber` (parsed bare version) |
| `versionType` | removed |
| `status` | `status`, now a build-time computed snapshot; recompute from `lifecycleEvents` for automation |
| `standardSupportEnd` | the `date` of the `STANDARD_SUPPORT_END` (or `END_OF_SUPPORT`) event in `lifecycleEvents` |
| `extendedSupport.end` | the `date` of the `EXTENDED_SUPPORT_END` event |
| `extendedSupport.start` | removed (it equalled the standard-support-end date) |
| `postDeprecationBehavior` | the `behavior` block on the standard-support-end event |
| `postExtendedSupportBehavior` | the `behavior` block on the `EXTENDED_SUPPORT_END` event |
| `dateConfidence` | removed; see `provenance.confidence` |
| `sourceUrl` | each `behavior.sourceUrl` (per lifecycle event). `provenance.sourceUrl` is set only for documentation-sourced overlay entries and is `null` for API-sourced entries |

## Reading dates and status in v2

Dates live inside `lifecycleEvents`. To read "end of standard support," find the
event whose `lifecycleEventType` is `STANDARD_SUPPORT_END` and read its `date`. The
README includes a `status_from_events()` helper to recompute `status` on read.

## New in v2

- Amazon Bedrock foundation models and Amazon EMR coverage.
- A per-event `behavior` block describing what actually happens at each milestone,
  including conditional outcomes that depend on a per-resource setting.
- `engine` and `versionNumber` as first-class, filterable fields.
- A `provenance` block on every entry.
- Documentation-sourced entries for versions the API does not return. Today these
  are 13 deprecated Lambda runtimes (for example `python3.9`, `nodejs18.x`,
  `dotnet6`), Aurora PostgreSQL 18 and Aurora MySQL 8.4 (8.4 has no lifecycle
  dates yet, because the Aurora calendar does not give exact ones), and 9 Amazon MQ
  versions (ActiveMQ and RabbitMQ). They have
  `provenance.source` `DOCS_CRAWLED`, `provenance.confidence`
  `NEEDS_REVIEW`, and `provenance.sourceUrl` set to the AWS documentation page the
  dates came from. All other entries have `source` `API`, `confidence` `VERIFIED`,
  and `sourceUrl` `null`. If you only want API-sourced data, filter on
  `provenance.source == "API"`.
- Top-level `generatedAt` (the build date) next to `lastUpdated` (the date the data
  last changed). v1's `lastUpdated` was the refresh date; in v2 use `generatedAt`
  for freshness checks.

## Removed in v2

- **RabbitMQ 3.9, 3.10 and 3.11 on Amazon MQ.** v2 carries Amazon MQ from the
  Amazon MQ documentation (`serviceCode` `mq`), and those three versions are no
  longer in its version-support table; Amazon MQ upgraded every broker on them after
  end of support. Their dates remain in the `v1.0.0` release. In v2, MQ versions
  read `activemq/5.18` or `rabbitmq/3.13`, with `engine` `activemq` or `rabbitmq`
  (v1 had no `engine` for MQ).

## Cost

Cost is answered by joining the companion
[aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing)
dataset. Each of its v2 entries has a `match` block holding this dataset's
`serviceCode` and `engine` (copied exactly, including `null`), plus optional
`versionNumbers` and, for Aurora, `capacityMode`. Its Aurora serviceCode was renamed
to `rds-aurora` in the same release, and its DocumentDB engine is now `null` to match
this dataset.
