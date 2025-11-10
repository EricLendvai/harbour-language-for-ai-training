---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 15
chapter_title: "The Harbour Extended System"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 15. The Harbour Extended System

A very important capability programming languages can have is the integration with the omnipotent **C language**. For example, Java programmers rely on the **Java Native Interface (JNI)**. This gives those languages virtually unlimited expansion capacity.

Harbour is no different and—better yet—instead of just one mechanism we have **two** ways to integrate C into our applications:

- **The Extended System**
- **The ITEM API**

You can even mix both technologies. In this chapter we will focus on the first one.

Old **Clipper** has always implemented the Extended System and, starting from the 5.xx versions, it also introduced the **ITEM API**. As you know, Harbour is a modern, improved version of old Clipper; it inherits everything and improves it across the board. Therefore, the Extended System is included in that inheritance. You can think of it as the *traditional* way to integrate C functions into our `.prg` programs.

As we said earlier, C functions intended to be called from PRG must meet certain conditions: they **do not return anything directly** and **do not accept parameters**. In C that shape would be:

```c
void MIFUNCION( void )
{
    // Function body
}
```

Harbour simplifies the declaration a bit, letting us write:

```c
HB_FUNC( MIFUNCION )
{
    // Function body
}
```

In other words, `HB_FUNC( MIFUNCION )` is an easier way to write `void MIFUNCION( void )`.

So if, as we see, these functions neither return nor receive parameters, **how can we pass parameters from a PRG and receive values back?** That’s where the **HVM**, **symbols**, and the **stack** come in.

These functions use the **stack** to receive parameters and the **return area** (*stackReturn*) to place the result.

Every time a PRG calls a function, the HVM **pushes** all parameters onto the stack. Conceptually:

```
PUSH  "Function_or_Method_Symbol"
PUSH  "NIL for functions or the object symbol for methods"
PUSH  "param1"
PUSH  "param2"
...
PUSH  "paramN"
DO( NumParams )  // executes and pops
```

`DO` can be one of these three:

- `hb_vmDo( nPar )` for functions, procedures, and methods
- `hb_vmProc( nPar )` for procedures and functions
- `hb_vmSend( nPar )` for object methods

(There are others, but these three are the most important. In the next chapter—where we’ll see how to execute PRG functions from C—we’ll revisit this in more detail.)

Extended System functions **retrieve parameters** that were pushed on the stack by their **positional index**, and they **place the return value** in the return area of the stack. That’s the magic.

From this we can say there are **two** main kinds of Extended System helper functions:

1. Those that **retrieve** parameters from the stack by position.
2. Those that **write** the **return value** into the stack’s return area.

The first group are the **`hb_par…()`** functions (e.g., `hb_parc()`, `hb_parni()`, etc.).  
The second group are the **`hb_ret…()`** functions (e.g., `hb_retc()`, `hb_retni()`, etc.).  
In addition, for parameters **passed by reference**, there are the **`hb_stor…()`** functions (e.g., `hb_storc()`, `hb_storni()`).

## Parameter Passing

These are the main functions for **retrieving** passed parameters:

```c
const char * hb_parc( int iParam );
const char * hb_parcx( int iParam );
HB_SIZE      hb_parclen( int iParam );
HB_SIZE      hb_parcsiz( int iParam );
const char * hb_pards( int iParam );
long         hb_pardl( int iParam );
double       hb_partd( int iParam );
int          hb_parl( int iParam );
int          hb_parldef( int iParam, int iDefValue );
double       hb_parnd( int iParam );
int          hb_parni( int iParam );
int          hb_parnidef( int iParam, int iDefValue );
long         hb_parnl( int iParam );
long         hb_parnldef( int iParam, long lDefValue );
HB_MAXINT    hb_parnint( int iParam );
HB_MAXINT    hb_parnintdef( int iParam, HB_MAXINT nDefValue );
void *       hb_parptr( int iParam );
```

There are more. **Boldface** in the original listed the ones present in Clipper 5.xx; the rest are Harbour extensions. We’ll first explain the classic ones and the most useful new ones.

Instead of going one by one, here is a **legend** of the **return types** (what each function returns), i.e., C types:

- `const char *` → a constant string
- `HB_SIZE` → a very large integer (like `long long`)
- `HB_ISIZ` → a very large integer (like `long long`)
- `long` → a long integer
- `double` → a double-precision real
- `HB_MAXINT` → the maximum available wide integer type (`long`/`long long`)
- `void *` → a typeless pointer

And for the **parameters**:

- `int iParam` → positional index of the parameter (1-based from PRG’s perspective)
- `DefValue` → default value if the PRG did not pass that parameter

It’s **very important** to understand this mapping.

**Example (PRG calling a C function):**

```harbour
PROCEDURE Main()
   LOCAL nAge, lIsAdult
   nAge := 57
   lIsAdult := miMayorEdad( nAge )
   Alert( "He is " + IIF( lIsAdult, "an adult", "a minor" ) )
RETURN
```

In the C function `miMayorEdad` we retrieve the parameter like this:

```c
int hb_parni( 1 );  // Retrieves parameter 1 as an integer
```

## Returning Values from C

The “return” functions are basically **symmetric** to the parameter ones: they can accept the same kinds of data that the `hb_par…()` group returns—except in this case we **pass** those values **to** the `hb_ret…()` functions. Main ones:

```c
void   hb_ret( void );
void   hb_retc( const char * szText );
void   hb_retclen( const char * szText, HB_SIZE nLen );
void   hb_retc_null( void );
void   hb_retc_buffer( char * szText );
void   hb_retclen_buffer( char * szText, HB_SIZE nLen );
void   hb_retds( const char * szDate );
void   hb_retd( int iYear, int iMonth, int iDay );
void   hb_retl( int iTrueFalse );
void   hb_retnd( double dNumber );
void   hb_retni( int iNumber );
void   hb_retnl( long lNumber );
void   hb_retnint( HB_MAXINT nNumber );
void   hb_reta( HB_SIZE nLen );
void   hb_retptr( void * ptr );
```

(Again, the ones in the original **bold** were supported by Clipper; the rest are Harbour extensions.)

Accepted parameter types (what you pass **into** `hb_ret…()` so Harbour receives them as a PRG value):

- `const char *` → constant string
- `char *` → mutable string
- `HB_SIZE` / `HB_ISIZ` → very large integer
- `int` → integer
- `long` → long integer
- `double` → double-precision real
- `void` → no parameters
- `HB_MAXINT` → max-width integer

**Naming hints** used in both families (`hb_par…()`, `hb_ret…()`):  
- `c` → strings (`hb_parc`, `hb_retc`)  
- `n` → numbers; followed by `i` (integer) or `l` (long): `hb_parni`, `hb_parnl`  
- `l` → logicals (booleans)  
- `d` → dates  
- `a` → arrays

Also note: `hb_retc_buffer()` and `hb_retclen_buffer()` work like their non-`_buffer` counterparts **but free** previously allocated memory.

## Upgrading the Build Script (Batch)

Let’s enhance the earlier batch file to stop on errors and run the program on success (MinGW 64-bit example, `do_mingw64.bat`):

```bat
@set comp=mingw64
@set DIR_HBBIN=u:\desarrollo\comp\xc\hb\bin
@set DIR_CCBIN=u:\desarrollo\comp\cc\mingw\64\9.30\bin
@set PATH=%DIR_HBBIN%;%DIR_CCBIN%;%PATH%
@hbmk2 -comp=%comp% %1 fuentes_c\cursode.c
@if %errorlevel% neq 0 goto bld_error
@cls
%1
goto fin_exec
:bld_error
@echo -----------------------------------
@echo    There were compilation errors
@echo -----------------------------------
pause
:fin_exec
```

## Example 1 — Calling C Functions from PRG

**`ej001.prg`**

```harbour
//------------------------------------------------------------------------------
// Exercise: using C functions
// ej001.prg
//------------------------------------------------------------------------------

PROCEDURE Main()

   LOCAL getlist := {}
   LOCAL nYear := Year( Date() )
   LOCAL nCube := 0
   LOCAL cNum, i, nDay := 0, cDay
   LOCAL cString := Space( 60 )
   LOCAL cChar := " "
   LOCAL n1 := 0, n2 := 0, nS, nR, nP, nD
   LOCAL nGrade := 0, cStatus
   LOCAL aNum := { 10, 20, 30 }

   CLS

   separa( .f. )

   @ 02, 01 SAY "C Course for Harbour"

   separa( .f. )

   @ 05, 01 SAY "Enter a year to check leap year...................:" GET nYear PICTURE "@K 9999"
   @ 06, 01 SAY "Enter a number to compute its cube................:" GET nCube PICTURE "@K 9999"
   @ 07, 01 SAY "Enter a string....................................:"
   @ 08, 10 GET cString PICTURE "@K"
   @ 09, 01 SAY "Enter a character to count in the string..........:" GET cChar PICTURE "@K"
   @ 10, 01 SAY "Enter a number for multiplication table...........:" GET n1 PICTURE "@K 99"
   @ 11, 01 SAY "Enter a number to run calculations................:" GET n2 PICTURE "@K 99"
   @ 12, 01 SAY "Enter a grade.....................................:" GET nGrade PICTURE "@K 99"
   @ 13, 01 SAY "Give a number 1–7 to compute day of week..........:" GET nDay PICTURE "@k 9"

   separa( .f. )

   READ

   CLS

   separa( .f. )

   // Test calls to C functions
   ? "The year " + hb_ntoc( nYear )
   IF mi_isLeap( nYear )
      ?? " is leap"
   ELSE
      ?? " is not leap"
   ENDIF

   separa( .f. )

   cNum := hb_ntoc( nCube ) // number as string

   // Does not return a value; uses the same variable (by reference) to store the cube
   mi_cubo( @nCube )

   ? "The cube of " + cNum + " is " + hb_ntoc( nCube )

   separa( .f. )

   cString := AllTrim( cString )
   n := scanChar( cString, cChar )
   IF n > 0
      ? [In the string: "] + cString + ["]
      ? "The first occurrence of character '" + cChar + "' is at position " + HB_NToS( n )
      n := charCount( cString, cChar )
      ? "and it repeats " + HB_NToS( n ) + " times"
   ELSE
      ? "There is no '" + cChar + "' in the string: " + cString
   ENDIF

   separa( .f. )

   calcula( n1, n2, @nS, @nR, @nP, @nD ) // Note: last 4 are by reference with '@'

   // Note: parameters to calcula() are passed by reference
   ? "Sum.......: " + HB_NToS( n1 ) + " + " + HB_NToS( n2 ) + " = " + HB_NToS( nS )
   ? "Subtract..: " + HB_NToS( n1 ) + " - " + HB_NToS( n2 ) + " = " + HB_NToS( nR )
   ? "Product...: " + HB_NToS( n1 ) + " X " + HB_NToS( n2 ) + " = " + HB_NToS( nP )
   ? "Division..: " + HB_NToS( n1 ) + " / " + HB_NToS( n2 ) + " = " + HB_NToS( nD )

   separa( .f. )

   esActo( nGrade, @cStatus )

   ? "You are " + cStatus

   separa( .t. )

   // We’ll add 3 elements to the array to verify that sumaArray() only sums numerics
   AAdd( aNum, "Hello" )
   AAdd( aNum, 25.75 )
   AAdd( aNum, 1000 )

   cambiaValor( aNum )
   FOR i := 1 TO Len( aNum )
      ? "Element", i, aNum[ i ]
   NEXT

   ? "The sum of array elements is:", sumaArray( aNum )

   separa( .f. )

   ? "Multiplication table of " + HB_NToS( n1 )

   aTabla := tabla( n1 )

   FOR i := 1 TO Len( aTabla )
      ? HB_NToS( n1 ) + " X " + HB_NToS( i ) + " = " + HB_NToS( aTabla[ i ] )
   NEXT

   separa( .t. )

   cDay := diaSemana( nDay )

   IF !Empty( cDay )
      ? "The day is: " + cDay
   ELSE
      ? "Input error: only numbers from 1 to 7..."
   ENDIF

   separa( .t. )

RETURN

// Draw a separator on screen
STATIC PROCEDURE separa( lPause )
   ? Replicate( "-", 70 )
   IF ValType( lPause ) == 'L' .AND. lPause
      ? "Press any key to continue..."
      Inkey( 100 )
   ENDIF
RETURN
```

```c
/*
 * Returns whether a year is leap or not
 */
HB_FUNC( MI_ISLEAP )
{
   HB_UINT uiYear = hb_parni( 1 );   // Get integer parameter from PRG
   HB_BOOL lRet = HB_FALSE;

   if( uiYear > 0 )
   {
      lRet = ( ( uiYear % 4 == 0 && uiYear % 100 != 0 ) || uiYear % 400 == 0 );
   }

   hb_retl( lRet );                  // Return a logical value via Harbour’s stack
}

/*
 * Cube of a number passed by reference
 */
HB_FUNC( MI_CUBO )
{
   HB_MAXINT iCubo = hb_parnint( 1 );
   iCubo = iCubo * iCubo * iCubo;
   hb_stornint( iCubo, 1 );
}

/*
 * Counts the occurrences of a character within a string
 */
HB_FUNC( CHARCOUNT )
{
   const char * szCadena = hb_parc( 1 );       // param 1 as string
   HB_SIZE uiLen = hb_parclen( 1 );            // length of param 1
   HB_UINT uiContador = 0;

   if( uiLen > 0 )
   {
      const char * cCaracter = hb_parc( 2 );   // param 2 as string
      HB_SIZE i;

      for( i = 0; i < uiLen; i++ )
      {
         if( szCadena[ i ] == cCaracter[ 0 ] )
         {
            ++uiContador;
         }
      }
   }

   hb_retnint( uiContador );
}

/*
 * Finds the first occurrence of a character within a string
 */
HB_FUNC( SCANCHAR )
{
   const char * szCadena = hb_parc( 1 );
   HB_SIZE uiLen = hb_parclen( 1 );
   HB_SIZE uiPos = 0;

   if( uiLen > 0 )
   {
      const char * cCaracter = hb_parc( 2 );
      HB_SIZE i;

      for( i = 0; i < uiLen; i++ )
      {
         if( szCadena[ i ] == cCaracter[ 0 ] )
         {
            uiPos = i + 1;   // +1 because C arrays are 0-based
            break;
         }
      }
   }

   hb_retnint( uiPos );
}
```

```c
/*
 * Returns whether a year is leap or not
 */
HB_FUNC( MI_ISLEAP )
{
   HB_UINT uiYear = hb_parni( 1 );
   HB_BOOL lRet = HB_FALSE;

   if( uiYear > 0 )
   {
      lRet = ( ( uiYear % 4 == 0 && uiYear % 100 != 0 ) || uiYear % 400 == 0 );
   }

   hb_retl( lRet );
}

/*
 * Cube of a number passed by reference
 */
HB_FUNC( MI_CUBO )
{
   HB_MAXINT iCubo = hb_parnint( 1 );
   iCubo = iCubo * iCubo * iCubo;
   hb_stornint( iCubo, 1 );
}

/*
 * Checks whether a grade is passing and stores a literal with the result
 */
HB_FUNC( ESACTO )
{
   unsigned int uiNota = hb_parnidef( 1, 3 ); // Default to 3 (minimum) if not passed
   if( uiNota > 4 )
      hb_storc( "APPROVED", 2 );
   else
      hb_storc( "FAILED", 2 );
}

/*
 * Day of week
 * Uses hb_xstrcpy() and hb_retc_buffer()
 */
HB_FUNC( DIASEMANA )
{
   unsigned int uiDia = hb_parni( 1 ); // Receives the number

   if( uiDia >= 1 && uiDia <= 7 )
   {
      // Holds the day literal; good practice is to init pointers to NULL
      char * szDia = NULL;

      // hb_xstrcpy allocates memory if first parameter is NULL
      switch( uiDia )
      {
         case 1: szDia = hb_xstrcpy( NULL, "Monday",    NULL ); break;
         case 2: szDia = hb_xstrcpy( NULL, "Tuesday",   NULL ); break;
         case 3: szDia = hb_xstrcpy( NULL, "Wednesday", NULL ); break;
         case 4: szDia = hb_xstrcpy( NULL, "Thursday",  NULL ); break;
         case 5: szDia = hb_xstrcpy( NULL, "Friday",    NULL ); break;
         case 6: szDia = hb_xstrcpy( NULL, "Saturday",  NULL ); break;
         case 7: szDia = hb_xstrcpy( NULL, "Sunday",    NULL ); break;
      }

      hb_retc_buffer( szDia ); // Return and free memory
   }
   else
   {
      hb_ret(); // Return NIL
   }
}

/*
 * Reverses a string
 */
HB_FUNC( REVERSE )
{
   const char * szInString = hb_parc( 1 );
   int iLen = hb_parclen( 1 );

   if( iLen )
   {
      char * szRetStr = hb_xgrab( iLen );
      int i;
      for( i = 0; i < iLen; i++ )
         szRetStr[ i ] = szInString[ iLen - i - 1 ];

      hb_retclen( szRetStr, iLen );
      hb_xfree( szRetStr );
   }
   else
   {
      hb_retc_null();
   }
}
```

## Passing Parameters by Reference

```c
/*
 * By-reference parameters
 * Receives 2 integers and returns sum, difference, product, and division into refs
 */
HB_FUNC( CALCULA )
{
   int i1 = hb_parni( 1 );
   int i2 = hb_parni( 2 );

   // By reference:
   hb_storni( i1 + i2, 3 );
   hb_storni( i1 - i2, 4 );
   hb_storni( i1 * i2, 5 );
   hb_stornd( ( double ) i1 / i2, 6 ); // Take care with the cast
}
```

## Working with Arrays

```c
/*
 * Increases the second element of an array by 100
 */
HB_FUNC( CAMBIAVALOR )
{
   int iVal = hb_parvni( 1, 2 );
   iVal = iVal + 100;
   hb_storvni( iVal, 1, 2 );
}
```

```c
/*
 * Sums numeric elements of a passed array
 */
HB_FUNC( SUMAARRAY )
{
   double uiTotal = 0.0;

   if( hb_extIsArray( 1 ) )
   {
      HB_SIZE i;
      HB_SIZE nLen = hb_parinfa( 1, 0 ); // number of elements

      for( i = 0; i < nLen; i++ )
      {
         switch( hb_parinfa( 1, i + 1 ) )
         {
            case HB_IT_INTEGER:
            case HB_IT_LONG:
               uiTotal += hb_parvnint( 1, i + 1 );
               break;
            case HB_IT_DOUBLE:
               uiTotal += hb_parvnd( 1, i + 1 );
               break;
         }
      }
   }

   hb_retnd( uiTotal );
}
```

```c
/*
 * Multiplication table for the passed integer
 */
HB_FUNC( TABLA )
{
   if( hb_parinfo( 1 ) == HB_IT_INTEGER )
   {
      HB_MAXINT iMul = hb_parnint( 1 );
      HB_UINT n = 10, i;

      hb_reta( n );  // create array in stackReturn

      for( i = 1; i <= n; i++ )
         hb_storvnint( iMul * i, -1, i ); // -1 accesses the return area
   }
   else
   {
      hb_ret();  // NIL
   }
}
```

## Working with C Structures

```harbour
//------------------------------------------------------------------------------
// Exercise: structure via the Extended System
// In PRG: an array with the same elements as the C structure
// ej002.prg
//------------------------------------------------------------------------------
```

```c
typedef struct
{
   char    szNIF[ 10 ];
   char    szNombre[ 20 ];
   HB_UINT uiCodigo;
   float   nSalrio;
   char    szFecha[ 9 ];
   HB_BOOL bSoltero;
} TPersona;

HB_FUNC( DAMEPERSONA )
{
   HB_UINT uiPersona = hb_parni( 1 );
   TPersona * persona = hb_xgrab( sizeof( TPersona ) );

   if( uiPersona == 1 )
   {
      hb_xstrcpy( persona->szNIF,    "53320105T", NULL );
      hb_xstrcpy( persona->szNombre, "Viruete", ", ", "Paco", NULL );
      persona->uiCodigo = 26212;
      persona->nSalrio  = 3500.97f;
      hb_xstrcpy( persona->szFecha,  "19561225", NULL );
      persona->bSoltero = HB_FALSE;
   }
   else
   {
      hb_xstrcpy( persona->szNIF,    "43310009H", NULL );
      hb_xstrcpy( persona->szNombre, "Grande", ", ", "Felix", NULL );
      persona->uiCodigo = 13101;
      persona->nSalrio  = 2750.75f;
      hb_xstrcpy( persona->szFecha,  "19660213", NULL );
      persona->bSoltero = HB_TRUE;
   }

   hb_reta( 6 );
   hb_storvc(  persona->szNIF,   -1, 1 );
   hb_storvc(  persona->szNombre,-1, 2 );
   hb_storvni( persona->uiCodigo,-1, 3 );
   hb_storvnd( persona->nSalrio, -1, 4 );
   hb_storvds( persona->szFecha, -1, 5 );
   hb_storvl(  persona->bSoltero,-1, 6 );
   hb_xfree( persona );
}
```

```harbour
//------------------------------------------------------------------------------
// Exercise: structure via the Extended System — object variant
// ej003.prg
//------------------------------------------------------------------------------

#include "hbclass.ch"

PROCEDURE Main
   LOCAL persona := TPersona():new( 1 )
   CLS
   persona:muestra()
   ?
   ? "Press ENTER..."
   Inkey( 100 )
   CLS
   persona := TPersona():new( 2 )
   persona:muestra()
   ?
   ? "Press ENTER..."
   Inkey( 100 )
   CLS
   TPersona():new( 1 ):muestra()
   ?
   ? "Press ENTER..."
   Inkey( 100 )
RETURN

CREATE CLASS TPersona
   DATA cNIF
   DATA cNombre
   DATA nCodigo
   DATA nSalrio
   DATA dFecha
   DATA lSoltero
   METHOD new( nPersona ) CONSTRUCTOR
   METHOD muestra()
END CLASS

METHOD new( nPersona ) CLASS TPersona
   dameObjPersona( Self, nPersona )
RETURN Self

PROCEDURE muestra() CLASS TPersona
   ? "Object person data:"
   ? ::cNIF
   ? ::cNombre
   ? ::nCodigo
   ? ::nSalrio
   ? ::dFecha
   ? ::lSoltero
RETURN
```

```c
#include "hbapiitm.h"

HB_FUNC( DAMEOBJPERSONA )
{
   PHB_ITEM obj = hb_param( 1, HB_IT_OBJECT );
   HB_UINT uiPersona = hb_parni( 2 );
   TPersona * persona = hb_xgrab( sizeof( TPersona ) );

   if( uiPersona == 1 )
   {
      hb_xstrcpy( persona->szNIF,    "53320105T", NULL );
      hb_xstrcpy( persona->szNombre, "Viruete", ", ", "Paco", NULL );
      persona->uiCodigo = 26212;
      persona->nSalrio  = 3500.97f;
      hb_xstrcpy( persona->szFecha,  "19561225", NULL );
      persona->bSoltero = HB_FALSE;
   }
   else
   {
      hb_xstrcpy( persona->szNIF,    "43310009H", NULL );
      hb_xstrcpy( persona->szNombre, "Grande", ", ", "Felix", NULL );
      persona->uiCodigo = 13101;
      persona->nSalrio  = 2750.75f;
      hb_xstrcpy( persona->szFecha,  "19660213", NULL );
      persona->bSoltero = HB_TRUE;
   }

   hb_itemReturn( obj );
   hb_storvc(  persona->szNIF,   -1, 1 );
   hb_storvc(  persona->szNombre,-1, 2 );
   hb_storvni( persona->uiCodigo,-1, 3 );
   hb_storvnd( persona->nSalrio, -1, 4 );
   hb_storvds( persona->szFecha, -1, 5 );
   hb_storvl(  persona->bSoltero,-1, 6 );
   hb_xfree( persona );
}
```

## Interfacing with C Library Functions (and Your Own C)

```c
char * cstrtran( const char * cString, HB_SIZE nLenStr,
                 const char * cFind,   HB_SIZE nLenFind,
                 const char * cReplace,HB_SIZE nLenRep )
{
   HB_SIZE i, n, w = 0;
   HB_BOOL fFind = HB_FALSE;
   char * cRet = ( char * ) hb_xgrab( nLenStr + 1 );

   for( i = 0; i < nLenStr; i++ )
   {
      for( n = 0; n < nLenFind; n++ )
      {
         fFind = ( cFind[ n ] == cString[ i ] );
         if( fFind )
         {
            if( n < nLenRep )
            {
               cRet[ w ] = cReplace[ n ];
               w++;
            }
            break;
         }
      }
      if( !fFind )
      {
         cRet[ w ] = cString[ i ];
         w++;
      }
   }
   cRet[ w ] = '\\0';
   return cRet;
}
```

```c
HB_FUNC( CSTRTRAN )
{
   hb_retc_buffer( cstrtran( hb_parc( 1 ), hb_parclen( 1 ),
                             hb_parc( 2 ), hb_parclen( 2 ),
                             hb_parc( 3 ), hb_parclen( 3 ) ) );
}
```

```harbour
//------------------------------------------------------------------------------
// Using interfaces to C standard library functions
// ej005.prg
//------------------------------------------------------------------------------

PROCEDURE Main

   LOCAL getlist := {}
   LOCAL cPhrase := Space( 60 )
   LOCAL cSep := Space( 2 )
   LOCAL cToken, n := 0
   LOCAL nNum := 0.0

   SET DECIMAL TO 6

   CLS

   @ 01, 01 SAY "Interfaces..."
   @ 02, 01 SAY "Enter a phrase:"   GET cPhrase PICTURE "@k!"
   @ 03, 01 SAY "Separator.....:"   GET cSep    PICTURE "@k!"
   @ 04, 01 SAY "Enter number..:"   GET nNum    PICTURE "@k"
   READ

   cPhrase := AllTrim( cPhrase )
   cSep    := AllTrim( cSep )

   IF Empty( cSep )
      cSep := " " // default: space
   ENDIF

   CLS
   ? "Phrase....:", cPhrase
   ? "Reversed..:", reverse( cPhrase )
   ?

   /* first token */
   cToken := strtok( cPhrase, cSep )

   /* loop while there are tokens */
   WHILE !Empty( cToken )
      ? ++n, "token ->", cToken
      cToken := strtok( NIL, cSep )
   END

   ?
   ? "Trigonometric functions for", nNum, "in radians:"
   ? "Cosine......:", c_Cos( nNum )
   ? "Sine........:", c_Sin( nNum )
   ? "Tangent.....:", c_Tan( nNum )

   ?
   ? "Press ENTER to continue..."
   Inkey( 100 )

RETURN
```

## Summary of the Extended System

There are basically **three** families that convert Harbour variables to/from primitive C types:

- **Parameter retrieval:** `hb_par…()`  
- **Return value emission:** `hb_ret…()`  
- **By-reference storage:** `hb_stor…()`  

**Skeleton of a C function called from PRG:**

```c
HB_FUNC( NOMBRE_FUNCION )
{
   // Receive parameters
   … = hb_par...();
   … = hb_par...();
   …

   // Process them (now C types)
   …

   // Return a single result
   hb_ret...( value );
}
```

With the **Extended System** you do **not** work directly with Harbour variables (ITEMs); it’s an intermediary layer. In the next chapter we’ll study the **ITEM API** (Clipper’s newer proposal, much improved by Harbour), which makes things more flexible and robust and lets you interact directly with Harbour ITEMs—including arrays, codeblocks, and hashes.
