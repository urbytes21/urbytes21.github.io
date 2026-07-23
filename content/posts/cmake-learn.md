---
author: "Phong Nguyen"
title: "CMake Notes"
date: "2025-08-21"
description: "CMake Notes."
tags: ["cmake"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

## 1. Introduction
**CMake** is a tool used for meta-build system generator.
- We write high-level instructions in a `CMakeLists.txt` file (platform-agnostic).Then CMake generates build system files for the platform we choose:
  - On Linux/Unix: generates Makefile (for make) or build.ninja (for ninja).
  - On Windows: generates Visual Studio solutions (.sln).
  - On macOS: can generate Xcode projects.
<br>

**CMake Generators:** CMake support multiple build systems output a.k.a generators. Which generator is used can be controlled via `CMAKE_GENERATOR` or `cmake -G` option

**Single/Multi-Configuration Generators**: Software builds often have several variants e.g. Debug, Release, RelWithDebInfo, and MinSizeRel. To select build type:
- single-generator(ninja,make): via `cmake --DCMAKE_BUILD_TYPE=<config>`
- multi-generator(ninja-mul,vs): single &  `cmake --build --config `

---
**Usage Basics:**
```bash
# install cmake
$ sudo apt install cmake

# verify cmake
$ cmake --version
cmake version 3.23.5

# specific project root dir, which contains the root CMakeLists.txt, default current dir
$ cmake -S <dir> 

# specific build dir, default current dir
$ cmake -B <dir> 

# specific generator to generate the build system in <dir>, must delete <dir> when switch <gen>
$ cmake  -G "<gen>" -B <dir>

# run the build system in the build dir, single-config
$ cmake --build <dir>

# multi-configs
$ cmake --build <dir> --config <cfg>

# clean when reorganization
$ cmake --build build --clean-first
```
<br>

**Example:**
```bash
$ cd project
# project structure
$ tree
.
├── CMakeLists.txt
└── HelloWorld.cxx

1 directory, 2 files

$ cat HelloWorld.cxx
#include <cstdio>

int main() {
  std::printf("Hello World\n");
}

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.23)

project(Tutorial)

add_executable(hello)
target_sources(hello
  PRIVATE
    HelloWorld.cxx
)
#######################################################################
# Step 1: generate build system in folder 'build' using Unix Makefiles
$ cmake -G "Unix Makefiles" -B build

# Step 2: run the build system in the folder 'build'
$ cmake --build build 

# Step 3: run the executable
$ ./build/hello
Hello World
```

----
## 2. Getting Started
This section help us will be able to describe `executables`, `libraries`, `source` and `header files`, and the `linkage relationship` between them.

A **CMakeLists.txt** (list file-CML) will exist within any directory where we want to provide instructions to CMake on how to handle files, and operations local to that dir or sub-dir.

There are four backbone commands of most CMake usage:
   - `add_executable()` and `add_library()` for describing **output artifacts** the software project wants to produce
   - `target_sources()` for **associating input files** with their respective output artifacts 
   - `target_link_libraries()` for associating **output artifacts** with one another.

  
- Strongly recommend that the **project root CMakeLists.txt**
**Example:**
```cmake
# should always contain these two commands at the top/near
cmake_minium_required(VERSION 3.23)
project(MyProjectName)
```
<br>

### 2.1. Building an Executable
We need at least four commands:
- **cmake_minium_required(VERSION `<min>`)**
- **project(`<name>` VERSION <ver>)**
- **add_executable(`<name>`)**: create a target, we can now start associating properties with it like source files we want to build and link.
- **target_sources(`<target>`  {INTERFACE|PUBLIC|PRIVATE} `<source>`)**: add source to target
   
The scope keyword  for executable should always be **PRIVATE** 
  **Example:**
  ```cmake
  # Set the minimum required version of CMake to be 3.23
  cmake_minimum_required(VERSION 3.23)
  # Create a project named Tutorial
  project(Tutorial)

  # Add an executable target called Tutorial to the project
  add_executable(Tutorial)
  # Add the Tutorial/Tutorial.cxx source file to the Tutorial target
  target_sources(Tutorial
          PRIVATE
          Tutorial/Tutorial.cxx
  )
  ```
<br>

### 2.2. Building a Library
**Example:**
```cmake
cmake_minimum_required(VERSION 3.23)
project(example)

# target 1
add_executable(Tutorial1)
target_sources(Tutorial1
    PRIVATE
        Tutorial/Tutorial.cxx
)

# target 2
add_executable(Tutorial2)
target_sources(Tutorial2
    PRIVATE
        Tutorial/Tutorial.cxx
)
```
=> Both `Tutorial1` and `Tutorial2` compile the same source file:

- Use `add_library` to add a library to the project
- Use a `FILE_SET` to describe a collection of header files in the `target_source`
**Example:**
```cmake
# Add a library called MyLibrary
add_library(MyLibrary)

target_sources(MyLibrary
  PRIVATE    # private source files, target cannot see
    library_implementation.cxx

  PUBLIC    # public source files
    # FILE_SET myHeaders # name
    # TYPE HEADERS       # kind
    FILE_SET HEADERS # don't need to provide type
    BASE_DIRS          # base locations of the file
      include
    FILES              # list of files
      include/library_header.h



########################################################
    PRIVATE        # PRIVATE is for me
        FILE_SET internalOnlyHeaders
        TYPE HEADERS
        FILES
          InternalOnlyHeader.h
    
    INTERFACE    # INTERFACE is for others
      FILE_SET consumerOnlyHeaders
      TYPE HEADERS
      FILES
        ConsumerOnlyHeader.h
  
    PUBLIC       # PUBLIC is for all of us
      FILE_SET publicHeaders
      TYPE HEADERS
      FILES
        PublicHeader.h
)
```
- We now can include directive were `#include <MyLibrary/library_header.h>`

### 2.3. Linking Together Libraries and Executables
- Use `target_link_libraries()` to invoke linkers to combine targets/libs
```bash
# Add the MyLibrary library as a linked dependency
# to the Tutorial target
target_link_libraries(Tutorial
  PRIVATE
    MyLibrary
)

target_link_libraries(Tutorial2
  PRIVATE
    MyLibrary
)
```

### 2.4. Subdirectories
- Use `add_subdirectory(<subname>)` to incorporate the CLMs - CMakeLists.txt located in a subdirectory of the project.
- The relative paths used inside that subdirectory's CMakeLists.txt are interpreted relative to that subdirectory.

**Example:**
```cmake
$ cd TutorialProject
# project structure
$ tree
├── CMakeLists.txt
├── Tutorial/
│   ├── CMakeLists.txt
│   └── Tutorial.cxx
└── MathFunctions/
    ├── CMakeLists.txt
    ├── MathFunctions.cxx
    └── MathFunctions.h

$ cat CMakeLists.txt # root CMakeLists.txt
cmake_minimum_required(VERSION 3.23)

project(Tutorial)

# include subdirectories so CMake processes their CMakeLists.txt
add_subdirectory(MathFunctions)
add_subdirectory(Tutorial)

$ cat ./Tutorial/CMakeLists.txt
add_executable(Tutorial)

# add source file for this executable
# path is relative to this directory (Tutorial/)
target_sources(Tutorial
  PRIVATE
    Tutorial.cxx
)

# link the MathFunctions library to the executable
target_link_libraries(Tutorial
  PRIVATE
    MathFunctions
)

$ cat ./MathFunctions/CMakeLists.txt
# create a library target MathFunctions
add_library(MathFunctions)

target_sources(MathFunctions
  PRIVATE
    MathFunctions.cxx
  
  # expose header file to other targets that link this library
  PUBLIC
    FILE_SET HEADERS
    FILES
      MathFunctions.h
)
```

- `add_subdirectory(src_dir <bin_dir>)`
  - bin_dir: where CMake generates build files and store build artifacts

----
## 3. CMake Language
The only **fundamental types** in CMake are:
-  `String`: e.g. "abc"
-  `Lists`: e.g. "abc;xyz"
<br>

**Commands:**
- `set()` to create a variable as a name for string.
- `${var}` to access variable's value
- `message()` to print 
- `cmake -P CMakeLists.txt` to run the script mode (not intended to have build software)
  ```bash
  $ cat CMakeLists.txt
  set(MYVAR "HelloWorld")
  message(${MYVAR})

  $ cmake -P CMakeLists.txt
  HelloWorld
  ```

- `list(APPEND list "new_item")` for manipulating the lists
- `foreach(l IN LISTS lists)` to iterate over a list
  ```bash
  $ cat CMakeLists.txt
  set(MYVAR "HelloWorld")
  message(${MYVAR})

  # create a list
  set(lists "first;second;third") # or set(lists first second third)
  # manipulate the list
  list(APPEND lists "fourth")
  # print out
  message("This is my list: ${lists}")
  # iterate to print each item
  foreach(i IN LISTS lists)
          message("item: ${i}")
  endforeach()

  $ cmake -P CMakeLists.txt
  HelloWorld
  This is my list: first;second;third;fourth
  item: first
  item: second
  item: third
  item: fourth
  ```
- `macro(name args)` to create a macro
- `function(name args)` to create a function
  ```bash
  $ cat CMakeLists.txt
  # create a macro
  macro(myM arg)
          message("myM called with arg: ${arg}")
  endmacro()

  # create a function
  function(myF arg)
          message("myF called with arg:${arg}")
          myM(${arg})
  endfunction()

  # call function
  myF("hello")

  $ cmake -P CMakeLists.txt
  HelloWorld
  This is my list: first;second;third;fourth
  item: first
  item: second
  item: third
  item: fourth
  myF called with arg:hello
  myM called with arg: hello
  ```
- "True", "On", "Yes" as `true`
  ```bash
  $ cat ConditionalValue.cmake
  if(True)
    message("Constant Value: True")
  else()
    message("Constant Value: False")
  endif()

  if(ConditionalValue)
    message("Undefined Variable: True")
  else()
    message("Undefined Variable: False")
  endif()

  set(ConditionalValue True)

  if(ConditionalValue)
    message("Defined Variable: True")
  else()
    message("Defined Variable: False")
  endif()

  $ cmake -P ConditionalValue.cmake
  Constant Value: True
  Undefined Variable: False
  Defined Variable: True
  ```

- We can organize the project let functions and utilities live in their own `.cmake` files outside the project CMLs and separate from the rest of the build system.
- We use the `include()` command to incorporate these separate ones.
  ```bash
  include(module1.cmake)
  include(module2.cmake)
  ```

----
## 4. Configuration And Cache Variables
### 4.1. Cache and normal variables
A **CMake Cache variables** are globally visible variables and persistent vars stored in CMakeCache.
- They are mainly used to configure build options and allow users to customize the build.
  - `-D<var>:<type>=<value>`: create/update a cache entry from the command line.
  - `option()`: define a boolean cache variable and provide a default value.
  - `set(<var> <value> CACHE <type> <docstring>)`: create/update a cache variable, but it will not overwrite a value that was already set by the user via `-D`.
  - `set()` / `unset()`: create/remove a normal variable that temporarily shadows the cache variable.

**Example:**
```cmake
$ tree
.
├── CMakeLists.txt
├── CMakePresets.json
├── MathFunctions
│   ├── CMakeLists.txt
│   ├── MathFunctions.cxx
│   └── MathFunctions.h
└── Tutorial
    ├── CMakeLists.txt
    └── Tutorial.cxx

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.23)

project(Tutorial)

# Add a default ON option for a cache variable 
option(TUTORIAL_BUILD_UTILITIES "Build the Tutorial executable des" ON)

# Add a conditional statement around add_subdirectory(Tutorial)
if(TUTORIAL_BUILD_UTILITIES)
        message("Build the Tutorial executable")
        add_subdirectory(Tutorial)
endif()

add_subdirectory(MathFunctions)

$ cmake -B build -DTUTORIAL_BUILD_UTILITIES=false
$ cmake --build build
$ ls build
# only build MathFunctions
CMakeCache.txt  CMakeFiles  Makefile  MathFunctions  cmake_install.cmake

$ cat build/CMakeCache.txt
// Build the Tutorial executable des
TUTORIAL_BUILD_UTILITIES:BOOL=false
```
- `CMAKE_CXX_STANDARD`: C++ standard
<br>

### 4.2. CMakePresets.json
**CMake Presets** is a CMake's built-in way to define reuseable and flexible build configurations
- This mechanism let us store build configurations in a file
  - `CMakePresets.json`: for the project and tracked in source control
  - `CMakeUserPresets.json`: for local user config

- We previously running a long commands likes:
  ```bash
  cmake -B build -DEXAMPLE_FOO=Bar -DEXAMPLE_QUX=Baz
  ```
- We can now create a `CMakePresents.json` in the CML's folder.
  ```json
  {
      "version":4,
      "configurePresets":[
      {
          "name": "example-preset", // preset name
          // "binaryDir": "${sourceDir}/build" // set the build dir to skip -B flag
          "cacheVariables": { // var configs
              "EXAMPLE_FOO": "Bar",
              "EXAMPLE_QUX": "Baz"
          }
      }
      ]
  }

  // Template:
  {
    "version": 4,
    "configurePresets": [
      {
        "name": "example-preset",
        "cacheVariables": {
          "EXAMPLE_FOO": "Bar",
          "EXAMPLE_QUX": "Baz"
        }
      }
    ]
  }
  ```
then use the preset:
  ```bash
  cmake -B build --preset example-preset
  ```
----

## 5. CMake Target Commands
**A target command** is one which modifies the properties of the target it is applied to.
- There are several target commands
  - Common/Recommended: `target_compile_definitions()` `target_compile_features()` `target_link_libraries()` `target_sources()`
  - Advanced/Caution: `get_target_property()` `set_target_properties()` `target_compile_options()` `target_link_options()` `target_precompile_headers()`
  - Esoteric/Footguns: `target_include_directories()` `target_link_directories()`

### 5.1. set_target_property - get_target_properties
To give direct access to a target's properties by name
**Example:**
```cmake
$ cat CMakeLists.txt
add_library(mylib)
# set properties to the target
set_target_properties(mylib
        PROPERTIES
         Key Value
         Key1 Value1
)

# get properties from the target
get_property(KeyVar mylib Key)
get_property(KeyVar1 mylib Key1)

# print out
message("Key: ${KeyVar}")
message("Key1: ${KeyVar1}")

$ cmake -B build
Key: Value
Key1: Value1
```

### 5.2. target_precompile_headers
It creates a precompiled header (PCH) from a list of commonly included header files.
- Reduces compilation time by compiling these headers only once instead of for every source file.
- Most effective for large C++ projects where many source files include the same standard or third-party headers.
- Should contain stable headers that change infrequently (e.g., `<vector>`, `<string>`, `<memory>`, Qt, Boost, or other library headers).
- Avoid placing project headers that change often, as modifying them forces the PCH to be rebuilt.

**Example:**
  ```cmake
  target_precompile_headers(my_app PRIVATE
      <vector>
      <string>
      <memory>
      <unordered_map>
  )
  ```

### 5.3. target_compile_features - target_compile_definitions
These commands are using to communicate language standard and compile definition requirements for the target.
  - `Feature` command describes a minimum language standard as a target property.
  - `Definition` command describes compile definitions as target properties.
**Example:**
```cmake
target_compile_features(MyTarget PRIVATE cxx_std_20) # std20

target_compile_definitions(MyTarget PRIVATE MY_DEFINITION)
# In the source code, the definition can be checked using the preprocessor:
# #ifdef MY_DEFINITION
# // do something
# #endif
```

### 5.4. target_compile_options - target_link_options
These two commands are using to specific the options being passed on the compile and link line.
**Example:**
```cmake
# Enable warnings when compiling the code
if(
  (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC") OR
  (CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
)
  target_compile_options(Tutorial PRIVATE /W3) # default recommended level

elseif(
  (CMAKE_CXX_COMPILER_ID STREQUAL "GNU") OR
  (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
)
  target_compile_options(Tutorial PRIVATE -Wall)  # enable common warnings

endif()
```
 
### 5.4. target_link_directories - target_include_directories
These two commands specify directories used during compilation and linking, and map to the `-L` (.a, .so, .lib, .dll) and `-I` (.h, .hpp) compiler flags, typically used when integrating a precompiled or vendored library into a project.
**Example:**
```cmake
Vendor$ tree
.
├── CMakeLists.txt
├── include
│   └── Vendor.h
└── lib
    ├── Vendor.cxx
    ├── Vendor.o
    └── libVendor.a    # static library , g++ -c Vendor.cxx & ar rvs libVendor.a Vendor.o

$ cat CMakeLists.txt
add_library(VendorLib INTERFACE) # INTERFACE library, only describes how to use, does not build any thing

# Now target that links VendorLib will automatically compile with:
# #define TUTORIAL_USE_VENDORLIB
target_compile_definitions(VendorLib
    INTERFACE
    TUTORIAL_USE_VENDORLIB
)

# Add the include directory to VendorLib to tell the compiler where the headers are.
# Now any consumer target can write:
# #include <vendor.h>
target_include_directories(VendorLib
        INTERFACE
         include
)

# Add the lib directory to VendorLib to tells the linker where to search for libraries.
target_link_directories(VendorLib
        INTERFACE
         lib
)

#  Add the Vendor archive to VendorLib to tell CMake that anything linking VendorLib must also link Vendor.
target_link_libraries(VendorLib
        INTERFACE
         Vendor
)
```

----

## 6. CMake Library Concepts
There is an optional argument in `add_library(<name> <type>)` command, it's <type>
  - `STATIC`: an archive of object files for use when linking other targets. (.a)
  - `SHARED`: a dynamic library that maybe linked by other targets and loaded at runtime. (.so)
  - `MODULE`
  - `OBJECT`
  - `INTERFACE`: a library target which specifies usage requirements for dependents but does not compile sources and does not produce a library artifact on disk.
  
### 6.1. Static and Shared
When not given a type, `add_library` will create either a **STATIC** or **SHARED** library depending on the `BUILD_SHARED_LIBS`
**Example:**
```cmake
add_library(MyLib-static STATIC)
add_library(MyLib-shared SHARED)

# Depends on BUILD_SHARED_LIBS, true = shared
add_library(MyLib)
```

### 6.2 Interface Libraries
**Interface libraries** are those which only communicate usage requirements for other targets, they do not produce any artifacts.
- It's use to create a header-only library, providing the necessary flags for the executable.
**Example:**
```cmake
$ cat MathFunctions/MathLogger/CMakeLists.txt¶
add_library(MathLogger INTERFACE)

target_sources(MathLogger
  INTERFACE
    FILE_SET HEADERS
)

$ cat MathFunctions/MathFunctions.cxx
add_subdirectory(MathLogger)

$ cat MathFunctions/file.cpp
#include <MathLogger.h>
```

### 6.3. Object Libraries
[TBD](https://cmake.org/cmake/help/latest/guide/tutorial/In-Depth%20CMake%20Library%20Concepts.html)

----

## 7. System Introspection
CMake provides modules to simplify checks.
- `CheckIncludeFiles`: check one ore more C/C++ header files
- `CheckCompileFlag`: check whether compiler supports a given flag
- `CheckSourceCompiles`: check whether source code can be built for a given language
- `CheckIPOSupported`: interprocedural optimization

**Example:**
```cmake
include(CheckIncludeFiles)
check_include_files(emmintrin.h HAS_EMMINTRIN LANGUAGE CXX)

if(HAS_EMMINTRIN)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_SSE2)
endif()

$ cat test.cpp
#ifdef TUTORIAL_USE_SSE2
#  include <emmintrin.h>
#endif
```

----

## 8. Custom Commands and Generated Files
- `add_custom_command()` for a code generator within the project, to add a custom build rule to the generated build system
- `add_dependencies()`: Add a dependency between top-level targets.
- `add_custom_target()`: Add a target with no output so it will always be built.
- `CMAKE_CURRENT_BINARY_DIR`: The path to the binary directory currently being processed.

**Example:**
```cmake
$ tree
.
├── CMakeLists.txt
├── MakeTable
│   ├── CMakeLists.txt
│   └── MakeTable.cxx

$ cat MakeTable/MakeTable.cxx
# // A simple program that builds a sqrt table
# #include <cmath>
# #include <fstream>
# #include <iostream>

# int main(int argc, char* argv[])
# {
#   // make sure we have enough arguments
#   if (argc < 2) { # arg[0] program names\
#     return 1;
#   }

#   std::ofstream fout(argv[1], std::ios_base::out);
#   bool const fileOpen = fout.is_open();
#   if (fileOpen) {
#     fout << "double sqrtTable[] = {" << std::endl;
#     for (int i = 0; i < 10; ++i) {
#       fout << sqrt(static_cast<double>(i)) << "," << std::endl;
#     }
#     // close the table with a zero
#     fout << "0};" << std::endl;
#     fout.close();
#   }
#   return fileOpen ? 0 : 1; // return 0 if wrote the file
# }

$ cat MakeTable/CMakeLists.txt
# Add a MakeTable executable
add_executable(MakeTable)

# Add MakeTable.cxx to the MakeTable executable
target_sources(MakeTable
  PRIVATE
    MakeTable.cxx
)

##############################################################
# Add a custom command which invokes MakeTable to generate SqrtTable.h
# MakeTable program → generates SqrtTable.h
add_custom_command(
  OUTPUT SqrtTable.h
  COMMAND MakeTable SqrtTable.h
  DEPENDS MakeTable
  VERBATIM
)

# Add a custom target which depends on SqrtTable.h
# If SqrtTable.h doesn't exist, the custom command runs.
add_custom_target(RunMakeTable DEPENDS SqrtTable.h)

##############################################################
# Add an INTERFACE library to describe the SqrtTable header
add_library(SqrtTable INTERFACE)

# Add the current binary directory (and optionally the SqrtTable.h FILE)
#        to a header file set of the interface library
target_sources(SqrtTable
  INTERFACE
    FILE_SET HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_BINARY_DIR}
    FILES
      ${CMAKE_CURRENT_BINARY_DIR}/SqrtTable.h
)

##############################################################
# Use add_dependencies to ensure the custom target always runs before
#        targets that depend on the interface library
add_dependencies(SqrtTable RunMakeTable)


$ cmake --build build
[ 12%] Building CXX object MakeTable/CMakeFiles/MakeTable.dir/MakeTable.o
[ 25%] Linking CXX executable MakeTable
[ 25%] Built target MakeTable
[ 37%] Generating SqrtTable.h

$ cat build/MakeTable/SqrtTable.h
double sqrtTable[] = {
0,
1,
1.41421,
1.73205,
2,
2.23607,
2.44949,
2.64575,
2.82843,
3,
0};
```

----

## 9. Testing and CTest
CTest is a task launcher which runs commands and reports if they have returned value.
- `enable_testing()`: Enables testing for the current directory
- `add_test()`: Add a test to the project to be run by `ctest`.
- `ctest --test-dir build`: To run ctest directly on the build dir with all available tests.
- `ctest --test-dir build -R specific_test`: with regular expressions

**Example:**
```cmake
$ cat CMakeLists.txt
option(BUILD_TESTING "Enable testing and build tests" ON)
if(BUILD_TESTING)
  enable_testing()    # enable test
  add_subdirectory(Tests) # add tests CLM
endif()

$ cat Tests/CMakeLists.txt
add_executable(TestMathFunctions)

target_sources(TestMathFunctions
  PRIVATE
    TestMathFunctions.cxx    # test source
)

target_link_libraries(TestMathFunctions
  PRIVATE
    MathFunctions # source
)

function(MathFunctionTest op)
  add_test(    # add test
    NAME ${op}    
    COMMAND TestMathFunctions ${op}
  )
endfunction()

MathFunctionTest(add)
MathFunctionTest(mul)
MathFunctionTest(sqrt)
MathFunctionTest(sub)

$ ctest --test-dir build # run all tests
$ ctest --test-dir build -R sqrt # run sqrt test 
```

## 10. Installation Commands and Concepts
[TBD](https://cmake.org/cmake/help/latest/guide/tutorial/Installation%20Commands%20and%20Concepts.html)

---
## 11. Finding Dependencies
CMake provides an extensive toolset for discovering and validating dependencies of different kinds.
- `find_package()` to import dependencies into the project.
  - <version> arg
  - `REQUIRED`: for non-optional dep which should abort  the build if not found
  - `QUIET` for optional dep
  - We ensure `find_package()` can discover <package name> by adding the install tree to `CMAKE_PREFIX_PATH`.
    ```json
    // /CMakePresets.json
    "cacheVariables": {
      "CMAKE_PREFIX_PATH": "${sourceParentDir}/install",
    }
    ```
- `find_file()`: Finds and reports the full path to a named file, this tends to be the most flexible of the find commands.
- `find_library()`: Finds and reports the full path to a static archive or shared object suitable for use with target_link_libraries().
- `find_path()`: Finds and reports the full path to a directory containing a file
- `find_program()`: Finds and reports and invocable name or path for a program. Often used in combination with execute_process() or add_custom_command().

[TBD](https://cmake.org/cmake/help/latest/guide/tutorial/Installation%20Commands%20and%20Concepts.html)

----

## 12. Miscellaneous Features
### 12.1. Target Aliases
- Creates an Alias Target, such that <name> can be used to refer to <target>
```bash
add_library(MyLib INTERFACE)
add_library(MyProject::MyLib ALIAS MyLib)

add_executable(mytool main.cpp)
add_executable(Tutorial::mytool ALIAS mytool)
```

### 12.2 Generator Expressions
- `target_compile_definitions(MyApp PRIVATE "MYAPP_BUILD_CONFIG=$<CONFIG>")`
- `$<CONFIG>` is a CMake Generator Expression:
  - Debug
  - Release
  - RelWithDebInfo
  - MinSizeRel

### 12.3 Download and build package 
- **Via FetchContent:**
```bash
include(FetchContent)

FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/03597a01ee50ed33e9dfd640b249b4be3799d395.zip
)

FetchContent_MakeAvailable(googletest)

# Download GoogleTest
# Extract the archive
# Run its CMakeLists.txt
# Add it to your build using `add_subdirectory()`
```

- **Install Globally:**
```bash
$ sudo apt install googletest # Problem: version mismatch.
```
 
- **Git Submodules:**
```bash
$ git submodule add googletest # Problem: messy repo management.
```

----
