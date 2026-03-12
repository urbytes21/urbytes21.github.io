---
author: "Phong Nguyen"
title: "Java Platform Standard Edition"
date: "2024-11-12"
description: "Basic Java Platform Standard Edition."
tags: ["java"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 100    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Java is a programming language created by James Gosling from Sun Microsystems (Sun) in 1991. Java allows to write a program and run it on multiple operating systems.
**References:** 
[Java Platform Standard Edition 8 Documentation](https://docs.oracle.com/javase/8/docs/)<br>
[Introduction to Java programming - Tutorial](https://www.vogella.com/tutorials/JavaIntroduction/article.html#introduction-to-java)<br>
## 1. Introduction to Java
Oracle has two products implement **Java SE**:
- **JDK** (Java SE Development Kit): is a superset of JRE.
- **JRE** (Java SE Runtime Environment): provides the libraries, the Java Virtual Machine (JVM) and other components.
![image](/images/java_concept_diagram.png)<br>

### 1.1. Development Process with Java
- Source code is first written in `.java` file.
- Those source files are then compiled into `.class` files by the Java compiler( `javac`).
- A `.class` file contains `bytecodes`(the machine language of the Java VM).
- The `.class` file will be run by Java tool as an instance of the Java VM. 
- The Java virtual machine (JVM) is a software implementation of a computer that executes programs like a real machine.
![image](/images/overview_java_process.png)<br>

Because the Java VM is available on many different OS, the same `.class` files are capable of running on many OS too.
![image](/images/java_capable_running_os.png)<br>

### 1.2.  Comparison of the C and Java Compilation & Linking Processes
![image](/images/java_c_processes.png)<br>
![image](/images/java_processes.png)<br>

### 1.3.  Garbage collector
- The Java runtime automatically frees the memory which is not referred to by other objects.
- The Java garbage collector checks all object references and finds the objects which can be automatically released.

### 1.4.  Classpath
- The `classpath` defines where the Java compiler and Java runtime look for `.class` files to load.
- If we want to use an external Java library we have to add this library to our `classpath`

To use the classpath, type the following command. Replace "mydirectory" with the directory which contains the test directory(`.class` file etc).
```
java -classpath "mydirectory" HelloWorld
```
<br>

## 2. Installation
1. To run Java programs, we must have `Java Runtime Environment (JRE)`
```
>java --version
java 21.0.3 2024-04-16 LTS
Java(TM) SE Runtime Environment (build 21.0.3+7-LTS-152)
Java HotSpot(TM) 64-Bit Server VM (build 21.0.3+7-LTS-152, mixed mode, sharing)
```
2. Write a source file named `HelloWorld.java`
```java
// a small Java program in HelloWord.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

3. Compile a Java source file in to a class file 
```javac HelloWorld.java```
4. Start the Java program and we should see "Hello Word" on the command line:
```
C:\Users\phong.nguyen-van\Desktop>java HelloWorld
Hello World
```

<br>


## 3. Trail: Learning the Java Language
This trail covers the fundamentals of programming in the Java programming language.
1. [**Object-Oriented Programming Concepts**](https://docs.oracle.com/javase/tutorial/java/concepts/index.html) teaches you the core concepts behind object-oriented programming: objects, messages, classes, and inheritance. This lesson ends by showing you how these concepts translate into code. Feel free to skip this lesson if you are already familiar with object-oriented programming.
2. [**Language Basics**](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html) describes the traditional features of the language, including variables, arrays, data types, operators, and control flow.
3. [**Classes and Objects**](https://docs.oracle.com/javase/tutorial/java/javaOO/index.html) describes how to write the classes from which objects are created, and how to create and use the objects.
4. [**Annotations**](https://docs.oracle.com/javase/tutorial/java/annotations/index.html) are a form of metadata and provide information for the compiler. This lesson describes where and how to use annotations in a program effectively.
5. [**Interfaces and Inheritance**](https://docs.oracle.com/javase/tutorial/java/IandI/index.html) describes interfaces—what they are, why you would want to write one, and how to write one. This section also describes the way in which you can derive one class from another. That is, how a subclass can inherit fields and methods from a superclass. You will learn that all classes are derived from the Object class, and how to modify the methods that a subclass inherits from superclasses.
6. [**Numbers and Strings**](https://docs.oracle.com/javase/tutorial/java/data/index.html) This lesson describes how to use Number and String objects The lesson also shows you how to format data for output.
7. [**Generics**](https://docs.oracle.com/javase/tutorial/java/generics/index.html) are a powerful feature of the Java programming language. They improve the type safety of your code, making more of your bugs detectable at compile time.
8. [**Packages**](https://docs.oracle.com/javase/tutorial/java/package/index.html) are a feature of the Java programming language that help you to organize and structure your classes and their relationships to one another.

<br>

9.  **Others**: 
- [JavaSE-8](https://docs.oracle.com/javase/8/docs/)
- [JavaSE-17](https://docs.oracle.com/en/java/javase/17/index.html)
- [JavaSE-21](https://docs.oracle.com/en/java/javase/21/index.html)


## 4. Study
### 4.1. Pattern/Matcher - java.until.regex
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

// Example 1: check matching
public class REGEX {
		public static void main(String[] args) {
			String regex = "com\\.something.*"; //$NON-NLS-1$
			String input = "com.something.test";
			Pattern pattern = Pattern.compile(regex);
			Matcher matcher = pattern.matcher(input);
			if(matcher.matches()) {
				System.err.println("Match !!!");
			}else {
				System.err.println("Do Not Match !!!");	
			}
		}
}

// Example 2: Find and replace
Map<String, String> replacements = new HashMap<String, String>() {{
    put("${env1}", "1");
    put("${env2}", "2");
    put("${env3}", "3");
}};

String line ="${env1}sojods${env2}${env3}";
String rx = "(\\$\\{[^}]+\\})";

StringBuilder sb = new StringBuilder(); //use StringBuffer before Java 9
// Note: StringBuffer is sync but slower than StringBuilder
Pattern p = Pattern.compile(rx);
Matcher m = p.matcher(line);

while (m.find())
{
    // Avoids throwing a NullPointerException in the case that you
    // Don't have a replacement defined in the map for the match
    String repString = replacements.get(m.group(1));
    if (repString != null)    
        m.appendReplacement(sb, repString);
}
m.appendTail(sb);

System.out.println(sb.toString());

```

### 4.2. URL - URI
- `URI`: Uniform Resource Identifier: the complete identification of any resources. e.g. `urn:isbn:1234567890` 
- `URL`: Uniform Resource Locator: a subset of URI that, in addition to identifying where a resource is available. And let we know the mechanism to access it. e.g. `"http://somehost:80/path?thequery"`

- Every `URL` is a `URI`, but the opposite is not true.

- URI syntax: `scheme:[//authority][/path][?query][#fragment]`
  - `scheme`: protocol to access the resource
  - `authority`: optional part, include user,host, port,...
  - `path`: identify a resource
  - `query`: also identify a resource, for URLs, this is the query string
  - `fragment`: optional part, specific  part of the resource

- URI have  ftp, http, https, file wil be a URL.
> The easiest way to see the difference between URL and URI is to remember that:
URI = identifier (the broad concept — anything that uniquely names or locates a resource)
URL = locator (a URI that tells you where and how to get it)
URI
├── URL   → has location + access method
└── URN   → name only, no access method
```java
URI.create("file:///Users/joe/FileTest.java"
```
