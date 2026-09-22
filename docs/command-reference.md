# Helm Complete Command Reference

Every Helm command, grouped by purpose, with the flags you'll actually use and
copy-paste examples. Run `helm <command> --help` for the exhaustive flag list.

> Global flags available on almost every command:
> `--namespace/-n`, `--kube-context`, `--kubeconfig`, `--debug`, `-o/--output {table|json|yaml}`.

---

## Chart lifecycle

### `helm create`
Scaffold a new chart.
```bash
helm create mychart
helm create mychart --starter mystarter   # use a starter/template chart
```

### `helm install`
Install a chart as a new release.
```bash
helm install <release> <chart>
helm install my-web bitnami/nginx
helm install demo ./mychart -f values-prod.yaml --set replicaCount=3
```
Useful flags:
| Flag | Purpose |
|------|---------|
| `-f, --values` | Supply a values file (repeatable) |
| `--set` | Override a value inline |
| `--set-string` | Force value to be treated as a string |
| `--set-file` | Set a value from a file's contents |
| `-n, --namespace` | Target namespace |
| `--create-namespace` | Create the namespace if missing |
| `--dry-run` | Render + validate without installing |
| `--debug` | Verbose output, shows rendered manifests |
| `--wait` | Wait until resources are ready |
| `--timeout` | How long to wait (e.g. `5m0s`) |
| `--atomic` | Roll back automatically if install fails |
| `--generate-name` | Auto-generate the release name |

### `helm upgrade`
Upgrade an existing release (create it if missing with `--install`).
```bash
helm upgrade demo ./mychart
helm upgrade --install demo ./mychart -f values-prod.yaml --atomic --wait
```
Extra flags: `--install` (install if absent), `--reuse-values` (keep previous
values), `--reset-values` (reset to chart defaults), `--force` (force resource
replacement).

### `helm rollback`
Revert a release to a previous revision.
```bash
helm rollback demo         # roll back one revision
helm rollback demo 2       # roll back to revision 2
helm rollback demo 2 --wait --timeout 5m0s
```

### `helm uninstall`
Remove a release.
```bash
helm uninstall demo
helm uninstall demo --keep-history   # keep history for later rollback
helm uninstall demo --dry-run
```

---

## Inspecting state

### `helm list`
```bash
helm list                 # releases in current namespace
helm list -A              # all namespaces
helm list --all           # include failed/uninstalled
helm list -o json
helm list --filter 'web'  # regex filter on names
```

### `helm status`
```bash
helm status demo
helm status demo --show-resources
helm status demo --revision 2
```

### `helm history`
```bash
helm history demo
helm history demo --max 5
```

### `helm get`
Pull details of a deployed release.
```bash
helm get all demo         # everything
helm get values demo      # user-supplied values
helm get values demo -a   # all computed values
helm get manifest demo    # the rendered YAML actually applied
helm get notes demo       # the NOTES.txt output
helm get hooks demo       # hook resources
```

---

## Authoring & validating

### `helm template`
Render templates locally to stdout (no cluster needed).
```bash
helm template ./mychart
helm template demo ./mychart -f values-prod.yaml
helm template ./mychart --show-only templates/deployment.yaml
```

### `helm lint`
```bash
helm lint ./mychart
helm lint ./mychart --strict         # warnings become errors
helm lint ./mychart -f values-prod.yaml
```

### `helm package`
```bash
helm package ./mychart
helm package ./mychart --version 1.2.3 --app-version 2.0.0
helm package ./mychart --sign --key 'me' --keyring ~/.gnupg/secring.gpg
```

### `helm show`
```bash
helm show chart bitnami/nginx
helm show values bitnami/nginx
helm show readme bitnami/nginx
helm show crds bitnami/nginx
helm show all bitnami/nginx
```

---

## Repositories & search

### `helm repo`
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update
helm repo update bitnami          # refresh one repo
helm repo remove bitnami
helm repo index ./charts --url https://example.com/charts
```

### `helm search`
```bash
helm search repo nginx            # search added repos
helm search repo nginx --versions # list all versions
helm search hub wordpress         # search Artifact Hub
```

### `helm pull`
```bash
helm pull bitnami/nginx
helm pull bitnami/nginx --version 18.2.1 --untar
helm pull oci://registry.example.com/charts/mychart --version 0.1.0
```

---

## Dependencies

### `helm dependency`
```bash
helm dependency list ./mychart
helm dependency update ./mychart    # fetch deps into charts/, write Chart.lock
helm dependency build ./mychart     # rebuild from Chart.lock
```

---

## OCI registries

### `helm registry`
```bash
helm registry login registry.example.com -u USER -p PASS
helm registry logout registry.example.com
```

### `helm push`
```bash
helm push mychart-0.1.0.tgz oci://registry.example.com/charts
```

---

## Testing & extending

### `helm test`
```bash
helm test demo
helm test demo --logs      # show test pod logs
```

### `helm plugin`
```bash
helm plugin list
helm plugin install https://github.com/databus23/helm-diff
helm plugin update diff
helm plugin uninstall diff
```

---

## Environment & misc

```bash
helm version                 # client version
helm env                     # show HELM_* environment paths
helm completion powershell   # generate shell completion
helm completion bash
```

---

## Common flag combos worth memorizing

```bash
# Safe production upgrade: install-if-missing, wait, auto-rollback on failure
helm upgrade --install demo ./mychart -f prod.yaml --atomic --wait --timeout 5m0s

# See exactly what will be applied before doing it
helm install demo ./mychart --dry-run --debug

# Deploy into a fresh namespace
helm install demo ./mychart -n demo --create-namespace
```
