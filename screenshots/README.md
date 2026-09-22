# Screenshots

This folder holds the screenshots referenced by the guide. They are **not**
generated automatically. Capture them yourself as you follow
[`../docs/walkthrough.md`](../docs/walkthrough.md) and save them here with these
exact filenames so the Markdown links resolve:

| Filename | Command to capture |
|----------|--------------------|
| `01-helm-version.png` | `helm version` |
| `02-repo-add.png` | `helm repo add bitnami ...` then `helm repo update` |
| `03-install.png` | `helm install my-web bitnami/nginx` |
| `04-list-status.png` | `helm list` and `helm status my-web` |
| `05-create-chart.png` | `helm create mychart` and the folder tree |
| `06-template-render.png` | `helm template mychart` |
| `07-upgrade-rollback.png` | `helm upgrade`, `helm history`, `helm rollback` |

Tip on Windows: use **Win + Shift + S** to snip the terminal, then paste into an
image editor and save as PNG here.

For step-by-step capture instructions (exact commands, cropping tips, and
per-OS shortcuts), see [`../docs/screenshot-guide.md`](../docs/screenshot-guide.md).
