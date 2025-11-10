---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 18
chapter_title: "The Error API — Handling Harbour Errors from C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 18. The Error API — Handling Harbour Errors from C

One of the most important responsibilities in a programming language is handling exceptions, warnings, and errors. These can be raised directly by the program at runtime or intentionally triggered by the programmer to control exceptional conditions that the language itself wouldn’t detect. For example, inserting a database record with a duplicate unique key is not a language-level error, but we *do* want to catch that database exception and handle it as a normal error.

Harbour provides a system that is both simple and powerful. The `TError` class is responsible for this work. This class has no methods; it only exposes instance variables (datas) that represent the state of the error object.

Usually, the runtime itself creates the error object; at most, you might have a custom error handler to control screen output or write to a log file. But sometimes you need to create your own error object and manage it manually. In PRG there is a function that does exactly that: `ErrorNew()`. Once the object is created, you can access its DATAs and assign values directly:

```harbour
local oError := ErrorNew() // Create the object

// Assign new values to the DATAs of the freshly created object:
oError:severity    := ES_ERROR
oError:genCode     := EG_UNSUPPORTED
oError:subSystem   := "MI_LIB"
oError:subCode     := 0
oError:description := "This is an error I’m handling myself..."
oError:canRetry    := .F.
oError:canDefault  := .F.
oError:fileName    := ""
oError:osCode      := 0

// Raise the error so it’s handled by the default (or your custom) error handler
Eval( ErrorBlock(), oError )
```

We can do the very same thing from C using the **Error API**. For that we have a set of functions to create the object, set/get the values of its datas, launch it once prepared, and finally free it. Below are the datas (instance variables) that you can access via C functions or directly from PRG:

- **Args**: Contains an array of arguments supplied to an operator or function when an argument error occurs. For other error types, `Args` contains `NIL`.
- **CanDefault**: Logical value indicating whether the subsystem can perform a default error recovery for the condition. `.T.` means a default recovery is available. Availability and actual strategy depend on the subsystem and the error condition. The minimal action is to simply ignore the error. Default recovery is requested by returning `.F.` from the error block invoked to handle the error. Note: `canDefault` is never `.T.` if `canSubstitute` is `.T.`
- **CanRetry**: Logical value indicating whether the subsystem can retry the operation that caused the error. `.T.` means retry is possible. Whether it’s actually possible depends on the subsystem and the condition. A retry is requested by returning `.T.` from the error block. `canRetry` is never `.T.` if `canSubstitute` is `.T.`
- **CanSubstitute**: Logical value indicating whether a new result can be substituted for the failed operation. Argument errors and some other simple errors allow the error handler to substitute a new result value. `.T.` means substitution is possible. Substitution is done by returning the new result value from the error block. `canSubstitute` is never `.T.` if `canDefault` or `canRetry` is `.T.`
- **Cargo**: Contains a value of any data type not used by the error system. This is a user‑definable slot allowing you to attach arbitrary information to an error object and retrieve it later.
- **Description**: String describing the error condition. An empty string indicates that the subsystem did not provide a printable description. If `genCode` is non‑zero, a printable description is always available.
- **FileName**: String representing the original name used to open the file associated with the error condition. An empty string indicates the error is not associated with a specific file, or the subsystem doesn’t retain the filename.
- **GenCode**: Integer value representing a generic Harbour error code. Generic codes allow default handling of similar errors across subsystems. A value of `0` indicates the condition is subsystem‑specific and does not correspond to any generic code. The header `error.ch` provides manifest constants for generic error codes.
- **Operation**: String describing the operation being attempted when the error occurred. For operators and functions, it contains the operator or function name. For undefined variables or functions, it contains their name. An empty string indicates the subsystem provided no printable description of the operation.
- **OsCode**: Integer value representing the OS error code associated with the condition. `0` indicates the condition was not caused by an OS error. When `osCode` is non‑zero, `DosError()` is set to the same value. `osCode` correctly reflects the DOS extended error code for file errors, allowing proper distinction between sharing violations (e.g. opening `EXCLUSIVE` while another process has the file open) and access violations (e.g. opening read/write when the file is read‑only). See the appendices/error messages guide for a list of DOS error codes.
- **Severity**: Integer indicating severity. Four standard values are defined in `error.ch`:
  - `ES_WHOCARES`: Informational; not a failure.
  - `ES_WARNING`: Does not prevent further operations but may lead to more serious errors later.
  - `ES_ERROR`: Prevents further operations without corrective action.
  - `ES_CATASTROPHIC`: Requires immediate termination of the application.

  Note: Harbour runtime support only generates errors with severities `ES_WARNING` or `ES_ERROR`.
- **SubCode**: Integer representing a subsystem‑specific error code. `0` indicates the subsystem does not assign a specific number to the condition.
- **Subsystem**: String with the name of the subsystem generating the error. For core Harbour operators/functions, the subsystem name is `BASE`. For database driver errors, `subsystem` contains the driver name.
- **Tries**: Integer representing how many times the failed operation has been attempted. When `canRetry` is `.T.`, `tries` can be used to limit the number of retries. `0` indicates the subsystem does not track the count.

## GET functions (read the value of a DATA)

```c
PHB_ITEM   hb_errGetCargo( PHB_ITEM pError );        // Cargo as ITEM (can be any Harbour type)
PHB_ITEM   hb_errGetArgs( PHB_ITEM pError );         // Args as ITEM array
const char *hb_errGetDescription( PHB_ITEM pError );
const char *hb_errGetFileName( PHB_ITEM pError );
HB_USHORT  hb_errGetFlags( PHB_ITEM pError );
HB_ERRCODE hb_errGetGenCode( PHB_ITEM pError );
const char *hb_errGetOperation( PHB_ITEM pError );
HB_ERRCODE hb_errGetOsCode( PHB_ITEM pError );
HB_USHORT  hb_errGetSeverity( PHB_ITEM pError );
HB_ERRCODE hb_errGetSubCode( PHB_ITEM pError );
const char *hb_errGetSubSystem( PHB_ITEM pError );
HB_USHORT  hb_errGetTries( PHB_ITEM pError );
```

All these functions take the error object as their only parameter and return C types (except the first two, which return Harbour `ITEM`s).

## SET functions (assign the value of a DATA)

```c
PHB_ITEM hb_errPutCargo( PHB_ITEM pError, PHB_ITEM pCargo );
PHB_ITEM hb_errPutArgsArray( PHB_ITEM pError, PHB_ITEM pArgs );
PHB_ITEM hb_errPutArgs( PHB_ITEM pError, HB_ULONG ulArgCount, ... );
PHB_ITEM hb_errPutDescription( PHB_ITEM pError, const char * szDescription );
PHB_ITEM hb_errPutFileName( PHB_ITEM pError, const char * szFileName );
PHB_ITEM hb_errPutFlags( PHB_ITEM pError, HB_USHORT uiFlags );
PHB_ITEM hb_errPutGenCode( PHB_ITEM pError, HB_ERRCODE uiGenCode );
PHB_ITEM hb_errPutOperation( PHB_ITEM pError, const char * szOperation );
PHB_ITEM hb_errPutOsCode( PHB_ITEM pError, HB_ERRCODE uiOsCode );
PHB_ITEM hb_errPutSeverity( PHB_ITEM pError, HB_USHORT uiSeverity );
PHB_ITEM hb_errPutSubCode( PHB_ITEM pError, HB_ERRCODE uiSubCode );
PHB_ITEM hb_errPutSubSystem( PHB_ITEM pError, const char * szSubSystem );
PHB_ITEM hb_errPutTries( PHB_ITEM pError, HB_USHORT uiTries );
PHB_ITEM hb_errNew( void );                             // Create an empty error object
HB_USHORT hb_errLaunch( PHB_ITEM pError );              // Launch the error (like: Eval( ErrorBlock(), oError ))
void      hb_errRelease( PHB_ITEM pError );             // Free datas and the error object itself
```

`hb_errLaunch()` can return `E_BREAK` (interrupts processing with no further retries), `E_RETRY` (the operation can be attempted again), or `E_DEFAULT` (initial/default value).


## Practical example

```harbour
//------------------------------------------------------------------------------
// Exercise: using error handling from C.
// ej015.prg
//------------------------------------------------------------------------------

#include "error.ch"

PROCEDURE main

   cls

   ? "Generate a system error from C:"
   miThowError( ES_WARNING, 23, "This is system warning 23" )
   miThowError( ES_ERROR, 55, "This is system error 55" )
   miThowError( ES_CATASTROPHIC, 63, "This is catastrophic error 63" )

RETURN
```

Another PRG where error messages are stored in an array; the error number is the index where the description is stored:

```harbour
//------------------------------------------------------------------------------
// Exercise: using error handling from C.
// ej016.prg
//------------------------------------------------------------------------------

#include "error.ch"

PROCEDURE main

   cls

   ? "Generate a system error from C:"
   autoErr( ES_WARNING, 2 )
   autoErr( ES_ERROR, 1 )
   autoErr( ES_WHOCARES, 3 )

RETURN

//------------------------------------------------------------------------------

procedure autoErr( nNivel, nCodErr )

   local aErr := { "This is error 1", ;
                   "This is error 2", ;
                   "This is error 3" }

   if nCodErr >= 1 .and. nCodErr <= 3
      miThowError( nNivel, nCodErr, aErr[ nCodErr ] )
   endif

return
```

And here is the C function (verbatim):

```c
#include "hbapierr.h"

HB_FUNC( MITHOWERROR ) // ( HB_USHORT uiLevel, HDO_ERRCODE errCode, const char *szDesc )
{
   PHB_ITEM pError = hb_errNew();
   HB_USHORT uiLevel = hb_parnidef( 1, ES_WARNING );
   HB_ERRCODE errCode = hb_parnidef( 2, 0 );
   const char *szMsg = hb_parc( 3 );

   hb_errPutSubSystem( pError, "Curso de C (Subsistema)" ); // Subsystem name
   hb_errPutSubCode( pError, errCode );                     // Our system error code
   hb_errPutDescription( pError, szMsg );                   // Message
   hb_errPutSeverity( pError, uiLevel );                    // Error severity

   hb_errPutFlags( pError, EF_CANDEFAULT );                 // Enable default button
   hb_errPutTries( pError, 5 );

   hb_errLaunch( pError );                                  // Launch the prepared error
   hb_errRelease( pError );                                 // Free the object
}
```
