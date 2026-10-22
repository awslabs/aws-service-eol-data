# Roadmap

Planned and under-consideration work. This is directional, not a commitment to dates
or scope. Feedback and requests are welcome via GitHub Issues and Discussions.

The dataset is updated monthly on a best-effort basis, sometimes more often. Every
item below follows the same rule as the rest of the dataset: lifecycle dates come
from the AWS Health Version Catalog API or, where the API does not cover a version,
from official AWS documentation cited in `provenance.sourceUrl`.

## Shipped in v2.0

- Rebuilt on the AWS Health Version Catalog API (see `MIGRATING.md`).
- A documentation-sourced overlay for versions the API does not return: 13 deprecated
  AWS Lambda runtimes, Amazon Aurora PostgreSQL 18, Aurora MySQL 8.4 (undated until
  AWS publishes exact dates), and Amazon MQ (ActiveMQ and RabbitMQ, 9 versions).
  Overlay entries are marked `DOCS_CRAWLED` / `NEEDS_REVIEW` and are replaced by the
  API entry once the API returns that version.
- The cost join with the companion pricing dataset, through its `match` block.
- **Stricter schema** - enums for `serviceCode` and every `behavior` outcome, and an
  official AWS `sourceUrl` required on documentation-sourced entries. Lifecycle
  event types and impact risks come from the API and stay open; the build warns
  when AWS adds a new value.
- **Build safeguards** - the build fails if two entries share a join key, and warns
  when a documentation-sourced entry has not been re-checked for more than 35 days.
- **`generatedAt` and `lastUpdated`** - when the dataset was last checked, and when
  the data last changed.

## Planned

- **More coverage through the documentation overlay** - Amazon RDS for SQL Server,
  Oracle, and Db2 (every major or minor version the RDS documentation gives a
  deprecation date for; none of them has RDS Extended Support). Only dates the AWS
  documentation states are added.
- **Retired versions** - keeping a version in the dataset after the API stops
  returning it (for example EKS 1.29), because many workloads keep running old
  versions long after support ends.
- **Release change log** - a generated `changes.json` per release listing moved dates,
  added and removed versions, and status changes.
- **Freshness automation** - a scheduled monthly rebuild with tests, validation, and
  the pricing join check. A person reviews each rebuild before it is published.
- **Flat exports** - one row per lifecycle event, as CSV and NDJSON, for Amazon
  Athena, Amazon QuickSight, or a spreadsheet.
- **Upcoming-milestones view** - a generated file of lifecycle events in the next
  180 days. For automation, computing from `lifecycleEvents` stays the recommended
  path, because a generated view is only as fresh as the last build.
- **Operational metadata** - per service, engine, and upgrade type (minor or major):
  upgrade complexity, upgrade path and successor version, upgrade methods, expected
  upgrade time, expected downtime, what you control if you upgrade early versus what
  AWS does at the date (and with how much notice), rollback options, and what one
  upgrade touches. This lets you rank your fleet by how hard each upgrade is, not
  only by date. Each row cites AWS documentation; where the docs give no figure, the
  field is left out.

## Under consideration

- **Amazon ElastiCache for Memcached**, if the ElastiCache documentation publishes
  end dates per version.
- **What happens when a forced upgrade fails** - a per-event fallback outcome, where
  the AWS documentation describes one.
- **Security metadata** - per lifecycle milestone: whether security patches and
  technical support continue, which fixes continue (for example all, critical and
  high CVEs only, OS only, or none), a severity derived from that by a fixed rule,
  and compliance references (for example PCI DSS, HIPAA, SOC 2, FedRAMP) only where
  the framework's own text is cited. Exposure (internet-facing, data
  classification) stays in your inventory; join it on `serviceCode`, `engine`, and
  `versionNumber`.
- **Successor version** - the version a resource moves to next, only where AWS
  names it.
- **Amazon Bedrock model identity** - the model ID split into provider, model
  family, variant, and version, so retiring models can be filtered by family.
- A helper for the Amazon Bedrock model-capability (modality) join against
  `ListFoundationModels`.

## Companion dataset

- Cost is answered by the companion
  [aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing)
  dataset: each of its entries has a `match` block naming the `serviceCode`, `engine`,
  and optionally `versionNumbers` and `capacityMode` it prices.
