---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 19
chapter_title: "The FileSys API: Working with Files from C"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 19. The FileSys API: Working with Files from C

In this chapter we cover another very important Harbour API: file handling. As usual, implementing our processes in C can speed up creation, deletion, and general file operations.

Most C functions in the FileSys API have a PRG counterpart, which makes their usage easy to intuit.

## Main functions

- `HB_BOOL hb_fsChDir( const char *pszDirName );`  
  Change to the specified directory.

- `void hb_fsClose( HB_FHANDLE hFileHandle );`  
  Close the file.

- `void hb_fsCommit( HB_FHANDLE hFileHandle );`  
  Flush pending file changes to disk.

- `HB_FHANDLE hb_fsCreate( const char *pszFileName, HB_FATTR ulAttr );`  
  Create and open a file.

- `HB_FHANDLE hb_fsCreateEx( const char *pszFileName, HB_FATTR ulAttr, HB_USHORT uiFlags );`  
  Create and open a file with a specific open mode.

- `const char *hb_fsCurDir ( int iDrive );`  
  Return the current directory for the specified drive.

- `int hb_fsCurDrv( void );`  
  Get the current drive number.

- `HB_BOOL hb_fsDelete( const char *pszFileName );`  
  Delete a file.

- `HB_BOOL hb_fsEof( HB_FHANDLE hFileHandle );`  
  Check whether end‑of‑file has been reached.

- `HB_ERRCODE hb_fsError( void );`  
  Retrieve the last filesystem error.

- `HB_BOOL hb_fsFile( const char *pszFileName );`  
  Determine whether the file exists.

- `HB_BOOL hb_fsIsDirectory( const char *pszFileName );`  
  Determine whether the given name is a directory.

- `HB_FOFFSET hb_fsFSize( const char *pszFileName, HB_BOOL bUseDirEntry );`  
  Get a file’s size.

- `HB_FHANDLE hb_fsExtOpen( const char *pszFileName, const char *pDefExt, HB_FATTR nFlags, const char *pPaths, PHB_ITEM pError );`  
  Open a file with a default extension, searching a list of paths.

- `HB_ERRCODE hb_fsIsDrv( int iDrive );`  
  Check whether a drive number is valid.

- `HB_BOOL hb_fsMkDir( const char *pszDirName );`  
  Create a directory.

- `HB_FHANDLE hb_fsOpen( const char *pszFileName, HB_USHORT uiFlags );`  
  Open or create a file.

- `HB_FHANDLE hb_fsOpenEx( const char *pszFileName, HB_USHORT uiFlags, HB_FATTR nAttr );`  
  Open or create a file with the given attributes.

- `HB_USHORT hb_fsRead( HB_FHANDLE hFileHandle, void *pBuff, HB_USHORT uiCount );`  
  Read up to 64 KiB from the file at the current position into a buffer.

- `HB_SIZE hb_fsReadLarge( HB_FHANDLE hFileHandle, void *pBuff, HB_SIZE nCount );`  
  Read more than 64 KiB from the file at the current position into a buffer.

- `HB_SIZE hb_fsReadAt( HB_FHANDLE hFileHandle, void *pBuff, HB_SIZE nCount, HB_FOFFSET nOffset );`  
  Read from a given file position into a buffer (can exceed 64 KiB).

- `HB_BOOL hb_fsRmDir( const char *pszDirName );`  
  Remove a directory.

- `HB_BOOL hb_fsRename( const char * pszOldName, const char *pszNewName );`  
  Rename a file.

- `HB_ULONG hb_fsSeek( HB_FHANDLE hFileHandle, HB_LONG lOffset, HB_USHORT uiMode );`  
  Move the file R/W pointer to a given position.

- `HB_FOFFSET hb_fsSeekLarge( HB_FHANDLE hFileHandle, HB_FOFFSET nOffset, HB_USHORT uiFlags );`  
  Move the file R/W pointer using the 64‑bit API.

- `HB_FOFFSET hb_fsTell( HB_FHANDLE hFileHandle );`  
  Return the current file position.

- `HB_FOFFSET hb_fsGetSize( HB_FHANDLE hFileHandle );`  
  Return the file size (warning: may change the current file position).

- `int hb_fsSetDevMode( HB_FHANDLE hFileHandle, int iDevMode );`  
  Set how a file is treated: text or binary.

- `HB_USHORT hb_fsWrite( HB_FHANDLE hFileHandle, const void *pBuff, HB_USHORT uiCount );`  
  Write up to 64 KiB from a buffer to an open file.

- `HB_SIZE hb_fsWriteLarge( HB_FHANDLE hFileHandle, const void *pBuff, HB_SIZE nCount );`  
  Write more than 64 KiB from a buffer to an open file.

- `HB_SIZE hb_fsWriteAt( HB_FHANDLE hFileHandle, const void *pBuff, HB_SIZE nCount, HB_FOFFSET nOffset );`  
  Write from a buffer to an open file starting at a given offset (can exceed 64 KiB).

- `HB_BOOL hb_fsNameExists( const char *pszFileName );`  
  Check whether a name exists in the system (no wildcards).

- `HB_BOOL hb_fsFileExists( const char *pszFileName );`  
  Check whether a file exists (no wildcards).

- `HB_BOOL hb_fsDirExists( const char *pszDirName );`  
  Check whether a directory exists (no wildcards).

- `HB_BOOL hb_fsCopy( const char *pszSource, const char *pszDest );`  
  Copy a file to a new name.

This is not a complete list of the API, but it does include most of the common functions.

**Parameter meanings:**

- `pszDirName` → Directory name  
- `pszFileName` → File name  
- `hFileHandle` → File handle  
- `ulAttr` → Attributes  
- `uiFlags` → Open flags  
- `iDrive` → Drive number  
- `pBuff` → Input/output buffer

## Example: A simple log system

### C implementation

```c
/******************************************************************************
 ************************** Using the FileSys API *****************************
 ******************************************************************************/

#include "hbapi.h"     /* Always include this for C functions */
#include "hbapifs.h"   /* System (filesystem) API */
#include "hbapiitm.h"  /* ITEM API */

/***
 * Log manager in C
 * IMPORTANT: the last argument must be NULL
 * Example: writeLog("miFic.log", HB_FALSE, "Prueba ", "de ", "LOGMANAGER", NULL);
 * Writes the strings to the file miFic.log; if it exists, append to the end;
 * if it does not, create it and write.
 */

static const char *nfln;
static int iNflnLen;

/*
 * Write to the LOG file
 */
void writeLog( const char *szFileName, HB_BOOL bCreate, const char *zStr, ... )
{
    if( szFileName && zStr )  /* If parameters are provided */
    {
        HB_FHANDLE hFile;
        HB_BOOL bExist = hb_fsFileExists( szFileName ); /* Check if file exists */

        if( bCreate == HB_TRUE || bExist == HB_FALSE )
        {
            hFile = hb_fsCreate( szFileName, FC_NORMAL ); /* Create the file */
        }
        else
        {
            hFile = hb_fsOpen( szFileName, FO_READWRITE ); /* Open the file */
        }

        if( hFile != FS_ERROR ) /* If no errors */
        {
            va_list vl;

            nfln = hb_conNewLine();
            iNflnLen = strlen( nfln );

            hb_fsSeekLarge( hFile, 0, FS_END );
            va_start( vl, zStr );

            do  /* Traverse the variadic arguments and write each to the LOG */
            {
                /* Use the Large variant in case a block exceeds 64K */
                hb_fsWriteLarge( hFile, zStr, strlen( zStr ) );
                hb_fsWrite( hFile, nfln, iNflnLen ); /* Write end-of-line */
            }
            while( ( zStr = va_arg( vl, char * ) ) != NULL );

            va_end( vl );

            /* Close also commits, so hb_fsCommit() is not required */
            hb_fsClose( hFile );
        }
    }
}

/*
 * writeLog for PRG usage
 * Accepts only one parameter to write, but it can be of any type (not just string)
 */
HB_FUNC( WRITELOG )
{
    const char *szFileName = hb_parc( 1 );            /* File name */
    HB_BOOL bCreate = hb_parldef( 2, HB_FALSE );      /* Force creation */

    if( szFileName && hb_pcount() > 2 )
    {
        HB_SIZE nLen;
        HB_BOOL bFreeReq;
        /* Convert the passed value to string */
        char *szStr = hb_itemString( hb_param( 3, HB_IT_ANY ), &nLen, &bFreeReq );

        /* Call the C function that writes to the LOG */
        writeLog( szFileName, bCreate, szStr, NULL );

        if( bFreeReq )
        {
            /* Free memory if hb_itemString() indicated that it’s required */
            hb_xfree( szStr );
        }
    }
}

/*
 * Load the LOG file into a variable
 */
char *loadLog( const char *szFileName, HB_SIZE *nSize )
{
    HB_FHANDLE hFile = hb_fsOpen( szFileName, FO_READ ); /* Open file read-only */
    char *szValue = NULL;

    if( hFile != FS_ERROR )
    {
        *nSize = hb_fsGetSize( hFile );            /* Get file size */

        /* Allocate buffer: file size + newline */
        szValue = ( char * ) hb_xgrab( *nSize + iNflnLen );

        hb_fsSeekLarge( hFile, 0, FS_SET );        /* Seek to beginning */
        hb_fsReadLarge( hFile, szValue, *nSize );  /* Read all into buffer */
        hb_fsClose( hFile );                       /* Close LOG file */
    }

    return szValue; /* Return the LOG content in this variable */
}

/*
 * Load the LOG file into a variable from PRG
 */
HB_FUNC( LOADLOG )
{
    const char *szFileName = hb_parc( 1 ); /* LOG file name */

    if( szFileName )
    {
        HB_SIZE nSize = 0;
        char *szValue = loadLog( szFileName, &nSize );

        /* Return the LOG file content */
        hb_retclen_buffer( szValue, nSize );
    }
    else
    {
        /* If no file name passed, return a NULL string */
        hb_retc_null();
    }
}
```

### PRG usage: several examples

```harbour
//------------------------------------------------------------------------------
// Example usage of the log manager system
// Program ej01.prg
//------------------------------------------------------------------------------

PROCEDURE Main

    LOCAL n := 0
    LOCAL cFNane := "miArchivo.log"

    CLS

    // Reset if the file exists
    // Simulate errors
    writeLog( cFNane, .t., hb_TToC( hb_DateTime() ) + ;
        " - Se ha producido el error numero: " + HB_NToS( n ) )
    // Append
    FOR n := 1 TO 20
        writeLog( cFNane, .f., hb_TToC( hb_DateTime() )  + ;
            " - Se ha producido el error numero: " + HB_NToS( n ) )
    NEXT

    ? loadLog( cFNane )

    ? "Ahora creamos y leemos otro con el mismo nombre"
    ? "escribiendo diferentes tipos de datos:"

    Inkey( 100 )

    ?

    // Write to the file. If it exists, destroy and recreate it (.t.)
    writeLog( cFNane, .t., "------------------------------------------------------------------------" )
    // Write the following lines to the existing file
    writeLog( cFNane,, "Esta es la priemra linea..." )
    writeLog( cFNane,, Date() )
    writeLog( cFNane,, Time() )
    writeLog( cFNane,, 1367.89 )
    writeLog( cFNane,, .t. )
    writeLog( cFNane,, "" )
    writeLog( cFNane,, "Esto es todo..." )

    ? loadLog( cFNane )

    Inkey( 100 )

RETURN
```

```harbour
//------------------------------------------------------------------------------
// Example usage of the log manager system
// Program ej02.prg
//------------------------------------------------------------------------------

#include "Box.ch"

PROCEDURE Main

    LOCAL n := 0
    LOCAL cFNane := "miArchivo.log"

    // Reset if the file exists
    // Simulate errors
    writeLog( cFNane, .t., hb_TToC( hb_DateTime() ) + ;
        " - Se ha producido el error numero: " + HB_NToS( n ) )
    // Append
    FOR n := 1 TO 20
        writeLog( cFNane, .f., hb_TToC( hb_DateTime() )  + ;
            " - Se ha producido el error numero: " + HB_NToS( n ) )
    NEXT

    verLog( cFNane )

    CLS

    ? "Ahora creamos y leemos otro con el mismo nombre"
    ? "escribiendo diferentes tipos de datos:"

    Inkey( 100 )

    // Write to the file. If it exists, destroy and recreate it (.t.)
    writeLog( cFNane, .t., "-------------------------------------------------------------" )
    // Write the following lines to the existing file
    writeLog( cFNane,, "Esta es la priemra linea..." )
    writeLog( cFNane,, Date() )
    writeLog( cFNane,, Time() )
    writeLog( cFNane,, 1367.89 )
    writeLog( cFNane,, .t. )
    writeLog( cFNane,, "" )
    writeLog( cFNane,, "Esto es todo..." )

    verLog( cFNane )

    CLS
    ? "Ahora vamos a leer cuaquier archivo, por ejemplo el código fuente ej01.prg"

    Inkey( 100 )

    verLog( "ej02.prg" )

RETURN


//------------------------------------------------------------------------------
// Show the file’s content using the log system
STATIC PROCEDURE verLog( cFileName )

    CLS

    @ 01, 25 SAY "Contenido de: " + cFileName COLOR "GR+/R+"
    @ 02, 09, 21, 71 BOX B_DOUBLE_SINGLE + Space(1)

    MemoEdit( loadLog( cFileName ), 03, 10, 20, 70, .f. )

RETURN

//------------------------------------------------------------------------------
```

## A class in C that buffers writes to a file

```harbour
//------------------------------------------------------------------------------
// Class that manages a file buffer
// Implemented almost entirely in C

#include "hbclass.ch"

#define FB_MAX_SIZE  65535

CLASS TFBuffer

    DATA hFB  // Internal use

    METHOD new( hFile, nSizeBuffer ) CONSTRUCTOR
    METHOD free()              // Free memory
    METHOD creaBuffer( hFile, nSizeBuffer )  // Internal use
    METHOD addString( cString ) // Append a string to the buffer; auto ::flush() as needed
    METHOD addValue( value )    // Convert a value to string and append to the buffer
    METHOD flush()              // Optionally force a manual flush to disk

END CLASS

//------------------------------------------------------------------------------
// Class constructor

METHOD new( xFile, nSizeBuffer ) CLASS TFBuffer

    LOCAL hFile
    LOCAL nType := ValType( xFile )

    IF nType == 'N'
        hFile := xFile
    ELSEIF nType == 'C'
        hFile := FCreate( xFile )
    ELSE
        Alert( "Falta el nombre o el manejador del fichero" )
    ENDIF

    IF hFile > 0
        IF ValType( nSizeBuffer ) != 'N' .OR. nSizeBuffer < 0
            nSizeBuffer := FB_MAX_SIZE
        ENDIF

        ::creaBuffer( hFile, nSizeBuffer )
    ELSE
        Alert( "No se ha podido usar fichero " )
    ENDIF

RETURN Self
```

### C implementation for the TFBuffer methods

```c
//------------------------------------------------------------------------------
// C implementation of the methods

#pragma BEGINDUMP

#include "hbapifs.h"
#include "hbapiitm.h"
#include "hbstack.h"

/* Buffer structure */
typedef struct
{
    unsigned char *pBuffer;
    unsigned int uiLen;
    unsigned int uiPos;
    HB_FHANDLE hFile;
} FBUFFER, *PFBUFFER;

static void __flushFB( PFBUFFER pFB );

//------------------------------------------------------------------------------
// Method to create the buffer structure
HB_FUNC_STATIC( TFBUFFER_CREABUFFER )
{
    PHB_ITEM Self = hb_stackSelfItem();
    PFBUFFER pFB = ( PFBUFFER ) hb_xgrab( sizeof( FBUFFER ) );

    pFB->uiLen   = hb_parnint( 2 );
    pFB->pBuffer = ( unsigned char * ) hb_xgrab( pFB->uiLen );
    pFB->uiPos   = 0;
    pFB->hFile   = ( HB_FHANDLE ) hb_parnint( 1 );

    hb_arraySetPtr( Self, 1, pFB );
}

//------------------------------------------------------------------------------
// Method to free the buffer
HB_FUNC_STATIC( TFBUFFER_FREE )
{
    PHB_ITEM Self = hb_stackSelfItem();
    PFBUFFER pFB = hb_arrayGetPtr( Self, 1 );

    if( pFB )
    {
        HB_BOOL fCloseFile = hb_parldef( 1, HB_FALSE );

        __flushFB( pFB );

        if( pFB->pBuffer )
        {
            hb_xfree( pFB->pBuffer );
        }

        if( fCloseFile && pFB->hFile )
        {
            hb_fsClose( pFB->hFile );
        }

        hb_xfree( pFB );
    }
}

//------------------------------------------------------------------------------
// Method to append strings to the buffer
HB_FUNC_STATIC( TFBUFFER_ADDSTRING )
{
    PHB_ITEM Self = hb_stackSelfItem();
    PFBUFFER pFB = hb_arrayGetPtr( Self, 1 );
    PHB_ITEM pString = hb_param( 1, HB_IT_STRING );

    if( pString )
    {
        unsigned int uiPos = 0;
        const char *szString = hb_itemGetCPtr( pString );
        unsigned int uiLen = hb_itemGetCLen( pString );

        while( uiPos < uiLen )
        {
            if( pFB->uiPos == pFB->uiLen )
            {
                __flushFB( pFB );
            }

            pFB->pBuffer[ pFB->uiPos++ ] = ( unsigned char ) szString[ uiPos++ ];
        }
    }
}

//------------------------------------------------------------------------------
// Method to append arbitrary values to the buffer (converted to string)
HB_FUNC_STATIC( TFBUFFER_ADDVALUE )
{
    PHB_ITEM Self = hb_stackSelfItem();
    PFBUFFER pFB = hb_arrayGetPtr( Self, 1 );
    PHB_ITEM pValue = hb_param( 1, HB_IT_ANY  );

    if( pValue )
    {
        unsigned int uiPos = 0;
        HB_SIZE uiLen;
        HB_BOOL fFree;
        char *szString = hb_itemString( pValue, &uiLen, &fFree );

        while( uiPos < uiLen )
        {
            if( pFB->uiPos == pFB->uiLen )
            {
                __flushFB( pFB );
            }

            pFB->pBuffer[ pFB->uiPos++ ] = ( unsigned char ) szString[ uiPos++ ];
        }

        if( fFree )
        {
            hb_xfree( szString );
        }
    }
}

//------------------------------------------------------------------------------
// Method to force a flush to disk and reset write position
HB_FUNC_STATIC( TFBUFFER_FLUSH )
{
    PHB_ITEM Self = hb_stackSelfItem();
    PFBUFFER pFB = hb_arrayGetPtr( Self, 1 );

    __flushFB( pFB );
}

//------------------------------------------------------------------------------
// Internal helper: write buffer to disk and reset position
static void __flushFB( PFBUFFER pFB )
{
    if( pFB->uiPos > 0 )
    {
        hb_fsWriteLarge( pFB->hFile, pFB->pBuffer, pFB->uiPos );
        pFB->uiPos = 0;
    }
}

#pragma ENDDUMP
```

### Usage examples of `TFBuffer`

```harbour
//------------------------------------------------------------------------------
// Example usage of TFBuffer
// Program ej03.prg
//------------------------------------------------------------------------------

#include "tfbuffer.prg"

PROCEDURE Main

    LOCAL hFile := FCreate( "prueba.log" )
    LOCAL o := TFBuffer():new( hFile )
    LOCAL i, t

    SET DATE FORMAT TO "yyyy-mm-dd"

    CLS

    @ 09, 10 SAY "Se escribiran un millon de lineas."
    @ 10, 10 SAY "Espere, estoy procesando datos..."

    t := Seconds()

    // For strings you can use ::addString(); for everything else, ::addValue()
    o:addString( "--------------------------------------------------------------" + hb_eol() )
    FOR i := 1 TO 1000000
        o:addValue( i )                  // Numeric
        o:addValue( ( i % 2 ) == 0 )     // Logical
        o:addString( " <> " )            // String
        o:addValue( Date() )             // Date
        o:addValue( " " )                // String
        o:addString( Time() + hb_eol() ) // String
    NEXT
    o:addString( "--------------------------------------------------------------" )

    t := Seconds() - t

    o:free()

    FClose( hFile )

    // Compute size in KiB
    i := Hb_FSize( "prueba.log" ) / 1024

    Alert( "Ha tardado: " + AllTrim( Str( t ) ) + ;
        " segundos con un fichero de: " + AllTrim( Str( i ) ) + " kB")

RETURN

//------------------------------------------------------------------------------
```

```harbour
//------------------------------------------------------------------------------
// Example usage of TFBuffer
// Program ej04.prg
//------------------------------------------------------------------------------

#include "tfbuffer.prg"

PROCEDURE Main

    LOCAL o := TFBuffer():new( "prueba.log" )  // If a name is passed, it creates and opens it
    LOCAL i, t

    SET DATE FORMAT TO "yyyy-mm-dd"

    CLS

    @ 09, 10 SAY "Se escribiran un millon de lineas."
    @ 10, 10 SAY "Espere, estoy procesando datos..."

    t := Seconds()

    // For strings you can use ::addString(); for everything else, ::addValue()
    o:addString( "--------------------------------------------------------------" + hb_eol() )
    FOR i := 1 TO 1000000
        o:addValue( i )                  // Numeric
        o:addValue( ( i % 2 ) == 0 )     // Logical
        o:addValue( " <> " )             // String
        o:addValue( Date() )             // Date
        o:addValue( " " )                // String
        o:addString( Time() )            // String
        o:addValue( hb_eol() )           // String
    NEXT
    o:addString( "--------------------------------------------------------------" )

    t := Seconds() - t

    o:free( .t. ) // true → also close the file

    // Compute size in KiB
    i := Hb_FSize( "prueba.log" ) / 1024

    Alert( "Ha tardado: " + AllTrim( Str( t ) ) + ;
        " segundos con un fichero de: " + AllTrim( Str( i ) ) + " kB")

RETURN
```

> Notes
>
> * Comments inside C/PRG examples have been translated to English. The string
>   literals remain as in the original examples, exactly as requested.
> * If you ever want the literals localized too, I can provide a separate variant.

