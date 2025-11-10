---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 17
chapter_title: "Execute Harbour functions, methods, and codeBlocks from C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 17. Execute Harbour functions, methods, and codeBlocks from C

Sometimes it’s necessary in C to execute a Harbour function (those commonly used in PRGs) or one written in PRG by third parties or by ourselves — and even retrieve its return value in order to process it from C. The good news is that this is very easy to do in Harbour.  
To understand what we’re going to cover now, it would be useful to review **Topic 12**, since we’re going to use the **stack**, the **symbol table**, and we’ll invoke the **virtual machine (VM)** ourselves.

This is the list of functions.

## For execution

```
void hb_vmDo( HB_USHORT uiParams );        // Invoke the virtual machine
void hb_vmProc( HB_USHORT uiParams );      // Execute a function or procedure
void hb_vmFunction( HB_USHORT uiParams );  // Execute a function
void hb_vmSend( HB_USHORT uiParams );      // Send a message to an object
```

The four functions above receive as a parameter an unsigned integer that represents the number of parameters that have been pushed onto the stack using the stack‑push functions (listed below).

## For handling codeBlocks

```
PHB_ITEM hb_vmEvalBlock( PHB_ITEM pBlockItem );                  // Evaluate the passed codeBlock without arguments
PHB_ITEM hb_vmEvalBlockV( PHB_ITEM pBlockItem, HB_ULONG ulArgCount, ... ); // Evaluate the passed codeBlock with a variable number of arguments
PHB_ITEM hb_vmEvalBlockOrMacro( PHB_ITEM pItem );                // Evaluate the codeBlock or macro pointed to by the given item
void     hb_vmDestroyBlockOrMacro( PHB_ITEM pItem );             // Destroy the codeBlock or macro in the given item
```

## To push parameters onto the stack

```
void hb_vmPush( PHB_ITEM pItem );                           // Push a generic item
void hb_vmPushNil( void );                                  // Push NIL (nothing)
void hb_vmPushNumber( double dNumber, int iDec );           // Push a number; decides integer/long/double
void hb_vmPushInteger( int iNumber );                       // Push an integer
void hb_vmPushLong( long lNumber );                         // Push a long integer
void hb_vmPushDouble( double dNumber, int iDec );           // Push a real number
void hb_vmPushSize( HB_ISIZ nNumber );                      // Push an HB_SIZE
void hb_vmPushNumInt( HB_MAXINT nNumber );                  // Push a number; decides integer or HB_MAXINT
void hb_vmPushLogical( HB_BOOL bValue );                    // Push a logical value
void hb_vmPushString( const char * szText, HB_SIZE length );// Push a string
void hb_vmPushStringPcode( const char * szText, HB_SIZE length ); // Push a pcode string
void hb_vmPushDate( long lDate );                           // Push a date as a long integer (Julian)
void hb_vmPushTimeStamp( long lJulian, long lMilliSec );    // Push two longs as a TimeStamp
void hb_vmPushSymbol( PHB_SYMB pSym );                      // Push a pointer to a function symbol
void hb_vmPushDynSym( PHB_DYNS pDynSym );                   // Push a pointer to a function/method symbol
void hb_vmPushEvalSym( void );                              // Push the evaluation symbol for codeBlocks
void hb_vmPushPointer( void * pPointer );                   // Push an HB_IT_POINTER
void hb_vmPushPointerGC( void * pPointer );                 // Push a GC‑managed HB_IT_POINTER
void hb_vmPushItemRef( PHB_ITEM pItem );                    // Push an item passed by reference
```

## Examples

### The PRG that calls C functions

```harbour
//------------------------------------------------------------------------------
// Exercise: using C functions. Execute PRG functions from C
// ej012.prg
//------------------------------------------------------------------------------

procedure main

    local n

    cls

    // Shows a string from C in an Alert; receives no parameters
    ejAlertDesdeC00()

    // Shows a string from C in an Alert; receives a string parameter "C"
    ejAlertDesdeC01( "ALERT from C passing this string" )
    // Warning message when checking that a string was not passed
    ejAlertDesdeC01( Date() )

    // Shows a string from C in an Alert; receives a parameter of any
    // type and converts it to "C" string
    ejAlertDesdeC02( "Hello, I'm a little string" ) // String
    ejAlertDesdeC02( Time() ) // String with time format
    ejAlertDesdeC02( Date() ) // Date
    ejAlertDesdeC02( 1200 ) // Integer
    ejAlertDesdeC02( 390.25 ) // Real number
    ejAlertDesdeC02( 1200 + 390.25 ) // Sum of integer and real
    ejAlertDesdeC02( .t. )  // Logical
    ejAlertDesdeC02() // Without parameter; shows message with a warning

    // Execute a function written by ourselves or by third parties
    n := ejSumaEnC()
    ? "Value returned by ejSumaEnC", n
    n := ejSumaEnCPar( 10, 34.79)
    ? "Value returned by ejSumaEnCPar -> 10 + 34.79 =", n
    n := ejSumaEnCParPro( 10, 34.79)
    ? "Value returned by ejSumaEnCParPro -> ( 10 + 34.79 ) * 100 =", n
    ? "------------------------------------------------------------------------"
    Inkey( 100 )

return

//------------------------------------------------------------------------------
// Add two numbers
//------------------------------------------------------------------------------

function suma( n1, n2 )

    if ValType( n1 ) != 'N'
        n1 := 0
    endif
    if ValType( n2 ) != 'N'
        n2 := 0
    endif

return n1 + n2

//------------------------------------------------------------------------------
```

### And now the C functions

```c
/*
 * Execute PRG functions from C
 */

#include "hbvm.h"    // Required to invoke the virtual machine and use its functions
#include "hbstack.h" // For stack management

/*
 * Call the ALERT function from C.
 * We also assign a string from C
 */
HB_FUNC( EJALERTDESDEC00 )
{
    // Look up the ALERT symbol in the dynamic symbol table.
    // The hb_dynsymFind() function searches for the symbol by name taking
    // uppercase/lowercase into account; if it doesn't find it, it returns NULL,
    // which should be handled. In this case I don’t handle it because it’s a
    // Harbour function that will always exist.
    PHB_DYNS pExecSym = hb_dynsymFind( "ALERT" );

    hb_vmPushDynSym( pExecSym ); // Push the function symbol onto the stack
    // IMPORTANT: The first position on the stack after the function symbol
    // will always be NIL for functions and procedures, or SELF for class methods.
    hb_vmPushNil(); // Push NIL
    // Push a string as a C string. The string length must be passed.
    hb_vmPushString( "Hello, this is an ALERT from C", 30 );
    // This function invokes the Harbour Virtual Machine (VM) to execute
    // the function. You must indicate the number of parameters.
    hb_vmDo( 1 );
}

/*
 * Call the ALERT function from C.
 * We pass a string from PRG and check that it is a string
 */
HB_FUNC( EJALERTDESDEC01 )
{
    PHB_DYNS pExecSym = hb_dynsymFind( "ALERT" );
    PHB_ITEM cItem = hb_param( 1, HB_IT_STRING );

    hb_vmPushDynSym( pExecSym );
    hb_vmPushNil();

    // Ensure the type expected in hb_param() — i.e., a string (HB_IT_STRING)
    if( cItem )
    {
        hb_vmPush( cItem ); // Push the parameter as an ITEM
    }
    else // If an Item of type "C" (string) was not passed
    {
        // Push a string as a C string. The length must be passed.
        hb_vmPushString( "ATTENTION;No string was passed", 35 );
    }

    // Invoke the VM to execute what is on the stack.
    // Either function may be used:
    //hb_vmDo( 1 );
    hb_vmProc( 1 );
}

/*
 * Call the ALERT function from C.
 * We pass any value and type from PRG and check that something was passed
 */
HB_FUNC( EJALERTDESDEC02 )
{
    PHB_DYNS pExecSym = hb_dynsymFind( "ALERT" );
    PHB_ITEM xItem = hb_param( 1, HB_IT_ANY );

    hb_vmPushDynSym( pExecSym );
    hb_vmPushNil();

    if( xItem )
    {
        // The hb_itemValToStr() function creates a new Item of character type
        // from an Item of any type passed to it; therefore, it must be released
        // once used.
        PHB_ITEM cItem = hb_itemValToStr( xItem );
        
        hb_vmPush( cItem );
        hb_itemRelease( cItem );
    }
    else
    {
        hb_vmPushString( "ATTENTION;No value was passed", 37 );
    }
    // Either function may be used:
    //hb_vmDo( 1 );
    hb_vmProc( 1 );
}

/*
 * Call the PRG function SUMA from C.
 * Numbers are provided from C
 */
HB_FUNC( EJSUMAENC )
{
    // hb_dynsymFind() is case sensitive; Harbour uppercases all symbols created
    // from PRG, but it respects case. If the name we pass as a parameter might be
    // in mixed case, hb_dynsymFindName() can be used as it uppercases before searching.
    PHB_DYNS pExecSym = hb_dynsymFind( "SUMA" );
    
    // When there is no certainty that the symbol exists, check it.
    if( pExecSym )
    {
        hb_vmPushDynSym( pExecSym );
        hb_vmPushNil();
        hb_vmPushInteger( 1500 );
        hb_vmPushDouble( 250.35, 2 );        
        // Either function may be used:
        //hb_vmDo( 2 ); 
        hb_vmProc( 2 );
    }
}

/*
 * Call the PRG function SUMA from C.
 * Numbers are passed from PRG
 */
HB_FUNC( EJSUMAENCPAR )
{
    PHB_DYNS pExecSym = hb_dynsymFind( "SUMA" );
    
    // When there is no certainty that the symbol exists, check it.
    if( pExecSym )
    {
        // The two parameters
        PHB_ITEM n1 = hb_param( 1, HB_IT_NUMERIC );
        PHB_ITEM n2 = hb_param( 2, HB_IT_NUMERIC );
        
        hb_vmPushDynSym( pExecSym );
        hb_vmPushNil();
        hb_vmPush( n1 );
        hb_vmPush( n2 );        
        //hb_vmDo( 2 ); // Either function may be used
        hb_vmProc( 2 );
    }
}

/*
 * Call the PRG function SUMA from C and process the result in C.
 * In this case we multiply by 100 and return the result
 * The two numbers are passed from PRG
 */
HB_FUNC( EJSUMAENCPARPRO )
{
    PHB_DYNS pExecSym = hb_dynsymFind( "SUMA" );
    
    // When there is no certainty that the symbol exists, check it.
    if( pExecSym )
    {
        double dNum, dRes;
        // The two parameters
        PHB_ITEM n1 = hb_param( 1, HB_IT_NUMERIC );
        PHB_ITEM n2 = hb_param( 2, HB_IT_NUMERIC );
        
        hb_vmPushDynSym( pExecSym );
        hb_vmPushNil();
        hb_vmPush( n1 );
        hb_vmPush( n2 );        
        //hb_vmDo( 2 ); 
        hb_vmProc( 2 ); // The result is placed on the stack in the return item
        
        // The return item is retrieved with hb_stackReturnItem()
        // Extract the number as C type double to process it
        dNum = hb_itemGetND( hb_stackReturnItem() ); // Get the number as C type
        // Do the processing; in this case, multiply by 100
        dRes = dNum * 100;
        // Place the result onto the stack in the return item
        hb_itemPutND( hb_stackReturnItem(), dRes ); 
    }
}
```

Up to now, we’ve only used `void hb_vmDo( HB_USHORT uiParams );` to invoke the VM, but in fact this function can be used for functions, procedures, and method sends. The rest are slightly more optimized for the specific task indicated by their names (as explained at the beginning of the topic).

```
void hb_vmProc( HB_USHORT uiParams );     // Should be used for procedures and functions alike
void hb_vmFunction( HB_USHORT uiParams ); // Guarantees the return stack is clean for the function's return value
void hb_vmSend( HB_USHORT uiParams );     // Preferred for sending messages to objects (methods); more optimized than hb_vmDo() for this
```

That said, I recommend using **hb_vmProc()** for both procedures and functions, and **hb_vmSend()** only for methods.

## Example of sending messages with hb_vmSend()

### The PRG with the example and the class

```harbour
//------------------------------------------------------------------------------
// Exercise: using methods in C. Execute PRG methods from C
// ej013.prg
//------------------------------------------------------------------------------

#include "HBClass.ch"

PROCEDURE main

   LOCAL oModel := TMiModelo():new( "Manu", "Exposito", 57, 1200 )

   oModel:verDatosModelo()

   cambiaDatosEnC( oModel )
   oModel:verDatosModelo()

   cambiaDatosEnCPar( oModel, "Gerogina", "Gamero", 33, 4523.65 )
   oModel:verDatosModelo()

RETURN

//------------------------------------------------------------------------------

CREATE CLASS TMiModelo

   HIDDEN:
   DATA cNombre
   DATA cApellido
   DATA nEdad
   DATA nSueldo

   EXPORTED:
   CONSTRUCTOR new( cNombre, cApellido, nEdad, nSueldo )
   METHOD cambiaDatos( cNombre, cApellido, nEdad, nSueldo )
   METHOD isMayorEdad()
   METHOD guardaModelo()
   METHOD leeModelo()
   METHOD verDatosModelo()

   // SET / GET methods
   METHOD getNombre()
   METHOD setNombre( cNombre )
   METHOD getApellido()
   METHOD setApellido( cApellido )
   METHOD getEdad()
   METHOD setEdad( nEdad )
   METHOD getSueldo()
   METHOD setSueldo( nSueldo )

END CLASS

//------------------------------------------------------------------------------

METHOD new( cNombre, cApellido, nEdad, nSueldo ) CLASS TMiModelo

   ::cambiaDatos( cNombre, cApellido, nEdad, nSueldo )

RETURN self

//------------------------------------------------------------------------------

METHOD cambiaDatos( cNombre, cApellido, nEdad, nSueldo ) CLASS TMiModelo

   IF ValType( cNombre ) == 'C'
      ::cNombre := cNombre
   ENDIF

   IF ValType( cApellido ) == 'C'
      ::cApellido := cApellido
   ENDIF

   IF ValType( nEdad ) == 'N'
      ::nEdad := nEdad
   ENDIF

   IF ValType( nSueldo ) == 'N'
      ::nSueldo := nSueldo
   ENDIF

return self

//------------------------------------------------------------------------------

METHOD isMayorEdad() CLASS TMiModelo
RETURN ::nEdad >= 18

//------------------------------------------------------------------------------

METHOD guardaModelo() CLASS TMiModelo

   LOCAL lRet := .F.

   Alert( "Here the model would be persisted" )

RETURN lRet

//------------------------------------------------------------------------------

METHOD leeModelo() CLASS TMiModelo

   LOCAL lRet := .F.

   Alert( "Here the model would be loaded from the DataSet" )

RETURN lRet

//------------------------------------------------------------------------------

METHOD verDatosModelo() CLASS TMiModelo

   Alert( "INFORMATION;-----------;" + ";" + ;
          "Name.....: " + ::getNombre() + ";" + ;
          "Surname..: " + ::getApellido() + ";" + ;
          "Age......: " + hb_ntos( ::getEdad() ) + ";" + ;
          "Salary...: " + hb_ntos( ::getSueldo() ) )

RETURN self

//------------------------------------------------------------------------------
// SET / GET Methods

METHOD getNombre() CLASS TMiModelo
RETURN ::cNombre

//------------------------------------------------------------------------------

METHOD setNombre( cNombre ) CLASS TMiModelo

   IF ValType( cNombre ) == 'C' .AND. !Empty( cNombre )
      ::cNombre := cNombre
   ENDIF

RETURN self

//------------------------------------------------------------------------------

METHOD getApellido() CLASS TMiModelo
RETURN ::cApellido

//------------------------------------------------------------------------------

METHOD setApellido( cApellido ) CLASS TMiModelo

   IF ValType( cApellido ) == 'C' .AND. !Empty( cApellido )
      ::cApellido := cApellido
   ENDIF

RETURN self

//------------------------------------------------------------------------------

METHOD getEdad() CLASS TMiModelo
RETURN ::nEdad

//------------------------------------------------------------------------------

METHOD setEdad( nEdad ) CLASS TMiModelo

   IF ValType( nEdad ) == 'N' .AND. edad > 0
      ::nEdad := nEdad
   ENDIF

RETURN self

//------------------------------------------------------------------------------

METHOD getSueldo() CLASS TMiModelo
RETURN ::nSueldo

//------------------------------------------------------------------------------

METHOD setSueldo( nSueldo ) CLASS TMiModelo

   IF ValType( nSueldo ) == 'N' .AND. nSueldo > 0
      ::nSueldo := nSueldo
   ENDIF

RETURN self

//------------------------------------------------------------------------------
```

### And now the C part

```c
/*
 * Example of executing methods from C
 * Takes the data directly from C with C‑type variables
 */
 
HB_FUNC( CAMBIADATOSENC )
{
    PHB_ITEM pObj = hb_param( 1, HB_IT_OBJECT );
    
    if( pObj )
    {                   
        PHB_DYNS pMsgSym = hb_dynsymFind( "CAMBIADATOS" );        
        
        hb_vmPushDynSym( pMsgSym ); // Push the method onto the stack
        hb_vmPush( pObj ); // This is super important: push the object first
        hb_vmPushString( "Isabel", 6 );
        hb_vmPushString( "Guerrero", 8 );
        hb_vmPushInteger( 52 );
        hb_vmPushDouble( 2500.75, 2 );
        hb_vmSend( 4 );
    }
}

/*
 * Example of executing methods from C
 * Takes the data passed from PRG as Items
 */
 
HB_FUNC( CAMBIADATOSENCPAR )
{
    PHB_ITEM pObj = hb_param( 1, HB_IT_OBJECT );
    
    if( pObj )
    {                
        PHB_ITEM cNombre  = hb_param( 2, HB_IT_STRING );
        PHB_ITEM cApellido= hb_param( 3, HB_IT_STRING );
        PHB_ITEM nEdad    = hb_param( 4, HB_IT_NUMINT );
        PHB_ITEM nSueldo  = hb_param( 5, HB_IT_DOUBLE );
        PHB_DYNS pMsgSym  = hb_dynsymFind( "CAMBIADATOS" );        
        
        hb_vmPushDynSym( pMsgSym ); // Push the method onto the stack
        hb_vmPush( pObj );          // This is super important: push the object first
        hb_vmPush( cNombre );
        hb_vmPush( cApellido );
        hb_vmPush( nEdad );
        hb_vmPush( nSueldo );
        hb_vmSend( 4 );
    }
}
```

## Now, codeBlocks…

Just like with executing functions, procedures, and methods, evaluating **codeBlocks** from C is very easy and follows the same approach already described — i.e., use of the **stack** and invoking the **virtual machine** to process them.

Clipper’s initial proposal was to use the following functions:

```
HB_BOOL  hb_evalNew( PHB_EVALINFO pEvalInfo, PHB_ITEM pItem ); // Prepare EVALINFO with the passed codeBlock, ready to receive parameters
HB_BOOL  hb_evalPutParam( PHB_EVALINFO pEvalInfo, PHB_ITEM pItem ); // Add a parameter to the structure (call once per parameter)
PHB_ITEM hb_evalLaunch( PHB_EVALINFO pEvalInfo ); // Evaluate the codeBlock with the parameters stored in EVALINFO
HB_BOOL  hb_evalRelease( PHB_EVALINFO pEvalInfo ); // Free memory occupied by copies of items used as parameters and re‑init the structure
// NOTE: If the items passed as parameters were the originals obtained with hb_param()
// (and not copies via hb_itemParam()), this function will cause an undefined error
// when attempting to free those original items.
```

### A couple of examples

```c
/*
 * Evaluate a codeBlock passed from C.
 * Clipper’s legacy way.
 * Without parameters
 */
HB_FUNC( EVALUA00 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock
    
    if( pCB ) // If a CB was passed
    {
        HB_EVALINFO EvalInfo; // Structure HB_EVALINFO 
        
        // Initialize HB_EVALINFO (pass its address with &)
        if( hb_evalNew( &EvalInfo, pCB ) )
        {
            hb_evalLaunch( &EvalInfo ); // Evaluate the CB
        }
        
        hb_evalRelease( &EvalInfo ); // Free the structure’s internal memory
    }
}

/*
 * Evaluate a codeBlock passed from C.
 * Clipper’s legacy way.
 * With parameters
 */
HB_FUNC( EVALUA01 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock
    
    if( pCB ) // If a CB was passed
    {        
        HB_EVALINFO EvalInfo; // Structure HB_EVALINFO
        
        // Initialize HB_EVALINFO
        if( hb_evalNew( &EvalInfo, pCB ) )
        {
            // NOTE: Use hb_itemParam(), which creates a copy of the passed parameter.
            // Do not pass the original parameter, because hb_evalRelease frees those copies.
            hb_evalPutParam( &EvalInfo, hb_itemParam( 2 ) ); // Add first parameter
            hb_evalPutParam( &EvalInfo, hb_itemParam( 3 ) ); // Add second parameter
            
            hb_evalLaunch( &EvalInfo ); // Evaluate the CB with the passed parameters
        }
        
        hb_evalRelease( &EvalInfo ); // Free the structure’s internal memory
    }
}
```

### Undocumented direct codeBlock evaluation helpers

```
void hb_evalBlock0( PHB_ITEM pCodeBlock );                         // Evaluate a codeBlock without parameters
void hb_evalBlock1( PHB_ITEM pCodeBlock, PHB_ITEM pParam );         // Evaluate a codeBlock with one parameter
void hb_evalBlock( PHB_ITEM pCodeBlock, ... );                      // Evaluate a codeBlock with an arbitrary number of parameters
// NOTE: The last parameter must be NULL.
```

**Examples:**

```c
/*
 * Evaluate a codeBlock passed from C.
 * Helper to evaluate a CB without parameters directly.
 */
HB_FUNC( EVALUA04 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock

    if( pCB )
    {
        hb_evalBlock0( pCB );    
    }
}

/*
 * Evaluate a codeBlock passed from C.
 * Helper to evaluate a CB with one parameter directly.
 */
HB_FUNC( EVALUA05 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock

    if( pCB )
    {
        hb_evalBlock1( pCB, hb_param( 2, HB_IT_ANY ) );    
    }
}

/*
 * Evaluate a codeBlock passed from C.
 * Helper to evaluate a CB with parameters directly.
 */
HB_FUNC( EVALUA06 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock

    if( pCB )
    {
        hb_evalBlock( pCB, hb_param( 2, HB_IT_ANY ), hb_param( 3, HB_IT_ANY ), NULL );    
    }
}
```

As a curiosity, it’s worth noting that Harbour internally treats codeBlocks as **objects**. The object would be the variable name, and the evaluation method would be `bCB:eval( p0, p1, …, pn )` with the parameters passed. This is precisely Harbour’s newer approach to internal evaluation of codeBlocks — it’s done *as if it were a method call*.

Let’s see it in the examples:

```c
/*
 * Evaluate a codeBlock passed from C.
 * Harbour’s newer proposal.
 * Works exactly like object methods.
 * This is the same example as the EVALUA00 function above.
 */
HB_FUNC( EVALUA02 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock

    if( pCB )
    {
        hb_vmPushEvalSym(); // Push the eval() symbol onto the stack
        hb_vmPush( pCB );   // Push the codeBlock onto the stack like an object
        hb_vmSend( 0 );     // Execute with no parameters, as if it were an object
    }
}

/*
 * Evaluate a codeBlock passed from C.
 * Harbour’s newer proposal.
 * Works exactly like object methods.
 * This is the same example as the EVALUA01 function above.
 */
HB_FUNC( EVALUA03 )
{
    PHB_ITEM pCB = hb_param( 1, HB_IT_BLOCK ); // The passed codeBlock

    if( pCB )
    {
        hb_vmPushEvalSym(); // Push the eval() symbol onto the stack
        hb_vmPush( pCB );   // Push the codeBlock onto the stack like an object
        hb_vmPush( hb_param( 2, HB_IT_ANY ) );
        hb_vmPush( hb_param( 3, HB_IT_ANY ) );
        hb_vmSend( 2 );     // Execute with two parameters, as if it were an object
    }
}
```

### Using the functions above from PRG

```harbour
//------------------------------------------------------------------------------
// Exercise: using codeBlocks in C. Evaluate a codeBlock without parameters from C,
// in Clipper’s old style and Harbour’s newer style.
// ej014.prg
//------------------------------------------------------------------------------

#include "HBClass.ch"

//------------------------------------------------------------------------------

PROCEDURE main

    local bCB0 := { || Alert( "Hello world" ) }
    local bCB1 := { | p1, p2 | Alert( "The parameters passed: " + HB_ValToStr( p1 ) + ;
                          " and " + HB_ValToStr( p2 ) ) }
    local bCB2 := { | p1 | Alert( "Direct with one parameter: " + HB_ValToStr( p1 ) ) }

    evalua00( bCB0 )
    evalua01( bCB1, "First parameter", Date() )
    // NOTE: In Harbour, codeBlocks can be treated as objects.
    // The object is the codeBlock variable and with the ::eval() method
    // you pass the parameters and evaluate it.
    bCB1:eval( "Treat codeBlock as object", Time() )
    // Using Harbour’s newer approach
    evalua02( bCB0 )
    evalua03( bCB1, "Parameter 1", Seconds() )

    Alert( "Direct functions..." )
    evalua04( bCB0 )
    evalua05( bCB2, "With parameter directly" )
    evalua06( bCB1, "With one parameter...", "another parameter" )

RETURN
```
