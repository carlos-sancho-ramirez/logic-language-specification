# Logic Programming Language Specification

## Objectives of the language

This language focuses on making business logic.
All tasks regarding optimizations or memory handling are outside of the scope of this language and should be covered by the compiler or other pieces of code written in other languages.

This language has these objectives in mind:
* Keep syntax as simple as possible. Extensions can be provided by libraries.
* Enforce best practises regarding the use of upper and lower cases.
* Abstract variable limits: So the developer should not care if the integer value is using 8 bits or 32 bits of memory. This allow the compiler take decisions to make the solution more efficient in speed or memory in use, and allow the developer to focus in the actual business logic.
* Abstract variable units: So the developer can use 3 booleans in a single register without thinking in optimizations like composing an integer and use each bit as a boolean instead.
* Exclude mutability: Developers will deal with a bunch of constants, but no variables where values can change. This simplifies the readability and reduces the complexity of the code.
* Avoid memory handling as much as possible. Developers should not care when a variable is allocated or removed from memory. They should not care either if variables are located in the heap, the stack or registers.

## Basic syntax

Code can be parsed as a list of tokens. Tokens may be operators, references or literals.

Tokens whose nature (operators vs references) do not share possible characters may not require explicit separation.
However, tokens of the same nature must be separated by an arbitrary combination of whitespaces, tabs, carriage return characters or new line characters.
Compilers will ignore the indentation used by the developer, or where a line breaks appear.

This language is case-sensitive, and compilers can rely in the nature of each token by checking its case.
These are the 5 types of token that can be found:

  * Keywords: a set of reserved English words. All its characters will be from *a* to *z*, all in lower case, no cyphers nor symbols are allowed.
  * Constant names: Constants created by the developer to assign expressions on it and enforce reusability. They follow the format known as camel-case. So, they must start with a lower-case character from *a* to *z* and may be followed by any arbitrary number of upper or lower case character from *a* to *z* or cyphers from *0* to *9*. Other symbols (even the underscore) are forbidden.
  * Types: Developer defined and built-in types. This follow the format known as Pascal-case. So, they must start with an upper-cased character from *A* to *Z* followed by any arbitrary number of upper or lower cased characters from *a* to *z* or cyphers from *0* to *9*.
  * Enumeration values: name that belongs to a set of possibilities and have a implicit value assigned. They must start with an upper case from *A* to *Z*, and followed by one or more upper-cased characters from *A* to *Z* or the underscore character to separate words. They must have at least 2 characters, this will allow developers, parsing tools or compilers to distinguish it from a type reference.
  * Literals: Specifies a constant value. They can be integer, character or string literals.
    * Integer literals: Defines a integer numeric value. They must start with a cypher (character from *0* to *9*), or the minus sign (-) followed by a cypher in case of a negative number. This language admits hexadecimal literals by starting the literal with *0x*. Except for the explicit zero value, starting an integer literal with *0* which is not followed by a *x* is not allowed and compilers should complain here. These guarantees that it can be extended in the future in case of being required. This token ends whenever any space, tab, carriage-return, new-line, operator or separator is found. If any other character, or a character from *a* to *z* or *A* to *Z* is attached to the end of this token, compilers should complain as well.
    * Character literal: It starts and ends with the single quote symbol ('), and can only contain one character between the quotes. Compilers should convert its codepoint into an integer and treat it like any integer literal.
    * String literal: It starts and ends with the double quote symbol ("). There can be any arbitrary number of characters between the quotes, even 0. This is sugar, and compilers should convert this literal into a construction of an array of characters.
  * Operators: a set of symbols reserved to perform logical, math or other operations among the tokens.

### Keywords

All reserved words within the language are expected to be short one-word tokens. All of them, fully written in lower case.

Unluckily, we are unsure on which keywords will be included in future versions of the language,
so there is a potential risk that old code may use keywords not defined before as constant names.
Compilers should consider any keyword as reserved and raise an error in case the developer uses any of the keyword as constant name.

This is the list of all keywords defined (sorted alphabetically):
  * else
  * if
  * then
  * type

### Operators

Operator are symbols used to define arithmetic or logical operators, comparisons and others.
There is a precedence order that any compiler should follow. If operators are in the same level of precedence, the compiler will execute them left to right.

#### Precedence level 0: Precedence orchestrators

    ( ) [ ] { }

Parentheses, square brackets and curly brackets form this level. These can alter the order in which other operators are executed.

#### Precedence level 1: Membership

    .     Membership

Dots can be used to point to a value that is included inside other value. For example, if we have a register for Student, we could use the dot operator to access details of that student, like its name.

    student.name

The dot operator implies that the left-side term is either a register or an array.

#### Precedence level 2: Range

    ..    Range

2 dots are used to define a range of numbers.
When using this, both left and right side terms must be numbers, and the left one must be lowe or equals than the right one.
Currently, this is only used to define explicit ranges for integer types.

#### Precedence level 3: Multiplication, division and module arithmetic operators

    *     Multiplication
    /     Division
    %     Module

The asterisk is used to multiply 2 values.
The use of this operator implies that any value in the left and any value on the right must be a number, if not, compiler should complain on this.

The slash character is the division operator, and divides the left term side by the right term side.
Like in the multiplication, the use of this operator implies that any value in the left and right of this operator is a number.
Compiler should abort compilation if that rule does not match.

On the other side, developers should care not to provide the value 0 in the right side. Ideally, compilers should complain if it is possible to have 0 in the right side.

The percentage sign is used to obtain the rest of a division (module). Note that the result of this operation will always go from 0 to the value in the right term side without reaching it.
Like in the others, this operator implies that the value on each side must be a number. Compilers must complain if that is not true.

In any case, the execution of any of these operators will result in another number, but its bounds may differ.

Example:

    a * 2

Assuming that *a* can have values 27, 140 or 213, the possible result value will be 54, 280 or 426. If internally *a* was stored in a byte (8-bits unit of memory) the result 54 still can fit in another byte, but not the other, that probably will need more bits. Compilers will handle this silently, so the developer should not worry on the actual limits of the numbers.

#### Precedence level 4: Addition and subtraction arithmetic operators

    +     Addition / Concatenation
    -     Subtraction

The plus sign is used in this language for 2 purposes: the sum of 2 numbers or the concatenation of 2 arrays. In any case, the precedence level will be the same.

If the left term side is a number, the right term value must be a number as well. But if the left term is an array, the right one must be an array as well.

The minus sign is used to subtract.

#### Precedence level 5: Comparisons

    ==    Equals than
    !=    Different from
    >=    Greater or equals than
    <=    Lower of equals than
    >     Greater than
    <     Lower than

There are 6 possible comparisons that developers can use.
When using them, the type of the value on the left must match the type of the value on the right.
In any case, this will result into a Boolean. So it will be either *TRUE* or *FALSE*.

#### Precedence level 6: And logical operation

    &     And logical operation

The use of this operator implies that left-side term and right-side terms are both Boolean expressions.
If that is not the case, compilers must complain on it.

Note that this is NOT a bitwise operation. Bitwise operations are explicitly excluded from the programming language.
Bit managements is responsibility of the compiler, not the responsibility of the developer using this language.
Because of this, there is no need of the *&&* operator, like in C/C++.

#### Precedence level 7: Or logical operation

    |     Or logical operation

Like with the *And* operator, the use of this operator implies that left-side term and right-side terms are both Boolean expressions.
If that is not the case, compilers must complain on it.

Note that this is NOT a bitwise operation. Bitwise operations are explicitly excluded from the programming language.
Because of this, there is no need of the *||* operator, like in C/C++.

#### Precedence level 8: Function definition

    ->    Arrow

The arrow is used to separate the list of parameters from the body expression of a function.

#### Precedence level 9: Separators

    =     Assignment
    ,     Comma
    ;     Semicolon
    :     Colon

The equals sign is the assignment. When used, the left-side term will be a name for a new constant, and the right-side term will be a expression.
Using the assignment operator transforms the expression into an statement, and because of that, the semicolon separator will be expected at the end of the expression assigned.

The comma separator is used to separate parameters within functions.

The semicolon is used at the end of statements.
This allows the concatenation of several statements.

The colon is used to define type explicitly.

### Enumeration constants

Enumeration constants names are always all in uppercase, and must be at least 2 characters long.

Currently only the Boolean type is built-in defined, whose values are:

    TRUE
    FALSE

These constants con be used to compare values against them.

### Types

All references for types in this language must start with upper-case, and must contain at least one lower case character. This is required distinguish them from any enumeration constant, where all its characters are upper-case.

This is the list of built-in types (sorted alphabetically):

    Array       Array of values
    Boolean     Enumeration that can include values TRUE and FALSE
    Int         Integer number
    String      Sugar for Array[Int]

Some type allow other types as parameters. Parameters are included using the square brackets.

Examples:

    Int[0..255]
    Array[Int[32..127]]

*String* is a shortcut (sugar) for *Array[Int]*.
