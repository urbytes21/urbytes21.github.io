# Structures, Classes, Enumerations, and Unions
In C++, struct, class, and union automatically create a new type name, so you don’t need to prefix variables with the keywords struct or union as in C.

## 1. Enumerations
An **enumeration (enum)** is a user-defined type whose values are restricted to a set of named integral constants called **enumerators**.
- `unscoped-enum`: put their enumerator names into the same scope as the enumeration definition itself 
- `scoped-enum`: keep their enumerators inside the enum’s own scope.Using `enum class` keyword.
- `using enum <EnumName>` statement imports all the enumerators from an enum into the current scope. 
- e.g.
```cpp
enum color {
    red,
    green,
};

//  Scoped enum: enumerators are inside the enum's scope
enum class shape {
    circle,
    square,
};

//  Scoped enum inside a namespace to prevents name pollution
namespace game {
    enum class direction {
        up,
        down,
        left,
    };
}

//  Scoped enum with explicit base type
enum class status : uint8_t {
    ok = 0,
    error = 1,
};
```

---
## 2. Union
A **union** is a user-defined type whose members share the same memory location.
- A union can contain members of different types.
- Only one member should be active at a time.
- Writing to one member overwrites the value of the previously active member.
- The size of a union is at least the size of its largest member and may be larger due to alignment requirements.

- e.g. 
```cpp
// +------------------+
// | Shared Memory    |
// |                  |
// | int i            |
// | float f          |
// | char c           |
// +------------------+

// All members occupy the same memory location
union Data
{
    int i;
    float f;
    char c;
};
```

---
## 3. Struct
- A **struct** is a user-defined **class type** that groups related data members into a single type.
- In C++, `struct` and `class` are nearly identical. The primary difference is that struct members are `public` by default, whereas class members are `private` by default.
- Structs are commonly used to represent simple data objects.
<!-- (In C, a struct can only contain data members.) -->
### 3.1. Defining a Struct
- Structs are defined using the `struct` keyword.
    ```cpp
    struct SensorData
    {
        double voltage {};
        int id {};
        char status {};
    };
    ```
### 3.2. Access struct members
- Use the member selection operator (`.`) when working with an object or reference.
- Use the member access operator (`->`) when working with a pointer.


### 3.3. Initialization
- Structs can be initialized using brace initialization (`{}`).
- Default member initializers may be provided.
- Prefer initializing all members.
```cpp
    Employee frank = { 1, 32, 60000.0 }; // copy-list initialization using braced list
    Employee joe { 2, 28, 45000.0 };     // list initialization using braced list (preferred)
``` 

### 3.4. Passing and returning structs
  - Passing reference (efficient and avoids copying)
  - Passing temporary 
  - Create a struct variable and return
  - Returning a temporary (unnamed/anonymous) object 

### 3.5. Struct Size and Alignment
- The size of a struct is at least the sum of the sizes of its members.
- Compilers may insert unused bytes called **padding** to satisfy alignment requirements.
- Padding improves memory access efficiency.
- To reduce padding, order members from largest to smallest type.

- e.g.
```cpp
// Memory Layout
// +---+-------+--------+----+-----+
// | c |  PAD  |   d    | i  | PAD |
// +---+-------+--------+----+-----+
//
// sizeof(Bad) = 24 bytes
struct Bad
{
    char   c;   // 1 byte
    double d;   // 8 bytes
    int    i;   // 4 bytes
};


// Memory Layout
// +--------+----+---+-----+
// |   d    | i  | c | PAD |
// +--------+----+---+-----+
//
// sizeof(Good) = 16 bytes
struct Good
{
    double d;   // 8 bytes
    int    i;   // 4 bytes
    char   c;   // 1 byte
};


// Memory Layout (#pragma pack(1))
// +---+--------+----+
// | c |   d    | i  |
// +---+--------+----+
//
// sizeof(Packed) = 13 bytes
#pragma pack(push, 1)
struct Packed
{
    char   c;   // 1 byte
    double d;   // 8 bytes
    int    i;   // 4 bytes
};
#pragma pack(pop)

```

- `#pragma pack(n)` changes the alignment requirements of structure members.
- Packed structures save memory but may reduce performance due to unaligned memory accesses.
- Use packed structures only when required (e.g., hardware registers, communication protocols, file formats).
