<!-- Updated: 2025-12-28 11:00 pm PST -->

# Harbour Language Rules — Unified Developer Guide (GPT‑Ready)

## Introduction

**Purpose.** Provide a clear, GPT-consumable rulebook for writing **syntactically correct, modern Harbour** code that compiles cleanly and follows consistent style.

Harbour is a modern, open-source, cross-platform programming language in the xBase/Clipper family.  
It preserves the clarity of classic xBase syntax while providing a powerful, optimized execution model that compiles source code into ISO C and then to native machine code. The result is a language that is both expressive and highly performant, well-suited for everything from small utilities to large, multi-module, multi-threaded applications.

Harbour is portable across all major operating systems (Windows, Linux, macOS, BSD) and supports both 32-bit and 64-bit targets. The language offers dynamic typing, strong runtime introspection, a modern object system, a flexible preprocessor, and first-class integration with C, C++, Python, JavaScript, SQL databases, and network protocols. UTF-8 is fully supported, with UTF8EX recommended for correct multilingual behavior.

Typical application domains include console tools, GUI or WebView-based desktop clients, FastCGI and HTTP servers, REST APIs, scripting runtimes, embedded components, and systems that require seamless interoperability with external libraries or services.

Harbour’s core language has remained stable for years. Most new activity happens in its extensive libraries and tools, making it an ideal foundation for long-term projects, code generation, documentation engines, and AI-assisted development workflows.

---

## Section List
1. Lexical rules
2. Identifiers & symbol length
3. Literals & core value types
4. Comparison & logical operators
5. Assignment & compound ops
6. Variables & scope (runtime vs compile-time)
7. Preprocessor (HbPP)
8. Routines & modules 
9. Control flow
10. Loops
11. Arrays & hashes (semantics that matter)
12. Code blocks (lambdas)
13. Objects & classes (modern Harbour)
14. Symbols & pointers (advanced)
15. Date, time, timestamp & UTC
16. NIL, NULL, empty — semantics & helpers
17. Legacy `SET` pseudo-commands and the `Set()` runtime (scope-limited)
18. Workareas, tables & indexes (ISAM model)
19. Error handling
20. Legacy pseudo-commands from `std.ch` beyond `set` (scope-limited)
21. System capacities (practical notes)
22. Database & ORM (orientation)
23. UTF-8 and Internationalization (DONE)
24. Harbour_EL (Extended Library) 
25. Multithreading & synchronization
26. GPT guardrails (for code generation)
27. Open items to expand later
28. Appendix — canonical “Hello World” build pointers

---

## 1) Lexical rules

This section describes how Harbour source code is tokenized and interpreted at the lexical level: characters, identifiers, literals, operators, comments, line breaks, and preprocessor constructs.

### 1.1 Character set and encoding

* **Language syntax is ASCII-only.**
  Keywords, identifiers, operators, and punctuation are defined over the ASCII subset.
* **String and character content may be UTF-8.**
  The parser treats source as bytes; the *semantics* of the string content (UTF-8 vs ANSI) is handled at runtime by the chosen codepage and string functions.
* **Recommended practice for internationalized code:**

  * Save `.prg` files as **UTF-8** in your editor.

  * In the PRG that contains `MAIN()` (or your entry point), request and select the UTF-8 extended codepage:

    ```harbour
    REQUEST HB_CODEPAGE_UTF8EX

    FUNCTION Main()
       hb_cdpSelect( "UTF8EX" )
       // ...
    RETURN NIL
    ```

  * `UTF8EX` is preferred over plain `UTF8` because it gives correct upper/lower case mapping for most European languages.

> Practical rule: The **language** is ASCII; your **data** (strings, DB content) should be UTF-8 in modern code.

---

### 1.2 Newlines and line separators

Harbour follows C-like line handling, *not* old Clipper’s strict DOS-only behavior:

* A logical “source line” ends at any of:

  * `LF` (`\n`)
  * `CR` (`\r`)
  * `CRLF` (`\r\n`)
* In practice:

  * Pure `LF` files (**Unix-style**, as VSCode can save) work fine.
  * `CRLF` (Windows-style) also works fine.
  * `CR`-only is unusual and may confuse tools/editors; don’t use it.

For generated source (e.g. when you emit PRG from another program), use:

```harbour
hb_eol()   // returns the OS-appropriate newline sequence
```

to remain portable.

---

### 1.3 Whitespace

* Spaces, tabs, and newlines separate tokens and make code readable.
* Indentation is **not** syntactically significant.
* There is **no** “Python-style” block layout. Block structure is controlled by keywords (`IF/ENDIF`, `DO CASE/ENDCASE`, `SWITCH/ENDSWITCH`, `CLASS/ENDCLASS`, `FUNCTION/RETURN`, etc.).

---

### 1.4 Comments

Four comment forms exist:

```harbour
/* block comment */
// single-line comment
*  whole-line comment (first non-space on the line)
&& end-of-line comment
```

* `/* ... */` can span multiple lines.
* `//` and `&&` run until the end of the current line.
* `*` is valid only when it is the first non-whitespace character.

---

### 1.5 Identifiers

* Allowed characters: `A–Z`, `a–z`, `0–9`, and `_`.
* Cannot start with a digit.
* **Case-insensitive:** `MyVar`, `MYVAR`, and `myvar` refer to the same identifier.
* **Significant length:** only the **first 63 characters** are significant for runtime symbol resolution.

  * Defining two variables whose names are identical in the first 63 characters (ignoring case) is a compile-time error.
* Preprocessor identifiers (`#define`, `#xtranslate`, etc.) are **case-sensitive** and may be much longer than 63 characters.

> Rule: For runtime identifiers, keep names ≤ 63 characters to avoid surprises.

---

### 1.6 Keywords and abbreviations

* Harbour keywords are **case-insensitive.**

* By design, it accepts **abbreviated keywords**:

  * If the keyword has 4 or more characters, typically the first **4 characters are sufficient**.
  * For example:

    ```harbour
    DO CASE         // full
    OTHE            // abbreviation of OTHERWISE
    RETU            // abbreviation of RETURN
    ```

* **Best practice:** always use the **full keyword** for readability and to avoid ambiguity.

---

### 1.7 Literals overview

Harbour supports the following literal forms:

* **String literals**
* **Numeric literals**
* **Logical literals**
* **Date literals**
* **Timestamp (datetime) literals**
* **Array literals**
* **Hash literals**
* **Codeblock literals**
* **Symbolic literals**
* **Pointer values** (from C-side, not a direct source literal)

Each is covered briefly below.

---

### 1.8 String literals

Supported delimiters:

```harbour
"Hello"
'Hello'
[Hello]
e"Line1\nLine2 \"quoted\""
```

* `"..."`, `'...'`, and `[...]` are regular string literals.
* `e"..."` is a **C-style escaped** string literal; the following escapes are supported:
  `\a \b \f \n \r \t \v \\ \' \" \? \xNN \0NNN`
* Runtime strings are limited by memory; **single PRG literals** have practical limits (see 21).
* String content can be ANSI or UTF-8; use functions like `hb_StrIsUTF8()` and `hb_utf8Len()` to inspect.

---

### 1.9 Numeric literals

Examples:

```harbour
1
3.1428
-12462
0xFF
0XFFFF
```

* Internally, numerics are stored by the VM as **integer/long/double** depending on magnitude and decimals (do not assume “double everywhere”).
* If a value is coerced to floating point, exact integer representation is guaranteed only up to about **2^53**; avoid mixing large integers with decimals when precision matters.
* Hex literals use `0x` or `0X`.

Type code: `ValType()` returns `"N"`.

---

### 1.10 Logical literals

```harbour
.t. .T. .f. .F.
```

* Logical literals are only `.T.` (true) and `.F.` (false); `NIL` is a distinct “no value” (`ValType() == "U"`) and should not be treated as a third logical state.
* Type code: `ValType()` returns `"L"`.

---

### 1.11 Date literals

Two equivalent forms:

```harbour
{^ 2022-03-20}   // ISO-style date literal
0d20220320       // compact date literal
```

* Template `{^ YYYY-MM-DD}` or `0dYYYYMMDD`
* Type code: `"D"`.

---

### 1.12 Timestamp (datetime) literals

Examples:

```harbour
{^ 2022-03-20 10:24:30 pm}
t"2022-03-20 22:24:30.500"
t"2022-03-20T22:24:30.500"
```

* Multiple flexible formats are accepted.
* Type code: `"T"`.
* Differences between two timestamps are expressed in **days (with fractional part)**, not seconds.

---

### 1.13 Array literals and multidimensional arrays

Basic arrays:

```harbour
aArray1 := { 1, 2, 3 }
aArray2 := { { 1, 2, 3 }, { 4, 5, 6 } }
aArray3 := { 1, 2, 3, "hello", "a", 2 }
```

* Arrays may contain values of **any type**, including other arrays, objects, hashes, codeblocks, etc.
* Arrays are **1-based**, not 0-based.
* Access forms:

```harbour
aArray1[1]        // first element
aArray2[2,1]      // row 2, col 1 (2D syntax)
aArray2[2][1]     // equivalent nested-array access
```

#### 3-dimensional (and higher) arrays

Harbour supports arbitrary nesting, so “N-dimensional arrays” are just arrays-of-arrays:

```harbour
a3D := { ;
    { { 1, 2 }, { 3, 4 } }, ;
    { { 5, 6 }, { 7, 8 } } ;
}

// All of these are valid:
? a3D[1][1][2]    // 2
? a3D[2,1,2]      // 6 (2D+1D style)
```

* `ACopy()` copies only the **first dimension**.
* `AClone()` does a **deep copy of arrays** (but not nested objects).
* Arrays are passed **by reference**; assigning one array to another variable copies the reference, not the content.

---

### 1.14 Hash literals

```harbour
hPerson := { ;
   "FirstName" => "Eric", ;
   "LastName"  => "Lendvai", ;
   1241        => "Seattle", ;
   {^ 2022-03-20} => .t. ;
}
```

* Keys may be `String`, `Numeric`, `Date`, or `Timestamp` (or more, but those are the common ones).
* Mixed key types are allowed in a single hash.
* Initialize with `{=>}` or `hb_Hash()` to ensure a true hash, especially when numeric keys are used.

---

### 1.15 Codeblock literals

Short form:

```harbour
{|a,b| a + b }
{|a,b| DoThing(a,b), 2*a + b }   // last expression is the return value
```

Extended (multi-line) form:

```harbour
{|a,b|
   LOCAL nMulti := 2
   DoThing( a, b )
   RETURN nMulti * a + b
}
```

* Codeblocks are like lambdas; they’re precompiled pieces of code used heavily in array functions, workarea operations, and event callbacks.

---

### 1.16 Symbolic literals

Symbolic literals are **references to functions/procedures/methods**:

```harbour
sFunc := @MyFunction()
sMeth := @oObject:Method()

? sFunc:name()        // name of the function
sFunc:exec( 1, 2, 3 ) // invoke with parameters
```

* Type code (in your own introspection) is often `"P"` (Pointer) or `"U"` depending on how you inspect.
* Think of them as late-bound function references at the Harbour VM level.
* Used for advanced callback systems, dispatch tables, and integration with C.

---

### 1.17 Assignment and compound assignment operators

#### 1.17.1 Basic assignment

Two assignment syntaxes exist:

```harbour
nValue := 1     // preferred
nValue =  1     // allowed, but not recommended
```

* `:=` is the **canonical assignment operator** and is what you should use almost everywhere.
* `=` is overloaded:

  * As an **assignment** operator.
  * As a **relational** (“loose”) equality operator (affected by `SET EXACT` for strings).

> **Rule:** always use `:=` to avoid confusion with equality tests.

#### 1.17.2 Required use with `LOCAL` and `STATIC`

When initializing local and static variables in declarations, **only `:=` is allowed**:

```harbour
FUNCTION Main()
   LOCAL nCount := 0        // ✅ valid
   // LOCAL nCount = 0      // ❌ invalid syntax

   STATIC sCache := {}      // ✅ valid
RETURN NIL
```

---

#### 1.17.3 Compound assignment

Harbour supports C-style compound assignments (as you already documented in your draft):

```harbour
nNumber += 3   // add and assign
nNumber -= 2   // subtract and assign
nNumber *= 4   // multiply and assign
nNumber /= 5   // divide and assign
nNumber %= 3   // modulo and assign
nNumber **= 2  // power and assign
nNumber ^=  2  // power and assign (alias of **=)
```

* These work as syntactic sugar:

  ```harbour
  n += x  // equivalent to  n := n + x
  ```

* `+=` and `-=` can also be used with **dates and timestamps** where the right-hand operand is a number of days (fraction allowed for timestamps).

> `=+` and `=-` are **not** separate operators.
> They parse as `=` assignment followed by unary `+` or `-` and are confusing; avoid them.

---

### 1.18 Increment / decrement operators (`++`, `--`)

Pre- and post-forms:

```harbour
nNumber := 3

? nNumber++   // prints 3, then nNumber becomes 4  (post-increment)
? nNumber     // prints 4

? --nNumber   // pre-decrement: nNumber becomes 3, then prints 3
? nNumber     // prints 3 again
```

* Valid for **numeric**, **date**, and **timestamp** values.
* For dates and timestamps, `++`/`--` adjust by **1 day**.
* This is the usual C-like behavior:

  * `++var` / `--var`: increment/decrement first, then evaluate.
  * `var++` / `var--`: evaluate first, then increment/decrement.

---

### 1.19 Comparison operators and “strict” equality

Operators (subset):

```harbour
=    // loose equality (relational equality, string behavior depends on SET EXACT)
==   // strict equality
!=   // not equal (non-strict)
<>   // not equal (non-strict)
<    // less than
>    // greater than
<=   // less than or equal
>=   // greater than or equal
$    // substring membership (string in string)
```

#### 1.19.1 `==` (strict equality)

* `==` is the **exact equality** operator:

  * For strings, it compares **length + every character**, including trailing spaces.
  * Not affected by `SET EXACT`.
* For non-strings, it behaves as usual equality with proper type semantics.

#### 1.19.2 `=` (loose equality)

* `=` is a **relational equality** operator that **is affected by `SET EXACT`** for strings:

  * With `SET EXACT OFF` (the typical setting), strings are considered equal if:

    * They match up to the length of the shorter one, ignoring trailing spaces.
  * This allows `"ABC"` to be “equal” to `"ABCD"` under some conditions — highly error-prone.

> **Big gotcha:**
> If you want **strict string equality**, always use `==`, never `=`.

#### 1.19.3 No “strict not equal” operator

There is **no dedicated strict-not-equal operator** corresponding to `==`.

* `!=` and `<>` use the same *loose* string semantics as `=` when `SET EXACT` is involved.
* To test for **strict inequality**, use:

```harbour
!( x == y )
```

This is the only fully reliable “strict not equal” for strings.

#### 1.19.4 `$` (string membership)

The `$` operator is string-specific:

```harbour
"ell" $ "Hello"   // .T.
"HEL" $ "Hello"   // .F. (case-sensitive)
```

* Both operands must be strings.
* Returns `.T.` if the left-hand string occurs anywhere inside the right-hand string.
* Always **case-sensitive**.

---

### 1.20 Arithmetic and strings

* Proper arithmetic operators (`+`, `-`, `*`, `/`, `%`, `**`) are intended for **numeric, date, or timestamp** operands.

* `+` is also used for **string concatenation**:

  ```harbour
  c := "Hello" + " World"
  ```

* `+=` is widely used as the **fastest way to build long strings**:

  ```harbour
  cBuffer := ""
  cBuffer += "Line 1" + hb_eol()
  cBuffer += "Line 2" + hb_eol()
  ```

* Other arithmetic operators on strings (`-`, `*`, `/`, `%`, powers) are **not recommended**:

  * Behaviour may be surprising or rely on implicit numeric conversion.
  * Treat it as undefined style-wise; only use `+`/`+=` for strings.

> Rule of thumb:
> **Only** `+` and `+=` are “approved” for strings. Everything else is numeric/date/time only.

---

### 1.21 Line continuation and TEXT/ENDTEXT

#### 1.21.1 Semicolon continuation

A physical line can be continued on the next line by ending it with `;`:

```harbour
cText := "This is a very long string " + ;
         "continued on the next line."
```

* The semicolon tells the preprocessor to join the next line with the current one before parsing.
* This is the opposite of JavaScript (where `;` ends a statement).

#### 1.21.2 TEXT / ENDTEXT block

Harbour’s preprocessor also supports `TEXT ... ENDTEXT` blocks, but only for "TO FILE" and "TO PRINTER". Should be avoided since TO PRINTER is mainly for legacy code and TO FILE could not handle exceptions.   
A much more practical version, using the Harbour_EL library, version 4.5 and higher, provide support to `TEXT TO VAR <cVariable> [ADDITIVE] ... ENDTEXT`.   

```harbour
TEXT TO VAR cTemplate
Hello <<cName>>,

Thank you for using our product.

Best regards,
The Team
ENDTEXT
```

* Inside a `TEXT ... ENDTEXT` block:

  * Lines are treated as **raw text** and not parsed as Harbour statements.
  * All lines are collected into a single variable.
* This is useful for:

  * Multi-line templates
  * Documentation blocks
  * Generating PRG, HTML, or other text from within Harbour

**TEXT/ENDTEXT suspends normal tokenization inside the block**.

---

### 1.22 Preprocessor directives

Harbour has a powerful preprocessor (HbPP). At the lexical level, you should be aware of:

```harbour
#include "file.ch"

#define NAME value
#undef  NAME

#translate pattern => replacement
#xtranslate pattern => replacement    // extended (xBase-style) patterns

#command  pattern => replacement
#xcommand pattern => replacement      // extended command syntax

#if expr
#elif expr
#else
#endif

#ifdef  NAME
#ifndef NAME
```

* `#translate` / `#xtranslate` operate on **expressions**.
* `#command` / `#xcommand` operate on **command-style syntax** (e.g. `USE`, `APPEND FROM`).
* `#x*` variants support wildcards and richer patterns than the base forms.

---

### 1.23 Macro expansion differences (Harbour vs VFP)

Harbour’s macro operator is `&`, but its behavior differs from VFP in important ways:

* Macros are **text-based** substitutions that are compiled/evaluated into an expression or statement.
* Harbour does **not** support some of the more dynamic VFP patterns where you build variable names piecemeal and then macro-expand them as identifiers.

  * For example, patterns like `&("c"+cSuffix)` to dynamically refer to a variable name are far less idiomatic and less reliable in Harbour.
* Good practice in Harbour:

  * Use macros sparingly, mainly for:

    * Dynamic expressions
    * Configurable codeblocks / SQL fragments
    * Some legacy xBase syntax
  * For dynamic “collections” of values, prefer:

    * Arrays / Hashes
    * Objects
    * Workareas and `FIELD->` / `M->` prefixes
    * The `__mv*` family for working with PUBLIC/PRIVATE variables at runtime.

> High-level rule: macros are powerful but not a replacement for proper data structures.
> VFP-style “build a variable name, then macro it” patterns should be replaced with hashes/arrays/objects in modern Harbour code.

---

## 2) Identifiers & symbol length

### 2.1 Allowed form  
- Must start with a **letter** (`A–Z`, `a–z`) or `_`.  
- After that: letters, digits, `_`.  
- Identifiers are **ASCII‑only**.  
- The compiler accepts long identifiers, but only the **first 63 characters** are significant for runtime name resolution; collisions beyond that can produce errors or overwrites (see 2.3).

---

### 2.2 Case‑insensitive runtime behavior  
Harbour resolves identifiers **case‑insensitively** in the runtime symbol space:

```
MyVar
MYVAR
myvar
```

All refer to the same identifier.

---

### 2.3 The **63‑character significant length** rule  
For all **runtime** identifiers, Harbour uses:

- **Case‑insensitive comparison**
- **Only the first 63 characters are significant**

Characters after the 63rd are ignored for matching.

Two identifiers that match in the first 63 chars (ignoring case) are considered **the same name**.

---

### 2.4 LOCAL variables (compile‑time only, collision‑checked)  
LOCALs do **not** enter the runtime symbol table, but *Harbour still enforces the 63‑character rule*:

```
local VeryLong_...ABC
local VeryLong_...XYZ   // first 63 chars equal → compile‑time error
```

Compiler error:

```
Error: Duplicate variable declaration
```

✔ LOCALs are the safest namespace — collisions are detected.  
✔ LOCALs do not silently overwrite.

---

### 2.5 STATIC variables  
STATIC variables *do* become runtime symbols:

- Case‑insensitive  
- First 63 characters significant  
- Collisions produce compile‑time errors  

STATIC behaves like function/procedure symbols.

---

### 2.6 PUBLIC / PRIVATE variables (memvar namespace — **no collision checking**)  
PUBLIC/PRIVATE identifiers use the **same 63‑character runtime significance**, but:

- **No collision warnings**  
- **Last assignment silently overwrites previous**  

Example:

```
private VeryLongName_...A := 1
private VeryLongName_...B := 2   // same 63 chars → overwrites previous
? VeryLongName_...A              // prints 2
```

⚠ Dangerous in large systems  
⚠ Avoid long PUBLIC/PRIVATE names  
⚠ Prefer LOCAL whenever possible

---

### 2.7 CLASS names  
CLASS names behave like FUNCTION/PROCEDURE names:

- Case‑insensitive  
- First 63 chars significant  
- **Collisions produce compile‑time error**:

```
Error: Redefinition of procedure or function
```

Class names participate in the **checked** symbol namespace.

---

### 2.8 METHOD names (**behave like DATA/VAR — no collision checking**)  
Your testing confirms:

- METHOD identifiers use the same 63‑char significance  
- Case‑insensitive  
- **No duplicate checking**  
- **Last one silently overwrites previous**  

METHOD names do **not** behave like function names — they behave like class slot identifiers.

---

### 2.9 DATA and VAR (class properties)  
DATA and VAR slot names:

- Use case‑insensitive 63‑char significance  
- Have **no collision detection**  
- Last defined wins silently  

Because the DATA/VAR namespace mirrors memvar‑style behavior, **using DATA is recommended** to avoid confusion with normal variables and to allow future getter/setter expansion.

---

### 2.10 Preprocessor identifiers (#define, #command, #translate)  
- **Case‑sensitive**  
- **Unlimited length**  
- Not part of the symbol table  
- Not affected by 63‑char limit  
- Cannot collide with runtime identifiers

The preprocessor is a separate textual‑replacement system.

---

### 2.11 Summary table

| Identifier Type | 63‑char rule? | Collision Checked? | Silent Overwrite? | Notes |
|-----------------|---------------|--------------------|-------------------|-------|
| **LOCAL** | yes | **Yes** | no | Compile‑time scope only |
| **STATIC** | yes | **Yes** | no | Runtime symbol table |
| **FUNCTION / PROCEDURE** | yes | **Yes** | no | Global runtime symbols |
| **CLASS names** | yes | **Yes** | no | Behave like function names |
| **METHOD names** | **yes** | **No** | **Yes** | Behave like DATA/VAR |
| **DATA / VAR** | yes | **No** | **Yes** | Class slot namespace |
| **PUBLIC / PRIVATE** | yes | **No** | **Yes** | Memvar namespace |
| **Preprocessor identifiers** | **no** | n/a | n/a | Separate, case‑sensitive |

---

### 2.12 Safe naming recommendations  
- Keep identifiers **≤ 40 chars** to avoid future 63‑char clashes.  
- Avoid long dynamically generated names.  
- Never rely on case differences.  
- Prefer LOCAL over PRIVATE/PUBLIC.  
- Prefer **DATA** over VAR for clarity and future expandability.

---

## 3) Literals & core value types

Harbour is dynamically typed. A variable’s type is determined at runtime by the value assigned to it.
**NIL** represents the absence of a value and is the only literal that means “no value”.

Harbour has the following core literal types:

| Type      | Literal examples                               | Notes                                             | `ValType()` |
| --------- | ---------------------------------------------- | ------------------------------------------------- | ----------- |
| NIL       | `NIL`                                          | Absence of value                                  | `"U"`     |
| Logical   | `.T.` `.F.`                                    | Case-insensitive                                  | `"L"`     |
| Numeric   | `0`, `-7`, `3.14159`, `0xFF`                   | No exponent notation (`1e3` not allowed)          | `"N"`     |
| String    | `'text'`, `"text"`, `[text]`, `e"line\nnext"` | Escapes only in `e"..."` / `e'...'`             | `"C"`     |
| Date      | `{^2025-11-12}`, `0d20251112`                  | Templates: `{^YYYY-MM-DD}` or `0dYYYYMMDD`        | `"D"`     |
| Timestamp | `{^ 2022-03-20 10:24:30 pm}`                   | Date+time (see 15)                                | `"T"`     |
| Array     | `{ 1, 2, "a", { "x" } }`                   | By reference (see 11)                             | `"A"`     |
| Hash      | `{=>}`, `{ "key" => 42, 7 => "seven" }`    | Associative map                                   | `"H"`     |
| Codeblock | *(see below)*                                  | Callable; executed via `Eval()`                   | `"B"`     |
| Symbol    | `@FuncName()`                                  | Advanced; symbol handle                           | `"S"`     |
| Pointer   | *(from C only)*                                | Advanced; raw pointer                             | `"P"`     |

Codeblock literal syntax:

```harbour
{|| ... }
```

Strings may hold ANSI or UTF-8 data; correct multilingual behaviour depends on the selected codepage (recommended: **UTF8EX**). 

---

### 3.1 String literals and encoding

Harbour supports four string literal syntaxes:

```harbour
"abc"
'abc'
[abc]
e"Line1\nLine2"
```

**`"..."` and `'...'`**

* Standard quoted string forms.
* No C-style escapes unless prefixed with `e`.
* To embed quotes, either switch delimiters or escape them in `e"..."` / `e'...'`.

**`[ ... ]` literals**

Square-bracket string literals begin at `[` and end at the **first** `]`.
Harbour does not support escaping or nesting of `]` inside the literal.

Example:

    [abc]def

The string literal is `"abc"` and `def` is parsed as normal source code.
If that trailing code is not valid Harbour syntax, a compile-time error occurs.

**Guideline:** do not place `]` inside `[ ... ]` literals. If you need embedded `]` characters, use `"..."` or `'...'` instead.


**`e"..."` / `e'...'` (escaped literals)**

* Support C-style escapes such as `\n`, `\r`, `\t`, `\\`, `\"`, `\'`, `\xNN`, and `\0NNN`.
* Use these when you explicitly need control characters or embedded quotes.

**Encoding**

* The compiler treats string literals as sequences of bytes.
* They may contain ANSI or UTF-8 sequences.
* Correct case conversion and substring handling for UTF-8 requires selecting a suitable codepage at runtime (e.g., `UTF8EX`, see §23).

**Size**

* Literal strings in source can be large (on the order of megabytes); the practical hard limit is ~16 MB per literal.

---

### 3.2 Numeric literals

Valid numeric literals include:

```harbour
123
-45
3.14159
0xFF
```

Rules:

* **No exponential notation.** Forms like `1e3`, `2E-5`, etc. are **not** valid Harbour numeric literals.
* Hexadecimal literals use `0x` / `0X`.
* Internally, numerics may be stored as **integer/long/double**; operations may promote to floating point when decimals are involved.
* Avoid relying on IEEE-754 corner cases; explicitly `Round()` / convert when precision is required.

---

### 3.3 Logical, date and timestamp literals

**Logical**

```harbour
.T.
.F.
```

* Case-insensitive.
* Only `.T.` and `.F.` are logical values; `NIL` is a separate “no value” and should be handled explicitly if it can occur.

**Date**

```harbour
{^2025-01-31}
0d20250131
```

* Must be valid calendar dates.
* `ValType()` returns `"D"`.

**Timestamp**

```harbour
{^ 2022-03-20 10:24:30 pm}
t"2022-04-06 20:44:45.437"
t"2022-04-06T20:44:45"
```

* Fractions of seconds are allowed (e.g. `.437`).
* `t"..."` is case-insensitive (`T"..."` also acceptable).
* **Timezone suffixes are not supported** (no `Z`, `+02:00`, etc.).
* Subtracting two timestamps yields a **numeric number of days** (fractional).

---

### 3.4 Arrays and hashes (literal aspects only)

**Array literals**

```harbour
a := { 1, 2, "a", { "x" } }
```

* 1-based indexing: `a[1]` is the first element.
* Can contain any value type, including other arrays, hashes, objects, codeblocks, etc.
* Literal arrays can contain up to **65,535** elements.
* `ACopy()` performs a shallow, first-dimension copy; `AClone()` deep-copies arrays (not objects).

**Hash literals**

```harbour
h := { ;
   "Name" => "Eric", ;
   7      => "Seven", ;
   {^2025-01-01} => .T. ;
}
```

* Keys may be `String`, `Numeric`, `Date`, or `Timestamp`.
* Mixed key types are allowed in the same hash.
* Use `{=>}` or `hb_Hash()` to create an empty hash.
* To preserve insertion order, call `hb_HKeepOrder( h, .T. )`.

---

### 3.5 Codeblocks and symbols

**Codeblocks** are Harbour’s lambda-like callables.

Short form:

```harbour
{|a,b| a + b }
```

Multi-line form:

```harbour
{|a,b|
   LOCAL n := a * 2
   RETURN n + b
}
```

Key points:

* Codeblocks are **callable values**; they are executed with `Eval( cb, args… )`.
* Without an explicit `RETURN`, the last expression is the return value.
* They **do not implement full lexical closures** like JavaScript/Python; they do not automatically capture arbitrary outer locals.
* When used inside methods, they can use `SELF` (class scope).

**Symbol literals**

```harbour
s := @MyFunction()
s:exec( 1, 2, 3 )
```

* `@FuncName()` creates a symbol reference to a function/procedure/method.
* Resolved at **compile time**, not constructed dynamically from strings.
* Useful for callback tables and dynamic dispatch.

---

### 3.6 Type interaction & string operations

This subsection is especially important for AI code generation.

#### 3.6.1 No automatic string↔numeric coercion

Harbour does **not** automatically convert between strings and numbers in arithmetic:

```harbour
"123" + 5        // invalid (runtime error), not 128
"12.4" * 2       // invalid (runtime error), not 24.8
```

If numeric behaviour is needed, convert explicitly:

```harbour
Val( "123" ) + 5     // 128
Val( "12.4" ) * 2    // 24.8
```

To concatenate numbers and strings, convert to string first:

```harbour
Str( 5 ) + "2"       // "         52" (default width is 10)
Str( 5, 1 ) + "2"    // "52"
```

Note: Because `Str( 5 )` defaults to a width of 10,
`Len( Str( 5 ) + "2" )` is `11`.

#### 3.6.2 String arithmetic and a known anomaly

The **only** arithmetic operators that should be used intentionally with strings are:

* `+`  (concatenation)
* `+=` (append / concatenation with assignment)

All other arithmetic operators on strings (`-`, `*`, `/`, `%`, `**`, etc.) must be considered invalid for new code.

There is one surprising behaviour:

```harbour
"5" - "2"    // yields "52"
```

This behaves as if it were `"5" + "2"`.
Treat this as an anomaly and never rely on it. AI must not intentionally generate such code.

#### 3.6.3 Valid arithmetic combinations

The safe, defined combinations are:

* Numeric ± Numeric
* Numeric × Numeric
* Numeric ÷ Numeric
* **Date ± Numeric**
* **Timestamp ± Numeric**

For dates/timestamps, the numeric operand represents **days** (fractional days for timestamps).

Anything else (e.g. Date + String, Array + Numeric, Logical × String) should be considered invalid and not generated.

---

### 3.7 NIL vs empty values

For literals:

* `NIL` — no value (`ValType() == "U"`).
* `""` — empty string.
* `{}` — empty array literal.
* `{=>}` — empty hash literal.
* There is **no `NULL` literal** in Harbour; do not confuse Harbour NIL with SQL NULL or VFP `.NULL.`.

---

### 3.8 NULL support via Harbour_EL (optional helper)

Core Harbour has **no native `NULL` literal**. At the language level, the only “no value” marker is **`NIL`**, which has `ValType() == "U"` and is what other languages would usually call `NULL`. 

When you use the **Harbour_EL** library, you gain an optional *naming convenience* that does **not** change the underlying value model but gives a more SQL/VFP-friendly API:

```harbour
// Provided by Harbour_EL (preprocessor macro)
IsNull( x )    // → expands to hb_IsNIL( x )
```

Implementation (conceptual):

```harbour
#xtranslate ISNULL( <x> ) => hb_IsNIL( <x> )
```

Key points for AI and human readers:

* There is still **no `NULL` literal** in Harbour source code.
* `IsNull( x )` (or `ISNULL( x )`) is just a **preprocessor alias** for `hb_IsNil( x )`. 
* `ValType()` for a “null” value is still `"U"` (NIL), exactly as in core Harbour.
* This helper exists purely to:

  * Make code more natural for developers coming from **SQL** or **Visual FoxPro** (where `NULL` and `IsNull()` are standard concepts).
  * Provide a **clear semantic mapping** in generated code: whenever a GPT wants to “test for NULL” in a Harbour_EL-based project, it should emit `IsNull( x )` (or `hb_IsNil( x )` if Harbour_EL is not assumed).

> For **database / ORM** handling of SQL `NULL` in table fields (as opposed to variables), see §16 “NIL, NULL, EMPTY — semantics & helpers” and §22 “Database & ORM (orientation)”, which describe how Harbour_ORM and related layers map backend `NULL` values to Harbour `NIL` plus ORM helpers.

---

## 4) Comparison & logical operators

Harbour’s comparison operators are type-dependent. In practice, **only string comparisons are affected by `SET EXACT`**, and only when using the `=` / `<>` operators.  
For numeric, logical, date, and timestamp values, `=` and `==` behave identically.

This section defines the exact semantics and establishes clear, human- and AI-safe usage rules.

---

### 4.1 Equality operators (`==`, `=`, `!=`, `<>`)

#### 4.1.1 `==` — strict equality (recommended for strings)

The `==` operator performs **strict, symmetric equality** and is **not affected** by `SET EXACT`.

Behavior by type:

- **String:** compares full length and every byte, including trailing spaces
- **Numeric:** compares numeric value
- **Logical:** compares `.T.` / `.F.`
- **Date / Timestamp:** compares absolute value
- **NIL:** `NIL == NIL` → `.T.`; `x == NIL` → `.T.` only if `x` is `NIL`

For **numeric, logical, date, and timestamp values**, `==` behaves **identically** to `=`.  
The distinction between `=` and `==` matters **only for strings**.

**Guideline:**  
Use `==` for all string comparisons. Using `==` for other types is preferred for readability, but not required.

---

#### 4.1.2 `=` — string comparison affected by `SET EXACT`

The single equals sign (`=`) is overloaded:

- Assignment operator
- Equality operator

For **non-string types** (numeric, logical, date, timestamp), `=` and `==` produce the **same result**.

For **strings**, `=` is affected by `SET EXACT` and should generally be avoided.

With `SET EXACT OFF`:

- The comparison is **not symmetric**
- The result depends on the **order of operands**
- The operator behaves like a *left-prefix test*

Examples:

| Expression | Result | Explanation |
|-----------|--------|-------------|
| `"ab" = "a"` | `.T.` | left starts with right |
| `"a" = "ab"` | `.F.` | left does not start with right |
| `"a " = "a"` | `.T.` | left starts with right |
| `"a" = "a "` | `.F.` | left is shorter than right |

With `SET EXACT ON`:

- `=` still does **not** provide strict, byte-for-byte equality
- Trailing spaces are not reliable comparison markers
- Behavior remains different from `==`

**Rule:**  
Avoid `=` for string comparisons. Do not rely on `SET EXACT` to achieve predictable string equality.

---

#### 4.1.3 Assignment vs comparison

Because `=` can be used for both assignment and comparison, the following convention is strongly recommended:

- Use `:=` **only** for assignment
- Use `==` **only** for comparison

This removes ambiguity for both humans and AI-generated code.

---

#### 4.1.4 `!=` and `<>` — not-equal

`!=` and `<>` are equivalent.

- For strings, they inherit the same `SET EXACT`-dependent behavior as `=`
- There is **no strict not-equal operator**

To express strict inequality, use:

```harbour
!( a == b )
```

---

### 4.2 Explicit alternatives to `=` for strings

If you intentionally need *left-prefix* semantics (the most common historical reason to rely on `=` with `SET EXACT OFF`), use the dedicated functions instead of `=`:

- `hb_LeftEq( <cString1>, <cString2> )`  
  Returns `.T.` if `<cString1>` starts with `<cString2>`

  Equivalent to:
  ```harbour
  Left( cString1, Len( cString2 ) ) == cString2
  ```

- `hb_LeftEqI( <cString1>, <cString2> )`  
  Same as `hb_LeftEq()`, but case-insensitive

These functions are explicit, symmetric in intent, and not affected by `SET EXACT`.

---

### 4.3 Ordering operators (`<`, `<=`, `>`, `>=`)

Ordering comparisons are valid only for **matching types**:

- Numeric ↔ Numeric
- Date ↔ Date
- Timestamp ↔ Timestamp
- String ↔ String (bytewise ordering)

Any other combination results in runtime errors or undefined behavior.

---

#### 4.3.1 String ordering

String ordering is **bytewise**, not locale-aware:

- No Unicode collation
- UTF-8 strings are compared by encoded bytes
- Case-sensitive

Do not use relational operators for linguistic sorting.

---

### 4.4 Logical values

For logical values (`.T.` / `.F.`):

- `=` and `==` behave identically
- `SET EXACT` has no effect

```harbour
.T. == .T.   // .T.
.F. =  .T.   // .F.
```

---

### 4.5 NIL in comparisons

Rules:

- `NIL == NIL` → `.T.`
- `x == NIL` → correct way to test for absence of value
- Ordering comparisons involving `NIL` are invalid and raise runtime errors

Avoid using `=` with `NIL`; prefer `==` or `hb_IsNil()`.

---

### 4.6 Comparison rules for hash keys

Hash key equality is strict:

- `"1"` and `1` are different keys
- `"A"` and `"A "` are different keys
- No type coercion occurs

---

### 4.7 Invalid comparison patterns

The following must not be generated:

- Array vs array
- Hash vs hash
- Object comparisons
- Codeblock comparisons
- Ordering with `NIL`
- Mixed string–numeric comparisons
- Mixed date–numeric without conversion

Explicit conversion must be used where appropriate.

---

### 4.8 Logical operators

Logical operators:

- `.and.`
- `.or.`
- `.not.`
- `!` (prefix)

Logical operators short-circuit:

- `.and.` stops if left operand is `.F.`
- `.or.` stops if left operand is `.T.`

---

### 4.9 Summary (human- and AI-friendly)

- `==` provides strict, symmetric string equality
- `=` on strings is `SET EXACT`-dependent and order-dependent
- Avoid `=` for string comparisons
- Use `hb_LeftEq()` / `hb_LeftEqI()` for explicit prefix tests
- `=` and `==` are identical for numeric, logical, date, timestamp
- Use `:=` for assignment
- Use `!(a == b)` for strict not-equal

---

## 5) Assignment & compound ops

Use `:=` for assignment. Single `=` can assign in some contexts, but should be avoided because `=` is also used for (non-strict) equality in expressions and is ambiguous in reviews.

```harbour
local l_nValue := 5
l_nValue := l_nValue + 1
```

### 5.1 Where assignment is allowed

The left-hand side (LHS) must be a mutable l-value:

- LOCAL / STATIC / PRIVATE / PUBLIC variable
- array element: `a[ i ]`
- hash element: `h[ key ]`
- object slot: `o:Prop` or `::Prop`

Not allowed on the LHS: literals, expressions, or function calls.

```harbour
++GetCount()          // invalid
( l_nA + l_nB ) := 1  // invalid
```

### 5.2 Compound assignment operators

Supported compound assignments:

- Numeric: `+=`, `-=`, `*=`, `/=`, `%=`, `**=` and `^=` (power)
- Date/Timestamp (days): `+=`, `-=`
- String: `+=` only (concatenation)

```harbour
local l_n := 5
l_n += 3       // 8
l_n **= 2      // 64

local l_c := "Hello"
l_c += " World"   // "Hello World"
```

Notes:

- `=+` and `=-` are **not operators** (and do not compile). Write `+=` / `-=` only.
- For dates/timestamps, the numeric operand is **days**. For timestamps, fractional days are meaningful.

```harbour
local l_t := hb_datetime()
l_t += 1/24        // +1 hour
l_t += 90/86400    // +90 seconds
```

### 5.3 Increment / decrement (`++`, `--`)

Pre/post forms exist and behave like C:

```harbour
local l_n := 10

? l_n++        // prints 10, then l_n becomes 11
? ++l_n        // increments first (12), then prints 12
```

Valid operand types: numeric, date, timestamp.

- For dates/timestamps: `++` / `--` adjusts by **1 day**.

Guideline: avoid embedding `++`/`--` inside complex expressions; prefer standalone statements for readability.

### 5.4 NIL and safe initialization

Compound operators and `++/--` require a non-NIL LHS value.

```harbour
local l_n := NIL
l_n += 1          // runtime error
```

Initialize explicitly (or default) before using compound operations:

```harbour
hb_Default( @l_n, 0 )
l_n += 1
```

### 5.5 Chained assignment and assignment in conditions

Both are allowed, but should be used sparingly:

```harbour
a := b := c          // allowed (right-to-left association)

if ( x := y ) == NIL // allowed but reduces readability
    ...
endif
```

Guideline: prefer explicit, separate statements unless the construct materially improves clarity.

---

## 6) Variables & scope (runtime vs compile-time)

Harbour supports **dynamic (runtime)** variables and **compile-time** variables.
Understanding their scope, lifetime, and performance characteristics is critical.

### Variable classes

| Feature | PUBLIC | PRIVATE | LOCAL | STATIC |
|---|---:|---:|---:|---:|
| Created at | runtime | runtime | compile-time | compile-time |
| Visible across PRGs | ✅ | ✅ | ❌ | module only |
| Name in symbol table | ✅ | ✅ | ❌ | ❌ |
| Speed | slow | slow | **fast** | **fast** |
| Lifetime | global | call-chain | routine | routine / module |
| Persists across calls | ❌ | ❌ | ❌ | ✅ |

**Best practice:**  
Prefer **LOCAL** almost everywhere for speed, safety, and determinism.  
Avoid `PUBLIC` entirely in modern code. Use `PRIVATE` only when intentionally relying on call-chain state.

---

### Declaration rules

- `LOCAL` and `STATIC` declarations **must appear before executable statements** in
  `FUNCTION`, `PROCEDURE`, and `METHOD` bodies.
- File-level `STATIC` variables are global **only within that PRG file**.
- Routine-level `STATIC` variables persist across calls but remain scoped to the routine.

---

### PRIVATE variables and call-chain scope

`PRIVATE` variables are **not function-local**.  
They are visible to the entire **call chain below the creator** and can be shadowed by inner PRIVATE declarations.

```harbour
function A()
    private p_Mode := "A"
    B()
    return NIL

function B()
    ? p_Mode        // "A"
    private p_Mode := "B"
    C()
    ? p_Mode        // "B"
    return NIL

function C()
    ? p_Mode        // "B"
    return NIL
```

---

### Dynamic variables, memvar, and workarea collisions

When using `PUBLIC` or `PRIVATE`, declare them with `memvar` and disambiguate explicitly:

- `M->Name` for memvars
- `Alias->Field` or `FIELD->Field` for workarea fields

```harbour
#include "hbmemvar.ch"
memvar City

function Demo()
    private City := "MemvarCity"
    select Customer
    ? M->City, Customer->City
    return NIL
```

This avoids ambiguity when identifiers overlap.

---

### `@` operator: pass-by-reference vs function reference

Harbour uses `@` in more than one way. The meaning is **context-dependent**.

#### A) Pass-by-reference (mutate caller's variable)

The `@` operator can pass a variable **by reference**, allowing the callee to modify the caller’s variable.

```harbour
function Inc( par_n )
    par_n++
    return NIL

function Demo()
    local l_n := 1
    Inc( @l_n )
    ? l_n   // 2
    return NIL
```

**Pass-by-reference requires an addressable l-value.** You cannot pass by reference:
- literals (e.g. `@1`)
- function call results (e.g. `@Foo()` when `Foo()` is being *called* to produce a value)
- expressions / temporaries (e.g. `@(x + y)`)

Reason: these do not have a stable memory location that can be referenced.

#### B) Function reference / callback (create an executable reference)

In callback-style APIs, `@` can be used to pass a **reference to a function** (often as a codeblock-like callable), which can be executed by the callee.

Typical evaluation is done with `Eval()` (or equivalent helper). Some environments/wrappers may expose an object-style method such as `:Exec()` / `:exec()` to run it.

```harbour
function Foo()
    ? "Foo ran"
    return NIL

function RunIt( par_cb )
    // Common Harbour form:
    Eval( par_cb )
    // In some wrappers, you may see:
    // par_cb:exec()
    return NIL

function Demo()
    RunIt( @Foo() )   // pass a callable reference to Foo (callback)
    return NIL
```

Important distinction:
- In **(A)** `@` is about mutating a caller variable.
- In **(B)** `@` is about passing an executable reference/callback.

---

### Arrays, hashes, and objects (reference semantics)

Arrays, hashes, and objects are **passed by reference by default**.

```harbour
function Modify( a_Data )
    a_Data[1] := 99
    return NIL

function Demo()
    local a := { 1, 2, 3 }
    Modify( a )
    ? a[1]   // 99
    return NIL
```

Implications:
- Functions can mutate passed arrays and objects.
- No `@` is required for element/property modification.
- This has significant design and safety implications and must be intentional.

---

### Compiler flags (strongly recommended)

These are **Harbour compiler flags** that control diagnostics and error handling:

- `-w3`  
  Enables high-level warnings (warning level 3), including warnings for use of undeclared variables.
- `-es2`  
  Treats selected warnings as compilation errors (error severity level 2).

Using `-w3 -es2` prevents silent creation of unintended `PRIVATE` variables and enforces explicit declarations.

---

## 7) Preprocessor (HbPP)

Harbour uses a powerful **textual preprocessor** (HbPP) that operates **before compilation**.
It performs **token-based source transformation** and conditional inclusion.
The preprocessor has **no runtime awareness** and does not understand types, variables, scope, or values.

> Fundamental rule: **the preprocessor runs before the compiler and only sees text/tokens**.

---

### 7.1 Core characteristics

* Textual and token-based (not semantic)
* Case-sensitive
* Executes before parsing and compilation
* Independent from runtime symbols and variables
* Can be used:
  * implicitly during compilation
  * explicitly at runtime via the `__PP_*()` API

The preprocessor does **not** interact with:
- runtime variables (LOCAL / STATIC / PRIVATE / PUBLIC)
- functions or methods
- types or values
- `SET` options
- execution flow

---

### 7.2 Preprocessor directives overview

```harbour
#include "file.ch"

#define NAME value
#undef  NAME

#translate  pattern => replacement
#xtranslate pattern => replacement

#command    pattern => replacement
#xcommand   pattern => replacement

#if expr
#elif expr
#else
#endif

#ifdef  NAME
#ifndef NAME

#pragma BEGINDUMP
// C code
#pragma ENDDUMP
```

---

### 7.3 Case sensitivity and namespaces

Preprocessor identifiers are **case-sensitive** and live in a namespace that is
**completely separate** from runtime identifiers.

```harbour
#define MyValue  1
#define MYVALUE  2   // different symbol

local myvalue := 10   // runtime variable, unrelated
```

| Aspect | Preprocessor | Runtime |
|------|--------------|---------|
| Case sensitivity | Sensitive | Insensitive |
| Symbol length limit | None | First 63 chars |
| Scope | Global after inclusion | Lexical / runtime |
| Type awareness | None | Dynamic |

---

### 7.4 `#define` vs `#translate` vs `#command`

These directives serve **different purposes** and are not interchangeable.

#### `#define`
* Simple token substitution
* No parameters
* No pattern matching
* **No `#xdefine` variant exists**

```harbour
#define APP_NAME "Demo"
```

---

#### `#translate` / `#xtranslate`
* Rewrite **expressions**
* Support parameterized patterns
* Used to normalize or adapt expression syntax

```harbour
#translate NVL(<v1>,<v2>) => hb_defaultValue(<v1>,<v2>)
```

---

#### `#command` / `#xcommand`
* Rewrite **command-style statements**
* Operate on whole lines
* Often used to implement declarative or DSL-like syntax

```harbour
#command USE <cFile> SHARED => dbUseArea( .t.,, <cFile> )
```

#### 7.4.1 Advanced `#command` use: redefining pseudo-commands (guardrails)

`#command` can redefine *command-style* syntax, including built-in pseudo-commands like `?`.
This is occasionally useful in **build/debug tooling** (e.g., to silence output by mapping `?` to nothing), but it has sharp edges.

Rules for AI-generated code:
- Do **not** redefine `?` / `??` / `@ ... SAY` / other ubiquitous constructs in application code.
- Only generate such overrides when explicitly writing a **debugger**, **logger shim**, or **compile-time instrumentation** layer.
- If you must, isolate the override behind a feature flag (e.g., `#ifdef _DEBUGDEBUG`) and keep it in a single dedicated module.

Typical pattern (tooling only):
```harbour
#ifdef _DEBUGDEBUG
    #command ? [<explist,...>] => dbgQOut( <explist> )
#else
    #command ? [<explist,...>] =>
#endif
```

---

### 7.5 Meaning of the `#x` prefix

The `#x` prefix exists **only** for:

- `#xtranslate`
- `#xcommand`

It enables **extended match markers**, allowing:

- optional clauses
- list matching
- wild token ranges
- restricted and expression-aware matching

There is **no** `#xdefine`.

> The `#x` prefix expands the **pattern grammar only**.
> It does not affect evaluation timing or semantics.

---

### 7.5.1 Pattern: overloading by arity with `#xtranslate`

A common HbPP idiom is to provide **two translations**: one “no-arg” form, and one form that takes an argument.
This feels like *overloading*, but it is purely **pattern matching**.

Example (used in the samples repo) to get a DBF full-path, optionally for a specific alias:

```harbour
#include "dbinfo.ch"

#xtranslate el_dbf()         => DBINFO( DBI_FULLPATH )
#xtranslate el_dbf(<xAlias>) => (<xAlias>)->( DBINFO( DBI_FULLPATH ) )
```

Notes:
- `<xAlias>` is a *match marker*. HbPP substitutes it verbatim into the output.
- `(<xAlias>)->( ... )` forces evaluation in that workarea context.
- Use this approach for concise wrappers, but keep semantics obvious; do not hide complex logic behind extensive preprocessor rewrites.

---

### 7.6 Match markers and placeholders

In `#translate` and `#command`, placeholders are **match markers**, not variables.

They:
- capture tokens
- have no scope or type
- are substituted verbatim into the result

| Marker form | Meaning |
|------------|--------|
| `<x>` | single token or expression |
| `<x,...>` | **comma-separated list** |
| `<x:Name>` | restricted token |
| `<*x*>` | wild token sequence |
| `<(x)>` | extended expression |

Example:

```harbour
#translate SUM(<a,...>) => hb_ArrayEval( { <a> }, {|x| x }, 0 )
```

The `...` syntax is **HbPP-specific**, not a Harbour language feature.

---

### 7.7 Whitespace and line continuation

Preprocessor rules are **whitespace-sensitive**.

```harbour
#translate FOO(<x>) => Bar(<x>)

FOO(1)     // matches
FOO (1)    // may not match
```

Line continuation using `;` occurs **before preprocessing**.

---

### 7.8 Conditional compilation

Supported directives:

- `#if`, `#elif`, `#else`, `#endif`
- `#ifdef`, `#ifndef`

Rules:

* Conditions operate **only on preprocessor symbols**
* When a condition is false:
  - all content is ignored
  - except nested conditional directives
* Directives are stack-based and must balance

```harbour
#ifdef DEBUG
   ? "debug build"
#elif defined(RELEASE)
   ? "release build"
#else
   ? "unknown build"
#endif
```

---

### 7.9 Defining symbols via `hbmk2 -d`

Preprocessor symbols may be defined at **build time** using the `hbmk2` `-d` option.

```text
hbmk2 myapp.prg -dDEBUGVIEW
```

This is equivalent to:

```harbour
#define DEBUGVIEW
```

Usage:

```harbour
#ifdef DEBUGVIEW
   ? "Debug view enabled"
#endif
```

Notes:

- The symbol has **presence only** (no value)
- It applies to **all files** in the build
- Commonly used for:
  - debug vs release builds
  - feature toggles
  - environment flags

Multiple symbols may be defined:

```text
hbmk2 myapp.prg -dDEBUG -dTRACE -dEXPERIMENTAL
```

---

### 7.10 Macro expansion and recursion

* Expansion is recursive
* HbPP does **not** detect infinite recursion

```harbour
#define A B
#define B A   // infinite expansion
```

Avoid chained macro definitions.

---

### 7.11 `#pragma BEGINDUMP / ENDDUMP`

These embed **raw C code** into the generated output.

```harbour
#pragma BEGINDUMP
#include <stdio.h>
void hello() { printf("Hello\n"); }
#pragma ENDDUMP
```

Rules:

- No Harbour syntax allowed
- Errors are C compiler errors
- No runtime context exists inside the block

#### 7.11.1 Compiler-switch pragmas (tooling)

HbPP also accepts pragmas that pass **compiler switches** as if they were given on the `harbour`/`hbmk2` command line.
These are typically used by tooling, debuggers, or portability shims.

Example: the `B` switch is commonly used to control **debug info / debugger support**. Some codebases explicitly disable it in special modules to avoid runtime/debugger edge cases:

```harbour
// Tooling-only: forces a compiler switch.
// (Some debugger helpers require this to avoid stack issues when debug info is present.)
#pragma -B-
```

Guidance:
- Treat switch pragmas as **build-tool configuration**, not normal application code.
- Only generate them when the user explicitly requests build/debug integration.

---

### 7.12 `.ch` header files

`.ch` files are **preprocessor-only** and typically contain:

- `#define`
- `#translate`
- `#command`

They should **not** contain executable Harbour code.

---

### 7.13 Runtime preprocessor API (advanced)

The preprocessor can be used programmatically:

- `__PP_INIT()`
- `__PP_PATH()`
- `__PP_RESET()`
- `__PP_ADDRULE()`
- `__PP_PROCESS()`

Properties:

- Multiple contexts can exist simultaneously
- Safe for multi-threaded use
- Intended for code generation and templating

---

### 7.14 Small DSLs (Domain-Specific Languages)

**DSL** stands for **Domain-Specific Language**.

In Harbour preprocessing, a *small DSL* means a **restricted, declarative syntax**
implemented using `#command` or `#translate`, expanded entirely at preprocessing time.

Characteristics:

- No logic
- No runtime dependency
- Pure syntactic transformation
- Expands into ordinary Harbour code

Example:

```harbour
#command LOG <cMsg> => Logger( <cMsg>, __FILE__, __LINE__ )

LOG "Starting process"
```

---

### 7.15 What the preprocessor cannot do

The preprocessor cannot:

- inspect runtime values
- test types
- execute functions
- loop
- perform arithmetic
- access variables

If behavior depends on runtime state, it must be written in Harbour code.

---

### 7.16 Best-practice guidance

Use the preprocessor for:

- constants
- build-time configuration
- compatibility shims
- small DSLs (as defined above)

Avoid using it for:

- logic
- decisions
- flow control
- runtime behavior

> If it depends on runtime state, it does not belong in the preprocessor.

---

## 8) Routines & modules 

Harbour programs are composed of **routines** (FUNCTION / PROCEDURE) grouped into **modules**, where one module corresponds to one `.prg` file.

This section defines routine semantics, visibility rules, parameter behavior, and how Harbour resolves routines across modules.

---

### 8.1 FUNCTION vs PROCEDURE

Harbour distinguishes between routines that conceptually *return a value* and those that do not.

#### FUNCTION
- Intended to return a value.
- May explicitly `return <expr>`.
- If execution reaches the end without `return`, **NIL is returned implicitly**.
- Returning `NIL` is legal but usually discouraged unless semantically meaningful.

```harbour
function Greeter(par_cName)
return "Hello "+par_cName
```

#### PROCEDURE
- Intended for side effects.
- `return` may be omitted.
- Always returns **NIL** implicitly.

```harbour
procedure SayHello(par_cName)
? "Hello "+par_cName
```

> Rule: The distinction is semantic and stylistic; at runtime both are callable routines.

---

### 8.2 Entry points and program startup

Harbour has **no language-level mandatory entry point**.

- `main()` / `Main()` is a **convention**, not a keyword.
- Any FUNCTION or PROCEDURE may act as the entry point if referenced by the build system.
- With `hbmk2`, the startup routine name can be overridden using `-main=<mainfunc>` (tooling behavior).

```harbour
function Main()
? "Program started"
return nil
```

---

### 8.3 Modules (PRG files) and symbol visibility

- One `.prg` file = one module.
- All non-STATIC FUNCTION/PROCEDURE symbols are **global across all PRGs** in the build.
- There is **no namespace system** beyond module scoping.

Symbol resolution rules:
- Case-insensitive
- First **63 characters** are significant (see §2)

---

### 8.4 STATIC FUNCTION / PROCEDURE (module-private routines)

Declaring a routine as `static` restricts its visibility to the **current PRG file only**.

```harbour
static function NormalizeName(par_cName)
return upper(alltrim(par_cName))
```

Properties:
- Not callable from other PRGs
- Still compiled normally
- Visibility is enforced at compile/link time
- `static` affects **visibility**, not lifetime

Design guideline:
- Use `static function` for helpers and internal utilities
- Use non-STATIC routines only for intentional APIs or entry points

Practical note:
- Prefer placing `static` helpers at the top of the PRG (before public/exported routines) to avoid link-time resolution edge cases.

---

### 8.5 Parameters, arity, and argument count

Terminology:
- **Arity** = the number of parameters declared by a routine (e.g., `function F(a,b)` has arity 2).

Parameter model:
- Parameters are **dynamically typed**.
- Each parameter is an implicit **local variable** within the routine.

Argument count behavior:
- If **fewer** arguments are passed than declared parameters, the missing parameters are set to `nil`.
- If **more** arguments are passed than declared parameters, the extra arguments are still available via `pcount()` and `hb_PValue(<n>)`.

Example (accepting extra arguments):

```harbour
function TestParameters()
local l_nNumberOfParameter := pcount()
local l_nLoop

for l_nLoop := 1 to l_nNumberOfParameter
? l_nLoop,hb_PValue(l_nLoop)
endfor

return pcount()
```

Notes:
- Declared parameters (if any) receive the first N arguments.
- `pcount()` returns the number of arguments actually passed.
- `hb_PValue(1)` is the first passed argument, `hb_PValue(2)` the second, etc.

---

### 8.6 Parameter passing semantics

Default behavior:
- Parameters are passed **by value**.

Reference semantics apply automatically for:
- Arrays
- Hashes
- Objects

#### Explicit pass-by-reference (`@`)

The `@` operator passes a variable **by reference**, allowing mutation of the caller’s variable.

You cannot pass by reference:
- literals
- expressions
- function call results
- temporaries

---

### 8.7 Return values and implicit NIL

- Reaching the end of a routine without `return` yields **nil**.
- This applies to both FUNCTION and PROCEDURE.
- Use explicit `return nil` when absence of a value is intentional.

---

### 8.8 Call order and forward references

- Routines may call routines defined later in the same PRG or in other PRGs.
- Textual order usually does not matter; no forward declarations are required.

Practical caveat (STATIC helpers):
- If a routine is declared `static` and its first **definition** appears after a call site, some Harbour versions/build pipelines may compile the call as a public reference and then fail at link time (hbmk2: “Referenced, missing, but unknown function(s)”).
- To avoid this, define `static function` / `static procedure` helpers **before** their first use within the PRG, or make the helper non-STATIC if it must be referenced from other PRGs.

Example (recommended layout):
```harbour
static function py_IsValidPythonTopLevelName(par_cName)
...
return .t.

function py_GetAvailablePythonPackages(par_lIncludeMetadata)
...
if py_IsValidPythonTopLevelName("abc")
...
endif
return {}
```

---

### 8.9 “Variadic” routines

A **variadic** routine is one that can accept a variable number of arguments.

Harbour does not declare variadic parameters using a special syntax (like C `...`), but a routine can still **accept and process any number of passed arguments** using `pcount()` and `hb_PValue(<n>)` (see 8.5).

---

### 8.10 No overloading or dispatch by type

Harbour does not support function/procedure overloading.

Rules:
- One name → one routine
- Parameter types do not affect dispatch
- Duplicate names (within the 63-character significance rule) are compile-time errors

---

### 8.11 Recursion and STATIC pitfalls

- Recursion is fully supported.
- `local` variables are stack-scoped and safe.
- `static` variables are shared across all calls, including recursive ones.

Guideline: Avoid `static` variables in recursive routines unless explicitly intended.

---

### 8.12 Interaction with OOP methods (cross-reference)

- Methods are not linkable across PRGs.
- All method implementations must be in the same PRG as the class declaration.
- See §13 Objects & classes (modern Harbour).

---

### 8.13 Summary (AI-safe rules)

- FUNCTION and PROCEDURE differ semantically, not mechanically
- `Main()` is a convention; `hbmk2 -main=<mainfunc>` can override the startup routine name
- One PRG = one module; `static` routines are file-private
- Parameters are dynamic; missing parameters become `nil`
- Extra passed arguments are accessible via `pcount()` and `hb_PValue(<n>)`
- Arrays/hashes/objects are passed by reference
- `@` enables explicit pass-by-reference
- No overloading exists
- Call order does not depend on textual position

---

## 9) Control flow

This section defines Harbour control-flow constructs and their exact semantics. Correct understanding of evaluation order, termination rules, and scope of effects is critical for both human-written and AI-generated code.

---

### General rules

* Control-flow keywords are case-insensitive, but should be written in lower case **per project style standards**.
* Block structure is explicit and lexical; indentation is cosmetic only.
* Prefer explicit terminators (`endif`, `endcase`, `endswitch`, etc.) over the generic `end`.
* Conditions should evaluate to logical values (`.t.` / `.f.`). Reliance on implicit truthiness is discouraged.
* There is no implicit fall-through unless explicitly stated.

---

### IF / ELSEIF / ELSE / ENDIF

```harbour
if l_lCondition1
    DoSomething()
elseif l_lCondition2
    DoSomethingElse()
else
    DoDefault()
endif
```

Semantics:

* Conditions are evaluated top-down.
* `elseif` is evaluated only if all previous conditions failed.
* The first matching branch executes; all others are skipped.
* `else` is optional but recommended.
* Nested `if` blocks are acceptable when they reflect logical structure.

---

### Expression-level conditional: `iif()` / `if()`

Harbour provides `iif(<lCondition>, <expTrue>, <expFalse>)` for expression-level conditional selection (and also provides `if()` as an equivalent function name in xBase/Clipper tradition). Use these when you need an inline conditional value inside an expression (assignment, parameter, array/hash literal, etc.). They are functions, not control-flow statements.

```harbour
local l_lPaid  := .t.
local l_cLabel := iif( l_lPaid, "Paid", "Unpaid" )
? l_cLabel
```

Semantics:

* `<lCondition>` is evaluated first.
* Exactly one branch expression is evaluated and returned:

  * If `<lCondition>` is `.t.`, `<expTrue>` is evaluated and returned.
  * If `<lCondition>` is `.f.`, `<expFalse>` is evaluated and returned.
* The non-selected branch is not evaluated (short-circuit behavior).
* The return type is the type of the selected branch; the two branches do not need to share a type.

Practical guardrails:

* Prefer `iif()` only for simple value selection; for multi-step logic or side effects, use `if/elseif/else`.
* Avoid embedding assignments, I/O, or other side effects inside `iif()` branches.

Short-circuit safety example (avoid divide-by-zero):

```harbour
local l_nNum := 10
local l_nDen := 0

local l_nResult := iif( l_nDen == 0, 0, l_nNum / l_nDen )
? l_nResult
```

---

### DO CASE / CASE / OTHERWISE / ENDCASE

```harbour
do case
case dow(Date()) == 2
    ? "Monday"
case dow(Date()) == 1 .or. dow(Date()) == 7
    ? "Weekend"
otherwise
    ? "Tue–Fri"
endcase
```

Semantics:

* Each `case` is an independent logical expression.
* Conditions are evaluated top-down.
* The first matching case executes; evaluation stops.
* There is no fall-through.
* `otherwise` is optional but strongly recommended.
* Prefer `do case` for complex conditions.

---

### SWITCH / CASE / EXIT / OTHERWISE / ENDSWITCH

```harbour
switch dow(Date())
case 2
    ? "Monday"
    exit
case 1
case 7
    ? "Weekend"
    exit
otherwise
    ? "Tue–Fri"
endswitch
```

Semantics:

* The switch expression is evaluated once.
* Each `case` label is compared using `==` semantics.
* Multiple `case` labels may stack.
* `exit` is required to stop evaluation.
* Without `exit`, execution continues into the next case.
* `otherwise` is optional but recommended.
* Prefer `switch` for discrete value comparisons.

---

### Truthiness and coercion

Harbour allows non-logical values in conditions, but implicit coercion is error-prone.

Rules:

* `NIL` → `.f.`
* `0` → `.f.`
* Non-zero numeric → `.t.`
* Empty string → `.f.`

Best practice: always generate explicit logical expressions.

---

### EXIT vs RETURN

* `exit` leaves the current control structure.
* `return` exits the current routine.

```harbour
for l_nIndex := 1 to 10
    if l_nIndex == 5
        exit
    endif
endfor
```

---

### Discouraged patterns (AI guardrails)

* Do not use bare `end`.
* Do not rely on `$` for numeric membership.
* Do not omit `exit` in `switch`.
* Do not rely on implicit truthiness.
* Do not simulate C-style fall-through.
* Do not embed assignments in conditions unless intentional.

---

## 10) Loops

This section documents Harbour’s loop constructs and the project’s preferred usage patterns for GPT-generated code.

### 10.1 General rules (AI guardrails)

- Harbour is **case-insensitive** for keywords. For consistency in this project, prefer emitting keywords in **lower case** (`for`, `next`, `endfor`, `exit`, `loop`, `do while`, `enddo`, ...).
- Prefer explicit terminators (`endfor`, `enddo`, ...) when available via Harbour_EL, but `next` / `enddo` are also acceptable in core syntax.
- Use `exit` to leave the **current control structure** (loop / case / switch). Use `return` only to exit the **current routine**.

---

### 10.2 `for ... next` / `for ... endfor`

Notes for AI:

- `for` counters may be **numeric**, **date**, or **timestamp**.
- For date/timestamp counters, `step` is expressed in **days** (fractional days allowed).

Example (preferred style):

```harbour
local l_nIndex

for l_nIndex := 1 to 10 step 2
    if l_nIndex > 6
        exit
    endif

    loop    // jump to next iteration of this for loop
next
```

Project preference:

- If Harbour_EL is in use, prefer emitting `endfor` over `next` for readability (both are accepted, but be consistent).

---

### 10.3 `for each` (enumeration)

`for each` can iterate:

- arrays
- hashes
- **bytes** of a string (not UTF-aware; string enumeration is byte-by-byte)

Multiple parallel enumerators are supported; iteration ends when the shortest enumerated sequence ends.

Enumerator meta fields exposed on each enumerator item:

- `:__enumindex` → 1-based index / position
- `:__enumkey` → hash key (or `NIL` for arrays/strings)
- `:__enumbase` → underlying collection
- `:__enumvalue` → current value


#### Multiple variables and expressions

`for each` supports **parallel enumeration** using multiple variables and expressions:

```harbour
for each l_xValue1, l_xValue2 in a_Array1, a_Array2
    ? l_xValue1, l_xValue2
endfor
```

Rules and notes for AI:

- The number of variables **must match** the number of expressions.
- Each variable is assigned the next value from its corresponding expression.
- Iteration count is limited by the **shortest** enumerated expression.
- Arrays, hashes, and strings are supported.
- When iterating strings, enumeration is **byte-based**, not UTF-aware.

#### DESCEND modifier

If `descend` is specified, iteration starts from the end of each expression:

```harbour
for each l_xValue in a_Array descend
    ? l_xValue
endfor
```

Example (hash enumeration):

```harbour
local l_h := {=>}
local l_xItem

hb_HKeepOrder(l_h, .t.)
l_h["City"] := "Seattle"
l_h["State"] := "WA"

for each l_xItem in l_h
    ? l_xItem:__enumkey, l_xItem:__enumvalue
endfor
```

UTF-8 note for AI:

- If you need **character-aware** traversal of UTF-8 strings, do not use `for each` over the string directly; use UTF-8 helper functions or a UTF8EX-based approach instead.

---

### 10.4 `do while ... enddo`

`do while` evaluates its condition at the top of each iteration.

Example:

```harbour
local l_nCount := 7

do while l_nCount > 0
    l_nCount -= 1
    ? l_nCount
enddo
```

---

### 10.5 `exit` and `loop` semantics inside loops

Inside any loop construct:

- `exit` leaves the loop immediately.
- `loop` skips the remainder of the current iteration and continues at the loop’s next iteration.

For AI-generated code:

- Prefer `exit` (with a comment explaining the stop condition) over complex boolean conditions when it improves clarity.
- Avoid embedding assignments in conditions unless it is deliberately used and clearly commented.

---

### 10.6 `scan ... endscan` (Harbour_ORM extension)

`scan ... endscan` is **not a core Harbour keyword**. In WharfSystems/DataWharf-style codebases that include **Harbour_ORM**, `scan ... endscan` is provided as a convenience loop for iterating ORM-backed cursors/workareas in an xBase-like way.

What AI should assume (safe defaults):

- `scan ... endscan` iterates records/rows in the **current ORM cursor/workarea**.
- The loop body runs once per row.
- `exit` and `loop` behave as expected (leave scan / continue scan).
- If you need deterministic ordering, ensure the underlying query/cursor is created with an explicit order (SQL `ORDER BY` or ORM ordering), rather than assuming physical order.

Example (conceptual):

```harbour
// Harbour_ORM provides scan/endscan as an xBase-like row loop over an ORM cursor/workarea.
scan
    if Empty(Field->Status)
        loop
    endif

    ? Field->Id, Field->Name
endscan
```

If you are generating code and Harbour_ORM is **not** explicitly in scope, do **not** emit `scan ... endscan`. Use `for` / `for each` / `do while` instead.

---
## 11) Arrays & hashes (semantics that matter)

This section documents **non-obvious but critical runtime semantics** of arrays and hashes in Harbour.
These rules are essential for both human authors and AI-generated code, because violations often lead to **silent data corruption** rather than explicit errors.

---

### 11.1 Reference semantics (core rule)

Arrays, hashes, and objects are **always reference types** in Harbour.

* Passing them as parameters passes the **reference**
* Assigning them to another variable copies the **reference**
* No implicit copy is ever performed

```harbour
local l_a1 := { 1, 2, 3 }
local l_a2 := l_a1      // same reference

l_a2[1] := 99
? l_a1[1]               // 99
```

**Rule:**
If two variables refer to the same array or hash, **mutating through either name mutates the same underlying structure**.

---

### 11.2 Assignment vs rebinding (aliasing hazards)

It is critical to distinguish between:

* **Mutating a referenced structure**
* **Rebinding a variable to a new structure**

```harbour
local l_a := { 1, 2, 3 }
local l_b := l_a

l_b[1] := 99
? l_a[1]      // 99  (mutation through alias)

l_b := { 7, 8, 9 }
? l_a[1]      // 99  (rebinding does not affect original)
```

**Key distinction:**

* `l_b[1] := ...` mutates shared data
* `l_b := ...` only changes what `l_b` points to

---

### 11.3 Array indexing rules (1-based, no negative indices)

Harbour arrays are **strictly 1-based**.

```harbour
local l_a := { "a", "b", "c" }

? l_a[1]          // "a"
? l_a[Len(l_a)]   // "c"
```

Rules:

* The **first element** is at index `1`
* The **last element** is at index `Len( array )`
* **Negative indices are not supported**

```harbour
// INVALID (unlike Python)
l_a[-1]           // runtime error
```

**AI rule:**
Never emit negative indices. Always compute the index explicitly using `Len()`.

---

### 11.4 Copying arrays: `ACopy()` vs `AClone()`

Harbour never performs deep copies automatically.
Copying must be explicit and intentional.

```harbour
local l_a1 := { 1, 2, { "x" } }
local l_a2 := ACopy( l_a1 )
local l_a3 := AClone( l_a1 )
```

| Element inside array                | `ACopy()`  | `AClone()` |
| ----------------------------------- | ---------- | ---------- |
| Numeric / String / Date / Timestamp | copied     | copied     |
| Nested array                        | **shared** | **cloned** |
| Hash                                | shared     | shared     |
| Object                              | shared     | shared     |
| Codeblock                           | shared     | shared     |

**Important:**

* `ACopy()` copies **only the first dimension**
* `AClone()` recursively clones **arrays only**
* Hashes and objects are **never deep-copied**

There is **no universal deep-copy operation** in Harbour.

---

### 11.5 Hashes (associative maps)

Hashes are associative containers mapping **scalar keys** to values.

#### Creating hashes

```harbour
local l_h := {=>}          // empty hash
// or
local l_h := hb_Hash()
```

Valid key types:

* String (`"C"`)
* Numeric (`"N"`)
* Date (`"D"`)
* Timestamp (`"T"`)

```harbour
l_h["Name"] := "Eric"
l_h[1241]   := "Seattle"
```

**Invalid keys (runtime error):**

* arrays
* hashes
* objects
* codeblocks

```harbour
// INVALID
l_h[ { 1, 2, 3 } ] := "x"
```

---

### 11.6 Hash cloning: `hb_HClone()`

To create a **copy of a hash table**, use:

```harbour
local l_h2 := hb_HClone( l_h1 )
```

Based on practical behavior, `hb_HClone()` should be treated as a **deep clone of hashes**:

- Creates a **new hash**
- Copies **keys and values**
- Nested **hash values are cloned** (so changes inside nested hashes do not affect the original)
- Arrays and objects stored as values remain **shared references** unless explicitly cloned

If you need full isolation for nested arrays inside a hash, clone them explicitly (for example with `AClone()`) before inserting or after retrieving.

---

### 11.7 Hash key equality is strict and type-sensitive

Hash keys are compared with **strict equality**:

* No type coercion
* No trimming
* No case folding

```harbour
local l_h := {=>}
l_h["1"] := "string key"
l_h[1]   := "numeric key"

? l_h["1"]    // "string key"
? l_h[1]      // "numeric key"
```

Rules:

* `"1"` ≠ `1`
* `"A"` ≠ `"A "`
* `"a"` ≠ `"A"`

---

### 11.8 Hash values are references too

Values stored in hashes are **not copied**.

```harbour
local l_h := {=>}
l_h["Items"] := { 1, 2, 3 }

local l_a := l_h["Items"]
l_a[1] := 99

? l_h["Items"][1]    // 99
```

If isolation is required, clone explicitly **before** storing or retrieving.

---

### 11.9 Iteration order of hashes

Hash iteration order is **undefined by default**.

If iteration order matters, it must be explicitly enabled:

```harbour
hb_HKeepOrder( l_h, .t. )
```

**AI rule:**

* Never rely on hash insertion order unless `hb_HKeepOrder()` is used
* Otherwise, iteration order must be treated as arbitrary

---

### 11.10 `for each` exposes live values

`for each` does **not** iterate over copies.

```harbour
for each l_x in l_a
    if ValType( l_x ) == "A"
        l_x[1] := 99
    endif
endfor
```

Rules:

* Enumerator variables reference the underlying value
* Mutating them mutates the source collection

---

### 11.11 Array growth and structure rules

Arrays are:

* **1-based**
* **contiguous**
* **non-sparse**

Preferred append method:

```harbour
AAdd( l_a, value )
```

Rules:

* Do not assign beyond `Len()+1`
* Do not assume sparse array behaviour
* Always validate indices explicitly

---

### 11.12 Objects vs arrays and hashes

Unlike arrays and hashes:

* There is **no concept of an “empty object”**
* Objects must be instantiated from a **specific class**

```harbour
// INVALID
local l_o := {}        // not an object

// VALID
local l_o := MyClass():New()
```

Objects always have:

* A class definition
* A fixed slot/method layout
* Reference semantics identical to arrays and hashes

---

### 11.13 Debugging vs serialization

For inspection and debugging only:

```harbour
? hb_jsonEncode( l_a )
```

Limitations:

* Output is JSON-like, not Harbour syntax
* Type fidelity is not preserved
* Order may differ
* Not reversible

**AI rule:**

* Never emit JSON output as Harbour code
* Never attempt to reconstruct arrays or hashes from JSON

---

### 11.14 AI-critical rules (summary)

* Arrays, hashes, and objects are always **shared by reference**
* Arrays are **1-based**; negative indices are invalid
* Assignment copies references, never data
* `AClone()` deep-copies **arrays only**
* `hb_HClone()` deep-clones hashes (nested hashes are duplicated)
* Hash keys are strictly typed (`C/N/D/T`)
* Hash iteration order is undefined unless explicitly preserved
* `for each` exposes live values, not copies
* Objects cannot be “empty”; they must belong to a class
* `hb_jsonEncode()` is diagnostic only

---

## 12) Code blocks (lambdas)

Code blocks are anonymous executable units that can be passed as values and executed later via `Eval()`. They are heavily used for callbacks, predicates, transformations, and deferred execution.

---

### 12.1 Syntax forms

Two forms exist:

#### 1) Single-line code block

* Returns the value of the last expression automatically.
* Intended for simple expressions only.

```harbour
local l_cb := {|x| x * x }
? Eval( l_cb, 7 )   // 49
```

#### 2) Extended (multi-line) code block

* May declare local variables.
* May use control flow (`if`, `do case`, loops, etc.).
* **Must use an explicit `return`.**

```harbour
local l_cb := {|x|
    local l_nResult
    l_nResult := x * x
    return l_nResult
}
```

As a project rule, multi-line code blocks should always include an explicit `return` to avoid ambiguity and improve readability.

---

### 12.2 Execution model

* A code block does nothing by itself.
* Execution occurs only when passed to `Eval()` (or to an API that internally evaluates it).

```harbour
Eval( l_cb, 10 )
```

The parameters passed to `Eval()` are mapped positionally to the parameters declared in the code block header.

---

### 12.3 Parameter and local scope

* Parameters declared in `{| ... |}` behave like routine parameters.
* `local` variables declared inside the block exist only for the duration of that block execution.
* Each call to `Eval()` creates a new execution context for the block.

---

### 12.4 Access to outer scope variables (closure behavior)

A code block may reference identifiers that are **not** declared as block parameters or locals.

Resolution rules are as follows:

* **Locals from the routine that created the block are captured.**
  Harbour code blocks implement true closure semantics for locals: locals referenced by a block are detached from the creator routine’s stack so they remain accessible even after that routine has returned.

* Static variables in the PRG/module are resolved normally.

* `private` / `public` variables (memvars) are resolved dynamically at evaluation time, following the runtime call chain.

* Workarea fields may be accessed when explicitly qualified.

Important implications:

* Locals captured by a block belong to the **creating routine**, not the routine that evaluates the block.
* Captured locals are **referenced**, not copied.
* The value observed depends on when the block is executed.
* Memvars behave differently from locals and must not be confused with closure capture.

Because of this, code blocks that capture external state should be kept short-lived and intentional unless closure semantics are explicitly desired.

---

### 12.5 Canonical closure example (creator-local capture)

The following example demonstrates, in a controlled and verifiable way, that code blocks capture **locals from the routine that creates them**, not from the routine that evaluates them.

```harbour
function Main()

    local l_cb
    local l_nCaptured

    l_cb := BuildBlock()

    l_nCaptured := 99   // local in the evaluating routine (Main)

    ? "Eval(l_cb) should return 10:"
    ? Eval( l_cb )

return NIL


static function BuildBlock()

    local l_nCaptured := 10

    // l_nCaptured is a LOCAL of BuildBlock().
    // This value is captured by the code block and remains accessible
    // even after BuildBlock() has returned.
    return {|| l_nCaptured }

return NIL
```

This example is authoritative: the result confirms closure capture of **creator-routine locals**.

---

### 12.6 Comparison of captured locals, static variables, and memvars

When code blocks reference identifiers outside their parameter list, it is essential to understand **how each category is bound and resolved**.

#### Captured locals (closure semantics)

* Origin: `local` variables from the routine that creates the block
* Binding time: Block creation
* Lifetime: Extended beyond creator routine execution
* Resolution: Direct reference
* Closure: Yes

Properties:

* Values remain accessible after the creator routine returns
* Values are not copied
* This is true lexical closure behavior

This is the most predictable and preferred form of outer-scope access.

---

#### Static variables

* Origin: `static` variables in the PRG/module
* Binding time: Load time
* Lifetime: Entire program execution
* Resolution: Lexical
* Closure: No

Properties:

* Shared across all callers and blocks
* Suitable for constants, caches, or controlled shared state
* Must be used intentionally due to global coupling

---

#### `private` / `public` variables (memvars)

* Origin: Dynamic variables
* Binding time: Evaluation time
* Lifetime: Depends on call chain
* Resolution: Dynamic (runtime stack)
* Closure: No

Properties:

* Code blocks do **not** capture memvars
* Value depends on who calls `Eval()` and when
* Highly error-prone if relied upon implicitly

---

#### Workarea fields

* Origin: Current or aliased workarea
* Resolution: Evaluation time
* Behavior: Context-dependent

Properties:

* Must be explicitly qualified inside code blocks
* Unqualified access can silently break if workarea changes

---

#### Summary table

| Variable kind    | Bound when | Resolved when | Closure | Predictability |
| ---------------- | ---------- | ------------- | ------- | -------------- |
| Captured `local` | Creation   | Evaluation    | Yes     | High           |
| `static`         | Load       | Evaluation    | No      | High           |
| `private/public` | Evaluation | Evaluation    | No      | Low            |
| Workarea field   | Evaluation | Evaluation    | No      | Medium         |

---

### 12.7 Variable shadowing

If a parameter or local inside a code block uses the same identifier as a variable in an outer scope, the inner definition takes precedence.

This is legal but error-prone and should be avoided unless explicitly required.

---

### 12.8 Common and idiomatic use cases

Code blocks are most appropriate for:

* Array helpers (`AEval()`, `AScan()`, `ASort()`)
* Predicates and filters
* Transformations
* Callbacks and hooks
* Behavior injection into helper routines

---

### 12.9 When *not* to use code blocks

Avoid code blocks for:

* Long-lived captured state
* Complex business logic
* Deeply nested control flow
* Implicit reliance on memvars

If reuse or complexity increases, a named routine is usually clearer.

---

### 12.10 Interaction with methods and objects

When created inside a method, a code block may reference object state depending on how it is written.

The block executes via `Eval()` as a standalone callable entity, not as a method invocation. Any object dependency should be explicit.

---

### 12.11 Alternative xHarbour-compatible syntax

When including `xhb.ch`, an alternative syntax is supported:

```harbour
<|a, b| return a + b>
```

Notes:

* Provided for xHarbour compatibility
* Modern Harbour fully supports multi-line `{| ... |}` blocks
* **Should not be used in new Harbour code**

---

### 12.12 Type identification

At runtime, code blocks have type `"B"`.

```harbour
if ValType( x ) == "B"
    Eval( x )
endif
```

---

### 12.13 Summary rules for AI-generated code

* Code blocks capture locals from the routine that creates them.
* Single-line blocks are for simple expressions only.
* Multi-line blocks must use explicit `return`.
* Prefer short-lived, localized blocks.
* Avoid implicit dependencies on dynamic scope.
* Do not use xHarbour-only syntax in new code.

---

## 13) Objects & classes (modern Harbour)

This section documents Harbour’s object system as implemented via `hbclass.ch`.  
All behaviors described here are **Harbour-specific** and must not be inferred from other OOP languages.

---

### 13.1 Enabling OOP

```harbour
#include "hbclass.ch"
```

All class-related syntax (`create class`, `method`, `data`, `setget`, etc.) is provided by the precompiler layer defined in `hbclass.ch`.

---

### 13.2 Object creation vs initialization

- Calling a class name as a function **creates an instance**:
  ```harbour
  local l_o := Person()
  ```
- **No initializer is invoked implicitly**.
- Initialization is **explicit** and entirely convention-based.

Common patterns:
```harbour
Person():New( ... )
Person():Init( ... )
```

`ClassName():New()` is simply *chaining* after creation. It is equivalent to:

```harbour
local l_o := Person()
l_o:New( ... )
```

Conventions:
- If you implement `New()`/`Init()`, prefer returning `self` (`return self`) so chaining is consistent.
- Do not assume `New()`/`Init()` ran just because an object exists; call them explicitly when required.

Rules:
- Objects may exist without `New()`, `Init()`, or any initializer.
- `init` expressions on `data` / `var` members always apply at instantiation time.
- Never assume a constructor is automatically executed.

---

### 13.3 `New()` / `Init()` convention

Many Harbour class hierarchies follow this *pattern* (not a rule):

```harbour
method New( ... )
    ::Init( ... )
return self
```

```harbour
method Init( ... )
    // override in subclasses
return self
```

Important:
- `New()` is an ordinary method.
- `Init()` is an ordinary method.
- Either may be overridden, bypassed, or omitted.
- `New()` has **no fixed or enforced semantics**.

---

### 13.4 `self` and `::`

Inside a method:

- `self` evaluates to the current object instance.
- `::Member` is shorthand for `self:Member`.
- `::Method()` is shorthand for `self:Method()` (self-send).

Guidance:
- Prefer `::` for clarity inside methods.
- Initializer-style methods commonly return self`.

---

### 13.5 Object identity and reference semantics

- Objects are **reference-like** values.
- Assignment copies the reference, not the state.

```harbour
local l_o1 := Person():New( "Bob", 50 )
local l_o2 := l_o1

l_o2:Age := 51
? l_o1:Age   // 51
```

Independent objects require **explicit construction or cloning**.

---

### 13.6 `data` / `var` and `classdata` / `classvar`

From observed behavior and documentation:

- `data` and `var` are **functionally equivalent instance slots**.
- `classdata` and `classvar` are **functionally equivalent class-wide slots**.

Guidance for this rulebook:
- Prefer **`data`** and **`classdata`** for semantic clarity:
  these members behave like **properties/slots**, not classic variables.
- Use `var` / `classvar` when matching an existing codebase convention.

---

### 13.7 Property-style member access patterns

Harbour supports multiple distinct “property” mechanisms.

#### Pattern A: Slot (`data` / `var`)
Direct storage in the object.

```harbour
create class LocationSlot
    data cCity init "Seattle"
endclass
```

#### Pattern B: Single-method property (`method ... setget`)
One method handles both read and write.

```harbour

static s_cCity

create class LocationSetGet
    method cCity() setget
endclass

method cCity( par_cValue ) class LocationSetGet
    if pcount() > 0
        s_cCity := par_cValue
    endif
return s_cCity
```

Assignment syntax:
```harbour
o:cCity := "Paris"
```

routes to the same method.

---

### 13.8 Inheritance, polymorphism, and `SUPER`

- Harbour supports **dynamic dispatch**.
- Single and multiple inheritance are allowed:
  ```harbour
  create class Child from ParentA, ParentB
  ```

Calling parent implementations:

```harbour
method Speak() class Dog
    ::super:Speak()
    ? "Woof"
return NIL
```

`SUPER` invokes the next method in the inheritance chain.

---

### 13.9 `with object` / `endwith`

`with object` temporarily sets an **implicit message target** for method calls and member access, reducing repetition.

```harbour
local l_oCar := Cars()

with object l_oCar
    :SetCarName( "Mustang" )
    :SetMaxSpeed( 320 )
    ? :CarToString()
endwith
```

Rules:
- `with object` is syntactic sugar; it does not change object lifetime or initialization.
- Avoid deep nesting or large `with object` blocks in complex code paths; explicit receivers are clearer for maintenance and for AI generation.
- Guard if the expression can be `NIL` or non-object (otherwise you will get a runtime error):

```harbour
if hb_IsObject( l_oCar )
    with object l_oCar
        :SetMaxSpeed( 320 )
    endwith
endif
```

---

### 13.10 Lifetime and cleanup

- Each class may define **at most one** `destructor`.
- The destructor **does execute** and is commonly used for cleanup.
- Execution timing depends on object lifetime.

Rules:
- Destructors are reliable for releasing resources.
- Do not rely on precise ordering or timing across multiple objects.

---

### 13.11 Method declaration attributes

Harbour supports method attributes in class declarations:

- `CONSTRUCTOR` – **documentation-only** marker; not enforced
- `VIRTUAL` – indicates intent to override (not required for dispatch)
- `SETGET` – enables unified property get/set behavior
- `INLINE` / `BLOCK` – inline method bodies
- `EXTERN <func>` – binds method to external function
- `OPERATOR <op>` – operator overloading
- `SYNC METHOD` – synchronization hint

Important:
- Attributes do **not** imply automatic behavior unless explicitly implemented by runtime code.

---

### 13.12 Method binding syntax

Methods may be implemented using either form:

```harbour
method ClassName:MethodName( ... )
```

or:

```harbour
method MethodName( ... ) class ClassName
```

The `class ClassName` clause is required only when multiple classes exist in the same PRG and the
method name would otherwise be ambiguous.

---

### 13.13 Visibility blocks and modifiers

Class members may be grouped under:

- `HIDDEN:` – private to class
- `PROTECTED:` – visible to class and subclasses
- `VISIBLE:` / `EXPORTED:` – public

Member modifiers:
- `READONLY` / `RO` – assignment inhibited

---

### 13.14 Friend declarations

Classes may grant privileged access:

```harbour
friend class OtherClass
friend function HelperFunc
```

This is distinct from `friendly` PRG-level access.

---

### 13.15 Object copying and cloning

- No implicit deep copy exists.

Shallow clone:
```harbour
__objClone( oObject )
```

Recommended:
- Implement explicit `Clone()` methods.
- Clone arrays (`AClone()`), hashes (`hb_HClone()`), and nested objects deliberately.
- Avoid calling `New()` inside `Clone()` unless side effects are intended.

---

### 13.16 File organization rule

For `hbclass.ch` classes:

- A class must be compiled as **one PRG unit**.
- You cannot spread class methods across independently compiled PRGs.

Allowed pattern:
```harbour
#include "Person_Methods.prg"
#include "Person_Extra.prg"
```

The compiler must see a single combined unit.

---

### 13.17 AI pitfalls (common incorrect assumptions)

The following assumptions are **incorrect** in Harbour:

1) `New()` is automatic  
2) `CONSTRUCTOR` is enforced  
3) Assignment copies objects  
4) Cloning is deep by default  
5) `o:Prop := x` always writes a slot  
6) Destructors are unreliable  
7) Visibility is compiler-enforced  
8) Harbour OOP behaves like C++ / Java / C#

Harbour OOP is **dynamic, message-based, and convention-driven**.

---

## 14) Symbols & pointers (advanced)

This section documents two advanced runtime mechanisms used primarily for
dynamic dispatch and C-level interoperability:

- **Symbol items**: first-class references to routines or messages.
- **Pointer items**: opaque handles to C memory or functions.

These mechanisms are powerful but low-level. They should be used deliberately
and sparingly. Normal function calls and codeblocks are preferred whenever
static structure is sufficient.

---

### 14.1 Symbols (routine references)

A **symbol item** is a runtime value representing a reference to a routine
(function or method). Symbols are executable objects and are distinct from
strings, macros, or codeblocks.

Symbols are most commonly created using the `@()` symbol literal syntax:

```harbour
function Add(par_nA, par_nB)
return par_nA + par_nB

function Main()
local l_sAdd := @Add()

? l_sAdd:name()
? l_sAdd:exec( 2, 3 )   // 5

return nil
```

Key characteristics:

* `@Add()` **does not call** `Add()`; it creates a symbol referencing it.
* The compiler emits a real symbol reference; this establishes an explicit
  binding to the target routine.
* Symbol items are evaluable runtime objects.

---

### 14.2 Symbol messages

Symbol items support at least the following standard messages:

* `:name()`
  Returns the name of the referenced routine as a string.

* `:exec( ... )`
  Executes the referenced routine with the supplied arguments.

Example:

```harbour
local l_s := @Upper()

? l_s:name()            // "UPPER"
? l_s:exec( "abc" )     // "ABC"
```

Notes:

* `:exec()` is preferred over indirect macro execution.
* Symbols can also be executed via `Eval( l_s, ... )`, but `:exec()` is clearer
  and explicit.

---

### 14.3 Symbols vs macros (binding matters)

There are two fundamentally different ways to obtain a symbol:

```harbour
local l_s1 := @Add()          // compile-time symbol literal
local l_s2 := &("@Add()")    // runtime macro expansion
```

Important distinctions:

* `@Add()` creates a **compile-time bound** symbol.
* `&("@Add()")` creates a symbol **at runtime**.
* Macro-created symbols do **not** guarantee the same linker visibility or
  dependency behavior.

Rule:

* Use `@()` whenever the target routine is known at authoring time.
* Use macro-generated symbols only when the name must be computed dynamically.

This distinction is critical for AI-generated code and for builds that rely on
explicit symbol references.

---

### 14.4 Symbols for dynamic dispatch (preferred pattern)

Symbols provide a safe, explicit alternative to macros for dynamic dispatch.

Canonical pattern:

```harbour
function Add(par_nA, par_nB)
return par_nA + par_nB

function Sub(par_nA, par_nB)
return par_nA - par_nB

function Main()
local l_hDispatch := {=>}

l_hDispatch[ "add" ] := @Add()
l_hDispatch[ "sub" ] := @Sub()

? l_hDispatch[ "add" ]:exec( 10, 7 )  // 17
? l_hDispatch[ "sub" ]:exec( 10, 7 )  // 3

return nil
```

Advantages:

* No macros
* Explicit bindings
* Easy to audit and reason about
* Safe for refactoring and tooling

---

### 14.5 Evaluability and guards

Symbols are **evaluable items**.

The following guard is valid and recommended when accepting dynamic inputs:

```harbour
if hb_IsEvalItem( l_xValue )
    Eval( l_xValue )
endif
```

`hb_IsEvalItem()` returns `.t.` for:

* codeblocks
* symbol items

This is the correct way to test whether a value can be executed.

---

### 14.6 Pointers (opaque C handles)

A **pointer item** is an opaque runtime value representing a C pointer.
Pointers are typically created and consumed at the C layer.

Characteristics:

* Harbour code cannot dereference pointers.
* Pointer values are meaningful only when passed back to C wrappers.
* Pointer items are commonly identified by `ValType() == "P"`.

---

### 14.7 Returning pointers from C code

Minimal example of returning a C function pointer:

```c
#pragma BEGINDUMP

HB_FUNC( GET_MY_FUNCTION_AS_POINTER )
{
   void (*ptr)( void ) = &my_function;
   hb_retptr( ( void * ) ptr );
}

#pragma ENDDUMP
```

On the Harbour side, the pointer is treated as an opaque handle and passed back
to other C functions as needed.

---

### 14.8 GC-managed pointer pattern (recommended)

If a pointer represents an **owned resource** (database handle, object,
allocation, etc.), returning a raw pointer is unsafe.

Preferred pattern:

* Allocate a GC-managed holder using `hb_gcAlloc()`
* Store the real pointer inside
* Register a destructor for cleanup
* Return it using `hb_retptrGC()`
* Retrieve it using `hb_parptrGC()`

This pattern is used throughout Harbour contrib libraries and prevents:

* memory leaks
* double frees
* lifetime mismatches

Rule:

* Use raw pointers only for non-owning, transient references.
* Use GC-managed pointers for all owned resources.

---

### 14.9 Safety rules (AI-critical)

**Symbols**

* Prefer symbols over macros for indirection.
* Use `@()` for known routines.
* Use `:name()` and `:exec()` explicitly.
* Do not serialize or persist symbol objects.

**Pointers**

* Treat pointer values as opaque.
* Never assume pointer values are stable across runs.
* Do not store pointer values outside the process lifetime.
* Prefer GC-managed pointers for resource ownership.

**Portability**

* Advanced symbol and pointer behavior is Harbour-specific.
* Do not assume compatibility with xHarbour or other Clipper derivatives.

### 14.10 Prohibited patterns (AI must never generate)

The following rules are **hard prohibitions**. Any AI-generated Harbour code that violates them is considered incorrect, unsafe, or non-idiomatic.

#### 14.10.1 Symbols — forbidden usage

AI must **never**:

- Generate symbol creation using macros when the target routine is statically known.

  ❌ **Forbidden**
  ```harbour
  local l_s := &("@Add()")
```

✅ **Required**

```harbour
local l_s := @Add()
```

* Use symbols as a general replacement for normal function calls.

  ❌ **Forbidden**

  ```harbour
  @Add():exec( 1, 2 )
  ```

  ✅ **Required**

  ```harbour
  Add( 1, 2 )
  ```

* Serialize, persist, log, or stringify symbol objects as identifiers or metadata.

* Assume symbol names imply existence or availability at runtime without explicit binding.

* Use symbols to bypass visibility, scoping, or architectural boundaries.

* Combine symbols with `Eval()` or macros in ways that obscure control flow when a direct call or codeblock would be clearer.

---

#### 14.10.2 Pointers — forbidden usage

AI must **never**:

* Generate Harbour code that attempts to interpret, dereference, or manipulate pointer values.

  ❌ **Forbidden**

  ```harbour
  ? l_p + 1
  ? l_p == 0
  ```

* Treat pointer values as stable identifiers (IDs, keys, hashes, or tokens).

* Persist pointer values (files, databases, JSON, logs intended for replay).

* Assume pointer values are portable across:

  * executions
  * processes
  * architectures (32-bit vs 64-bit)
  * operating systems

* Generate raw pointer ownership patterns without an explicit GC strategy at the C layer.

* Mix raw pointers and GC-managed pointers interchangeably.

---

#### 14.10.3 Cross-boundary misuse (symbols + pointers)

AI must **never**:

* Generate code where a symbol is converted into a pointer or vice versa.
* Attempt to pass symbol objects to C code expecting function pointers.
* Attempt to invoke pointer-returning C functions as if they were symbols.

These are distinct runtime concepts and are **not interchangeable**.

---

#### 14.10.4 Architectural intent

Symbols and pointers exist to solve **specific problems**:

* Symbols → controlled indirection and explicit dynamic dispatch
* Pointers → C-level interoperability and opaque resource handles

AI must not introduce either mechanism unless the problem statement **explicitly requires**:

* runtime dispatch tables,
* plugin-style extensibility,
* or C interop/resource management.

Default rule:

> If static structure suffices, symbols and pointers must not be used.

---

## 15) Date, time, timestamp & UTC

Harbour supports **Date** and **Timestamp** values natively.

- `ValType()` returns `"D"` for Date and `"T"` for Timestamp.
- A timestamp can represent either a full datetime **or** a **time-only** value (time-only is represented by a timestamp with an empty date).
- Timestamps may include fractions of a second (up to milliseconds).

---

### 15.1 Empty dates/timestamps are not `nil`

Harbour supports **empty** dates (and timestamps). This is distinct from `nil` and is commonly used in DBF fields and variables.

```harbour
function Main()

    local l_dEmpty := ctod( "" )

    ? l_dEmpty
    ? empty( l_dEmpty )    // .t.

return nil
```

Empty-date behavior matters because **time-only** values are modeled as timestamps with an empty date.

---

### 15.2 Arithmetic rules (the “days” rule)

- Subtracting two dates yields a numeric **day count**.
- Subtracting two timestamps yields a numeric **day count**, including a fractional day (it is *not* “seconds”).

Canonical conversions:

- `nDays := ( t2 - t1 )`
- `nSeconds := ( t2 - t1 ) * 86400`
- Convert seconds to a day fraction for timestamp arithmetic: `Days( nSeconds )`

```harbour
function Main()

    local l_tStart := hb_datetime()
    // ... work ...
    local l_tEnd   := hb_datetime()

    local l_nDays    := ( l_tEnd - l_tStart )
    local l_nSeconds := l_nDays * 86400

    ? l_nDays
    ? l_nSeconds

    // Add 30 seconds to a timestamp:
    ? hb_TSToStr( l_tStart + Days( 30 ) )

return nil
```

---

### 15.3 Building and splitting datetime values

**Extract date + time-of-day** from a timestamp:

```harbour
function Main()

    local l_tNow := hb_datetime()

    local l_dDate
    local l_nSeconds

    l_dDate := hb_TtoD( l_tNow, @l_nSeconds )  // l_nSeconds includes fractions
    ? l_dDate
    ? l_nSeconds

return nil
```

`hb_TtoD()` returns the date part and can also output time-of-day as seconds (or as a formatted string if you pass a string reference plus format).

**Construct / recombine**

Preferred canonical options:

- `hb_DtoT( <dDate>, <cnTime> )` creates a timestamp from a date plus a time value (seconds or time string).
- Combine a date with a **time-only timestamp** (`hb_SecToT()`), which yields a timestamp with that date + time.

```harbour
function Main()

    local l_dDate     := date()
    local l_nSeconds  := 9 * 3600 + 30 * 60
    local l_tTimeOnly := hb_SecToT( l_nSeconds )

    local l_t1 := hb_DtoT( l_dDate, l_nSeconds )
    local l_t2 := l_dDate + l_tTimeOnly

    ? hb_TSToStr( l_t1 )
    ? hb_TSToStr( l_t2 )

return nil
```

Note: adding two *dates* (or two full *timestamps*) is generally nonsensical; the “date + time-only timestamp” pattern is the intended use.

#### Duration vs component extraction functions

Do not confuse:

- **Component extraction** (read parts of a timestamp): `hb_Hour( <t> )`, `hb_Minute( <t> )`, `hb_Sec( <t> )`
- **Duration conversion** (convert a numeric time span expressed in *days*): `hb_NToHour( <nDays> )`, `hb_NToMin( <nDays> )`, `hb_NToSec( <nDays> )`, `hb_NToMSec( <nDays> )`

```harbour
function Main()

    local l_t1 := hb_datetime()
    local l_t2 := l_t1 + Days( 90 )   // +90 seconds

    local l_nSpanDays := ( l_t2 - l_t1 )

    ? hb_NToSec( l_nSpanDays )   // 90
    ? hb_Sec( l_t2 )             // seconds component from the timestamp (00-59.xxx)

return nil
```

---

### 15.4 Parsing and formatting (stable vs locale-dependent)

Avoid ambiguous parsing/formatting that depends on global `set` state:

- `CToD()` / `DToC()` are affected by `set date`, `set date format`, and `set epoch`.

Stable, locale-independent alternatives (recommended for storage, logs, DB keys, and APIs):

- Dates:
  - `DtoS( <d> )` → `YYYYMMDD`
  - `StoD( <cYYYYMMDD> )` / `hb_StoD( <cYYYYMMDD> )` → Date
- Timestamps:
  - `hb_TtoS( <t> )` → `YYYYMMDDHHMMSSFFF` (fixed-width, sortable, stable)
  - `hb_StoT( <cDateTime> )` parses `YYYYMMDDHHMMSSFFF` (and shorter variants) and also supports time-only inputs like `"HHMMSS"`
  - `hb_StrToTS( <cTimeStamp> )` parses an ISO-like `YYYY-MM-DD hh:mm:ss.fff` form
  - `hb_TSToStr( <t>, [<lPacked>] )` formats `YYYY-MM-DD hh:mm:ss.fff`; `<lPacked> == .t.` strips empty parts

If you must parse/format locale/UI strings, prefer the Harbour extended forms and pass **explicit formats**:

- `hb_CtoD( <cDate>, <cDateFormat> )` / `hb_DtoC( <d>, <cDateFormat> )`
- `hb_CtoT( <cTimeStamp>, <cDateFormat>, <cTimeFormat> )`
- `hb_TtoC()` / `hb_CtoT()` are format-driven and will use the format set by `SET_TIMEFORMAT` unless you pass explicit formats—pass explicit formats for deterministic behavior.

---

### 15.5 UTC, local time, and DST

Harbour timestamps do not intrinsically store a timezone; treat them as “local” or “UTC” by convention at your system boundaries.

- `hb_TSToUTC( <tLocalTime> )` converts a local timestamp to UTC.
- `hb_UTCOffset( <tLocalTime> )` returns the signed UTC offset in **seconds** (DST-aware for that timestamp).
- `hb_UTCToTS( <tUTCTime>, [@<lUTCOffset>] )` converts UTC to local time in newer Harbour sources (added 2025-12-12).

Recommended boundary rule for multi-system apps:

- **Store timestamps in UTC** (or store the offset alongside the local timestamp).
- Convert to local time only for display or local scheduling.

---

### 15.6 Handy APIs (common core set)

- Current datetime: `hb_datetime()` / `hb_DateTime()`
- Split/merge: `hb_TtoD()`, `hb_DtoT()`
- Seconds/time-only: `hb_TToSec()`, `hb_SecToT()`, `hb_Sec()`
- Duration conversions (numeric day spans): `hb_NToHour()`, `hb_NToMin()`, `hb_NToSec()`, `hb_NToMSec()`
- Serialization/parsing (stable): `DtoS()`, `StoD()`, `hb_StoD()`, `hb_TtoS()`, `hb_StoT()`, `hb_StrToTS()`, `hb_TSToStr()`
- UTC: `hb_TSToUTC()`, `hb_UTCOffset()` (and `hb_UTCToTS()` if available in your build)

---

## 16) NIL, NULL, empty — semantics & helpers

This section defines how Harbour represents “no value” (`NIL`), how database `NULL` should be mapped in application code, and how “empty values” (`Empty()`) differ from `NIL`. These distinctions are mandatory for correct optional-parameter handling, ORM writes, JSON interop, and defensive runtime checks.

---

### 16.1 Terminology and the one rule that prevents most bugs

**Concepts that must not be conflated:**

* **`NIL`**: Harbour’s language-level “no value” / “unset”.

  * `ValType( NIL ) == "U"`.
  * Uninitialized locals/privates/memvars are `NIL` unless explicitly assigned.
* **SQL `NULL`**: A database-level “unknown / missing / not applicable” value.

  * Harbour itself does not have a built-in SQL `NULL` scalar; you must **map** SQL `NULL` to something at the application boundary.
  * **Project rule:** represent SQL `NULL` as `NIL` *in row buffers / field values*, and use ORM metadata/helpers when you must distinguish “field missing” vs “field present but NULL”.
* **Empty values**: valid, intentional values such as `""`, `0`, `.f.`, `{}`, `{=>}`, and empty dates/timestamps.

  * These are **values**, not “unset”.

**Canonical guidance:**
Use `hb_IsNil()` to test for “unset”. Use `Empty()` only when you explicitly mean “empty-ish value (including NIL)”.

---

### 16.2 Quick reference matrix

Use this matrix when deciding whether you need `hb_IsNil()`, `Empty()`, or type-specific checks.

| Value / scenario                        | `ValType()` | `hb_IsNil(x)` |      `Empty(x)` | Notes                                                         |
| --------------------------------------- | ----------: | ------------: | --------------: | ------------------------------------------------------------- |
| `NIL`                                   |       `"U"` |         `.t.` | typically `.t.` | `Empty()` is **not** a presence check.                        |
| `""`                                    |       `"C"` |         `.f.` |           `.t.` | Empty string is a value.                                      |
| `" "` (space)                           |       `"C"` |         `.f.` |           `.f.` | Often you want `Empty( AllTrim(c) )` instead.                 |
| `{}`                                    |       `"A"` |         `.f.` |           `.t.` | Array length 0.                                               |
| `{=>}`                                  |       `"H"` |         `.f.` |           `.t.` | Hash length 0.                                                |
| `0`                                     |       `"N"` |         `.f.` |           `.t.` | Numeric zero is a value.                                      |
| `.f.`                                   |       `"L"` |         `.f.` |           `.t.` | Logical false is a value.                                     |
| Empty date                              |       `"D"` |         `.f.` |           `.t.` | Your date “empty” convention must be consistent project-wide. |
| “Time-only” timestamp / empty timestamp |       `"T"` |         `.f.` |         depends | See Section 15; define your canonical “empty timestamp” rule. |

**Important:** `Empty()` returning `.t.` does **not** mean “missing”; it means “empty-ish under Harbour’s rules”, which usually includes `NIL`, `0`, and `.f.`.

---

### 16.3 Recommended checks (what to use, when)

**Presence / unset checks**

* Use `hb_IsNil( x )` to detect “unset / not provided / missing”.
* Prefer guarding optional behavior with `PCount()` when the difference between “omitted” and “passed NIL” matters (see 16.4).

**Empty-value checks**

* Use `Empty( x )` when you intentionally want Harbour’s notion of “empty-ish”.
* When the type is known, type-specific checks are often clearer:

  * `Len( cValue ) == 0` for strings
  * `Len( aValue ) == 0` for arrays
  * `Len( hValue ) == 0` for hashes
* For user-entered strings, your typical “blank” check should be:

  * `Empty( AllTrim( cValue ) )` (blank-or-whitespace)

**Do not mix these semantics**

* “Unset” (`NIL`) is handled with `hb_IsNil()`.
* “Blank string” is handled with `Empty(AllTrim())`.
* “False” is `.f.` and must not be treated as “missing”.

---

### 16.4 Optional parameters: omitted vs explicitly passed `NIL`

AI-generated code must preserve the semantic difference between:

* *argument omitted* (caller did not provide it), and
* *argument provided as `NIL`* (caller explicitly supplied NIL).

Use `PCount()` to detect omission.

```harbour
function NormalizeLimit(par_nLimit)
local l_nLimit

if PCount() < 1
    // argument omitted: apply routine default
    l_nLimit := 100
elseif hb_IsNil(par_nLimit)
    // argument explicitly passed as NIL: treat as “no limit”
    l_nLimit := NIL
else
    l_nLimit := par_nLimit
endif

return l_nLimit
```

**Rule:** do not “helpfully” collapse omission and explicit `NIL` into the same path unless the API contract explicitly says they are equivalent.

---

### 16.5 Defaulting patterns

Use defaults to turn `NIL` into a concrete value **only when the API contract calls for it**.

**Pattern A: normalize a working local**

```harbour
function BuildGreeting(par_cName)
local l_cName := par_cName

if hb_IsNil(l_cName)
    l_cName := ""
endif

return "Hello " + l_cName
```

**Pattern B: in-place default on a local/param variable**

```harbour
function BuildGreeting2(par_cName)
hb_Default(@par_cName, "")
return "Hello " + par_cName
```

**Pattern C: expression default**

```harbour
function BuildGreeting3(par_cName)
return "Hello " + hb_DefaultValue(par_cName, "")
```

**Project rule:** do not default by using `Empty()` unless you explicitly want `0`/`.f.`/`""` treated the same as `NIL`. In most application code, that is incorrect.

---

### 16.6 Equality and comparisons involving `NIL`

**Presence testing**

* Prefer `hb_IsNil( x )` over `x == NIL` for readability and intent.

**Comparisons / ordering**

* Avoid ordering comparisons (`<`, `>`, `<=`, `>=`) on values that may be `NIL`. Normalize first:

  * either default `NIL` to a safe sentinel **with an explicit rule**, or
  * branch on `hb_IsNil()`.

**Collections**

* Arrays may contain `NIL` elements; treat this as a supported scenario and normalize when needed (e.g., before sorting).
* Hash keys must not be `NIL`; do not generate code that attempts to use `NIL` as a hash key.

---

### 16.7 SQL NULL mapping and Harbour_ORM expectations

This document is Harbour-first. However, most real applications touch SQL. Your ORM boundary must define a strict mapping.

**Canonical mapping policy**

* **Read path (SQL → Harbour):** map SQL `NULL` to `NIL` in the returned field value.
* **Write path (Harbour → SQL):** distinguish the three intents explicitly:

  1. **Leave unchanged** (typical for partial updates)
  2. **Set to a concrete value** (including empty string / 0 / false when those are valid values)
  3. **Set to SQL NULL** (explicitly requested)

**Rule:** do not use `""`, `0`, or `.f.` as stand-ins for SQL `NULL`.

**ORM null-aware checks**

* When reading from a work-area/alias where the ORM tracks null state, prefer ORM helpers (example):

  * `hb_orm_isnull(cAlias, cField)`
* Use ORM-provided setters for writing SQL `NULL` (whatever their exact names are in your codebase). Do not invent setter APIs.

---

### 16.8 JSON interop rules (hb_jsonEncode / hb_jsonDecode)

When values cross a JSON boundary, you must preserve the difference between:

* property **missing** (no key),
* property present with JSON `null`,
* property present with an empty value.

**Recommended policy**

* JSON `null` ↔ Harbour `NIL`.
* Missing key ↔ key not present in the hash (do not auto-insert `NIL` keys unless your API contract requires it).
* If an API requires “omit means no change”, then the encoder must omit keys rather than sending `null`.

**Implementation note**

* If you require omission, remove the key from the hash rather than relying on encoder behavior.

---

### 16.9 Explicit constraints: what AI must never generate in this section’s domain

AI-generated Harbour code must **not**:

* Use `Empty(x)` as a synonym for `hb_IsNil(x)` (they are not equivalent).
* Treat `.f.` or `0` as “missing” in business logic unless explicitly documented.
* Convert SQL `NULL` into `""`, `0`, or `.f.` implicitly.
* Write partial-update logic that collapses:

  * “argument omitted” and “argument passed as NIL”
    into the same behavior, unless explicitly required.
* Invent ORM APIs for setting SQL `NULL`. Only use functions/macros that exist in the project’s function set.

---

### 16.10 Conformance test snippet (recommended to keep in the repo)

Use this routine to verify runtime behavior after Harbour upgrades or when validating AI-generated code.

```harbour
function TestNilNullEmpty()
local l_xNil := NIL
local l_cEmpty := ""
local l_cSpace := " "
local l_nZero := 0
local l_lFalse := .f.
local l_aEmpty := {}
local l_hEmpty := {=>}
local l_dEmpty := CToD("")

? "NIL      :", ValType(l_xNil),   hb_IsNil(l_xNil),   Empty(l_xNil)
? '""       :', ValType(l_cEmpty), hb_IsNil(l_cEmpty), Empty(l_cEmpty)
? '" "      :', ValType(l_cSpace), hb_IsNil(l_cSpace), Empty(l_cSpace)
? "0        :", ValType(l_nZero),  hb_IsNil(l_nZero),  Empty(l_nZero)
? ".f.      :", ValType(l_lFalse), hb_IsNil(l_lFalse), Empty(l_lFalse)
? "{}       :", ValType(l_aEmpty), hb_IsNil(l_aEmpty), Empty(l_aEmpty), Len(l_aEmpty)
? "{=>}     :", ValType(l_hEmpty), hb_IsNil(l_hEmpty), Empty(l_hEmpty), Len(l_hEmpty)
? "empty D  :", ValType(l_dEmpty), hb_IsNil(l_dEmpty), Empty(l_dEmpty)

return NIL
```

This test intentionally prints only the load-bearing semantics needed for code generation: type, NIL detection, and `Empty()` behavior.

---

## 17) Legacy `SET` pseudo-commands and the `Set()` runtime (scope-limited)

Harbour supports a large set of xBase-style **pseudo-commands** (preprocessor `#command` rules) for legacy compatibility. `SET` is the most important family: most `SET ...` statements are expanded at compile time into calls to `Set( _SET_..., ... )`.

This section provides:

- A complete, up-to-date list of `Set()` indices (`_SET_*`) based on `set.ch`.
- A practical mapping of commonly relevant `SET ...` pseudo-commands to their **recommended function form**.
- A clear **scope boundary**: console and full-screen `@ ... SAY/GET/READ`-era behavior is intentionally out of scope.

---

### 17.1 Scope boundary (what is and is not documented)

**Documented here (in-scope):**

- `Set()` runtime behavior and `_SET_*` indices.
- `SET` options that affect **data conversion/formatting**, **search semantics**, **file/path resolution**, and **RDD/DBF behavior** (typical back-end/server relevance).

**Intentionally not documented (out-of-scope):**

- Full-screen console UI and `@ ... SAY/GET/READ` workflows.
- Console device routing and output redirection (`SET CONSOLE`, `SET PRINTER`, `SET ALTERNATE`, etc.).
- Keyboard/typeahead and UI ergonomics (`SET BELL`, `SET CURSOR`, `SET SCOREBOARD`, etc.).

These legacy switches may still exist for compatibility; they are listed for completeness but not explained in detail.

---

### 17.2 Where the truth lives (authoritative sources)

In Harbour, the definitive list is the **source**:

- `include/set.ch` — defines the `_SET_*` numeric indices.
- `include/std.ch` — defines `#command` rules like `SET DATE ...` and expands them into `Set()` (or related helper calls).

If your Harbour build is customized, these two files are the ground truth.

---

### 17.3 Prefer function calls over `SET` pseudo-commands

For modern Harbour code (especially non-console apps), prefer the function form:

```harbour
#include "set.ch"

local cOld := Set( _SET_DATEFORMAT, "yyyy-mm-dd" )
local cFmt := Set( _SET_DATEFORMAT )     // query current value
```

`Set()` is both a **getter** (when called with only the index) and a **setter** (when called with `xNewSetting`). It returns the previous value.

Some `_SET_*` indices accept an extra option (commonly an additive flag or similar), so `Set()` also accepts an optional third argument.

---

### 17.4 Canonical mappings: `SET ...` pseudo-command → function form

Below are the recommended function equivalents for the most relevant `SET` pseudo-commands. (The `SET ...` forms are still accepted and can be useful when porting legacy xBase code.)

#### Date and time

```harbour
#include "set.ch"

// SET DATE FORMAT TO <cFormat>
Set( _SET_DATEFORMAT, "yyyy-mm-dd" )

// SET TIME FORMAT TO <cFormat>   (Harbour extension)
Set( _SET_TIMEFORMAT, "hh:mm:ss" )

// SET EPOCH TO <nYear>
Set( _SET_EPOCH, 1950 )
```

`SET CENTURY ON/OFF` is implemented through a helper (historically `__SetCentury()`), and it indirectly affects the effective date picture used by conversions/formatting.

#### Numeric formatting

```harbour
#include "set.ch"

// SET FIXED ON/OFF
Set( _SET_FIXED, .T. )

// SET DECIMALS TO <n>
Set( _SET_DECIMALS, 4 )
```

**Definition / mapping**

- `_SET_FIXED` is the `Set()` index **2**.
- The legacy pseudo-command expands to `Set( _SET_FIXED, <x> )`.

- `_SET_DECIMALS` is the `Set()` index **3**.
- The legacy pseudo-command expands to:
  - `Set( _SET_DECIMALS, <x> )` when a value is supplied, and
  - `Set( _SET_DECIMALS, 0 )` when omitted.

**What they do (practical semantics)**

`_SET_FIXED` controls **how numeric values are formatted when converted to strings using default/implicit formatting rules**. When it is **ON**, numeric-to-string formatting is typically “fixed-point”: it keeps (or pads) trailing zeros up to the current decimals setting. When it is **OFF**, formatting may be “floating” in the sense that trailing zeros may be omitted by default-format conversions.

`_SET_DECIMALS` defines the **default number of decimal places** used by numeric-to-string formatting **when the formatting routine does not specify decimals explicitly**.

Key points:

- These do **not** change numeric precision or arithmetic. They only affect **formatting** (string rendering).
- Their effect is most visible when a numeric value is rendered **without an explicit picture/format**.

**Interaction between `_SET_FIXED` and `_SET_DECIMALS`**

A useful mental model:

- `_SET_DECIMALS` answers: “How many decimal places do we want by default?”
- `_SET_FIXED` answers: “When printing, do we always show that many decimals (including trailing zeros), or do we allow trimming?”

A common compatibility pattern is:

```harbour
#include "set.ch"

Set( _SET_DECIMALS, 4 )
Set( _SET_FIXED, .T. )
```

This tends to produce stable fixed-width numeric renderings (legacy reporting style) when default formatting is used.

**Guidance for modern code**

For non-console/server code, prefer **explicit formatting** rather than relying on ambient global state. Use `_SET_FIXED` / `_SET_DECIMALS` only when you are intentionally emulating legacy xBase output semantics.

#### String comparison semantics

```harbour
#include "set.ch"

// SET EXACT ON/OFF
Set( _SET_EXACT, .T. )
```

This influences legacy string comparisons using `=` and certain xBase behaviors. In modern Harbour code, prefer `==` for exact equality and purpose-built functions for prefix/substring tests.

#### File and search paths

```harbour
#include "set.ch"

// SET DEFAULT TO <cPath>
Set( _SET_DEFAULT, "c:\data" )
```

**Definition / mapping**

- `_SET_DEFAULT` is the `Set()` index **7**.
- The legacy pseudo-command expands to:
  - `Set( _SET_DEFAULT, <path> )`, and
  - `Set( _SET_DEFAULT, "" )` when omitted (i.e., “clear it”).

**What it does (practical semantics)**

`_SET_DEFAULT` sets Harbour’s **default directory context** used by many xBase-style file operations, particularly those that work with **relative paths** or traditional DBF/RDD workflows.

Key points:

- It influences how **relative filenames** are resolved by code paths that consult the xBase environment settings.
- It is commonly paired with `_SET_PATH` (search path list) to emulate “current directory + search path” resolution.

**Operational guidance (especially for services / toolchains)**

- Prefer **absolute paths** in modern apps (services, FastCGI, background workers) unless you are deliberately preserving a legacy behavior layer.
- Treat `_SET_DEFAULT` as **ambient process state**: changing it inside a shared codebase can have surprising downstream effects on code that opens files by relative name.
- If you must use it, adopt a “save/restore” discipline:

```harbour
#include "set.ch"

local cPrevDefault := Set( _SET_DEFAULT )
Set( _SET_DEFAULT, "c:\data" )

// ... operations that rely on relative paths ...

Set( _SET_DEFAULT, cPrevDefault )
```

This makes the scope of the side effect explicit and reduces accidental coupling.

#### DBF/RDD behavior (still relevant when using DBF/RDDs)

```harbour
#include "set.ch"

// SET DELETED ON/OFF
Set( _SET_DELETED, .T. )

// SET EXCLUSIVE ON/OFF
Set( _SET_EXCLUSIVE, .F. )

// SET SOFTSEEK ON/OFF
Set( _SET_SOFTSEEK, .T. )

// SET UNIQUE ON/OFF
Set( _SET_UNIQUE, .F. )
```

---

### 17.5 Complete `_SET_*` index list (from `set.ch`)

The following table is meant to be a practical “inventory” for documentation and tooling.  
`Has SET ... pseudo-command` means `std.ch` contains at least one `#command SET ...` translation that calls `Set()` with that index.

| ID | Constant | Origin | Has `SET ...` pseudo-command | Scope in this guide | Notes |
|---:|---|---|---|---|---|
| 1 | `_SET_EXACT` | clipper | yes | documented |  |
| 2 | `_SET_FIXED` | clipper | yes | documented |  |
| 3 | `_SET_DECIMALS` | clipper | yes | documented |  |
| 4 | `_SET_DATEFORMAT` | clipper | yes | documented |  |
| 5 | `_SET_EPOCH` | clipper | yes | documented |  |
| 6 | `_SET_PATH` | clipper | yes | documented |  |
| 7 | `_SET_DEFAULT` | clipper | yes | documented |  |
| 8 | `_SET_EXCLUSIVE` | clipper | yes | documented |  |
| 9 | `_SET_SOFTSEEK` | clipper | yes | documented |  |
| 10 | `_SET_UNIQUE` | clipper | yes | documented |  |
| 11 | `_SET_DELETED` | clipper | yes | documented |  |
| 12 | `_SET_CANCEL` | clipper | no | legacy console/UI (out of scope) |  |
| 13 | `_SET_DEBUG` | clipper | no | legacy/compat (listed only) |  |
| 14 | `_SET_TYPEAHEAD` | clipper | yes | legacy console/UI (out of scope) |  |
| 15 | `_SET_COLOR` | clipper | no | legacy console/UI (out of scope) |  |
| 16 | `_SET_CURSOR` | clipper | no | legacy console/UI (out of scope) |  |
| 17 | `_SET_CONSOLE` | clipper | yes | legacy console/UI (out of scope) |  |
| 18 | `_SET_ALTERNATE` | clipper | yes | legacy console/UI (out of scope) |  |
| 19 | `_SET_ALTFILE` | clipper | yes | legacy console/UI (out of scope) |  |
| 20 | `_SET_DEVICE` | clipper | yes | legacy console/UI (out of scope) |  |
| 21 | `_SET_EXTRA` | clipper | no | legacy console/UI (out of scope) |  |
| 22 | `_SET_EXTRAFILE` | clipper | no | legacy console/UI (out of scope) |  |
| 23 | `_SET_PRINTER` | clipper | yes | legacy console/UI (out of scope) |  |
| 24 | `_SET_PRINTFILE` | clipper | yes | legacy console/UI (out of scope) |  |
| 25 | `_SET_MARGIN` | clipper | yes | legacy console/UI (out of scope) |  |
| 26 | `_SET_BELL` | clipper | yes | legacy console/UI (out of scope) |  |
| 27 | `_SET_CONFIRM` | clipper | yes | legacy console/UI (out of scope) |  |
| 28 | `_SET_ESCAPE` | clipper | yes | legacy console/UI (out of scope) |  |
| 29 | `_SET_INSERT` | clipper | no | legacy console/UI (out of scope) |  |
| 30 | `_SET_EXIT` | clipper | no | legacy console/UI (out of scope) |  |
| 31 | `_SET_INTENSITY` | clipper | yes | legacy console/UI (out of scope) |  |
| 32 | `_SET_SCOREBOARD` | clipper | yes | legacy console/UI (out of scope) |  |
| 33 | `_SET_DELIMITERS` | clipper | yes | legacy/compat (listed only) |  |
| 34 | `_SET_DELIMCHARS` | clipper | yes | legacy/compat (listed only) |  |
| 35 | `_SET_WRAP` | clipper | yes | legacy console/UI (out of scope) |  |
| 36 | `_SET_MESSAGE` | clipper | yes | legacy console/UI (out of scope) |  |
| 37 | `_SET_MCENTER` | clipper | yes | legacy console/UI (out of scope) |  |
| 38 | `_SET_SCROLLBREAK` | clipper | no | legacy console/UI (out of scope) |  |
| 39 | `_SET_EVENTMASK` | clipper | yes | legacy console/UI (out of scope) | CA-Cl*pper 5.3 compatible |
| 40 | `_SET_VIDEOMODE` | clipper | yes | legacy console/UI (out of scope) | CA-Cl*pper 5.3 compatible |
| 41 | `_SET_MBLOCKSIZE` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 42 | `_SET_MFILEEXT` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 43 | `_SET_STRICTREAD` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 44 | `_SET_OPTIMIZE` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 45 | `_SET_AUTOPEN` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 46 | `_SET_AUTORDER` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 47 | `_SET_AUTOSHARE` | clipper | yes | legacy/compat (listed only) | CA-Cl*pper 5.3 compatible |
| 100 | `_SET_LANGUAGE` | harbour | no | advanced/rare (listed only) | Harbour extension |
| 101 | `_SET_IDLEREPEAT` | harbour | no | advanced/rare (listed only) | Harbour extension |
| 102 | `_SET_FILECASE` | harbour | yes | documented | Harbour extension |
| 103 | `_SET_DIRCASE` | harbour | yes | documented | Harbour extension |
| 104 | `_SET_DIRSEPARATOR` | harbour | yes | documented | Harbour extension |
| 105 | `_SET_EOF` | harbour | no | documented | Harbour extension |
| 106 | `_SET_HARDCOMMIT` | harbour | yes | documented | Harbour extension |
| 107 | `_SET_FORCEOPT` | harbour | no | advanced/rare (listed only) | Harbour extension |
| 108 | `_SET_DBFLOCKSCHEME` | harbour | yes | documented | Harbour extension |
| 109 | `_SET_DEFEXTENSIONS` | harbour | no | documented | Harbour extension |
| 110 | `_SET_EOL` | harbour | yes | documented | Harbour extension |
| 111 | `_SET_TRIMFILENAME` | harbour | no | documented | Harbour extension |
| 112 | `_SET_HBOUTLOG` | harbour | no | advanced/rare (listed only) | Harbour extension |
| 113 | `_SET_HBOUTLOGINFO` | harbour | no | advanced/rare (listed only) | Harbour extension |
| 114 | `_SET_CODEPAGE` | harbour | no | documented | Harbour extension |
| 115 | `_SET_OSCODEPAGE` | harbour | no | documented | Harbour extension |
| 116 | `_SET_TIMEFORMAT` | harbour | yes | documented | Harbour extension |
| 117 | `_SET_DBCODEPAGE` | harbour | no | documented | Harbour extension |

---

### 17.6 Notes on legacy `SET` coverage and no-ops

Not every `SET ...` spelling necessarily maps to `Set()`:

- Some are implemented via helpers (e.g., `SET CENTURY`, `SET KEY`, `SET FUNCTION`).
- Some legacy spellings are accepted as compatibility **no-ops** depending on how `std.ch` defines them.

For modern non-console applications, prefer explicit function calls and treat `SET ...` as a compatibility layer.

---

### 17.7 References (for humans)

- Harbour Reference Guide: `Set()` and `_SET_*` documentation.
- Harbour core sources: `include/set.ch`, `include/std.ch`.

---

## 18) Workareas, tables & indexes (ISAM model)

This section defines the classic xBase / ISAM execution model used by DBF/RDD backends: **workareas**, **record pointers**, **orders (indexes)**, **filters/scopes**, and **locking**. Correct code must respect that many of these are **ambient (implicit) state** associated with the *currently selected* workarea.

> Terminology used here is intentionally RDD-centric. ORMs and SQL layers may *materialize* result sets into in-memory tables that still behave like workareas.

---

### 18.1 What a workarea is

A **workarea** is a logical “cursor slot” that can hold **one** opened table/view at a time. Each workarea has:

* A selected **alias** (string name like `"CUST"`), addressable by alias or numeric workarea id.
* A **current record pointer** (`recno()`).
* A **table size / last record** (`reccount()`).
* A **controlling order** (index/tag) that defines traversal order and enables `dbSeek()`.
* Optional **filter** and **relations**, which can change which records are visible during navigation.

Multiple workareas can open the **same physical DBF** concurrently under different aliases (and potentially different orders/filters).

**Important in-memory note (ORM/materialized tables):**

* Treat a materialized **in-memory table** as *owned by exactly one workarea*. If you need two independent cursors, materialize twice (or explicitly clone/copy into a second in-memory table). Do not assume you can “mount” the same in-memory table into multiple workareas.

---

### 18.2 Ambient state and the “current workarea” rule

Many commands/functions operate on the **currently selected** workarea. This is convenient in interactive code, but fragile in reusable routines.

**Canonical rule:** any routine that changes the selected workarea must **restore it** before returning.

#### 18.2.1 `select` (command) vs `Select()` (function)

* `select Cust` is the **command** that changes the current workarea.
* `Select()` is a **function** that returns a workarea number; it does **not** change the current workarea. Do not confuse these.

#### 18.2.2 Save/restore selection safely (including “no active area”)

If a routine may be called when *no DBF is open in the current area*, store/restore like this:

```harbour
static function WithCustArea( par_bAction )

    local l_nPreviousArea := iif( Used(), Select(), 0 )

    select Cust
    Eval( par_bAction )

    select ( l_nPreviousArea )

return NIL
```

Notes:

* The `Used()` guard avoids capturing a misleading “current area number” when nothing is open.
* Restoring with `select ( 0 )` is accepted practice when the caller might not have had an active used area; it selects a workarea per xBase semantics (commonly “first free”).

#### 18.2.3 Avoid selection changes when possible: aliased evaluation

When you want to query or execute logic in another workarea without changing the current selection, use aliased evaluation:

```harbour
local l_cAlias := "CUST"

? Cust->( RecCount() )
? ( l_cAlias )->( RecNo() )

Cust->( dbGoTop() )
? Cust->City
```

This pattern is especially useful inside helpers: it reduces ambient-state coupling, and it keeps “who changed the current workarea” easier to reason about.

---

### 18.3 Opening and closing tables

Most code uses the `use` command, which is translated into `dbUseArea()`.

#### 18.3.1 Prefer full paths for free tables

In modern apps/services, **avoid relative DBF paths** unless you are deliberately emulating legacy “current directory + SET PATH” behavior. Prefer an absolute or computed full path:

```harbour
local l_cDbf := cPath + "Customer.dbf"

use ( l_cDbf ) alias Cust shared new
```

#### 18.3.1.1 Portable path construction

When you need portable code across Windows/Linux/macOS, prefer building paths using `hb_ps()` (path separator) rather than hard-coding `\` or `/`.

```harbour
local l_cPath := "." + hb_ps() + "Tables" + hb_ps()
local l_cDbf  := l_cPath + "Customer.dbf"
```

Notes:
- `hb_ps()` returns the platform path separator (`"\\"` or `"/"`).
- Keep the separator logic in one place. If you need richer semantics (join/normalize), implement a small helper and reuse it.

#### 18.3.2 Preferred explicit open: `dbUseArea()`

In many codebases it is safer to use `dbUseArea()` directly so you can specify all important parameters in one place.

Signature (conceptual):

* `dbUseArea( [lNewArea], [cRddName], cDatabase, [cAlias], [lShared], [lReadOnly], [cCodePage], [nConnection] ) -> lSuccess`

Parameter intent:

* `lNewArea`: `.t.` opens in the next available unused workarea; `.f.` uses the current area (closing any table already open there).
* `cRddName`: driver name (e.g. `"DBFCDX"`).
* `cDatabase`: DBF file name (strongly prefer full path).
* `cAlias`: workarea alias (if omitted, defaults to file base name).
* `lShared`: shared vs exclusive (if omitted, it is derived from ambient `_SET_EXCLUSIVE` rules).
* `lReadOnly`: open read-only vs read-write.
* `cCodePage`: workarea codepage (project policy should define this consistently).
* `nConnection`: connection handle for RDDs that support it (often omitted for plain DBF).

Canonical example (your style):

```harbour
local l_cDbf := cPath + cContactTableName + ".dbf"

if ! dbUseArea( .t., "DBFCDX", l_cDbf, "MyContactsByFirstName", .t., .f., "EN" )
    // handle failure: return .f. or raise an error per project rules
endif
```

#### 18.3.3 Closing

```harbour
select Cust
dbCloseArea()

// Close by alias stored in a variable (aliased evaluation; does not change the caller selection):
( Select( par_cTableName ) )->( dbCloseArea() )

// Close everything only in well-defined shutdown/tooling code:
dbCloseAll()
```

Guidance:

* Use `alias ... new` (or `lNewArea := .t.`) when there is any chance the alias is already in use.
* Prefer `dbCloseArea()` in library code; reserve `dbCloseAll()` for explicit shutdown/tooling.

#### 18.3.4 Canonical “create-if-missing” DBF + CDX (free table)

A practical local-DBF pattern is:

1) ensure the folder exists  
2) create the DBF if missing  
3) create indexes if missing  
4) reopen tables in the desired shared/read-write mode  
5) close explicitly

Example (DBFCDX + VFP table type):

```harbour
request DBFCDX
request DBFFPT
request HB_CODEPAGE_EN

static function CreateTableAsNeeded( par_cDbfFullPath, par_aStructure )

    if !File( par_cDbfFullPath )
        select 0

        // dbCreate(<cFileName>,<aStruct>,[<cRDD>],[<lKeepOpen>],[<cAlias>],[<cDelimArg>],[<cCodePage>],[<nConnection>]) -> <lSuccess>
        DbCreate( par_cDbfFullPath, par_aStructure, "DBFCDX", .f.,,,"EN" )

        // Some table types may still end up opened; close so we can reopen in shared mode.
        if Used()
            dbCloseArea()
        endif
    endif

return nil

static procedure EnsureContactTables( par_cPath )

    local l_aStructContact := {;
              { "KEY"      , "I:+", 4, 0 },;
              { "FIRSTNAME", "C"  ,60, 0 },;
              { "LASTNAME" , "C"  ,60, 0 },;
              { "NOTE"     , "M"  , 4, 0 },;
              { "DOB"      , "D"  , 8, 0 } }

    hb_DirCreate( par_cPath )

    RddSetDefault( "DBFCDX" )
    rddInfo( RDDI_TABLETYPE, DB_DBF_VFP )
    Set( _SET_CODEPAGE, "EN" )

    CreateTableAsNeeded( par_cPath + "Contact.dbf", l_aStructContact )

    // Build indexes once
    if !File( par_cPath + "Contact.cdx" )
        dbUseArea( .t., "DBFCDX", par_cPath + "Contact.dbf", "Contact", .f., .f., "EN" )
        OrdCreate( par_cPath + "Contact.cdx", "tag1", "upper(FIRSTNAME+LASTNAME)",,.f. )
        OrdCreate( par_cPath + "Contact.cdx", "tag2", "upper(LASTNAME+FIRSTNAME)",,.f. )
        dbCloseArea()
    endif

    // Normal shared opens
    dbUseArea( .t., "DBFCDX", par_cPath + "Contact.dbf", "MyContactsByFirstName", .t., .f., "EN" )
    ordSetFocus( "tag1" )

return
```

Guidance:
- Keep “DDL-like” logic (create, index creation) separate from normal runtime paths.
- Always reopen tables with the desired `shared`/`readonly` flags after creation.
- If you use non-default table types (e.g., `DB_DBF_VFP`), document why (memos, codepages, compatibility).

---

### 18.4 Navigation and iteration (record pointer semantics)

Core primitives:

* `dbGoTop()` / `dbGoBottom()` position at first/last record under the current order/filter/relation context.
* `dbSkip( n )` moves the record pointer by `n` records (typically `1`).
* `bof()` / `eof()` signal pointer before-first / after-last.

Canonical full-table loop:

```harbour
dbGoTop()

do while ! Eof()
    // process current record
    dbSkip()
enddo
```

Notes that matter for correct code generation:

* `go top` / `go bottom` are **order-aware**. If an order is active, traversal is by the index order.
* Filters/relations can change what “first/last/next” means; never assume `reccount()` equals “visible rows”.

---

### 18.5 Field access: explicit vs implicit (and update forms)

Field access options:

* **Explicit (preferred in reusable code):** `Alias->Field`
* **Implicit (depends on current workarea):** unqualified `Field`

Example:

```harbour
select Cust

// explicit: safe even if workarea changes elsewhere
replace Cust->City with "Seattle"
Cust->City := "Seattle"

// implicit: only correct if "CUST" is currently selected
replace City with "Seattle"
City := "Seattle"
```

Name collisions (field vs memvar):

When a field name conflicts with a memory variable (PUBLIC/PRIVATE), you can force resolution with special aliases:

* `FIELD->FieldName` forces a field reference in the current workarea.
* `MEMVAR->VarName` forces a memory variable reference; `M->VarName` is a common shorthand.

```harbour
select Cust
? FIELD->City
? M->City
```

Because `M->` is a memvar alias, avoid using `"M"` as a workarea alias in project code.

Rules for AI-generated code:

* Prefer `Alias->Field` (or aliased evaluation) when:

  * the routine can be called from multiple places, or
  * it selects/uses multiple workareas, or
  * it is inside a helper/utility function.

---

### 18.6 Updating records, locks, and committing visibility

Updates may be performed via `replace`, direct field assignment, `fieldput()`, etc. In shared/multi-process usage, correctness typically requires a **lock strategy**.

Recommended pattern (record update in shared DBF scenarios):

```harbour
// assume record is positioned
if dbRLock()

    Cust->City := "Seattle"
    // or: replace Cust->City with "Seattle"

    dbCommit()
    dbUnlock()

endif
```

Guidance:

* Keep lock scope minimal: lock → update → commit → unlock.
* Use `dbCommit()` as the explicit “flush to disk/index buffers” step when correctness depends on other processes seeing changes promptly.
* Do not rely on implicit commit timing.

---

### 18.7 Filters and relations: they change what “all records” means

Filters and relations are per-workarea state and can silently affect navigation and loops.

Key operations:

* Clear current filter:

  ```harbour
  dbClearFilter()
  ```

* Inspect active filter (returns codeblock or `NIL`):

  ```harbour
  local l_bFilter := hb_dbGetFilter()
  ```

Relations (when used) also affect traversal; always document relation assumptions near the code that sets them, and clear them when leaving a scoped routine:

```harbour
dbClearRelation()
```

**Rule:** if a routine assumes “scan everything”, it must ensure filters/relations are not unexpectedly active (or explicitly account for them).

---

### 18.8 Indexes and orders: order list vs controlling order

An important distinction:

* **Order list**: which index files/tags are opened/available in the workarea.
* **Controlling order**: which tag currently controls traversal and enables keyed operations like `dbSeek()`.

Common primitives:

* Add an index file to the order list:

  ```harbour
  ordListAdd( cPath + "customer.cdx" )
  ```

* Set controlling order (by tag name or numeric order):

  ```harbour
  local l_cPrevOrder := ordSetFocus( "CITY" )
  ```

* Clear the order list / indexes:

  ```harbour
  ordListClear()
  // or:
  dbClearIndex()
  ```

* Rebuild indexes (maintenance):

  ```harbour
  ordListRebuild()
  ```

Canonical seek pattern (with explicit order discipline):

```harbour
select Cust

ordListAdd( cPath + "customer.cdx" )

local l_cPrevOrder := ordSetFocus( "CITY" )

if dbSeek( "Seattle" )
    ? Cust->Name
endif

ordSetFocus( l_cPrevOrder )
```

Rules that prevent subtle bugs:

* Do not call `dbSeek()` unless you have explicitly selected the correct controlling order.
* Do not assume the current order is “natural record order”; it is whatever the code (or upstream caller) last set.

---

### 18.9 “Scan” style operations: core RDD vs ORM syntax

At the RDD level, `dbEval()` is the canonical “evaluate a block for records” primitive (often used like a scan). When Harbour_ORM adds higher-level `scan ... endscan`, it is conceptually layered on top of the same workarea mechanics (record pointer, filter, order).

If you use `dbEval()`, be explicit about the assumed order/filter context, and do not leak selection changes outside the routine.

---

### 18.10 Locking and concurrency (shared DBF scenarios)

DBF/RDD concurrency is based on explicit locks:

* Record locking (typical before update):

  ```harbour
  if dbRLock()
      // update fields
      dbCommit()
      dbUnlock()
  endif
  ```

* File locking is possible (`dbLock()`), but should be used sparingly and only for tooling/maintenance workflows.

Rules:

* Never update shared tables without an explicit lock strategy.
* Keep lock scope minimal: lock → update → commit → unlock.

---

### 18.11 Multithreading: detaching and restoring workareas

In multi-threaded programs, you must not assume a workarea context can be freely used across threads.

When you need to move or “lend” a workarea between threads, use the detach/request mechanism:

* `hb_dbDetach()` to detach a workarea handle from the current thread/context.
* `hb_dbRequest()` to restore/request it in another context.

If your project uses a higher-level abstraction, document it there—but the underlying rule remains:

> Workareas are stateful; treat them as thread-affine unless explicitly detached/requested.

---

### 18.12 AI must never generate

The following are hard constraints for AI-generated Harbour code:

* **Must not** change the selected workarea inside a helper routine without saving/restoring the previous area. Use a `Used()`-guarded save when appropriate:

  * `l_nPrevArea := iif( Used(), Select(), 0 )`
  * `select ( l_nPrevArea )` on exit.

* **Must not** confuse `Select()` (function) with `select` (command). `Select()` does not select anything; it only returns a workarea number.

* **Must not** rely on unqualified field references (`Field`) in reusable code when an explicit alias form (`Alias->Field`) or aliased evaluation (`Alias->( … )`) avoids ambient-state bugs.

* **Must not** use `"M"` as a workarea alias in project code; `M->` / `MEMVAR->` are reserved aliases for memory variables. When a field/memvar name conflict is possible, use `FIELD->` and `MEMVAR->` explicitly.

* **Must not** open free tables using ambiguous relative paths in application/service code. Prefer full paths (or an explicitly documented legacy path-resolution layer).

* **Must not** assume a materialized in-memory table can be mounted into multiple workareas. If a second cursor is needed, materialize/clone explicitly.

* **Must not** call `__dbZap()` or `__dbPack()` in application logic. These are destructive maintenance operations and may only appear in explicitly labeled admin/maintenance tools with prominent warnings.

* **Must not** perform `dbSeek()` without explicitly selecting/setting the controlling order (`ordSetFocus()`), and must not assume a previously-set order remains in effect.

* **Must not** clear filters, relations, or order lists “globally” (`dbClearFilter()`, `dbClearRelation()`, `ordListClear()`, `dbCloseAll()`) unless the routine’s contract is explicitly to reset state (tooling/shutdown).

* **Must not** update shared tables without an explicit lock strategy; must not unlock before `dbCommit()` when immediate visibility is required.

* **Must not** assume workareas are safely shared across threads; must not pass workarea-dependent logic across threads without `hb_dbDetach()` / `hb_dbRequest()` (or an approved project abstraction).

---

> Guidance: In modern apps, avoid free tables for core business data. Prefer a SQL server (e.g., PostgreSQL) with an ORM for transactional integrity, concurrency, schema evolution, and corruption resistance. Use DBF/ISAM workareas primarily for legacy compatibility, offline/niche workflows, or controlled local tooling.

---

## 19) Error handling

This section defines the canonical error/exception mechanisms used in Harbour-based codebases:

* **Structured containment**: `begin sequence / recover / end`
* **Non-local escape into `recover`**: `Break( xValue )` (and any `break` *syntax sugar* if your codebase provides it)
* **Handler installation**: `ErrorBlock( bHandler )`

It also documents the **Harbour_EL** `try/catch/endtry` syntax as an **optional extension**.

Primary goal for AI-generated code: **no silent failures**, **predictable control flow**, and **resource-safe cleanup**.

---

### 19.1 Building blocks and how they relate

There are two distinct layers:

1. **Structured control-flow layer**: `begin sequence ... recover ... end`

   This is the containment scope. Control reaches `recover` in two main ways:
   * A **runtime error** occurs while a sequence is active (the runtime creates an Error object).
   * Code explicitly calls **`Break( xValue )`** to abort into the nearest enclosing `recover`.

2. **Error interception layer**: `ErrorBlock( bHandler )`

   `ErrorBlock()` installs a handler that is invoked when a runtime error is raised. It is orthogonal to `begin sequence`: it can be used to log, normalize, or redirect errors. In server code it is common to use the errorblock only for **logging** and then **re-raise into `recover`** so the *control flow remains sequence-based*.

Optional syntax layer:

* **Harbour_EL `try/catch/endtry`** — optional syntax (typically implemented via includes/xcommands). Generate it only when the target project is explicitly Harbour_EL-enabled.

---

### 19.2 Structured containment with `begin sequence / recover`

Canonical pattern:

```harbour
function DoWork()
    local l_xResult := NIL
    local l_xCaught := NIL

    begin sequence
        l_xResult := RiskyOperation()
    recover using l_xCaught
        // l_xCaught may be an Error object OR any value passed to Break(x).
        HandleFailure( l_xCaught )
        return NIL
    end

    return l_xResult
```

Semantics that matter:

* `recover using <var>` receives the recovered value (the “reason”):

  * A runtime error usually yields an **Error object**
  * `Break( xValue )` yields exactly the value you passed (string/number/hash/object/etc.)
* Therefore: **do not assume** `l_xCaught` has `:description`. It might be `"ValidationFailed"`.

---

### 19.3 `Break()` is not loop control (`exit` / `loop`)

Do not confuse these families:

* **Loop control**

  * `exit` leaves the current loop
  * `loop` continues the current loop
* **Non-local escape into `recover`**

  * `Break( xValue )` transfers control to the nearest enclosing `recover` in an active `begin sequence`.

Validation “fail fast” example:

```harbour
function SaveRecord(par_xPayload)
    local l_xCaught := NIL

    begin sequence
        if empty( par_xPayload )
            Break( "ValidationFailed" )
        endif

        PersistPayload( par_xPayload )

    recover using l_xCaught
        // l_xCaught == "ValidationFailed" in this path
        return .f.
    end

    return .t.
```

Notes:

* `Break()` requires an argument; pass `NIL` explicitly if you want “no payload”.
* If there is **no enclosing** `begin sequence`, do not generate `Break()` as a “flow control” shortcut. Treat it as a bug.

---

### 19.4 Installing and restoring handlers with `ErrorBlock()`

`ErrorSys()` installs the default error handler at startup. Custom handlers are typically installed via `ErrorBlock()`.

#### Why you often see `ErrorBlock()` combined with `begin sequence`

This is intentional and is about **state restoration**:

* `ErrorBlock()` changes a global (or at least broader-than-local) handler state.
* If protected work triggers a runtime error (or calls `Break()`), control jumps to `recover`.
* Therefore, use `begin sequence / recover` to **guarantee restoration** even on non-local exits.

Canonical “save / set / restore” wrapper:

```harbour
function WithErrorHandler(par_bWork)
    local l_bPrev   := NIL
    local l_xResult := NIL
    local l_xCaught := NIL

    l_bPrev := ErrorBlock( {|par_oErr| GlobalErrorHandler( par_oErr ) } )

    begin sequence
        l_xResult := Eval( par_bWork )
    recover using l_xCaught
        ErrorBlock( l_bPrev )
        Break( l_xCaught )   // re-raise unless you intentionally consume it
    end

    ErrorBlock( l_bPrev )
    return l_xResult
```

Guidelines:

* Never install a new `ErrorBlock()` without restoring the previous one.
* In long-running server processes, keep handlers minimal (log/annotate) and re-raise unless you explicitly implement a fallback.

---

### 19.5 Recommended containment patterns

#### Pattern A: Convert error to return-code

```harbour
function TryOpenFile(par_cPath)
    local l_hFile   := NIL
    local l_xCaught := NIL

    begin sequence
        l_hFile := FOpen( par_cPath )
        if l_hFile < 0
            Break( "OpenFailed" )
        endif
    recover using l_xCaught
        return NIL
    end

    return l_hFile
```

#### Pattern B: Log and re-raise (do not swallow)

```harbour
function DoNetworkCall()
    local l_xCaught := NIL

    begin sequence
        CallRemote()
    recover using l_xCaught
        LogFailure( l_xCaught )
        Break( l_xCaught )
    end

    return NIL
```

---

### 19.6 Harbour_EL `try/catch/endtry` (optional syntax)

Some projects (including those using Harbour_EL) provide `try/catch` syntax via includes/xcommands.

Rules for AI-generated code:

* Use `try/catch` only when the target project explicitly depends on Harbour_EL and enables the syntax.
* Otherwise, generate the core form (`begin sequence / recover / end`) because it is portable and unambiguous.

Correct terminator: `endtry`.

```harbour
function ExampleTryCatch()
    local l_xCaught := NIL

    try
        RiskyOperation()
    catch l_xCaught
        HandleFailure( l_xCaught )
        return .f.
    endtry

    return .t.
```

---

### 19.7 Error object received by `recover using` / `catch`: canonical properties

When a runtime error is raised (or when code explicitly `Break()`s an Error object), the value you receive is typically an **Error object**. The canonical Error object fields you can rely on are:

* `:args` (array or NIL)
  Arguments related to the failure (e.g., invalid operands or parameters), when provided.

* `:canDefault` (logical)
  Indicates the operation could continue using a default action/value.

* `:canRetry` (logical)
  Indicates retrying the operation may be possible.

* `:canSubstitute` (logical)
  Indicates the handler may substitute a return value and continue.

* `:cargo` (any)
  User-defined payload for application-specific tagging/annotation.

* `:description` (string)
  Human-readable message.

* `:filename` (string)
  Related file name when the failure is file-oriented.

* `:genCode` (numeric)
  General error code (broad classification).

* `:operation` (string)
  The operation name, if known (e.g., `"++"`, `"dbUseArea"`, `"hb_compilefrombuf"`, etc.).

* `:osCode` (numeric)
  OS error code (when applicable).

* `:severity` (numeric)
  Severity level classification.

* `:subCode` (numeric)
  More specific error code within the subsystem.

* `:subSystem` (string)
  Subsystem identifier (e.g., `"BASE"`, `"DBFNTX"`, `"RDD"`, etc.), when provided.

* `:tries` (numeric)
  Retry counter used by some handlers.

Practical rules:

* Always defensively handle the case where the caught value is **not** an object (because `Break( xValue )` can pass anything).
* Even when it *is* an Error object, some fields may be empty or `NIL`.

Minimal safe “summarize error” helper pattern:

```harbour
function SummarizeCaughtValue(par_xCaught)
    local l_c := ""

    if ValType( par_xCaught ) == "O"
        l_c += "subsystem=" + alltrim( par_xCaught:subsystem ) + " "
        l_c += "gencode="   + alltrim( Str( par_xCaught:gencode ) ) + " "
        l_c += "subcode="   + alltrim( Str( par_xCaught:subcode ) ) + " "
        l_c += "oscode="    + alltrim( Str( par_xCaught:oscode ) ) + " "
        l_c += "op="        + alltrim( par_xCaught:operation ) + " "
        l_c += "desc="      + alltrim( par_xCaught:description )
    else
        l_c := "caught=" + hb_ValToStr( par_xCaught )
    endif

    return l_c
```

---

### 19.8 Canonical logging ErrorBlock (one-line file log + minimal `OutErr()`, then re-raise into `recover`)

Goal:

* Log a compact one-line record to a file (append).
* Emit a minimal stderr line via `OutErr()` (server-friendly).
* Re-raise into the nearest `recover` so control flow remains `begin sequence`-based.

```harbour
#include "fileio.ch"

static procedure AppendLogLine(par_cLogFile, par_cLine)
    local l_hFile := FOpen( par_cLogFile, FO_READWRITE + FO_SHARED )
    if l_hFile < 0
        l_hFile := FCreate( par_cLogFile, FC_NORMAL )
    endif

    if l_hFile >= 0
        FSeek( l_hFile, 0, FS_END )
        FWrite( l_hFile, par_cLine + hb_eol() )
        FClose( l_hFile )
    endif
return

static function OneLineError(par_oErr)
    // Keep this minimal: errorblocks must avoid doing “risky” work.
    return hb_ValToStr( hb_datetime() ) + ;
           " | " + alltrim( par_oErr:subsystem ) + ;
           " | " + alltrim( Str( par_oErr:gencode ) ) + ;
           " | " + alltrim( Str( par_oErr:subcode ) ) + ;
           " | " + alltrim( Str( par_oErr:oscode ) ) + ;
           " | " + alltrim( par_oErr:operation ) + ;
           " | " + alltrim( par_oErr:description )

function WithCanonicalLogging(par_bWork, par_cLogFile)
    local l_bPrev   := NIL
    local l_xResult := NIL
    local l_xCaught := NIL

    l_bPrev := ErrorBlock( {|par_oErr|
        local l_cLine := OneLineError( par_oErr )

        // 1) File log (best-effort)
        AppendLogLine( par_cLogFile, l_cLine )

        // 2) Minimal stderr signal (best-effort)
        OutErr( l_cLine )

        // 3) Convert runtime error into sequence-based control flow
        Break( par_oErr )
    } )

    begin sequence
        l_xResult := Eval( par_bWork )
    recover using l_xCaught
        ErrorBlock( l_bPrev )
        Break( l_xCaught )   // re-raise to caller's sequence (or let caller translate to return-code)
    end

    ErrorBlock( l_bPrev )
    return l_xResult
```

Usage pattern:

```harbour
function Main()
    local l_xCaught := NIL

    begin sequence
        WithCanonicalLogging( {|| RiskyOperation() }, "server_errors.log" )
    recover using l_xCaught
        // l_xCaught is typically the Error object re-raised by the wrapper.
        return .f.
    end

    return .t.
```

---

### 19.9 Unhandled errors, diagnostic logs, and error I/O

#### If you do nothing, does Harbour “dump to a file”?

For normal runtime errors handled by the default `ErrorSys()` handler, behaviour is typically interactive/console-oriented, and file logging is not guaranteed. If you need deterministic logging, install it explicitly (e.g., via the canonical ErrorBlock above).

Separately, Harbour supports logging for certain internal/unrecoverable conditions (e.g., low-level faults / internal errors) that can be directed to a file via runtime settings (commonly discussed as `hb_out.log`-style logging). Treat this as a separate mechanism from application-level exception handling.

#### Does Harbour ever write to stderr?

Yes—`OutErr()` writes to the standard error device (stderr). If you want server-friendly log capture, emit a single compact `OutErr()` line in your handler and rely on external process supervisors/log collectors to capture it.

---

### 19.10 What AI must not generate

* Do not use `Break()` (or any `break` sugar) to exit loops. Use `exit` / `loop`.
* Do not assume `recover using l_oErr` is always an object with `:description` (it may be any type when `Break( x )` is used).
* Do not install an `ErrorBlock()` without restoring the previous one.
* Do not swallow errors silently. If you consume an error, do it intentionally and return a clear signal (`.f.`, `NIL`, or a structured result).
* Do not generate Harbour_EL `try/catch/endtry` unless the project is explicitly Harbour_EL-enabled.

---

### 19.11 ErrorBlock in multi-threaded programs (thread-local)

Harbour maintains **thread-local runtime state**. In multi-threaded programs, you must treat the **error block** as **thread-local**, not process-global:

* A new thread may inherits some configuration (codepage, language, SETs, default RDD), but its **error block is initialized to the default handler** (via `ErrorSys()`).
* Therefore, a custom `ErrorBlock()` installed in one thread does **not** automatically apply to other threads.
* If you want consistent logging and consistent “re-raise into `recover`” control-flow, install your handler inside each thread entry routine.

This is especially important in server-style applications: thread failures must be logged deterministically, and thread code should still be structured around `begin sequence / recover / end`.

#### Canonical thread entry wrapper (per-thread ErrorBlock + begin sequence)

```harbour
#include "fileio.ch"

// Install a per-thread ErrorBlock() that logs and Break()s into the nearest RECOVER.
// Returns the previous error block so it can be restored.
static function Thread_InstallErrorBlock(par_cLogFile, par_pLogMutex)
    local l_bPrev := ErrorBlock()

    ErrorBlock( {|par_oErr| CanonicalLogAndBreak( par_oErr, par_cLogFile, par_pLogMutex ) } )

    return l_bPrev

// Minimal, server-friendly handler:
//  1) emit one stderr line (OutErr())
//  2) append one line to a shared log file (mutex + timeout)
//  3) Break(oErr) so control flow remains begin-sequence based
static function CanonicalLogAndBreak(par_oError, par_cLogFile, par_pLogMutex)
    local l_cOneLine := MakeOneLineError( par_oError )

    OutErr( l_cOneLine + hb_eol() )

    // Best-effort. Timeout avoids deadlocks if the logger mutex is contended.
    AppendLogLine( par_cLogFile, l_cOneLine, par_pLogMutex )

    Break( par_oError )

    return NIL

static function MakeOneLineError(par_oError)
    local l_cLine := ""
    local l_xTid := hb_threadID()

    l_cLine := hb_ValToStr( hb_datetime() ) + ;
        " tid=" + hb_ValToStr( l_xTid ) + ;
        " g=" + hb_ValToStr( par_oError:gencode ) + ;
        "/" + hb_ValToStr( par_oError:subcode ) + ;
        " os=" + hb_ValToStr( par_oError:oscode ) + ;
        " " + alltrim( hb_ValToStr( par_oError:subsystem ) ) + ;
        " " + alltrim( hb_ValToStr( par_oError:description ) )

    if ! Empty( par_oError:operation )
        l_cLine += " op=" + alltrim( hb_ValToStr( par_oError:operation ) )
    endif

    if ! Empty( par_oError:filename )
        l_cLine += " file=" + alltrim( hb_ValToStr( par_oError:filename ) )
    endif

    // Normalize to a single physical line
    l_cLine := StrTran( StrTran( l_cLine, Chr(13), " " ), Chr(10), " " )

    return l_cLine

static function AppendLogLine(par_cLogFile, par_cLine, par_pLogMutex)
    local l_nH := -1
    local l_lLocked := .f.

    // If you have no mutex, you can pass NIL and accept interleaving.
    if Empty( par_pLogMutex )
        l_lLocked := .t.
    else
        l_lLocked := hb_mutexLock( par_pLogMutex, 250 )  // timeout in ms
    endif

    if l_lLocked
        l_nH := FOpen( par_cLogFile, FO_READWRITE + FO_SHARED )
        if l_nH < 0
            l_nH := FCreate( par_cLogFile, FC_NORMAL )
        endif

        if l_nH >= 0
            FSeek( l_nH, 0, FS_END )
            FWrite( l_nH, par_cLine + hb_eol() )
            FClose( l_nH )
        endif

        if ! Empty( par_pLogMutex )
            hb_mutexUnlock( par_pLogMutex )
        endif
    endif

    return NIL

// Thread entry that guarantees:
// - per-thread error block
// - begin sequence / recover boundary
// - restoration of previous error block
function Thread_RunWithErrors(par_bWork, par_cLogFile, par_pLogMutex)
    local l_bPrev := NIL
    local l_xResult := NIL
    local l_xCaught := NIL

    l_bPrev := Thread_InstallErrorBlock( par_cLogFile, par_pLogMutex )

    begin sequence
        l_xResult := Eval( par_bWork )
    recover using l_xCaught
        // l_xCaught is whatever was passed to Break( x ); often an Error object.
        // Decide if this thread should swallow, translate, or re-raise.
        // Break( l_xCaught )
        l_xResult := NIL
    end

    ErrorBlock( l_bPrev )

    return l_xResult

procedure Example_StartThread()
    local l_pLogMutex := hb_mutexCreate()
    local l_pTh

    l_pTh := hb_threadStart( @Thread_RunWithErrors(), {|| Risky() }, "server_errors.log", l_pLogMutex )

    hb_threadJoin( l_pTh )

    return
```

Practical notes:

* Prefer a **timeout** in `hb_mutexLock()` inside the error handler to reduce deadlock risk.
* Keep the handler minimal. Do not perform complex work in the `ErrorBlock()` itself; log and `Break()`.
* Do not rely on “implicit propagation” of a thread error to the parent thread. Instead, return a structured result, push to a shared queue/channel, or signal via your framework.

---

## 20) Legacy pseudo-commands from `std.ch` beyond `set` (scope-limited)

Harbour supports many xBase-style **pseudo-commands** implemented as preprocessor rules (`#command` / `#xcommand`) defined primarily in `include/std.ch`. These exist for both compatibility and developer ergonomics.

This section complements **§17 (SET pseudo-commands)** by documenting commonly encountered **non-`set`** pseudo-commands from `std.ch`, with their typical runtime equivalents.

---

### 20.1 Scope boundary (what is and is not documented)

**Documented here (in-scope):**

- Output shorthands (`?`, `??`).
- Workarea / DB operations that frequently appear in modern Harbour code:
  `select`, `use`, `close`, `go`, `goto`, `skip`, `seek`, `locate`,
  `append`, `replace`, `delete`, `recall`, `commit`, `unlock`,
  `set filter`, `set relation`, `set order`,
  `index on`, `reindex`, `delete tag`.
- A small subset of file-related pseudo-commands commonly seen in legacy code: `erase`, `rename`, `copy file`, `delete file`.

**Intentionally out of scope unless explicitly requested:**

- Full-screen console UI / “GET/READ”-era constructs: `@ ... say/get`, `read`, `menu to`, `accept`, `wait`, screen save/restore.
- Report and listing commands: `report form`, `label form`, `display`, `list`, `browse`.
- Shell/process commands (`run`, `!`, etc.) in server code.

---

### 20.2 Pseudo-command vs function form (both are valid)

Many Harbour developers prefer pseudo-commands; others prefer explicit function calls. **Both forms are valid Harbour.**

**Rule for AI-generated code:**

- Follow the project’s declared preference, as specified in:
  - `personal_coding_standards` (developer preference), or
  - `application_coding_standards` (application/team preference).
- If no preference is declared, either form may be used, but code must remain **unambiguous** about workarea/alias context and side effects.

---

### 20.3 Workareas, aliases, and explicit execution context

Pseudo-commands often act on the **current workarea**. To make intent explicit (and to avoid accidental workarea changes), Harbour supports several canonical patterns.

#### 20.3.1 Select then call (explicit, global current area)

```harbour
select Customer
go top

// function form
dbSelectArea( "Customer" )
dbGoTop()
```

#### 20.3.2 Alias-qualified evaluation (`Alias->( ... )`)

```harbour
Customer->( dbGoTop() )
Customer->( dbSkip( 1 ) )
```

This does not require changing the current workarea.

#### 20.3.3 Expression-qualified evaluation (`(<expr>)->( ... )`)

When the target workarea is only known at runtime, use an expression that resolves to a workarea context (commonly via `select()`):

```harbour
( select( par_cAlias ) )->( dbCloseArea() )
( select( par_cAlias ) )->( dbGoTop() )
( select( par_cAlias ) )->( dbSkip( 1 ) )
```

This pattern is preferred when you must operate on a variable alias/workarea without disturbing the caller’s current selection.

---

### 20.4 Common pseudo-commands and typical runtime equivalents

This section lists high-signal pseudo-commands defined in `std.ch` and the corresponding runtime calls that represent the same intent. The exact preprocessor expansion is defined in `std.ch`, but the function forms below are the stable, readable equivalents.

#### 20.4.1 Output shorthands

```harbour
?  "Hello", 123
?? "No newline"

// function form
QOut( "Hello", 123 )
QQOut( "No newline" )
```

#### 20.4.2 Selecting a workarea

```harbour
select Customer
select 3

// function form
dbSelectArea( "Customer" )
dbSelectArea( 3 )
```

`std.ch` also supports functional selection patterns such as:

```harbour
select SomeFunc( par_x )
```

which is equivalent to:

```harbour
dbSelectArea( SomeFunc( par_x ) )
```

#### 20.4.3 Opening and closing (`use`, `close`)

```harbour
use Customer alias Customer shared
use
close
close databases

// common function forms
dbUseArea( .f., NIL, "Customer", "Customer", .t., .f. )
dbCloseArea()
dbCloseArea()
dbCloseAll()
```

Notes:

- `use` has many option combinations (RDD driver, `new`, `alias`, `shared/exclusive`, codepage). If you need deterministic behavior, use `dbUseArea()` directly and pass explicit flags.
- Workarea-safe close (runtime alias/workarea):

```harbour
( select( par_cAlias ) )->( dbCloseArea() )
```

#### 20.4.4 Navigation (`go`, `goto`, `skip`)

```harbour
go top
go bottom
goto 100
skip
skip -1

// function form
dbGoTop()
dbGoBottom()
dbGoto( 100 )
dbSkip()
dbSkip( -1 )
```

Alias-targeted skip is also commonly used:

```harbour
skip alias Customer
Customer->( dbSkip() )
```

#### 20.4.5 Seek and locate

```harbour
seek par_xKey
// function form
dbSeek( par_xKey )
```

`locate` is macro-driven in `std.ch` and typically expands to an internal helper call. For AI-generated code, prefer an explicit loop when readability and control-flow transparency matter:

```harbour
dbGoTop()
do while ! Eof()
    if Amount > 100
        exit
    endif
    dbSkip()
enddo
```

#### 20.4.6 Filter and relation (note: these are not `Set()` indices)

These pseudo-commands begin with `set` but are implemented as DB-state operations, not as `Set( _SET_* )` runtime indices.

```harbour
set filter to Amount > 0
set filter to
set relation to
set relation to CustomerId into Customer additive

// function form
dbSetFilter( {|| Amount > 0 }, "Amount > 0" )
dbClearFilter()

dbClearRelation()
dbSetRelation( "Customer", {|| CustomerId }, "CustomerId", .f. )
```

#### 20.4.7 Record modifications (`append`, `replace`, `delete`, `recall`)

```harbour
append blank
// function form
dbAppend()
```

`replace` has two major forms in `std.ch`:

- A multi-record form that expands to `dbEval()` and uses a field-qualified alias internally.
- A simple single-record form that assigns directly to fields.

Example intent-equivalents:

```harbour
replace LastName with Upper( LastName )

// function-style, single-record intent
FIELD->LastName := Upper( FIELD->LastName )
```

Bulk operations (FOR/WHILE/NEXT/REST/ALL) for `replace`, `delete`, and `recall` expand to `dbEval()` in `std.ch`. Use them when you intentionally want a `dbEval()`-style traversal; otherwise, prefer an explicit loop for clarity.

```harbour
delete
recall

// function form
dbDelete()
dbRecall()
```

#### 20.4.8 Commit and unlock

```harbour
commit
unlock
unlock all

// function form
dbCommitAll()
dbUnlock()
dbUnlockAll()
```

#### 20.4.9 Destructive maintenance (`pack`, `zap`)

```harbour
pack
zap
```

These pseudo-commands expand to helper calls in `std.ch` (commonly `__dbPack()` / `__dbZap()`).

**AI must not generate these by default.** They are destructive operations and must only be produced when explicitly requested and context confirms that the operational impact is acceptable.

#### 20.4.10 Index and order maintenance (high impact)

Common pseudo-commands:

```harbour
set order to LastName
index on LastName tag LastName to "customer.cdx"
reindex
delete tag LastName
```

Typical runtime equivalents use the `ord*()` API family (exact calls depend on index driver and configuration):

- `set order to ...` maps to `ordSetFocus( ... )`
- `reindex` maps to `ordListRebuild()`
- `index on ...` and `delete tag ...` map to `ordCreate()` / `ordDestroy()` variants

**AI must not generate index creation/rebuild/destruction by default.** These operations are environment-dependent and can be expensive or destructive.

---

### 20.5 File-oriented pseudo-commands (legacy compatibility)

These are simple shorthands and are often encountered in older code:

```harbour
erase "file.txt"
delete file "file.txt"
rename "a.txt" to "b.txt"
copy file "a.txt" to "b.txt"

// function form
FErase( "file.txt" )
FErase( "file.txt" )
FRename( "a.txt", "b.txt" )
__CopyFile( "a.txt", "b.txt" )
```

**AI guidance:** avoid shell/process pseudo-commands (`run`, `!`, etc.) in server code unless explicitly requested.

---

### 20.6 What AI must not generate by default (unless explicitly requested)

- Full-screen console UI workflows: `@ ... say/get`, `read`, `menu to`, `accept`, `wait`, screen save/restore.
- Report/listing commands: `report form`, `label form`, `display`, `list`, `browse`.
- Destructive DB maintenance: `pack`, `zap`.
- Index creation/rebuild/destruction: `index on`, `reindex`, `delete tag`.

---

## 21) System capacities (practical notes)

This section captures practical “shape limits” and resource ceilings that matter when generating Harbour code. Some limits are enforced by the compiler/VM; others are OS- or build-dependent.

### 21.1 Known hard ceilings (VM / compiler)

* **Global symbol table:** up to ~4.29e9 symbols (VM).
* **Workareas:** up to **65,534**.
* **Locals per function:** up to **32,767**.
* **Static vars per module:** up to **65,535**.
* **Literal string size in PRG:** up to **16 MB** per single literal.
* **Literal array length in PRG:** up to **65,535** elements per single literal array.
* **All practical limits then depend on compiler/OS/memory.**

(These are the canonical limits already established in this document.)

### 21.2 “Hard limit” vs “you will run out of RAM first”

AI engines should distinguish:

* **Compile-time structure limits** (you will get compiler errors or codegen failures):

  * Too many locals in one routine.
  * Literal constants that are too large (string/array literal limits).
  * Excessive per-module static declarations.
* **Runtime resource limits** (code compiles, but fails under load):

  * Memory pressure (large arrays/hashes/objects, large memo/JSON payloads).
  * OS handle limits (files, sockets).
  * Per-thread stack/VM overhead in multi-threaded services.
  * Many simultaneously open workareas/tables (especially with indexes/memos).

### 21.3 Practical guidance for AI-generated code

**A) Keep routines small enough to stay well below local/static ceilings**

* Do not generate routines with hundreds/thousands of locals “just because it’s convenient”.
* Prefer:

  * small helper functions,
  * a hash/array “context” structure,
  * objects for cohesive state,
  * `static` helpers where reuse matters, but do not accumulate enormous static state in one PRG.

**B) Avoid huge literals; build or load data instead**

* Do not generate:

  * megabyte-scale inline JSON literals,
  * enormous literal arrays (thousands of elements) as “configuration”.
* Prefer:

  * reading from a file/resource,
  * assembling strings incrementally,
  * generating arrays/hashes with loops (and documenting intent).

**C) Workareas are finite and are a global shared resource**

* In server-style applications and tooling:

  * explicitly manage open/close of tables,
  * minimize the number of concurrently open aliases,
  * avoid “leaking” workareas (opening without closing).
* Prefer explicit alias usage when many tables are in play (clarity and safety), especially in larger routines.

**D) Assume concurrency magnifies resource limits**

* A design that is safe in a single-thread tool may fail in a multi-threaded server due to:

  * duplicated per-thread state,
  * per-thread DB/workarea usage patterns,
  * increased file/socket handles.
* Guidance for AI: do not “fan out threads” for trivial work unless there is a clear requirement.

### 21.4 Failure modes to expect (and what AI should do instead)

When approaching these limits, typical remedies are structural—not “try again”:

* **Too many locals:** split the routine; move cohesive state into a hash/object; use helper routines.
* **Static sprawl in one PRG:** split into multiple modules; reduce persistent global state.
* **Literal too large:** store externally; load at runtime; build programmatically.
* **Too many open workareas:** close earlier; reuse aliases; process tables in batches.

### 21.5 Explicit constraints for AI-generated code in this section’s domain

AI-generated Harbour code must **not**:

* Emit extremely large inline literals (multi-MB strings / massive literal arrays) as a default tactic.
* Generate single “god routines” with unusually high local counts (hundreds+) without a compelling reason.
* Open tables/workareas in loops without a clear close/reuse strategy.
* Create concurrency patterns that multiply resource usage (threads/workareas/handles) unless explicitly requested.

---

## 22) Database & ORM (orientation)

This section orients AI-generated Harbour code around **data access**, including classic xBase RDD workareas and **SQL backends** via an ORM that materializes results as **in-memory cursors/workareas**.

---

### 22.1 Two worlds: xBase RDD workareas vs SQL backends

**xBase RDDs (DBF/NTX/NDX/CDX/…):**

* Data lives in **workareas** (aliases). Navigation and state are implicit: record pointer, current index order, filters, relations.
* Semantics vary by RDD and configuration (memos, index behavior, NULL-like values, locking, etc.).

**SQL backends (PostgreSQL/MariaDB/MySQL/…):**

* Prefer an ORM layer that can:

  * build SQL safely,
  * execute via a connection,
  * **materialize result sets into in-memory workareas** (cursor aliases) so the rest of the application can use standard Harbour workarea semantics.

---

### 22.2 Workarea field addressing and name collisions

In Harbour, name collisions between **memvars** and **fields** are a recurring source of subtle bugs, especially when working with cursors produced by an ORM.

Rules to generate robust code:

* Prefer **explicit aliasing** when reading fields:
  `<alias>->FieldName`
* Use `FIELD->FieldName` when you explicitly want “field of current workarea” and might otherwise collide with a variable name.
* Use `M->VarName` for memvars (PUBLIC/PRIVATE) when collisions are possible.

Practical guidance:

* Avoid PUBLIC/PRIVATE memvars for application state when working heavily with cursors.
* Do not reuse the same identifier for both a local variable and a cursor field in the same routine.

---

### 22.3 Harbour_ORM orientation (what it is, what it returns)

Available from: https://github.com/EricLendvai/Harbour_ORM   
Harbour_ORM is designed around these core concepts:

* A **connection object** that owns the ODBC/SQL connection and backend configuration.
* A **query builder** that composes FROM/JOIN/WHERE/GROUP/ORDER/LIMIT and executes.
* Result sets are typically materialized as **in-memory cursors** (workareas), not entity objects.

Key consequence for AI-generated code:

* After `:SQL( "SomeCursor" )`, your code should treat `"SomeCursor"` like any other workarea:

  * `select SomeCursor`
  * `dbGoTop()`, `scan`, `reccount()`
  * `SomeCursor->FieldName`

---

### 22.4 Canonical pattern: connect → SELECT into a cursor → scan results

```harbour
function DemoOrmSelect()
local l_oConn
local l_oDB

l_oConn := hb_orm_SQLConnect():new()

l_oConn:SetBackendType( "PostgreSQL" )
l_oConn:SetServer( "localhost" )
l_oConn:SetPort( 5432 )
l_oConn:SetUser( "postgres" )
l_oConn:SetPassword( "secret" )
l_oConn:SetDatabase( "datawharf" )

// Optional (PostgreSQL identifier casing behavior):
// l_oConn:PostgreSQLIdentifierCasing := HB_ORM_POSTGRESQL_CASE_ALL_LOWER

if ! l_oConn:Connect()
    ? l_oConn:GetErrorMessage()
    return NIL
endif

l_oDB := hb_orm_SQLData():new()
l_oDB:UseConnection( l_oConn )

l_oDB:Table( "E1", "public.Table001", "t1" )
l_oDB:Column( "t1.pk",    "pk" )
l_oDB:Column( "t1.fname", "fname" )
l_oDB:Column( "t1.lname", "lname" )

// Prefer ^ placeholders over concatenating values into SQL strings.
l_oDB:Where( "t1.pk = ^", 123 )

// Direction is interpreted by first character:
// if Upper( Left( cDirection, 1 ) ) == "A" => ascending, otherwise descending.
l_oDB:OrderBy( "t1.pk", "ascending" )

if ! l_oDB:SQL( "Table001Cursor" )
    ? l_oDB:ErrorMessage()
    l_oConn:Disconnect()
    return NIL
endif

select Table001Cursor
dbGoTop()
scan
    ? Table001Cursor->pk, Table001Cursor->fname, Table001Cursor->lname
endscan

// Close cursor/workarea when done (recommended).
(select( "Table001Cursor" ))->( dbCloseArea() )

l_oConn:Disconnect()

return NIL
```

---

### 22.5 Safe value substitution: the `^` placeholder convention

Many Harbour_ORM expression builders accept:

* an expression string containing one or more `^` placeholders, plus
* one or more values that will be encoded safely for the backend.

Example:

```harbour
l_oDB:Where( "t1.fname = ^", "Jodie" )
```

AI-generation rule:

* Do **not** build SQL by concatenating raw user input into SQL strings when a `^` placeholder can be used.

---

### 22.6 Canonical CRUD pattern (FieldValue + Add/Update/Delete)

Harbour_ORM’s CRUD flow is driven by:

* selecting a table via `:Table()`,
* setting fields via `:FieldValue()` / `:FieldExpression()` / `:FieldArray()`,
* calling `:Add()`, `:Update()`, or `:Delete()`.

```harbour
function DemoOrmCrud()
local l_oConn := hb_orm_SQLConnect():new()
local l_oDB

l_oConn:SetBackendType( "PostgreSQL" )
l_oConn:SetServer( "localhost" )
l_oConn:SetPort( 5432 )
l_oConn:SetUser( "postgres" )
l_oConn:SetPassword( "secret" )
l_oConn:SetDatabase( "datawharf" )

if ! l_oConn:Connect()
    ? l_oConn:GetErrorMessage()
    return NIL
endif

l_oDB := hb_orm_SQLData():new()
l_oDB:UseConnection( l_oConn )

// Insert
l_oDB:Table( "E10", "public.Table001", "t1" )
l_oDB:FieldValue( "fname", "Alice" )
l_oDB:FieldValue( "lname", "Ng" )

if ! l_oDB:Add()
    ? l_oDB:ErrorMessage()
    l_oConn:Disconnect()
    return NIL
endif

// Update (by primary key)
l_oDB:Table( "E11", "public.Table001", "t1" )
l_oDB:FieldValue( "lname", "Nguyen" )

if ! l_oDB:Update( 123 )
    ? l_oDB:ErrorMessage()
    l_oConn:Disconnect()
    return NIL
endif

// Delete (event id + table + key)
if ! l_oDB:Delete( "E12", "public.Table001", 123 )
    ? l_oDB:ErrorMessage()
    l_oConn:Disconnect()
    return NIL
endif

l_oConn:Disconnect()

return NIL
```

Notes for generated code:

* Use `:ErrorMessage()` for diagnostics on failure.
* Use `:Tally` (when relevant) to confirm affected row counts.
* Prefer stable, human-readable event ids (`"E10"`, `"E11"`, …) to locate failing call sites.

---

### 22.7 In-memory cursors as first-class staging tables (`hb_orm_Cursor`)

When you need an in-memory cursor not tied to a specific SQL query (staging, merges, scratch workareas), use `hb_orm_Cursor`:

```harbour
function DemoCursorStaging()
local l_oCur := hb_orm_Cursor():new()

l_oCur:Field( "pk",    "N", 10, 0, .t., .t. )
l_oCur:Field( "fname", "C", 50, 0, .t., .f. )
l_oCur:Field( "lname", "C", 50, 0, .t., .f. )

l_oCur:CreateCursor( "StageCursor" )

l_oCur:Index( "pk_tag", "pk", .t. )
l_oCur:CreateIndexes()
l_oCur:SetOrder( "pk_tag" )

l_oCur:InsertRecord( {=> "pk" => 1, "fname" => "A", "lname" => "One" } )
l_oCur:InsertRecord( {=> "pk" => 2, "fname" => "B", "lname" => "Two" } )

select StageCursor
? reccount()

// Destructive operation: removes records but keeps structure/indexes.
l_oCur:Zap()

l_oCur:Close()

return NIL
```

AI-generation rule:

* Do not emit `:Zap()` unless the intent is explicit (it clears all records).

---

### 22.8 Transactions (basic orientation)

When executing multi-step writes that must succeed or fail as a unit, use transaction helpers on the connection:

* `BeginTransactionReadCommitted()`
* `BeginTransactionRepeatableRead()`
* `BeginTransactionSerializable()`
* `EndTransactionCommit()`
* `EndTransactionRollback()`

Guidance:

* On any write failure, rollback and report `:ErrorMessage()` (or connection error text where appropriate).

---

### 22.9 Diagnostics and troubleshooting hooks

When a query fails or is unexpectedly slow, Harbour_ORM exposes useful introspection:

* `:LastSQL()` to retrieve the last executed SQL command
* `:LastRunTime()` for execution time (seconds)
* `:GetLastEventId()` to identify which `:Table(<eventId>, ...)` context was active

Generate these outputs only in developer-facing diagnostics paths (not in normal UI paths).

---

### 22.10 AI guardrails for ORM code generation

When generating Harbour code that uses Harbour_ORM, enforce these rules:

**Must**

* Keep a connection object and pass it into ORM query objects.
* Use `:SQL( "<cursorAlias>" )` when the next code expects a workarea/cursor.
* Use `^` placeholders (when available) instead of raw string concatenation for values.
* Use explicit alias field addressing (`<alias>->FieldName`) in non-trivial routines.

**Must not**

* Document or call members declared under `hidden:` in class definitions.
* Rely on reading/writing `p_*` members directly as part of application logic (treat as internal state by convention, even when technically exported).
* Generate backend-specific SQL features (e.g., PostgreSQL-only constructs) unless the code is explicitly PostgreSQL-targeted or guarded by backend checks.

---

## 23) UTF-8 and Internationalization (DONE)

Harbour fully supports UTF-8 string content at runtime.  
The Harbour language syntax itself is ASCII, but string literals, variables, JSON content and database values may contain UTF-8 sequences.

The Harbour compiler accepts UTF-8 encoded source files without special flags.  
Unicode behaviour in Harbour is controlled at **runtime** by the selected **codepage**, not by compiler options.

### 23.1 Recommended runtime UTF-8 configuration

For correct Unicode-aware string operations such as:

- `Upper()` / `Lower()`
- `SubStr()`, `Left()`, `Right()`
- `PadL()` / `PadR()`
- `hb_utf8Len()`
- `hb_utf8Asc()` / `hb_utf8Chr()`

you should use the **extended UTF-8 codepage**.

#### 23.1.1 Request the UTF8EX codepage

```harbour
REQUEST HB_CODEPAGE_UTF8EX
```

#### 23.1.2 Select it early in your main routine

```harbour
function Main()
    hb_cdpSelect( "UTF8EX" )

    // your initialization code here

    return NIL
```

#### 23.1.3 Why “UTF8EX”?

`HB_CODEPAGE_UTF8EX` provides:

* Correct upper/lower case transformation for multi-byte characters
  (for example: `"élève"` → `"ÉLÈVE"`)
* Proper handling of accented European characters
* More consistent behaviour in comparisons, sorting, padding and scanning
* Better overall Unicode behaviour than the basic UTF-8 codepage

For modern Harbour applications, **UTF8EX should be considered the default choice**.

### 23.2 Runtime behaviour

* Harbour stores strings exactly as provided (UTF-8 byte sequences or generic bytes).
* The active codepage affects how Harbour interprets and manipulates these sequences:

  * Case conversion (`Upper()`, `Lower()`)
  * Length and slicing helpers (especially when using `hb_utf8*` functions)
  * Some collation/comparison logic, depending on context
* The codepage setting does **not** change how strings are stored inside external SQL databases.

### 23.3 Database considerations

#### 23.3.1 DBF / ISAM tables

* Some RDDs may translate between the DBF’s stored codepage and the active Harbour codepage.
* UTF-8 DBFs are possible but require care; for new applications with heavy Unicode usage, SQL backends are usually a better choice.

#### 23.3.2 SQL servers (PostgreSQL, MySQL, MariaDB, etc.)

* Harbour sends and receives UTF-8 byte sequences to/from the SQL driver.
* The database’s own encoding and collation (for example `UTF8` or `UTF8MB4`) determine how data is stored and compared.
* Harbour’s codepage affects only **in-memory** string operations, not the on-disk encoding in the SQL server.

### 23.4 Example

```harbour
REQUEST HB_CODEPAGE_UTF8EX

function Main()
    local l_cUString := "UTF-8 String őöüóéáűÉÁŰŐÚçèéù"

    hb_cdpSelect( "UTF8EX" )

    ? "Is UTF-8:", hb_StrIsUTF8( l_cUString )
    ? "UTF-8 length:", hb_utf8Len( l_cUString )
    ? "Byte length:", len( l_cUString )

    ? "Upper:", Upper( l_cUString )
    ? "Lower:", Lower( l_cUString )

    return NIL
```
---

## 24) Harbour_EL (Extended Library)

Harbour_EL (“Harbour Extended Library”) is an **optional add-on library** that extends core Harbour with:

- **HbPP preprocessor syntax sugar** (via `#include "hb_el.ch"`), and
- a set of pragmatic **runtime utilities** (PRG + C), plus
- optional **CPython embedding / interop** entrypoints.

This section is **only relevant** when the application being worked on **links the Harbour_EL library** (and, for the macro syntax sugar, includes `hb_el.ch`).

Repository: https://github.com/EricLendvai/Harbour_EL

---

### 24.1 Applicability and how it changes “valid syntax”

Harbour_EL affects “what code is valid” in two distinct ways:

1. **HbPP syntax sugar via `hb_el.ch`**  
   If `hb_el.ch` is **not** included, constructs like `try/catch/finally/endtry`, `scan/endscan`, and `text to var ... endtext` are **not available** (they are not core Harbour keywords; they are preprocessor rewrites).

   ```harbour
   #include "hb_el.ch"   // enables Harbour_EL macro syntax
   ```

2. **Linked runtime library** (PRG/C functions)  
   Even without `hb_el.ch`, linked functions exist and can be called normally. However, you lose the “language-like” syntactic conveniences provided by HbPP macros.

AI/codegen implication: only emit Harbour_EL-specific macro syntax when the build context clearly indicates Harbour_EL is used (linked) and `hb_el.ch` is included.

---

### 24.2 HbPP macro features (syntax sugar in `hb_el.ch`)

These are **not** core Harbour language constructs. They are HbPP rewrites that expand into standard Harbour forms.

#### 24.2.1 `try / catch / finally / endtry` (SEQUENCE sugar)

Harbour_EL maps:

- `try`     → `begin sequence`
- `catch`   → `recover` (optionally `recover using <var>`)
- `finally` → `always`
- `endtry`  → `end`

AI guidance:
- If Harbour_EL is **not** assumed, generate explicit `begin sequence ... recover ... always ... end` instead of `try` forms.

#### 24.2.2 `scan ... endscan` (record iteration macro)

Harbour_EL provides VFP-style `scan` forms (implemented using low-level DB operations and a small helper stack).

Behavioral notes that matter for code generation:

- `scan all ...` performs `dbGoTop()` first.
- `scan ...` (without `all`) starts from the **current** record.

AI guidance:
- If Harbour_EL is not present (or `hb_el.ch` is not included), use explicit loops such as:
  - `dbGoTop()` + `do while ! eof()` + `dbSkip()`
  - or explicit locate/continue patterns where needed.

#### 24.2.3 `text to var ... endtext` (heredoc-like capture)

Harbour_EL adds:

- `text to var <var>`           → sets `<var> := <captured text>`
- `text to var <var> additive`  → does `<var> += <captured text>`

AI guidance:
- Use only when HbPP macro syntax is acceptable in the project.
- Otherwise, generate explicit concatenation with `+=` and explicit newlines (e.g., `hb_eol()`) as needed.

#### 24.2.4 “Only-when-NIL assignment”: `default ... to` / `default ... :=`

Harbour_EL supports:

- `default <var> to <value>`
- `default <var> := <value>`

This is sugar for “assign only if NIL” (implemented using a by-reference helper such as `hb_default( @var, value )`).

AI guardrail:
- The left-hand side must be a **variable symbol** (passed by reference). Avoid emitting `default o:Prop to ...` or `default a[i] to ...` unless you have verified the macro supports that exact pattern.

#### 24.2.5 Small “dialect convenience” macros

Harbour_EL includes convenience rewrites such as:

- `this.<member>`  → `Self:<member>` (member access sugar)
- `.null.` / `NULL` / `ISNULL(x)` → `NIL` / `hb_IsNil(x)` (NULL-friendly naming)
- `NVL(v1,v2)`     → `el_nvl(v1,v2)` (helper-backed)
- `used(cAlias)`   → `( select( cAlias ) > 0 )`
- `cd <path>`      → `hb_cwd( <path> )`

---

### 24.3 Runtime helpers (non-Python surface)

Harbour_EL provides additional runtime utilities (implemented in PRG and C). The complete inventory should be referenced from the Harbour_EL documentation; for AI orientation, it is sufficient to recognize these categories:

- **File/path and environment utilities** (e.g., folder creation, process ID, path normalization).
- **Workarea/DBF helpers** used by the macro layer.
- **String utilities**, including replace/transform helpers and “write string to file” patterns.
- **Debug/trace output helpers** (e.g., writing compact records to DebugView-style sinks).

Representative names (not exhaustive): `el_CreateFolder()`, `el_GetProcessID()`, `el_UTCOffsetMinToText()`, `el_StrToFile()`, `el_AddPs()`, `el_occurs()`, `el_SendToDebugView()`, `hb_el_buildinfo()`.

AI guidance:
- Do not “guess” new `el_*` function names. Only use functions that already exist in the application’s codebase or in the known Harbour_EL docs.

---

### 24.4 CPython embedding / interop (optional subsystem)

Harbour_EL includes a CPython bridge that exports a family of functions as `PY_*` entrypoints (also callable as `Py_*()`, since Harbour is case-insensitive).

AI orientation:
- If a project is using this subsystem, expect patterns like:
  - initialization/finalization (`PY_INITIALIZE()`, `PY_FINALIZE()`)
  - importing modules (`PY_IMPORTPYTHONMODULE()`)
  - calling functions/methods (`PY_CALLFUNCTION()`, `PY_CALLMETHOD()`)
  - marshaling primitives/lists and extracting values

Build note:
- Downstream projects may need to update Python include/lib paths to match their installed Python version and layout.

---

### 24.5 AI/codegen guardrails for Harbour_EL

When generating Harbour code:

1. **Do not emit Harbour_EL macro syntax unless the project is known to include `hb_el.ch`.**  
   If unknown, generate pure-core Harbour constructs.

2. **Prefer explicit, core-Harbour equivalents** when writing library-agnostic code:  
   - `begin sequence / recover / always / end` instead of `try/catch/finally/endtry`  
   - explicit loops (`dbGoTop()` + `do while ! eof()`) instead of `scan`  
   - explicit string building instead of `text to var`

3. Treat Harbour_EL as an **application-level dependency** (similar to Harbour_ORM): only “activate” it in generated output when the build/link context confirms it is present.

---

## 25) Multithreading & synchronization

This section describes Harbour’s multithreading model, the core MT APIs, and the practical rules needed to build correct multi-threaded applications.

Harbour’s MT VM maps each Harbour thread directly to an OS thread, and the VM relies on thread-local storage (TLS) to access the current thread’s VM stack efficiently.

### 25.1 MT readiness: compile-time and runtime checks

Harbour multithreading is enabled by building/compiling the application with the `-mt` switch (hbmk2/Harbour). At runtime you can verify MT readiness using:

- `hb_mtvm() -> lMultiThreadVM`  
  Returns `.t.` when the executable is multi-thread ready (compiled with `-mt`).

**Guideline (AI + humans):**
- If your program requires threads, treat `hb_mtvm() == .f.` as a hard configuration error and fail fast with a clear message.

### 25.2 Thread creation & lifecycle

#### 25.2.1 Creating threads

- `hb_threadStart( [nThreadAttrs], <@sStart()> | bStart | cStart, [params...] ) -> pThreadID | nil`  
  Creates a new thread and starts executing a function symbol, codeblock, or function name. Returns a thread handle pointer or `nil` on failure.

`nThreadAttrs` selects inheritance behavior (defined in `hbthread.ch`). Prefer explicitly choosing what the new thread inherits (e.g., memvars / SET state), rather than relying on defaults.

#### 25.2.2 Identifying threads

- `hb_threadSelf() -> pThreadID | nil`  
- `hb_threadID( [pThreadID] ) -> nThreadNo`  
- `hb_threadIsMain( [pThreadID] ) -> lIsMain`

Use `hb_threadID()` in logs and error reports so production traces can be correlated to a specific execution stream.

#### 25.2.3 Waiting and shutdown

- `hb_threadJoin( pThreadID, [@xRetCode] ) -> lOk`  
  Waits until the thread finishes; optionally captures its return/exit value.
- `hb_threadDetach( pThreadID ) -> lOk`  
  Lets a thread run independently; resources are reclaimed automatically at termination.
- `hb_threadWait( pThreadID | aThreadIDs, [nTimeout], [lAll] ) -> nCompletedOrZero`
- `hb_threadQuitRequest( pThreadID ) -> lOk`
- `hb_threadTerminateAll()` / `hb_threadWaitForAll()` (main thread only)

**Guideline:**
- Prefer **join** for “owned” worker threads (thread pools, background tasks you manage).
- Use **detach** only for genuinely independent, fire-and-forget tasks where shutdown ordering is irrelevant and you accept reduced observability/control.

#### 25.2.4 One-time initialization

- `hb_threadOnce( @onceControl, [bAction | @sAction()] ) -> lFirstCall`
- `hb_threadOnceInit( @item, value ) -> lInitialized`

Use these for “initialize once across all threads” patterns (e.g., lazy singleton init guarded safely).

### 25.3 Mutexes and cross-thread coordination

Harbour’s mutex API supports both classic “critical section” locking and a practical “queue/event” pattern via `Notify`/`Subscribe`.

#### 25.3.1 Classic lock/unlock

- `hb_mutexCreate() -> pMutex`
- `hb_mutexLock( pMutex, [nTimeoutMs] ) -> lLocked`
- `hb_mutexUnlock( pMutex ) -> lOk`

Use this for protecting shared state (statics, globals, shared objects, shared caches).

**Rule:** Any shared mutable state reachable by multiple threads must be protected by a mutex (or a `sync method` lock), unless it is provably thread-local or immutable.

#### 25.3.2 Producer/consumer queue with `Notify`/`Subscribe`

- `hb_mutexNotify( pMutex, [xVal] )`
- `hb_mutexNotifyAll( pMutex, [xVal] )`
- `hb_mutexSubscribe( pMutex, [nTimeout], [@xSubscribed] ) -> lSubscribed`
- `hb_mutexSubscribeNow( pMutex, [nTimeout], [@xSubscribed] ) -> lSubscribed`

This enables a clean pattern:

- Producer enqueues work items via `hb_mutexNotify( pQueue, xJob )`
- Worker threads block on `hb_mutexSubscribe( pQueue, timeout, @xJob )`
- Shutdown is often done by sending a sentinel value (e.g., `nil`) per worker.

This is especially useful when building:
- socket/web request dispatchers,
- UI-thread message pumps,
- background job queues,
- thread pools.

#### 25.3.3 Introspection and guarded evaluation

- `hb_mutexQueueInfo( pMutex, [@nWaiters], [@nQueueLength] ) -> .t.`
- `hb_mutexEval( pMutex, bCode | @sFunc(), [params...] ) -> xResult`

`hb_mutexEval()` is a compact way to guarantee “run this block under mutual exclusion”.

### 25.4 Thread-local runtime state, memvars, and workareas

#### 25.4.1 What is thread-local in practice

In Harbour’s MT model, each thread has its own VM stack and its own thread-local execution context (enabled by TLS).

Thread creation typically inherits much of the “environment” from the creating thread, but threads still have important independent state.

**Rule (DBF/RDD and ORM workareas):**
- Treat the **current workarea and alias selection as thread-local**.
- Do not assume that `select()` state, active orders, relations, filters, etc. are valid across threads without explicitly establishing them.

#### 25.4.2 Moving a workarea between threads (advanced; usually avoid)

A mechanism exists in the ecosystem to pass a workarea between threads using `dbRelease()` / `dbRequest()` (xHarbour compatibility notes). This is an advanced technique and generally a last resort; prefer per-thread connections/workareas or message passing.

### 25.5 Error handling in threads (must be explicit)

**Practical rule:** Each thread must establish its own error-handling policy (e.g., `ErrorBlock()` + `begin sequence`) at the thread entry point. Do not assume the main thread’s errorblock automatically protects worker threads.

### 25.6 Object-level synchronization: `sync method`

Harbour supports `sync method` as a declarative, object-centric synchronization mechanism.

**Core semantics:** when an object is visible in multiple threads, a `sync method` will not run simultaneously in two or more threads for the same object instance. Concurrent calls serialize.

**Important practical nuance:** `sync method` locking is **object-wide**, not “per method”. If you declare multiple methods as `sync`, they all share the same per-object synchronization gate—so a `sync` getter can block behind a `sync` mutator on the same object.

**Design guidance:**
- Use `sync method` to protect *object invariants* that span multiple fields/steps.
- Keep `sync` method bodies short and non-blocking when possible.
- Avoid long waits (I/O, sleeps, blocking queue waits) inside `sync` methods unless you fully understand lock interactions and your deadlock-avoidance strategy.

### 25.7 Recommended patterns

#### Pattern A: Thread pool + queue (server-style)

- Create a shared queue mutex via `hb_mutexCreate()`
- Pre-spawn N workers with `hb_threadStart()`
- Main thread enqueues jobs with `hb_mutexNotify()`
- Workers `hb_mutexSubscribe()` and process
- Shutdown by sending a sentinel value (e.g., `nil`) per worker, then `hb_threadJoin()`

This pattern naturally supports backpressure and avoids “one OS thread per request” explosion.

#### Pattern B: Single “owner” thread for UI/event loop + worker threads

- UI thread owns GUI/WebView objects and state that must be single-threaded.
- Workers perform blocking work (network, DB, CPU tasks) and post results back via queue.

#### Pattern C: Shared object with `sync method` for invariants

- Use a dedicated “state coordinator” object with `sync` methods.
- Only mutate shared state via those `sync` methods.
- For composite operations (read-modify-write across multiple members), keep them in one `sync` method to guarantee atomicity at the object level.

### 25.8 Guardrails (what AI must not do)

- Do not introduce shared `static` / `public` mutable state accessed by multiple threads without explicit synchronization (`hb_mutex*` or `sync method`).
- Do not assume DBF workareas or “current alias” can be safely shared across threads; always re-establish thread-local context.
- Do not block for long periods inside `sync` methods unless the design explicitly requires it and you have a clear deadlock-avoidance strategy.
- Do not detach threads by default; prefer joinable threads unless the task is intentionally fire-and-forget.

### 25.9 Minimal canonical examples

#### Example 1: Simple worker queue (mutex notify/subscribe)

```harbour
static function ThreadWorker( par_pQueue )
    local l_xJob

    do while hb_mutexSubscribe( par_pQueue, 1000, @l_xJob )
        if l_xJob == nil
            exit
        endif

        // do work here (must be thread-safe)
        // ...

    enddo

return nil

procedure Main()
    local l_pQueue
    local l_aThreads := {}
    local l_i

    if ! hb_mtvm()
        ? "error: this executable is not multi-thread enabled (compile with -mt)"
        return
    endif

    l_pQueue := hb_mutexCreate()

    for l_i := 1 to 4
        aAdd( l_aThreads, hb_threadStart( @ThreadWorker(), l_pQueue ) )
    next

    // enqueue jobs
    hb_mutexNotify( l_pQueue, "job-1" )
    hb_mutexNotify( l_pQueue, "job-2" )

    // stop workers (one sentinel per worker)
    for l_i := 1 to len( l_aThreads )
        hb_mutexNotify( l_pQueue, nil )
    next

    aEval( l_aThreads, {|l_pTh| hb_threadJoin( l_pTh ) } )

return
```

#### Example 2: `sync method` protects object invariants

```harbour
#include "hbclass.ch"

class Counter
    exported:
        var nValue init 0
        sync method Inc()
        sync method Get()
endclass

method Counter:Inc()
    ::nValue++
return ::nValue

method Counter:Get()
return ::nValue
```

Note: because both methods are `sync`, they serialize for the same object instance. That is often desirable for correctness, but can become a throughput bottleneck if overused.

---

## 26) GPT guardrails (for code generation)

These are hard constraints intended to keep AI-generated Harbour code predictable, portable, and compatible with your ecosystem.

### 26.1 Do not generate debugger traps or debug-only scaffolding

- Never emit `ALTD()` (or similar interactive debugger triggers) in generated code unless the user explicitly requests it.
- Do not include compile switches or pragmas intended for debugger integration (e.g., `#pragma /B+`, `#pragma -B-`) unless asked.

### 26.2 Do not redefine ubiquitous syntax in application code

- Do not generate `#command` / `#translate` rewrites for core pseudo-commands like `?`, `??`, `@ ... SAY`, etc., in normal application code.
- If the user requests tooling (debugger, logger shim, instrumentation), isolate rewrites behind `#ifdef` and keep them in one module.

### 26.3 Prefer explicitness over “clever” HbPP

- Avoid large DSL-style rewrites that hide control flow or side effects.
- Use HbPP for constants, small convenience wrappers, and compatibility shims that remain obvious from the call site.

### 26.4 Keep I/O and file paths portable

- Prefer `hb_ps()` for separator portability (or a single project helper).
- Prefer explicit, full paths for DBF tables (see §18.3.1).

### 26.5 Respect repository function sets

- Only use functions that exist in the project’s canonical Harbour function dataset (the YAML function catalog) and in the configured dependencies.
- If a needed function is uncertain, generate a small helper function instead of guessing API names.

### 26.6 Text encoding

- Assume UTF-8 by default.
- For lower/upper casing of UTF-8 strings with diacritics, use UTF8EX and `hb_utf8*` helpers.

---

## 27) Open items to expand later

- I/O and networking conventions (hb_httpd/FCGI, sockets, REST patterns)  
- Testing patterns/macros and logging conventions  

---

## 28) Appendix — canonical build patterns (hbmk2 / .hbp / .hbm)

This appendix captures **practical, repeatable** build patterns seen in the samples repo. It is intentionally short and “copy/paste friendly.”

### 28.1 Minimal hbmk2 one-liner

Console program:

```text
hbmk2 hello.prg -hbexe -o build/hello
```

Notes:
- `-hbexe` targets an executable.
- `-o` sets the output name/path.

### 28.2 Recommended project structure: platform `.hbp` + shared `.hbm`

A clean approach is to keep **platform-specific** settings in a small `.hbp` and put shared settings + file lists in a `.hbm`.

Example (Windows `.hbp`):

```text
-obuild\win64\${HB_COMPILER}\${BuildMode}\${EXEName}.exe
HelloHarbour.hbm
```

Example (shared `.hbm`):

```text
-hbexe
-trace
-gtcgi

HelloHarbour.prg
```

Why this helps:
- One shared file list and common switches (`.hbm`).
- Only the output path and platform quirks vary (`.hbp`).

### 28.3 Common hbmk2 flags used in samples

- `-b` : debug build mode (enables debug features in toolchains that support it)
- `-p` : keep preprocessor output (`.ppo`) for inspection
- `-w3` : increase warning level
- `-dNAME` : define a preprocessor symbol (equivalent to `#define NAME`)
- `-workdir=...` : redirect intermediate outputs (useful if you want generated C files in a dedicated folder)
- `-static` : static link (when supported and desired)

Guidance:
- Use `-p` and a dedicated `-workdir` when debugging HbPP expansions or build issues.
- Prefer `-d` switches for build-time feature flags over editing sources.

### 28.4 VS Code task pattern (conceptual)

The samples repo demonstrates a practical approach:
- a single build script (`BuildEXE.bat` / `BuildEXE.sh`) that sets toolchain env vars and calls `hbmk2`
- VS Code tasks that pass `EXEName`, `BuildMode`, `HB_COMPILER` via environment variables

This keeps the workspace consistent across Windows and Linux (including devcontainers).

---

**End of unified guide.**
 