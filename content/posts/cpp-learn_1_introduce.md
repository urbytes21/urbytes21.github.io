---
author: "Phong Nguyen"
title: "C++ - Chapter 1: Introduction"
date: "2026-05-24"
description: "C++ Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
# C++ Introduction

## 1. Introduction
C++ was developed as an extension to C. It adds many few features to the C language, and tis perhaps best through of as a superset of C. 

#### How To Program 
**Step 1:** Define the problem
- I want to write a program that will ...

**Step 2:** Determine how to solve the problem
Determine how we are going to solve the problem you came up with in step 1.
- They are straightforward (not overly complicated or confusing).

**Step 3:** Write the program
  ```c++
  #include <iostream>
  int main() {
      std::cout << "Hello World";
      return 0;
  }
  ```

**Step 4:** Compiling the source code
- Use a C++ compiler: MinGW/GCC, Clang, ... for many different OS.
- The C++ compiler sequentially goes through each source code file:
  - Checks the C++ code to make sure it **follows the rules** of the C++ language.
  - Translates your C++ code into **machine language instructions**. These instructions are stored in an intermediate file called **an object file**.

**Step 5:** Linking object files and libraries
- After the compiler has successfully finished, another program called the **linker** kicks in: `ar`,`ld`, ...
- Linking is to combine all the object files and produce the desired output file (.exe, .elf, .hex ..)

> NOTE: Building refer to the full process of converting source code files into an executable that can be run.

**Steps 6 & 7:** Testing and Debugging

---
## 2. Memory
### 2.1. Memory Types
  - **RAM (Random Access Memory):**
    - Read/write memory
    - High-speed memory used during program execution
    - Data is lost when power is turned off

  - **ROM (Read-Only Memory):**
    - Read-only
    - Data persists after power off

  - **Flash Memory:**
    - Non-volatile memory (data persists without power)
    - Slower than RAM
    - Can be erased and rewritten

### 2.2. Memory Layout

```c++
[High Address]  +-----------------------------+
                |            Stack            | => Local variables, Function calls, Return addresses, Grows downward:
                |                             |
                |                             |     void f(){ int val = 0};              ///< Local variable
                |                             |     ...
                |                             |
                |-----------------------------|
                v                             v
                            Free Space
                ^                             ^
                |-----------------------------|
                |             Heap            | => Dynamic memory new / malloc, Grows upward:
                |                             |
                |                             |     int val = new int;
                |                             |     delete val;
                +-----------------------------+
                |     Uninitialized Data      | => Global/static variables are not initialized or initialized with `0`:
                |           .bss              |
                |                             |     int global_val;                             ///< Global variable has initialized
                |                             |     int global_val_zr = 0;
                |                             |     void f(){ static int val};
                +-----------------------------+
                +-----------------------------+
                |      Initialized Data       | => Global/static variables are initialized with values:
                |           .data             |
                |                             |     int global_val = 100;                       ///< Global variable has initialized
                |                             |     void f(){ static int val = 1};              ///< Static variable has initialized
                +-----------------------------+
                +-----------------------------+
                |      Initialized Data       | => Global/static variables are initialized with values:
                |           .data             |
                |                             |     int global_val = 100;                       ///< Global variable has initialized
                |                             |     void f(){ static int val = 1};              ///< Static variable has initialized
                +-----------------------------+
                +-----------------------------+
                |          Text/Code          | => Read only data: 
                |           .text             |     
                |           .rodata           |     printf("Hello World");                      ///< String literals
                |                             |     const uint32_t BACCATE = 115200             ///< const global variables
                |                             |     __asm__ volatile("nop"); void f(){};        ///< Program instructions & Compiled machine code
[Low Address]   +-----------------------------+
```

### 2.3. Analyze
- **List all sections:**
    ```bash
    $ size ./build/cpp-lab # cpp
    $ arm-none-eabi-size firmware.elf #arm
    text    data     bss     dec     hex    filename
    14791     792     280   15863           ./build/cpp-lab
    12456     124    2048   14628    3924   firmware.elf
     # RAM usage = bss + data
     # Flash usage = text + data
    ```

  - `.text`: **Text/Code** Segment: the executable code size, including: **complied function, inline, template, constants, string literals**
  - `.data`: **Initialized Data Segment** 
    - stores initialized global and static variables. The initial values are stored in the `.bin` / `.hex` file during compilation
    - **copied from Flash to RAM at startup**
    - remain alive for the entire program lifetime.
  - `.bss`: **Uninitialized Data Segment**
    - Stores global and static variables that are not initialized or initialized with `0`.
    - Flash memory only stores the required size, not the actual zero values.
    - The system automatically initializes the `.bss` section to `0` during startup.
    - Helps reduce `.bin` / `.hex` file size and speed up start up.
  - **Stack Segment:**
    - Example: `main()` -> `function_A(int x)` -> `function_B(int y)` -> `function_C(int z)`

        ```cpp
            ┌─────────────────────────┐
            │ main()                  │
            ├─────────────────────────┤
            │ Return to main()        │
            │ function_A()            │
            │ x = 1                   │
            │ local_a = 10            │
            ├─────────────────────────┤
            │ Return to function_A()  │
            │ function_B()            │
            │ y = 2                   │
            │ local_b = 20            │
            ├─────────────────────────┤
            │ Return to function_B()  │
            │ function_C()            │
            │ z = 3                   │
            │ local_c = 30            │
            │ buffer[16]              │ <-> Stack Pointer (SP)
            └─────────────────────────┘
        ```

### 2.4. Optimize

```cpp
/// @brief Use bit-fields for flags
struct Status {
    uint8_t is_ready : 1;   // 1 bit
    uint8_t is_error : 1;   // 1 bit
    uint8_t mode     : 3;   // 3 bits (0-7)
    uint8_t reserved : 3;   // 3 bits
}; // Total: 1 byte

static_assert(sizeof(Status) == 1, "Status must be 1 byte");

/// @brief Memory layout (with padding)
struct StructNormal {
    uint32_t b;     // 4 bytes
    uint8_t a;      // 1 byte
    uint8_t c;      // 1 byte
}; // Total: 8 bytes (2 bytes padding)

static_assert(sizeof(StructNormal) == 8,
              "StructNormal must be 8 bytes");

/// @brief Packed structure (no padding)
#pragma pack(push, 1)
struct StructPacked {
    uint32_t b;     // 4 bytes
    uint8_t a;      // 1 byte
    uint8_t c;      // 1 byte
}; // Total: 6 bytes
#pragma pack(pop)

static_assert(sizeof(StructPacked) == 6,
              "StructPacked must be 6 bytes");

/// @brief Use const for read-only data, stored in Flash/ROM instead of RAM
const uint8_t gamma_table[256] = {
    /* read-only lookup table */
};
```

---
## 3. Setup Environment, IDE (Integrated Development Environment):
Installing a compiler that supports at least C++17: **GCC/G++7, Clang++ 8,...**
Some of the options typically does:
  - **Build:** compiles all modified code files in the project or workspace/solution, and then links the object files into an executable. If no code files have been modified since the last build, this option does nothing.
  - **Clean:** removes all cached objects and executables so the next time the project is built, all files will be recompiled and a new executable produced.
  - **Rebuild:** does a "clean" -> "build"
  - **Compile:** recompile a single code file (regardless of whether it has been cached previously). This option does not invoke the linker or produce an executable.
  - **Run/Start:** executes the executable from a prior build

- **Examples:**
  ```cpp
  // main.cpp
  #include <iostream>

  int main(){
  	std::cout << "Hello World";
  	return 0;	
  }
  ```
    
  ```bash
  $ ls
  main.cpp
  $ g++ --version
  $ g++ main.cpp -o main.o
  $ g++ main.o -o main.exe
  $ ./main.exe
  Hello world
  ```

---
## 4. Configuring The Compiler
### 4.1. Build Configurations
- It is a collection of project settings that determines how the project will be built.
- **Debug configurations**: for debugging, turns off all optimizations, larger and slow, but ease
- **Release configurations**: for releasing, optimized for size and performance.
- For gcc/clang, the `-0#` option is used to control optimize settings.

### 4.2. Compiler Extensions
- Many compilers implement their own changes to the language, often to enhance compatibility with other versions of the language
- For gcc/clang, the `-pedantic-errors` option is used to disable the compiler extension.

### 4.3. Warning/Error Level
- When compile the program, the compiler will check the rules of languages/compiler extension, and emit diagnostic messages.
- For gcc users: the `-Wall -Weffc++ -Wextra -Wconversion -Wsign-conversion` options is used to enable the warning levels.

### 4.4. Language  standard 
- `C++98, C++03, C++11, C++14, C++17, C++20, C++23,...`
- For gcc/g++/clang, the `-std=c++17` option is used to set the language standard.

---
## 5. Command Line
- Command line arguments are optional string arguments that are passed by the operating system to the program when it launch.
- Passing command line arguments: we simply list the command line arguments right after the executable name.
- Using command line arguments: by using different form of `main()`: `main(int argc, char* argv[])` / `main(int argc, char** argv)`
  - `argc`: argument count, always be at least 1, because the first argument **argv[0]** is always the name of the program itself.
  - `argv`: is where the actual argument values are stored (think: argv = argument values)
  - If an argument contains spaces, wrap it in quotes
  - Example:
    ```bash
    ./executable_app input.txt
    ./executable_app "hello world" 1 2 3
    ```

---
## 6. Performance
- **Things that can impact program performance:**
  - CPU speed
  - Memory usage
  - Disk and file access
  - Network speed
  - Compiler optimizations
  - Algorithm efficiency
  - Too much logging/output
  - Running too many tasks/threads
  - Debug vs release build
  - TBD

- **Measuring performance:**
  - Gather at least 3 test results.
  - Run the program long enough to get meaningful results (e.g. 10 seconds or more).
  - Use the same test conditions for each run.
  - Compare average execution times.
  - TBD

