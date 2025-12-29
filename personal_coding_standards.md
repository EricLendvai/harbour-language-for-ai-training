<!-- Updated: 2025-12-29 -->

# personal_coding_standards.md

This document defines **personal, cross-project coding standards** intended to keep Harbour code (and supporting code) consistent, readable, and AI-friendly. Repositories may extend or override these rules, but deviations should be documented explicitly.

---

## 1) Scope and priorities

1. **Correctness first**, then readability, then terseness.
2. Prefer **explicit** code over clever code.
3. Prefer **maintainability** (easy to diff, easy to review) over micro-optimizations unless performance is demonstrably impacted.
4. Avoid relying on undocumented or ambiguous language behavior.

---

## 2) Formatting

### 2.1 Indentation and whitespace

- Indentation is **4 spaces**.
- Inside `function`/`procedure`/method bodies, the **initial statement level is not indented** (column 1). Indent **only nested blocks** using 4 spaces per level.
- **Tabs are not allowed** (neither for indentation nor alignment).
- Do not include trailing whitespace at end-of-line.
- Use consistent newline endings (LF preferred; repository may override if needed).

### 2.2 Line breaks and wrapping

- Prefer **longer lines** when they remain readable.
- Avoid unnecessary line wrapping that introduces continuation lines.
- This preference assumes modern wide displays (e.g. 32" monitors, 4K resolution with high scaling).
- Wrap lines **only when readability clearly suffers**, not to satisfy an arbitrary line-length limit.
- When wrapping is required, align continuation lines to preserve structure (no arbitrary or decorative alignment).

### 2.3 No initial indentation for top-level constructs

- Do **not** indent top-level declarations:
  - `function`, `procedure`
  - `class`
  - method definitions (when written outside the class body depending on style)

### 2.4 Blank lines after block openers

- Do **not** insert a blank line immediately after a `function`/`procedure`/method declaration line.
- Do **not** insert a blank line immediately after a `for` loop header line.
- (General) Use blank lines to separate logical blocks of code, not to pad structure.

---

## 3) Naming and identifiers

### 3.1 No abbreviations (with limited acronym exceptions)

- **No abbreviations** for any user-defined tokens:
  - variables, parameters, functions, procedures, methods, classes, properties, constants
- Prefer full words, even if longer. Clarity is the priority.

Allowed exceptions (common technical acronyms / domain terms) may appear in identifiers:
- `API`, `DB`, `FCGI`, `HTML`, `HTTP`, `HTTPS`, `JSON`, `ORM`, `SQL`, `URL`, `UUID`, `UTF8`

Examples:
- Prefer `l_cCustomerName` over `l_cCustNm`
- Prefer `par_nInvoiceNumber` over `par_nInvNo`

### 3.2 Local variables

All local variables:

- Start with `l_`
- Then a **single type letter**
- Then a **CamelCase** descriptive name
- Avoid additional underscores after the prefix (i.e., do not create `l_c_my_value`)

Type letters (core):

- `c` = string/character
- `n` = numeric
- `i` = integer (use when you intentionally treat it as an integer)
- `d` = date
- `t` = timestamp (date+time)
- `l` = logical/boolean

Type letters (data structures / objects):

- `a` = array
- `h` = hash/map
- `o` = object (class instance)

Type letters (special cases):

- `x` = unknown or variable type (type can change, or is intentionally not constrained)

Examples:
```harbour
local l_cCustomerName := ""
local l_nTotalAmount  := 0
local l_lIsEnabled    := .f.

local l_aRecordList   := {}
local l_hRowByName    := {=>}
local l_oConnection   := nil

local l_xResult       := nil
```

### 3.3 Parameters

Parameters follow the same scheme as locals, except the prefix is `par_`:

```harbour
par_cCustomerName
par_nTotalAmount
par_lIsEnabled
par_xResult
```

### 3.4 Preprocessor symbols

- All preprocessor symbols must be **uppercase**.
- Use `_` (underscore) as a separator between words.
- Version and configuration symbols must be explicit and descriptive.

Examples:
```harbour
#define MIN_HARBOUR_ORM_VERSION  "4.17"
#define MIN_HARBOUR_EL_VERSION   "4.4"
#define MIN_HARBOUR_FCGI_VERSION "1.15"
```

### 3.5 Other scopes (recommended extensions)

If you use additional scopes consistently, apply the same philosophy:

- `s_` for `static` (recommended): `s_cCacheKey`, `s_nCounter`
- `g_` for explicit global concepts (discouraged; prefer explicit injection)

If a repository defines additional prefixes, document them near the top of that repo.

---

## 4) Harbour keyword casing

- All Harbour language commands/syntax must be **lower case**.
- This includes keywords such as: `function`, `local`, `if`, `elseif`, `endif`, `return`, `class`, `method`, etc.

This rule applies to Harbour source and any generated Harbour code.

---

## 5) File layout and separators

### 5.1 Routine separators

- Place a separator line between functions/procedures/methods:

```harbour
//--------------------------------------------------------------------------------
```

That is `//` followed by **80** `-` characters.

- Do **not** insert a blank line between a routine’s final `return` statement and the separator line that follows.

### 5.2 Class separators

- Place a separator line between classes. The separator may be **80 or more** `=` characters.

Common forms:
```harbour
//================================================================================
```

Or longer (allowed/preferred in some codebases):
```harbour
//=================================================================================================================
```

Rule:
- Be consistent within a file/module. Do not reformat separators purely for style if it increases diff noise.

---

## 6) Assignment and mutation rules

### 6.1 Assignment operator

- All variable assignments use `:=` (never `=`).
- Do not rely on contexts where `=` can assign.

### 6.2 String concatenation into an existing variable

- When appending string content to an existing variable, use `+=`:

```harbour
l_cHtml += "<div>"
l_cHtml += "</div>"
```

### 6.3 One declaration per `local`

- Avoid defining multiple variables in a single `local` statement.
- Define each variable on its own line to keep diffs clean and to allow per-variable comments.

Preferred:
```harbour
local l_cCustomerName := ""
local l_nTotalAmount  := 0
local l_lIsEnabled    := .f.
```

Avoid:
```harbour
local l_cCustomerName := "", l_nTotalAmount := 0, l_lIsEnabled := .f.
```

---

## 7) Alignment rules for readability

### 7.1 Align consecutive assignments

If multiple consecutive lines assign values, vertically align `:=`:

```harbour
local l_cFirstName      := ""
local l_cLastName       := ""
local l_cFullName       := ""
local l_nCustomerNumber := 0
```

### 7.2 Align trailing comments (controlled)

When multiple consecutive lines include trailing comments:

- Use **exactly three spaces** before the comment on the longest line.
- Shorter identifiers align naturally.

Example:
```harbour
local l_cFirstName := ""   // given name
local l_cLastName  := ""   // family name
local l_cFullName  := ""   // display name
```

Rules:
- Use alignment only within a short, related block.
- Do not attempt to align across large spans of code.

### 7.3 No decorative spacing inside expressions

Do **not** add extra spacing inside expressions purely for visual grouping.

Avoid:
```harbour
l_nTotal := l_nValue1 + ( l_nValue2 * l_nValue3 )
```

Preferred:
```harbour
l_nTotal := l_nValue1+(l_nValue2*l_nValue3)
```

---

## 8) Comments

### 8.1 Comment style

- Prefer `//` comments for single-line commentary.
- Use comments to explain **why**, not what the code obviously does.

### 8.2 Comments on declarations

Since each `local` is declared on its own line, add comments where it improves clarity:

```harbour
local l_tLastUpdateTimestamp := hb_DateTime()   // record update time
```

---

## 9) Control-flow and structure preferences

These are **intentional personal preferences**, not defaults imposed by the language:

- Deeply nested control blocks are acceptable when they reflect the logical flow.
- Avoid introducing temporary variables solely to flatten logic.
- Helper functions are created **only if reused in more than one place**.
- Longer functions are acceptable as long as they serve a **single, unique purpose**.
- Prefer **single-exit routines**; avoid mid-routine `return` statements.

---

### 9.1 Explicit END forms (Harbour Extended Library)

Although Harbour allows the generic `end`, prefer explicit terminators for clarity:

- `endtry`
- `enddo`
- `endfor`
- `endcase`
- `endwith`
- `endif`
- `endswitch`
- `endwhile`

These are defined in `hb_el.ch` (Harbour Extended Library, `harbour_el`):

```harbour
#xcommand ENDTRY     => END
#xcommand ENDDO      => END
#xcommand ENDFOR     => END
#xcommand ENDCASE    => END
#xcommand ENDWITH    => END
#xcommand ENDIF      => END
#xcommand ENDSWITCH  => END
#xcommand ENDWHILE   => END
```

#### Casing rule for explicit END forms

- **Always write these keywords in lower case** in Harbour source code (`endtry`, `endif`, etc.).
- The uppercase forms exist **only at the preprocessor definition level**.
- This follows the global rule that all Harbour commands and syntax must be lower case.

### 9.2 `iif()` vs `if()` (function form)

Harbour provides both `iif()` and `if()` as expression-level conditional functions. Prefer `iif()` exclusively to avoid visual ambiguity with the `if/elseif/else/endif` statement syntax.

---

## 10) Preprocessor usage (recommended defaults)

- Prefer `#define` for constants and build flags only.
- Avoid using the preprocessor for runtime logic.
- If build flags are needed, prefer `hbmk2 -dFLAGNAME` and test via `#ifdef`.

---

## 11) Exception handling and errors (recommended defaults)

- Use a consistent error-reporting format across projects.
- When generating error output, include:
  - operation
  - filename (if present)
  - call stack / procedure name list (as applicable)
- Prefer structured helpers over ad-hoc concatenation.

---

## 12) Repository-level overrides

If a repo must diverge from this document:

- Add a short `Repository overrides` section in that repo’s README or `/docs/`.
- List each override with:
  - the rule being overridden
  - the reason
  - the scope (file set, module, or entire repo)

---

**End of personal_coding_standards.md**
