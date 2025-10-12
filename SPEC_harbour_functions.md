# Harbour Functions — YAML Specification

This spec describes the **Harbour function dataset** in `harbour_functions/`.  
Data is split into size-balanced YAML parts with a lean manifest.

---

## File layout

```
harbour_functions/
  index.yaml            # Manifest (Option A): parts list + slug → part map
  parts/
    harbour_functions_part_001.yaml
    harbour_functions_part_002.yaml
    ...
```

### `harbour_functions/index.yaml` (Option A manifest)

```yaml
version: 1
generated_at: "2025-10-12T00:00:00Z"
parts:
  - file: harbour_functions_part_001.yaml
    size_bytes: 815234          # optional
    sha256: 6b6f...             # optional
  - file: harbour_functions_part_002.yaml
    size_bytes: 812901
map:
  ains: harbour_functions_part_001.yaml
  hb_ains: harbour_functions_part_001.yaml
  afill: harbour_functions_part_002.yaml
  # slug → part file (authoritative)
```

- `map` is the **authoritative** slug→file index.
- `parts[].size_bytes` / `sha256` are optional integrity hints.

### `harbour_functions/parts/harbour_functions_part_###.yaml`

Each part has a header and an array of functions:

```yaml
dataset_version: 1              # int ≥ 1
part: 1                          # this part index (≥ 1)
total_parts: 37                  # total number of parts (≥ part)
count_functions: 226             # must equal len(functions)
functions:                       # array of function objects (see below)
  - { ... }
```

---

## Function object schema

Each item in `functions` describes one Harbour function.

### Required
- **`name`** (string): Canonical function name, author’s chosen casing (e.g., `AIns`).
- **`slug`** (string): Lowercase stable ID, `^[a-z0-9_]+$`, globally unique.
- **`categories`** (array of string): One or more topical categories (author-defined).
- **`return_type`** (string): Harbour return type (`Array`, `Numeric`, `String`, `Logical`, `Date`, `Hash`, `Any`, …).

### Common optional fields
- **`description`** (string): Markdown description (preserve formatting).
- **`parameters`** (array of objects):
  - `name` (string)
  - `type` (string)
  - `required` (boolean)
  - `default` (scalar or null)
  - `description` (string)
  - `passed_by` (string; e.g., `value`, `reference`, `macro`, `block`)
- **`return_name`** (string): Name used in docs for the return value.
- **`return_description`** (string): Markdown describing the return value semantics.
- **`library`** (string or null): Library/module if applicable; omit or set null if N/A.
- **`source_code_file`** (string): Source file path related to the function.
- **`platforms`** (array of string): Target OS list, e.g., `['Windows', 'Linux', 'Mac']`.
- **`generate_user_interface`** (boolean or string): flag/directive indicating UI scaffolding potential. false if missing.
- **`examples`** (array of strings): Harbour code examples (preserve formatting).
- **`see_also`** (array of string): Related function slugs (must correspond to existing `slug`s).

---

## Formatting conventions

To keep parsing robust and files consistent:

- **Indentation**: 2 spaces per level. One space after `-` in lists.
- **Scalars** (single line): use **single quotes**; escape apostrophes by doubling (`It''s`).
- **Markdown or multi-line**: use **literal blocks**:
  - `description_md`: `|-` (keeps Markdown exactly, drops trailing newline)
  - `examples[]`: `|` or `|-` (keeps code formatting)
- **Slug**: lowercase letters, digits, underscore only.
- **Cross-refs**: `see_also` items are **slugs** (not names) and should resolve via the manifest.

---

## Example function entry

```yaml
- name: 'AIns'
  slug: 'ains'
  categories: ['Arrays']
  library: 'hbvm'
  source_code_file: 'src/vm/arrayshb.c'
  platforms: ['Windows', 'Linux', 'Mac']
  parameters:
    - name: 'aArray'
      type: 'Array'
      required: true
      description: 'Target array (modified in place).'
      passed_by: 'reference'
    - name: 'nPosition'
      type: 'Numeric'
      required: false
      default: 1
      description: '1-based index for insertion.'
      passed_by: 'value'
  return_type: 'Array'
  return_name: 'aArray'
  return_description: 'Same array reference after modification.'
  generate_user_interface: false
  description_md: |-
    Inserts a new element into an array at the given position,
    shifting existing elements to the right and discarding the last.
  examples:
    - |-
      LOCAL a := { 1, 2, 3, 4, 5 }
      AIns( a, 1 )
      a[1] := 100   // { 100, 1, 2, 3, 4 }
  see_also: ['hb_ains', 'aadd', 'asize']
```

---

## Guidance for AI/Tools

1. **Resolve a function**: Load `harbour_functions/index.yaml`, map `slug` → part file, then open that file and locate the entry by `slug`.
2. **Code generation**:
   - Use `parameters[]` (`type`, `required`, `default`, `passed_by`) to form call signatures.
   - Honor `return_type` and `return_description` in completions and validations.
   - Prefer `examples` as canonical usage patterns.
   - Respect `platforms` when generating OS-specific code.
3. **Navigation**: `see_also` slugs provide related APIs to suggest to the user.
4. **UI scaffolding** (optional): If `generate_user_interface` is true or a directive, you may generate basic UI forms from parameter metadata.

---

*This spec covers the current dataset. Fields may evolve; consumers should ignore unknown fields gracefully.*
