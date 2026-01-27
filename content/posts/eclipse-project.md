---
author: "Phong Nguyen"
title: "Eclipse Platform Plug-in Development"
date: "2025-02-05"
description: "Platform Plug-in Developer Guide."
tags: ["eclipse"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 10    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Notes for the eclipse plugin development.

**References:** 
[Wizards and Dialogs](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.pde.doc.user%2Fguide%2Ftools%2Fproject_wizards%2Fnew_fragment_project.htm)
[Eclipse fragment projects - Tutorial](https://www.vogella.com/tutorials/EclipseFragmentProject/article.html#example-manifest-for-a-fragment)
[Refer](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Fguide%2FresInt.htm)<br>

## 1. New Project Creation Wizards
- Every `plugin, fragment, feature` and update site is represented by a single project in the workspace and allow PDE(Plugin Development Environment) to validate their manifest file(s). 
- `File > New > Project... > Plug-in Development`

- Use a Plugin Project if we're building new functionality.
- Use a Fragment Project if we need to modify an existing plugin without changing its core code.
### 1.1. Plugin project
- **A plugin project** (or "bundle") is the fundamental building block in an Eclipse-based application.

- **Use-cases:**
  - We are developing a primary feature of an application.
  - We need to define new APIs, extension points, or contribute UI components.

- **Manifest for a plugin project (OSGi bundle)**
```xml
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: Plugin Name
Bundle-SymbolicName: test.plugin.project;singleton:=true       # unique identifier for an plugin, The singleton:=true means only one instance of this plugin can be active in the runtime environment.
Bundle-Version: 1.0.0.qualifier                                # is mandatory and must be of the form major.minor.micro.qualifier
Bundle-Activator: test.plugin.Activator                        # used to control the bundle's life cycle
Bundle-Vendor: Vendor Name
Require-Bundle: org.eclipse.swt,                               # lists dependencies on other plugins, this means the current plugin depends on org.eclipse.swt, ... and can use all exported packages from it.
 org.eclipse.jface;bundle-version="[3.205.0,4.0.0)",
 org.eclipse.jdt.annotation;bundle-version="[2.2.0,3.0.0)";resolution:=optional # optional, if the bundle is missing, the plugin must still work without failing.
Import-Package: com.example.utils                              # this means the plugin only depends on specific packages, not the whole bundle.
Automatic-Module-Name: test.plugin.project
Bundle-ActivationPolicy: lazy                                  # Means the plugin will not be activated until needed
Export-Package: com.example.utils                              # Other plugins can use the classes in com.example.utils
Bundle-RequiredExecutionEnvironment: JavaSE-21                 # the Java version required to run the plugin. 
```

- **Versioning Rule** (Semantic Versioning - SemVer)
MAJOR version (X.0.0): Incompatible API changes (breaking changes).
MINOR version (X.Y.0): Backward-compatible feature additions.
PATCH version (X.Y.Z): Backward-compatible bug fixes.
<br>


### 1.2. Fragment project
- **A fragment** is an optional attachment to another plug-in. This other plug-in is called the host plug-in. At runtime the fragment is merged with its host plug-in.

- **Use-cases:**
  - Contain test classes: This way, tests can access the internal API of the plugin classes and test it. (Tests can be contained in own their `plugin project`, but they can only test the external API of the other plugin)
  - Contribute property files for additional translations.
  - Provide native code which is specific to certain operating systems (OS)
  - Contain resources like icon sets or other images

- **Manifest for a fragment project**

```xml
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: %fragmentName
Bundle-SymbolicName: org.eclipse.ui.win32
Bundle-Version: 3.4.300.qualifier # is mandatory and must be of the form major.minor.micro.qualifier
Bundle-ClassPath: .
Bundle-Vendor: %providerName
Fragment-Host: org.eclipse.ui.ide;bundle-version="[3.2.0,4.0.0)"
Bundle-Localization: fragment-win32
Export-Package: org.eclipse.ui.internal.editorsupport.win32;x-internal:=true
Eclipse-PlatformFilter: (osgi.ws=win32)
Bundle-RequiredExecutionEnvironment: JavaSE-21
Automatic-Module-Name: org.eclipse.ui.win32
```

## 2. Eclipse Runtime
- Eclipse runtimes defines the plugins (osgi & runtime) on which all other plugins depend.
- `org.eclipse.osgi`: the OSGi (Open Services Gateway initiative) framework implementation

- `org.eclipse.core.runtime`: provides runtime services like extension registry, preferences, logging, etc.

### 2.1. Runtime application model
- `Applications`: an Eclipse application is a plug-in that creates an extension for the extension point `org.eclipse.core.runtime.applications`
- `Application Container`: control and execute applications. It discovers all available applications and register and `ApplicationDescriptor` OSGi service for each application.
- `ApplicationDescriptor`: can be used to launch an application
- `ApplicationHandle`: when an application is launch, this one OSGi service is registered to represent the instance of the running application, can be used to shutdown an app.

#### 2.1.1. The Default Application
- A given configurations may contain many products and applications.
- The default app can be specified:
  - `eclipse.product`
  - `eclipse.application`

#### 2.1.2. Defining an Application
- Using the `org.eclipse.core.runtime.applications` extension.
- The class which implements the applications is used to launch and shutdown the app instances.
- e.g.

```xml
   <extension id="coolApplication" point="org.eclipse.core.runtime.applications"> 
      <application> 
         <run class="com.xyz.applications.Cool"> 
            <parameter name="optimize" value="true"/> 
         </run> 
      </application> 
   </extension> 
```

```java
public class Cool implements IApplication{
	@Override
	public Object start(IApplicationContext context) throws Exception {
    // do something here
    return IApplication.EXIT_OK;
  }
}

```

- `eclipse -application coolApplication`

## 3. Resources
- The resources plug-in (**org.eclipse.core.resource**) provides services for accessing the projects, folders, and files that a user is working with.
### 3.1. Resources and Workspace
- The resources plug-in provides APIs for resources in a workspace. Our plug-in can also use these APIs.
### 3.2. Resources and file system
-  `.metadata`: platform metadata directory for holding its internal information, including workspace structure.
-  `.project`: project metadata
-  `IWorkspace`: an instance represent of the workspace when the platform is running and resources plug-in is active.
-  `IWorkspaceRoot`: the top o the resource hierarchy in the workspace
-  `IProjectDescription`: contain the project information (like .project)
-  `IProject, IFolder, IFile`: resource types
-  `org.eclipse.core.runtime.IPath`: resouce /file system paths. A path is simple (String -> IPath) or (String -> URI)
-  `IProgressMonitor`: use to report progress and allow user to cancel etc, use this when our code has a user interface (UI). `null` indicating no progress monitor

- Based on `java.io.File`

```java
// get workspace/workspace root
IWorkspace workspace = ResourcesPlugin.getWorkspace();
IWorkspaceRoot workspaceRoot = workspace.getRoot();

// get and must open project
IProject projectA = workspaceRoot.getProject("projectNameA");
if(projectA != null && !projectA.isOpen()){
    projectA.open(null);
}

// open folder and create st
IFolder folder = projectA.getFolder("folderName");
if(folder.exists()){
    // create a new file
    IFile newFile = folder.getFile("newFile.txt");

    FileInputStream fileStream = new FileInputStream(
        "c:/MyOtherData/newLogo.png");
    newLogo.create(fileStream, false, null);
    // create closes the file stream, so no worries.   
    
}
```
### 3.3 Mapping resources to disk locations
- Resource paths are always based o the project's location (workspace directory e.g. C:\MySDK\workspace). e.g. IFile file1 = src/com/example/Main.java.
- To get the full file system path to a resource, we must query its location using `IResource.getLocationURI`
- To get the corresponding resource object given a file system path, we can using `IWorkspaceRoot.findFilesForLocationURI/findContainersForLocationURI`

- When we make some changes to resource files using external methods, we need to call a file system refresh to sync with platform. e.g. `IResource.refreshLocal(int dept, IProgressMonitor monitor)`

### 3.4. Resource Properties
- There are two kinds of this:
  - Session properties: cache
  - Persistent properties: 
- Using `IResource API` to get this info.

### 3.5. Project-scoped preferences
- How to define additional scope for preferences.
- TBD

### 3.6. File encoding and content types
- Content types for data stream (Charset etc)
- TBD

### 3.7. Linked resources
- This concept let we know how the files and folders inside a project can be stored in a file system outside of the project's location.
-

## 4. Advanced resource
### 4.1. Alternate file system
- How to contribute/work with resources stored in the different file systems.
#### 4.1.1. File System API
- The `org.eclipse.core.filesystem` plug-in provides a lot of API which is similar to `java.io.File`. With a few differences:
    - Plug-ins can install providers for different types of file systems.
    - All methods integrate support for reporting progress and responding to cancelation, making it easier to integrate into a graphical user interface.
    - There is support for some additional functionality not available in java.io.File, such as getting and setting file permissions.
    - There are more convenience methods, such as copy, move, and recursive deletion.
  
- `java.net.URI`: represent for any given file in the File System API
- `IFileStore`: represents a single file (like IResource). Use this to create, delete, copy, move...
- `IFileSystem`: represents a single URI scheme (e.g. file:, ftp: )
- `IFileInfo`: represents the state(isExist etc) of a file at a particular moment
- `org.eclipse.core.filesystem.EFS`: is the main entry point for clients of the Eclipse file system API. This class has factory methods for obtaining instances of file systems and file stores, and provides constants for option values and error codes.

```java
URI uri = ...;
IFileStore store = EFS.getStore(uri); // get file store from URI
store.openOutputStream(EFS.APPEND, null); // using EFS flag to allow extra options

IFileInfo info = store.fetchInfo();
boolean state = info.isDirectory();

```
#### 4.1.2. Working With Resources in Other File Systems

### 4.2. org.eclipse.core.runtime.FileLocator
```java
String fileLocation = "resources/file.txt";
Bundle bundle = FrameworkUtil.getBundle(getClass());
IPath filePath = Path.fromOSString(fileLocation);
try{
    InputStream stream = FileLocation.openStream(bundle,filePath,true);
    // do something with input stream
    stream.close();
}catch(IOException e){
} 

```

## 5. System property & Dynamic variables
[refer](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)

### 5.1. System properties
- **System properties** include information about the current user, the current version of the Java runtime, and the character used to separate components of a file path name.
- The `System` class/ a `Properties` object
> Key - Meaning:s
"file.separator" -	Character that separates components of a file path. This is "/" on UNIX and "\" on Windows.
"java.class.path"	- Path used to find directories and JAR archives containing class files. Elements of the class path are separated by a platform-specific character specified in the path.separator property.
"java.home"	- Installation directory for Java Runtime Environment (JRE)
"java.vendor"	- JRE vendor name
"java.vendor.url"	- JRE vendor URL
"java.version"	- JRE version number
"line.separator"	- Sequence used by operating system to separate lines in text files
"os.arch"	- Operating system architecture
"os.name"	- Operating system name
"os.version" -	Operating system version
"path.separator" -	Path separator character used in java.class.path
"user.dir" -	User working directory
"user.home" -	User home directory
"user.name" -	User account name

- Examples:
```xml
VM Arguments
-Dmykey="stringValue"


```

- **Read/write system properties**:
```java
System.getProperty("user.home"); // get value of "user.home"
System.getProperties().list(System.out); // print out all the system properties


```

### 5.2. Dynamic variables (Eclipse dynamic variables)
- Eclipse dynamic variables are placeholders (like macros) that Eclipse can resolve at runtime to context-specific values.
- Some commonly used Eclipse dynamic variables:
  - `${eclipse_home}` → resolves to the Eclipse installation directory.
  - `${workspace_loc}` → resolves to the absolute path of the current workspace.
  - `${project_loc:MyProject}` → resolves to the path of the project MyProject.
  - `${resource_loc}` → resolves to the full path of the selected resource in the workspace.
  - `${java_home}` → resolves to the JRE home directory used by Eclipse.

- Example:
```xml
Program arguments
-os ${target.os} -ws ${target.ws} -arch ${target.arch} -nl ${target.nl} -consoleLog

```

- Define a dynamic variables: [refer](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fextension-points%2Forg_eclipse_core_variables_dynamicVariables.html)

```java
 <extension point="org.eclipse.core.variables.dynamicVariables">
   <variable 
      name="resource_name"
      expanderClass="com.example.ResourceNameExpander"
      description="The name of the selected resource">
   </variable>
 </extension>

public class MyVariable implements IDynamicVariableResolver {

	@Override
	public String resolveValue(IDynamicVariable variable, String argument) throws CoreException {
		return "myVariableVarlue";
	}

}
```


- Use the defined dynamic variables:
```java
import org.eclipse.core.variables.VariablesPlugin;

final static String getDynamicVariableValue(String variableName){
    return VariablesPlugin.getDefault().getStringVariableManager().getDynamicVariable(variableName).getValue(null);
} 

```
