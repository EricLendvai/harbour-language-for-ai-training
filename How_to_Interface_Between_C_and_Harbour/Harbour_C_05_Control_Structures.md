---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 5
chapter_title: "Control Structures"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 5. Control Structures

In this section, we will review the different programming structures in **C**, each with its corresponding syntax diagram.

---

## 5.1 Conditional Structures

### `if`
Works similarly to how it does in **Harbour**:

```c
if ( condition )
{
    statements;
}
```

### `if ... else`
```c
if ( condition )
{
    statements;
}
else
{
    default_condition_statements;
}
```

### `if ... else if`
```c
if ( condition1 )
{
    statements;
}
else if ( condition2 )
{
    statements;
}
// There can be more else if() blocks
else
{
    default_condition_statements;
}
```

---

### Ternary Conditional Operator `?`
It has the following syntax:

```c
expression1 ? expression2 : expression3;
```

In **Harbour**, there is an equivalent construct that works exactly the same way, using `iif()`:

```harbour
iif( expression1, expression2, expression3 )
```

If `expression1` evaluates to true, the result takes the value of `expression2`; otherwise, it takes the value of `expression3`.

**Example:**
Finding the maximum between two numbers:

```c
iMax = a > b ? a : b;
```

Equivalent to:

```c
if ( a > b )
{
    iMax = a;
}
else
{
    iMax = b;
}
```

---

### `switch`
Works similarly to Harbour’s `do case` structure, but in C it can only be used with integer or character expressions.  
It behaves like nested `if` statements.

```c
switch ( expression )
{
    case exprConst1:
        statements;
        break;

    case exprConst2:
        statements;
        break;

    case exprConstN:
        statements;
        break;

    default:
        default_statements;
        break;
}
```

---

## 5.2 Iterative Structures

### `while`
```c
while ( condition )
{
    statements;
}
```

### `do ... while`
This construct **does not exist in Harbour**.  
It guarantees that the body statements are executed **at least once**.

```c
do
{
    statements;
}
while ( condition );
```

### `for`
Its syntax is quite different from Harbour’s:

```c
for ( initialization_block; conditional_expression; increment_block )
{
    statements;
}
```

- **initialization_block**: evaluated once; initializes one or more variables.  
- **conditional_expression**: evaluated on each iteration; checks the loop’s exit condition.  
- **increment_block**: executed after each iteration; usually increments or decrements control variables.  
- Each block can contain more than one expression, separated by commas `,`.

**Example:** prints the multiplication table of a given number `n`:

```c
for ( i = 0; i <= 10; i++ )
{
    printf( "n x %d = %d\n", i, i * n );
}
```

---

## 5.3 Control Statements: `continue` and `break`

For these kinds of programming structures:

- `continue` forces the **next iteration** inside a loop.  
- `break` forces an **exit** from a selective or repetitive structure.

---
