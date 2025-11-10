---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 10
chapter_title: "Pointers in C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 10. Pointers in C

As my friend Antonio says — this topic is for those aiming for top grades! 😊  
It may seem complex at first, but once you grasp the concept, it’s not so bad.  
Let’s dive in...

Before using any variable in C, you must declare it — and optionally, you may also initialize it.  
Every variable has a **value** that occupies a **memory space**.

For a program to access a variable’s value, it must know:  
1. where it is located in memory,  
2. how much space it occupies, and  
3. what type it is.

In simple terms, a **pointer** is a variable whose **content is the memory address** of another variable.

You can imagine memory as a street full of houses of equal size, each with a number (its address).  
If you know the number, you can send a letter to that house — that’s your pointer to that house. (Hopefully, that analogy helps! 😄)

## 10.1 Declaring Pointers

```c
type_variable * variable_name;  // Note the "*" symbol
```

As you can see, a pointer is declared like any other variable, specifying a type (which can be primitive or complex) and prefixing its name with an asterisk (`*`).

**Examples:**
```c
int *piAge;        // Pointer to int
float *plDistance; // Pointer to float
```

The “p” prefix is just a naming convention I use to indicate that the variable is a pointer — it’s not required.

There’s a special type of pointer called a **void pointer**.  
These are pointers that don’t have a specific type yet. When we eventually know the type, we must **cast** it, like so:

```c
void *pUnknown;
(int*) pUnknown;
```

Later, when assigning a specific value, we’ll need to perform a **forced cast**:
```c
(type_name *) pUnknown;
```

We’ll revisit this when discussing **dynamic memory allocation**, but for now, just remember that `void *` pointers exist.

## 10.2 Initializing Pointers

Declaring a pointer to a certain type is **not** the same as declaring a variable of that type.  
To avoid problems, always **initialize pointers** when you declare them.  
If you don’t yet have a valid address to assign, use the special value `NULL`.

**Examples:**
```c
int *piAge = NULL;          // Pointer to int
float *plDistance = NULL;   // Pointer to float
void *pUnknown = NULL;      // Untyped pointer
```

This practice prevents many problems, because an uninitialized pointer can contain **any random value**, which could crash your program.

## 10.3 Pointer Operators

C provides two key operators for working with pointers:

1. **Address-of operator (`&`)**  
   - Placed before a variable, it returns that variable’s memory address.

   **Example:**
   ```c
   int *piNumber = NULL;
   int iNumber = 100;

   piNumber = &iNumber;  // piNumber now contains the address of iNumber
   ```

2. **Dereference operator (`*`)**  
   - Placed before a pointer variable, it returns the value stored at that address.

   **Example:**
   ```c
   int *piNumber = NULL;
   int iNumber = 100;

   piNumber = &iNumber;  // piNumber points to iNumber

   DimeN( iNumber );   // Displays 100
   DimeN( *piNumber ); // Also displays 100
   ```

⚠️ **Important:**  
The `*` operator is used in two distinct contexts:
- In a **declaration**, it indicates that a variable is a pointer.  
- In an **expression**, it means “dereference this pointer.”

So `*` is an **overloaded operator** in this sense.

You **cannot** use dereferencing with `void` pointers directly because C needs to know the data type and memory size to retrieve the value properly.

Also, just because two pointers contain the same value (i.e., they point to the same address) doesn’t mean the pointers themselves are identical objects.

The `&` operator can only be applied to **variables**, while `*` applies to **pointers and pointer expressions**.

**Invalid example:**
```c
&( iNumber + 10 ); // ❌ incorrect
```

## 10.4 Pointer Arithmetic

C allows **adding or subtracting integers** from pointers.  
However, this is not the same as normal arithmetic with integers or floating-point numbers.

The allowed pointer operations are:
- Add or subtract an integer, resulting in another pointer of the same type.

Pointers increment (`+`) or decrement (`-`) according to the size of the data type they point to.  
If you increment a pointer by one, it moves to the **next element** of that data type in memory.

This topic will be explored further when we discuss **memory management** later.

## 10.5 Pointer to Pointer

Like any variable, a pointer can also **point to another pointer**.  
We’ll see its usefulness later, especially when dealing with **strings** and **dynamic memory**.

## 10.6 Passing Variables by Reference

In C, pointers can be used to **pass variables by reference** — meaning changes made inside a function affect the variable in the calling function.

**Example:**

Function definition:
```c
void myFunc( int *piOne, int *piTwo );
```

Usage:
```c
int iOne = 5;
int iTwo = 7;

myFunc( &iOne, &iTwo ); // Changes to iOne and iTwo will be visible here
```

This works exactly the same way as passing **by reference** in **Harbour**.

## 10.7 Example: Swapping Values

Here’s a practical example — a function that swaps the values of two variables:

```c
// Swaps the values of x and y
void swap( int *x, int *y )
{
    int temp = *x;
    *x = *y;
    *y = temp;
}
```

---

Pointers are an essential concept in C, and once you understand them,  
you’ll start to see how much control they give you over memory and performance.
