# Exceptions
Exception handling provides a mechanism for separating error handling and other exceptional conditions from the normal execution flow of a program.
## Why We Prefer Using Exceptions
- They force the caller to recognize and handle error conditions instead of allowing the program to continue incorrectly or terminate unexpectedly.
- They allow errors to propagate up the call stack until they reach a layer with sufficient context to handle them properly.
- During stack unwinding, `destructors` for objects in scope are automatically called, helping prevent resource leaks.

The exception mechanism introduces minimal performance overhead when no exception is thrown.
When an exception is thrown, the cost of stack traversal and unwinding is generally comparable to the cost of several function calls.

## Designing Exception-Safe Code
- **Low- and middle-level layers:** Catch and rethrow exceptions when there is insufficient context to handle them. This allows exceptions to propagate to higher layers where appropriate decisions can be made.
- **Highest-level layers:** If an exception remains unhandled, terminate the program gracefully (for example, by calling `exit(-1)`).

## 1. Exit Codes
An exit code (return code) is a value returned by a program to indicate its execution status to determine whether the program completed successfully or encountered an error.

### 1.1. Common Exit Codes
- `0`: Successful execution.
- Non-zero values: Indicate an error or abnormal termination.

### 1.2. Returning an Exit Code / std::exit
A program can return an exit code by returning a value from `main()` or by calling `std::exit()`.

## 1. Basic Exception Handling <except>
The `throw` expression is used to transfer control to an exception handler higher in the execution stack.
The `try` keyword defines a block of code that may generate exceptions. Any exception thrown within the `try` block can be handled by one or more associated `catch` blocks.
The `catch` keyword defines an exception handler. Each `catch` block is responsible for handling exceptions of a specific type.

TODO: https://www.learncpp.com/cpp-tutorial/stdmove_if_noexcept/

## Access Violations
```cpp
try
{
    int* ptr = nullptr;
    *ptr = 10;
}
catch (...)
{
    // Will not catch a typical access violation
}
```

### 6.3. Halts
- Halts allow us to terminate our program.
- `std::exit` is called implicitly when main() returns, it does not clean up local variables in the current function or up the call stack.
- `std::abort()` function causes your program to terminate abnormally. Abnormal termination means the program had some kind of unusual runtime error and the program couldn’t continue to run. 
- `std::terminate()` function is typically used in conjunction with exceptions . By default, it calls `std::abort()`
```cpp
#include <iostream>
#include <cstdlib>    // for std::exit, std::abort
#include <exception>  // for std::terminate

void cleanup() {
    std::cout << "Cleaning up...\n";
}

void riskyFunction(bool fatalError) {
    if (fatalError) {
        std::cout << "Fatal error occurred!\n";

        // std::abort: abnormal termination, no cleanup
        std::abort();

        // Or: std::terminate(); // usually called when exception handling fails
    }
}

int main() {
    cleanup(); // local function call

    // Example 1: return normally from main
    std::cout << "Program running normally...\n";

    // Example 2: using std::exit (implicit when main returns)
    if (false) {
        std::cout << "Exiting via std::exit...\n";
        std::exit(0); // does not call destructors of locals in main()
    }

    // Example 3: risky code that might abort/terminate
    riskyFunction(true);

    // Example 4: normal end of main calls std::exit implicitly
    std::cout << "Main returns normally.\n";
    return 0; // std::exit(0) is called implicitly here
}
```