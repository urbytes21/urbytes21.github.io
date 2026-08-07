---
author: "Phong Nguyen"
title: "CI/CD Notes"
date: "2026-03-12"
description: "CI/CI Notes"
tags: ["ci,cd,docker,git"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 10   # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
## 1. Introduction
**CI/CD** is a continuous method of software development, where you continuously build, test, deploy, and monitor iterative code changes.
- **CI - Continue Integration** used to make sure new code integrates cleanly with the existing codebase. (Can this code be safely merged?)
- **CD - Continue Delivery / Continues Deployment** used to handle releasing the software
  - **Continuous Delivery:** The system automatically prepares a release, but deployment to production requires manual approval.
  - **Continuous Deployment:** Everything is automatic.

**Example:**
```bash
**Developer pushes code**
        │
        v
      [ CI ]
        │
        ├─ Build #Compiled application
        ├─ Run tests #Test report
        ├─ Static analysis / linting #Code quality report
        └─ Notify results #Build status (Pass/Fail)
        │
        v
    **CI Passed**
        │
        ▼
      [ CD ]
        │
        ├─ Create release artifact #Firmware (.bin), package, installer, etc.
        ├─ Deploy to staging #Staging environment updated
        ├─ Manual approval (Continuous Delivery) #Deployment approval
        └─ Automatic deployment (Continuous Deployment) #Production environment updated
        │
        v
      **Release**
```
<br>

---
## 2. GitLab CI/CI
### Concepts:
- **Job** is an individual task within a pipeline, such as building the project, running tests, or deploying an application. For example, a job can compile or test code.
- **Pipeline** is an automated workflow consisting of multiple stages and jobs that build, test, and deploy code changes.
- **Runner** is an agent that picks up and executes the jobs, it can run on physical machines or virtual instances. 
- **Agent** is a program or service that performs tasks on behalf of another system. (`Gitlab Runner, Github Action Runner, Kubernetes Agent ...`)
- **Stages** define the order of execution. Typical stages might be `build`, `test`, and `deploy`.
- **Component** is a reuseable pipeline configuration unit to compose an entire pipeline configuration or a small part of a larger pipeline. 

### Step 1: Configure pipeline
- A `.gitlab-ci.yml` file at the root of the project, contains pipeline, and executes when the file runs on a runner.

- E.g: Create a .gitlab-ci.yml and commit to a repo
```yml
# .gitlab-ci.yml

stages: # List of stages for jobs, order of execution
  - build
  - test
  - deploy

build-job:
  stage: build # runs in the `build` stage
  script:
    - echo "Compiling code..."
    - sleep 10
    - echo "Build complete."

test-job-1:
  stage: test
  script:
    - echo "Running test 1..."
    - sleep 10
    - echo "Test code coverage 99%"

test-job-2:
  stage: test # can run that the same time as test-job-1
  script:
    - echo "Running test 2..."
    - sleep 10
    - echo "Test code coverage 99%"

deploy-job:
  stage: deploy
  environment: production
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```

### Step 2: Find/Create runners
- Register runners or use runners already registered for your GitLab Self-Managed instance.s
- Create a runner on your local machine.

### Step 3: Use CI/CD variables and expressions
- CD/CI variables includes custom variables and predefined variables

- CI/CD expression use the `$[[]]` to enable dynamic configuration based on different contexts:
  - Input context `$[[ inputs.INPUT_NAME ]]`: Access typed parameters passed into configuration files with include:inputs or when a new pipeline is run
  - Matrix context `$[[ matrix.IDENTIFIER ]]`:  Access matrix values in job dependencies to create 1:1 mappings between matrix jobs

### Step 4: User CI/CD Component
- Add a component to the pipeline configuration with `include:component`

---
## 2. Docker:
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
$ cat Dockerfile

# Define a new image from Ubuntu 2404
FROM ubuntu:24.04

# Install prerequisites
#  - the backslash \ is used for line continuation
#  - `-y` to automatically confirm installation
RUN \
    # updates the package lists for upgrades for packages that need upgrading,
    apt-get update && \
    apt-get install -y cmake && \
    apt-get install -y gcc g++  && \
    apt-get install -y cppcheck && \
    apt-get install -y clang-tidy && \
    apt-get install -y lcov

# Set the working directory inside the Docker image
WORKDIR /cpp-lab

# Copy all things from the build context (the directory where we run docker build) into the docker image being built
# Copy source code from host (build context) to image because the WORKDIR already set
# COPY . .
COPY . /cpp-lab
```

4. Getting Started
- Create, build and push your first image: https://docs.docker.com/get-started/introduction/build-and-push-first-image/