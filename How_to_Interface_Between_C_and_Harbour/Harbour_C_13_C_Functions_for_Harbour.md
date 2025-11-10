---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 13
chapter_title: "Creating C Functions to Be Used from Harbour PRG Programs"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 13. Creating C Functions to Be Used from Harbour PRG Programs

As we have already mentioned, functions written in the **C language** that are intended to be used from **Harbour PRG** programs must follow certain rules.  
Let’s go over them.

All functions that execute within Harbour **do not return or receive anything directly**, and, moreover, they are of **PASCAL calling convention**.  
As stated earlier, Harbour uses the **stack** to handle both **parameters** and **return values**.

Here is the prototype skeleton of a C function:

```c
void MIFUNCION( void )
{
    ...
    // Function body
    ...
}
```

But this probably doesn’t look familiar to you... maybe this does:

```c
HB_FUNC( MIFUNCION )
{
    ...
    // Function body
    ...
}
```

This:
```c
HB_FUNC( MIFUNCION )
```
instead of:
```c
void MIFUNCION( void )
```

It’s much easier to use the first one, isn’t it?

Notice that the name is in uppercase, and practically speaking, we are not limited to 10 characters as in the legendary **Clipper**.  
This means we can now create **self-descriptive function names**.

## 13.1 Handling Data Between Harbour and C

Remember that **C** only works with **C data types**.  
To process Harbour data (which are stored as `ITEM`s), we must extract their **internal values** expressed as C types.

Once we have them, we can use the functions provided by the **C standard library**.  
However, as a best practice, we should use **Harbour’s internal C API functions** whenever possible.

We’ll see this in the examples later, but here’s a simple comparison:

| Harbour API Function | C Standard Equivalent |
|----------------------|------------------------|
| `hb_xstrcat()`       | `strcat()`             |
| `hb_xstrcpy()`       | `strcpy()`             |

The internal Harbour versions are usually **more powerful**, adapted to Harbour’s requirements — for instance, **multithreading** compatibility.  
There are internal Harbour functions for almost everything, and you can safely use them.

## 13.2 Returning Values to Harbour

Once processing is complete, we must **return the results** — converting from C types back into Harbour **ITEMs** so they can be handled from the PRG level.

For this, Harbour provides the **Extended System and the ITEM API**, which is the main subject of this book and will be explained in detail later.

## 13.3 Summary

To recap:

- Use `HB_FUNC()` to declare C functions callable from Harbour.  
- Always work through the **ITEM API** when passing or returning values between C and Harbour.  
- Prefer Harbour API equivalents (`hb_x*()`, `hb_item*()`, etc.) over the C standard library for thread safety and integration.  
- The Extended System and ITEM API will be your main tools for creating seamless C extensions.

That’s all for now…
