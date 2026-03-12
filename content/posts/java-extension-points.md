---
author: "Phong Nguyen"
title: "Java Extension Points - Extensions"
date: "2025-02-27"
description: "Java: Eclipse Extensions and Extension Points"
tags: ["java","eclipse"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 100    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Explain how to use the Extension Point and Extensions.
[Refer1](https://www.vogella.com/tutorials/EclipseExtensionPoint/article.html)
[Refer2](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.pde.doc.user%2Fguide%2Ftools%2Feditors%2Fmanifest_editor%2Fextension_points.htm)
[Refer3](https://vogella.com/blog/getting-started-with-osgi-declarative-services-2024/)<br>
## 1. Introduction
- **Extensions** are the central mechanism for contribute behavior to the platform. We used it to contribute functionality to a certain type of API.
- **Extension points** define new functions points for the platform that other plug-ins can plug into.
- **E.g.** When you want to create modular that can be added or removed at runtime, you can use the OSGi service.
- These extensions and extension points are defined via the `plugin.xml` file.

![image](/images/java_extensions_extensionpoints.png)<br>


## 2. Using Extension points/ Extensions
  
### 2.1. Create an EXTENSION POINT
- A plugin which declares an **extension point** must declare the extension point in its plugin.xml.
- It will contain code to evaluate the **extensions**.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?eclipse version="3.4"?>
<plugin>
    <extension-point id="com.extensionpoint.greeters"
        name="Greeters" 
        schema="schema/com.extensionpoint.greeters.exsd" />
</plugin>
```
- It has three attributes:
  - `id`: a required attr, simple name id
  - `name`: a required attr, sime name string
  - `schema`: optional attr, describe

- **Examples**: Create a plugin named `com.extensionpoint.definition` contained the a extension point.
```XML
//MANIFEST
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: A Plugin for the Extension Point Definition
Bundle-SymbolicName: com.extensionpoint.definition;singleton:=true
Bundle-Version: 1.0.0.qualifier
Export-Package: com.extensionpoint.definition
Bundle-Vendor: Phong
Automatic-Module-Name: com.extensionpoint.definition
Bundle-RequiredExecutionEnvironment: JavaSE-21
Import-Package: org.eclipse.core.runtime;version="[3.7.0,4.0.0)",
 org.eclipse.e4.core.di.annotations;version="[1.6.0,2.0.0)"
```

```XML
// plugin.xml
<?xml version="1.0" encoding="UTF-8"?>
<?eclipse version="3.4"?>
<plugin>
   <extension-point id="com.vogella.extensionpoint.definition.greeter" name="Greeter (Extension Point Name)" schema="schema/com.vogella.extensionpoint.definition.greeter.exsd"/>

</plugin>
```
// exsd
<?xml version='1.0' encoding='UTF-8'?>
<!-- Schema file written by PDE -->
<schema targetNamespace="com.vogella.extensionpoint.definition" xmlns="http://www.w3.org/2001/XMLSchema">
<annotation>
      <appinfo>
         <meta.schema plugin="com.vogella.extensionpoint.definition" id="com.vogella.extensionpoint.definition.greeter.id" name="Greeter (Extensio Point Name)"/>
      </appinfo>
      <documentation>
         [Enter description of this extension point.]
      </documentation>
   </annotation>

   <element name="extension">
      <annotation>
         <appinfo>
            <meta.element />
         </appinfo>
      </annotation>
      <complexType>
         <choice minOccurs="1" maxOccurs="unbounded">
            <element ref="client"/>
         </choice>
         <attribute name="point" type="string" use="required">
            <annotation>
               <documentation>
                  
               </documentation>
            </annotation>
         </attribute>
         <attribute name="id" type="string">
            <annotation>
               <documentation>
                  
               </documentation>
            </annotation>
         </attribute>
         <attribute name="name" type="string">
            <annotation>
               <documentation>
                  
               </documentation>
               <appinfo>
                  <meta.attribute translatable="true"/>
               </appinfo>
            </annotation>
         </attribute>
      </complexType>
   </element>

   <element name="client">
      <complexType>
         <attribute name="class" type="string">
            <annotation>
               <documentation>
                  
               </documentation>
               <appinfo>
                  <meta.attribute kind="java"/>
               </appinfo>
            </annotation>
         </attribute>
      </complexType>
   </element>

   <annotation>
      <appinfo>
         <meta.section type="since"/>
      </appinfo>
      <documentation>
         [Enter the first release in which this extension point appears.]
      </documentation>
   </annotation>

   <annotation>
      <appinfo>
         <meta.section type="examples"/>
      </appinfo>
      <documentation>
         [Enter extension point usage example here.]
      </documentation>
   </annotation>

   <annotation>
      <appinfo>
         <meta.section type="apiinfo"/>
      </appinfo>
      <documentation>
         [Enter API information here.]
      </documentation>
   </annotation>

   <annotation>
      <appinfo>
         <meta.section type="implementation"/>
      </appinfo>
      <documentation>
         [Enter information about supplied implementation of this extension point.]
      </documentation>
   </annotation>


</schema>

```xml

```

```java
// Evaluation class
package com.extensionpoint.definition;

import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IConfigurationElement;
import org.eclipse.core.runtime.IExtensionRegistry;
import org.eclipse.core.runtime.ISafeRunnable;
import org.eclipse.core.runtime.SafeRunner;
import org.eclipse.e4.core.di.annotations.Execute;

public class EvaluateContributionsHandler {
    private static final String IGREETER_ID = 
            "com.extensionpoint.definition.greeter";
    
    @Execute
    public void execute(IExtensionRegistry registry) {
        IConfigurationElement[] config =
                registry.getConfigurationElementsFor(IGREETER_ID);
        try {
            for (IConfigurationElement e : config) {
                System.out.println("Evaluating extension");
                final Object o =
                        e.createExecutableExtension("class");
                if (o instanceof IGreeter) {
                    executeExtension(o);
                }
            }
        } catch (CoreException ex) {
            System.out.println(ex.getMessage());
        }
    }

    private void executeExtension(final Object o) {
        ISafeRunnable runnable = new ISafeRunnable() {
            @Override
            public void handleException(Throwable e) {
                System.out.println("Exception in client");
            }

            @Override
            public void run() throws Exception {
                ((IGreeter) o).greet();
            }
        };
        SafeRunner.run(runnable);
    }
}
```

```java
// Attribute class
package com.vogella.extensionpoint.definition;

public interface IGreeter {
    void greet();
}

```
<br>

### 2.2. Create/adding EXTENSIONS to EXTENSION POINTS
- **Examples**: Create a plugin named `com.extension.contribution` contained the a extension.
```XML
// MANIFEST
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: Contribution
Bundle-SymbolicName: com.vogella.extensionpoint.contribution;singleton:=true
Bundle-Version: 1.0.0.qualifier
Require-Bundle: org.eclipse.core.runtime;bundle-version="3.32.0",
 com.vogella.extensionpoint.definition;bundle-version="1.0.0"
Bundle-Vendor: VOGELLA
Automatic-Module-Name: com.vogella.extensionpoint.contribution
Bundle-RequiredExecutionEnvironment: JavaSE-21

```

```XML
// plugin.xml

```

```java
// The client class that implemented attribute class

package com.vogella.extensionpoint.contribution;

import com.vogella.extensionpoint.definition.IGreeter;

public class GreeterExensionContribution implements IGreeter {

	@Override
	public void greet() {
		System.err.println("This call from GreeterExensionContribution");
	}

}
```

### 2.3. Accessing EXTENSIONS
- The information about the available extensions point and the provided extensions are stored in a class of type `IExtensionRegistry` .
- We can use the dependency injection mechanism to get the `IExtensionRegistry` class that injected.
- `IExtensionRegistry` contains all extensions of all extensions point.
```java
// The following will read all existing extensions 
// for the defined ID of the extension point
Platform.getExtensionRegistry().
    getConfigurationElementsFor("yourextensionpointid");

```

```java
		IConfigurationElement[] elements = Platform.getExtensionRegistry()
				.getConfigurationElementsFor("com.renesas.cdt.devicesupport.api.builderDeviceSupport");
		for (IConfigurationElement ele : elements) {

        }

```

## 3. Filter Element with Choice/ Sequence
### 3.1. <sequence>
- It means child elements must appear in the exact oder listed.
```xml
<sequence minOccurs="1" maxOccurs="unbounded">
   <element ref="client1"/>
   <element ref="client2"/>
</sequence>

<element name ="client1" type="string">
<element name ="client2" type="string">
```

- Valid:

```xml
<c>
  <client1>John</client1>
  <client2>Doe</client2>
</c>
```

### 3.2. <choice>
- It means only one of the child elements is allowed at a time.
```xml
<choice>
   <element ref="client1"/>
   <element ref="client2"/>
</choice>

<element name ="client1" type="string">
<element name ="client2" type="string">
```

- Valid:

```xml
<c>
  <c1>John</c1>
</c>

```

### 3.3. Filter
- Combine the <choice> and <sequence> to make the robust filter.