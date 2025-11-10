---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 16
chapter_title: "The Item API. Extending the Extended System"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 16. The Item API. Extending the Extended System

As I’ve said several times before, this was the new approach Clipper introduced with Clipper 5.00 and later. Harbour has expanded and improved it.

## What is an API in Harbour?

An **Application Programming Interface (API)** is a set of function definitions, rules, and protocols that let us write C code to be used from our Harbour PRG programs. Function signatures specify the return type and the parameter types. The **Extended System** is an API, and **the ITEM API** is also an API.

## What is an ITEM in Harbour?

We saw this earlier, but let’s revisit it now. An **ITEM** in Harbour is a C structure that represents a Harbour variable. It has two members:

- `HB_TYPE type`: holds the ITEM type as an integer (all variable types are represented).
- A C `union` whose members are structures for each Harbour type. Those structures each have a `value` member that contains the variable’s value.

```c
typedef struct _HB_ITEM
{
   HB_TYPE type;
   union
   {
      struct hb_struArray     asArray;
      struct hb_struBlock     asBlock;
      struct hb_struDateTime  asDateTime;
      struct hb_struDouble    asDouble;
      struct hb_struInteger   asInteger;
      struct hb_struLogical   asLogical;
      struct hb_struLong      asLong;
      struct hb_struPointer   asPointer;
      struct hb_struHash      asHash;
      struct hb_struMemvar    asMemvar;
      struct hb_struRefer     asRefer;
      struct hb_struEnum      asEnum;
      struct hb_struExtRef    asExtRef;
      struct hb_struString    asString;
      struct hb_struSymbol    asSymbol;
      struct hb_struRecover   asRecover;
   } item;
} HB_ITEM, * PHB_ITEM;
```

The `HB_TYPE type` member may be any of the following:

```c
/* Element types and type-checking macros */
#define HB_IT_NIL       0x00000
#define HB_IT_POINTER   0x00001
#define HB_IT_INTEGER   0x00002
#define HB_IT_HASH      0x00004
#define HB_IT_LONG      0x00008
#define HB_IT_DOUBLE    0x00010
#define HB_IT_DATE      0x00020
#define HB_IT_TIMESTAMP 0x00040
#define HB_IT_LOGICAL   0x00080
#define HB_IT_SYMBOL    0x00100
#define HB_IT_ALIAS     0x00200
#define HB_IT_STRING    0x00400
#define HB_IT_MEMOFLAG  0x00800
#define HB_IT_MEMO      ( HB_IT_MEMOFLAG | HB_IT_STRING )
#define HB_IT_BLOCK     0x01000
#define HB_IT_BYREF     0x02000
#define HB_IT_MEMVAR    0x04000
#define HB_IT_ARRAY     0x08000
#define HB_IT_ENUM      0x10000
#define HB_IT_EXTREF    0x20000
#define HB_IT_DEFAULT   0x40000
#define HB_IT_RECOVER   0x80000
#define HB_IT_OBJECT    HB_IT_ARRAY
#define HB_IT_NUMERIC   ( HB_IT_INTEGER | HB_IT_LONG | HB_IT_DOUBLE )
#define HB_IT_NUMINT    ( HB_IT_INTEGER | HB_IT_LONG )
#define HB_IT_DATETIME  ( HB_IT_DATE | HB_IT_TIMESTAMP )
#define HB_IT_ANY       0xFFFFFFFF
#define HB_IT_COMPLEX   ( HB_IT_BLOCK | HB_IT_ARRAY | HB_IT_HASH | HB_IT_POINTER | /* HB_IT_MEMVAR | HB_IT_ENUM | HB_IT_EXTREF |*/ HB_IT_BYREF | HB_IT_STRING )
#define HB_IT_GCITEM    ( HB_IT_BLOCK | HB_IT_ARRAY | HB_IT_HASH | HB_IT_POINTER | HB_IT_BYREF )
#define HB_IT_EVALITEM  ( HB_IT_BLOCK | HB_IT_SYMBOL )
#define HB_IT_HASHKEY   ( HB_IT_INTEGER | HB_IT_LONG | HB_IT_DOUBLE | HB_IT_DATE | HB_IT_TIMESTAMP | HB_IT_STRING | HB_IT_POINTER )
```

The `type` member determines which union structure contains the appropriate `value`. That union is what allows Harbour variables to hold different data types simply by assigning a given value.

### PRG illustration

```harbour
local xVar

ValType( xVar )         // -> U
xVar := 1000
ValType( xVar )         // -> N
xVar := "Hello world"
ValType( xVar )         // -> C
xVar := Date()
ValType( xVar )         // -> D
xVar := Array( 5 )
ValType( xVar )         // -> A
xVar := {||}
ValType( xVar )         // -> B
```

This is not possible in C, where variables must have a fixed type at declaration, and that type cannot change within the same scope.

The ITEM API functions are declared in `hbapiitm.h`. Broadly, they include:

### Parameter count

```c
HB_USHORT hb_itemPCount( void );  // Same as hb_pcount()
```

### Fetch parameters passed from PRG (as ITEMs)

```c
PHB_ITEM hb_param( int iParam, long lMask ); 
PHB_ITEM hb_itemParam( HB_USHORT uiParam );
PHB_ITEM hb_itemParamPtr( HB_USHORT uiParam, long lMask ); // Same as hb_param()
```

### Return an ITEM from C to PRG

```c
PHB_ITEM hb_itemReturn( PHB_ITEM pItem );
PHB_ITEM hb_itemReturnForward( PHB_ITEM pItem );
void     hb_itemReturnRelease( PHB_ITEM pItem );
```

### Get size of an ITEM (Array, Hash, or String)

```c
HB_SIZE hb_itemSize( PHB_ITEM pItem );
```

### Get current ITEM type

```c
HB_TYPE     hb_itemType( PHB_ITEM pItem );        // numeric type code
const char* hb_itemTypeStr( PHB_ITEM pItem );     // string type, like ValType()
```

### Create and free an ITEM

```c
PHB_ITEM hb_itemNew( PHB_ITEM pNull );
HB_BOOL  hb_itemRelease( PHB_ITEM pItem );
```

### Basic Array handling

```c
PHB_ITEM hb_itemArrayNew( HB_SIZE nLen );
PHB_ITEM hb_itemArrayGet( PHB_ITEM pArray, HB_SIZE nIndex );
PHB_ITEM hb_itemArrayPut( PHB_ITEM pArray, HB_SIZE nIndex, PHB_ITEM pItem );
```

### Extract C values from an ITEM

```c
char     * hb_itemGetC   ( PHB_ITEM pItem );
HB_SIZE    hb_itemGetCLen( PHB_ITEM pItem );
char     * hb_itemGetDS  ( PHB_ITEM pItem, char *szDate );
char     * hb_itemGetTS  ( PHB_ITEM pItem, char *szDateTime );
HB_BOOL    hb_itemGetL   ( PHB_ITEM pItem );
double     hb_itemGetND  ( PHB_ITEM pItem );
int        hb_itemGetNI  ( PHB_ITEM pItem );
long       hb_itemGetNL  ( PHB_ITEM pItem );
HB_MAXINT  hb_itemGetNInt( PHB_ITEM pItem );
void     * hb_itemGetPtr ( PHB_ITEM pItem );
```

### Assign C values to an ITEM (create if NULL)

```c
PHB_ITEM hb_itemPutC   ( PHB_ITEM pItem, const char * szText );
PHB_ITEM hb_itemPutCL  ( PHB_ITEM pItem, const char * szText, HB_SIZE nLen );
PHB_ITEM hb_itemPutCPtr( PHB_ITEM pItem, char * szText );
PHB_ITEM hb_itemPutD   ( PHB_ITEM pItem, int iYear, int iMonth, int iDay );
PHB_ITEM hb_itemPutDS  ( PHB_ITEM pItem, const char * szDate );
PHB_ITEM hb_itemPutTS  ( PHB_ITEM pItem, const char * szDateTime );
PHB_ITEM hb_itemPutDL  ( PHB_ITEM pItem, long lJulian );
PHB_ITEM hb_itemPutTD  ( PHB_ITEM pItem, double dTimeStamp );
PHB_ITEM hb_itemPutTDT ( PHB_ITEM pItem, long lJulian, long lMilliSec );
PHB_ITEM hb_itemPutL   ( PHB_ITEM pItem, HB_BOOL bValue );
PHB_ITEM hb_itemPutND  ( PHB_ITEM pItem, double dNumber );
PHB_ITEM hb_itemPutNI  ( PHB_ITEM pItem, int iNumber );
PHB_ITEM hb_itemPutNL  ( PHB_ITEM pItem, long lNumber );
PHB_ITEM hb_itemPutNInt( PHB_ITEM pItem, HB_MAXINT nNumber );
PHB_ITEM hb_itemPutPtr ( PHB_ITEM pItem, void * pValue );
```

### Assign NIL

```c
HB_ITEM hb_itemPutNil( PHB_ITEM pItem );
```

There are more, but these are enough for now.

## Passing parameters from PRG to C

Unlike the Extended System—where there’s one function per C type—in the ITEM API parameters arrive as **ITEMs**. In practice:

- `PHB_ITEM hb_param( int iParam, long lMask )` and  
  `PHB_ITEM hb_itemParamPtr( HB_USHORT uiParam, long lMask )` are synonyms. Both fetch a parameter as an ITEM and filter it with a type mask (`HB_TYPE`). If the parameter doesn’t match the mask, they return `NULL`.

- `PHB_ITEM hb_itemParam( HB_USHORT uiParam )` works similarly but **makes a copy** of the ITEM, so you **must** release it before leaving your function.

## Returning values from C to PRG

We always return an **ITEM** (no per-type return functions as in the Extended System). The main ones are:

- `PHB_ITEM hb_itemReturn( PHB_ITEM pItem )` — return the ITEM to PRG.
- `void hb_itemReturnRelease( PHB_ITEM pItem )` — return and free the ITEM.
- `PHB_ITEM hb_itemReturnForward( PHB_ITEM pItem )` — return an ITEM owned by Harbour (e.g., taken from the stack). The GC will free it.

## Basic handling of Harbour arrays in C

As in PRG, there are essentially three basics:

- Create an array: `PHB_ITEM hb_itemArrayNew( HB_SIZE nLen )`
- Put an element:   `PHB_ITEM hb_itemArrayPut( PHB_ITEM pArray, HB_SIZE nIndex, PHB_ITEM pItem )`
- Get an element:   `PHB_ITEM hb_itemArrayGet( PHB_ITEM pArray, HB_SIZE nIndex )`

There are many more functions for arrays which we’ll see later. Those variants avoid converting to/from ITEM by directly working with C values.

## Creating and releasing an ITEM in C

If you need to create an ITEM inside a C function, use `PHB_ITEM hb_itemNew( PHB_ITEM pNull )`. If `pNull` is another ITEM, it clones it; if `NULL`, it creates an undefined-item (`ValType() == "U"`) until you assign a value. In all cases, release the ITEM before returning with `HB_BOOL hb_itemRelease( PHB_ITEM pItem )`.

## Getting the current C value from an ITEM

Use the `hb_itemGet...()` functions listed above. The source ITEM must exist (e.g., passed from PRG or obtained from another API call).

## Assigning a C value to an ITEM

Use `hb_itemPut...()` functions. If the first parameter is `NULL`, a new ITEM is created with that value.

## Other useful ITEM API functions

- `HB_BOOL hb_itemEqual( PHB_ITEM pItem1, PHB_ITEM pItem2 )` — compare two ITEMs.
- `void hb_itemCopy( PHB_ITEM pDest, PHB_ITEM pSource )` — copy an ITEM preserving content.
- `void hb_itemMove( PHB_ITEM pDest, PHB_ITEM pSource )` — move value without increasing ref-count; source becomes `U`.
- `void hb_itemClear( PHB_ITEM pItem )` — clear to `U`.
- `PHB_ITEM hb_itemClone( PHB_ITEM pItem )` — clone an ITEM.
- `char * hb_itemStr( PHB_ITEM pNumber, PHB_ITEM pWidth, PHB_ITEM pDec )` — format numeric ITEM as string.
- `char * hb_itemString( PHB_ITEM pItem, HB_SIZE *nLen, HB_BOOL *bFreeReq )` — convert any ITEM to string; returns length and whether you must free.
- `PHB_ITEM hb_itemValToStr( PHB_ITEM pItem )` — like previous but returns a string ITEM.
- `void hb_itemSwap( PHB_ITEM pItem1, PHB_ITEM pItem2 )` — swap contents.

There are more, but this set suffices for now.

### Naming notes

- `PHB_ITEM` — pointer to an ITEM (i.e., a PRG variable as seen from C).
- `PHB_ITEM pArray` — a Harbour array as an ITEM.
- `HB_SIZE nIndex` — array index (1-based for Harbour arrays).
- `HB_SIZE nLen` — length of an array, hash, or string.

**Suffixes** in `hb_itemGet...()` / `hb_itemPut...()` mirror the Extended System:

- `N` (numbers): `NI` (int), `NL` (long), `ND` (double), `NInt` (max int)
- `D` (date): `D` (Y,M,D), `DS` (YYYYMMDD), `DL` (Julian long)
- `TS` (datetime as string)
- `Ptr` (pointer)
- `C` (string)
- `L` (logical)

> **IMPORTANT:** Any ITEM created directly via `hb_itemNew()`, or received via `hb_itemParam()` or `hb_itemPut...()` with `NULL` as first arg (e.g., `hb_itemPut...( NULL, … )`), **must** be freed with `hb_itemRelease()` to avoid memory leaks.

## Example: Variables in PRG are ITEMs in C

**PRG**

```harbour
/* 
 * ej000.prg
 * Using ITEM
 */

procedure main

    local cVar, nVar, dVar, lVar
    local aVar := Array( 4 )

    SET DATE FORMAT TO "dd/mm/yyyy"
    cls

    ? "Assignment of variables from C:"
    ? "--------------------------------"
    ?
    asignaVarC( @cVar, @nVar, @dVar, @lVar, aVar )

    ? "String...:", cVar
    ? "Number...:", nVar
    ? "Date.....:", dVar
    ? "Logical..:", lVar
    ?
    ? "Array....:"
    for i := 1 to 4
        ? i, aVar[ i ]
    next

    ?
    ? "Press ENTER to continue..."
    Inkey( 100 )

return
```

**C**

```c
/*
 * Assign values to parameters passed by reference
 */

HB_FUNC( ASIGNAVARC )
{
    PHB_ITEM cVar = hb_param( 1, HB_IT_BYREF );
    PHB_ITEM nVar = hb_param( 2, HB_IT_BYREF );
    PHB_ITEM dVar = hb_param( 3, HB_IT_BYREF );
    PHB_ITEM lVar = hb_param( 4, HB_IT_BYREF );
    PHB_ITEM aVar = hb_param( 5, HB_IT_ARRAY );

    // String
    if( cVar )
    {
        hb_itemPutC( cVar, "String assigned in C" );
    }

    // Numeric
    if( nVar )
    {
        hb_itemPutNI( nVar, 100 );
    }

    // Date
    if( dVar )
    {
        hb_itemPutDS( dVar, "20210517" );
    }

    // Logical
    if( lVar )
    {
        hb_itemPutL( lVar, HB_TRUE );
    }

    // Array
    if( aVar )
    {
        hb_itemArrayPut( aVar, 1, cVar );
        hb_itemArrayPut( aVar, 2, nVar );
        hb_itemArrayPut( aVar, 3, dVar );
        hb_itemArrayPut( aVar, 4, lVar );
    }
}
```

## Further examples with the ITEM API

(Leap year test, cube, char count/scan, arithmetic by-ref, pass/fail literals, array element update, array sum, multiplication table, weekday name, string reverse, custom `cstrtran`, libc `strtok`, and math wrappers.)

All code remains identical in PRG; only the C side uses the ITEM API variants.

## Array helpers (ITEM API)

The basic array primitives are:

- `PHB_ITEM hb_itemArrayNew( HB_SIZE nLen )`
- `PHB_ITEM hb_itemArrayGet( PHB_ITEM pArray, HB_SIZE nIndex )`
- `PHB_ITEM hb_itemArrayPut( PHB_ITEM pArray, HB_SIZE nIndex, PHB_ITEM pItem )`

These work only with `ITEM`s. If you want to set an element from a plain C value (like `char *` or `double`), you would normally create an `ITEM`, put the value into it with `hb_itemPut…()`, and then use `hb_itemArrayPut()`.

To avoid that boilerplate, Harbour adds **array helpers** that *get* and *set* C types directly, creating or reading the transient `ITEM` under the hood.

### Generic maintenance helpers

- `HB_BOOL hb_arrayNew( PHB_ITEM pItem, HB_SIZE nLen )` — turns `pItem` into an array of `nLen` elements.
- `HB_BOOL hb_arrayAdd( PHB_ITEM pArray, PHB_ITEM pItemValue )` — append (like `AAdd()`).
- `HB_BOOL hb_arrayIns( PHB_ITEM pArray, HB_SIZE nIndex )` — insert empty item at `nIndex` (like `AIns()`).
- `HB_BOOL hb_arrayDel( PHB_ITEM pArray, HB_SIZE nIndex )` — delete at `nIndex` (like `ADel()`).
- `HB_BOOL hb_arraySize( PHB_ITEM pArray, HB_SIZE nLen )` — resize (like `ASize()`).
- `HB_BOOL hb_arrayLast( PHB_ITEM pArray, PHB_ITEM pResult )` — copy last item to `pResult`.
- `HB_BOOL hb_arrayGet( PHB_ITEM pArray, HB_SIZE nIndex, PHB_ITEM pItem )` — copy element into `pItem`.
- `HB_BOOL hb_arraySet( PHB_ITEM pArray, HB_SIZE nIndex, PHB_ITEM pItem )` — set element from `pItem`.
- `HB_TYPE hb_arrayGetType( PHB_ITEM pArray, HB_SIZE nIndex )` — get element type.
- `HB_SIZE hb_arrayLen( PHB_ITEM pArray )` — `Len( aArray )`.

### Get element as C types

- `char *hb_arrayGetC( PHB_ITEM pArray, HB_SIZE nIndex )`
- `HB_SIZE hb_arrayGetCLen( PHB_ITEM pArray, HB_SIZE nIndex )`
- `HB_BOOL hb_arrayGetL( PHB_ITEM pArray, HB_SIZE nIndex )`
- `int hb_arrayGetNI( PHB_ITEM pArray, HB_SIZE nIndex )`
- `long hb_arrayGetNL( PHB_ITEM pArray, HB_SIZE nIndex )`
- `HB_MAXINT hb_arrayGetNInt( PHB_ITEM pArray, HB_SIZE nIndex )`
- `double hb_arrayGetND( PHB_ITEM pArray, HB_SIZE nIndex )`
- `char *hb_arrayGetDS( PHB_ITEM pArray, HB_SIZE nIndex, char * szDate )`
- `long hb_arrayGetDL( PHB_ITEM pArray, HB_SIZE nIndex )`

### Set element from C types

- `HB_BOOL hb_arraySetDS( PHB_ITEM pArray, HB_SIZE nIndex, const char *szDate )`
- `HB_BOOL hb_arraySetDL( PHB_ITEM pArray, HB_SIZE nIndex, long lDate )`
- `HB_BOOL hb_arraySetL( PHB_ITEM pArray, HB_SIZE nIndex, HB_BOOL fValue )`
- `HB_BOOL hb_arraySetNI( PHB_ITEM pArray, HB_SIZE nIndex, int iNumber )`
- `HB_BOOL hb_arraySetNL( PHB_ITEM pArray, HB_SIZE nIndex, long lNumber )`
- `HB_BOOL hb_arraySetNInt( PHB_ITEM pArray, HB_SIZE nIndex, HB_MAXINT nNumber )`
- `HB_BOOL hb_arraySetND( PHB_ITEM pArray, HB_SIZE nIndex, double dNumber )`
- `HB_BOOL hb_arraySetC( PHB_ITEM pArray, HB_SIZE nIndex, const char *szText )`

### Example 1 — process an array by reference and create a new array

**PRG**

```harbour
/* Process an array and create another from C (ITEM API helpers) */

PROCEDURE Main()

    LOCAL aDesdeC
    LOCAL aArray := { 12.30, 11, 20, 3, 23, 89, 5, 15, 33.75, 1.98 }
    LOCAL nLen := Len( aArray )
    LOCAL nSuma := 100.55    // Try changing this number

    SET DATE FORMAT TO "dd-mm-yyyy"
    CLS

    // Adds nSuma to each element (array passed by reference)
    tramita( aArray, nSuma )

    ?
    ? "Value added:", nSuma
    ? "------------------------------------------------------------------------"
    ? "Previous values", "New values"
    ?

    FOR i := 1 TO nLen
        ? aArray[ i ] - nSuma, aArray[ i ]
    NEXT

    ?
    ? "Press a key to continue..."
    Inkey( 100 )
    CLS

    ?
    ? "------------------------------------------------------------------------"
    ? "Array created in C:"
    ?

    aDesdeC := creaArray()
    nLen := Len( aDesdeC )

    FOR i := 1 TO nLen
        ? "Element " + HB_NToS( i ), aDesdeC[ i ]
    NEXT

    Inkey( 100 )

RETURN
```

**C**

```c
/* Adds a numeric delta to every numeric element of an array (by reference) */
HB_FUNC( TRAMITA )
{
   PHB_ITEM aDatos = hb_param( 1, HB_IT_ARRAY );

   if( aDatos )
   {
      PHB_ITEM nMod = hb_param( 2, HB_IT_NUMERIC );

      if( nMod )
      {
         HB_SIZE nLen = hb_arrayLen( aDatos );
         HB_SIZE i;
         double dMod = hb_itemGetND( nMod );

         for( i = 1; i <= nLen; i++ )
         {
            hb_arraySetND( aDatos, i, hb_arrayGetND( aDatos, i ) + dMod );
         }
      }
   }
}

/* Create and fill an array with mixed types */
HB_FUNC( CREAARRAY )
{
   PHB_ITEM aArray = hb_itemNew( NULL );  /* empty item */
   PHB_ITEM xItem  = hb_itemNew( NULL );

   /* Turn empty item into a 10-element array.
      (Could also do: aArray = hb_itemArrayNew( 10 ) ) */
   hb_arrayNew( aArray, 10 );

   hb_arraySetDS( aArray, 1, "20211231" );
   hb_arraySetDL( aArray, 2, hb_dateEncStr( "20211231" ) );
   hb_arraySetL(  aArray, 3, HB_FALSE );
   hb_arraySetNI( aArray, 4, 35789 );
   hb_arraySetNL( aArray, 5, 15578952 );
   hb_arraySetNInt( aArray, 6, 95415545541 );
   hb_arraySetND( aArray, 7, 34954155455.41 );
   hb_arraySetC(  aArray, 8, "Esto es una prueba de cadena" );

   /* Put an uninitialized item (nil) at pos 9 */
   hb_arraySet( aArray, 9, xItem );

   /* Now assign a string to xItem and store at pos 10 */
   hb_itemPutC( xItem, "Asignacion del item con valor tipo cadena" );
   hb_arraySet( aArray, 10, xItem );

   hb_itemRelease( xItem );
   hb_itemReturnRelease( aArray );
}
```

## Gather/Scatter using arrays

A practical buffer pattern: read a DBF record into an array (`aGather()`), modify it, and write it back (`aScatter()`).
Below, the PRG driver and the two C functions.

**PRG**

```harbour
PROCEDURE Main()

    LOCAL aBuf

    CLS
    USE test NEW

    test->( DBGoTo( 10 ) )
    aBuf := test->( aGather() )   // read current record

    ? "------------------------------------------------------------------------"
    ? "Type: " + ValType( aBuf ), "  /  Elements: " + HB_NToS( Len( aBuf ) )
    ? "------------------------------------------------------------------------"
    ? test->( RecNo() ), aBuf[ 2 ], aBuf[ 9 ]
    ? "------------------------------------------------------------------------"
    ? "Change AGE with a random integer"
    aBuf[ 9 ] := HB_RandomInt( 0, 99 )
    ? "Buffer value:", aBuf[ 9 ]
    test->( aScatter( aBuf ) )    // write back
    ? "------------------------------------------------------------------------"
    ? "DBF AGE is now", test->( FieldGet( 9 ) )
    ? "------------------------------------------------------------------------"
    Inkey( 100 )

RETURN
```

**C**

```c
/* Read current record into an array buffer */
HB_FUNC( AGATHER )
{
   PHB_ITEM aBuffer = hb_itemNew( NULL );
   PHB_ITEM pValue  = hb_itemNew( NULL );
   AREAP pArea = ( AREAP ) hb_rddGetCurrentWorkAreaPointer();
   HB_USHORT uiFields, i;

   SELF_FIELDCOUNT( pArea, &uiFields );
   hb_arrayNew( aBuffer, uiFields );

   for( i = 1; i <= uiFields; i++ )
   {
      SELF_GETVALUE( pArea, i, pValue );
      hb_arraySet( aBuffer, i, pValue );
   }

   hb_itemRelease( pValue );
   hb_itemReturnRelease( aBuffer );
}

/* Write an array buffer to the current record */
HB_FUNC( ASCATTER )
{
   PHB_ITEM aBuffer = hb_param( 1, HB_IT_ARRAY );

   if( aBuffer )
   {
      AREAP pArea = ( AREAP ) hb_rddGetCurrentWorkAreaPointer();
      HB_USHORT uiFields, i;
      PHB_ITEM pValue = hb_itemNew( NULL );

      SELF_FIELDCOUNT( pArea, &uiFields );

      for( i = 1; i <= uiFields; i++ )
      {
         hb_arrayGet( aBuffer, i, pValue );
         SELF_PUTVALUE( pArea, i, pValue );
      }

      hb_itemRelease( pValue );
   }
}
```

> **Note on dates:** like most languages, DBF stores dates as a long integer (Julian.)

## Hash tables (associative arrays)

Harbour hashes are (key,value) containers optimized for key lookups. In C with the ITEM API, the hash object and both key/value are all `ITEM`s. That means we still create/assign `ITEM`s and let the API manage references.

### Core hash functions

- `PHB_ITEM hb_hashNew( PHB_ITEM pItem )`
- `HB_SIZE  hb_hashLen( PHB_ITEM pHash )`
- `HB_BOOL  hb_hashDel( PHB_ITEM pHash, PHB_ITEM pKey )`
- `HB_BOOL  hb_hashAdd( PHB_ITEM pHash, PHB_ITEM pKey, PHB_ITEM pValue )`
- `HB_BOOL  hb_hashAddNew( PHB_ITEM pHash, PHB_ITEM pKey, PHB_ITEM pValue )`
- `HB_BOOL  hb_hashRemove( PHB_ITEM pHash, PHB_ITEM pItem )`
- `HB_BOOL  hb_hashClear( PHB_ITEM pHash )`
- `void     hb_hashSort( PHB_ITEM pHash )`
- `PHB_ITEM hb_hashClone( PHB_ITEM pHash )`
- `PHB_ITEM hb_hashCloneTo( PHB_ITEM pDest, PHB_ITEM pHash )`
- `void     hb_hashJoin( PHB_ITEM pDest, PHB_ITEM pSource, int iType )`
- `HB_BOOL  hb_hashScan( PHB_ITEM pHash, PHB_ITEM pKey, HB_SIZE * pnPos )`
- `HB_BOOL  hb_hashScanSoft( PHB_ITEM pHash, PHB_ITEM pKey, HB_SIZE * pnPos )`
- `void     hb_hashPreallocate( PHB_ITEM pHash, HB_SIZE nNewSize )`
- `PHB_ITEM hb_hashGetKeys( PHB_ITEM pHash )`
- `PHB_ITEM hb_hashGetValues( PHB_ITEM pHash )`
- `void     hb_hashSetDefault( PHB_ITEM pHash, PHB_ITEM pValue )`
- `PHB_ITEM hb_hashGetDefault( PHB_ITEM pHash )`
- `void     hb_hashSetFlags( PHB_ITEM pHash, int iFlags )`
- `void     hb_hashClearFlags( PHB_ITEM pHash, int iFlags )`
- `int      hb_hashGetFlags( PHB_ITEM pHash )`
- `void *   hb_hashId( PHB_ITEM pHash )`
- `HB_COUNTER hb_hashRefs( PHB_ITEM pHash )`

### Example — build a hash in C, iterate in PRG

**PRG**

```harbour
PROCEDURE Main()

    LOCAL hTabla, xClave, xPar
    LOCAL nLen, i

    CLS

    ?
    ? "------------------------------------------------------------------------"
    ? "Hash created in C:"
    ? "------------------------------------------------------------------------"
    ?

    hTabla := creaHash()
    nLen := Len( hTabla )

    hb_HCaseMatch( hTabla, .f. )  // case-insensitive

    ?
    ? "Hash has " + HB_NToS( nLen ) + " pairs (key,value)"
    ?
    ? "------------------------------------------------------------------------"
    ? "Plain FOR/NEXT:"
    ? "------------------------------------------------------------------------"
    FOR i := 1 TO nLen
        xClave := hb_HKeyAt( hTabla, i )
        ? "Key: ", xClave, " -> Value: ", hb_HGet( hTabla, xClave )
    NEXT

    ? "------------------------------------------------------------------------"
    ? "FOR EACH (recommended):"
    ? "------------------------------------------------------------------------"
    FOR EACH xPar IN hTabla
        ? "Key: ", xPar:__enumKey(), " -> Value: ", xPar:__enumValue()
    NEXT

    Inkey( 100 )

RETURN
```

**C**

```c
/* Build a small sample hash with multiple types */
HB_FUNC( CREAHASH )
{
   PHB_ITEM hHash  = hb_hashNew( NULL );
   PHB_ITEM pKey   = hb_itemNew( NULL );
   PHB_ITEM pValue = hb_itemNew( NULL );

   /* Dates */
   hb_itemPutC(  pKey, "fechaDS" );
   hb_itemPutDS( pValue, "20211231" );
   hb_hashAdd( hHash, pKey, pValue );

   hb_itemPutC(  pKey, "fechaDL" );
   hb_itemPutDL( pValue, hb_dateEncStr( "20211231" ) );
   hb_hashAdd( hHash, pKey, pValue );

   /* Logical */
   hb_itemPutC(  pKey, "logico" );
   hb_itemPutL(  pValue, HB_TRUE );
   hb_hashAdd( hHash, pKey, pValue );

   /* Integers */
   hb_itemPutC(  pKey, "entero" );
   hb_itemPutNI( pValue, 35789 );
   hb_hashAdd( hHash, pKey, pValue );

   hb_itemPutC(  pKey, "largo" );
   hb_itemPutNL( pValue, 15578952 );
   hb_hashAdd( hHash, pKey, pValue );

   /* Double */
   hb_itemPutC(  pKey, "real" );
   hb_itemPutND( pValue, 34954155455.41 );
   hb_hashAdd( hHash, pKey, pValue );

   hb_itemRelease( pKey );
   hb_itemRelease( pValue );

   hb_itemReturnRelease( hHash );
}
```

## Gather/Scatter using hashes

A more convenient buffer uses field *names* as keys instead of numeric positions.

**PRG**

```harbour
PROCEDURE Main()

    LOCAL hBuf

    CLS
    USE test NEW

    test->( DBGoTo( 10 ) )
    hBuf := test->( hGather() )  // read current record

    ? "------------------------------------------------------------------------"
    ? "Type: " + ValType( hBuf ), "  /  Elements: " + HB_NToS( Len( hBuf ) )
    ? "------------------------------------------------------------------------"
    ? test->( RecNo() ), hBuf[ "Last" ], hBuf[ "age" ]
    ? "------------------------------------------------------------------------"
    ? "Change AGE with a random integer"
    hBuf[ "age" ] := HB_RandomInt( 0, 99 )
    ? "Buffer value:", hBuf[ "age" ]
    test->( hScatter( hBuf ) )     // write back
    ? "------------------------------------------------------------------------"
    ? "DBF AGE is now", test->( FieldGet( FieldPos( "age" ) ) )
    ? "------------------------------------------------------------------------"
    Inkey( 100 )

RETURN
```

**C**

```c
/* Read current record into a hash buffer (key = field name) */
HB_FUNC( HGATHER )
{
   PHB_ITEM hBuffer = hb_hashNew( NULL );
   PHB_ITEM pKey    = hb_itemNew( NULL );
   PHB_ITEM pValue  = hb_itemNew( NULL );
   AREAP pArea = ( AREAP ) hb_rddGetCurrentWorkAreaPointer();
   char *szFldName = ( char * ) hb_xgrab( pArea->uiMaxFieldNameLength + 1 );
   HB_USHORT uiFields, i;

   hb_hashClearFlags( hBuffer, HB_HASH_BINARY );
   hb_hashSetFlags( hBuffer, HB_HASH_IGNORECASE | HB_HASH_RESORT );

   SELF_FIELDCOUNT( pArea, &uiFields );
   hb_hashPreallocate( hBuffer, uiFields );

   for( i = 1; i <= uiFields; i++ )
   {
      SELF_FIELDNAME( pArea, i, szFldName );
      hb_itemPutC( pKey, szFldName );
      SELF_GETVALUE( pArea, i, pValue );
      hb_hashAdd( hBuffer, pKey, pValue );
   }

   hb_itemRelease( pKey );
   hb_itemRelease( pValue );
   hb_xfree( szFldName );

   hb_itemReturnRelease( hBuffer );
}

/* Write a hash buffer back to the current record */
HB_FUNC( HSCATTER )
{
   PHB_ITEM hBuffer = hb_param( 1, HB_IT_HASH );

   if( hBuffer )
   {
      AREAP pArea = ( AREAP ) hb_rddGetCurrentWorkAreaPointer();
      HB_USHORT uiFields, i;

      SELF_FIELDCOUNT( pArea, &uiFields );

      for( i = 1; i <= uiFields; i++ )
      {
         SELF_PUTVALUE( pArea, i, hb_hashGetValueAt( hBuffer, i ) );
      }
   }
}
```

## `TWABuffer` — a tiny work-area buffer class

Finally, a class that encapsulates a DBF work-area buffer as a hash, minimizing loose variables and wiring.

**Public methods**

- `new()` – initialize (binds to the current work area)
- `load()` – read current record into buffer
- `save()` – write buffer back to current record
- `blank()` – clear buffer with blank values (via phantom record 0)
- `getBuffer()` – return the underlying hash
- `set( cFieldName, xValue )` – assign a value
- `get( cFieldName )` – fetch a value
- `getAlias()` – current alias (like `Alias()`)
- `getArea()` – current work area number (like `Select()`)
- `getLen()` – number of pairs in the buffer

**Definition in Harbour; implementation in C**

```harbour
// thbuffer.prg (class definition)

#include "HBClass.ch"

CREATE CLASS TWABuffer

    DATA iData PROTECTED  // internal pointer to C-side state

    CONSTRUCTOR new()
    METHOD load()
    METHOD save()
    METHOD blank()
    METHOD getBuffer()
    METHOD set( cFieldName, xValue )
    METHOD get( cFieldName )
    METHOD getAlias()
    METHOD getArea()
    METHOD getLen()

   PROTECTED:
    DESTRUCTOR free()

END CLASS
```

**C implementation (selected parts, fully translated & annotated)**

```c
#pragma BEGINDUMP

#define _IDATA 1

#include "hbapi.h"
#include "hbapiitm.h"
#include "hbapirdd.h"
#include "hbstack.h"

typedef struct
{
    AREAP    pArea;
    PHB_ITEM hBuffer;
    HB_USHORT uiFields;
} DATA, *PDATA;

/* Constructor */
HB_FUNC_STATIC( TWABUFFER_NEW )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    AREAP pArea = ( AREAP ) hb_rddGetCurrentWorkAreaPointer();

    if( pArea )
    {
        PDATA pData = ( PDATA ) hb_xgrab( sizeof( DATA ) );
        PHB_ITEM hBuffer = hb_hashNew( NULL );
        PHB_ITEM pKey = hb_itemNew( NULL );
        HB_USHORT uiFields, i;
        char *szFldName = ( char * ) hb_xgrab( pArea->uiMaxFieldNameLength + 1 );

        SELF_FIELDCOUNT( pArea, &uiFields );
        hb_hashPreallocate( hBuffer, uiFields );

        hb_hashClearFlags( hBuffer, HB_HASH_BINARY );
        hb_hashSetFlags( hBuffer, HB_HASH_IGNORECASE | HB_HASH_RESORT );

        for( i = 1; i <= uiFields; i++ )
        {
            SELF_FIELDNAME( pArea, i, szFldName );
            hb_itemPutC( pKey, szFldName );
            hb_hashAdd( hBuffer, pKey, NULL );
        }

        pData->pArea    = pArea;
        pData->hBuffer  = hBuffer;
        pData->uiFields = uiFields;

        hb_arraySetPtr( pSelf, _IDATA, ( void * ) pData );
        hb_itemRelease( pKey );
        hb_xfree( szFldName );
    }

    hb_itemReturnForward( pSelf );
}

/* Load current record into the buffer */
HB_FUNC_STATIC( TWABUFFER_LOAD )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    PHB_ITEM pValue = hb_itemNew( NULL );
    HB_USHORT i;

    for( i = 1; i <= pData->uiFields; i++ )
    {
        SELF_GETVALUE( pData->pArea, i, pValue );
        hb_hashAdd( pData->hBuffer, hb_hashGetKeyAt( pData->hBuffer, i ), pValue );
    }

    hb_itemRelease( pValue );
    hb_itemReturn( pData->hBuffer );
}

/* Save buffer to current record */
HB_FUNC_STATIC( TWABUFFER_SAVE )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    HB_USHORT i;

    for( i = 1; i <= pData->uiFields; i++ )
    {
        SELF_PUTVALUE( pData->pArea, i, hb_hashGetValueAt( pData->hBuffer, i ) );
    }
}

/* Blank buffer using phantom record 0 */
HB_FUNC_STATIC( TWABUFFER_BLANK )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    PHB_ITEM pRecNo = hb_itemNew( NULL );
    PHB_ITEM pRec0  = hb_itemPutNL( NULL, 0 );

    SELF_RECID(  pData->pArea, pRecNo );
    SELF_GOTOID( pData->pArea, pRec0 );
    HB_FUNC_EXEC( TWABUFFER_LOAD );
    SELF_GOTOID( pData->pArea, pRecNo );

    hb_itemRelease( pRec0 );
    hb_itemRelease( pRecNo );

    hb_itemReturn( pData->hBuffer );
}

/* Get underlying hash */
HB_FUNC_STATIC( TWABUFFER_GETBUFFER )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    hb_itemReturn( ( ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA ) )->hBuffer );
}

/* Get value by field name */
HB_FUNC_STATIC( TWABUFFER_GET )
{
    PHB_ITEM cKey = hb_param( 1, HB_IT_STRING );
    PHB_ITEM pRes = NULL;

    if( cKey )
    {
        PHB_ITEM pSelf = hb_stackSelfItem();
        PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
        pRes = hb_hashGetItemPtr( pData->hBuffer, cKey, HB_HASH_AUTOADD_ACCESS );
    }

    hb_itemReturn( pRes );
}

/* Set value by field name */
HB_FUNC_STATIC( TWABUFFER_SET )
{
    PHB_ITEM pKey = hb_param( 1, HB_IT_STRING );
    PHB_ITEM pValue = hb_param( 2, HB_IT_ANY );
    HB_BOOL fRes = HB_FALSE;

    if( pKey && pValue )
    {
        PHB_ITEM pSelf = hb_stackSelfItem();
        PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );

        fRes = hb_hashScan( pData->hBuffer, pKey, NULL );
        if( fRes )
            hb_hashAdd( pData->hBuffer, pKey, pValue );
    }

    hb_itemPutL( hb_stackReturnItem(), fRes );
}

/* Count pairs */
HB_FUNC_STATIC( TWABUFFER_GETLEN )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    hb_itemPutNI( hb_stackReturnItem(), pData->uiFields );
}

/* Alias of bound work area */
HB_FUNC_STATIC( TWABUFFER_GETALIAS )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    char *szAlias = ( char * ) hb_xgrab( HB_RDD_MAX_ALIAS_LEN + 1 );

    SELF_ALIAS( pData->pArea, szAlias );
    hb_itemPutC( hb_stackReturnItem(), szAlias );
    hb_xfree( szAlias );
}

/* Work area number (Select()) */
HB_FUNC_STATIC( TWABUFFER_GETAREA )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );
    hb_itemPutNI( hb_stackReturnItem(), pData->pArea->uiArea );
}

/* Destructor */
HB_FUNC_STATIC( TWABUFFER_FREE )
{
    PHB_ITEM pSelf = hb_stackSelfItem();
    PDATA pData = ( PDATA ) hb_arrayGetPtr( pSelf, _IDATA );

    if( pData )
    {
        hb_itemRelease( pData->hBuffer );
        hb_xfree( pData );
        hb_arraySetPtr( pSelf, _IDATA, NULL );
    }
}

#pragma ENDDUMP
```

### Takeaways

- ITEM API lets you work **directly with Harbour variables** (`ITEM`s) in C.
- Array/hash helpers remove a lot of glue code when you need C primitives in/out.
- Gather/Scatter patterns are straightforward and can be wrapped in classes for cleaner PRG code.

> Next chapter: the new ITEM API extensions for arrays and hashes in more advanced scenarios.
