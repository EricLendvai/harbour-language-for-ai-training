---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 6
chapter_title: "Elementary Data Types in C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 6. Elementary Data Types in C

Now things are getting interesting…  
> “You are my only constant in this world of variables!”  
Nice phrase, isn’t it? But what does it really mean?

Every program needs to store the information it will use.  
That memory space is used by **variables** and **constants**.  

In **Harbour**, there is an additional concept beyond variables and constants — **FIELDs**, which hold the values of DBF table fields.

## 6.1 Variables

Formally, a variable is an *object with a name* that can contain a value which may be modified during the program’s execution.

In **C**, variables have a *type* that must be specified.  
They are stored in memory, and the amount of space they occupy depends on their type.

In **Harbour**, it is not necessary to declare the type since it depends on the value assigned to it.  
(We’ll talk later about why Harbour is so clever!)

**Examples:**
```c
char cAracter;
long lNum;
char *szCadena;
```

## 6.2 Constants

A **constant** is like a variable, but its value cannot change during program execution.

The best way to declare a constant is using `#define`, but it is also possible to use the qualifier `const`.

**Examples:**
```c
#define TRUE 1
#define FALSE 0
#define MAX_AGE 90
```

`10`, `-5`, `"Manu"` are also constants.

In this case, these are called **symbolic constants**, and they are used to make programs more understandable and easier to maintain.  
The compiler automatically replaces the symbol with its actual value.

For example, if `MAX_AGE` later needed to change from 90 to 100, you would only have to modify it in the `#define` line, not everywhere in the code.

This method of defining constants can also be used in **Harbour**.

Another way to define constants in **C** is with the `const` keyword:

```c
const int iMaxAge = 90;
const char *szName = "Manu";
```

However, standard C can sometimes be fooled by this second form, so the `#define` method is generally preferred.

## 6.3 Declaring Variables and Understanding Data Types

After discussing variables and constants, let’s see how they are declared.  
This will lead us to the concepts of **types**, **modifiers**, and **qualifiers**.

A variable is declared as follows:

```c
type name;
```

Unlike in Harbour, in C you must **declare the variable before using it**, and you must also specify its type.

The main types in C are:

| Type | Description |
|-------|--------------|
| `int` | Integer numbers |
| `char` | Characters |
| `float`, `double` | Real numbers |

Each occupies a certain amount of memory, and the compiler must know the type to reserve the necessary amount.  
That’s why C is said to be a **strongly typed language**.

## 6.4 Modifiers

There are modifiers that affect how much memory is reserved for a given type.

For integers (`int`), the most common are `short` and `long`.

**Examples:**
```c
char cLetter = 'A';
int iAge;
long int lDistance;
short int sHeight;
```

The last two could also be declared without explicitly writing `int`:

```c
long lDistance;
short sHeight;
```

## 6.5 Character Strings

A string of characters can be declared as follows:

```c
char *szString = "This is a string"; // or
char szString[] = "This is a string";
```

We’ll go deeper into this topic later.  
For now, remember that in C it’s **mandatory to specify the type**, and that **each type occupies a specific space in memory**.

## 6.6 Character Type

In C, the `char` type is treated as an **integer of one byte**, meaning it can also be modified with `unsigned`, just like integers.

**Examples:**
```c
unsigned int uiAge;
unsigned long ulDistance;
unsigned char cLetter;
```

## 6.7 Data Type Sizes (on i686 machines)

| Type | Size (bytes) |
|-------|--------------|
| `char`, `unsigned char` | 1 |
| `short int`, `unsigned short int` | 2 |
| `int`, `unsigned int`, `long int`, `unsigned long int` | 4 |
| `float` | 4 |
| `double` | 8 |
| `long double` | 12 |

Remember: **1 byte = 8 bits**.

Unfortunately, the size of `int` can vary depending on whether the platform is 32-bit or 64-bit.  
For this reason, it’s best to avoid relying too much on plain `int` and instead use explicit sizes when possible.

That concludes the discussion of **elementary, simple, or primitive types** in C.
