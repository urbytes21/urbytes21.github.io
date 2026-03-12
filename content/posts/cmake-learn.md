---
author: "Phong Nguyen"
title: "CMake Guide"
date: "2025-08-21"
description: "CMake Learning."
tags: ["cmake"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

## 1. Introduction
- It's a meta-build system generator.
- How it works:
    - We write high-level instructions in a `CMakeLists.txt` file (platform-agnostic).Then CMake generates build system files for the platform we choose:
      - On Linux/Unix: generates Makefile (for make) or build.ninja (for ninja).
      - On Windows: generates Visual Studio solutions (.sln).
      - On macOS: can generate Xcode projects.

- **CMake Generators:** CMake support multiple build systems output a.k.a generators. Which generator is used can be controlled via `CMAKE_GENERATOR` or `cmake -G` option
- **Single/Multi-Configuration Generators**: Software builds often have several variants e.g. Debug, Release, RelWithDebInfo, and MinSizeRel. To select build type:
  - single-generator(ninja,make): via `cmake --DCMAKE_BUILD_TYPE=<config>`
  - multi-generator(ninja-mul,vs): single &  `cmake --build --config `

- **Usage Basics:**
```bash
# install cmake
$ sudo apt install cmake

# verify cmake
$ cmake --version
cmake version 3.23.5

# specific project root dir, which contains the root CMakeLists.txt
# default current dir
$ cmake -S <dir> 

# specific build dir
# default current dir
$ cmake -B <dir> 

# specific generator to generate the build system in <dir>
# must delete <dir> when switch <gen>
$ cmake  -G "<gen>" -B <dir>

# run the build system in the build dir
# single-config
$ cmake --build <dir>

# multi-configs
$ cmake --build <dir> --config <cfg>

# clean when reorganization
$ cmake --build build --clean-first
```

- **Example:**
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

int main()
{
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

## 2. Getting Started
- This section help us will be able to describe executables, libraries, source and header files, and the linkage relationship between them.

- A `CMakeLists.txt` (list file-CML) will exist within any directory where we want to provide instructions to CMake on how to handle files, and operations local to that dir or sub-dir.
- There are four backbone commands of most CMake usage:
    - `add_executable() and add_library()` for describing output artifacts the software project wants to produce
    - `target_sources()` for associating input files with their respective output artifacts 
    - `target_link_libraries()` for associating output artifacts with one another.

  
- Strongly recommend that the **project root CMakeLists.txt**
```bash
$ cat CMakeLists.txt

# should always contain these two commands at the top/near
cmake_minium_required(VERSION 3.23)
project(MyProjectName)
```

### 2.1. Building an Executable
- We need at least four commands:
  - **cmake_minium_required(VERSION <min>)**
  - **project(<name> VERSION <ver>)**
  - **add_executable(<name>)**: create a target, we can now start associating properties with it like source files we want to build and link.
  - **target_sources(<target>  {INTERFACE|PUBLIC|PRIVATE} <source>)**: add source to target 
  
- The scope keyword  for executable should always be **PRIVATE** 
- e.g.
```bash
$ cat CMakeLists.txt
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


### 2.2. Building a Library
- For example:
```bash
$ cat CMakeLists.txt
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
=> Both Tutorial1 and Tutorial2 compile the same source file:

- Use `add_library` to add a library to the project
- Use a `FILE_SET` to describe a collection of header files in the `target_source`
```bash
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
- Use add_subdirectory(<subname>) to incorporate the CLMs - CMakeLists.txt located in a subdirectory of the project.
- The relative paths used inside that subdirectory’s CMakeLists.txt are interpreted relative to that subdirectory.

- e.g
```bash
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

## 3. CMake Language
- The only fundamental types in CMake are:
  -  `String`: e.g. "abc"
  -  `Lists`: e.g. "abc;xyz"
- `set()` command to create a variable as a name for string.
- `${var}` to access variable's value
- `message()` command to print 
- `cmake -P CMakeLists.txt` to run the script mode (not intended to have build software)

e.g.
```bash
$ cat CMakeLists.txt
set(MYVAR "HelloWorld")
message(${MYVAR})

$ cmake -P CMakeLists.txt
HelloWorld
```

- `list(APPEND list "new_item")` for manipulating the lists
- `foreach(l IN LISTS lists)` to iterate over a list
e.g.
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
e.g.
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
e.g.
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

## 4. Docker:
1.  Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker's methodologies for shipping, testing, and deploying code, you can significantly reduce the delay between writing code and running it in production.

2. Install
docs.docker.com

3. Docker Architectures:
What is an image/Dockerfile?
- Without going too deep yet, think of a container image as a single package that contains everything needed to run a process. In this case, it will contain a Node environment, the backend code, and the compiled React code.
- Any machine that runs a container using the image, will then be able to run the application as it was built without needing anything else pre-installed on the machine.
- A Dockerfile is a text-based script that provides the instruction set on how to build the image. For this quick start, the repository already contains the Dockerfile.

3.1. Images:
- An image is a read-only template with instructions for creating a Docker container. 
3.2. Containers: 
- A container is a runnable instance of an image.
- https://www.docker.com/resources/what-container/	
- Containers and virtual machines have similar resource isolation and allocation benefits, but function differently because containers virtualize the operating system instead of hardware. Containers are more portable and efficient.

3.3. Docker registries
- A Docker registry stores Docker images. Docker Hub is a public registry that anyone can use, and Docker looks for images on Docker Hub by default. You can even run your own private registry.
- When you use the docker pull or docker run commands, Docker pulls the required images from your configured registry. When you use the docker push command, Docker pushes your image to your configured registry.

3.4. Container images
If you’re new to container images, think of them as a standardized package that contains everything needed to run an application, including its files, configuration, and dependencies. These packages can then be distributed and shared with others.

3.5. Docker Hub
To share your Docker images, you need a place to store them. This is where registries come in. While there are many registries, Docker Hub is the default and go-to registry for images. Docker Hub provides both a place for you to store your own images and to find images from others to either run or use as the bases for your own images.

3.6. Docker Instructions
- Note:
  - The build context is the set of files and folders on your host machine that Docker sends to the Docker daemon when you run:
  - The dot (.) at the end of this command tells Docker:        “Use the current directory as the build context.”, the folder contain `Dockerfile`
```bash
# Create a new build stage from a base image
FROM



```

4. Getting Started
- Create, build and push your first image: https://docs.docker.com/get-started/introduction/build-and-push-first-image/

## 5. GoogleTest
- Getting Started:
https://google.github.io/googletest/quickstart-cmake.html
my_project$ cmake -S . -B build
my_project$ cmake --build build
my_project$ cd build && ctest

- References:
https://google.github.io/googletest/samples.html