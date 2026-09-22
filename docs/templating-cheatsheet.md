# Helm Templating Cheatsheet

Helm templates = **Go templates** + the **Sprig** function library + a few
Helm-specific objects and functions. Everything between `{{ }}` is evaluated at
render time.

---

## Built-in objects

| Object | Description |
|--------|-------------|
| `.Values` | Values from `values.yaml`, `-f`, and `--set` |
| `.Release.Name` | Name of the release |
| `.Release.Namespace` | Namespace the release targets |
| `.Release.IsInstall` | `true` on first install |
| `.Release.IsUpgrade` | `true` during an upgrade |
| `.Release.Revision` | Revision number |
| `.Chart` | Contents of `Chart.yaml` (`.Chart.Name`, `.Chart.Version`, `.Chart.AppVersion`) |
| `.Files` | Access non-template files (`.Files.Get`, `.Files.Glob`) |
| `.Capabilities` | Cluster info (`.Capabilities.KubeVersion`, `.Capabilities.APIVersions.Has`) |
| `.Template` | `.Template.Name`, `.Template.BasePath` of the current template |

---

## Whitespace control

`{{-` trims whitespace before, `-}}` trims after. This prevents blank lines in
rendered YAML.
```yaml
{{- if .Values.enabled }}
enabled: true
{{- end }}
```

---

## Conditionals

```yaml
{{- if .Values.ingress.enabled }}
# ingress block
{{- else if .Values.service.external }}
# service block
{{- else }}
# default
{{- end }}
```

Truthiness: `false`, `0`, `""`, empty list/map, and `nil` are all "false".

---

## Loops with `range`

```yaml
env:
{{- range .Values.env }}
  - name: {{ .name }}
    value: {{ .value | quote }}
{{- end }}

# range with index/key
{{- range $key, $val := .Values.labels }}
  {{ $key }}: {{ $val | quote }}
{{- end }}
```

---

## `with` (scope narrowing)

```yaml
{{- with .Values.resources }}
resources:
  limits:
    cpu: {{ .limits.cpu }}
    memory: {{ .limits.memory }}
{{- end }}
```
Inside `with`, `.` refers to `.Values.resources`. Use `$` to reach the root scope.

---

## Variables

```yaml
{{- $fullName := printf "%s-%s" .Release.Name .Chart.Name }}
name: {{ $fullName }}
```

---

## Named templates (`_helpers.tpl`)

Define reusable snippets:
```yaml
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
{{- end -}}
```
Use them:
```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

> Prefer `include` over `template` because `include` can be piped (e.g. into
> `nindent`), while `template` cannot.

---

## Most useful Sprig / Helm functions

| Function | Example | Result |
|----------|---------|--------|
| `default` | `{{ .Values.tag | default "latest" }}` | fallback value |
| `quote` | `{{ .Values.name | quote }}` | wraps in `"` |
| `upper` / `lower` | `{{ "Hi" | upper }}` | `HI` |
| `trim` | `{{ " x " | trim }}` | `x` |
| `nindent` | `{{ include "x" . | nindent 4 }}` | newline + indent |
| `indent` | `{{ .Values.block | indent 2 }}` | indent lines |
| `toYaml` | `{{ .Values.resources | toYaml | nindent 2 }}` | object to YAML |
| `printf` | `{{ printf "%s-db" .Release.Name }}` | formatted string |
| `required` | `{{ required "tag is required!" .Values.tag }}` | fail if empty |
| `tpl` | `{{ tpl .Values.template . }}` | render a string as a template |
| `b64enc` | `{{ "secret" | b64enc }}` | base64 (for Secrets) |
| `sha256sum` | `{{ .Values | toYaml | sha256sum }}` | checksum (roll pods on config change) |

---

## Real-world snippet: roll pods when a ConfigMap changes

```yaml
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

---

## Validating input

```yaml
image: "{{ required "image.repository is required" .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"

{{- if not (has .Values.service.type (list "ClusterIP" "NodePort" "LoadBalancer")) }}
{{- fail "service.type must be ClusterIP, NodePort, or LoadBalancer" }}
{{- end }}
```
