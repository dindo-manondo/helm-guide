# Screenshots

Real terminal output used by the guide, captured from Helm v3.16.2 against a
local test cluster. They're embedded in
[`../docs/walkthrough.md`](../docs/walkthrough.md) and the main
[`../README.md`](../README.md).

| Filename | Command |
|----------|---------|
| `01-helm-version.png` | `helm version` |
| `02-repo-add.png` | `helm repo add bitnami ...` then `helm repo update` |
| `03-install.png` | `helm install my-web bitnami/nginx` |
| `04-list-status.png` | `helm list` and `helm status my-web` |
| `05-create-chart.png` | `helm create mychart` and the folder tree |
| `06-template-render.png` | `helm template mychart` |
| `07-upgrade-rollback.png` | `helm upgrade`, `helm history`, `helm rollback` on `./examples/mychart` |
| `08-lint-get-values.png` | `helm lint --strict` and `helm get values --all` |

To re-capture any of them (for example after a Helm upgrade), keep the same
filenames so the links keep working. See
[`../docs/screenshot-guide.md`](../docs/screenshot-guide.md).
