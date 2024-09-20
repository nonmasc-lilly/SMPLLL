# Simple Macro Predicated Low Level Language
### Lilly H. St Claire

***

## 1.0 Introduction

The Simple Macro Predicated Low Level Language (SMPLLL, shortened
to SMPL) is a programming language which compiles to 8086 machine
code. It exists as a tool for writing small sixteen bit operating
systems for Legacy BIOS machines. It follows a form in which
each line begins with the kind of argument and then its arguments.
The four arguments are as follows:

- including a file
- Embed byte list
- Invoke macro
- Create an argument macro
- Create byte list macro
- Create invoker macro
- Remark on the code

### 1.1 Syntax conventions

Before we begin its important to note that all syntax in this
document is using ABNF as described in **RFC 5234** *Augmented
BNF for Syntax Specifications: ABNF* found at:
<https://datatracker.ietf.org/doc/html/rfc5234>

An example of the form described in Section 1.0 is displayed below:

```ABNF
ws = 1*(%x20 / %x9)
nl = 1*(%xA  / %xD)
integer = *DIGIT / '$' *HEXDIG
identifier = ALPHA *(ALPHA / DIGIT)
program = *([include / byte_list | invoke_macro | define_macro
    / remark] nl)
define_macro = define_byte_list_macro / define_invoker_macro |
    define_argument_macro
```

### 2.0 The Preprocessor

The preprocessor in SMPL serves two purposes, to replace remarks
with empty lines and to replace include statements with the
contents of the file which its argument refers to (within it's
quotes). The syntax for both of these follows:

```ABNF
include = 'i' [ws] '"' 1*(VCHAR / ws) '"'
remark  = 'r' [ws] '"'  *(VCHAR / ws) '"'
```

### 3.0 Byte Lists

"Byte lists" in SMPL simply refer to a series of bytes. When the
code is compiled these byte lists are directly translated to output
code. This is why the act of creating a byte list is called
"embedding", as we are literally embedding the list into the code
at the place of embed.

The syntax for embedding a byte list is as follows:

```ABNF
byte_list = '<' [ws] integer [ws] *(',' [ws / nl] integer [ws]) '>'
```

### 4.0 Macros

Macro's are the core usefulness of SMPL, they are a transformation
of some arguments into a byte list (which may then be embedded).
There are three kinds of macros.

- Argument macros
- Byte list macros
- Invoker macros

They are described in the following sections.

### 4.1 Argument Macros

An argument macro is simply a macro which can be used as an
argument to another macro, they themselves do not take arguments
but instead refer to some string value. The syntax is as follows:

```ABNF
define_argument_macro = '%' [ws] identifier [ws]
    '"' *(VCHAR / ws) '"'
```

### 4.2 Byte List Macros

A byte list macro simply takes some number of arguments and
produces a byte list based on said arguments. The syntax follows:

```ABNF
define_byte_list_macro = '#' [ws] identifier ws integer [ws] '<'
    argument_expression [ws] *(',' [ws / nl] argument_expression
    [ws]) '>'
```

### 4.3 Invoker Macros

An invoker macro simply takes some number of arguments and invokes
a series of macros in order. Macros whose arguments are based on
the arguments of the invoker macro. The syntax for this is as
follows:

```ABNF
define_invoker_macro = ']' [ws] identifier ws integer [ws] '{'
        [nl] *(invoke_macro_wargexpr nl)
    '}'
```

### 4.4.0 Argument Expressions

An argument expression is a set of values which may be included in
the bodies of macro definitions, and are one of the following
values:

- Integer
- Argument
- Argument macro
- Condition

Thusly the syntax is as follows:

```ABNF
argument_expression = integer / identifier / argument / condition /
    arithmetic
argument = '@' integer
```

### 4.4.1 Conditions

The integer and argument macro are rather self explanatory (in case
they are not, integers are simply numbers, and argument macros are
identifiers which are one of the predefined *Argument Macros*)
conditions, however, are not so. The idea is relatively simple.
If the argument to a macro meets a certain condition, then
substitute the condition to another argument expression, otherwise
substitute a second argument expression. The syntax for this is
as follows:

```ABNF
condition = '?' [ws] condition_expression [ws] '->' [ws]
    argument_expression [ws] ':' [ws] argument_expression
condition_expression = ('=' / '>' / '!=') [ws] argument_expression
    [ws] ',' [ws] argument_expression / ('&' / '/') [ws]
    condition_expression [ws] ',' [ws] condition_expression
```

An example would be a macro which checks if its first argument is
equal or greater to its second and subs a one if true, or a two
if false:

```SMPL
#oneortwo 2 <?/ =@0,@1 >@0,@1 -> 1 : 0>
```

### 4.4.2 Arithmetic

Arithmetic in SMPL is rather simple, there are the following
operations:

- Add
- Subtract
- Multiply
- Modulo

And they have the following syntax:

```
arithmetic = ('+' / '-' / '*' / '//' / '/%') [ws]
    argument_expression [ws] ',' [ws] argument_expression
```

### 4.5 Invokation

Invoking a macro simply means that wherever that macro is placed in
the code will be replaced by whatever the contents of the macro
are, which if they invoke a macro, will recurse. The syntax is
as follows:

```ABNF
invoke_macro = 'm' ws identifier [ws] [(integer / identifier) [ws]
    *(',' [ws / nl] (integer / identifier) [ws])]
invoke_macro_wargexpr = '.' [ws] identifer [ws] [
    argument_expression [ws] *(',' [ws / nl] argument_expression
    [ws])]
```

# 5 Full Syntax

```ABNF
; basic lexical tokens
ws = 1*(%x20 / %x9)
nl = 1*(%xA  / %xD)
; primitives
integer = *DIGIT / '$' *HEXDIG
identifier = ALPHA *(ALPHA / DIGIT)
argument = '@' integer
; program definition
program = *([include / byte_list | invoke_macro | define_macro |
    remark] nl)
; preprocessor
include = 'i' [ws] '"' 1*(VCHAR / ws) '"'
remark  = 'r' [ws] '"'  *(VCHAR / ws) '"'
; basic expressions
byte_list = '<' [ws] integer [ws] *(',' [ws / nl] integer [ws]) '>'
invoke_macro = 'm' ws identifier [ws] [(integer / identifier) [ws]
    *(',' [ws] (integer / identifier) [ws])]
; macros
define_macro = define_byte_list / define_invoker_macro |
    define_argument_macro
define_argument_macro = '%' [ws] identifier [ws]
    '"' *(VCHAR / ws) '"'
define_byte_list = '#' [ws] identifier ws integer [ws] '<'
    argument_expression [ws] *(',' [ws / nl] argument_expression
    [ws]) '>'
define_invoker_macro = ']' [ws] identifier ws integer [ws] '{'
        [nl] *(invoker_macro_wargexpr nl)
    '}'
invoke_macro_wargexpr = '.' [ws] identifier [ws] [
    argument_expression [ws] *(',' [ws / nl] argument_expression
    [ws])]
; argument_expression
argument_expression = integer / identifier / argument / condition /
    arithmetic
condition = '?' [ws] condition_expression [ws] '->' [ws]
    argument_expression [ws] ':' [ws] argument_expression
condition_expression = ('=' / '>' / '!=') [ws] argument_expression
    [ws] ',' [ws] argument_expression / ('&' / '/') [ws]
    condition_expression [ws] ',' [ws] condition_expression
arithmetic = ('+' / '-' / '*' / '//' / '/%') argument_expression
    [ws] ',' [ws] argument_expression
```
