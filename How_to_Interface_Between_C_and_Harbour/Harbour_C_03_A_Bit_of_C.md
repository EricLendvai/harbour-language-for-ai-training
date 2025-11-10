---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 3
chapter_title: "A Little Bit of C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 3. Introduction — A Little Bit of C

In this section, we will very quickly review the history of the C language.  
There are many accounts of how the C language was born, so this will be brief, and we will use the opportunity to discuss some basic programming concepts.

The C language was born a few years before Clipper and, therefore, before Harbour.  
It was the year **1972** when a bearded man named **Dennis Ritchie** published the first version.  
He worked in the legendary **Bell Labs** of an American telecommunications company, **AT&T**.

But why the name *C language*?  
Well, yes — because previously there had been a *Language B* and later a *Language D*.  
I think good old Dennis didn’t strain his brain too much to baptize it.

Initially, C was used to implement the **UNIX operating system**, the father of all modern Linux systems.  
It was later used to build other operating systems (OS) such as **Linux**, **iOS**, and **Windows**.

It is such a powerful, robust, and structured language — with very few keywords.  
All of this has made it the language used to develop other languages, like our beloved **Harbour**.

---

## 3.1 Types of Languages

As you know, there are several types of programming languages:

### • Low-Level Languages  
They are completely tied to the machine.  
- **Machine Language**, which is the only one our computer truly understands.  
- **Assembly Language**, which is an evolution of machine language that assigns abbreviated names to machine instructions.

### • High-Level Languages  
They are independent of the machine on which they will run and resemble human natural language (which happens to be English).  
In this category are **C**, **Harbour**, **Java**, **PHP**, **Basic**, **Pascal**, **C#**, **Fortran**, **COBOL**, etc.

Among these languages, some are general-purpose and others are specialized for specific tasks.  
For example, **C** and **Harbour** are general-purpose, while **Fortran** specializes in mathematical computation and **COBOL** in business management programs.

For a computer to understand high-level languages, they must be converted into machine language.

---

## 3.2 Types of Translators

There are several types of translators:

- **Assemblers**: generate machine code from assembly language.  
- **Interpreters**: verify and translate each instruction line by line into machine code and then execute it.  
- **Compilers**: analyze and translate the entire program into machine code before execution.

Each type of translator has its advantages and disadvantages (we can talk about that in class 😉).

C belongs to the family of **compiled languages**.  
When we build a C executable, it generates an error-free **OBJ** file, and then the **linker (LINK)** retrieves the necessary components from **LIBs** or other **OBJs** to generate a standalone **EXE**.

The **BASIC** language is the typical example of an interpreted language: it verifies each instruction as it encounters it, translates it into machine language, and executes it.

**Harbour** is interpreted like BASIC, except that — to make the process faster and more efficient — it creates a highly optimized code that is not pure machine language but which we mere mortals could not decipher.  
That code is called **pCode**, and it is then interpreted by the **Harbour Virtual Machine (HVM)**.

Harbour’s pCode executes very quickly under the virtual machine and is also free from lexical errors.

If you compile a PRG with the `-gh` option, a file with the extension `*.hrb` is generated containing the pCode.  
In Harbour this is called a **portable binary**, and it can be executed using the included tool **HbRun.exe**.

But you may ask: does Harbour also generate an EXE like the C language?  
The trick is that both the **virtual machine** and the **pCode** are actually embedded inside the EXE — that’s why those EXEs are so large!  
But don’t be alarmed: that “obesity” doesn’t stop them from being extremely fast at the same time.  
For **Mod Harbour**, this behavior is fantastic!

Let’s leave Harbour aside for a moment and return to C.

---

## 3.3 Dissecting C Programs

C is a very structured language, organized around functions or procedures.  
In C, the term *function* is generally used for both — after all, the only difference is that a function returns a value while a procedure does not.  
Both can receive parameters or not.

In **Harbour**, we must indicate to the compiler whether we are defining a `FUNCTION` or a `PROCEDURE`; failing to do so may cause a warning or an error during compilation.

By the way, there is great similarity between both languages in this regard.

Unlike Harbour, **C is case-sensitive** — meaning it distinguishes between uppercase and lowercase.  
For C, “Value” is not the same as “value.”  
Although this might be tricky at first, it can actually be useful.  
For example, in this course, all functions created for Harbour will be written in **uppercase**, while those in C will use **Hungarian Notation**, ensuring that names never collide.  
You can find a lot of material online about different naming conventions.

Just like in Harbour, if there is a function called **main**, it will be the first one executed.

In C, a function is a way to group instructions in a structured and coherent way to perform a task — coincidentally, the same as in Harbour.  
In other words, a function is simply a set of instructions that perform a specific task.

---

## 3.4 The Skeleton of a Typical C Function

```c
/* This is a comment */
#include <stdio.h>

return_type function_name(type parameter, …)
{
    local_variable_declarations;
    …
    statements; // This is a comment
    …
    return variable_of_the_type_defined_by_the_function;
}
```

---

## 3.5 Comments

The two types of comments in C also exist in Harbour and work the same way:

- `/* ... */` for comments that can extend across multiple lines, and  
- `//` for inline comments.

In both cases, the compiler ignores them.  
They are used only for annotations among programmers.

---

## 3.6 Preprocessor Directives

In this, Harbour and C also coincide in how they are defined, though each has its own specific directives and some overlaps.  
We will start by looking at the two main ones:

### `#include`
This directive tells the compiler to include another file within the current source file.  
The included file usually has the extension `.h`, although it may have any extension.  
Angle brackets `< >` are used when including system headers (those that come with the compiler), and quotes `" "` are used for local include files.  

This works exactly the same in Harbour, except that include files there typically have the `.ch` extension.  
As you can see, both compilers handle this identically.

In C, before you can use something, it must be defined.  
That is the main purpose of `.h` header files, which are usually included at the beginning of programs.

Examples:

```c
#include <stdio.h>
#include "hdo.api"
```

### `#define`
Used to define symbolic names that will not change throughout the program — that is, constants.  
Fortunately, this works exactly the same in both compilers.  
The compiler replaces the symbol with its actual value at compile time.

Examples:

```c
#define MAXIMUM 90
#define MINIMUM 25
```

---

## 3.7 Function Names, Definitions, and Implementations

The **function name** is the identifier we use whenever we want to call it.  
Therefore, it is very important that the name be self-descriptive.  
As mentioned earlier, it must be defined before being used.  
The declaration must specify the type of value it returns, its name, and — in parentheses — the parameters it receives, each preceded by its type and separated by commas.

Example — **function declaration**:

```c
int double_value(int);
```

Now the implementation:

```c
#include <stdio.h>
/* Function that returns twice the integer value passed */
int double_value(int iNum)
{
    int iRet;
    iRet = iNum * 2;
    return iRet;
}
```

In the declaration, it’s not necessary to name the parameter variables — only their types — but in the implementation, it is mandatory.  
Next come the curly braces `{}` — everything between them forms the **body** of the function.  
Be careful: every instruction ends with a semicolon (`;`).

Another example with more than one parameter:

```c
#include <stdio.h>

int multiply(int iParam1, int iParam2)
{
    int iRet;
    iRet = iParam1 * iParam2;
    return iRet;
}
```
