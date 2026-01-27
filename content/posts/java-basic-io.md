---
author: "Phong Nguyen"
title: "Java Basic IO"
date: "2024-11-14"
description: "Java: Basic I/O."
tags: ["java"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 14   # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Explain how to use basic I/O.
**References:** 
[Basic I/O](https://docs.oracle.com/javase/tutorial/essential/io/index.html)<br>
[Reading and writing files in Java (Input/Output) - Tutorial](https://www.vogella.com/tutorials/JavaIO/article.html)<br>
## 1. I/O Streams
- An I/O Stream represents and input source or an output destination.
- A stream is a sequence of data.
- `input stream`: is used to **read** data from source, once item at a time.
- `outputs stream`: is used to **write** data to a destination, once item |at a time.
 
```
--stream-->
0101010101...
---------->
```
- **java.io** package

- **Streams**:
  - Byte streams: `InputStream/OutputStream/FileInputStream/FileOutputStream`
  - Character streams: `Reader/Writer/FileReader/FileWriter`
  - Buffered streams: `BufferedReader/BufferedWriter/BufferedInputStream/BufferedOutputStream`
  
- **Scanner**: `Scanner.java`
- **Formatter**: `sys.printf/String.format`

- **Always close streams.**
- **Line Terminator**: 
  -  a carriage-return/line-feed sequences `\r\n`
  -  a single carriage-return `\r`
  -  a single line-feed `\n`
  -  
### 1.1. Bytes Streams
- Bytes Streams is used to perform input and output of `8-bit bytes`.
- All other stream types are built on byte streams.
- All byte stream classes are descended from `InputStream.java` and `OutputStream.java`

- Examples:
```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class CopyBytes {
    public static void main(String[] args) throws IOException {

        FileInputStream in = null;
        FileOutputStream out = null;

        try {
            in = new FileInputStream("xanadu.txt");
            out = new FileOutputStream("outagain.txt");
            int c;

            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            if (in != null) {
                in.close();
            }
            if (out != null) {
                out.close();
            }
        }
    }
}

```

### 1.2. Character Streams
- Character values is stored using Unicode conventions.
- Character stream I/O **automatically** translates this internal format to and from the local character set.( E.g: Western: ASCII, Japan:Shift-JIS)
- All character stream classes are descended[dɪˈsend] from `Reader.java` and `Writer.java`


- Example:
```java
	private boolean writeXmlFile(Map<String, String> object, IPath filePath) {
		try (FileOutputStream outputStream = new FileOutputStream(filePath.toFile())) {
			XMLStreamWriter writer = XMLOutputFactory.newFactory().createXMLStreamWriter(outputStream,
					StandardCharsets.UTF_8.name());
			writer.writeStartDocument(StandardCharsets.UTF_8.name(), "1.0"); //$NON-NLS-1$
			writer.writeStartElement("A");
			writer.writeAttribute("xmlns:i", "http://www.w3.org/2001/XMLSchema-instance");
			writer.writeAttribute("xmlns", "http://schemas.microsoft.com/2003/10/Serialization/Arrays");
			for (Map.Entry<String, String> entry : object.entrySet()) {
				writer.writeStartElement("A");
				writer.writeStartElement("A");
				writer.writeCharacters("A");
				writer.writeEndElement();
				writer.writeStartElement("A");
				writer.writeCharacters("A");
				writer.writeEndElement();
				writer.writeEndElement();
			}
			writer.writeEndElement();
			writer.writeEndDocument();
			writer.flush();
			return true;
		} catch (IOException | XMLStreamException e) {
			System.out("Error writing XML file", e); //$NON-NLS-1$
		}
		return false;
	}
```

=> They are using the int variable to store the temp value.However, CopyBytes: int variable hold its last 8 bits , CopyCharacters hold 16bits.

> UTF-8 là một encoding để biểu diễn ký tự Unicode dưới dạng 1–4 bytes.
Ví dụ:
Ký tự ASCII 'A' → 0x41 → lưu 1 byte trong file.
Ký tự 'é' (U+00E9) → UTF-8 là 11000011 10101001 → lưu 2 byte.
Ký tự '𠜎' (U+2070E) → UTF-8 là 11110000 10100000 10011100 10001110 → lưu 4 byte.
=> Chung ta can encode/decode dung.

## 1.3. Buffered Streams
- The unbuffered IO are much less efficient because they call directly to the OS/native I/O (triggers disk access etc.) => Should use the buffered IO streams to reduce system call -> faster for most of use cases 
-  `BufferedReader/BufferedWriter`: create buffered `character streams`
-  `BufferedInputStream/BufferedOutputStream`: create the buffered `bytes streams`

- **Flushing buffered streams**: to write out a buffer
  - the buffer full
  - close stream
  - flushing the buffer (.flush)
  - auto flushing (.`println` or `format` )

## 1.4. Scanning and Formatting
- Scanning: API breaks input into individual tokens associated with bits of datas
- Formatting: API assembles data into nicely format, human-readable
### 1.4.1 Scanning: Scanner.java
- Breaking Input into Tokens: by default, we uses a scanner uses while space to separate tokens
- We can use a different token separator, invoke `useDelimiter()`, specifying a regular expression.

### 1.4.2 Formatting: 
- Bytes stream class: `PrintStream`, syserr,sysout ,  using String
e.g.
`System.out.println("The square root of " + i + " is " + r + ".")`

- Character stream class: `PrintWriter`, printf, using format
`        System.out.format("The square root of %d is %f.%n", i, r)`

- `%x`: conversions
- In the Java programming language, the `\n` escape always generates the linefeed character (\u000A). Don't use `\n` unless you specifically want a linefeed character. To get the correct line separator for the local platform, use `%n`.


## 2. File I/O
- **java.nio** package

### 2.1. Path
- File systems store the files in a tree structure, included: One root nodes and files and directories (folders), subdirectories(subfolders) 
- Delimiter is the character to separate the directories names: linux OS uses the `forwardslash`(/), windows using the `backslash`(\)
-  Root: `/` or `C:\`
-  Folders: `/home/phong/myfolder` or `C:\phong\myfolder`

=> **Path used to identify a file through the system file**.

### 2.2. Relative/ Absolute:
- A path is either relative or absolute.
- **Absolute** path always contains the root element. e.g. `/home/phong/myfolder`
- **Relative** needs to be combined with another path in order to access file. e.g.g `/phong/foo`


### 2.3. Symbolic link
- A symbolic link is a special file that serves as a reference to another file.
- Reading or writing to a symbolic link is the same as reading or writing to any other file or directory. For example, resolving `logFile` yields `dir/logs/HomeLogFile` (actual)

## 2.4. **java.nio.file.Path.java**
- `Path.java`: A Path instance contains the information used to specify the location of a file or directory
- `Paths.java`: Helper class


- 1. Creating a Path
- 2. Retrieving information
- 3. Removing redundancies
- 4. Converting a path
- 5. Joining two paths
- 6. Create a Path between two paths
- 7. Comparing two paths
```java
// 1. Creating a path
		// String filePath = "resources/files"; // forward-slash
		String filePath = "resources\\files"; // back-slash
		Path path = Paths.get(filePath);

		// 2. Retrieving the info
		System.out.format("toString: %s%n", path.toString());

		// 3. Removing Redundancies From a Path
		// Path path.normalize();

		// 4. Converting a Path
		// path.torUr()

		// 5. Joining two paths
		Path parentPath = Paths.get(filePath);
		String subFolderName = "subfolder";
		Path subPath = parentPath.resolve(subFolderName);
		System.out.format("subPath toString: %s%n", subPath.toString());

		// 6. Creating a path between two paths
		Path path1 = Paths.get("path1");
		Path path2 = Paths.get("path2");
		Path p2_to_p1 = path2.relativize(path1); // get relative path1 from path2
		System.out.format("p2_to_p1 toString: %s%n", p2_to_p1.toString());

		// 7. Comparing two paths
		if (path1.equals(path2)) {

		} else if (path1.endsWith(path2)) {

		} else if (path1.startsWith(path2)) {

		}
```


## 2.5. **java.nio.file.Files.java**
- This class offers a rich set of static methods for reading, writing, and manipulating files and directories.
- The methods works on instances of Path objects.

- 1. Checking a File or Directory
- 2. Deleting a File or Directory
- 3. Copying a File or Directory
- 4. Moving a File or Directory
```java
public class FileOperations {
	public static void main(String[] args) throws IOException {
		// 1. Checking Files or Directory:
		// Checking exist
		Path path = Paths.get("resources/files");
		if (Files.exists(path)) {
			System.out.println(path.toString() + " is exist !!");
		}

		// Checking Accessis
		boolean isRegularFile = Files.isDirectory(path) && Files.isReadable(path) && Files.isWritable(path);
		if (isRegularFile) {
			System.out.println(path.toString() + " is regular");
		}

		// Checking whether two paths locate the sample file
		Path p1 = Paths.get("resources/files");
		Path p2 = Paths.get("resources/files");

		if (Files.isSameFile(p1, p2)) {
			// Logic when the paths locate the same file
			System.out.println("They are location the same file");
		}

		// 2. Deleting
		try {
			// Files.delete(p1);
		} catch (Exception e) {
			// TODO: handle exception
		}
		
		// 3. Copying
//		Files.copy(source, target, REPLACE_EXISTING);
		
		// 4. Moving
//		Files.move(source, target, REPLACE_EXISTING);
	}
}
```

### 2.5.1. Managing Metadata (File and File Store Attributes)
### 2.5.2 Reading, Writing, and Creating Files
- There are a wide array of file I/O methods to choose
  - `ReadAllBytes/ReadAllLines`: commonly used, small files
  - `newBuffedReader/newBufferedWriter`: text files
  - `newInputStream/newOutputStream`: streams, unbuffered,
  - `newByteChannel`: channel and byte buffers
  - `FileChannel`: advanced,file-locking


-  Creating regular/temporary files
- Creating files:
```java
Path file = ...;
try {
    // Create the empty file with default permissions, etc.
    Files.createFile(file);
} catch (FileAlreadyExistsException x) {
    System.err.format("file named %s" +
        " already exists%n", file);
} catch (IOException x) {
    // Some other sort of failure, such as permissions.
    System.err.format("createFile error: %s%n", x);
}
```


### 2.5.3 Walking the File Tree
- This will recursively visit all the files in a file tree.

1. Create the FileVisitor
- `FileVisitor Interface`: interface has four methods 
  - `pre/postVisitDirectory`:Invoked before/after a directory's entries are visited.
  - `visitFile`: invoked on the file being visited
  - `visitFileFailed`: invoked when the file cannot be accessed.
- `SimpleFileVisitor`: class which implements FileVisitor interface, we can extend this

2. Kickstarting the process
```java
Path startingDir = ...;
PrintFiles pf = new PrintFiles();
Files.walkFileTree(startingDir, pf);
```


### 2.5.4 Others
- 7. Random Access Files
- 8. Creating and Reading Directories
- 9. Links, Symbolic or Otherwise
- 11. Finding Files
- 12. Watching a Directory for Changes
- 13. Other Useful Methods
- 14. Legacy File I/O Code