# 03-helm-templating-and-functions: Exercises

## Exercise 1: Master Template Syntax Basics
Learn Go templating and Helm-specific syntax.

**Task:**
- Create simple template with variable substitution: `{{ .Values.appName }}`
- Use nested values: `{{ .Values.image.repository }}`
- Access chart metadata: `{{ .Chart.Name }}`, `{{ .Release.Name }}`
- Test with `helm template` command
- Display rendered output

## Exercise 2: Implement Built-in Functions
Use Helm's collection of functions for value transformation.

**Task:**
- Create template using functions:
  - `quote`: Add quotes to strings
  - `upper`, `lower`: Case conversion
  - `default`: Provide fallback values
  - `trunc`: Truncate strings
  - `nindent`: Manage indentation
- Apply each function with examples
- Show original and transformed values

## Exercise 3: Create Conditional Templates
Implement if/else logic for optional features.

**Task:**
- Create template with `{{- if .Values.feature }}` block
- Add else clause: `{{- else }}`
- Use logical operators: `not`, `eq`, `ne`
- Implement nested conditions
- Show conditional rendering with `helm template`
- Deploy with feature enabled and disabled

## Exercise 4: Design Loops and Iterations
Use range to iterate over lists and maps.

**Task:**
- Loop over array: `{{- range .Values.items }}`
- Loop over map with key-value: `{{- range $k, $v := .Values.config }}`
- Access loop index: `{{- range $i, $item := .Values.list }}`
- Access parent scope in loop with `$`
- Create ConfigMap with looped config values
- Show rendered output with multiple iterations

## Exercise 5: Implement Named Templates
Create and reuse template blocks.

**Task:**
- Define named template: `{{- define "custom.resource" }}`
- Call with `include`: `{{ include "custom.resource" . }}`
- Call with `template`: `{{ template "custom.resource" . }}`
- Pass context and scope
- Create 3 named templates and use in different manifests
- Show reusability benefits

## Exercise 6: Handle Complex Data Structures
Transform nested objects and YAML output.

**Task:**
- Use `toYaml` filter for nested data
- Apply `fromYaml` for parsing
- Transform lists to JSON with `toJson`
- Use `keys` function to extract map keys
- Create template handling labels, annotations, and environment variables
- Show hierarchical data preservation

## Exercise 7: String and List Manipulation
Apply string and list functions in templates.

**Task:**
- String functions: `substr`, `contains`, `hasPrefix`, `hasSuffix`, `split`, `join`
- List functions: `first`, `last`, `index`, `append`, `prepend`
- Create validation template checking app name format
- Build dynamic resource names with string manipulation
- Show practical use cases

## Exercise 8: Use Comparison and Logical Functions
Implement complex conditional logic.

**Task:**
- Comparison: `eq`, `ne`, `lt`, `gt`, `le`, `ge`
- Logical: `and`, `or`, `not`
- Create template with multi-condition logic
- Implement feature flags based on environment
- Build decision tree for configuration
- Show all condition combinations

## Exercise 9: Apply Math and Type Functions
Use calculations and type checking.

**Task:**
- Math functions: `add`, `sub`, `mul`, `div`, `mod`
- Type functions: `typeOf`, `kindOf`, `asFloat`, `asInt`
- Create resource calculation template (CPU, memory)
- Build scaling formula based on replica count
- Handle type conversions for mixed data types
- Show computed values in ConfigMap

## Exercise 10: Validate Complete Templating Implementation
Test all template features together in production chart.

**Task:**
- Create chart with all template types (conditionals, loops, helpers, functions)
- Combine multiple features in single template
- Test with varied input values
- Validate with `helm lint` and `helm template`
- Install and verify rendered manifests
- Check pod logs and resource configuration
- Uninstall and cleanup thoroughly
