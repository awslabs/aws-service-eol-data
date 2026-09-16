# AWS Service End-of-Life Data

![License](https://img.shields.io/github/license/awslabs/aws-service-eol-data)
![Last commit](https://img.shields.io/github/last-commit/awslabs/aws-service-eol-data)
![Latest release](https://img.shields.io/github/v/release/awslabs/aws-service-eol-data?sort=semver)
[![Validate](https://github.com/awslabs/aws-service-eol-data/actions/workflows/validate.yml/badge.svg)](https://github.com/awslabs/aws-service-eol-data/actions/workflows/validate.yml)
[![Discussions](https://img.shields.io/github/discussions/awslabs/aws-service-eol-data)](https://github.com/awslabs/aws-service-eol-data/discussions)

> **NOT AN OFFICIAL AWS API.** This is a community-maintained dataset provided on a best-effort basis. It is not an official AWS product, service, or commitment. Always verify dates against the official AWS documentation linked in each entry's `sourceUrl` field before making business decisions.

A machine-readable dataset of AWS service version lifecycle dates: end of standard support, extended support periods, and post-deprecation behaviors.

## Why This Exists

Managing version lifecycles across multiple AWS services requires consolidating information from many different documentation sources, each service publishes its own lifecycle dates in its own format. For organizations running workloads across EKS, RDS, Lambda, OpenSearch, and ElastiCache, keeping track of upcoming end-of-support dates at scale is a significant operational effort.

This repository provides a consolidated, machine-readable dataset of AWS service lifecycle dates, enabling customers to programmatically integrate lifecycle intelligence into their FinOps platforms, ITSM systems, compliance dashboards, and upgrade planning workflows, all from a single source.


### File Location

```
data/eol.json
```

### Schema

```json
{
  "schemaVersion": "1.0",
  "lastUpdated": "2026-06-10",
  "services": [
    {
      "serviceCode": "eks",
      "serviceName": "Amazon EKS",
      "engine": null,
      "versionType": "kubernetesVersion",
      "versions": [
        {
          "version": "1.32",
          "status": "STANDARD_SUPPORT",
          "standardSupportEnd": "2026-03-23",
          "extendedSupport": {
            "start": "2026-03-24",
            "end": "2027-03-23"
          },
          "postDeprecationBehavior": "CHARGES_APPLY",
          "postExtendedSupportBehavior": "AUTO_UPGRADE",
          "dateConfidence": "COMMITTED",
          "sourceUrl": "https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html"
        }
      ]
    }
  ]
}
```

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `serviceCode` | string | AWS service identifier (e.g., `eks`, `rds`, `lambda`, `elasticache`, `opensearch`) |
| `serviceName` | string | Human-readable service name |
| `engine` | string \| null | Database engine for multi-engine services (e.g., `mysql`, `postgresql`, `redis`) |
| `versionType` | string | What the version represents: `kubernetesVersion`, `engineMajorVersion`, `runtime`, `engineVersion` |
| `versions[].version` | string | The version identifier |
| `versions[].status` | enum | Current lifecycle status (see below) |
| `versions[].standardSupportEnd` | string | ISO 8601 date, last day of standard support |
| `versions[].extendedSupport` | object \| null | Extended support period (`start`, `end`) or `null` if not available |
| `versions[].postDeprecationBehavior` | enum | What happens after standard support ends (see below) |
| `versions[].postExtendedSupportBehavior` | enum \| null | What happens after extended support ends (see below). `null` for services without extended support |
| `versions[].dateConfidence` | enum | How certain the dates are (see below) |
| `versions[].sourceUrl` | string | Public AWS documentation URL verifying the dates |

### Status Values

| Value | Meaning |
|-------|---------|
| `STANDARD_SUPPORT` | Version is fully supported with regular patches and updates |
| `EXTENDED_SUPPORT` | Version is past standard support; patches continue but extra charges apply |
| `DEPRECATED` | Version is no longer supported; no patches, limited functionality |
| `END_OF_LIFE` | Version is fully unsupported; may be forcibly upgraded or blocked |

### Post-Deprecation Behavior

| Value | What happens if the customer takes no action after standard support ends |
|-------|--------------------------------------------------------------------------|
| `CHARGES_APPLY` | Resource keeps running with patches, but Extended Support fees begin automatically |
| `AUTO_UPGRADE` | Resource is forcibly upgraded to the next supported version |
| `CREATE_BLOCKED` | Existing resources continue running, but new resources cannot be created on this version |
| `NO_PATCHES` | Resource keeps running indefinitely without security patches |

### Post-Extended-Support Behavior

| Value | What happens if the customer takes no action after extended support ends |
|-------|-------------------------------------------------------------------------|
| `AUTO_UPGRADE` | Resource is forcibly upgraded to the next supported version |
| `DOMAIN_ISOLATED` | Resource becomes inaccessible until manually migrated to a supported version |
| `NO_PATCHES` | Resource continues running without any patches or security updates |
| `null` | Service does not offer extended support (e.g., Lambda) |

### Date Confidence

| Value | Meaning |
|-------|---------|
| `COMMITTED` | Date is final and publicly announced |
| `PROJECTED` | Date is estimated based on historical patterns; may change |
| `TENTATIVE` | Version is tracked in AWS lifecycle documentation but dates have not yet been announced |

### Interpreting `null` Dates

When `standardSupportEnd` or `extendedSupport` is `null`, it means AWS has acknowledged the version in its lifecycle documentation but has not yet announced specific dates. Consumers should:

- Monitor the `sourceUrl` for announcements
- Treat these versions as "currently supported with no announced end date"
- Use `dateConfidence: "TENTATIVE"` as the filter to identify these entries programmatically

## Usage Examples

### Find all versions entering Extended Support within 90 days

```python
import json
from datetime import datetime, timedelta

with open('data/eol.json') as f:
    data = json.load(f)

cutoff = (datetime.now() + timedelta(days=90)).strftime('%Y-%m-%d')
today = datetime.now().strftime('%Y-%m-%d')

for service in data['services']:
    for v in service['versions']:
        eos = v['standardSupportEnd']
        if today <= eos <= cutoff:
            print(f"{service['serviceName']} {v['version']} - EOS: {eos}")
```

### Integrate with Terraform pre-plan checks

```bash
# Check if your EKS version is approaching EOL before terraform apply
EKS_VERSION="1.29"
STATUS=$(jq -r ".services[] | select(.serviceCode==\"eks\") | .versions[] | select(.version==\"$EKS_VERSION\") | .status" data/eol.json)

if [ "$STATUS" != "STANDARD_SUPPORT" ]; then
  echo "WARNING: EKS $EKS_VERSION is $STATUS"
  exit 1
fi
```

### Feed into ServiceNow / Jira automation

```python
# Create tickets for versions entering extended support
for service in data['services']:
    for v in service['versions']:
        if v['status'] == 'EXTENDED_SUPPORT' and v['postDeprecationBehavior'] == 'CHARGES_APPLY':
            create_ticket(
                title=f"Upgrade {service['serviceName']} {v['version']} - Extended Support charges active",
                due_date=v['extendedSupport']['end'],
                priority="high"
            )
```

## Covered Services

| Service | Versions Tracked |
|---------|-----------------|
| Amazon EKS | Kubernetes versions |
| Amazon RDS | MySQL, PostgreSQL |
| Amazon Aurora | MySQL-Compatible, PostgreSQL-Compatible |
| AWS Lambda | Runtime versions (Python, Node.js, Java, .NET, Ruby) |
| Amazon ElastiCache | Redis |
| Amazon OpenSearch Service | OpenSearch, Elasticsearch (legacy) |
| Amazon MSK | Apache Kafka versions |
| Amazon MQ | ActiveMQ, RabbitMQ |
| Amazon DocumentDB | Engine major versions |

## Companion dataset

For end-to-end automation, pair this with the companion pricing dataset,
[aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing),
which provides the Extended Support pricing models. Together they answer two
questions: "When does my version lose support?" and "What will it cost if I don't upgrade?"

### Example: cost a version that is in Extended Support

Join on `serviceCode` + `engine`. For per-instance services (DocumentDB by instance family, ElastiCache by node type) the rate lives in `instanceRates[region][instanceKey][tier]`:

```python
import json
from datetime import datetime

with open('data/eol.json') as f:
    lifecycle = json.load(f)
with open('../aws-service-extended-support-pricing/data/pricing.json') as f:
    pricing = json.load(f)

today = datetime.now().strftime('%Y-%m-%d')

# Illustrative resource: 10 DocumentDB r5 instances, 2 vCPU each, in us-east-1.
region, family, vcpus, instances, hours = 'us-east-1', 'r5', 2, 10, 730

for lc in lifecycle['services']:
    if lc['serviceCode'] != 'docdb':
        continue
    price = next((s for s in pricing['services']
                  if s['serviceCode'] == lc['serviceCode']
                  and s.get('engine') == lc.get('engine')), None)
    if not price or 'instanceRates' not in price:
        break
    rate = price['instanceRates'][region][family]['year1_2']  # $/vCPU-hour
    for v in lc['versions']:
        ext = v.get('extendedSupport') or {}
        in_es = (v.get('standardSupportEnd') or '') <= today < (ext.get('end') or '')
        if in_es and v.get('postDeprecationBehavior') == 'CHARGES_APPLY':
            monthly = rate * vcpus * instances * hours
            print(f"{lc['serviceName']} {v['version']} in Extended Support -> "
                  f"{instances}x {family} ({vcpus} vCPU) in {region}: "
                  f"${monthly:,.2f}/month (Year 1-2)")
```

For ElastiCache, use `serviceCode == 'elasticache'` and index `instanceRates` by node type (e.g. `instanceRates[region]['cache.r6g.large']['year1_2']`) times node count instead of vCPUs.

## Update Cadence

This dataset is updated manually as AWS announces new lifecycle dates. Each entry includes a `sourceUrl` pointing to the official AWS documentation for independent verification.

**Freshness signal:** Check the top-level `lastUpdated` field. If it's more than 30 days old, dates should be re-verified against source URLs.

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Common contributions:
- Adding new service versions when AWS announces EOL dates
- Updating dates when AWS publishes changes
- Adding new services as they announce lifecycle policies
- Reporting inaccuracies via GitHub Issues

**Every contribution must include a `sourceUrl` pointing to official AWS documentation.**

## Disclaimer

This dataset is provided as-is for informational purposes on a best-effort basis. Dates are sourced from public AWS documentation and may change without notice. This project is community-maintained and there is no guarantee of completeness, accuracy, or timeliness of updates. Always verify critical dates against the official AWS documentation linked in each entry's `sourceUrl` field before making business decisions.

## Important: Validation for Critical Decisions

If you are using this data for automated decisions (blocking deployments, triggering upgrades, generating compliance reports, or projecting costs), you SHOULD:

1. **Validate dates against `sourceUrl`** - each entry links to the official AWS documentation page. Cross-reference before acting on the data.
2. **Validate `status` against date fields programmatically** - rather than relying solely on the pre-computed `status` field, compute expected status from `standardSupportEnd` and `extendedSupport` dates in your integration.
3. **Pin to a tagged release** - for production integrations, pin to a specific release rather than tracking the main branch.
4. **Check `lastUpdated`** - if the dataset is more than 30 days old, re-verify dates against source URLs before acting.

## License

This project is licensed under the Apache-2.0 License. See the [LICENSE](LICENSE) file for details.

## Security

See [SECURITY.md](SECURITY.md) for our security policy and vulnerability reporting process.
