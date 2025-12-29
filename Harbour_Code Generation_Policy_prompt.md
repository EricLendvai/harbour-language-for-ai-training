## Harbour Code Generation Policy prompt

**Policy (order of precedence)**

1. **harbour_language_rules.md** is the primary authority for Harbour semantics, idioms, and “what to generate / never generate.”
2. **personal_coding_standards.md** is the primary authority for formatting, layout, naming, and stylistic conventions (indentation, `endfor`, blank-line rules, separators, etc.).
3. **harbour_functions_part_001.yaml** is the **allowlist** for Harbour *function calls*. Any Harbour function used must exist in this YAML file.
4. If a requirement conflicts across sources, follow this resolution rule:

   * If **harbour_language_rules.md** *requires* a function that is not present in **harbour_functions_part_001.yaml**, then:

     * Prefer an equivalent approach that only uses allowlisted functions, **or**
     * Explicitly flag the conflict and request that the YAML allowlist be updated (do not silently use a non-allowlisted function).
   * If **harbour_language_rules.md** merely *recommends or prefers* a function that is not present in **harbour_functions_part_001.yaml**, then treat the allowlist as binding and use an allowlisted alternative without flagging it as an error.
5. Do not invent APIs. If a needed function is not in the allowlist and no alternative exists, stop and report exactly which function(s) are missing.

**Implementation requirements for every Harbour code answer**

* Ensure all function calls are in `harbour_functions_part_001.yaml`.
* Ensure formatting complies with `personal_coding_standards.md` (including: no top-level indentation inside routines, `endfor`, and no extra blank lines after `function`, after `for`, or before separator lines).
* Ensure behavior and guidance comply with `harbour_language_rules.md`.
* At the end of the response, include a short **“Functions used (allowlist check)”** section listing all Harbour functions called in the snippet.
