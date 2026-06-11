# CVE Fixes with the Ambient Code Platform — Component Lead Guide

This guide walks you through setting up the [Ambient Code Platform](https://ambient-code.apps.rosa.vteam-uat.0ksl.p3.openshiftapps.com) and running the CVE fixer workflow as a component lead.

---

## Overview

The CVE automation workflow automates the full CVE remediation cycle:

1. **Triage** — fetches open CVEs from the Jira CVE tracker and corrects wrong component assignments
2. **Validate** — verifies vulnerabilities are present using `govulncheck`
3. **Fix** — identifies fixed versions, opens pull requests with dependency updates
4. **Update** — comments on and updates the associated Jira tickets

**Relevant links**

| Resource | URL |
|----------|-----|
| Ambient Code Platform | https://ambient-code.apps.rosa.vteam-uat.0ksl.p3.openshiftapps.com |
| Workflow repository | https://github.com/openshift-pipelines/ambient-workflows |
| CVE automation workflow | https://github.com/openshift-pipelines/ambient-workflows/tree/main/cve-fixer |
| Component mappings | https://github.com/openshift-pipelines/ambient-workflows/blob/main/cve-fixer/component-repository-mappings.json |

---

## Prerequisites

Before starting, make sure you have:

- [ ] A Red Hat Jira account with access to the SRVKP project
- [ ] A [Jira API token](https://id.atlassian.com/manage-profile/security/api-tokens)
- [ ] A [GitHub Personal Access Token (PAT)](https://github.com/settings/tokens) with `repo` scope
- [ ] Your component listed in [`component-repository-mappings.json`](../cve-fixer/component-repository-mappings.json)


---

## Step 1 — Create a Workspace

1. Log in to the [Ambient Code Platform](https://ambient-code.apps.rosa.vteam-uat.0ksl.p3.openshiftapps.com).
2. Click **New Workspace** and give it a name that identifies your component (e.g. `tektoncd-operator`).

---

## Step 2 — Configure Workspace Environment Variables

Navigate to **Workspace Settings → Custom Environment Variables** and add the following:

| Variable | Value | Notes |
|----------|-------|-------|
| `GIT_AUTHOR_NAME` | Your display name | Used for git commits |
| `JIRA_EMAIL` | `your-email@redhat.com` | Jira account email |
| `JIRA_API_TOKEN` | Your Jira API token | Required for Jira access; generate at [Atlassian](https://id.atlassian.com/manage-profile/security/api-tokens) |

> **Note:** Jira integration may occasionally be unstable. The tokens are also used as a direct fallback so they must be set even if SSO is available.

---

## Step 3 — Add Your GitHub Integration

1. Click the **Integrations** icon in the top-right corner of the platform.
2. Select **GitHub** and enter your Personal Access Token (PAT).
3. Verify the integration shows a green connected status.

The PAT needs at minimum **`repo`** scope so the workflow can clone repositories and open pull requests on your behalf.

---

## Step 4 — Load the CVE Fixer Workflow

1. Open your workspace and go to **Session → Chat box → Load Custom**.
2. Fill in the workflow source:

   | Field | Value |
   |-------|-------|
   | Workflow repo | `https://github.com/openshift-pipelines/ambient-workflows` |
   | Branch | `main` |
   | Path | `cve-fixer` |

3. Click **Load** — the agent will pull the workflow instructions and register all slash commands.

---

## Step 5 — Run the CVE Workflow

### Option A — Fix by component name (recommended)

Run the following prompt in the chat box:

```
/cve.fix <COMPONENT_NAME> --max 4
```

**Example:**

```
/cve.fix Operator --max 4
```

The `--max` flag limits the number of CVEs processed in a single run (recommended: `4` to keep sessions manageable).

---


## What the Workflow Does

When you run `/cve.fix`, the agent will automatically:

1. **Query Jira** — fetch open CVE tickets assigned to your component from the SRVKP project
2. **Validate** — run `govulncheck` against your repositories to confirm each CVE is present
3. **Identify fixes** — read `govulncheck` output to find the minimum version that resolves each CVE
4. **Open pull requests** — create one PR per CVE against the correct branch (upstream and/or downstream)
5. **Update Jira** — comment on the ticket with the PR link and status

PRs are created with:
- Conventional commit messages
- A description including CVE ID, affected package, fixed version, and breaking-change analysis
- Test results (the PR is created even if tests fail, so you can review the fix manually)

---

## Artifacts

All outputs are saved inside the workspace under `artifacts/`:

```
artifacts/
├── cve-triage/
│   ├── fetch/       # raw triage data (triage-<timestamp>.json)
│   └── report/      # component-grouped report (triage-<timestamp>.md)
└── cve-fixer/
    ├── find/        # list of Jira CVE issues (cve-issues-<timestamp>.md)
    └── fixes/       # per-CVE fix logs, PR summaries, skip reasons
```

---


## Component Mapping Reference

The workflow uses [`component-repository-mappings.json`](../cve-fixer/component-repository-mappings.json) to resolve Jira component names to GitHub repositories and branches.

If your component is missing, add an entry like this and open a PR:

```json
{
  "Your Component Name": {
    "container_to_repo_mapping": {
      "openshift-pipelines/your-container-rhel9": "org/your-repo"
    },
    "repositories": {
      "org/your-upstream-repo": {
        "github_url": "https://github.com/org/your-upstream-repo",
        "default_branch": "main",
        "active_release_branches": ["release-1.0"],
        "branch_strategy": "Fix in main. Release branches follow pattern release-X.Y.",
        "repo_type": "upstream"
      },
      "org/your-downstream-repo": {
        "github_url": "https://github.com/org/your-downstream-repo",
        "default_branch": "main",
        "active_release_branches": ["rhoai-3.4"],
        "branch_strategy": "Fork of upstream. Release branches follow pattern rhoai-X.Y.",
        "repo_type": "downstream"
      }
    }
  }
}
```

See [`FIELD_REFERENCE.md`](../cve-fixer/FIELD_REFERENCE.md) for the full schema.
