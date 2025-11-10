---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 20
chapter_title: "Creating our own libraries of functions"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 20. Creating our own libraries of functions

Most of us are familiar with the concept of a **library** of functions. In fact, we use one every day whenever we build our programs.  
But what *is* a function library? We could say it’s a **container of functions and procedures** stored in a file, usually with an extension like `*.lib`, `*.a`, `*.dll`, or `*.so`. We’ll explain each of these later.

That file should be organized around functions and procedures that share a common purpose — for example: math routines, string processing, screen output, reporting, etc.

When we have a set of functions used in more than one program and their code is well tested, it’s time to create a **library**. Doing so gives us clarity and convenience.

Many of the libraries we use already come with our favorite compiler or are provided by a third‑party vendor. The good news is that **we can create our own**, which lets us organize our code better and manage those functions and procedures we use across all our programs.

In this chapter, we’ll focus on creating **libraries for Harbour**, so the code we put in them has to be understood by our compiler. That means they must be written in **PRG** or **C** (using the various APIs Harbour provides).

To use the functions and procedures included in libraries, we need to know the **parameters** they receive and the **values** they return. This applies to both **static** libraries (`*.lib` or `*.a`) and **dynamic** ones.

Basically, there are two kinds of libraries (plus one related case):

1. **Static.** The functions or procedures they contain are **included in the final executable** once it’s compiled and linked, so the library file is no longer required for the program to run. On Windows, they’re typically `*.lib` for Borland 32‑bit, MSVC, Pelles C, etc. On Unix/Linux/macOS and similar, they’re typically `*.a`. Some Windows‑ported compilers also use `*.a`, such as MinGW, Clang, and Clang‑based toolchains (e.g., Borland 64‑bit 7.xx).
2. **Dynamic.** The functions are **not** included in the executable; the exe only **references** them. This means the library must be present at runtime for the program to execute. On Windows the usual extension is `*.dll`, and on Unix/Linux/macOS the extension is typically `*.so`.
3. **Import libraries.** These are **static** libraries that contain **references** to the functions provided by **dynamic** libraries — essentially a **symbol table**. They’re needed only at **compile/link time**, and therefore do **not** need to be present at runtime.

As mentioned earlier, we continually use libraries provided by Harbour, the C compilers, and third parties such as FWH, Xailer, or xHarbour.com. But the important point is that **we can also create our own**, and that’s the purpose of this chapter.

Each C compiler has its own way to build libraries, so we would need to explain the process for each of them. The good news is that Harbour’s **`hbmk2`** tool takes care of invoking the necessary commands for each C compiler, freeing us from that complexity.

I think the best way to explain how to do this is with an example that you can adapt to your needs.

To create the library we’ll use Harbour’s **`hbmk2`** tool, so we need to know a few options from the tool:

- `-o<outname>` — output file name  
- `-i<p>` or `-incpath=<p>` — additional include paths for header files  
- `-hblib` — create a **static** library  
- `-hbdyn` — create a **dynamic** library (**without** linking to the Harbour VM)  
- `-hbdynvm` — create a **dynamic** library (**with** linking to the Harbour VM)  
- `-implib=<output>` — create an **import** library (in `-hbdyn`/`-hbexe` mode) named `<output>` (default: same as output)  
- `-hbimplib` — create an **import** library (**Windows only**)  

These are the basic options we’ll use; `hbmk2` has many more.

We should also create a **project file** where we include the options and the PRG/C source files that will contain the functions and procedures of our library. Harbour’s `hbmk2` project files use the `*.hbp` extension.

> **`*.hbp` — project file.** May contain any number of command‑line options (the same ones you’d pass to create a final target). Lines starting with `#` are ignored. Newlines are optional; options should be separated by spaces, as on the command line. Options containing spaces must be wrapped in double quotes. Each referenced `.hbp` file is executed as a **sub‑project**.

With all this, we’re ready to create our libraries. Typically, we create a project file listing the PRG and C sources and the options.

Below is the **batch** script that will call `hbmk2`:

**Source `buildlib.bat`:**
```bat
@rem ---------------------------------------------------------------------------
@rem Batch file to build libraries
@rem To adapt to your system, change these variables:
@rem COMP  C compiler for Windows; valid values include: mingw, mingw64, clang,
@rem       clang64, msvc, msvc64, clang-cl, clang-cl64, watcom, icc, icc64,
@rem       iccia64, msvcia64, bcc, bcc64, pocc, pocc64
@rem DIR_HBBIN  Harbour bin directory
@rem DIR_CCBIN  C compiler bin directory
@rem ---------------------------------------------------------------------------
@set COMP=mingw64
@set DIR_HBBIN=d:\mio\programacion\comp\xc\hb\bin
@set DIR_CCBIN=d:\mio\programacion\comp\cc\mingw\32\9.30\bin
@set PATHOLD=%PATH%
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%COMP% cursodec.hbp
@pause
@if %errorlevel% neq 0 goto bld_error
@goto fin_exec
:bld_error
@echo -----------------------------------
@echo    There are compilation errors
@echo -----------------------------------
@pause
:fin_exec
@set PATH=%PATHOLD%
```

And here is the **first** `hbmk2` project file template:

```text
#-------------------------------------------------------------------------------
#  AUTHOR....: Manuel Expósito Suárez 2021
#  CLASS.....: cursodec.hbp
#  MOD DATE..: 2021-08-05
#  VERSION...: 1.00
#  PURPOSE...: Build script for the library for all C compilers using Harbour hbmk2
#-------------------------------------------------------------------------------

# Indicate that we are going to build a library
-hblib

# If there were header (include) files for our library, we would indicate their location
# in our case there are none
# -i./src/include

# List all PRGs (and directories) that are part of the library
./src/prg/tfbuffer.prg
./src/prg/thbuffer.prg

# List all C files (and directories) that are part of the library
./src/c/cursode1.c
./src/c/cursode2.c

# Library name and where to place it once built
-o./lib/cursode

#-------------------------------------------------------------------------------
```

And here is a **variant** that automatically adds all sources from a folder:

```text
#-------------------------------------------------------------------------------
#  AUTHOR....: Manuel Expósito Suárez 2021
#  CLASS.....: cursodec.hbp
#  MOD DATE..: 2021-08-05
#  VERSION...: 1.00
#  PURPOSE...: Build script for the library for all C compilers using Harbour hbmk2
#-------------------------------------------------------------------------------

# Indicate that we are going to build a library
-hblib

# If there were header (include) files for our library, indicate their location
# -i./src/include

# Indicate that all PRGs in the given directory are part of the library
./src/prg/*.prg

# Indicate that all C files in the given directory are part of the library
./src/c/*.c

# Library name and where to place it once built
-o./lib/cursode

#-------------------------------------------------------------------------------
```

This last template has the drawback that if there are files in the folder **by mistake** that should not be part of our library, they will be included as well — which is why I recommend the **first** template.

The first time you build the library it usually takes longer than subsequent builds because `hbmk2` is smart and only processes those files whose timestamp has changed since the last build.
