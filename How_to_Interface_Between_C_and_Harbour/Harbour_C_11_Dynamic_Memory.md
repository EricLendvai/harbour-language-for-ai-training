---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 11
chapter_title: "Dynamic Memory Allocation and Release"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 11. Dynamic Memory Allocation and Release

When we declare a variable, the compiler automatically allocates the memory that variable will use.  
However, there are situations where this is not possible — for example, arrays that may grow during the program’s execution.

C provides three primary functions for **dynamic memory management**:

## 11.1 Standard C Memory Functions

### `void *malloc( size_t size );`
Reserves the amount of memory specified by the `size` parameter and returns a pointer to the beginning of the allocated block.  
If the allocation fails, it returns **NULL**.

### `void *calloc( size_t number, size_t size );`
Similar to `malloc`, this function also attempts to allocate memory and returns **NULL** if unsuccessful.  
It differs in two ways:
1. It allocates memory for `number × size` bytes.  
2. It initializes all allocated bytes to **zero**.

Use **`malloc()`** when allocating memory for a single element, and **`calloc()`** when allocating space for multiple elements (such as an array).

### `void free( void *pointer );`
Releases the memory previously allocated by either `malloc()` or `calloc()`.

All three functions require **type casting**, since they return or receive a `void *` type pointer.

## 11.2 Memory Allocation in Harbour C Extensions

⚠️ **IMPORTANT:**  
Do **not** use these standard C memory functions in your C functions intended for **Harbour**,  
because Harbour’s standard library provides its own specialized memory management API.

These Harbour API functions offer automatic integration with Harbour’s internal memory tracking and are safer to use in Harbour extensions.

The functions we will use throughout this book are:

| Function | Description |
|-----------|--------------|
| `void *hb_xalloc( HB_SIZE nSize );` | Allocates `nSize` bytes of memory and returns `NULL` if it fails. |
| `void *hb_xgrab( HB_SIZE nSize );` | Allocates `nSize` bytes of memory and **terminates the program** if it fails. |
| `void *hb_xrealloc( void *pMem, HB_SIZE nSize );` | Resizes the memory block referenced by `pMem` to the new size `nSize`. |
| `void hb_xfree( void *pMem );` | Frees the memory pointed to by `pMem`. Returns nothing. |

There are other Harbour API functions related to memory management,  
but these are the ones we’ll use most frequently.

**Important note:** Always use the Harbour API functions whenever possible —  
for memory, string manipulation, file I/O, etc.  
We’ll explore those later.

## 11.3 Examples

### Allocating and Freeing Memory

```c
char *szName = ( char * ) hb_xgrab( 20 );
// Now szName can be used. If the allocation fails, the program will terminate.
```

With `hb_xalloc()`, we must explicitly check whether the memory was allocated:

```c
char *szName = ( char * ) hb_xalloc( 20 );
if( szName != NULL )
{
    // szName can be used
}
else
{
    // Handle the allocation error
}
```

To **resize** an existing memory block:

```c
szName = hb_xrealloc( szName, 50 );
```

To **free** allocated memory:

```c
hb_xfree( szName );
```

Remember: every block of memory allocated with these functions must be explicitly freed,  
since **C does not have a garbage collector**.

Throughout this book, you’ll often see **`hb_xgrab()`** and **`hb_xfree()`**.

## 11.4 A Note on Pure C vs. Harbour C

Up to this point, we’ve covered “pure C” concepts.  
Many topics remain to be explored, and some will be discussed later in more depth.  
Others will remain outside the scope of this book.  
But hopefully, these notes will serve as a foundation for understanding the vast C literature available online.

I have intentionally not included traditional pure-C examples here to avoid confusion.  
For example, a simple “Hello World” program in **pure C** might look like this:

```c
void funHelloWorld( void )
{
    printf( "Hello world" );
}
```

However, if we want to create a function callable from **Harbour**,  
we cannot use the C standard output functions such as `printf()`,  
since Harbour handles screen output through its **GT (General Terminal)** system  
or other standard Harbour libraries.

The equivalent Harbour-compatible function would be:

```c
HB_FUNC( FUNHELLOWORLD )
{
    hb_gtOutStd( "Hello world", 10 );
}
```

As you can see, there are important differences and rules to follow  
when writing C functions intended for use from Harbour code (`.prg` files).

From this point on, we’ll focus entirely on **Harbour** —  
and how to use C to expand the capabilities of our favorite language.
