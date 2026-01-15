# 03-helm-templating-and-functions: Cheatsheet

## Template Syntax Basics

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ }}` | Template expression | `{{ .Values.name }}` |
| `{{- }}` | Left whitespace trim | `{{- .Values.name }}` |
| `{{ -}}` | Right whitespace trim | `{{ .Values.name -}}` |
| `{{- - }}` | Both sides trim | `{{- .Values.name -}}` |
| `.` | Current context | `{{ . }}` |
| `$` | Variable/parent context | `{{ $.Release.Name }}` |

## Variable Access

| Pattern | Purpose | Example |
|---------|---------|---------|
| `.Values.xxx` | Access value | `{{ .Values.appName }}` |
| `.Values.x.y.z` | Nested value | `{{ .Values.image.repository }}` |
| `.Chart.Name` | Chart name | `{{ .Chart.Name }}` |
| `.Chart.Version` | Chart version | `{{ .Chart.Version }}` |
| `.Release.Name` | Release name | `{{ .Release.Name }}` |
| `.Release.Namespace` | Namespace | `{{ .Release.Namespace }}` |

## String Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `quote` | Add quotes | `{{ .Values.name \| quote }}` |
| `upper` | Uppercase | `{{ .Values.name \| upper }}` |
| `lower` | Lowercase | `{{ .Values.name \| lower }}` |
| `title` | Title case | `{{ .Values.name \| title }}` |
| `repeat n` | Repeat string | `{{ "x" \| repeat 5 }}` |
| `trunc n` | Truncate | `{{ .Values.name \| trunc 10 }}` |
| `substring` | Extract substring | `{{ substr 0 5 .Values.name }}` |
| `replace old new` | Replace string | `{{ replace " " "-" .Values.name }}` |
| `split sep` | Split string | `{{ .Values.name \| split "-" }}` |
| `join sep` | Join array | `{{ .Values.hosts \| join "," }}` |

## List Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `first` | First element | `{{ first .Values.list }}` |
| `last` | Last element | `{{ last .Values.list }}` |
| `index n` | Element at index | `{{ index .Values.list 0 }}` |
| `append item` | Add to list | `{{ .Values.list \| append "new" }}` |
| `prepend item` | Add to start | `{{ .Values.list \| prepend "start" }}` |
| `reverse` | Reverse list | `{{ .Values.list \| reverse }}` |
| `sort` | Sort list | `{{ .Values.list \| sort }}` |
| `uniq` | Remove duplicates | `{{ .Values.list \| uniq }}` |
| `compact` | Remove nil | `{{ .Values.list \| compact }}` |

## Comparison Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `eq` | Equal | `{{ eq .Values.env "prod" }}` |
| `ne` | Not equal | `{{ ne .Values.env "dev" }}` |
| `lt` | Less than | `{{ lt .Values.count 5 }}` |
| `le` | Less/equal | `{{ le .Values.count 5 }}` |
| `gt` | Greater than | `{{ gt .Values.count 5 }}` |
| `ge` | Greater/equal | `{{ ge .Values.count 5 }}` |

## Logical Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `and` | Logical AND | `{{ and .Values.x .Values.y }}` |
| `or` | Logical OR | `{{ or .Values.x .Values.y }}` |
| `not` | Logical NOT | `{{ not .Values.disabled }}` |

## Data Structure Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `toYaml` | Convert to YAML | `{{ .Values.config \| toYaml }}` |
| `fromYaml` | Parse YAML | `{{ .Values.raw \| fromYaml }}` |
| `toJson` | Convert to JSON | `{{ .Values.data \| toJson }}` |
| `fromJson` | Parse JSON | `{{ .Values.json \| fromJson }}` |
| `keys` | Map keys | `{{ keys .Values.config }}` |
| `values` | Map values | `{{ values .Values.config }}` |
| `has` | Check key exists | `{{ has "key" .Values.map }}` |
| `pick` | Select fields | `{{ pick .Values.obj "field1" "field2" }}` |

## Math Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `add x y` | Addition | `{{ add 5 3 }}` |
| `sub x y` | Subtraction | `{{ sub 10 3 }}` |
| `mul x y` | Multiplication | `{{ mul 4 5 }}` |
| `div x y` | Division | `{{ div 20 4 }}` |
| `mod x y` | Modulo | `{{ mod 10 3 }}` |
| `max` | Maximum | `{{ max .Values.a .Values.b }}` |
| `min` | Minimum | `{{ min .Values.a .Values.b }}` |
| `round` | Round number | `{{ .Values.decimal \| round 2 }}` |

## Type Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `typeOf` | Get type | `{{ typeOf .Values.x }}` |
| `kindOf` | Get kind | `{{ kindOf .Values.x }}` |
| `asInt` | Convert to int | `{{ .Values.str \| asInt }}` |
| `asFloat` | Convert to float | `{{ .Values.str \| asFloat }}` |
| `asString` | Convert to string | `{{ .Values.num \| asString }}` |

## Conditional Syntax

```bash
# Simple if
{{- if .Values.enabled }}
content
{{- end }}

# If-else
{{- if .Values.enabled }}
enabled content
{{- else }}
disabled content
{{- end }}

# If-else if-else
{{- if eq .Values.env "prod" }}
production
{{- else if eq .Values.env "dev" }}
development
{{- else }}
other
{{- end }}

# Multiple conditions
{{- if and .Values.x .Values.y }}
both true
{{- end }}

# Negation
{{- if not .Values.disabled }}
enabled
{{- end }}
```

## Loop Syntax

```bash
# Array loop
{{- range .Values.items }}
{{ . }}
{{- end }}

# Map loop with key-value
{{- range $k, $v := .Values.config }}
{{ $k }}: {{ $v }}
{{- end }}

# Loop with index
{{- range $i, $item := .Values.list }}
{{ $i }}: {{ $item }}
{{- end }}

# Access parent scope in loop
{{- range .Values.items }}
item: {{ . }}
release: {{ $.Release.Name }}
{{- end }}
```

## Named Template Syntax

```bash
# Define
{{- define "custom.name" -}}
content here
{{- end }}

# Call with include (handles whitespace)
{{ include "custom.name" . }}

# Call with template
{{ template "custom.name" . }}

# With indentation
{{- include "custom.name" . | nindent 4 }}
```

## Indentation Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `nindent n` | Indent n spaces | `\| nindent 4` |
| `indent n` | Indent with newline | `\| indent 2` |

## Default & Validation

| Function | Purpose | Example |
|----------|---------|---------|
| `default value` | Fallback value | `{{ .Values.opt \| default "none" }}` |
| `required msg` | Force required | `{{ .Values.key \| required "key needed" }}` |
| `empty` | Check if empty | `{{ if empty .Values.x }}` |

## Sprig Functions

| Category | Functions |
|----------|-----------|
| String | `contains`, `hasPrefix`, `hasSuffix`, `trimPrefix`, `trimSuffix` |
| List | `first`, `last`, `initial`, `rest`, `chunk`, `concat` |
| Math | `add`, `sub`, `mul`, `div`, `mod`, `min`, `max`, `ceil`, `floor` |
| Date | `now`, `date`, `dateInZone`, `ago`, `duration` |

## Debugging

```bash
# Template debug output
helm template . --debug

# Render only one resource
helm template . -s templates/deployment.yaml

# Print to see raw YAML
helm template . | cat -A

# Validate syntax
helm lint .
```
