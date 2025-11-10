---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 7
chapter_title: "Structured Data Types: Arrays, Vectors, Matrices, and Multidimensional Tables in C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 7. Structured Data Types: Arrays, Vectors, Matrices, and Multidimensional Tables in C

First, what are they?

They are called *structured* or *complex types* because they are composed of the **elementary types** already discussed.  
This same definition will also apply to the structures we’ll examine in the next chapter.

The concept of a *table* is exactly the same as in **Harbour**, but only conceptually.  
In general, we can say that a table is a **structured variable** that contains elements of the same type, stored consecutively in memory.

We must remember that an array (or table) in **Harbour** is a variable of type `'A'` that can contain elements of **any type** supported by Harbour.  
In **C**, however, arrays can contain **only elements of the same type**, which is a major difference.

C needs to know the size of the type that makes up the array in order to lay it out properly in memory.  
Among other things, this allows the program to iterate through its elements using repetition statements such as `for` or `while`.

You can calculate the number of elements in an array by dividing the total memory occupied by the array by the size of one element.  
The `sizeof` operator is very useful for this, as it returns the size of a type or variable.

**Example:**
```c
iWidthInt = sizeof( int ); // Note: 'int' is the type, not a variable
```

## 7.1 Vector (One-Dimensional Array)

A **vector** is a one-dimensional array, meaning its contents are not other arrays.

Declaration:
```c
type name[ dimension ]; // dimension = number of elements
```

**Example:**
```c
long lDistances[3] = { 2445456, 123785, 9842456 };
```

Important: in C, there is **no native string type**.  
In C, a *string* is a **character array**.

**Examples (all equivalent):**
```c
char szString[5] = { 'h', 'o', 'l', 'a', '\0' };
char szString[]  = { 'h', 'o', 'l', 'a', '\0' };
char szString[5] = "hola";
char szString[]  = "hola";
```

Or using pointer notation:
```c
char *szString = "hola";
```

This pointer notation will be covered later in the topic on **pointers**.  
For now, just take my word for it. 🙂

Notice in the first notation the special character `'\0'`, which indicates the end of the string.  
That’s why, in my Hungarian notation, I use the prefix **`sz`** in variable names like `szString` — to remind you that it’s a **string (s)** terminated by **zero (z)**.

## 7.2 Matrix (Two-Dimensional Array)

A **matrix** is a two-dimensional array — that is, it contains two vectors.

Declaration:
```c
type name[ dim1 ][ dim2 ];
```

**Example:**
```c
long lDistances[2][3] = {
    { 2445456, 123785, 9842456 },
    { 3456454, 754332, 4656567 }
};
```

## 7.3 Multidimensional Array

A **multidimensional array** is an array containing more than two vectors.  
Technically, a matrix is a two-dimensional multidimensional array — but we programmers like to make that distinction.  
In reality, vectors and matrices are the most commonly used forms.

Multidimensional arrays are rarely initialized this way, but the following example is given to help visualize them.

Declaration:
```c
type name[ dim1 ][ dim2 ][ dim3 ]; // etc.
```

**Example:**
```c
long lDistances[2][3][3] = {
    { { 2445456, 123785, 9842456 }, { 3456454, 754332, 4656567 } },
    { { 2445456, 123785, 9842456 }, { 3456454, 754332, 4656567 } },
    { { 2445456, 123785, 9842456 }, { 3456454, 754332, 4656567 } }
};
```

But really, C doesn’t care about visual representation.  
It simply uses the dimensions specified in the brackets to manage memory slicing.

So yes — you could write the same initialization *linearly*, and the C compiler would not complain.  
Even better, it would treat it exactly the same:

```c
long lDistances[2][3][3] = {
    2445456, 123785, 9842456, 3456454, 754332, 4656567,
    2445456, 123785, 9842456, 3456454, 754332, 4656567,
    2445456, 123785, 9842456, 3456454, 754332, 4656567
};
```

## 7.4 Accessing Array Elements

To access or assign a value, use **indices in brackets**:

```c
lDistance[2][1][1];
```

Important: in C, array indices start at **0**, not **1** as in Harbour.

**Example:**
```c
char szString[5] = "hola";
// To get the second character:
szString[1]; // The first would be szString[0]
```
