# Contributing to AWS Service EOL Data

Thank you for your interest in contributing! This project relies on community reports to keep lifecycle data accurate.

## How this dataset is maintained

`data/eol.json` is **generated** from the AWS Health Version Catalog and a curated behavior-enrichment layer, then published here. The generator is maintained by AWS; this repository ships the **dataset** (`data/eol.json` + `data/schema.json`) and its documentation. Because the data is generated and regenerated from source, we do **not** accept pull requests that hand-edit `data/eol.json`, those changes would be overwritten on the next build.

The best way to contribute is to **report an inaccuracy**, and we will correct it at the source and republish.

## Reporting an inaccuracy

Open a [GitHub Issue](https://github.com/awslabs/aws-service-eol-data/issues) including:

- the `service` / `serviceCode` and the `version` (or `versionNumber`) affected;
- what is wrong and what it should be, for example:
  - a wrong or missing lifecycle **date**,
  - a wrong **behavior** outcome (what happens at end of support),
  - a missing **version** or **service**;
- a link to the **official AWS documentation** that shows the correct value (a specific page, not a service landing page).

Behavior outcomes and coverage gaps are the most valuable reports, those are the enrichment this dataset adds on top of the raw lifecycle dates.

## What makes a good report

- Cite official AWS documentation (`docs.aws.amazon.com` or an official AWS page).
- Be specific about the version: use the `version` string as it appears in the dataset (e.g. `aurora-postgresql/15`, `redis/7.0`, `python3.12`).
- For dates: note that lifecycle dates are first-party (from the AWS Health Version Catalog). If a date looks wrong it may be an upstream value, but please still report it.

## What we don't track here

- Pricing. Use the companion [aws-service-extended-support-pricing](https://github.com/awslabs/aws-service-extended-support-pricing) dataset: each of its entries has a `match` block naming the `serviceCode`, `engine` (copied exactly, including `null`), and optionally `versionNumbers` and `capacityMode` it prices.
- Customer-specific data or resource counts.

## Using the data

See [README.md](README.md) for the schema, field reference, and usage examples (including how to recompute `status` from `lifecycleEvents` and how to join cost and Bedrock modality).

## Code of Conduct

This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct). For more information, see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact opensource-codeofconduct@amazon.com with any additional questions or comments.

## Security

If you discover a potential security issue in this project, please notify AWS Security via our [vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public GitHub issue.

## Licensing

See the [LICENSE](LICENSE) file for this project's licensing.
