---
author: "Phong Nguyen"
title: "Eclipse Apache Maven-Tycho"
date: "2025-06-26"
description: "Eclipse: Maven Tycho."
tags: ["java"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 100    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

Introducing about Maven and how to use Maven Tycho plugin to build plugins into the SNAPSHOT

[Refer1](https://www.vogella.com/tutorials/EclipseTycho/article.html)<br>
[Refer2](https://www.vogella.com/tutorials/ApacheMaven/article.html)<br>
[Refer3](https://www.vogella.com/tutorials/EclipseMaven/article.html)<br>

## 1. Apache Maven
- Apache Maven is an powerful *build tool* primary for Java software projects. It is implemented in Java which makes it platform-independent.

### 1.1. Setup Env
- Install maven: https://maven.apache.org/download.cgi
- Verify: `mvn --version`
- Config proxy (ONLY IF NEED): C:\Users\${username}\.m2\settings.xml or ${maven_home}\apache-maven-3.9.10\conf\settings.xml
```xml
<settings>
  <proxies>
   <proxy>
      <id>example-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.example.com</host>
      <port>8080</port>
      <username>proxyuser</username>
      <password>somepassword</password>
      <nonProxyHosts>www.google.com|*.example.com</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```
### 1.2. Concepts
- Need the pom file defines: identifiers for the project to be build/ properties relevant for build configuration/ plugins which providefunctionality for the build via a build section. /library and project dependencies via the dependencies section
- Each project have its onw pom file but What is pom file ? `https://maven.apache.org/pom.html`
-  Example:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>        // version of the pom file               

  <groupId>com.vogella.maven.first</groupId>          // group id   -> maven will build the package: <groupId>:<artifactId>:<version>   
  <artifactId>com.vogella.maven.first</artifactId>    // project id
  <version>1.0-SNAPSHOT</version>                     // project version    

  <name>com.vogella.maven.first</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>   // java version
    <maven.compiler.target>11</maven.compiler.target>   
  </properties>

</project>
```

- A Maven project uses the groupId, artifactId, version (also knows as GAV) and the packaging property to uniquely identify a Maven component.

- ...
### 1.3. Maven wrapper
- Script which downloads automatically the correct version of Maven
```powershell
cd {your-project}
mvn -N io.takari:maven:wrapper
```
- To run: 
```powershell 
UNIX:

./mvnw clean package
WINDOWS:

mvnw.cmd clean package
```

### 1.4. Running a Maven build

```powershell
mvn install // creates the build in the target folder
mvn clean install // delete before 
mvn clean install -T4 // have 4 processes  
mvn -o clean install // offline

```

### 1.5. Create and build the project
- 1. Generate the project:
```powershell
mvn archetype:generate -DarchetypeGroupId=com.vaadin -DarchetypeArtifactId=vaadin-archetype-application -DarchetypeVersion=7.1.8 -DgroupId=im.sma.testproject -DartifactId=testproject -Dversion=1.0 -Dpackaging=war
```

- 2. Update Java version
```powershell
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
  </properties>
```
- 3. Build the project
```powershell
mvn compile
```

- 4. Create a JAR file
```powershell
mvn clean package // clean the previous artifacts and create the new
```

- 5. Validate
```powershell
java -cp target/com.vogella.maven.first-1.0-SNAPSHOT.jar com.vogella.maven.first.App
// java -cp snapshot.jar class
```

- 6. Running the test
```powershell
mvn test
```

- 7. Update dependency
```xml
  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.7.2</version>    // here
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>5.7.2</version>    // here
      <scope>test</scope>
    </dependency>
  </dependencies>
```
- 8. Specify the main class in the MANIFEST
- Cd `target` and run with ` java -cp com.vogella.maven.first-1.0-SNAPSHOT.jar com.vogella.maven.first.App`
- Change pom file:
```xml
    <build>
        <plugins>
            <!-- all the existing build entries-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.vogella.maven.first.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
- Rebuild: `mvn clean verify`
- From now on, we can run: `java -jar com.vogella.maven.first-1.0-SNAPSHOT.jar`
