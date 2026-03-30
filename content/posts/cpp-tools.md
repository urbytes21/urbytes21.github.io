---
author: "Phong Nguyen"
title: "C/C++ Tools"
date: "2026-03-30"
description: "Useful Tools For C/C++ Development "
tags: ["tool,c,cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 10    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

## 1. Doxygen
- Code documentation can be achieved with the Doxygen framework which is one of the most standard one for many languages. 

### 1.1. Getting Started
- Install:
    ```bash
    $ sudo apt install doxygen
    ```
- Create a config and update:
    ```bash
    $ mkdir docs && cd docs
    $ doxygen -g Doxyfile
    $ vi Doxyfile
    ```
- Running doxygen:
    ```bash
    $ cd docs && doxygen <config-file>
    & htlm/index.htlm
    ```
- Install the VSCode Extension: `Doxygen Documentation Generator`

### 1.2. Doxygen with CMake
- Create a Doxygen file template for CMake:
    ```bash
    $ cd docs
    $ vi Doxyfile.in
    # Project settings
    PROJECT_NAME           = "Hello"
    PROJECT_NUMBER         = 1.0
    PROJECT_BRIEF          = "An Hello World library"
    
    # Input and output settings
    INPUT            = @CMAKE_CURRENT_SOURCE_DIR@/src/     \
                       @CMAKE_CURRENT_SOURCE_DIR@/include/ \
                       @CMAKE_CURRENT_SOURCE_DIR@/docs
    OUTPUT_DIRECTORY = @CMAKE_CURRENT_BINARY_DIR@/docs/
    
    # Doxygen settings
    DOXYFILE_ENCODING     = UTF-8
    MARKDOWN_SUPPORT      = YES
    OPTIMIZE_OUTPUT_FOR_C = YES
    FILE_PATTERNS         = *.c *.cxx *.cpp *.h *.hpp *.md
    RECURSIVE             = YES
    
    # Path settings
    STRIP_FROM_PATH       = @CMAKE_CURRENT_SOURCE_DIR@
    STRIP_FROM_INC_PATH   = @CMAKE_CURRENT_SOURCE_DIR@
    FULL_PATH_NAMES       = NO
    
    # Code and preprocessor settings
    EXTRACT_ALL           = YES
    EXTRACT_PRIVATE       = YES
    EXTRACT_STATIC        = YES
    
    ENABLE_PREPROCESSING   = YES
    SEARCH_INCLUDES        = YES
    INCLUDE_PATH           = @CMAKE_CURRENT_SOURCE_DIR@/include
    INCLUDE_FILE_PATTERNS  = *.h *.h *.hpp
    
    # Source browser settings
    SOURCE_BROWSER         = YES
    INLINE_SOURCES         = YES
    REFERENCES_RELATION    = YES
    REFERENCES_LINK_SOURCE = YES
    SOURCE_TOOLTIPS        = YES
    
    # HTML documentation
    GENERATE_HTML = YES
    
    # Disable other formats
    GENERATE_LATEX = NO
    GENERATE_RTF = NO
    
    # Diagram settings
    HAVE_DOT               = YES
    
    INCLUDE_GRAPH          = YES
    INCLUDED_BY_GRAPH      = YES
    CALL_GRAPH             = YES
    CALLER_GRAPH           = YES
    GROUP_GRAPHS           = YES
    
    GRAPHICAL_HIERARCHY    = YES
    DIRECTORY_GRAPH        = YES
    ```    

- Create a home page:
    ```bash
    $ cd docs
    $ vi main_page.md
    # Hello Project {#mainpage}
    
    This is a simple project to demonstrate the use of the `Hello` module.
    It contains a single function that prints "Hello, World!" to the console.
    
    The `main()` function in the `src/hello.c` file.
    ```

- Update root CLM:

```bash
$ vi CMakeLists.txt
# Documentation build as an option (default: ON)
option(BUILD_DOC "Build documentation" ON)

# check if Doxygen is installed
find_package(Doxygen)
if (DOXYGEN_FOUND)
    # set input and output files
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/docs/Doxyfile.in)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

    # request to configure the file
    configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
    message("Doxygen build started")

    # Option ALL allows to build the docs together with the application
    add_custom_target( docs ALL
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM )
else (DOXYGEN_FOUND)
  message("Doxygen is required to generate the documentation")
endif (DOXYGEN_FOUND)
```
- Create a dummy source file:
```bash
$ vi src/Test.h
#ifndef UTILS_H
#define UTILS_H

/**
 * @brief Returns the maximum of two integers.
 *
 * @param a First integer.
 * @param b Second integer.
 * @return The maximum of a and b.
 */
int max (int a, int b);

/**
 * @brief Prints "Hello World!" a specified number of times.
 *
 * @param count The number of times to print "Hello World!".
 */
void print_hello (int count);

#endif
```

- Validate:
```bash
$ cmake -B build && cmake --build build
$ firefox  ./build/docs/html/index.html
```

---
## 2. GDB

- Useful cmd:
   - `layout src`: Displays the source window only.
   - `layout split`: Displays both the source and assembly windows.
   - `ref` or `ctrl + L` to refresh
   - `next`: next step
   - `step` 

```bash
$ gcc -o main main.c -s # build without debug info & symbol
$ wc -c main # word count

$ gdb ./main
lay next # switch to TUI mode
tui disable # quit TUI mode (or ctrl X + A to switch between mode)
quit # quit gdb

$ gcc -o main main.c -g # build for debug infos
$ gdb ./main
lay next
layout src
layout split
break main
break <line_num>
run # run program
next # step over
step # step in
finish # step out
info locals # local variables
print <variable> # print expression
```

---
## 3. Debug in VSCode 
- Install `C/C++ Extension Pack`
- Create a launch configuration for debug in `launch.json`:
  ```json
  {
      "version": "0.2.0",
      "configurations": [
          {
              "name": "(gdb) Launch",
              "type": "cppdbg",
              "request": "launch",
              "program": "${input:programPath}",
              "args": [],
              "stopAtEntry": true,
              "cwd": "${workspaceFolder}",
              "environment": [],
              "externalConsole": false,
              "MIMode": "gdb",
              "preLaunchTask": "Build Debug Task",
              "setupCommands": [
                  {
                      "description": "Enable pretty-printing for gdb",
                      "text": "-enable-pretty-printing",
                      "ignoreFailures": true
                  },
                  {
                      "description": "Set Disassembly Flavor to Intel",
                      "text": "-gdb-set disassembly-flavor intel",
                      "ignoreFailures": true
                  }
              ]
          },
      ],
      "inputs": [
          {
              "id": "programPath",
              "type": "promptString",
              "description": "Path to executable",
              "default": "${workspaceFolder}/build/"
          }
      ]
  }
  ```

- Create task for the build/re-build in `tasks.json`:
    ```json
    {
    	"version": "2.0.0",
    	"tasks": [
    		{
    			"label": "Build Debug Task",
    			"type": "shell",
    			"command": "${workspaceFolder}/make.sh",
    			"options": {
    				"cwd": "${workspaceFolder}"
    			},
    			"group": {
    				"kind": "build",
    				"isDefault": true
    			},
    			"problemMatcher": []
    		}
    	]
    }
    ```
- Run Debug: `Ctrl + F5`
    
---
## 4. Other