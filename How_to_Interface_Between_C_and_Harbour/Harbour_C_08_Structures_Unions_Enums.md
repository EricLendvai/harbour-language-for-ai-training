---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 8
chapter_title: "Structured Data Types: Structures, Unions, and Enumerations"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 8. Structured Data Types: Structures, Unions, and Enumerations

## 8.1 Structures

A **structure** is a complex data type because, like arrays, it can contain one or more members.  
As my professor used to say:  
> “A structure is a finite set of elements of any type grouped under a single name to make their use more efficient and intuitive.”

Each element of a structure is called a **field** or **member**.  
Unlike arrays, however, the elements of a structure are **not guaranteed** to be stored consecutively in memory.

In other languages like **Pascal**, structures are known as *records*.  
In **Harbour**, they don’t exist as such, but can be emulated with **classes** that only contain *datas* or instance variables.

To declare a structure, we use the `struct` keyword followed by the name and a list of members enclosed in braces `{}`.

**Example:**
```c
struct person {
    char NIF[10];
    char name[30];
    int age;
    int gender;
};
```

⚠️ **Important:** Declaring a structure only *describes its composition* — it does **not** declare a variable or reserve memory.  

To compare this with **Harbour**, it’s like defining a class with the `CLASS` clause — the variable is only created when you instantiate it, e.g.:

```harbour
local oPerson := TPerson():new()
```

In **C**, to declare a variable of a given structure type (using the above example):

```c
struct person student;
```

You can also declare a structure and its variable at the same time:

```c
struct {
    char NIF[10];
    char name[30];
    int age;
    int gender;
} student;
```

This form is typically used when the structure definition will only be used once.

Later, we’ll see a more elegant and intuitive way to define our own “data types” using **`typedef`**.

### Initializing Structures

A structure can be initialized at the same time it’s declared:

```c
struct person student = { "98879345J", "Pablo", 21, 0 };
```

There’s also a lesser-known but perfectly valid method:

```c
struct person student = { .age = 21, .name = "Pablo" };
```

### Accessing Structure Fields

To access or modify structure fields:

```c
student.age = 19;
student.gender = 1;
```

If you have a pointer to a structure, you use the **arrow operator (`->`)** instead:

```c
student->age = 19;
```

We’ll explore this in detail later when we cover **pointers**.

As a closing note, it’s worth mentioning that the **Harbour API** is full of structures — a sign of how useful they are.  
In fact, from the perspective of C, a Harbour **ITEM** is implemented as a structure.

## 8.2 Unions

**Unions** are very similar to structures, with one key difference:  
all their fields start at the **same memory address** — they occupy the same area of memory.  
Therefore, a union can be interpreted as **different types at different times**.

The size of a union in memory is **not the sum** of its members’ sizes,  
but rather the size of its **largest field**.

They are declared exactly like structures, except the keyword is `union` instead of `struct`.

**Example:**
```c
union myItem {
    int integer;
    float real;
    char string[30];
};
```

Everything said about structures applies equally to unions — declaration, assignment, and access.  
The main difference is how memory is shared among members.

Interestingly, **Harbour variables are based on unions**.  
In fact, a Harbour variable’s representation in C is a structure that includes a field indicating its type, plus a union containing each of the possible Harbour data representations.

That structure is known as **`HB_ITEM`**.

## 8.3 Enumerations

Just like structures and unions, **enumerations** allow us to define new data types, using the keyword `enum`.

Enumerations let us define a set of values that represent some discrete, finite parameter —  
for example, months of the year or days of the week.

**Example:**
```c
enum months { jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec };
```

We can then declare a variable of this type:

```c
enum months monthOfYear;
```

These identifiers are **not strings**, but symbolic constants.  
Internally, C converts each identifier into an integer, starting from **0** and increasing consecutively.

We can also assign specific integer values manually:

```c
enum months { jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec };
```

This version starts from **1** instead of 0 — all subsequent identifiers automatically increase by 1.

You can even assign independent values:

```c
enum pets { dog = 101, cat = 202, hedgehog = 305, parrot = 418 };
```

In C, each identifier in an enumeration becomes an integer constant, so you can use them in arithmetic expressions:

```c
dog + parrot; // 101 + 418
```

### Defining a Boolean Type

We can create our own Boolean type in C using an enumeration:

```c
enum bool { F, T }; // F → 0, T → 1
```

## Summary

- **Structures** group multiple fields of possibly different types under one name.  
- **Unions** allow different data types to share the same memory space.  
- **Enumerations** define symbolic constants as integer values, improving readability and safety in code.

These constructs make C more expressive and organized, and they are heavily used throughout the **Harbour C API**.
