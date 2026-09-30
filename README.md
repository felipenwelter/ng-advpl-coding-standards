# AdvPL Coding Standards

This document sets out a standard of good practices for the NG development teams working on the Protheus AdvPL platform. The rules listed here are **not restrictive**; they aim to give the source code a single, consistent identity. Following the standard should naturally become a habit, making the code easier to read and making system maintenance safer.

New functions should be written following these practices, and existing functions should be brought in line gradually. However, changing code that is outside the scope of a change is **not recommended**, so as not to complicate the code review or accidentally introduce new bugs.


## Files

- The file extension must be lowercase (for example: `.prw`, `.apw`)

- The file name should preferably be lowercase

## Style

- Use tabs only for indentation, not for alignment

> Tabs should only be used on the left, before the line starts, and never in the middle of a line. That way the formatting is preserved when the file is opened in different editors (TDS, VSCode, Sublime, etc.)

- Variable names should follow Hungarian notation

>Start with:
>- n - numeric
>- c - char/string
>- d - date
>- l - logical/boolean
>- a - array/matrix
>- o - object
>- b - code block
>- x - undefined

- Avoid variable names like `nX` or `nY` (except for indexes). Be more descriptive

- Prefer declaring variables grouped logically by type or purpose, on separate lines

- Initializing variables with `Nil` is redundant, except for public variables, which should be rare anyway: they are generally considered bad practice because they pollute the global scope

- Language keywords should use **UpperCamelCase** (examples: `If`, `EndIf`, `While`)

- To end a `While` or `Do While` statement, prefer `End` and `EndDo` respectively

- To end a `For` statement, prefer `Next <variable>` over just `Next`

- Local variable names should use **lowerCamelCase** (examples: `cName`, `nAge`)

- Function names in Hungarian notation should use **lowerCamelCase** (example: `aAdd`)

- Always write a variable name with the same length and case (don't mix `thisIsMyVari` and `THISISMYVARI`)

- Function names without Hungarian notation should use **UpperCamelCase** (example: `RetSqlName`)

- Preprocessor directives (#define < include >) should reference the include with the same capitalization as the physical file, preferably lowercase.

> This is a good practice that avoids case-sensitivity conflicts when compiling on Linux, for example.

- Avoid going past 120 columns. Break the code with `;` when needed

- Logical values should be uppercase (example: `.F.`)

- Put spaces around operators. Use `nValue > nExpected` instead of `nValue>nExpected`

- Use one language consistently. Avoid mixing Portuguese and English where possible

- Leave 1 blank line after each *statement*, except for groups of related *statements*

```
Function Test()

    If Something
        ...
    EndIf

    For nI := 1 To 10
        For nZ := 1 To 10
            ...
        Next nZ
    Next nI

Return
```

- Separate functions with a blank line after the return, before starting a new block

- Leave 1 empty line at the end of each file

> Diff tools get confused without a final newline, and some editors already add one by default. Some compilers don't recognize the end of the file without it (not the case for AdvPL), so it's a good programming practice.

- Prefer single quotes `'` over double quotes `"`

> Single quotes make the code easier to read and let you use double quotes inside strings (e.g. cString := 'Check the "quantity" field on the screen')

- To access multiple indexes, avoid `aList[ nI ][ nJ ]`. Use `aList[ nI, nJ ]`

- Avoid nesting more than three `For.. Next` loops

- Functions should **not** take more than 6 parameters (this keeps a function cohesive, with a single purpose)

> This standard applies to practically any programming language with functions. Large "master" functions are much harder to maintain and test than functions that do one thing and do it well. Unit tests fit simple functions easily but are hard to write for functions with many parameters. It's also common for people to get lost among so many empty parameters (e.g. TObj:New(100,30,,,,,,.F.)).

> Keep in mind that each function should be exactly that: one function. There are dozens of software engineering articles on how functions with many parameters break modularity and reuse. Functions that need many parameters usually point to an architecture that could be better abstracted.

- Don't write tightly coupled functions, meaning functions that depend on other functions or variables in order to work (such as functions that use private variables declared in another source file)

- Avoid nesting more than 3 statements (for example: an `If` inside an `If` inside an `If`)

- The inline If function should be written with both I's in uppercase: `IIf`

- Use `If` only for *statements* (If .. Else .. EndIf) and `IIf` for *expressions* (x:= IIf(lVar,10,20))

> *Statements* don't return a value, for example `If (lVar)`, while *expressions* do, for example `x := If(lVar, 10, 20)`. AdvPL allows the ambiguity of using both `If` and `IIf` in expressions. In that case, always use `IIf`, because with `If` the compiler will look for the `EndIf`, which could cause errors in other languages.

- Use `!=` instead of `<>`

- Prefer `==` for comparison instead of just `=`

- Don't write `== .T.`

- Use `!` instead of `.Not.`

## Spacing

- Put 1 space inside function arguments, blocks and arrays (example: `RetSqlName( 'STJ' )`)

- Put 1 space after each comma (example: `{ 1, 2, 3 }`)

- Put spaces between function parameters (use `Call( 1, 2, 3 )`, not `Call(1,2,3)`)

> The recommendation is to add a space before the first parameter, between parameters and after the last one. The first and last spaces are optional.

- `Return` may be aligned to the left (same alignment as Function), even though it is not a terminator

- In comments, put 1 space after `//`

> While not mandatory, a space before the comment text makes it easier to read and to find and replace language patterns programmatically.

- Prefer `//` for single-line comments and `/**/` for blocks of code or parameter comments (e.g. function(10`/*nHeight*/`,20`/*nWidth*/`))

## Redundancy

- Replace an `If` inside an `If` with `.And.`

## Behavior

- When creating a temporary table, remember to close it with `dbCloseArea`, and when using FWTemporaryTable, call its `delete` method

- Remember to close the file _handler_ with `fClose` when using `fOpen`

- When moving to another record with `dbSkip`, make sure the right table is selected; otherwise use `TABLE->( dbSkip() )` or call `dbSelectArea( TABLE )` before `dbSkip()`

- Be careful when calling functions (e.g. FWTemporaryTable) inside transaction blocks, because on Oracle the call may commit the data in the middle of the process

## Functions

- Every function must have a documentation header based on the Protheus.doc template, and a generic author (such as "NG Informática") must not be used

- **Generic functions** are those that apply to any module, usually without handling business concepts or rules. They should be created in generic source files such as NGUtil and preferably start with `NG`

- **Module generic functions** are those that handle more general rules of a module and are used by a group of routines. They should be created in the module's source files, such as MNTUtil or MDTUtil, and preferably start with the module prefix, e.g. `MNT` or `MDT`

- **Routine functions** are those that handle rules specific to one routine but can also be called from elsewhere. They should be created in the source file that uses them and preferably start with the routine's own name, such as `MNTA080Cad()`. Also avoid shortening the source identifier to `MNA080` or `MNT080`

- **Static functions** are those used only by a single source file, with no external calls from other files. They should be created in the source file that uses them and preferably start with `f`, as in `fCalcHora()`


## References

AdvPL Coding Standards - https://github.com/haskellcamargo/advpl-coding-standards by @haskellcamargo
