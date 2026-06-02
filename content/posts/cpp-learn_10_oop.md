# Object Oriented Programming

**Four Pillars of OOP in C++**:
  - **Abstraction** is the `process of hiding the implementation details` and only `showing the essential details or features` to the user. It `allows to focus on` what an object `does` rather than `how it does` it.
    It is achieved using `abstract classes` (classes that have at least one pure virtual function).

  - **Encapsulation** is the `process of bundling data and methods into a single unit` (class) and `restricting direct access` to some components.
    Data is hidden and accessed through public methods.
    It is achieved using access specifiers like `private, protected, and public`.

  - **Inheritance**  is `a mechanism` where a derived class acquires the properties and behaviors of a base class, forming an `is-a` relationship.
    It improves code reuse and extensibility.
    It is implemented using `:` followed by an access specifier `public, private, protected`.

  - **Polymorphism** means `many forms`. It allows the `same interface (function or method)` to `behave differently` depending on the context.
    It is achieved through:
      - `Compile-time polymorphism`: `function overloading, operator overloading`
      - `Runtime polymorphism`: `virtual functions`

### Dynamic Polymorphism
### Virtual Methods/ Virtual Tables
### Inheritance
### Rule of Zero, Five, Three

## 1. Object Relationships
- The process of building complex objects from simpler ones is called **object composition**.
- There are two types of object composition: `composition`, and `aggregation`.
- **Composition** exists when a member of a class has a part-of relationship with the class. In a composition relationship, the class manages the existence of the members. 
  - To qualify as a composition, an object and a part must have the following relationship:
    - The part (member) is part of the object (class)
    - The part (member) can only belong to one object (class) at a time
    - The part (member) has its existence managed by the object (class)
    - The part (member) does not know about the existence of the object (class)
  - Typically implemented via normal member variables, or by pointers where the class manages all the memory allocation and deallocation. If you can implement a class as a composition, you should implement a class as a composition.

- **Aggregations** exists when a class has a has-a relationship with the member. In an aggregation relationship, the class does not manage the existence of the members. 
  - To qualify as an aggregation, an object and its parts must have the following relationship:
    - The part (member) is part of the object (class)
    - The part (member) can belong to more than one object (class) at a time
    - The part (member) does not have its existence managed by the object (class)
    - The part (member) does not know about the existence of the object (class)
  - Typically implemented via pointer or reference.

- **Associations** are a looser type of relationship, where the class uses-an otherwise unrelated object. 
  - To qualify as an association, an object and an associated object must have the following relationship:
    - The associated object (member) is otherwise unrelated to the object (class)
    -  associated object (member) can belong to more than one object (class) at a time
    - The associated object (member) does not have its existence managed by the object (class)
    - The associated object (member) may or may not know about the existence of the object (class)
  - May be implemented via pointer or reference, or by a more indirect means (such as holding the index or key of the associated object).

- In a **dependency**, one class uses another class to perform a task. The dependent class typically is not a member of the class using it, but rather is temporarily created, used, and then destroyed, or passed into a member function from an external source.

- In a **container** class one class provides a container to hold multiple objects of another type. A value container is a composition that stores copies of the objects it is holding. A reference container is an aggregation that stores pointers or references to objects that live outside the container.

- **std::initializer_lis**t can be used to implement constructors, assignment operators, and other functions that accept a list initialization parameter. 
- std::initializer_list lives in the <initializer_list> header.

|Property\Type 	|Composition 	|Aggregation 	|Association 	|Dependency|
|---|---|---|---|---|
|Relationship type 	|Whole/part 	|Whole/part 	|Otherwise unrelated 	|Otherwise unrelated|
|Members can belong to multiple classes 	|No 	|Yes 	|Yes 	|Yes|
|Members existence managed by class 	|Yes 	|No 	|No 	|No|
|Directionality 	|Unidirectional 	|Unidirectional 	|Unidirectional or bidirectional 	|Unidirectional|
|Relationship verb 	|Part-of 	|Has-a 	|Uses-a 	|Depends-on|

---
## 2. Inheritance
**Inheritance** allows us to reuse classes by having other classes inherit their members.
- Syntax: Use a colon `:` after the class declaration, the word `public` and the name of the class we wish to inherit.
- e.g.
    ```Cpp
    class Base{};

    class Derived: public Base{};
    ```
- `C++ constructs` derived classes in phases, starting with the **most-base class** (at the top of the inheritance tree) and finishing with the **most-child class** (at the bottom of the inheritance tree).

### 2.1. Constructors and Initialization of Derived Classes
- `Constructors`: the `derived class constructor` is responsible for determining which `base class constructor` is called. If no base class constructor is specified, the default base class constructor will be used.
- `Destructors:` When a derived class is destroyed, each destructor is called in the `reverse order of construction`.
- In more detail:
  - Memory for the derived class is set aside (enough for both the base and derived portions).
  - The appropriate derived class constructor is called.
  - The base class object is constructed first using the appropriate base class constructor. If no base class constructor is specified, the default constructor will be used.
  - The initialization list of the derived class initializes members of the derived class.
  - The body of the derived class constructor executes.
  - Control is returned to the caller.

### 2.2. Inheritance and Access Specifiers
- C++ defaults to **private inheritance**
- `private-inaccessible` does not affect the way that the derived class accesses members inherited from its parent! It only affects the code trying to access those members through the derived class.
  - e.g.
    ```cpp
    // Inherit from Base publicly
    class Pub : public Base {
    // Public inheritance means:
    // Public inherited members stay public (so m_public is treated as public)
    // Protected inherited members stay protected (so m_protected is treated as
    // protected) Private inherited members stay inaccessible (so m_private is
    // inaccessible)
    };

    // Inherit from Base protectedly
    class Pro : protected Base {
    // Protected inheritance means:
    // Public inherited members stay protected (so m_public is treated as
    // protected) Protected inherited members stay protected (so m_protected is
    // treated as protected) Private inherited members stay inaccessible (so
    // m_private is inaccessible)
    };

    // Inherit from Base privately
    class Pri : private Base {
    // Private inheritance means:
    // Public inherited members become private (so m_public is treated as private)
    // Protected inherited members become private (so m_protected is treated as
    // private) Private inherited members stay inaccessible (so m_private is
    // inaccessible)
    };

    class Def : Base  // Defaults to private inheritance
    {};
    ```
### 2.3. Calling and Modifying Inherited Function Behavior
- **Adding new functionality to a derived class:** A derived class can inherit the functionality of its base class and then add new functionality, modify existing functionality, or hide functionality that is not desired.
<br>

- **Calling inherited functions:** Inherited member functions can be called just like members defined in the derived class. When `derived.baseFunction()` is called, the compiler first looks for `baseFunction()` in `Derived`. If it is not found there, the compiler continues searching in the base class (`Base`). If `Base` defines `baseFunction()`, that function is used.
<br>

- **Redefining behavior:** To change how a function inherited from a base class behaves, redefine the function in the derived class.
<br>

- **Extending existing functionality:** A derived function can call the base class version of a function and then perform additional work. Use the scope resolution operator (`Base::`) to explicitly invoke the base class implementation.
<br>

- **Overload resolution in derived classes:** When a derived class declares a function with the same name as one in the base class, all base-class overloads with that name become hidden. A using-declaration such as `using Base::function;` brings the hidden overloads back into the scope of `Derived`, making them available for overload resolution. As a result, `Base::function(int)` can be selected instead of `Derived::function(double)` when calling `derived.function(5)`, if it provides a better match.
- e.g.
```cpp
// ===== Base class =====
class Base {
 public:
  void baseFunction() { cout << "Base::baseFunction()" << endl; }

  void greet() { cout << "Hello from Base!" << endl; }

  void function(int x) {
    cout << "Base::function(int) called with " << x << endl;
  }
};

// ===== Derived class =====
class Derived : public Base {
 public:
  // 1. Redefining (overriding) behavior
  void greet() { cout << "Hello from Derived!" << endl; }

  // 2. Adding to existing functionality
  void greetWithBase() {
    cout << "Derived part first -> ";
    Base::greet();  // call the Base version explicitly
  }

  // 3. Hiding base function by defining same name
  void baseFunction() { cout << "Derived::baseFunction()" << endl; }

  // 4. Overload resolution
  void function(double x) {
    cout << "Derived::function(double) called with " << x << endl;
  }

  // Bring Base::function(int) into scope for overload resolution
  using Base::function;
};

int main() {
  Derived d;

  cout << "\n--- Calling inherited function ---" << endl;
  d.Base::baseFunction();  // explicitly call base version
  d.baseFunction();        // calls Derived::baseFunction()

  cout << "\n--- Redefining behavior ---" << endl;
  d.greet();  // calls Derived version

  cout << "\n--- Adding to existing functionality ---" << endl;
  d.greetWithBase();  // calls Derived + Base

  cout << "\n--- Overload resolution ---" << endl;
  d.function(10);    // selects Base::function(int)
  d.function(3.14);  // selects Derived::function(double)

  return 0;
}
```

### 2.4. Hiding Inherited Functionality
**Changing an inherited member's access level:** A derived class can change the access level of an inherited member by using a `using` declaration under a different access specifier. The member keeps its original behavior, but its accessibility is changed within the derived class.
- This technique is commonly used to:
  - Expose a protected base-class member as public.
  - Restrict a public base-class member to protected or private access.
<br>

**Hiding functionality:** A derived class can hide inherited functionality by changing the access level of inherited members, preventing them from being accessed through objects of the derived class.
<br>

**Deleting functions in the derived class:** A derived class can mark inherited member functions as deleted using the `= delete` specifier. This prevents those functions from being called through objects of the derived class.
- Even when a function is deleted in the derived class, the base class version can still be called by explicitly qualifying the function with the base class name or by upcasting the derived object to the base type.
- e.g.
```cpp
#include <iostream>

class Base {
 public:
  int m_value{};

  Base(int value) : m_value{value} {}

  int getValue() const { return m_value; }

 protected:
  void printValue() const { std::cout << m_value << '\n'; }
};

class Derived : public Base {
 private:
  using Base::m_value;  // public -> private

 public:
  using Base::printValue;  // protected -> public

  Derived(int value) : Base{value} {}

  int getValue() const = delete;  // disable inherited function
};

int main() {
  Derived derived{7};

  // std::cout << derived.m_value; // error: m_value is private in Derived

  Base& base{derived};
  std::cout << base.m_value << '\n';  // okay: still public in Base

  derived.printValue();  // okay: made public via using declaration

  // std::cout << derived.getValue(); // error: deleted in Derived

  std::cout << derived.Base::getValue() << '\n';  // okay

  std::cout << static_cast<Base&>(derived).getValue() << '\n';  // okay

  return 0;
}
```

### 2.5. Multiple inheritance
- C++ provides the ability to do multiple inheritance. Multiple inheritance enables a derived class to inherit members from more than one parent.
- Avoid multiple inheritance unless alternatives lead to more complexity. (**diamond problem**.)
    ```cpp
    /// @brief The Diamond Problem
    //        Person
    //       /      \
    //  Employee   Student
    //       \      /
    //    TeachingAssistant
    class Person {
    public:
    void print() {}
    };

    class Employee : public Person {};

    class Student : public Person {};

    class TeachingAssistant : public Employee, public Student {};

    /// @brief Solution: Virtual Base Classes
    class Person {
    public:
    void print() {}
    };

    class Employee : virtual public Person {};
    class Student : virtual public Person {};

    class TeachingAssistant : public Employee, public Student {};
    ```

---
## 3. Polymorphism
**Polymorphism** refers to the ability of an entity to have multiple forms (the term "polymorphism" literally means "many forms").
- **Compile-time polymorphism** refers to forms of polymorphism that are resolved by the compiler. These include function overloading and template resolution.
- **Runtime polymorphism** refers to forms of polymorphism that are resolved at runtime. This is primarily achieved through virtual functions.

### 3.1. Pointers and References to Base Classes
**Pointers, references, and derived classes:** We can not only assign `Derived*` pointers and `Derived&` references to derived objects, but also assign `Base*` pointers and `Base&` references to derived objects. This is known as **upcasting** and happens implicitly.

> A `Base*` or `Base&` can only directly access members that exist in `Base`.
>
> If a member function is virtual, calling it through a `Base*` or `Base&` will invoke the most-derived override at runtime.
>
> A derived object contains a base-class subobject, so a `Base*` can safely point to the base portion of a `Derived` object.
>
> The compiler adjusts the pointer or reference as necessary to refer to the base-class subobject.

- **Using pointers and references to base classes:** A base-class pointer or reference can refer to any object derived from that base class, allowing a single interface to work with multiple derived types.
- However, calls made through a base-class pointer or reference use the base-class version of a function unless that function is declared `virtual`.

### 3.2. Virtual Functions and Runtime Polymorphism
A **virtual function** is a special type of member function that, when called through a base-class pointer or reference, resolves to the **most-derived override** for the actual type of the object at runtime.
  - To make a function virtual, place the `virtual` keyword before the function declaration in the base class.
  - A derived function is considered an **override** if it has the same signature (name, parameter types, cv-qualifiers, and ref-qualifiers) and a compatible return type as the virtual function in the base class.
  - Virtual functions use **late binding** (dynamic dispatch), meaning the function call is resolved at runtime.
  - Non-virtual functions use **early binding**, meaning the function call is resolved at compile time.
<br>

- **Return types of virtual functions:** The return type of a virtual function and its override must match, except for **covariant return types**, where an override may return a pointer or reference to a more-derived type.

- **Do not call virtual functions from constructors or destructors:** During construction and destruction, virtual dispatch is restricted to the class currently being constructed or destroyed. Calls will not dispatch to more-derived overrides.
- e.g.
```cpp
#include <iostream>
#include <string_view>

class Base {
 public:
  virtual std::string_view getName() const {
    return "Base";
  }                           // note addition of virtual keyword
  virtual ~Base() = default;  // virtual destructor
};

class Derived : public Base {
 public:
  virtual std::string_view getName() const { return "Derived"; }
};

int main() {
  Derived derived{};
  Base& rBase{derived};
  std::cout << "rBase is a " << rBase.getName() << '\n';

  return 0;
}

// RESULT: rBase is a Derived
```

### 3.3. The `override` and `final` Specifiers, and Covariant Return Types
#### 3.3.1. The `override` specifier
The `override` specifier can be applied to a virtual function in a derived class to tell the compiler to verify that the function is actually overriding a virtual function from a base class.
  - The `override` specifier is placed at the end of the function declaration.
  - If a member function is `const` and an override, the `const` qualifier must appear before `override`.
  - Using `override` is recommended for all overriding functions because it helps catch mistakes at compile time.
  - The `virtual` keyword is typically used only in the base-class declaration. Overriding functions should use `override` instead.
<br>

#### 3.3.2. The `final` specifier
The `final` specifier can be used to prevent a virtual function from being overridden in further-derived classes.
  - It is placed in the same location as the `override` specifier.
  - A class can also be marked `final` to prevent inheritance.
<br>

#### 3.3.3. Covariant return types:
Normally, an overriding function must have the same return type as the virtual function it overrides. However, C++ allows an override to return a pointer or reference to a more-derived type. This is known as a **covariant return type**.
  - Covariant return types are only allowed for pointers and references.
  - Value return types must match exactly.
<br>

e.g.
```cpp
class Animal {
 public:
  virtual ~Animal() = default;

  virtual Animal* clone() const { return new Animal(*this); }

  virtual void speak() const { std::cout << "Animal\n"; }
};

/// @brief final class: cannot be inherited from
class Dog final : public Animal
{
 public:
  /// @brief override + covariant return type
  Dog* clone() const override { return new Dog(*this); }

  /// @brief  override + final function: cannot be overridden further
  void speak() const override final { std::cout << "Woof\n"; }
};

int main() {
  Dog dog{};

  Animal* animal{&dog};

  animal->speak();  // calls Dog::speak()

  Animal* copy{animal->clone()};  // actually returns a Dog*

  delete copy;

  return 0;
}
```

### 3.4. Virtual Destructors and Ignoring Virtualization
- **Virtual destructors:** If a class is intended to be used polymorphically, its destructor should generally be declared `virtual`. This ensures that deleting a derived object through a base-class pointer correctly calls the entire destructor chain.
- **Ignoring virtualization:** Virtual dispatch can be bypassed by explicitly qualifying the function with the class name and scope resolution operator (`::`).
- e.g.
```cpp
#include <iostream>
#include <string_view>

class Base {
 public:
  /// @brief Virtual destructor
  virtual ~Base() { std::cout << "Calling ~Base()\n"; }
  virtual std::string_view getName() const { return "Base"; }
};

class Derived : public Base {
 private:
  int* m_array{};

 public:
  Derived(int length) : m_array{new int[length]} {}

  ~Derived() override {
    std::cout << "Calling ~Derived()\n";
    delete[] m_array;
  }

  std::string_view getName() const override { return "Derived"; }
};

int main() {
  Derived derived{5};
  Base& baseRef{derived};

  // Virtual dispatch
  std::cout << baseRef.getName() << '\n';  // Derived

  // Ignore virtualization
  std::cout << baseRef.Base::getName() << '\n';  // Base

  Base* basePtr{new Derived{5}};
  delete basePtr;  // Calls ~Derived() then ~Base()

  return 0;
}

/// Output:
// Derived
// Base
// Calling ~Derived()
// Calling ~Base()
```

### 3.5. Early Binding and Late Binding

