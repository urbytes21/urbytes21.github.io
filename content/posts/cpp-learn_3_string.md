# String

## 1. C-Style String
A C-Style string is any **null-terminated byte string**, where this is a sequence of nonzero bytes followed by a byte with zero (0) value (the terminating null character).
- The terminating null character is represented as the character literal **'\0'**;
- The length of an NTBS is the number of elements that precede the terminating null character. An empty NTBS has a length of zero.
- The size of an NTBS is the size of the entire array, including the terminating null character.
- A single quotes `(')` are used to identify **character literals**.
- A double quotes `('')` are used to identify **string literals**. String literals are stored in your program image, usually in a read-only section (.data),

<br>

### 1.1. Create a String
```c
// *1. Ptr
char* strMessagePtr= "abc";
// sizeof(strMessage) = 32 or 64 (ptr)
strMessagePtr[0] = 1; // error, ptr to const

// *2. Arrays
char strMessage[] = "abc";
// sizeof(strMessage ) = 4 
strMessage[1] = 'a'; // OK
```
- For `strMessage`, the memory for the array is allocated on the stack at runtime. The compiler initializes it from the string literal. At runtime, the program memory copies the string literal into the array 
- For `strMessagePtr`, only the address of the string literal is held on the stack, and there is no copying of string literal.

<br>

### 1.2. Characters
- Null: `\0`, `0x00`, `NULL`
- Carriage Return And New Line: `\r\n`
- Case switching: `'A' ^ ' '` & `'a' ^ ' '`
- Special: `\\`, 
- **Escape sequences:**
    | Name            | Symbol       | Meaning                                        |
    | --------------- | ------------ | ---------------------------------------------- |
    | Alert           | `\a`         | Makes an alert, such as a beep                 |
    | Backspace       | `\b`         | Moves the cursor back one space                |
    | Formfeed        | `\f`         | Moves the cursor to next logical page          |
    | Newline         | `\n`         | Moves cursor to next line                      |
    | Carriage return | `\r`         | Moves cursor to beginning of line              |
    | Horizontal tab  | `\t`         | Prints a horizontal tab                        |
    | Vertical tab    | `\v`         | Prints a vertical tab                          |
    | Single quote    | `\'`         | Prints a single quote                          |
    | Double quote    | `\"`         | Prints a double quote                          |
    | Backslash       | `\\`         | Prints a backslash                             |
    | Question mark   | `\?`         | Prints a question mark *(no longer relevant)*  |
    | Octal number    | `\{number}`  | Translates into char represented by octal      |
    | Hex number      | `\x{number}` | Translates into char represented by hex number |

<br>

### 1.3. C String Libraries
- Copying strings : `strcpy`, `strncpy`
- Concatenating strings: `strcat`, `strncat`
- Comparing strings: `strcmp`, `strncmp`
- Parsing strings: `strtok`, `strcspn`
- Length: `strlen`
- Examples:
```cpp
#include <stdio.h>
#include <string.h>

int main() {
    // -------------------------------
    // 1. Copying strings
    // -------------------------------
    char src[] = "Hello";
    char dst[20];

    strcpy(dst, src);           // copy full string
    // dst = "Hello"

    strncpy(dst, "World", 3);   // copy only 3 chars
    dst[3] = '\0';              // ensure null-termination
    // dst = "Wor"

    // -------------------------------
    // 2. Concatenating strings
    // -------------------------------
    char text[20] = "Hi";
    strcat(text, " there");     // append full string
    // text = "Hi there"

    strncat(text, "!!!", 2);    // append only 2 chars
    // text = "Hi there!!"

    // -------------------------------
    // 3. Comparing strings
    // -------------------------------
    int r1 = strcmp("abc", "abc");   // r1 = 0  (equal)
    int r2 = strcmp("abc", "abd");   // r2 < 0  (abc < abd)
    int r3 = strncmp("abcdef", "abcxyz", 3); // r3 = 0 (first 3 chars equal)

    // -------------------------------
    // 4. Parsing strings
    // -------------------------------
    char line[] = "A,B,C";
    char* token = strtok(line, ",");  // first token: "A"
    while (token != NULL) {
        printf("token: %s\n", token);
        token = strtok(NULL, ",");
    }

    // strcspn: find first occurrence of any chars in reject set
    char sample[] = "hello123world";
    size_t pos = strcspn(sample, "0123456789");
    // pos = 5 (first digit is at index 5)

    // -------------------------------
    // 5. Length
    // -------------------------------
    size_t len = strlen("abc");  // len = 3

    return 0;
}
```

<br>

### 1.4. String/Numbers Conversion
- Integer to String: `itoa()` (non-standard)
- String to Double: `atof()`
- String to Double (with error checking): `strtod()`
- String to Long (with base + error checking): `strtol()`
- e.g.
```c
#include <stdio.h>
#include <stdlib.h>   // atof, strtod, strtol
#include <string.h>   // itoa (non-standard on some compilers)

int main() {
    // -------------------------------
    // 1. Integer → String (itoa)
    // -------------------------------
    char buf[32];
    itoa(1234, buf, 10);   // convert integer to string in base 10
    // buf = "1234"

    itoa(255, buf, 16);    // convert to hex
    // buf = "ff"

    // -------------------------------
    // 2. String → Double (atof)
    // -------------------------------
    double d1 = atof("3.14159");
    // d1 = 3.14159

    double d2 = atof("12.5xyz"); 
    // d2 = 12.5 (atof stops at non-numeric chars)

    // -------------------------------
    // 3. String → Double (strtod)
    // -------------------------------
    char* end;
    double d3 = strtod("45.67abc", &end);
    // d3 = 45.67
    // end -> "abc"

    // -------------------------------
    // 4. String → Long (strtol)
    // -------------------------------
    long v1 = strtol("1234", NULL, 10);
    // v1 = 1234 (decimal)

    long v2 = strtol("FF", NULL, 16);
    // v2 = 255 (hex → decimal)

    char* end2;
    long v3 = strtol("100xyz", &end2, 10);
    // v3 = 100
    // end2 -> "xyz"

    return 0;
}
```

<br>

---
## 2. C++ String **<string>**
Strings are objects that represent sequences of characters.

### 2.1. Create a String
```cpp
std::string s = "hello";    ///< initialize from a string literal
std::string s2("hello");    ///< direct initialization from a string literal
std::string s3(5, 'a');     ///< create a string with 5 copies of 'a' -> "aaaaa"
```

### 2.2. Basic Properties
| Function     | Description           |
| ------------ | --------------------- |
| `s.size()`   | number of characters  |
| `s.length()` | same as size          |
| `s.empty()`  | check empty           |
| `s.clear()`  | remove all characters |

### 2.3. Access Characters
```cpp
s[0]        // no bounds check
s.at(0)     // bounds check
s.front()   // first char
s.back()    // last char
```
### 2.4. Modify String
```cpp
/// @brief append
s.append(" world");
s += " world";

/// @brief insert
s.insert(5, "XXX");

/// @brief erase
s.erase(0,2);

/// @brief replace
s.replace(0,5,"hi");

/// @brief remove `\n`
std::string s = "hello\nworld\n";
auto new_end = std::remove(s.begin(), s.end(), '\n');
s.erase(new_end)
// std::erase(s,'\n')
s.erase(std::remove(s.begin(), s.end(), '\n'), s.end());
```
### 2.5. Substring
```cpp
/// @brief std::string sub = s.substr(pos, len); , substr() does NOT modify original string
std::string s = "abcdef";
s.substr(2,3);   // "cde"
```
### 2.6. Find / Search
```cpp
/// @brief find character
s.find('a');


/// @brief find last
s.find('a');


/// @brief find substring
size_t pos = s.find("abc");
if (pos != std::string::npos)
    std::cout << "found";
```
### 2.6. Compare Strings
```cpp
if (a == b)
if (a != b)
if (a < b)

// 0  -> equal
// <0 -> smaller
// >0 -> larger
auto ret = a.compare(b);
```
### 2.7. Start/End Checking
```cpp
/// @brief C++20
s.starts_with("abc");
s.ends_with("xyz");

/// @brief before c++ 20
s.rfind("abc",0) == 0
s.size() >= 3 &&
s.compare(s.size()-3,3,"xyz") == 0
```
### 2.8. Convert String
```cpp
/// @brief string to int
int n = std::stoi(s);
double x = std::stod("3.14");

/// @brief int to string
std::string s = std::to_string(123);
```

### 2.9. String Parsing
```cpp
#include <sstream>

std::string line = "a,b,c";
std::stringstream ss(line);
std::string item;

while(std::getline(ss,item,',')) {
    std::cout << item << std::endl;
}
```

### 2.10. Trim Whitespace
```cpp
s.erase(0, s.find_first_not_of(" \t\n\r"));
s.erase(s.find_last_not_of(" \t\n\r") + 1);
```

### 2.11. Convert Case
```cpp
#include <algorithm>

std::transform(s.begin(), s.end(), s.begin(), ::tolower);
std::transform(s.begin(), s.end(), s.begin(), ::toupper);
```

### 2.12. String Formatting

| Method            | Standard | Pros                     | Cons                         |
|------------------|----------|--------------------------|------------------------------|
| `std::format`     | C++20    | Clean, safe, Python-like | Needs newer compiler         |
| `+` concatenation | C++98    | Simple                   | Hard to format numbers       |
| `stringstream`    | C++98    | Flexible streaming       | Slow, verbose                |
| `snprintf`        | C        | Very fast, classic       | Unsafe if misused            |

---
## 3. std::cout << , std::cin >> , std::getLine(std::cin >> std::ws, std::string string)
- `std::ws` tells `std::cin` to ignore leading whitespace(tab/enter/newline(s)) before extraction.  
- `std::string::length` returns length of a string that does not included the null terminator character.
- `s` suffix is a `std::string` literally, no suffix is a C-style string literally. e.g. `std::cout << "goo\n"s`
> Initializing and copy a `std::string` is slow. That's inefficient 

---
## 4. std::string_view  (C++17)
`std::string_view` provides read-only access to an existing string (a C-style string literal, a std::string, or a char array) without making a copy. 
- A std::string_view that is viewing a string that has been destroyed is sometimes called a dangling view. When a std::string is modified, all views into that std::string are invalidated, meaning those views are now invalid. Using an invalidated view (other than to revalidate it) will produce undefined behavior.

- `sv` suffix is a `std::string_view` literally
- Modify a std::string is likely to invalidate all std::string_view that view into that.
- It may or may not be null-terminated.