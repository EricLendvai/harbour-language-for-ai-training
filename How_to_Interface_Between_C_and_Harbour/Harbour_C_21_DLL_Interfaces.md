---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 21
chapter_title: "Interfaces to DLL Functions"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 21. Interfaces to DLL Functions

In this chapter, we’ll learn how to use functions stored in a **DLL** from Harbour.  
But first, we must be clear about a few ideas…

As we’ve seen in a previous topic, there are **two types of libraries** or **function collections**: *static* and *dynamic*.  
What they have in common is that both store **functions** that can be used by our programs. In other words, a function library is a **container** where a set of reusable functions is kept so they can be included in our executable programs.  

What determines whether a library is one type or the other is the **moment it links** with our program.

- **Static libraries** are linked during the **compile/link process**, so their functions are embedded inside the executable, and the library itself is no longer needed at runtime.  
- **Dynamic libraries**, on the other hand, provide their functions only when the executable **requests** them dynamically — hence the name. Therefore, the file containing those functions must be **present while the program is running**.

For the executable to know about the public functions in a DLL, it needs a mechanism that provides that information.  
This is done via **import libraries**, which are special **static libraries** that act as **symbol tables**, containing metadata about the DLL functions (their names and an ordinal number that will later be replaced by the memory address where the executable should look to locate each function).

Each type of library has its **advantages** and **drawbacks**:

- Using static libraries makes executables **larger**, but removes dependence on external programs and avoids compatibility problems that might arise with different versions of dynamic libraries (DLLs).  
- Using dynamic libraries keeps executables **smaller**, and allows several programs to share the same DLL functions.

The way to build **static** and **dynamic** libraries differs.  
Static libraries typically use extensions like `*.lib` or `*.a`, while dynamic ones are `*.dll` on Windows and `*.so` on Linux and other operating systems.

The term **DLL** stands for *Dynamic Link Library* in Windows, and the equivalent concept in Unix-like systems is **SO**, meaning *Shared Object*.

To use DLLs from a Harbour PRG we need:
1. the **import library** for the DLL, and  
2. another library containing the **interface functions**.

## How to create import libraries

### Example: generating an import definition file (.DEF)

**With GNU tools:**
```bash
pexports libmariadb.dll > libmariadb.def
```
or
```bash
gendef libmariadb.dll
```

**With Borland:**
```bash
impdef libmariadb.def libmariadb.dll
```

### Example: creating the import library

**With GNU:**
```bash
dlltool -d libmariadb.def -D libmariadb.dll -k -l libmariadb.a
```

**With Borland:**  
*(the next steps would follow the same logic but using Borland’s import-library tools; details appear later in the book)*
