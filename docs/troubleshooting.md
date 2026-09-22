# Helm Troubleshooting Guide

Common Helm problems, what causes them, and how to fix them.

---

## Diagnostic commands (start here)

```bash
helm template ./mychart                       # does it render at all?
helm lint ./mychart                           # obvious chart issues
helm install x ./mychart --dry-run --debug    # what would be applied + why it fails
helm get manifest <release>                   # what was actually applied
helm get values <release> -a                  # all computed values
helm history <release>                        # revision timeline
kubectl get events -n <ns> --sort-by=.lastTimestamp
```

---

## Common errors

### `Error: INSTALLATION FAILED: cannot re-use a name that is still in use`
A release with that name already exists.
```bash
helm list -A                 # find it
helm uninstall <name>        # remove it, or...
helm upgrade --install <name> ./mychart   # upgrade instead
```

### `Error: UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress`
A previous operation got stuck (often "pending-install" / "pending-upgrade").
```bash
helm history <release>
helm rollback <release> <last-good-revision>
# If truly stuck and unrecoverable:
helm uninstall <release>
```

### `Error: template: ...: nil pointer evaluating interface {}...`
A template referenced a value that doesn't exist. Add a default or guard:
```yaml
{{ .Values.image.tag | default .Chart.AppVersion }}
{{- if .Values.ingress }}...{{- end }}
```

### `Error: YAML parse error ... did not find expected key`
Almost always an indentation problem from a template helper. Use `nindent`:
```yaml
labels:
  {{- include "mychart.labels" . | nindent 4 }}
```
Debug by rendering: `helm template ./mychart | less`.

### `Error: unable to build kubernetes objects ... no matches for kind "X" in version "Y"`
The chart uses an API version your cluster doesn't support. Check with:
```bash
kubectl api-resources
helm template ./mychart --api-versions <group>/<version>
```

### `Error: failed to download <chart>`
Repo index is stale or the repo isn't added.
```bash
helm repo add <name> <url>
helm repo update
```

### Values not taking effect
Check precedence: `--set` beats `-f`, which beats `values.yaml`. Confirm what's
actually in use:
```bash
helm get values <release> -a
```

### Release stuck in `pending-upgrade`
```bash
helm rollback <release> <previous-revision>
```

---

## Debugging templates interactively

```bash
# Render only one template file
helm template ./mychart --show-only templates/deployment.yaml

# Print a value while rendering (temporary)
{{ printf "DEBUG value = %v" .Values.someKey }}

# Fail fast with a clear message
{{ required "You must set image.tag" .Values.image.tag }}
```

---

## Cluster-side checks after install

```bash
kubectl get all -l app.kubernetes.io/instance=<release>
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events -n <ns> --sort-by=.lastTimestamp
```
