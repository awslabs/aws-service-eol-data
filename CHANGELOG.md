# Changelog

All notable changes to this dataset are recorded here. The format is based on
Keep a Changelog. Releases are dated; consumers should pin to a tagged release
rather than tracking `main`.

## [2.1.0] - 2026-11-26

### Added
- **Flat exports.** `data/exports/events.csv` and `data/exports/events.ndjson` have one row per lifecycle event, for Amazon Athena, Amazon QuickSight or a spreadsheet (README, "Flat exports").
- **Release change log.** `data/changes.json` lists what changed since the previous release: versions added and removed, status changes, moved event dates, and events added or removed (README, "Release change log").
- **Retired versions are kept.** A version that was in the previous release and that the API no longer returns stays in the dataset with `provenance.source: "DERIVED"`, its dates, a recomputed status and `lastVerified` set to the last API sighting. The API entry replaces it if the API returns the version again (README, "Retired versions"). No version is affected in this release.
- **Events added from the documentation.** A correction with `apiDate: null` adds an event the API does not return. Used for Lambda `nodejs26.x` and `python3.15`, which the API returns with no events: `END_OF_SUPPORT`, `BLOCK_RESOURCE_CREATE` and `BLOCK_RESOURCE_UPDATE` (2029-04-30, 2029-06-01, 2029-07-01 and 2029-06-30, 2029-07-31, 2029-08-31). `correction.apiDate` in `data/schema.json` now accepts `null`.
- **Amazon RDS for MySQL 9.7** (documentation-sourced, released 2026-11-18): end of standard support 2031-07-31, end of Extended Support 2034-07-31. It is replaced by the API entry once the API returns it.
- Three Amazon Bedrock models returned by the API: `anthropic.claude-fable-5-2`, `moonshotai.kimi-k3.1` and `openai.gpt-6.1-terra`.

### Fixed
- **Amazon EMR dates now follow the EMR documentation.** The API END_OF_SUPPORT and END_OF_LIFE dates do not show the EMR support policy changes of 31 July and 14 August 2026 (Bridge support to 31 August 2026; free Extended Support for 5.36, 6.6 to 6.15 and 7.0 to 7.10). Releases 7.0.0-7.2.x and 7.3.0 were `DEPRECATED` in v2.0.1 and are `STANDARD_SUPPORT` now. 2.x to 6.x take 2026-09-01 and 2027-09-01, and 7.0 to 7.10 take 2027-08-31 and 2028-08-31. The API lists 5.x and 6.x as ranges, so 5.36 and 6.6 to 6.15 (documented as 2027-06-30 and 2028-07-01) are understated until the ranges are split. Each corrected event carries a `correction` object with the API date. 1-day differences on 7.11 to 7.13 are not corrected.
- Lambda `nodejs22.x` `END_OF_SUPPORT`: 2027-04-30 -> 2027-05-30 (Lambda runtimes page).

### Changed
- **AWS fixed four of the v2.0.1 corrections in the API**, so they are removed: Lambda `nodejs22.x` block create and block update, RDS for MariaDB 10.5, and RDS for MariaDB 11.8, whose API date is now 2028-06-30 (the docs say June 2028). RDS for MariaDB 12.3 is now returned by the API (with end of standard support 2029-06-30), so its documentation-sourced entry is removed.
- **Status changes since v2.0.1:** EKS 1.31 is `END_OF_LIFE`; Lambda `dotnet8`, `dotnet9` and `python3.10` are `DEPRECATED`; 14 Amazon Bedrock models moved to `EXTENDED_SUPPORT` (Legacy dates 2026-09-09 to 2026-09-30, end of life 2027-03-09 to 2027-03-30) and two Jamba models are `END_OF_LIFE`. The full list is in `data/changes.json`.
- `generatedAt` and `lastUpdated` are 2026-11-26.

## [2.0.1] - 2026-10-24

### Fixed
- **Dates the API returns but the AWS documentation contradicts.** Found in community
  checks against the AWS docs (#15, #16, and a check of all 38 Lambda entries); thank you.
  The dataset now uses the documented dates (see README, "Corrected dates"):
  - Lambda `nodejs22.x`: BLOCK_RESOURCE_CREATE 2027-06-01 -> 2027-07-29 and
    BLOCK_RESOURCE_UPDATE 2027-07-01 -> 2027-08-31 (Lambda runtimes page).
    END_OF_SUPPORT (2027-04-30) was already correct.
  - RDS for MariaDB 10.5: STANDARD_SUPPORT_END 2026-02-28 -> 2026-08-31.
  - RDS for MariaDB 11.8: STANDARD_SUPPORT_END 2030-06-30 -> no date. The RDS for
    MariaDB page gives June 2028, a month only, so the API date (two years later) is
    removed and no day is invented. It gets a date once AWS publishes an exact day.
  - Aurora PostgreSQL 14: SUPPORTED 2022-03-10 -> 2022-02-24 (Aurora release calendar).
  - RDS for PostgreSQL 16: SUPPORTED 2023-11-17 -> 2023-11-20 (RDS release calendar).
- Not changed: Aurora PostgreSQL 16 end of Extended Support stays 2032-02-29. The API
  and the RDS PostgreSQL calendar give 29 February 2032; the Aurora calendar gives
  28 February 2032. The AWS pages disagree, so the API date stands (#16).

### Added
- **Amazon RDS for MariaDB 12.3** (documentation-sourced): release date 2026-08-07 only;
  end of standard support is published as a month (June 2029).
- **Corrected dates.** A lifecycle event whose API date the AWS documentation
  contradicts now uses the documented date and carries a `correction` object
  (`apiDate`, `source: "DOCS_CRAWLED"`, `sourceUrl`, `lastVerified`). The entry keeps
  `provenance.source: "API"` with `confidence: "NEEDS_REVIEW"`. A correction is dropped
  automatically once the API returns the documented date. `data/schema.json` defines
  the new object; it is additive, so existing consumers are unaffected. See README,
  "Corrected dates".

## [2.0.0] - 2026-10-21

### Changed
- **BREAKING: rebuilt on the GA AWS Health Version Catalog API.** The dataset now
  mirrors `DescribeServiceLifecycle` and is a flat array of per-version `entries`
  (previously a nested `services[]` -> `versions[]` structure). Dates now live inside
  each entry's `lifecycleEvents`; the flattened `standardSupportEnd` /
  `extendedSupport` fields are removed. See `MIGRATING.md` for the full field map.
- `serviceCode` normalized, with Aurora split from RDS as `rds-aurora` (was `aurora`).
- `versionType` and `dateConfidence` removed. `status` is now a computed snapshot
  (recompute from `lifecycleEvents` for critical automation).
- Post-support behavior moved from the flat `postDeprecationBehavior` /
  `postExtendedSupportBehavior` fields to a per-event `behavior` block merged inside
  the matching `lifecycleEvents` entry, with conditional branches where the outcome
  depends on a per-resource setting.

### Added
- Coverage for **Amazon Bedrock** foundation models and **Amazon EMR**.
- `version` (raw API id) plus `versionNumber` (parsed), `engine` as a first-class
  field, and a `provenance` block (`source`, `confidence`, `sourceUrl`, `lastVerified`).
- Engine-level behavior overrides in the support model. RDS for MariaDB has no
  Extended Support (RDS Extended Support covers MySQL and PostgreSQL only), so its
  `STANDARD_SUPPORT_END` behavior is `AUTO_UPGRADE`, not the RDS default
  `CHARGES_APPLY`. Note that the API still tags this event `BILLING`.
- **13 deprecated AWS Lambda runtimes** that the Health Version Catalog does not
  return: `nodejs20.x`, `ruby3.2`, `python3.9`, `nodejs18.x`, `dotnet6`,
  `python3.8`, `nodejs16.x`, `java8`, `go1.x`, `provided`, `ruby2.7`, `nodejs14.x`,
  `python3.7`. They come from the
  [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
  deprecated-runtimes table and are marked `provenance.source` `DOCS_CRAWLED`,
  `confidence` `NEEDS_REVIEW`, with `provenance.sourceUrl` set to that page. Each
  carries `END_OF_SUPPORT`, `BLOCK_RESOURCE_CREATE`, and `BLOCK_RESOURCE_UPDATE`
  events. The scope is deprecated runtimes whose block-function-update date has not
  passed; runtimes that can no longer be updated (for example `dotnetcore3.1`) and
  container-only runtimes with no block dates (`dotnet7`) are not included. If the
  API later returns one of these runtimes, the API entry replaces the overlay entry.
  Thanks to @Anshuman-Prakash, whose PR #13 raised the missing runtimes.
- A uniqueness check in the build: `serviceCode` + `engine` + `versionNumber` must
  identify one entry for every service except Bedrock (match Bedrock on `version`),
  and `service` + `version` must be unique. The README's new "Joining to your
  inventory" section explains which key to use, including why Lambda keeps the OS
  suffix in `versionNumber` (`java8` and `java8.al2` have different dates).
- **Amazon Aurora PostgreSQL 18 and Aurora MySQL 8.4**, current GA versions that are
  in the Aurora release calendars but not yet in the Health Version Catalog. Same
  overlay rules (`DOCS_CRAWLED` / `NEEDS_REVIEW`, `provenance.sourceUrl` set to the
  release calendar). Aurora PostgreSQL 18 carries its release (2026-06-11), end of
  standard support (2031-02-28) and end of Extended Support (2034-02-28). Aurora
  MySQL 8.4 carries no lifecycle events yet: the calendar gives end of standard
  support as a month only (April 2032) and Extended Support dates as to be
  determined, and the dataset adds only exact dates. It reports
  `STANDARD_SUPPORT` until AWS publishes exact dates.
- **Amazon MQ** (ActiveMQ, RabbitMQ), which is not in the Health Version Catalog:
  every version in the Amazon MQ version-support tables (ActiveMQ 5.15 to 5.19,
  RabbitMQ 3.12, 3.13, 4.2, 4.3), marked `DOCS_CRAWLED` / `NEEDS_REVIEW`.
  `serviceCode` `mq`, `engine` `activemq` or `rabbitmq`, `version` such as
  `activemq/5.18`. Dated versions carry an `END_OF_SUPPORT` event with behavior
  `AUTO_UPGRADE` (Amazon MQ upgrades brokers to the next supported version within
  45 days); versions listed without a date have no events yet. RabbitMQ 4.3 is new
  since `v1.0.0`.
- A verification date per documentation-sourced entry: `provenance.lastVerified` is
  the day the cited page was last checked, and the build warns when that is more
  than 35 days before the build, so a stale documentation date cannot ride along
  unnoticed. API entries are not affected.
- `generatedAt`: the date of the build. `lastUpdated` now means the date the data
  last changed (an entry added, removed, or changed, ignoring the per-build
  `lastVerified` of API entries), so a monthly rebuild that finds nothing new keeps
  it. Use `generatedAt` for freshness.
- A stricter schema. `serviceCode` and every `behavior` outcome (`outcome`,
  `default`, and each branch) are enums, and a `DOCS_CRAWLED` entry must have
  `confidence` `NEEDS_REVIEW` and an official AWS `sourceUrl`. `lifecycleEventType`
  and `impactRisks` come from the API and stay open, so a new AWS value does not
  fail validation; the build warns about new values instead.
- README note on Bedrock `engine`: it is the model ID prefix as AWS publishes it, so
  Moonshot AI models appear under both `moonshot` and `moonshotai`.

### Removed
- **RabbitMQ 3.9, 3.10 and 3.11 on Amazon MQ.** `v1.0.0` carried them; the Amazon
  MQ version-support table no longer lists them, so their dates cannot be
  re-verified, and Amazon MQ upgraded every broker on them after end of support.
  The dates remain available in the `v1.0.0` release.

### Migration
- v1.x consumers pinned to a release tag are unaffected. v2 is the recommended path.
  See `MIGRATING.md` for the v1 -> v2 field mapping.

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
