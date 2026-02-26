# oz-slack-analyzer

Prototype repo for testing Oz cloud agents. Contains a skill that analyzes Slack workspace
activity data per tenant and produces an anomaly detection report.

## Structure

- `.agents/skills/slack-activity-analyzer/` - Oz skill definition
- `data/slack_activity.csv` - Sample Slack activity data
- `reports/` - Output directory for analysis reports (created by the agent)

## Sample tenants

- **T-1001** - Clean activity, no anomalies expected
- **T-1002** - Multiple high-severity anomalies (bulk downloads at 3 AM, mass channel deletions, privilege escalation, anomalous IP/user-agent)
- **T-1003** - Subtle anomalies (app installation spike, privilege escalation, minor off-hours activity)

## Architecture

```
+------------------+       +---------------------+       +----------------------+
|  Oz Web App /    |       |   Oz Cloud Agent    |       |   GitHub Repo        |
|  CLI             |       |                     |       |   (this repo)        |
+------------------+       +---------------------+       +----------------------+
|                  |       |                     |       |                      |
| 1. User kicks    +------>| 2. Agent spins up   +------>| 3. Checks out repo   |
|    off a run     |       |    in environment   |       |    into workspace    |
|    with prompt:  |       |                     |       |                      |
|    "Analyze for  |       +----------+----------+       +----------------------+
|     tenant_id=   |                  |
|     T-1002"      |                  v
|                  |       +---------------------+
+------------------+       | 4. Loads skill from |
                           |    .agents/skills/  |
                           +----------+----------+
                                      |
                                      v
                           +---------------------+
                           | 5. Reads CSV data,  |
                           |    filters by       |
                           |    tenant_id        |
                           +----------+----------+
                                      |
                                      v
                           +---------------------+
                           | 6. Applies anomaly  |
                           |    criteria from    |
                           |    references/      |
                           +----------+----------+
                                      |
                                      v
                           +---------------------+       +----------------------+
                           | 7. Writes report to +------>| 8. Opens PR with     |
                           |    reports/<id>-    |       |    analysis report   |
                           |    analysis.md      |       |                      |
                           +---------------------+       +----------------------+
```

## Setup

1. Create an Oz environment pointing to this repo:

```sh
oz environment create \
  --name "slack-analyzer" \
  --repo "aartij17/oz-slack-analyzer" \
  --output-format text
```

2. Note the environment ID from the output (e.g., `UA17BXYZ`).

## Usage

Run the agent with a tenant-specific prompt:

```sh
oz agent run-cloud \
  --environment <ENV_ID> \
  --prompt "Analyze Slack activity for tenant_id=T-1002"
```

Monitor the run:

```sh
oz run get <RUN_ID>
```

The agent will read the skill, apply detection criteria from `references/anomaly-criteria.md`,
and write a report to `reports/<tenant_id>-analysis.md`, then open a PR.
