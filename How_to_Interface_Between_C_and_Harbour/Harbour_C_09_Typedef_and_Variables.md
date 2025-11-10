---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 9
chapter_title: "Creating Our Own Types with `typedef`"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 9. Creating Our Own Types with `typedef`

C allows us to assign **synonyms** (aliases) to any data type — both primitive and structured.  
This makes it much easier to define variables of those types, especially when dealing with **structures**, **unions**, and **enumerations**.  
To achieve this, we use the keyword **`typedef`**.

For example, given the following structure:

```c
struct person {
    char NIF[10];
    char name[30];
    int age;
    int gender;
};
```

We can create our own type as follows:

```c
typedef struct person TPerson;  // Now we have our own type TPerson
```

And then declare a variable of this new type:

```c
TPerson student;
```

As you can see, it’s simpler than writing:

```c
struct person student;
```

In Harbour’s include files, there are many types defined in this way —  
for instance, `PHB_ITEM`, which we’ll encounter frequently throughout this book.  

In Harbour, you’ll also find typedef-based definitions such as `HB_INT`, `HB_LONG`, `HB_BOOL`, etc.  
It’s **recommended** to use these when writing C functions for Harbour, instead of the plain C types `int`, `long`, etc.  
(Note that the native C language did not originally have a `bool` type.)

The combination of `typedef` and structures greatly simplifies the use of these types in C code.

---

## Important Summary — Data Type Recap

### Declaring a Variable
```c
type variable;
```
**Example:**
```c
int age;
```

### Assigning a Value to an Existing Variable
```c
age = 57;
```

## ⚠️ Important Notice — Variable Scope and Lifetime

### Scope (Visibility)
Depending on where a variable is declared, it can be:

- **Local:** declared inside a function; visible only within that function.  
- **Global:** declared at the beginning of a program and outside any function; visible throughout the program —  
  and even in other programs if preceded by the reserved word `extern`.

### Lifetime (Duration)
- **Global variables** remain valid for the entire duration of the program (and can even persist across modules when declared with `extern`).  
- **Local variables** cease to exist once the function they belong to finishes execution.  
- When a variable is declared as `static`, its value persists until the program ends —  
  this behavior is similar to **Harbour**’s static variables.

## Type Casting (Forced Conversion)

If you wish to store a value of one type into another, you can **cast** it explicitly.  
For example, to store an `int` into a `long`, you can cast using the following syntax:

```c
(type_name) expression
```

**Example:**
```c
int sum = 17, counter = 5;
float average;

average = (float) sum / counter;
```

Casting allows explicit conversion between compatible types.

---

That concludes the topic of **variables**.
