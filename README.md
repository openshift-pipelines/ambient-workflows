# OpenShift Pipelines — Ambient Workflows

Custom [Ambient Code](https://ambient-code.github.io/platform/) workflows for the **OpenShift Pipelines** team.

These workflows are designed to run inside [Ambient Code](https://ambient-code.github.io/platform/) AI agents and automate
recurring engineering tasks — CVE triage, security fixes, and more.
Each workflow lives in its own directory and can be registered as an independent Ambient project.

> **Platform docs**: [https://ambient-code.github.io/platform/concepts/workflows/](https://ambient-code.github.io/platform/concepts/workflows/)

---

## Available Workflows

| Workflow | Directory | Purpose |
|----------|-----------|---------|
| CVE Fixer | [`cve-fixer/`](./cve-fixer/) | Triage ProdSec CVE tickets in Jira, correct component assignments, and create pull requests with dependency fixes |

---

## How to Use

1. Go to your [Ambient Code project dashboard](https://ambient-code.apps.rosa.vteam-uat.0ksl.p3.openshiftapps.com).
2. Create a new project and point it at this repository (`openshift-pipelines/ambient-workflows`).
3. Set the **workflow path** to the directory of the workflow you want (e.g. `cve-fixer`).
4. Add the required environment secrets for that workflow (see the workflow's own `README.md`).
5. Start a session — the agent will load the workflow instructions automatically.

---

## Contributing

To add a new workflow, create a new directory at the root of this repo with the following structure:

```
my-workflow/
├── .ambient/
│   └── ambient.json          # workflow name, description, system prompt
├── .claude/
│   └── commands/
│       └── my-command.md     # slash commands available in the agent
└── README.md                 # workflow documentation
```

See the [Ambient platform workflow docs](https://ambient-code.github.io/platform/concepts/workflows/) for the full schema reference.
