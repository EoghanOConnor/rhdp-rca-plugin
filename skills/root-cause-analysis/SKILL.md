---
name: root-cause-analysis
description: Perform root cause analysis and log analysis for failed jobs with AAP, using Splunk and AgnosticD/V configurations.
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__github__search_code
  - mcp__github__get_file_contents
---

# Root Cause Analysis

Correlate AAP job logs with Splunk OCP pod logs and analyze AgnosticD/V configuration for root cause identification.

## Automatic Execution

Start automatically when analyzing failed jobs. Use the skill's base path for relative script execution.

### Preflight Check

```bash
# Setup environment if necessary
python3 -m venv .venv && .venv/bin/pip install -q -r requirements.txt
# Check prerequisites
.venv/bin/python scripts/cli.py setup --json
```

Review JSON output to configure:

**Required**:
- **JOB_LOGS_DIR**: Local job log directory
- **JUMPBOX_URI**: SSH jumpbox for analysis result upload

**Recommended**:
- **MLFlow**: Config for recording analysis

**Optional**:
- **SSH / REMOTE_HOST**: Needed for `--fetch`
- **Splunk**: Needed for log correlation
- **GitHub token**: Needed for config fetching

#### Interactive Setup for Missing Configs

For missing configurations:
1. List missing items.
2. Offer help to configure.
3. For `"configurable": true`, guide through required inputs.
4. For SSH, check/create settings in `~/.ssh/config`.
5. Update `.claude/settings.json` with inputs.
6. Ensure session restarts for changes to apply.
7. Secure `.claude/settings.json` in `.gitignore`.

Report non-configurable errors with fix commands.

#### MLFlow Server Startup

Handles connectivity automatically:
- Establishes SSH tunnel if server unreachable and `JUMPBOX_URI` configured.

If checks (like `JOB_LOGS_DIR`, `JUMPBOX_URI`) are unmet, halt analysis with notification. Proceed if critical requirements are fulfilled.

### Analysis Execution

```bash
# Run analysis by Job ID
.venv/bin/python scripts/cli.py analyze --job-id <JOB_ID> --fetch
# Or by manual log path
.venv/bin/python scripts/cli.py analyze --job-log <path-to-job-log>
```

The `cli.py analyze` performs:
- **Step 1**: Parse logs, extract data
- **Step 2**: Query Splunk logs
- **Step 3**: Correlate events
- **Step 4**: Fetch GitHub configurations

**Outputs**:
- Job context, Splunk logs, correlations, GitHub fetch history.

#### Manual Usage

```bash
# Initial setup
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
# Analyze by Job ID or path
.venv/bin/python scripts/cli.py analyze --job-id 1234567 --fetch
.venv/bin/python scripts/cli.py analyze --job-log /path/to/job_123.json.gz
```

---

For errors in **Step 4**, verify using MCP tools as outlined in the verification guide.

### Summary Generation

Combine data from:
- `step1_job_context.json` - Job details
- `step3_correlation.json` - Correlation analysis