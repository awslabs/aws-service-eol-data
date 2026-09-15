# Security Policy

## Reporting a Vulnerability

If you discover a potential security issue in this project, we ask that you notify AWS Security via our [vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public GitHub issue for security vulnerabilities.

## Scope

This repository contains a static JSON dataset with no executable code. Security concerns for this project include:

- **Data integrity:** Incorrect lifecycle dates or statuses that could cause consumers to make flawed operational decisions
- **Source authenticity:** Broken or manipulated `sourceUrl` references that undermine the verification model
- **Supply-chain trust:** Unauthorized modifications to the dataset that bypass the review process

## Security Controls

- All data changes require CODEOWNERS approval before merge
- CI validates JSON schema, date consistency, and sourceUrl patterns on every PR
- Every entry includes a `sourceUrl` pointing to official AWS documentation for independent verification
- The `lastUpdated` field signals dataset freshness

## Consumer Guidance

- For critical business decisions (upgrade deadlines, cost forecasting, compliance), always verify dates against the `sourceUrl` provided in each entry
- Pin to tagged releases rather than tracking the main branch for production integrations
- Treat the `status` field as advisory; validate against date fields programmatically for automated decisions
