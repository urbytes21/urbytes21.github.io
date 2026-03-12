---
author: "Phong Nguyen"
title: "Java NPE"
date: "2025-02-27"
description: "Java: NPE stuffs"
tags: ["java","eclipse"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 100    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Some stuffs related to NPE
[Refer1](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)
[Refer2](https://www.baeldung.com/spring-null-safety-annotations)
## 1. Optional<T>
- This class will be an container object which may or may not contain a non-null value
- Examples:
```java
// Potential dangers of null
String version = computer.getSoundcard().getUSB().getVersion();


// Solution 1: Ugly due to the nested checks, decreasing the readability
String version = "UNKNOWN";
if(computer != null){
  Soundcard soundcard = computer.getSoundcard();
  if(soundcard != null){
    USB usb = soundcard.getUSB();
    if(usb != null){
      version = usb.getVersion();
    }
  }
}

// Solution 2(JavaSE7): 
String version = computer?.getSoundcard()?.getUSB()?.getVersion();
String version = 
    computer?.getSoundcard()?.getUSB()?.getVersion() ?: "UNKNOWN";

// Solution 3 (JavaSE8)
public class Computer {
  private Optional<Soundcard> soundcard;  
  public Optional<Soundcard> getSoundcard() { ... }
  ...
}

public class Soundcard {
  private Optional<USB> usb;
  public Optional<USB> getUSB() { ... }

}

public class USB{
  public String getVersion(){ ... }
}

String name = computer.flatMap(Computer::getSoundcard)
                          .flatMap(Soundcard::getUSB)
                          .map(USB::getVersion)
                          .orElse("UNKNOWN");

```

### 1.1 Creating Optional objects - Optional.of(var)/ Optional.ofNullable(var)
```java
SoundCard soundCard = new SoundCard();
Optional<SoundCard> sc = Optional.of(soundCard);    // NPE if soundCard is null
Optional<SoundCard> sc = Optional.ofNullable(soundCard); // may hold a null value
```

### 1.2. Check presenting - .ifPresent(//)
```java
// ugly code
SoundCard soundcard = ...;
if(soundcard != null){
  System.out.println(soundcard);
}

// solution 1
SoundCard soundcard = ...;
if(soundcard != null){
  System.out.println(soundcard);
}
// solution 2
if(soundcard.isPresent()){
  System.out.println(soundcard.get());
}
```

### 1.3. Default - .orElse/ .orElseThrow
```java
// ugly code
Soundcard soundcard = 
  maybeSoundcard != null ? maybeSoundcard 
            : new Soundcard("basic_sound_card");

// solution 1
Soundcard soundcard = maybeSoundcard.orElse(new Soundcard("defaut"));    // default value
Soundcard soundcard = maybeSoundCard.orElseThrow(IllegalStateException::new);    // default throw E
```
...


<br>


## 2. Null-Safety Annotations
- This is the null-safety feature produces warnings at compile time, which help prevent NPE at runtime.

## 2.1. @NonNull, @Nullable
- We can use this to declare non-null(null) constraint.
- Examples:
  
```java
import org.eclipse.jdt.annotation.*;

public class Person {
	@NonNull
    private String fullName;

    void setFullName(String fullName) {
    	fullName = null;    // warning or error
      this.fullName = fullName; //warning or error
    }
}

public class Person {
	@Nullable
    private String fullName;

    void setFullName(String fullName) {
    	fullName = null; 
      this.fullName = fullName; 
    }
}

public class Person {
	private String fullName;

	void setFullName(@NonNull String fullName) {
		this.fullName = fullName;
	}

	void setFullNameNull() {
		this.setFullName(null);    // Error here
	}
}
```
