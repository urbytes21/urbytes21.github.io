---
author: "Phong Nguyen"
title: "C++ - Chapter 6: Pointer and Reference"
date: "2026-05-24"
description: "C++ Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

# Pointer and Reference

#### lvalue/rvalue
- An **lvalue** is an expression that refers to an identifiable object, function, or bit-field. It has an address that can be taken and typically persists beyond a single expression.
- An **rvalue** is an expression whose primary purpose is to provide a value rather than identify an object. e.g. `temporary objects` and `literals`
- An **lvalue** can generally be used wherever an **rvalue** is expected because its value can be read.
- An **rvalue** cannot generally be used where a modifiable **lvalue** is required.

- The assignment operator (`=`) requires:
  - left operand to be a **modifiable lvalue**.
  - right operand to be an expression whose value can be assigned to the left operand.
    ```cpp
            int x {10};  // x is an lvalue
            x = 20;        // OK
    ```

---
## 1. Raw Pointer
### 1.1. Introduction
**A Pointer** is a variable that store the memory address of another variable.

```cpp
                        Memory Layout

    ┌──────────────────────────────────────────────┐
    │ Address  │ Variable │ Size    │ Stored Value │
    ├──────────┼──────────┼─────────┼──────────────┤
    │ 0x2000   │ value    │ 4 bytes │ 10           │
    │ 0x3000   │ ptr      │ 8 bytes │ 0x2000       │
    └──────────┴──────────┴─────────┴──────────────┘

                    ┌─────────────┐
 0x3000  ( &ptr ) ->│     ptr     │
                    │   0x2000    │
                    └──────┬──────┘
                           │
                           │ points to
                           V
                       ┌─────────────┐
   0x2000 (&value) --->│    value    │
                       │     10      │
                       └─────────────┘
    int value = 10; 
    int* ptr = &value;  // &value has type: int*, not "0x2000" literal
```

**Syntax:** `<type>* ptr_name`

The size of a pointer depends on the target architecture. It is typically `4 bytes` on a 32-bit system and `8 bytes` on a 64-bit system.
- `pointer-type` is a type that specifies a pointer by using an asterisk `(<type>*)`. It has to match the type of the object being pointed at.
- `address-of-operator (&var)` returns **the memory address** of its operand, but not as an address literal. Instead, it returns a pointer to the operand.
- `dereference-operator (`*`addr)` returns **the value** at a given memory address as an lvalue, used to access the object being pointed at.
<br>

- A `null-pointer` is a pointer that does not point to any object or function. Since C++11, `nullptr` should be used instead of `NULL` or `0`. The type of `nullptr` is `std::nullptr_t`.
- `wild pointer` is a pointer that has not been initialized
- `dangling pointer`: pointer that is holding the address of an object that is no longer valid
<br>

### 1.2. Const and Pointers
**`pointer-to-const (const <type> ptr_name*)`** that points to a value that cannot be modified through the pointer, but the pointer itself is not const. It may also point to non-const variables.
**`const-pointer (<type>* const ptr_name)`** is the pointer whose stored address cannot be changed after initialization, but the value at that address can be modified.
**`const-pointer-to-const(const <type>* const ptr_name)`** cannot be reseated (address fixed) and cannot modify the value it points to
> const qualifies the thing immediately to its left; if there is nothing to its left, it qualifies the thing to its right.
```cpp
int value = 10;
const int pi = 3;

const int* ptr_to_const = &pi;
*ptr_to_const = 5; // ERROR
int* const const_ptr = &value;
const_ptr = &pi; // ERROR
const int* const const_ptr_to_const = &pi;
```
<br>

### 1.3. Types of Pointers
**`pointer-to-pointer`** is a pointer that holds the address of another pointer. Using two asterisks to declare a pointer to pointer. 
- e.g. `int** ptr_to_ptr;`

**`void-pointer`** is the generic pointer, is a special type of pointer that can be pointed at objects of any data type
- It must first be cast to another pointer type before the dereference can be performed.
- Deleting this will result in undefined behavior cause we don't know its type

**function-pointer** has the the type (parameters and return type) of the function pointer must match the type of the function. `type (*name)(param..){};`
- Calling a function using function pointer:
  - Explicitly dereference: `(*fcnPtr)(5); // call function foo(5)`
  - Implicitly dereference: ` fcnPtr2(5); // call function foo(5)`
  - Can passing functions as arguments to other functions. Functions used as arguments to another function are sometimes called **callback functions**.
<br>

### 1.4. Dynamic Memory
- Memory for local variables is allocated on the stack at runtime, which has a limited size.
- Dynamic Memory allocation is performed on the heap using the `new` operator for allocation and the `delete` operator for deallocation.
- Memory leakage occurs when dynamically allocated memory is no longer accessible by the program, but the program fails to release it back to the operating system. 
    ```cpp
    int* ptr = new int(42);  // Allocate an int on the heap
    delete ptr;              // Release the memory
    ptr = nullptr;           // Avoid dangling pointer
    ```
---
## 2. Smart Pointers
- The STL includes smart pointers, which are defined in the <memory> to help ensure that programs are free of memory and resource leaks.
- Raw pointers are only used in small code blocks where performance is critical and we can control the ownership stuff.
    ```cpp
    void use_raw_pointer(){
        Song* pSong = new Song(L"Nothing on You", L"Bruno Mars"); 

        // Use pSong...

        delete pSong;   
    }

    void use_smart_pointer(){
        // Declare a smart pointer on stack and pass it the raw pointer.
        unique_ptr<Song> song2(new Song(L"Nothing on You", L"Bruno Mars"));

        // Use song2...
        string s = song2->duration_;

    } // song2 is deleted automatically here.
    ```

- After being initialized, the smart pointer owns the raw pointer.
- Use the overloaded `->` and `*` operators to access the object.
- Use `get()` to access the raw pointer directly.
- There are some kinds of the smart pointer:
    - `unique_ptr`: this is the default choice for POCO, **one owner**
    - `shared_ptr`: **multiple owners**, the raw pointer deleted when all owners have gone out of scope or have otherwise given up ownership.
    - `weak_ptr`: provides access to an object that is owned by one or more `shared_ptr`, but holds a **non-owning** reference to this object.
<br>

### 2.1. std::unique_ptr
**`std::unique_ptr`** is a smart pointer that has exclusive ownership of a dynamically allocated object.
- It can **only be moved** and cannot be passed by value, copied.
    ```cpp
    unique_ptr<Song> song_factory(const std::string& artist, const std::string& title){
        // Implicit move operation into the variable that stores the result.
        return make_unique<Song>(artist, title);
    }

    void make_songs()
    {
        // Create a new unique_ptr with a new object.
        auto song = make_unique<Song>(L"Mr. Children", L"Namonaki Uta");

        // Use the unique_ptr.
        vector<string> titles = { song->title };

        // Move raw pointer from one unique_ptr to another.
        unique_ptr<Song> song2 = std::move(song);

        // Obtain unique_ptr from function that returns by value.
        auto song3 = SongFactory(L"Michael Jackson", L"Beat It");
    }

    // Create a unique_ptr to an array of 5 integers.
    auto p = make_unique<int[]>(5);

    // Initialize the array.
    for (int i = 0; i < 5; ++i)
    {
        p[i] = i;
        cout << p[i] << endl;
    }
    ```
<br>

### 2.3. std::shared_ptr
**`std::shared_ptr`** is a smart pointer that allows multiple owners to share ownership of the same dynamically allocated object.
- It is designed for scenarios in which more than one owner needs to manage the lifetime of an object.
- It can be copied, passed by value in function arguments, and assigned to other `shared_ptr` instances.
- All the instances point to the same object, and share access to one "control block" that increments and decrements the reference count whenever a new shared_ptr is added, goes out of scope, or is reset. **When the reference count reaches zero, the control block deletes the memory resource and itself**.
    ```cpp
    // Use make_shared function when possible.
    auto sp1 = make_shared<Song>(L"The Beatles", L"Im Happy Just to Dance With You");

    // Ok, but slightly less efficient. 
    // Note: Using new expression as constructor argument
    // creates no named variable for other code to access.
    shared_ptr<Song> sp2(new Song(L"Lady Gaga", L"Just Dance"));

    // When initialization must be separate from declaration, e.g. class members, 
    // initialize with nullptr to make your programming intent explicit.
    shared_ptr<Song> sp5(nullptr);
    //Equivalent to: shared_ptr<Song> sp5;
    //...
    sp5 = make_shared<Song>(L"Elton John", L"I'm Still Standing");

    //Initialize with copy constructor. Increments ref count.
    auto sp3(sp2);

    //Initialize via assignment. Increments ref count.
    auto sp4 = sp2;

    //Initialize with nullptr. sp7 is empty.
    shared_ptr<Song> sp7(nullptr);

    // Initialize with another shared_ptr. sp1 and sp2
    // swap pointers as well as ref counts.
    sp1.swap(sp2);
    ```
<br>

### 2.4. std::weak_ptr
**`std::weak_ptr`** is a smart pointer that observes an object managed by a `std::shared_ptr` without owning it.
- It's used to to access the underlying object of a shared_ptr without causing the reference count to be incremented.
- If the memory has already been deleted, the weak_ptr's bool operator returns false.
    ```cpp
    // Source - https://stackoverflow.com/a/21877073
    // Posted by sunefred, modified by community. See post 'Timeline' for change history
    // Retrieved 2026-02-06, License - CC BY-SA 4.0
    int main() {
        // OLD, problem with dangling pointer
        // PROBLEM: ref will point to undefined data!
        int* ptr = new int(10);
        int* ref = ptr;
        delete ptr;

        // NEW
        // SOLUTION: check expired() or lock() to determine if pointer is valid

        // empty definition
        std::shared_ptr<int> sptr;

        // takes ownership of pointer
        sptr.reset(new int);
        *sptr = 10;

        // get pointer to data without taking ownership
        std::weak_ptr<int> weak1 = sptr;

        // deletes managed object, acquires new pointer
        sptr.reset(new int);
        *sptr = 5;

        // get pointer to new data without taking ownership
        std::weak_ptr<int> weak2 = sptr;

        // weak1 is expired!
        if(auto tmp = weak1.lock()){
            std::cout << "weak1 value is " << *tmp << '\n';
        } else {
            std::cout << "weak1 is expired\n";
        }
        // weak2 points to new data (5)
        if(auto tmp = weak2.lock()){
            std::cout << "weak2 value is " << *tmp << '\n';
        } else {
            std::cout << "weak2 is expired\n";
        }
    }
    ```
    > std::weak_ptr is a very good way to solve the dangling pointer problem. By just using raw pointers it is impossible to know if the referenced data has been deallocated or not. Instead, by letting a std::shared_ptr manage the data, and supplying std::weak_ptr to users of the data, the users can check validity of the data by calling expired() or lock().
    You could not do this with std::shared_ptr alone, because all std::shared_ptr instances share the ownership of the data which is not removed before all instances of std::shared_ptr are removed. Here is an example of how to check for dangling pointer using lock():

---
## 3. Reference
**Reference** is an alias for an existing object/function.
```cpp
          Memory

      Address: 0x1000
    ┌─────────────────┐
    │      value      │
    │       10        │
    └─────────────────┘
          ^     ^
          │     │
          │     │
       value   ref

value and ref refer to the SAME object

int value = 10;
int& ref = value;
```

- Declared as `<type>& reference_name`
- Any operation on the reference is applied to the object being referenced.
- All references **must be initialized** and **cannot be reseated**
- References are not objects and do not have their own size
- Can only accept modifiable lvalue arguments (const or non-const)
<br>

**pass-by-reference** allows to:
- pass arguments to a function without making copies of those arguments each time the function is called. (class types)
- change the value of an argument
<br>

**pass-by-const-reference** guaranteeing that the function can not change the value being referenced.
<br>

**lvalue-reference** just a reference for an existing lvalue.
**lvalue-reference-types** determines what type of object it can reference by using a single ampersand  `<type>&` .
**lvalue-reference-variable** is a variable that acts as a reference to an lvalue.
**lvalue-reference-to-const** can bind with const or non-const objects. `const <type>& name`

---
## 4. Pass by Value/Reference/Address
### 4.1. Pass-by-Value
- A **copy** of the argument is passed to the function.
- Modifying the parameter does not affect the original argument.
- **C++ really passes everything by value**
<br>

### 4.2. Pass-by-Reference
- The function parameter becomes an alias for the argument.
- No copy is made.
- Changes made through the reference affect the original object.
- Cannot be null.
- **Pass-by-Const-Reference** avoids copying large objects and prevents modification of the argument.
<br>

### 4.3. Pass-by-Address
- A pointer containing the object's address is passed to the function.
- No copy of the object is made.
- Changes made through the pointer affect the original object.
- Supports null checking.
- The pointer can be reseated.
<br>

---
## 5. Return by Value/Reference/Address
- **T returnValue(...)** returns a copy (or move) of the object. The caller gets its own value.
- **T& returnReference(...)** returns a reference to an existing object. The caller does not own it, so the object must outlive the reference.
- **T * returnAddress(...)** returns a pointer (an address) to an object. The caller must handle the pointer carefully (ensure it's valid and points to a live object).
<br>

**`return-by-reference`**:
- Avoids making a copy of the object.
- The referenced object must live beyond the scope of the function, otherwise the reference will dangle.
- Never return a non-static local variable or temporary by reference.
<br>

**`return-by-address`** works almost identically to return-by-reference.
**`return-by-value`** just make a copy

---
### 6. In/Out Params
**`in-parameters`** are typically passed `by value` or `by const reference`
**`out-parameters`**: A function parameter that is used only for the purpose of returning information back to the caller.
  - Avoid out-parameters (except in the rare case where no better options exist).
  - Prefer pass by reference for non-optional out-parameters.
    ```cpp
    // In-parameter by value (cheap to copy)
    void greet(std::string name) {}

    // In-parameter by const reference (avoid copy for large objects)
    int length(const std::string& text) {
        return text.size();
    }

    // Out-parameter by reference (rare case)
    void square(int input, int& output) {
        output = input * input;
    }

    int main() {
        greet("Alice");

        int value =  length(msg);

        int result;
        square(5, result);
    }
    ```