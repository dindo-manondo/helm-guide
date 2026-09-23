# Screenshot Capture Guide

The repo already ships a full set of screenshots. Use this guide when you want
to refresh them, for example after upgrading Helm. Run the commands below, snip
the terminal, and save each image in the
[`../screenshots/`](../screenshots/) folder with the **exact filename** shown so
the Markdown links in [`walkthrough.md`](walkthrough.md) resolve.

---

## How to capture a screenshot on each OS

| OS | Shortcut / tool | Notes |
|----|-----------------|-------|
| **Windows** | `Win + Shift + S` (Snipping Tool) | Drag over the terminal region, then paste into Paint and save as PNG. |
| **Windows (full window)** | `Alt + PrtScn` | Captures the active window to clipboard. |
| **macOS** | `Cmd + Shift + 4` | Crosshair to select a region; saves to Desktop as PNG. |
| **macOS (window)** | `Cmd + Shift + 4` then `Space` | Click a window to capture just it. |
| **Linux (GNOME)** | `Shift + PrtScn` | Region select; saves to Pictures. |

**Tips for clean screenshots**

- Maximize the terminal font (Ctrl + `+`) so text is readable in the image.
- Clear the screen first (`clear` / `cls`) so only the relevant command shows.
- Use a light or high-contrast theme so it reads well in both light and dark
  GitHub modes.
- Crop tightly to the command and its output. No taskbars or other windows.
- Keep image width under ~1200px so it renders nicely inline on GitHub.

---

## The captures

Run each block, then snip the command **and** its output together.

### `screenshots/01-helm-version.png`
```bash
clear
helm version
```
Capture: the single `version.BuildInfo{...}` line.

---

### `screenshots/02-repo-add.png`
```bash
clear
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
Capture: the "has been added" line plus the "Update Complete" line.

---

### `screenshots/03-install.png`
```bash
clear
helm install my-web bitnami/nginx
```
Capture: the `NAME: my-web ... STATUS: deployed` block and the NOTES output.

---

### `screenshots/04-list-status.png`
```bash
clear
helm list
helm status my-web
```
Capture: the `helm list` table and the top of `helm status`.

---

### `screenshots/05-create-chart.png`
```bash
clear
helm create mychart
# Windows PowerShell:
Get-ChildItem -Recurse mychart | Select-Object FullName
# macOS/Linux:
# tree mychart
```
Capture: the created file tree.

---

### `screenshots/06-template-render.png`
```bash
clear
helm template mychart | head -40   # macOS/Linux
# Windows PowerShell:
# helm template mychart | Select-Object -First 40
```
Capture: the first rendered manifest (ServiceAccount/Service/Deployment).

---

### `screenshots/07-upgrade-rollback.png`
Run from the repo root (uses the example chart, so no Docker Hub pulls):
```bash
helm install demo ./examples/mychart   # once, before clearing
clear
helm upgrade demo ./examples/mychart --set replicaCount=3
helm history demo
helm rollback demo 1
helm history demo
```
Capture: the upgrade confirmation, the history table showing multiple revisions,
and the rollback confirmation.

---

### `screenshots/08-lint-get-values.png`
```bash
clear
helm lint ./examples/mychart --strict
helm get values demo --all
```
Capture: the lint summary and the top of the computed values.

---

## After capturing

1. Confirm all eight PNGs are in `screenshots/` with the exact names above.
2. Open [`walkthrough.md`](walkthrough.md) locally or on GitHub. The images
   should now render inline instead of showing as broken links.
3. Commit them:
   ```bash
   git add screenshots/*.png
   git commit -m "Refresh walkthrough screenshots"
   git push
   ```
