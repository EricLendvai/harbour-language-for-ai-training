# Harbour Language — AI Training Dataset (and companion references)

This repository provides **resources to help Harbour developers use AI-assisted development effectively and safely**.

It currently contains four core resource groups:

- A **GPT-ready Harbour language rulebook** (living document).
- An example **personal coding standards** document to keep AI-generated Harbour code consistent (adapt it to your needs).
- A machine-readable **Harbour function definitions** dataset in **YAML**, intended for AI-assisted code generation and documentation.
- A **C ↔ Harbour interop guide** (Markdown chapters + a compiled PDF), including an **English translation** (with author approval) of *“The New Buccaneers — Episode 2”* by Manu Expósito Suárez.

---

## What’s in this repo

### 1) GPT-ready guidance documents

- Unified Harbour rulebook (codegen guardrails + semantics): [`harbour_language_rules.md`](./harbour_language_rules.md)
- Example “project defaults” prompts/snippets (how to constrain AI in a specific repo):  
  [`Example_harbour_language_rules_ProjectDefaults.md`](./Example_harbour_language_rules_ProjectDefaults.md)
- Personal cross-project coding standards (formatting, naming, etc.):  
  [`personal_coding_standards.md`](./personal_coding_standards.md)

### 2) Function dataset (machine-readable YAML)

- YAML schema and conventions: [`SPEC_harbour_functions.md`](./SPEC_harbour_functions.md)
- Manifest (slug → part file map): [`harbour_functions/index.yaml`](./harbour_functions/index.yaml)
- Function definitions (current single-part dataset):  
  [`harbour_functions/harbour_functions_part_001.yaml`](./harbour_functions/harbour_functions_part_001.yaml)
- Convenience snapshot (zip of the two files above):  
  [`harbour_functions/harbour_functions_2025_12_25_001.zip`](./harbour_functions/harbour_functions_2025_12_25_001.zip)

### 3) C ↔ Harbour interop guide (book-style)

Folder: [`How_to_Interface_Between_C_and_Harbour/`](./How_to_Interface_Between_C_and_Harbour/)

- Examples bundle: [`How_to_Interface_Between_C_and_Harbour/Examples.zip`](./How_to_Interface_Between_C_and_Harbour/Examples.zip)
- Cover / index: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_00_Cover_and_Index.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_00_Cover_and_Index.md)
- Chapter 01: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_01_Prologue_and_Purpose.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_01_Prologue_and_Purpose.md)
- Chapter 02: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_02_Tools_Needed.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_02_Tools_Needed.md)
- Chapter 03: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_03_A_Bit_of_C.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_03_A_Bit_of_C.md)
- Chapter 04: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_04_Operators.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_04_Operators.md)
- Chapter 05: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_05_Control_Structures.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_05_Control_Structures.md)
- Chapter 06: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_06_Elementary_Data_Types.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_06_Elementary_Data_Types.md)
- Chapter 07: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_07_Structured_Data_Types.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_07_Structured_Data_Types.md)
- Chapter 08: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_08_Structures_Unions_Enums.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_08_Structures_Unions_Enums.md)
- Chapter 09: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_09_Typedef_and_Variables.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_09_Typedef_and_Variables.md)
- Chapter 10: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_10_Pointers.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_10_Pointers.md)
- Chapter 11: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_11_Dynamic_Memory.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_11_Dynamic_Memory.md)
- Chapter 12: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_12_VM_Stack_and_Symbol_Table.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_12_VM_Stack_and_Symbol_Table.md)
- Chapter 13: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_13_C_Functions_for_Harbour.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_13_C_Functions_for_Harbour.md)
- Chapter 14: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_14_Compiling_C_in_PRG.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_14_Compiling_C_in_PRG.md)
- Chapter 15: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_15_Extended_System.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_15_Extended_System.md)
- Chapter 16: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_16_Item_API.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_16_Item_API.md)
- Chapter 17: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_17_Execute_Functions_Methods_CodeBlocks_from_C.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_17_Execute_Functions_Methods_CodeBlocks_from_C.md)
- Chapter 18: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_18_Error_API.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_18_Error_API.md)
- Chapter 19: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_19_FileSys_API_.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_19_FileSys_API_.md)
- Chapter 20: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_20_Creating_Libraries.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_20_Creating_Libraries.md)
- Chapter 21: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_21_DLL_Interfaces.md`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_21_DLL_Interfaces.md)
- Compiled PDF: [`How_to_Interface_Between_C_and_Harbour/Harbour_C_2025_11_10.pdf`](./How_to_Interface_Between_C_and_Harbour/Harbour_C_2025_11_10.pdf)
- Diagram image: [`How_to_Interface_Between_C_and_Harbour/stack_diagram.png`](./How_to_Interface_Between_C_and_Harbour/stack_diagram.png)

### 4) Repo meta

- License: [`LICENSE`](./LICENSE) (CC BY-NC-SA 4.0)

---

## How to use these resources with AI

Typical workflow:

1. Provide the rulebook (`harbour_language_rules.md`) and your project defaults as context (system prompt / project instructions).
2. Provide the function dataset (YAML) as an authoritative “allow-list” for which functions can be used.
3. Use `personal_coding_standards.md` to enforce consistency (formatting, naming, patterns).
4. Use the interop guide when generating `#pragma BEGINDUMP` code or writing C extensions.

If you are publishing derivative work that includes the translated book content, consider adding an explicit attribution note in your downstream project and verifying that your usage matches the license terms and the author approval conditions described in this repository.

---

## Contributing

The schema and companion documents may evolve. Please open an issue with suggestions or corrections. Pull requests are welcome.

---

© 2025 Eric Lendvai — Licensed under CC BY-NC-SA 4.0
