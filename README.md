# The Complete Helm Guide (for Beginners)

[![Helm CI](https://github.com/dindo-manondo/helm-guide/actions/workflows/helm-ci.yml/badge.svg)](https://github.com/dindo-manondo/helm-guide/actions/workflows/helm-ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A hands-on, detailed guide to **Helm** — the package manager for Kubernetes. This
repo is written for someone who is still learning, so it explains the *why* behind
each concept, not just the commands.

> **Helm version targeted:** v3 / v4 (the commands are the same for both; where v4
> differs, it is called out). Check yours with `helm version`.

---

## Table of Contents

1. [What is Helm and why use it?](#1-what-is-helm-and-why-use-it)
2. [Core concepts and vocabulary](#2-core-concepts-and-vocabulary)
3. [Installing Helm](#3-installing-helm)
4. [Your first deployment (quick start)](#4-your-first-deployment-quick-start)
5. [Working with repositories](#5-working-with-repositories)
6. [Anatomy of a Helm chart](#6-anatomy-of-a-helm-chart)
7. [Creating your own chart](#7-creating-your-own-chart-step-by-step)
8. [Templating deep dive](#8-templating-deep-dive)
9. [Values, overrides, and precedence](#9-values-overrides-and-precedence)
10. [Releases, upgrades, rollbacks](#10-releases-upgrades-and-rollbacks)
11. [The complete command reference](#11-the-complete-command-reference)
12. [Dependencies and subcharts](#12-dependencies-and-subcharts)
13. [Packaging and sharing charts](#13-packaging-and-sharing-charts)
14. [Hooks and chart tests](#14-hooks-and-chart-tests)
15. [Best practices](#15-best-practices)
16. [Troubleshooting](#16-troubleshooting)
17. [Screenshots](#17-screenshots)

Supporting files in this repo:

- [`docs/command-reference.md`](docs/command-reference.md) — every Helm command with flags and examples
- [`docs/templating-cheatsheet.md`](docs/templating-cheatsheet.md) — Go template + Sprig functions
- [`docs/troubleshooting.md`](docs/troubleshooting.md) — common errors and fixes
- [`docs/walkthrough.md`](docs/walkthrough.md) — hands-on 9-step walkthrough with screenshots
- [`docs/screenshot-guide.md`](docs/screenshot-guide.md) — how to re-capture the screenshots
- [`examples/mychart/`](examples/mychart/) — a complete, working example chart you can install
- [`.github/workflows/helm-ci.yml`](.github/workflows/helm-ci.yml) — CI that lints and renders the chart on every push

---

## 1. What is Helm and why use it?

Kubernetes applications are described by YAML manifests: Deployments, Services,
ConfigMaps, Ingresses, and so on. A real application can easily need a dozen of
these files, and you often need slightly different versions for dev, staging, and
production. Managing all of that by hand is error-prone.

**Helm** solves this. Think of it as `apt`, `yum`, or `npm`, but for Kubernetes.

- A **chart** is a package of pre-configured Kubernetes resources.
- Helm lets you **template** those resources so one chart works across many
  environments by swapping in different values.
- Helm tracks each installation as a **release**, so you can upgrade and roll
  back cleanly.

```
Without Helm                          With Helm
-----------                           ---------
deployment-dev.yaml                   mychart/
deployment-staging.yaml       -->       templates/deployment.yaml   (one template)
deployment-prod.yaml                    values-dev.yaml
service-dev.yaml                        values-staging.yaml
service-staging.yaml                    values-prod.yaml
... (kubectl apply each one)          helm install ... -f values-prod.yaml
```

---

## 2. Core concepts and vocabulary

| Term | Meaning |
|------|---------|
| **Chart** | A Helm package. A directory (or `.tgz`) containing templates + metadata. |
| **Release** | A running instance of a chart in a cluster. Install the same chart 3 times = 3 releases. |
| **Repository** | An HTTP server (or OCI registry) that hosts packaged charts. |
| **Values** | Configuration passed to a chart to customize it (`values.yaml` or `--set`). |
| **Template** | A Kubernetes manifest with Go templating placeholders. |
| **Revision** | A numbered version of a release. Every upgrade creates a new revision. |
| **Manifest** | The final, rendered YAML that Helm sends to Kubernetes. |
| **Subchart** | A chart that another chart depends on. |

---

## 3. Installing Helm

**Windows (winget):**
```powershell
winget install Helm.Helm
```

**Windows (Chocolatey):**
```powershell
choco install kubernetes-helm
```

**macOS (Homebrew):**
```bash
brew install helm
```

**Linux (script):**
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify:
```bash
helm version
```
![helm version output](screenshots/01-helm-version.png)

> Helm talks to your cluster using the same `kubeconfig` that `kubectl` uses. If
> `kubectl get nodes` works, Helm will work too.

---

## 4. Your first deployment (quick start)

Install the Bitnami nginx chart and see Helm in action:

```bash
# 1. Add a public chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# 2. Refresh your local index of available charts
helm repo update

# 3. Install the chart as a release named "my-web"
helm install my-web bitnami/nginx

# 4. See what you deployed
helm list

# 5. Check the release status
helm status my-web

# 6. Clean up
helm uninstall my-web
```

Example `helm list` and `helm status` output:

![helm list and status](screenshots/04-list-status.png)

> **Heads-up on Bitnami:** since August 2025 only a limited set of Bitnami images
> and charts are free, and the install output shows a warning about it. It's fine
> for learning. For real workloads see
> [Bitnami catalog changes](docs/troubleshooting.md#bitnami-catalog-changes-august-2025).

---

## 5. Working with repositories

```bash
helm repo add <name> <url>     # register a repo
helm repo list                 # list registered repos
helm repo update               # refresh cached index files
helm repo remove <name>        # unregister a repo
helm search repo <keyword>     # search repos you've added
helm search hub <keyword>      # search the public Artifact Hub
```

Inspect a chart before installing it:
```bash
helm show chart bitnami/nginx       # Chart.yaml metadata
helm show values bitnami/nginx      # all configurable values
helm show readme bitnami/nginx      # the chart's README
helm show all bitnami/nginx         # everything
```

---

## 6. Anatomy of a Helm chart

When you run `helm create mychart`, Helm scaffolds this structure:

```
mychart/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml         # Default configuration values
├── charts/             # Subcharts (dependencies) live here
├── templates/          # The templated Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── NOTES.txt       # Message shown after install
│   ├── _helpers.tpl    # Reusable template snippets
│   └── tests/
│       └── test-connection.yaml
└── .helmignore         # Files to exclude when packaging
```

**`Chart.yaml`** — the identity of the chart:
```yaml
apiVersion: v2            # v2 = Helm 3/4 charts
name: mychart
description: A Helm chart for Kubernetes
type: application         # or "library"
version: 0.1.0            # the CHART version (bump on any chart change)
appVersion: "1.16.0"      # the version of the APP being deployed
```

> **Key distinction:** `version` is the chart's own version. `appVersion` is the
> version of the software the chart installs. They change independently.

---

## 7. Creating your own chart (step by step)

```bash
# Scaffold a new chart
helm create mychart

# Render the templates locally WITHOUT installing (great for learning)
helm template mychart

# Validate the chart for issues
helm lint mychart

# Do a dry-run install to see exactly what would be sent to the cluster
helm install demo ./mychart --dry-run --debug

# Actually install it
helm install demo ./mychart

# Change a value on the fly
helm install demo ./mychart --set replicaCount=3
```

See [`examples/mychart/`](examples/mychart/) for a complete working chart with
comments explaining every line.

---

## 8. Templating deep dive

Helm templates use **Go templating** plus the **Sprig** function library. The
templating engine replaces `{{ ... }}` placeholders with real values at render time.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```

**Built-in objects you'll use constantly:**

| Object | What it holds |
|--------|---------------|
| `.Values` | Everything from `values.yaml` and `--set` |
| `.Release.Name` | The release name you chose |
| `.Release.Namespace` | Target namespace |
| `.Chart.Name` / `.Chart.Version` | From `Chart.yaml` |
| `.Files` | Access to non-template files in the chart |
| `.Capabilities` | Cluster/K8s version info |

See [`docs/templating-cheatsheet.md`](docs/templating-cheatsheet.md) for pipelines,
conditionals, loops, `with`, `range`, named templates, and the most useful Sprig
functions.

---

## 9. Values, overrides, and precedence

Values can come from several places. Later sources **override** earlier ones:

```
chart's values.yaml   (lowest priority)
        ↓
parent chart values / -f myvalues.yaml   (in the order given)
        ↓
--set / --set-string / --set-file        (highest priority)
```

```bash
# Use a custom values file
helm install demo ./mychart -f values-prod.yaml

# Combine multiple files (right-most wins)
helm install demo ./mychart -f base.yaml -f prod.yaml

# Override single values inline
helm install demo ./mychart --set image.tag=1.2.3 --set replicaCount=5

# Nested and list values
helm install demo ./mychart --set service.type=NodePort --set ingress.hosts[0]=example.com
```

---

## 10. Releases, upgrades, and rollbacks

```bash
# Upgrade an existing release (or install if missing)
helm upgrade --install demo ./mychart -f values-prod.yaml

# See the revision history
helm history demo

# Roll back to a previous revision
helm rollback demo 1

# Uninstall but keep history so you can roll back
helm uninstall demo --keep-history
```

Upgrade, history, and rollback in action (a rollback adds a new revision
rather than deleting history):

![upgrade and rollback](screenshots/07-upgrade-rollback.png)

---

## 11. The complete command reference

The full reference with every flag and example lives in
[`docs/command-reference.md`](docs/command-reference.md). Quick index:

| Command | Purpose |
|---------|---------|
| `helm create` | Scaffold a new chart |
| `helm install` | Install a chart as a release |
| `helm upgrade` | Upgrade a release |
| `helm rollback` | Revert to a previous revision |
| `helm uninstall` | Remove a release |
| `helm list` | List releases |
| `helm status` | Show release status |
| `helm history` | Show revision history |
| `helm get` | Get release details (values, manifest, notes, hooks) |
| `helm template` | Render templates locally |
| `helm lint` | Check a chart for issues |
| `helm package` | Package a chart into a `.tgz` |
| `helm pull` | Download a chart |
| `helm push` | Push a chart to an OCI registry |
| `helm repo` | Manage repositories |
| `helm search` | Search for charts |
| `helm show` | Inspect chart info |
| `helm dependency` | Manage subcharts |
| `helm test` | Run a chart's tests |
| `helm plugin` | Manage Helm plugins |
| `helm registry` | Log in/out of OCI registries |
| `helm env` | Show Helm's environment info |
| `helm version` | Show the Helm version |

---

## 12. Dependencies and subcharts

Declare dependencies in `Chart.yaml`:
```yaml
dependencies:
  - name: postgresql
    version: "15.5.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

```bash
helm dependency update ./mychart   # download deps into charts/
helm dependency build ./mychart    # rebuild from Chart.lock
helm dependency list ./mychart     # show dependency status
```

---

## 13. Packaging and sharing charts

```bash
# Package into mychart-0.1.0.tgz
helm package ./mychart

# Build a repo index (for an HTTP-hosted repo)
helm repo index . --url https://example.com/charts

# OCI registry workflow (modern approach)
helm registry login registry.example.com
helm push mychart-0.1.0.tgz oci://registry.example.com/charts
helm pull oci://registry.example.com/charts/mychart --version 0.1.0
```

---

## 14. Hooks and chart tests

**Hooks** let you run resources at specific points in a release lifecycle
(e.g. a Job before install). Annotate a resource:
```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
```
Hook events: `pre-install`, `post-install`, `pre-upgrade`, `post-upgrade`,
`pre-delete`, `post-delete`, `pre-rollback`, `post-rollback`, `test`.

**Tests** verify a release works:
```bash
helm test <release-name>
```

---

## 15. Best practices

- **Pin versions.** Always set explicit image tags and chart dependency versions.
- **Use `helm upgrade --install`** in CI so the same command works first time and every time.
- **Lint and dry-run** before applying: `helm lint` then `helm install --dry-run --debug`.
- **Keep secrets out of values.yaml.** Use a secrets manager or `helm-secrets`.
- **Bump `version` in `Chart.yaml`** on every chart change; bump `appVersion` when the app changes.
- **Prefer named templates** (`_helpers.tpl`) to avoid repeating label blocks.
- **Quote strings** that could be misread as numbers/booleans: `tag: "1.0"`.
- **Use `--atomic`** on upgrades so a failed upgrade auto-rolls-back.

---

## 16. Troubleshooting

See [`docs/troubleshooting.md`](docs/troubleshooting.md) for a full list. Fast checks:

```bash
helm template ./mychart              # does it even render?
helm lint ./mychart                  # obvious mistakes?
helm install x ./mychart --dry-run --debug   # what would be applied?
helm get manifest <release>          # what was actually applied?
helm history <release>               # what changed and when?
```

---

## 17. Screenshots

Every screenshot is real output from Helm v3.16.2 against a local test cluster,
and they're embedded in [`docs/walkthrough.md`](docs/walkthrough.md). Your
timestamps and chart versions will differ.

| Screenshot | Shows |
|------------|-------|
| [`01-helm-version.png`](screenshots/01-helm-version.png) | `helm version` |
| [`02-repo-add.png`](screenshots/02-repo-add.png) | `helm repo add` + `helm repo update` |
| [`03-install.png`](screenshots/03-install.png) | `helm install my-web bitnami/nginx` |
| [`04-list-status.png`](screenshots/04-list-status.png) | `helm list` and `helm status` |
| [`05-create-chart.png`](screenshots/05-create-chart.png) | `helm create mychart` + the tree |
| [`06-template-render.png`](screenshots/06-template-render.png) | `helm template mychart` output |
| [`07-upgrade-rollback.png`](screenshots/07-upgrade-rollback.png) | `helm upgrade`, `helm history`, `helm rollback` |
| [`08-lint-get-values.png`](screenshots/08-lint-get-values.png) | `helm lint --strict` and `helm get values --all` |

To refresh them after a Helm upgrade, follow
[`docs/screenshot-guide.md`](docs/screenshot-guide.md).
