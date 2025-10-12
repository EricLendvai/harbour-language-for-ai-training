# Harbour Language — AI Training Dataset

This repository contains machine-readable **Harbour function definitions** in YAML, intended for AI-assisted code generation and documentation tools.

## What’s here

- **Schema spec**: see [`SPEC_harbour_functions.md`](./SPEC_harbour_functions.md)
- **Dataset**:
  - Manifest: [`harbour_functions/index.yaml`](./harbour_functions/index.yaml)
  - Parts: [`harbour_functions/parts/`](./harbour_functions/parts/)
- **License**: Creative Commons BY-NC-SA 4.0 — non-commercial, share-alike (see [`LICENSE`](./LICENSE)).

## Intended usage

- **AI/RAG**: Load `index.yaml` to resolve a function `slug` → open the correct part → read the function object.
- **Docs/Tools**: Parse the YAML to render documentation, generate stubs, or validate source code.

## Contributing

Schema is evolving; please open an issue if you find errors or have suggestions. PRs welcome once the first full release is frozen.
