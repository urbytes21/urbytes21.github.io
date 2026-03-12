---
author: "Phong Nguyen"
title: "Java JNA"
date: "2024-11-21"
description: "Java: JNA."
tags: ["java"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 100    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Explain how to use the Java Native Access library (JRA) to access native libraries.
**References:** 
[Using JNA to Access Native Dynamic Libraries](https://www.baeldung.com/java-jna-dynamic-libraries)<br>
## 1. Introduction
- Sometimes we need to use native code to implement some functionality:
  - Reusing legacy code written in C/C++ or any other language able to create native code.
  - Accessing system-specific functionality not available in the standard Java runtime.
- Trace-offs:
  - Can't directly use static libraries
  - Slower when compared to handcrafted JNI code.
=> JNA today is probably the best available choice to access native code from Java.

## 2. JNA Project Setup
- Add JNA dependency to the project's pom.xml ([latest version of jna-platform](https://mvnrepository.com/artifact/net.java.dev.jna/jna-platform))
```xml
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna-platform</artifactId>
    <version>5.6.0</version>
</dependency>
```
## 3. Using JNA
- **Step1**: Create a **Java interface** that *extends* **JNA's Library interface** to describe the methods and types used when calling the target native code.
- **Step2**: Pass this interface to JNA which returns a concrete implementation of this interface that we use to invoke native methods.<br>
  
### 3.1. Example 1: Calling methods from the C Standard Library

1. CMath.java
```java

import com.sun.jna.Library;
import com.sun.jna.Native;
import com.sun.jna.Platform;

public interface CMath extends Library {
    CMath INSTANCE = Native.load(Platform.isWindows() ? "msvcrt" : "c", CMath.class);
    double cosh(double value);
}

```
- load() method takes two arguments, the return a concrete implementation of this interface, allowing to call any of its methods.
  - The dynamic lib name
  - The Java interface describing the methods that we'll use
-  We don't have to add the .so (Linux) or .dll(Window) extension as they'are implied. Also, for Linux-based systems, we don't need to specific the `lib` prefix.

1. Main.java
```java
public class Main {

	public static void main(String[] args) {
		System.out.println(CMath.INSTANCE.cosh(180));
	}

}
```

### 3.2. Example 2: Basic types Mapping
```java
char => byte
short => short
wchar_t => char
int => int
long => com.sun.jna.NativeLong
long long => long
float => float
double => double
char * => String
```
- JNA provides the NativeLong type, which uses the proper type depending on the system'architecture.

### 3.3. Structures and Unions
- Given this C struct and union
```c
struct foo_t {
    int field1;
    int field2;
    char *field3;
};

union foo_u {
    String foo;
    double bar;
};

```
- Java peer class would be
```java
@FieldOrder({"field1","field2","field3"})
public class FooType extends Structure {
    int field1;
    int field2;
    String field3;
};

public class MyUnion extends Union {
    public String foo;
    public double bar;
};

MyUnion u = new MyUnion();
u.foo = "test";
u.setType(String.class);
lib.some_method(u);

```
- Java requires the @FiledOrder annotation so it can properly serialize data into a memory buffer before using it as an argument to the target method.



