# Harbour Language — AI Training Dataset

Machine-readable **Harbour function definitions** in YAML for AI-assisted code generation and documentation.

## Revision Date

2025-10-12-001

## What’s in this repo

- **Dataset**
  - Manifest: [`harbour_functions/index.yaml`](./harbour_functions/index.yaml)
  - Parts: [`harbour_functions/parts/`](./harbour_functions/parts/)
- **Schema spec**  
  See [`SPEC_harbour_functions.md`](./SPEC_harbour_functions.md)
- **License**  
  Creative Commons **BY-NC-SA 4.0** — non-commercial, share-alike (see [`LICENSE`](./LICENSE))

## How tools should use this

1. Load `harbour_functions/index.yaml`.
2. Resolve a function by `slug` via `map` → open the corresponding part file.
3. Read the function object (parameters, types, Markdown description, examples, etc.) and generate code/documentation.
4. Use `see_also` for related APIs; observe `platforms` for OS specifics.

## Folder layout

```
harbour_functions/
  index.yaml            # Option A manifest: parts + slug → file map
  parts/
    harbour_functions_part_001.yaml
    harbour_functions_part_002.yaml
    ...
SPEC_harbour_functions.md
LICENSE
```

## Contributing

Schema may evolve. Please open an issue with suggestions or corrections. PRs welcome once the first full dataset is stable.

---
© 2025 Eric Lendvai — Licensed under CC BY-NC-SA 4.0
