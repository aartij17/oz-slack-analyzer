---
name: slack-activity-analyzer
description: >
  Analyze Slack workspace activity data for a specific tenant to detect anomalous
  or suspicious behavior patterns. Use when asked to analyze Slack activity, detect
  anomalies in workspace behavior, or investigate suspicious tenant activity.
  Triggers on prompts mentioning Slack analysis, tenant investigation, or activity
  anomaly detection. The tenant_id will be provided in the instance-specific prompt.
---

# Slack Activity Analyzer

Analyze Slack workspace activity for a given tenant and produce a written report
identifying anomalous or suspicious patterns.

## Workflow

1. Read the tenant_id from the prompt.
2. Load `data/slack_activity.csv` from the repository root.
3. Filter all rows matching the provided tenant_id.
4. Read `references/anomaly-criteria.md` for the full list of detection rules.
5. Apply each detection rule against the filtered data.
6. Write the analysis report to `reports/<tenant_id>-analysis.md`.
7. Create a pull request with the report.

## Report format

The report must include:
- Tenant summary (tenant_id, date range of activity, total event count)
- Findings: each anomaly detected, with severity (high / medium / low), supporting evidence (specific rows/timestamps), and a plain-language explanation of why it is anomalous
- If no anomalies are found, state that explicitly
- Recommendations: concrete next steps for each finding

## Constraints

- Do not fabricate data. Only reference rows present in the CSV.
- If the tenant_id is not found in the data, report that clearly and stop.
- Keep the report concise. Prioritize high-severity findings.
