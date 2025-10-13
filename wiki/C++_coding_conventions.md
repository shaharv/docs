# C++ Coding Conventions

This page proposes C++ coding conventions.
For enforcing conventions automatically, use clang-tidy version 18 and later (for static analysis and naming conventions) and clang-format version 18 and later (for code formatting).

## C++ standard and Compilers

The recommended standard is C++20. Feel free to use any C++20 features in your code.

Note: libc++ (LLVM) currently doesn't support several C++ standard library features. Known missing features:
- `std:derived_from` (concepts)
- `std::jthread` (jthread)

## Code formatting

Code is formatted using clang-format. The `.clang-format` file lists the formatting rules.

The recommended formatting rules are:
- Indentation using spaces, never with tabs. Indent size is 4 spaces.
- Maximum line length is 120 characters.
- Always use block braces, even if block body is only 1 line.
- Open block brace is always followed by a newline (i.e. `if (a) { func(); }` is forbidden).
- `#include`'s must be sorted alphabetically and separated to project includes, 3rd party includes and system includes.

## Naming conventions

The naming conventions are listed in the `.clang-tidy` file.

The following naming rules apply:
- Type names - classes, structs, unions: using **CamelCase**
- Function/method names: using **camelBack**
- Variables, data members: using **camelBack**
- Private data members: using **camelBack** with underscore suffix. For example: `eventCounter_`.
- Namespaces: using **snake_case**.
- Constants: using **UPPER_CASE**.
- Boolean variables naming:
  * The name **isOk** should be used for Boolean variables that hold success or failure status of an operation.
  * Otherwise, Boolean variables should start with the is prefix.

## Namespaces

- Never add `using namespace` in header files, for not polluting the namespace of the includers. Always fully qualified type name in header files.
- Never use `using namespace std`, not even in source files. If needed, import individual symbols, for example `using std::cout`. Importing the entire std namespace is overkill and could easily create ambiguities.

## Variable declarations

- When declaring simple variables (such as scalars and `std::string`), prefer initializing using assignment operator, and not using function-like brackets. For example:
  ```
  int32_t myInt = 5;              // good
  std::string myString = "hello"; // good
  int myInt(5);                   // bad
  std::string myString("hello");  // bad
  ```
- For other cases, prefer `{}` initializer syntax.
- References:
  * https://www.cppstories.com/2022/init-string-options
  * https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es23-prefer-the--initializer-syntax

## Integer types

- Prefer using `<cstdint>` integer types with explicit sizes, such as `int32_t`, `uint32_t`, `int64_t` and `uint64_t`. Using these types improves code readability and is more portable than `int`, `unsigned`, `long long`, etc.

## Default Parameters

- Refrain from using C++ default parameters as much as possible. Giving function parameters default values may be error prune:
  * When adding a parameter with a default value to an existing function, we may miss function calls that should have been updated, and should not get the default parameter value.
  * Having to specify the parameter value improves maintainability - it forces the programmer to assign a thought-of value, and not a default which might not be suitable for all cases.

## Enumerations

- Use C++ `enum class` for defining enumerations. Avoid using C enums, or groups of static integer constants. A C++ enum class is a type-safe alternative to a C enum and offers the following benefits:
  * Requires explicit cast to integer, which helps to avoid coding errors and misuse.
  * Enumerator names are not exported to the global namespace (which otherwise could cause name clashes).
  * The underlying type of a C enum cannot be specified, causing confusion, compatibility problems, and makes forward declaration impossible.
  * For more details see: https://stackoverflow.com/questions/18335861/why-is-enum-class-considered-safer-to-use-than-plain-enum

## Const correctness

- Never return const by-value. 
  * For further reading: https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#f49-dont-return-const-t 
  * Examples:
  ```
  // BAD: returning a const scalar is meaningless; const is ignored by the compiler
  const int foo() { return val; } 

  // BAD: returning a const object interferes with rvalue optimization and move semantics
  const std::string foo() { return "boo"; }

  // GOOD: returning a const reference or const pointer
  const MyObj& foo() { return myObj; }
  const MyObj* const foo() { return &myObj; }
  ```
- Prefer using `static constexpr` for integer constants. For example:  
  ```
  static constexpr int32_t FOO = 5;
  ```

## Unit tests

- Avoid using random values in tests. While random values may increase test coverage over time, they result with unstable test execution, and could result with productivity loss (investigating non-issues).
- Naming unit tests:
  * Whitespace characters and special characters are forbidden. Use only alphanumeric characters, `_` and `-` as delimiters.
  * Test name should be informative and concise.
- In Catch2 test files, do not define global functions - for avoiding collisions between different files when there are same named functions in different source files that are linked into the same Catch2 tests executable. If helper functions are needed, define them in file scope by using anonymous namespaces.
- Do not use Catch2 `REQUIRE` checks in destructors, as they throw exception if they fail (throwing exception from a destructor could result with undefined behavior).

## Included headers

- Every .cpp file should directly include the header files which are needed. For example, if `std::string` is used in the .cpp file, the header `<string>` should be included (and not rely on it being included by another header file). This rule is enforced automatically by clang-tidy's `misc-include-cleaner` check.
- System headers and third-party headers should be included using `<>` while user (implementation) headers should be included with `""`. In the CMake project it's useful to mark the third party libraries as "system" (which affect the compiler include search paths).
- Headers should be organized in include groups according to the following order:
  * The interface header for this file, if any. For example, `Foo.cpp` should include `Foo.h`.
  * Any headers required for the implementation, including auto-generated headers such as protobuf headers (that are not system headers or third party headers).
  * Third party headers, such as Arrow, spdlog, etc.
  * System headers.
- Each include group should be separated by a newline.
- Includes in each include group should be sorted. This rule is enforced automatically by clang-tidy's `llvm-include-order` check.
