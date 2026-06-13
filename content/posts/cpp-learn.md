---
author: "Phong Nguyen"
title: "Cpp"
date: "2025-02-09"
description: "Cpp Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
See plus plus :) .

<br>

[Refer](https://www.learncpp.com/)

1. [Introduce](cpp-learn_1_Introduction.md.md)
2. [Fundamentals](cpp-learn_2_fundamentals.md)
2. [String](cpp-learn_3_string.md)

### 18.3. Class template
- a **class template** is a template definition for instantiating class types (structs, classes, or unions). Class template argument deduction (CTAD) is a C++17 feature that allows the compiler to deduce the template type arguments from an initializer.s
- Using class template in a function:
- e.g.
```cpp
#include <iostream>

template <typename T>
struct Pair
{
    T first{};
    T second{};
};

template <typename T>
constexpr T max(Pair<T> p)
{
    return (p.first < p.second ? p.second : p.first);
}

int main()
{
    Pair<int> p1{ 5, 6 };        // instantiates Pair<int> and creates object p1
    std::cout << p1.first << ' ' << p1.second << '\n';

    Pair<double> p2{ 1.2, 3.4 }; // instantiates Pair<double> and creates object p2
    std::cout << p2.first << ' ' << p2.second << '\n';

    Pair<double> p3{ 7.8, 9.0 }; // creates object p3 using prior definition for Pair<double>
    std::cout << p3.first << ' ' << p3.second << '\n';

    std::cout << max<int>(p1) << " is larger\n"; // explicit call to max<int>

    return 0;
}

// Compiler ===================================================================================
#include <iostream>

// A declaration for our Pair class template
// (we don't need the definition any more since it's not used)
template <typename T>
struct Pair;

// Explicitly define what Pair<int> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<int>
{
    int first{};
    int second{};
};

// Explicitly define what Pair<double> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<double>
{
    double first{};
    double second{};
};

int main()
{
    Pair<int> p1{ 5, 6 };        // instantiates Pair<int> and creates object p1
    std::cout << p1.first << ' ' << p1.second << '\n';

    Pair<double> p2{ 1.2, 3.4 }; // instantiates Pair<double> and creates object p2
    std::cout << p2.first << ' ' << p2.second << '\n';

    Pair<double> p3{ 7.8, 9.0 }; // creates object p3 using prior definition for Pair<double>
    std::cout << p3.first << ' ' << p3.second << '\n';

    return 0;
}
```

- **Using class templates in multiple files**:Just like function templates, class templates are typically defined in header files

- **Alias templates**: is a template that can be used to instantiate type aliases.
- e.g.
```cpp
// Alias templates must be defined in global scope
template <typename T>
using Coord = Pair<T>; // Coord is an alias for Pair<T>
```



### 21.8. Printing inherited classes using operator<<
- Refer to the learncpp.com


```
## 23. I/O 
### 23.1. I/O Streams
- It is a part of the STL.
- I/O is implemented with `streams`.
<br>

- **stream** is a sequence of bytes that can be accessed sequentially. It may produce or consume amounts data over time.
	- **input stream**: used to hold input data from a data producer.
	- **output stream**: used to hold output data for a particular data consumer.
	- *e.g.* when writing/input data to an device, the device may not be ready to accept that data, so the data will sit in the stream. 
	- [C++] <=> streams <=> [os]
<br>

- **I/O in C++**: we can use the STL classes to deal with streams
	- **input stream**: `istream` & extraction operation (`>>`) is used to remove values from the stream
	- **output stream**: `ostream` & insertio operation (`<<`) is used to put vlaues in the stream.\
	- `iostream` can handle both i & o
<br>

- **Standard streams**:
	- `cin`: an `istream` object tied to the standard input
	- `cout`: an `ostream` object tied to the standard output
	- `cerr`: an `ostream` object tied to the standard error - unbuffered output
	- `clog`: an `ostream` object tied to the standard error - buffered output

### 23.2. Input with istream
- Use `extraction operator (>>)` to read information from an input stream. It skips **whitespace (blanks, tabs, and newlines)**. Use `get(), getLine()` to not discard the whitespace.
- `manipulator` is an object that is used to modify a stream when applied with the `extraction (>>)` or `insertion (<<)` operators. <iomanip>

### 23.3. Output with ostream 
https://www.learncpp.com/cpp-tutorial/output-with-ostream-and-ios/

### 23.4. Stream classes for string <sstream>
- String streams provide a buffer to hold data but are not connected to an I/O channel.
- The primary uses of the string stream:
  - Display data later then or proccess i/o data.
  - Get data into a string stream
  - Get data from a string stream
  - Convertion strings/numbers
  - Clear a string stream

- e.g:
```cpp
#include <sstream>
#include <string>
int main() {
  std::stringstream os{};
  // input
  os << "0xF";
  std::cout << os.str();
  
  os.str("0x1 0x2");
  std::cout << os.str();

  // output
  std::string bytesStr  = os.str();
  std::cout << bytesStr;

  os.str("0x0 0xF 0xE 0x2");
  os >> bytesStr;
  std::cout <<bytesStr;

  // conversions
  int byte_low = 0xFFF;
  int byte_high = 0x001;
  os.clear();
  os << byte_low << ' ' << byte_high;
  std::cout << os.str();
}
```

### 23.5. Validation
https://www.learncpp.com/cpp-tutorial/stream-states-and-input-validation/

### 23.6. File I/O <fstream>
- There are three basics file I/O classes:
  - `ifstream`: derived from `istream`
  - `ofstream`: derived from `ostream`
  - `fstream`: derived from `stream`
<br>

- **important:** We have to explicitly setup to use file streams:
  - 1. **Open a file: instantiate an object of file I/O class, with the name of the file as a param**
  - 2. **Use the insertion (<<) or extraction (>>) to r/w**
  - 3. **Close a file: call the close() or let file I/O go out of scope.**
  - 4. **To delete a file, simply use the remove() function.**
<br>

- **File output:**
```cpp
#include <fstream>
#include <iostream>
#include <string>

static void fileOutput() {
  std::ofstream outfile{"grs_bytes.csv"};
  if (!outfile || !outfile.is_open()) {
    std::cerr << "[E] Cannot create the output file";
    return;
  }

  // Put bytes data to the file
  // put string
  std::string elfBytes{
      R"""(
        time_s,
        gsr_value 0.0, 45.27761157741693 0.005, 41.69912812397066 0.01,
        38.13110177547114 0.015, 35.32162785580394 0.02,
        31.75617843363382 0.025, 28.352875321528607 0.03,
        25.23210282654006 0.035, 21.769688905641132 0.04,
        17.99031153059391 0.045, 15.073732055666543 0.05,
        15.13550182371759 0.055, 14.69547985289048 0.06,
        14.867397107985468 0.065, 14.982082556093832 0.07,
        14.893751010484861 0.075, 14.877034044202343 0.08,
        14.820590581790071 0.085, 15.1065350504897 0.09,
        15.152287796727098 0.095, 14.764395300078201 0.1,
        15.118189654760348 0.105, 15.473255351586635 0.11,
        14.913896402347113
    \n )"""};
  outfile << elfBytes;

  elfBytes =
      "0x0 0x0"
      "0x0 0x0"
      "0x0 0x0";
  outfile << elfBytes;

  elfBytes =
      "0xF 0xA \
              0xE 0xB \
              0x0 0x0";

  outfile << elfBytes;

  outfile.put('E');  // put char
  outfile.close();
}
```
<br>

- **File input:**
```cpp
static void fileInput() {
  std::ifstream inFile{"grs_bytes.csv"};
  if (!inFile.is_open()) {
    std::cerr << "Error to open file";
  }

  std::string inputStr{};
  std::cout << "========" << std::endl;
  while (
      inFile >>
      inputStr) {  //  Note that ifstream returns a 0 if we’ve reached the end of the file (EOF)
    std::cout << inputStr;
  }
  std::cout << "========" << std::endl;

  // not skip whitespace
  inFile.close();
  inFile.open("grs_bytes.csv"); // explicitly call open()
  inputStr.clear();
  while (std::getline(inFile, inputStr)) {
    std::cout << inputStr << std::endl;
  }
  std::cout << "========" << std::endl;

  inFile.close();
}
```
<br>

- **File Mode:**
  - `app`: opens the file in append mode
  - `ate`: seeks to the end of the file before reading/writing
  - `binary`: opens the file in binary mode (instead of text mode)
  - `in`: opens the file in read mode (default for ifstream)
  - `out`: opens the file in write mode (default for ofstream)
  - `trunc`: erases the file if it already exists
<br>

### 23.6 Ramdom File I/O
- **File pointer:** Each file stream class contains a file pointer that is used to keep track of the current read/write position within the file
- **Ramdom access** with `seekg() & seekp()`
- **IOS seek flags:**
	- `beg`: the offset is relative to the beginning of the file (default)
	- `cur`: the offset is relative to the current location of the file pointer
	- `end`: the offset is relative to the end of the file

> In programming, a newline (‘\n’) is actually an abstraction.
On Windows, a newline is represented as sequential CR (carriage return) and LF (line feed) characters (thus taking 2 bytes of storage).
On Unix, a newline is represented as a LF (line feed) character (thus taking 1 byte of storage).

> // assume iofile is an object of type fstream
iofile.seekg(iofile.tellg(), std::ios::beg); // seek to current file position
<br>