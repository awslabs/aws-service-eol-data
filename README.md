# AWS Service End-of-Life Data

![License](https://img.shields.io/github/license/awslabs/aws-service-eol-data)
![Last commit](https://img.shields.io/github/last-commit/awslabs/aws-service-eol-data)
![Latest release](https://img.shields.io/github/v/release/awslabs/aws-service-eol-data?sort=semver)
[![Validate](https://github.com/awslabs/aws-service-eol-data/actions/workflows/validate.yml/badge.svg)](https://github.com/awslabs/aws-service-eol-data/actions/workflows/validate.yml)
[![Discussions](https://img.shields.io/github/discussions/awslabs/aws-service-eol-data)](https://github.com/awslabs/aws-service-eol-data/discussions)

> **NOT AN OFFICIAL AWS API.** This is a community republication of AWS lifecycle data, not an official AWS product, service, or commitment. The dates come from the first-party AWS Health Version Catalog API, plus a small, clearly marked set of entries from official AWS documentation for versions the API does not return yet (`provenance.source` `DOCS_CRAWLED`, with the page in `provenance.sourceUrl`). Each lifecycle event's `behavior.sourceUrl` links the official AWS documentation for what happens at that milestone.

A machine-readable dataset of AWS service version lifecycle dates: end of standard support, extended support periods, and what actually happens to your resources after support ends.

Version 2.0 mirrors the structure of the GA **AWS Health Version Catalog API** (`DescribeServiceLifecycle`) and enriches it with a normalized `serviceCode`, a parsed `engine` and `versionNumber`, a computed `status`, and a per-event `behavior` block describing the real-world outcome at each lifecycle milestone.

## Why This Exists

Managing version lifecycles across multiple AWS services requires consolidating information from many different documentation sources, each service publishes its own lifecycle dates in its own format. For organizations running workloads across EKS, RDS, Aurora, Lambda, OpenSearch, ElastiCache, MSK, DocumentDB, EMR, Amazon MQ, and Bedrock, keeping track of upcoming end-of-support dates at scale is a significant operational effort.

This repository provides a consolidated, machine-readable dataset of AWS service lifecycle dates, enabling customers to programmatically integrate lifecycle intelligence into their FinOps platforms, ITSM systems, compliance dashboards, and upgrade-planning workflows, all from a single source.

## Examples of use cases

- **Upgrade planning and roadmaps** - see which versions are approaching or past end of standard support and schedule upgrades before they hit Extended Support.
- **Dashboards and reports** - power executive and operational reporting in any BI or dashboard tool (QuickSight, Grafana, Tableau, or your own tooling) on lifecycle exposure across the fleet, each date traceable to the AWS Health Version Catalog or to a cited AWS documentation page.
- **CI/CD gates** - warn or block a pipeline when it deploys an image, runtime, or engine version that is end of life or nearing end of support.
- **Change management workflows** - feed structured lifecycle dates into change and approval processes so upgrades are planned and approved ahead of the deadline.
- **Security and compliance** - show that workloads run supported versions, and audit the fleet against a "no unsupported versions" policy with citable dates.
- **ITSM integration** - auto-raise tickets or tasks (ServiceNow, Jira, and similar) when a version enters Extended Support or reaches end of life.
- **Proactive alerting** - notify owners on lifecycle transitions, driven off structured dates instead of scraped web pages.
- **Fleet and account audits** - join your running versions (EKS, RDS, Aurora, Lambda, ElastiCache, OpenSearch, MSK, DocumentDB, EMR, Amazon MQ, Bedrock) against the lifecycle dates to find exposure.
- **Agents and chatbots** - answer "when does X lose support, and what happens if I don't upgrade?" from a structured source with a citation.
- **Pair with [aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing)** to answer "when does my version lose support, and what will it cost if I don't upgrade?"

## File Location

```
data/eol.json
```

## Schema

The dataset is a flat array of per-version `entries`. Each entry carries the API base fields verbatim plus the enrichment fields.

```json
{
  "schemaVersion": "2.0",
  "lastUpdated": "2026-10-19",
  "generatedAt": "2026-10-19",
  "_disclaimer": "Community republication of AWS lifecycle data, NOT an official AWS API. Dates come from the AWS Health Version Catalog API. ...",
  "entries": [
    {
      "service": "RDS",
      "version": "aurora-postgresql/15",
      "versionNumber": "15",
      "serviceCode": "rds-aurora",
      "engine": "postgresql",
      "title": "Amazon Aurora PostgreSQL 15",
      "recommendedVersion": null,
      "status": "STANDARD_SUPPORT",
      "lifecycleEvents": [
        {
          "lifecycleEventType": "SUPPORTED",
          "date": "2023-04-11",
          "regions": ["all"],
          "impactRisks": []
        },
        {
          "lifecycleEventType": "STANDARD_SUPPORT_END",
          "date": "2027-02-28",
          "regions": ["all"],
          "impactRisks": ["BILLING", "END_OF_SUPPORT"],
          "behavior": {
            "conditionalOn": "EngineLifecycleSupport",
            "branches": {
              "open-source-rds-extended-support": "CHARGES_APPLY",
              "open-source-rds-extended-support-disabled": "AUTO_UPGRADE"
            },
            "default": "CHARGES_APPLY",
            "note": "Default auto-enrolls in Extended Support (charges). If disabled, auto-upgrades to the next supported major instead.",
            "sourceUrl": "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/extended-support-overview.html",
            "confidence": "VERIFIED"
          }
        },
        {
          "lifecycleEventType": "EXTENDED_SUPPORT_END",
          "date": "2030-02-28",
          "regions": ["all"],
          "impactRisks": ["AVAILABILITY", "END_OF_SUPPORT"],
          "behavior": {
            "conditionalOn": null,
            "outcome": "AUTO_UPGRADE",
            "sourceUrl": "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/extended-support.html",
            "confidence": "VERIFIED"
          }
        }
      ],
      "provenance": {
        "source": "API",
        "confidence": "VERIFIED",
        "sourceUrl": null,
        "lastVerified": "2026-10-19"
      }
    }
  ]
}
```

### Top-level fields

| Field | Type | Description |
|-------|------|-------------|
| `schemaVersion` | string | Dataset schema version (`2.0`) |
| `lastUpdated` | date | The date the **data** last changed: an entry was added, removed, or changed (a moved date, a new status, a re-checked documentation source). A monthly rebuild that finds nothing new keeps the previous value |
| `generatedAt` | date | The date of the build that produced the file, that is, the last check against the API. Use this for freshness |
| `entries[]` | array | One object per version (below) |

The schema (`data/schema.json`) closes the vocabularies this dataset defines: `serviceCode` and every `behavior` outcome are enums, and a `DOCS_CRAWLED` entry must carry an official AWS `sourceUrl`. Fields copied from the API (`lifecycleEventType`, `impactRisks`) stay open strings, so a value AWS adds later does not fail your validation; the build warns about new values so they get behavior coverage.

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `service` | string | AWS Health service name, verbatim (e.g. `RDS`, `EKS`, `BEDROCK`) |
| `version` | string | The version identifier **as the API returns it**, including any engine prefix (e.g. `aurora-postgresql/15`, `redis/7.0`, `amazon.nova-2-sonic-v1:0`, `python3.12`) |
| `versionNumber` | string \| null | The bare version after the engine/provider prefix is stripped (e.g. `15`, `7.0`, `3.12`). The field you usually filter on. For Lambda it keeps any OS suffix (`8.al2`, `al2023`), and for MSK the `-tiered` suffix, because those runtimes and versions have their own dates. `null` for Bedrock (model ids are not reliably parseable) and for the bare Lambda `provided` runtime (no version part) |
| `serviceCode` | string | Normalized lowercase code: `rds`, `rds-aurora`, `eks`, `lambda`, `opensearch`, `elasticache`, `docdb`, `msk`, `emr`, `mq`, `bedrock` |
| `engine` | string \| null | The service's primary variant: DB engine (`mysql`, `postgresql`), cache engine (`redis`, `valkey`), `opensearch`/`elasticsearch`, broker engine (`activemq`, `rabbitmq`), Lambda runtime language (`python`, `nodejs`, ...), or Bedrock provider (`amazon`, `anthropic`, ...). `null` for single-variant services (`eks`, `msk`, `docdb`, `emr`) |
| `title` | string \| null | Human-readable name (from the API) |
| `recommendedVersion` | string \| null | The version AWS currently recommends for the service. **Not a per-version upgrade target**, and populated for only some services. Treat as a reference, not a guaranteed upgrade path |
| `status` | enum | Current lifecycle status, computed from the event dates (see below) |
| `lifecycleEvents[]` | array | The lifecycle milestones, verbatim from the API, each optionally enriched with a `behavior` block |
| `provenance` | object | `source` (`API`/`DOCS_CRAWLED`/`DERIVED`), `confidence`, `sourceUrl`, `lastVerified` |

### Joining to your inventory

- For every service except Bedrock, `serviceCode` + `engine` + `versionNumber` identifies one entry. The build fails if two entries ever share that key.
- For Bedrock, match the model ID against `version` (`versionNumber` is `null` for every model). Bedrock `engine` is the model ID prefix exactly as AWS publishes it, so one provider can appear under more than one `engine`: Moonshot AI models use both `moonshot` (`moonshot.kimi-k2-thinking`) and `moonshotai` (`moonshotai.kimi-k2.5`, `moonshotai.kimi-k3`). To select every model from one provider, filter on all of its prefixes.
- For Lambda, match the runtime identifier your functions report (for example `java8.al2`) against `version`. The OS suffix matters: `java8` (Amazon Linux) and `java8.al2` have different lifecycle dates, so they are separate entries and `versionNumber` keeps the suffix (`8` vs `8.al2`). To find every Java 8 runtime, filter on `engine == "java"` and a `versionNumber` that starts with `8`.
- `service` + `version` is unique for every entry.

### `lifecycleEvents[]`

Each event is one lifecycle milestone:

| Field | Type | Description |
|-------|------|-------------|
| `lifecycleEventType` | string | `SUPPORTED`, `STANDARD_SUPPORT_END`, `EXTENDED_SUPPORT_END`, `END_OF_SUPPORT`, `END_OF_LIFE`, `BLOCK_RESOURCE_CREATE`, `BLOCK_RESOURCE_UPDATE` |
| `date` | string \| null | ISO 8601 date. `null` for an acknowledged-but-undated version |
| `regions` | string[] | Regions the event applies to (often `["all"]`) |
| `impactRisks` | string[] | `AVAILABILITY`, `BILLING`, `END_OF_SUPPORT` |
| `description` | string | API description of the event (when present) |
| `behavior` | object | What actually happens at this event (see below). Absent on events with no modeled behavior, such as `SUPPORTED` |

### `behavior` (what happens at an event)

The `behavior` block is the dataset's core enrichment: the real-world outcome at a lifecycle event, which the version catalog does not encode. It comes in two shapes.

Non-conditional:

```json
"behavior": { "conditionalOn": null, "outcome": "AUTO_UPGRADE", "sourceUrl": "...", "confidence": "VERIFIED" }
```

Conditional (the outcome depends on a per-resource setting):

```json
"behavior": {
  "conditionalOn": "EngineLifecycleSupport",
  "branches": { "open-source-rds-extended-support": "CHARGES_APPLY",
                "open-source-rds-extended-support-disabled": "AUTO_UPGRADE" },
  "default": "CHARGES_APPLY",
  "sourceUrl": "...",
  "confidence": "VERIFIED"
}
```

Outcome vocabulary, defined by effect on **new** vs **existing** resources:

| Outcome | New resources | Existing resources |
|---------|---------------|--------------------|
| `CHARGES_APPLY` | - | Keep running on paid Extended Support |
| `AUTO_UPGRADE` | - | Force-migrated to a newer version |
| `DOMAIN_ISOLATED` | - | Still exists but unreachable by network; manual migrate (OpenSearch) |
| `RUNS_UNPATCHED` | may still be allowed | Keep running without patches |
| `CREATE_BLOCKED` | Blocked | Keep running **and** remain fully manageable via API (terminate / upgrade still work) |
| `UPDATE_BLOCKED` | - | Config/code updates blocked; resource keeps running |
| `EOL_INACCESSIBLE` | Blocked | Stop working; API calls fail (Bedrock EOL) |
| `ENTERS_LEGACY` | new provisioned throughput blocked | Keep working (Bedrock Legacy) |
| `ACCESS_LOSS_IF_INACTIVE` | - | May lose access if inactive 15+ days (Bedrock Legacy) |

### Status Values

| Value | Meaning |
|-------|---------|
| `STANDARD_SUPPORT` | Today is before the standard-support-end date (or no end date is announced) |
| `EXTENDED_SUPPORT` | Past standard support, within the extended-support window |
| `DEPRECATED` | Past standard support with no extended-support window (e.g. Lambda, MSK, EMR, Amazon MQ) |
| `END_OF_LIFE` | Past the extended-support end, or past an `END_OF_LIFE` event |

`status` is a build-time snapshot computed from the event dates. **For automated decisions, recompute it from `lifecycleEvents` at read time** rather than trusting the stored value (see the helper below).

### Provenance

| Field | Meaning |
|-------|---------|
| `source` | `API` (from the Health Version Catalog), `DOCS_CRAWLED` (documentation-sourced overlay for versions the API does not return), or `DERIVED` |
| `confidence` | `VERIFIED` for API-sourced; `NEEDS_REVIEW` for crawled/overlay |
| `sourceUrl` | The AWS documentation page the entry came from. Set when `source = DOCS_CRAWLED`, `null` for API entries |
| `lastVerified` | ISO date. For API entries, the build date. For `DOCS_CRAWLED` entries, the date someone last checked the cited page; the build warns when that is more than 35 days old |

#### Corrected dates

When the API returns a date that the AWS documentation contradicts, the dataset uses the documented date for that one event and marks it. The event carries a `correction` object, and the entry keeps `provenance.source: "API"` but has `provenance.confidence: "NEEDS_REVIEW"`:

```json
{
  "lifecycleEventType": "BLOCK_RESOURCE_CREATE",
  "date": "2027-07-29",
  "correction": {
    "apiDate": "2027-06-01",
    "source": "DOCS_CRAWLED",
    "sourceUrl": "https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html",
    "lastVerified": "2026-10-24"
  }
}
```

`apiDate` is the date the API returned. When the documentation gives the date only as a month (AWS marks those as approximate), `date` is `null`: the contradicted API date is removed and no day is invented. A correction stays only while the API still returns that date; once the API is fixed, the next monthly update drops it. To use API dates only, take `correction.apiDate` wherever a `correction` is present.

Corrected entries today:

| Entry | Event | API date | Dataset date | Source |
|-------|-------|----------|--------------|--------|
| Lambda `nodejs22.x` | `BLOCK_RESOURCE_CREATE` | 2027-06-01 | 2027-07-29 | [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) |
| Lambda `nodejs22.x` | `BLOCK_RESOURCE_UPDATE` | 2027-07-01 | 2027-08-31 | [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) |
| RDS `mariadb/10.5` | `STANDARD_SUPPORT_END` | 2026-02-28 | 2026-08-31 | [RDS for MariaDB versions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MariaDB.Concepts.VersionMgmt.html) |
| RDS `mariadb/11.8` | `STANDARD_SUPPORT_END` | 2030-06-30 | `null` (docs: June 2028) | [RDS for MariaDB versions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MariaDB.Concepts.VersionMgmt.html) |
| Aurora `aurora-postgresql/14` | `SUPPORTED` | 2022-03-10 | 2022-02-24 | [Aurora PostgreSQL release calendar](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/aurorapostgresql-release-calendar.html) |
| RDS `postgresql/16` | `SUPPORTED` | 2023-11-17 | 2023-11-20 | [RDS PostgreSQL release calendar](https://docs.aws.amazon.com/AmazonRDS/latest/PostgreSQLReleaseNotes/postgresql-release-calendar.html) |

### Reading dates and status

Dates live inside `lifecycleEvents`, not in flattened top-level fields. A small helper recreates `status` from the events:

```python
def status_from_events(events, today):
    d = {e["lifecycleEventType"]: e["date"] for e in events if e.get("date")}
    se = d.get("STANDARD_SUPPORT_END") or d.get("END_OF_SUPPORT")
    ee = d.get("EXTENDED_SUPPORT_END")
    eol = d.get("END_OF_LIFE")
    if eol and today >= eol:
        return "END_OF_LIFE"
    if ee:
        if today >= ee:
            return "END_OF_LIFE"
        if se and today >= se:
            return "EXTENDED_SUPPORT"
        return "STANDARD_SUPPORT"
    if se and today >= se:
        return "DEPRECATED"
    return "STANDARD_SUPPORT"
```

## Usage Examples

### Find versions entering Extended Support within 90 days

```python
import json
from datetime import datetime, timedelta

with open("data/eol.json") as f:
    data = json.load(f)

today = datetime.now().strftime("%Y-%m-%d")
cutoff = (datetime.now() + timedelta(days=90)).strftime("%Y-%m-%d")

for e in data["entries"]:
    se = next((ev["date"] for ev in e["lifecycleEvents"]
               if ev["lifecycleEventType"] == "STANDARD_SUPPORT_END" and ev.get("date")), None)
    if se and today <= se <= cutoff:
        print(f"{e['title']} ({e['serviceCode']} {e['versionNumber']}) - standard support ends {se}")
```

### Cross-reference your RDS fleet

Build a lookup keyed by `(serviceCode, engine, versionNumber)` and recompute status from events:

```python
import json, boto3
from datetime import datetime

with open("data/eol.json") as f:
    data = json.load(f)
today = datetime.now().strftime("%Y-%m-%d")

lookup = {(e["serviceCode"], e.get("engine"), e["versionNumber"]): e for e in data["entries"]}

rds = boto3.client("rds")
at_risk = []
for page in rds.get_paginator("describe_db_instances").paginate():
    for db in page["DBInstances"]:
        engine = "postgresql" if db["Engine"] in ("postgres", "aurora-postgresql") else \
                 "mysql" if db["Engine"] in ("mysql", "aurora-mysql") else db["Engine"]
        service_code = "rds-aurora" if db["Engine"].startswith("aurora") else "rds"
        major = db["EngineVersion"].split(".")[0] if engine == "postgresql" else \
                ".".join(db["EngineVersion"].split(".")[:2])
        info = lookup.get((service_code, engine, major))
        if info and status_from_events(info["lifecycleEvents"], today) != "STANDARD_SUPPORT":
            at_risk.append((db["DBInstanceIdentifier"], service_code, engine, major,
                            status_from_events(info["lifecycleEvents"], today)))

for ident, sc, eng, ver, st in at_risk:
    print(f"{ident}: {sc} {eng} {ver} is {st}")
```

### Scan EKS clusters across regions

```python
import boto3, json

with open("data/eol.json") as f:
    data = json.load(f)
eks_versions = {e["versionNumber"]: e for e in data["entries"] if e["serviceCode"] == "eks"}

ec2 = boto3.client("ec2")
for region in [r["RegionName"] for r in ec2.describe_regions()["Regions"]]:
    eks = boto3.client("eks", region_name=region)
    try:
        for name in eks.list_clusters()["clusters"]:
            v = eks.describe_cluster(name=name)["cluster"]["version"]
            info = eks_versions.get(v)
            print(name, region, v, info and info["status"])
    except Exception:
        continue
```

### Lambda runtime check in CI/CD

The Lambda runtime identifier is the raw `version` field:

```bash
#!/bin/bash
# Fails closed if the runtime is not found or is past support.
RUNTIME="$1"   # e.g. python3.9
STATUS=$(jq -r --arg rt "$RUNTIME" \
  '.entries[] | select(.serviceCode=="lambda" and .version==$rt) | .status' data/eol.json)

if [ -z "$STATUS" ] || [ "$STATUS" = "null" ]; then
  echo "BLOCKED: runtime $RUNTIME not found in lifecycle dataset."
  exit 1
elif [ "$STATUS" = "DEPRECATED" ] || [ "$STATUS" = "END_OF_LIFE" ]; then
  echo "BLOCKED: runtime $RUNTIME is $STATUS."
  exit 1
else
  echo "OK: runtime $RUNTIME is $STATUS."
fi
```

### What happens after support ends

Read the `behavior` on the relevant event:

```python
for e in data["entries"]:
    for ev in e["lifecycleEvents"]:
        b = ev.get("behavior")
        if b:
            outcome = b.get("outcome") or f"{b['default']} (default; conditional on {b['conditionalOn']})"
            print(f"{e['serviceCode']} {e['versionNumber']} @ {ev['lifecycleEventType']}: {outcome}")
```

## Covered Services

| Service | `serviceCode` | Engines / variants |
|---------|---------------|--------------------|
| Amazon RDS | `rds` | MySQL, PostgreSQL, MariaDB |
| Amazon Aurora | `rds-aurora` | MySQL-Compatible, PostgreSQL-Compatible |
| Amazon EKS | `eks` | Kubernetes versions |
| AWS Lambda | `lambda` | Python, Node.js, Java, .NET, Ruby, Go, and OS-only runtimes (see the note below) |
| Amazon ElastiCache | `elasticache` | Redis OSS, Valkey |
| Amazon OpenSearch Service | `opensearch` | OpenSearch, Elasticsearch (legacy) |
| Amazon MSK | `msk` | Apache Kafka versions |
| Amazon DocumentDB | `docdb` | Engine major/minor versions |
| Amazon EMR | `emr` | Release series (for example `6.x`, `7.10.0`) |
| Amazon MQ | `mq` | ActiveMQ, RabbitMQ (documentation-sourced, see the note below) |
| Amazon Bedrock | `bedrock` | Foundation models (by provider) |

The Health Version Catalog does not return most deprecated Lambda runtimes. v2.0 adds 13 of them from the [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) deprecated-runtimes table (for example `python3.9`, `nodejs18.x`, `dotnet6`, `java8`, `go1.x`), marked `DOCS_CRAWLED` / `NEEDS_REVIEW` with `provenance.sourceUrl` set. These are the deprecated runtimes whose block-function-update date has not passed. Runtimes that can no longer be updated at all (for example `dotnetcore3.1`, `nodejs12.x`) are not included, so a CI/CD check like the one above still fails closed on them as "not found".

The Health Version Catalog also does not return Amazon Aurora PostgreSQL 18 or Aurora MySQL 8.4 yet. v2.0 adds both from the Aurora release calendars, marked `DOCS_CRAWLED` / `NEEDS_REVIEW`. Aurora PostgreSQL 18 has its release, end of standard support and end of Extended Support dates. Aurora MySQL 8.4 has no lifecycle events yet, because the calendar gives end of standard support as a month only (April 2032) and Extended Support dates as to be determined; it reports `STANDARD_SUPPORT` until AWS publishes exact dates.

The Health Version Catalog also does not return Amazon RDS for MariaDB 12.3 yet. It is added from the [RDS for MariaDB versions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MariaDB.Concepts.VersionMgmt.html) page, marked `DOCS_CRAWLED` / `NEEDS_REVIEW`, with its RDS release date (2026-08-07) only: the page gives end of standard support as a month (June 2029).

Amazon MQ (ActiveMQ, RabbitMQ) is not in the Health Version Catalog. v2.0 carries every version listed in the Amazon MQ version-support tables, from the [ActiveMQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/activemq-version-management.html) and [RabbitMQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/rabbitmq-version-support.html) pages, marked `DOCS_CRAWLED` / `NEEDS_REVIEW`: ActiveMQ 5.15 to 5.19 and RabbitMQ 3.12, 3.13, 4.2 and 4.3. The `version` field reads `activemq/5.18` or `rabbitmq/3.13`, and `engine` is `activemq` or `rabbitmq`. Versions with an end-of-support date carry one `END_OF_SUPPORT` event whose `behavior` is `AUTO_UPGRADE`: Amazon MQ upgrades the brokers to the next supported version within 45 days. Versions the tables list without a date have no events yet. RabbitMQ 3.9, 3.10 and 3.11, which `v1.0.0` carried, are no longer in the Amazon MQ tables, so they are not in v2.0; their brokers were upgraded automatically after end of support.

## Companion dataset: cost

Pair with [aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing). Cost is **computed at consumption time by joining**, the two datasets are not merged. This dataset supplies the trigger (a version in Extended Support with a `CHARGES_APPLY` behavior); the pricing dataset supplies the rate. Each pricing entry carries a `match` block with this dataset's `serviceCode` and `engine` (copied exactly, so `null` matches `null`), plus optional `versionNumbers` and, for Aurora, `capacityMode`.

```python
import json
from datetime import datetime

with open("data/eol.json") as f:
    eol = json.load(f)
with open("../aws-service-extended-support-pricing/data/pricing.json") as f:
    pricing = json.load(f)
today = datetime.now().strftime("%Y-%m-%d")

def find_price(entry, capacity_mode="provisioned"):
    for s in pricing["services"]:
        m = s["match"]
        if m["serviceCode"] != entry["serviceCode"] or m["engine"] != entry.get("engine"):
            continue
        if m.get("versionNumbers") and entry["versionNumber"] not in m["versionNumbers"]:
            continue
        if m.get("capacityMode") and m["capacityMode"] != capacity_mode:
            continue
        return s
    return None

for e in eol["entries"]:
    ev = {x["lifecycleEventType"]: x.get("date") for x in e["lifecycleEvents"] if x.get("date")}
    se, ee = ev.get("STANDARD_SUPPORT_END"), ev.get("EXTENDED_SUPPORT_END")
    if not (se and ee and se <= today < ee):          # in Extended Support today?
        continue
    price = find_price(e)
    if price:
        print(f"{e['title']} ({e['serviceCode']} {e['versionNumber']}): rate model = {price['pricingUnit']}")
```

In SQL or Athena, remember that `NULL = NULL` is not true: join `engine` with `COALESCE(engine, '')` on both sides or `IS NOT DISTINCT FROM`.

## Companion lookup: Bedrock model capability (modality)

Model modality and live status (`ACTIVE`/`LEGACY`) are **not stored** in this dataset, they are region-scoped and best fetched live. Join them yourself from `bedrock:ListFoundationModels` on `modelId` (which equals this dataset's `version` for Bedrock entries):

```python
import boto3
fm = {m["modelId"]: m for m in boto3.client("bedrock", region_name="us-east-1")
      .list_foundation_models()["modelSummaries"]}
# join: eol entry.version == fm modelId  ->  inputModalities / outputModalities / modelLifecycle.status
# normalize throughput suffixes (:24k/:300k) off the modelId before joining
```

## Update Cadence

The dataset is updated monthly on a best-effort basis, sometimes more often (for example when AWS announces a lifecycle change). Each entry's dates come from the AWS Health Version Catalog (first-party) or, for overlay entries, from official docs cited in `provenance.sourceUrl`. Check the top-level `generatedAt` (the last check against the API). If it is more than 30 days old, re-verify time-sensitive dates before acting. `lastUpdated` tells you when the data itself last changed, which can be older when a monthly check finds nothing new.

## Contributing

We welcome contributions. See [CONTRIBUTING.md](CONTRIBUTING.md). The dataset is generated, so the most useful contribution is a report through GitHub Issues: a wrong date or `behavior` outcome, or a version or service the API does not cover, with a link to the official AWS documentation that shows the correct value.

## Important: Validation for Critical Decisions

If you use this data for automated decisions (blocking deployments, triggering upgrades, compliance reports, cost projections), you SHOULD:

1. **Recompute `status` from `lifecycleEvents`** rather than trusting the stored value.
2. **Verify critical dates** against the AWS documentation (the `behavior.sourceUrl` on each event, or `provenance.sourceUrl` on documentation-sourced entries) before acting.
3. **Pin to a tagged release** for production integrations.
4. **Check `generatedAt`** and re-verify if it is more than 30 days old.

## Disclaimer

This dataset is provided as-is for informational purposes. Dates are sourced from the first-party AWS Health Version Catalog API and public AWS documentation, and may change as AWS updates them. This is a community-maintained republication rather than an official AWS product. API-sourced dates come from `DescribeServiceLifecycle`; documentation-sourced entries and every `behavior` link the AWS page they came from, so you can confirm critical dates against official AWS documentation before making business decisions.

## License

Licensed under the Apache-2.0 License. See [LICENSE](LICENSE).

## Security

See [SECURITY.md](SECURITY.md).
