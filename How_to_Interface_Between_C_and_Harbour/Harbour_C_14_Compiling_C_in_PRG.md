---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 14
chapter_title: "How to Compile C Code in Our PRG Programs"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 14. How to Compile C Code in Our PRG Programs

We are now finally reaching the part we all enjoy the most — **programming**.  
But before that, we need to understand **how to compile those C functions**.

Technically, we could invoke the C compiler directly.  
However, each compiler has its own options and syntax, which can be tedious to learn.  
So, how can we avoid that?  
How can we simplify the process of building **EXE**, **LIB**, or even just **OBJ** files?

If your answer was **“Use a MAKE tool”**, you are absolutely right!

And that’s where Harbour’s amazing build tool comes in: **`hbmk2`**.

## 14.1 What Is `hbmk2`

**Syntax:**
```
hbmk2 [options] [<command files>] <sources>[.prg|.c|.obj|.o|.rc|.res|.def|.po|.pot|.hbl|@.clp|.d|.ch]>
```

**Description:**

`hbmk2` is an **integrated and portable build automation tool**,  
allowing you to create different types of binary executables  
(executables, dynamic libraries, static libraries, portable Harbour binaries)  
from multiple source file types (C, C++, Objective-C, Harbour, gettext translations, Windows resources).

- **Integrated** means a single `.hbmk2` project file can control nearly every aspect of the build process.  
- **Portable** means the same `.hbmk2` project file works across all supported operating systems and compilers.

It also simplifies most build processes through small, straightforward **project option files**.

`hbmk2` supports C/C++/Objective-C projects, even those unrelated to Harbour.  

To achieve this, it automatically detects:
- The Harbour installation,
- The C compiler,
- Other required tools,  

then configures and invokes them as needed.

You can even **extend supported source types** via plug-ins.

In addition to building executables, `hbmk2` can also:
- Directly run Harbour source or precompiled code,  
- Provide an **interactive command interpreter**.

## 14.2 Example: Generic Batch Compilation Setup

Below is a **generic batch script proposal** you can adapt to your environment.

We’ll assume the C function examples are stored in a file named `cursode.c`,  
and that each `.PRG` file will have its own name.  
Eventually, you could compile all of them into a single **library (LIB)**,  
but for now, we’ll just compile the `.C` file each time as a source.  

Don’t worry about build times — `hbmk2` is smart enough to perform **incremental builds** (it recompiles only if changes are detected).

## 14.3 Directory Structure

```
curso_c
 ├── ejemplos
 │    ├── <your_prg_files.prg>
 └── fuentes_c
      └── cursode.c
```

Your batch file will be called:

```
do_<compiler>.bat
```

Example: `do_mingw64.bat`

## 14.4 Generic Batch Template

```bat
@set comp=MyCCompiler
@set DIR_HBBIN=MyHarbourBinDir
@set DIR_CCBIN=MyCBinDir
@rem The next two lines usually do not need to be changed
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%comp% %1 fuentes_c\cursode.c
```

## 14.5 Variable Description

- **MyCCompiler →**
  - 32-bit: `mingw`, `msvc`, `clang`, `bcc`, `watcom`, `icc`, `pocc`, `xcc`
  - 64-bit: `bcc64`, `mingw64`, `msvc64`, `msvcia64`, `iccia64`, `pocc64`

  For **MSVC** (both 32 and 64-bit), you must also call a script that initializes the environment.

- **MyHarbourBinDir →** Path to the **BIN** directory of your Harbour installation.  
- **MyCBinDir →** Path to the **BIN** directory of your C compiler.

> ⚠️ The Harbour **LIB** files must match the bitness (32/64) of the C compiler you are using.

## 14.6 Practical Examples

### 🔹 MinGW 64-bit
```bat
@set comp=mingw64
@set DIR_HBBIN=u:\desarrollo\comp\xc\hb\bin
@set DIR_CCBIN=u:\desarrollo\comp\cc\mingw\64\9.30\bin
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%comp% %1 fuentes_c\cursode.c
```

### 🔹 MSVC 32-bit
```bat
@set comp=msvc
@set DIR_HBBIN=u:\desarrollo\comp\xc\hb\bin
@call "%ProgramFiles(x86)%/Microsoft Visual Studio/2019/Community/VC/Auxiliary/Build/vcvars32.bat"
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%comp% %1 fuentes_c\cursode.c
```

### 🔹 MSVC 64-bit
```bat
@set comp=msvc
@set DIR_HBBIN=u:\desarrollo\comp\xc\hb\bin
@call "%ProgramFiles(x86)%/Microsoft Visual Studio/2019/Community/VC/Auxiliary/Build/vcvars64.bat"
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%comp% %1 fuentes_c\cursode.c
```

## 14.7 Notes

These batch files are deliberately simple — we’ll refine them later by adding:
- Error handling,
- Build logging,
- Automatic library creation.

But for now, they’re perfect for quick experiments and small projects.

✅ **Summary:**
- `hbmk2` is Harbour’s universal build tool.  
- Use simple batch scripts to manage builds.  
- Match your Harbour installation and C compiler versions (32-bit or 64-bit).  
- Incremental compilation saves time and keeps your workflow clean.
