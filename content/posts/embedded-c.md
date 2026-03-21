---
author: "Phong Nguyen"
title: "Embedded C"
date: "2026-03-21"
description: "Embedded C Notes"
tags: ["c, embedded"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Embedded C .

## 1. C Language
### 1.1. Introduction
- **Programming language** `is a set of instructions` that `used to` `communicate with a` computer and `control its behavior`.
- C is a `general-purpose procedural` programming language.

### 1.2. Expression
- An **expression** `is a combination of` operators, constants and variables that evaluate to a value.
- There are `several types of expressions`, including:
  - constant expr: `5, 10+3`
  - integral expr: `x*y`
  - floating expr: `x + 10.75`
  - relational expr: `x <=y`
  - logical expr: `x > y && z == 6`
  - pointer expr: `&x`
  - bitwise: `x<<3`

### 1.3. Variables & Data Types
- A **variable** is `a named location` in memory `that stores data`. Every variable must be declared to specify its type. e.g. `int a;`
- **Common Data Types in C**:

| Data Type       | Size (Typical) | Range                              |
|-----------------|---------------|------------------------------------|
| char            | 1 byte        | -128 to 127                        |
| unsigned char   | 1 byte        | 0 to 255                           |
| short           | 2 bytes       | -32,768 to 32,767                  | 
| int (16-bit)    | 2 bytes       | -32,768 to 32,767                  | 
| int (32-bit)    | 4 bytes       | -32,768 to 32,767                  | 
| unsigned int    | 2 bytes       | 0 to 65,535 (16-bit)               |
| long            | 4 bytes       | -2,147,483,648 to 2,147,483,647    |
| float           | 4 bytes       | ±3.4e±38 (~6–7 digits precision)   |
| double          | 8 bytes       |                                    |

> On a 32-bit machine, `short` is not efficient because the CPU is designed to process 32-bit data in one step.

### 1.4. Operators
- An **operator** is `a symbol` that `performs an operation on value`.
- There are several operators, including:
  - **Arithmetic operators** are used to perform arithmetic operations on operands, include `addition +`, `subtraction -`, `multiplication *`, `division /`, and `remainder %`  
  - **Logical operators** are used to combine two or more conditions into a single expression, include `and &`, `or |`, and `not !`
  - **Bitwise operators** are used to perform bit-level operations on the operands, include `and &`, `or |`, `exclusive-or ^`, `negation ~`, `left shift <<`, and `right shift >>`
  - **Relational operators** are used for the comparison of the two operands, include `equal ==`, `not equal !=`, `greater/e >/ >=`, and `less/e </<=`
  - **Assignment operators** are used to assign value to a variable, include `= ,` and <operator>=`
  - **Casting operators** convert one data to another `(new_type) operand;`
  - **Other**: `& *` ...

> When adding/subtracting integer to pointer, the integer is multiplied by the size of the pointed data.
e.g. long *p; p+1 adds 4 to the address value p.
Thus, **p[i]** is an abbreviation of ***(p+i)**

### 1.5. Statements 
- A **statement** `is a complete instruction` that the program executes.
> An expression produces **a value**, but a statement performs **an action**.

- There are several kinds of statements, including:
  - Expression statements `a=b`
  - Condition statements `if(con), switch`
  - Iteration statements `while(ter_con), for(init,ter_con,do_end_step), do{state} while(ter_con);`
  - Compound statements `{ // list of statements }`
  - Jump statements `goto, break, continue`
  
### 1.6. Data Structures
- **Data structures** `is a way of organizing data` in memory `for efficient use`.
- `It can be classified into` two main types: built-in and user-defined
- `Built-in data structures` includes arrays, structures and unions.
- `User-defined data structures` are implemented using structures and pointers.

#### 1.6.1. Array
- An array `is a collection` of elements `of the same type` stored in contiguous memory locations.
> A multi-dimensional array is an array with more than one dimension, 2-D,3D etc.

#### 1.6.2. Structure
- A **structure** `is a user-defined data type` that groups different types of data together.
- **Data alignment** means placing data in memory `at addresses that match the CPU’s preferred boundaries`. (`short` 2 - others 4)
- **Padding** is `extra unused memory` added by the compiler `to satisfy alignment requirements`.
- e.g.
```cpp
// ======== Memory layout ========
// address	content
// 0	    a         (1 byte)
// 1–3	    padding   (3B)
// 4–7	    b         (4 bytes)
struct Example {
    char a;   // 1 byte
    int b;    // 4 bytes
};
```

#### 1.6.3. Union
- A union is like a structure, but its members `are allocated at the same address` and share the same memory area.
- e.g.
```cpp
// ======== Memory layout (union) ========
// All members share the same starting address
// address    content
// 0–3        i (4 bytes)
//            f (4 bytes)
//            c (1 byte, but overlaps)

union Data {
    int i;     // 4 bytes
    float f;   // 4 bytes
    char c;    // 1 byte
};

enum Type {
    INT_TYPE,
    FLOAT_TYPE
};

struct TaggedData {
    enum Type type;   // tag
    union {
        int i;
        float f;
    } data;
};
```

#### 1.6.4. Enum
- An enumeration (or enum) is a user defined data type that `contains a set of named integer constants`.

### 1.7. Function
- A **function** `is a reusable block of code` used to perform a specific task.
- A **function pointer** `is a pointer that stores the address of a function`, allows indirect function calls.
- A **function prototype** tells the compiler what the function looks like before it is actually defined. `int f(int, int *);`

### 1.8. Preprocessing Directives
- **Declarations** mean giving types to variables or functions, using keyword `extern`. e.g. `extern int x;`
- **Definitions** mean defining the contents for variables or functions.  e.g. `int x = 1;`

- The keyword `extern` is used to declare a variable or function defined elsewhere.
- The keyword `static` is used to specify `internal linkage at file scope` or to `extend the lifetime of a variable inside a function`.

## 2. C Embedded
### 2.1. Program Memory Model
- The **memory layout** of a program `shows how its data is stored` in memory during execution. It typically include the following segments:
  - `Code (Text) Segment`: stores executable code and is usually read-only.
  - `Data Segment`:
    - `Initialized Data Segment`: contains `global` and `static` variable `that are initialized`. 
    - `Uninitialized Data Segment (BSS)`: stores `global` and `static` variable `that are not initialized`.
  - `Heap Segment`: is used for dynamic memory allocation.
  - `Stack Segment`: is used for storing local variables, functions params, and return addresses for each function call.