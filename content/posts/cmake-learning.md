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
weight: 2    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
## 1. Introduction
- It's a meta-build system generator.
- How it works:
    - We write high-level instructions in a CMakeLists.txt file (platform-agnostic).Then CMake generates build system files for the platform we choose:
    - On Linux/Unix → generates Makefile (for make) or build.ninja (for ninja).
    - On Windows → generates Visual Studio solutions (.sln).
    - On macOS → can generate Xcode projects.

## 2. Setup
- Linux: Install: `sudo apt install cmake` , Verify: `cmake --version` 

## 3. How to work with CMake

1. Create CMakeLists.txt and resources file
2. Create and cd to `project-build` folder & Run `cmake <dir-contain-CMakeLists.txt>` (`cmake ../`) to set up & generate build system.
3. Run `cmake --build <dir-contain-buildsystem>` ( `cmake --build ./`) to actually build/compile the project.
4. Run the target build (e.g. `./targetName`)

### 3.1. 
> CMakeLists.txt
```makefile
cmake_minimum_required(VERSION 3.0)	// specifying a minimum CMake version
project(CmakeprojectName)	// set the project name
add_executeable(targetName "headerfile.h" "sourcefile1.cpp" "sourcefile2.cpp") // specified source code files
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