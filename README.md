# Rust Programming Notes

DONT FOTGET TO INSTALL rust Clippy

# Table of Contents

1. [Chapter 1: Rust Basics](#chapter-1-rust-basics)
2. [Chapter 2: Variables and Mutability](#chapter-2-variables-and-mutability)
   - [Data Types](#data-types)
3. [Chapter 3: Functions and Control Flow](#chapter-3-functions-and-control-flow)
4. [Chapter 4: Borrowing and Ownership](#chapter-4-borrowing-and-ownership)
   - [The Stack and Heap](#the-stack-and-heap)
   - [Move, Copy, and Clone](#move-copy-and-clone)
   - [Ownership and Functions](#ownership-and-functions)
   - [Rules of Borrowing and Ownership](#rules-of-borrowing-and-ownership)
   - [Referencing and Borrowing](#referencing-and-borrowing)
   - [Mutable References and Borrowing Rules](#mutable-references-and-borrowing-rules)
   - [Dangling References](#dangling-references)
   - [The Slice Type](#the-slice-type)
   - [Chapter 4 Summary](#chapter-4-summary)
5. [Chapter 5: Structs in Rust](#chapter-5-structs-in-rust)
6. [Chapter 6: Enums and Pattern Matching](#chapter-6-enums-and-pattern-matching)
7. [Chapter 7: Managing Growing Projects with Packages, Crates, and Modules](#chapter-7---managing-growing-projects-with-packages-crates-and-modules)
   - [Introduction: Why Organize Code?](#1-introduction-why-organize-code)
   - [Packages and Crates](#2-packages-and-crates)
   - [Modules: Organizing Code Inside a Crate](#3-modules-organizing-code-inside-a-crate)
   - [Module File Structure in Rust](#4-module-file-structure-in-rust)
   - [Controlling Scope & Visibility with pub](#5-controlling-scope--visibility-with-pub)
   - [Paths and the use Keyword](#6-paths-and-the-use-keyword)
   - [Summary of Key Concepts](#summary-of-key-concepts)
8. [Chapter 8: Common Collections in Rust](#chapter-8---common-collections-in-rust)
   - [Vectors (`Vec<T>`)](#2-storing-lists-of-values-with-vectors-vect)
   - [Strings (`String`)](#storing-utf-8-encoded-text-with-strings)
   - [Hash Maps (`HashMap<K, V>`)](#storing-keys-with-associated-values-in-hash-maps)
   - [Chapter 8 Summary](#chapter-8-summary---common-collections-in-rust)
9. [Chapter 9: Error Handling in Rust](#chapter-9---error-handling-in-rust)
10. [Chapter 10: Generic Types, Traits, and Lifetimes](#chapter-10-generic-types-traits-and-lifetimes-in-rust)
11. [Chapter 11: Traits in Rust](#understanding-traits-in-rust)
12. [Chapter 12: Writing Automated Tests in Rust](#chapter-12---writing-automated-tests-in-rust)

```md

---

**Rust need to know all types at compile time**

## 1. **Build Time vs Compile Time vs Run Time – What's the Difference?**

### **Build Time** (Everything before execution)
This is a general term that includes all steps involved in **producing the final executable program**, before you actually run it.

In Rust, `cargo build` handles **build time**. This includes:

- **Compiling** your Rust code into machine code (`.exe`, `.out`, etc.)
- **Linking** with external libraries (for example, when you use `rand`, `serde`, or the standard library).
- **Packaging** the final binary (the output file you run).

Build time is a broad umbrella.

---

### **Compile Time** (A phase within build time)
This is a **specific part** of build time. It’s when the compiler (Rust’s `rustc`) takes your `.rs` source files and checks them **for correctness** and **transforms them into lower-level instructions**.

The Rust compiler performs several important checks during this time:

1. **Syntax & Grammar**  
   Does your code follow the rules of the language?

2. **Type Checking**  
   Are your variables and functions used with the correct types?
   - Example: Adding a number to a string? That’s invalid in Rust.
     ```rust
     let x = 5 + "hello"; // Compile-time error
     ```
Rust sees an equal sign, it always evaluates the right hand side first.

And if the right hand side consists of a block, Rust is going to execute the block.

And then whatever is the final value that is produced is what it's going to assign to the variable on the left

3. **Borrow Checker**  
   Rust ensures memory safety using its ownership system.
   - It checks that references (`&x`, `&mut x`) are valid and safe.
   - You cannot use a variable after moving it or while it's borrowed elsewhere.

4. **Monomorphization** (when using generics)
   - If you use generics (like `fn print<T>(value: T)`), Rust **generates a concrete version** of that function for every type you use it with.
   - This means Rust code is zero-cost: generics don’t add overhead at runtime.

If there’s any problem during compile time, **Rust refuses to build the binary**. No bugs sneak into runtime due to type or ownership errors.

---

### **Run Time** (After successful compilation)
This is when you **run the binary** file that was created by the compiler.

- Your program logic (like `main()`) executes.
- It reads files, gets user input, makes API calls, etc.
- If anything goes wrong here (like trying to read a missing file), that’s a **runtime error**.

Runtime is real-world behavior. Compile time is like a safety inspection.

---

## 2. **Rust Execution Flow (What Happens in Order?)**

| Step | Phase | What Happens |
|------|-------|---------------|
| 1.   | You write code | In `main.rs`, etc. |
| 2.   | **Build time** (cargo build) | Syntax checks, type checks, borrow checking |
| 3.   | **Compile time** | Code turns into machine code (`.exe`, etc.) |
| 4.   | **Run time** | Program is executed; logic runs |

---

## 3. **What is a "Statically Typed Language"?**

> Rust is a **statically typed** language.

### What does this mean?

- **All types are known at compile time.**
- You can’t have a variable that changes from an integer to a string at runtime.
- The **compiler must know the type** of every variable, function parameter, and return value **before** it generates the final executable.

### For example:

```rust
let x: i32 = 5;
let y: String = "hello".to_string();
```

Here, Rust knows:
- `x` is an `i32` (a 32-bit integer)
- `y` is a `String` (owned string type)

---

### If you try this:
```rust
let x: i32 = "oops"; // Compile-time error!
```
Rust says: "You told me `x` is an integer, but you're giving me a string. That's invalid."

---

### Contrast: **Dynamically Typed Languages**
Languages like **Python** or **JavaScript** don’t check types until the program runs.

```python
x = 5
x = "hello"  # Totally fine in Python
```

In these languages:
- You can assign a number to `x`, and later reassign a string.
- The language does not enforce types — errors happen **during execution**, not before.

---

## So Why Does Rust Need Types Before Compilation?

Because Rust **generates machine code specific to the types** you use. This gives you:

1. **Speed** – There’s no type guessing at runtime. Rust binaries are super fast.
2. **Safety** – Many bugs (nulls, type mismatches, memory leaks) are caught **before** the program runs.
3. **Optimization** – Knowing the exact types allows the compiler to optimize your code heavily.

This also explains why:
- You need to tell Rust what type your variable is (or let it infer it once and lock it in).
- Rust can’t compile if types are ambiguous or misused.

---

# Chapter 1: Rust Basics

## File Naming
Rust enforces a consistent naming style using **snake_case** for filenames, functions, and variables, following the community's conventions to improve readability and maintain consistency with built-in libraries.  

- **Why?**  
  - Snake case (e.g., `my_file.rs`) is preferred because it improves readability and aligns with Rust’s conventions for variables and function names.  
  - Consistency in naming helps when using tools like linters and formatters, which expect a uniform style across files.  
- **Tip:** Avoid using camelCase (e.g., `myFile.rs`) since it does not match the idiomatic style in Rust and may lead to confusion when collaborating with other developers.  

## Formatting Code
Proper formatting improves code readability and maintainability.  

- **Tool:**  
  - Rust provides the `rustfmt` tool to automatically format your code.  
- **Usage:**  
  ```bash
  rustfmt main.rs
  ```
- **Explanation:**  
  - This command automatically applies Rust’s style guidelines, ensuring that your code has consistent indentation, spacing, and line breaks. This not only makes your code look neat but also reduces the cognitive load when reading or reviewing code.  

## Macros(Not finished, see next chapters)
Macros in Rust differ from functions and have unique behavior.  

- **Key Point:**  
  - Macros are identified by the exclamation mark (`!`) and are expanded at compile time.  
- **Example:**  
  ```rust
  println!("Hello, world!");  // println! is a macro
  ```
- **Explanation:**  
  - Unlike regular functions, macros can accept a variable number of arguments and perform meta-programming tasks. They allow you to generate code during compilation, which can lead to more efficient runtime performance. However, they can also make error messages less straightforward, so understanding their expansion is crucial.  

## Cargo – Rust’s Package Manager
Cargo is Rust’s package manager and build system, comparable to `npm` in JavaScript or `pip` in Python. However, unlike those, Cargo also handles compilation, testing, and project metadata management in a unified tool.  

- **Benefits:**  
  - It streamlines building, testing, and managing your project, making it easier to maintain code and share it with others.  

## Crates vs Packages
Understanding the distinction between a crate and a package is fundamental.  

- **Crate:**  
  - A crate is the smallest unit of code compilation in Rust. It can be either a **library crate** (for reuse) or a **binary crate** (executable program).  
- **Package:**  
  - A package is a collection of one or more crates. A package can include both a library crate and one or more binary crates.  

- **Expanded Insight:**  
  - This distinction helps organize your code, especially when your project grows. Packages allow you to bundle related functionality together while still keeping separate concerns isolated in different crates.  

## Common Cargo Commands
Cargo provides a set of commands that make managing a Rust project straightforward:  

```bash
cargo new my_app      # Creates a new Rust project with the standard directory structure.
cargo build           # Compiles the project in debug mode.
cargo run             # Builds (if necessary) and runs the project.
cargo check           # Analyzes your code for errors without producing an executable binary.
cargo build --release # Builds an optimized binary suitable for production.
```
'
rustc --explain E0004 (E0004 is a error code every error have a code)
- **Explanation:**  
  - These commands help you quickly iterate on your code. For example, `cargo check` is particularly useful during development because it is faster than a full build, allowing you to catch errors early.

## Project Structure (from `cargo new my_app`)
When you create a new project using Cargo, it sets up a standard directory layout:  

```
my_app/
├── Cargo.toml  # Contains project metadata, dependencies, and configuration.
├── Cargo.lock  # Automatically managed file that records exact dependency versions.
└── src/
    └── main.rs  # The main source file where the execution of your program begins.
```
- **Cargo.toml Example:**
  ```toml
  [package]
  name = "hello_cargo"
  version = "0.1.0"
  edition = "2021"

  [dependencies]
  ```
- **Insight:**  
  - This structure helps keep your project organized. `Cargo.toml` defines the project's identity and dependencies, while the `src/` directory contains your Rust source files. The `Cargo.lock` file ensures that your builds are reproducible by locking dependency versions.  

- **Compiled Output:**  
  - The compiled binaries are placed in:
    - `target/debug/` for regular builds.
    - `target/release/` for optimized builds produced with `cargo build --release`.  

---

## Chapter 2: Variables and Mutability

### Variables & Mutability
Rust’s approach to variables emphasizes immutability by default.
- **Immutable by Default:**  
  ```rust
  let x = 5; // Immutable variable; its value cannot be changed.
  ```
- **Making Variables Mutable:**  
  ```rust
  let mut x = 5; // Mutable variable; its value can be modified later.
  ```
- **Explanation:**  
  - This design choice encourages safe programming practices by preventing unintended changes to data. By requiring explicit opt-in for mutability, Rust helps developers avoid bugs that are common in languages where variables are mutable by default.

### Constants
Constants are similar to immutable variables but with key differences.
- **Definition:**  
  - Declared with the `const` keyword.
  - Must have a type annotation.
  - Their values must be known at compile time.
- **Naming Convention:**  
  - Use ALL_CAPS_WITH_UNDERSCORES for clarity.
- **Example:**
  ```rust
  const SECONDS_IN_MINUTE: u32 = 60;
  ```
- **Expanded Explanation:**  
  - Constants differ from immutable variables in that they are inlined wherever they are used and cannot be changed at runtime. This guarantees that a constant remains the same throughout the program, which is ideal for values like conversion factors or configuration parameters.

### Shadowing
Allows redeclaring a variable with the same name. Each shadowed variable replaces the previous one within its scope. Use let again to shadow a variable. Unlike mut, shadowing allows type changes.
- **Example:**
  ```rust
  fn main() {
      let x = 5;  
      let x = x + 1; // Shadowing x, which now becomes 6.

      {
          let x = x * 2; // Inside this block, x is 12.
          println!("Inner x: {x}");
      }

      println!("Outer x: {x}"); // Outside the block, x remains 6.
  }
  ```
- **Explanation:**  
  - Shadowing lets you transform a value while keeping the same variable name. It’s especially useful when you want to convert a variable’s type (for example, from a string to its length as an integer) without introducing a new variable name. This mechanism also ensures that the previous value is completely replaced, which can prevent accidental misuse of outdated data.

### Shadowing vs. Mutability

**Using Shadowing:**
```rust
let spaces = "   ";  // Initially a string.
let spaces = spaces.len();  // Shadowed variable becomes a number.
```
- **Explanation:**  
  - With shadowing, you effectively create a new variable that replaces the old one. This allows for a change in data type, which cannot be achieved with mutable variables alone.

**Using `mut` (Error):**
```rust
let mut spaces = "   ";  
spaces = spaces.len();  // Error: The type cannot change.
```
- **Key Point:**  
  - Mutability only permits the modification of the value without altering its type. When you need to change the type, shadowing is the appropriate approach.

- **When to Use Shadowing:**  
  - Use shadowing when you need to perform a transformation on a variable while preserving its name and later use its updated value.
  - It allows you to modify the variable’s type if necessary, making your code more flexible.

---

### Data Types

#### 1) Scalar Types (Single Value)
Rust has four primary scalar types that represent a single value:
- **Integers:** Include types such as `i8`, `u8`, `i16`, `u16`, etc.
- **Floating-point Numbers:** `f32` and `f64`(is default type for floating-type).
- **Booleans:** Represent true or false values.
- **Characters:** Represent a single Unicode character. (string with '' and not "")

##### Integer Types
- **Details:**
  - **Signed integers** can represent both negative and positive values.
  - **Unsigned integers** represent only positive values (including zero).
  
| Length   | Signed  | Unsigned | Memory (Bytes) |
|----------|---------|----------|----------------|
| 8-bit    | `i8`    | `u8`     | 1 byte         |
| 16-bit   | `i16`   | `u16`    | 2 bytes        |
| 32-bit   | `i32`   | `u32`    | 4 bytes        | --> i32 is the default integer type 
| 64-bit   | `i64`   | `u64`    | 8 bytes        |
| 128-bit  | `i128`  | `u128`   | 16 bytes       |
| Arch     | `isize` | `usize`  | 4 or 8 bytes¹  |


- **Explanation:**  
  - Choosing the correct integer type is important for performance and memory usage, because 8 bit is 1 byte and if you need number maximum positive number 255 you can choose u8(0-255) and not u16(0-66,535) which will take double memory size. The table above helps you understand the trade-offs between different sizes and whether you need signed or unsigned values.

##### What is `isize` and `usize`?

- These types are **architecture-dependent**:
  - On **32-bit systems**, they are **4 bytes** (like `i32`/`u32`)
  - On **64-bit systems**, they are **8 bytes** (like `i64`/`u64`)
- They're primarily used for indexing collections (like arrays, slices, vectors) or representing sizes in memory because they match the pointer size of the platform.
- **Use case:** When you call `.len()` on a vector or access `arr[index]`, the index will usually be a `usize`.

> Always use `usize` or `isize` when dealing with memory offsets, indexing, or sizes of data structures.


##### Integer Literals in Rust

| Number Type    | Example         |
|----------------|-----------------|
| Decimal        | `98_222`        |
| Hexadecimal    | `0xff`          |
| Octal          | `0o77`          |
| Binary         | `0b1111_0000`   |
| Byte (u8 only) | `b'A'`          |

- **Insight:**  
  - The use of underscores (e.g., `98_222`) improves readability for large numbers. Rust also supports multiple numeral systems, allowing you to define values in the format that best suits your context.

#### 2) Floating-Point Types
- **Overview:**
  - Rust offers two floating-point types: `f32` for 32-bit precision and `f64` for 64-bit precision. By default, floating-point literals are interpreted as `f64`.
  
```rust
fn main() {
    let x = 2.0; // f64 by default.
}
```
- **Explanation:**  
  - Floating-point numbers in Rust are always signed and are used when precise mathematical computations are needed. The default to `f64` ensures higher precision in most calculations, reducing potential rounding errors.

#### 3) The Character Type
- **Overview:**
  - The `char` type in Rust is designed to represent a single Unicode scalar value. This means it can store not only ASCII characters but also characters from various languages and even emojis.
  
```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // With explicit type annotation.
}
```
- **Explanation:**  
  - Unlike many languages where characters are represented as integers, Rust’s `char` type is four bytes in size, allowing it to represent a wide range of characters. This makes Rust especially powerful for internationalized applications.

#### 4) Compound Types
Rust provides two compound types:
- **Tuple**
- **Array**

##### The Tuple Type (Stack)
- **Definition:**  
  - A tuple groups values of different types together into a single compound type. Once created, the length of a tuple is fixed, it cannot grow of shrink.
  
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```
- **Accessing Values:**
  - **Destructuring:**  
    ```rust
    let (x, y, z) = (500, 6.4, 1);
    ```
  - **Indexing:**  
    ```rust
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = tup.0; // Access first element.
    ```
- **Explanation:**  
  - Tuples are useful when you want to return multiple values from a function. Their fixed length and ability to contain different types make them a versatile tool in Rust programming.

##### The Array Type (Stack)
- **Definition:**  
  - An array is a collection of element.elements must have the same type. Fixed length (unlike vectors). Stored on the stack (faster than heap allocation).
  
```rust
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
}
```
- **Accessing Values:**
  ```rust
  fn main() {
      let a = [10, 20, 30, 40, 50];
      let first = a[0];  // Access the first element.
  }
  ```
- **Mutable Arrays & Tuples:**
  - Arrays can be made mutable if you need to change their elements.
    ```rust
    let mut numbers = [1, 2, 3, 4, 5];
    numbers[0] = 10;
    ```
  - Tuples can also be mutable if declared with `mut`.
    ```rust
    let mut tup = (1, 2, 3);
    tup.0 = 100;
    ```
- **Explanation:**  
  - Arrays are best when you need a fixed-size collection of elements, whereas tuples are best when you have a fixed grouping of different types. Both are stored on the stack, which means they offer very fast access times compared to heap-allocated structures.

##### The Vector Type (Heap)
- **Definition:**  
  - A vector (`Vec<T>`) is a dynamic, resizable array stored on the heap, it is possible to shrink or grow. All elements must be the same type.
  
```rust
fn main() {
    let mut numbers: Vec<i32> = Vec::new(); // Creates an empty vector.
    numbers.push(10);  // Adds an element.
    numbers.push(20);
    println!("{:?}", numbers); // Output: [10, 20]
}
```
- **Explanation:**  
  - Vectors are ideal when the number of elements is not known at compile time or may change during program execution. Because vectors are allocated on the heap, they offer flexibility at the cost of slightly slower access compared to arrays. Their dynamic nature makes them widely used in many Rust applications.

---

## Chapter 3 Functions and Control Flow

### Functions & Parameters
Functions in Rust encapsulate reusable code.
- **Definition:**
  - Declared using the `fn` keyword.
  - Can take parameters and return values.
  
**Example:**
```rust
fn greet(name: &str) {
    println!("Hello, {name}!");
}
```
- **Expanded Explanation:**
  - Functions help break down your program into smaller, manageable pieces. They promote code reuse and make your program easier to understand and maintain. Parameters allow functions to operate on different inputs, and their return values let functions communicate results back to the caller.

### Statements & Expressions
Rust differentiates between statements and expressions:
- **Statements:**  
  - Perform an action (e.g., variable assignments) but do not return a value.
- **Expressions:**  
  - Evaluate to a value and are used within functions, also can be used as part of larger expressions.
  
**Example:**
```rust
let x = 5; // A statement.
let y = {
    let z = 3;
    z + 1 // An expression that evaluates to 4.
}; // y becomes 4.
```
- **Explanation:**  
  - Understanding the difference between statements and expressions is essential in Rust. Expressions can be nested inside other expressions, making Rust’s syntax very powerful and concise. This design is used throughout Rust, from simple arithmetic to control flow constructs.

### Functions with Return Values
- **Definition:**  
  - Functions can return values using the `->` syntax. The last expression in the function is automatically returned if it is not terminated by a semicolon.
  
**Example:**
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b // This expression is returned.
}
```
- **Expanded Explanation:**  
  - Returning values from functions in Rust follows a clear and consistent rule: if the last line of a function is an expression without a semicolon, that value becomes the function’s return value. This feature helps prevent common errors and makes the control flow explicit.

- **Note on Early Returns:**  
  - You can also use the `return` keyword to exit a function early, which immediately returns the specified value.
  ```rust
  fn always_five() -> i32 {
      return 5;
  }
  ```

### Control Flow
Rust provides several control flow constructs to direct program execution.

#### if Statement
- **Usage:**
  ```rust
  fn main() {
      let number = 5;
      if number > 0 {
          println!("Positive number");
      } else {
          println!("Negative number or zero");
      }
  }
  ```
- **Expanded Explanation:**  
  - The `if` statement allows your program to make decisions based on Boolean conditions. It ensures that only the code block corresponding to the true condition is executed, which is fundamental for branching logic in programs.

#### Loops
Rust supports three primary loop constructs: `loop`, `while`, and `for`.

##### `loop` (Infinite Loop)
- **Definition:**  
  - The `loop` keyword creates an infinite loop that continues until explicitly broken out of.
  
```rust
fn main() {
    loop {
        println!("Running forever!");
        break; // Terminates the loop.
    }
}
```
- **Explanation:**  
  - Infinite loops are useful in scenarios where you want to run continuously until a certain condition is met. The explicit use of `break` ensures that the loop can be controlled precisely.

##### Loop Labels
- **Usage:**  
  - Useful when dealing with nested loops. Allows breaking or continuing a specific loop.
  
```rust
fn main() {
    let mut count = 0;
    'outer: loop {
        println!("count = {count}");
        let mut remaining = 10;
        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break; // Breaks out of the inner loop.
            }
            if count == 2 {
                break 'outer; // Breaks out of the outer loop.
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {count}");
}
```
- **Explanation:**  
  - Loop labels are especially helpful when dealing with multiple nested loops. They let you clearly communicate which loop you intend to break from, improving both code readability and safety.

##### `while` Loop
- **Definition:**  
  - The `while` loop runs as long as a condition is true.
  
```rust
fn main() {
    let mut number = 3;
    while number > 0 {
        println!("{number}");
        number -= 1;
    }
}
```
- **Explanation:**  
  - While loops are intuitive for condition-based repetition. They are particularly useful when the number of iterations isn’t known ahead of time. However, for iterating over collections, `for` loops are generally more efficient.

##### `for` Loop
- **Definition:**  
  - The `for` loop provides a concise way to iterate over elements of a collection.
  
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a {
        println!("The value is: {element}");
    }
}
```
- **Explanation:**  
  - For loops abstract away the complexity of manually indexing collections. They reduce the risk of errors (such as out-of-bounds access) and make your intent clear: you are iterating over each element in a collection.

---

## Chapter 4 - Borrowing and Ownership

### The Stack and Heap

#### The Stack
- **Definition:**  
  - The stack is a region of memory used for static memory allocation. It follows a Last In, First Out (LIFO) order.
- **How It Works:**  

- **Characteristics:**
  - **Speed:** Allocation and deallocation are extremely fast since they only involve adjusting the stack pointer.
  - **Size:** Memory on the stack is limited and must be known at compile time.
  - **Usage:** Ideal for storing fixed-size data like primitives and arrays with a known length.
- **Expanded Explanation:**  
  - Because the stack is tightly managed by the system, accessing data stored on it is very fast. However, the stack’s limited size means that larger or dynamically sized data must be allocated on the heap.

#### The Heap
- **Definition:**  
  - The heap is a region of memory used for dynamic memory allocation.
- **How It Works:**  
  - When storing data, you request memory from the allocator. The allocator finds available space, marks it as used, and returns a pointer to that location. Think of a restaurant: a host finds an available table and gives you directions (a pointer).
- **Characteristics:**
  - **Flexibility:** Data on the heap can grow or shrink as needed.
  - **Performance:** Accessing heap data is slower because it involves pointer dereferencing and potential cache misses.
  - **Usage:** Used for data types like `String` and `Vec<T>` where size may change.
- **Explanation:**  
  - Although heap allocation offers flexibility, it comes with the overhead of managing memory allocation and deallocation. Rust’s ownership system is designed to handle this automatically and safely, ensuring that heap memory is cleaned up when no longer needed.

#### Key Differences: Stack vs. Heap
- **Structure:**  
  - **Stack:** Organized, with a simple push/pop mechanism.  
  - **Heap:** Unstructured, requiring dynamic allocation.
- **Size:**  
  - **Stack:** Fixed size determined at compile time.  
  - **Heap:** Can grow and shrink dynamically during program execution.
- **Speed:**  
  - **Stack:** Fast, since it only involves push/pop operations.  
  - **Heap:** Slower, because it requires searching for space and managing memory.
- **Usage Scenarios:**  
  - **Stack:** Use for temporary variables and fixed-size data.
  - **Heap:** Use for dynamic data that might grow or shrink.
- **Access:**  
  - **Stack**: Direct access—CPU operations are very fast.  
  - **Heap**: Pointer-based access, requiring extra lookup time.
- **Use Cases:**  
  - **Stack**: Function calls, fixed-size data (integers, floats, booleans).  
  - **Heap**: Dynamic data (e.g., Strings, Vectors, complex objects).

#### Why Does Rust Care About Stack & Heap?
- **Function calls use the stack:**  
  Arguments and local variables are pushed onto the stack. When the function ends, they are popped off automatically.
- **Heap memory needs careful management:**  
  The ownership system ensures that data is cleaned up when no longer needed. This prevents memory leaks and duplicate heap data.
- **Stack is faster, heap is flexible:**  
  Rust optimizes for performance by keeping data on the stack whenever possible. If data size is unknown or needs to change, Rust safely uses the heap with ownership rules.

#### Key Takeaways
- Stack = Fast, simple, fixed-size values (pushed and popped).
- Heap = Flexible, dynamic data, but slower (uses pointers).
- Rust’s Ownership System automatically manages heap memory.
- Understanding this helps explain why Rust has ownership and borrowing rules!

### Variables and Data: Move, Copy, and Clone in Rust

#### Stack Data (Fixed Size) → Copy
- Copy(Simple Bitwise Duplication) means that when a variable is assigned to another or passed to a function, Rust creates a bitwise duplicate of its value instead of transferring ownership. Only small, fixed-size types stored on the stack can implement Copy.

**Types That Implement Copy**
✔ All primitive types implement Copy because they are stored entirely on the stack and are cheap to duplicate:
- Integer types: i8, i16, i32, i64, i128, u8, u16, u32, u64, u128
- Floating-point types: f32, f64
- Boolean: bool
- Character: char
- Pointer types:
- Immutable references: &T (if T: Copy)
- Raw pointers: *const T, *mut T

- **Example:**
  ```rust
  let x = 5;
  let y = x;  // x is copied; both x and y are independent.
  ```

✔ Numbers (`i32`, `bool`, `char`) **implement the `Copy` trait** because they are small, fixed-size, and live entirely on the stack.  
✔ Copying stack values is **cheap** because it involves only simple duplication of memory.  
✔ Both `x` and `y` exist independently, and modifying one does **not** affect the other.  
✔ Useful for Simple Calculations: When you need to pass values into functions without worrying about ownership transfer, Copy types are ideal.
✔ Safe for Multiple Uses: Since copying does not affect the original variable, it is perfect for cases where the same value is used in multiple places.

- **Explanation:**  
  - Simple, fixed-size types like integers are stored entirely on the stack. Copying such types is inexpensive and doesn’t require complex memory management. This is why these types implement the `Copy` trait, allowing for a bitwise copy of the value.

#### Heap Data (Dynamic Size) → Move
- Move(Ownership Transfer) occurs when ownership of a heap-allocated value is transferred to another variable. After the move, the original variable becomes invalid, preventing multiple ownership and ensuring memory safety. Rust enforces this rule to prevent double free errors.

✔ Move is the default behavior for non-Copy types.
✔ Rust automatically "moves" ownership when a non-Copy type (like String) is assigned to another variable

**Types That Follow Move Semantics**
✔ Any type that does not implement Copy follows move semantics. ✔ This includes:
- Heap-allocated types:
- String
- Vec<T>
- Box<T>
- HashMap<K, V>
- BTreeMap<K, V>
- Custom structs (unless explicitly marked Copy)
- Tuples, arrays, or structs containing non-Copy elements

- **Example:**
  ```rust
  let s1 = String::from("hello");
  let s2 = s1;  // Ownership moves from s1 to s2; s1 becomes invalid.
  ```

✔ A `String` **stores only a pointer, length, and capacity on the stack**.  
✔ **`s1` is on the stack**, while its **actual contents ("hello") live on the heap**.  
✔ When ownership moves from `s1` to `s2` (**which gets a new stack address**), Rust only moves the stack data (pointer, length, and capacity).  
✔ The **heap data remains unchanged**—Rust **does not copy** the actual string `"hello"`.  
✔ **`s1` is now invalid**, ensuring **only `s2` has ownership** and preventing **double free errors*

- **Explanation:**  
  - For types that store data on the heap (like `String`), Rust only copies the pointer, length, and capacity—leaving the heap data intact. The original variable is invalidated to prevent multiple ownership and potential double-free errors. This move semantics helps ensure memory safety without the need for a garbage collector.

#### What Happens During a Move?
- **Walkthrough:**
  ```rust
  let s1 = String::from("genius");
  let s2 = s1;  // s1’s pointer data (pointer, length, capacity) is copied to s2.
  ```
  1. **Rust copies the stack data** (pointer, length, capacity) from `s1` to a **new stack entry for `s2`**
  2. The heap data (the actual string content) is not duplicated.
  3. `s1` is marked as invalid, ensuring that only `s2` has ownership.
- **Explanation:**  
  - This move mechanism is a cornerstone of Rust’s memory safety, as it ensures that only one variable is responsible for freeing heap memory, eliminating the risk of double-free errors.

#### 3️⃣ Cloning Heap Data -> Clone
- Clone(Explicit Deep Copy) creates a new, independent copy of heap-allocated data, duplicating both stack metadata and heap contents. Unlike Copy, Clone is explicit and usually expensive, so Rust requires manually calling .clone().

**Types That Implement Clone Trait**
✔ Most heap-allocated types implement Clone, allowing deep copies:
- String
- Vec<T>
- Box<T>
- HashMap<K, V>
- BTreeMap<K, V>
- Rc<T> (Reference-counted pointer)
- Arc<T> (Thread-safe reference-counted pointer)
- Custom types (if manually implemented)

- **Example:**
  ```rust
  let s1 = String::from("hello");
  let s2 = s1.clone();  // Performs a deep copy of the heap data.
  println!("s1 = {s1}, s2 = {s2}");
  ```
✔ clone() explicitly duplicates the heap data, so both s1 and s2 have separate memory allocations.
✔ Unlike a move, s1 remains valid after s1.clone().
✔ Cloning is expensive (it performs a full copy of heap data), so Rust makes it explicit to avoid unnecessary performance costs.

- **Explanation:**  
  - When you explicitly call `clone()`, Rust duplicates both the stack metadata and the actual heap data. This is more expensive than a move operation, which is why Rust forces you to use `clone()` explicitly to signal that you intend to create a full copy. This prevents accidental performance penalties.

#### Quick Recap in Bullet Points
1. **Stack variables** (like `i32`, `bool`) are automatically copied because their fixed size makes duplication inexpensive.
2. **Heap variables** (like `String`) are moved by default to avoid duplicating large amounts of data.
3. **After a move, the original variable is invalidated** to maintain safety.
4. **Using `clone()` creates an independent copy** of the heap data, ensuring both variables own separate memory.
5. **Move semantics help avoid memory errors** such as double-free by ensuring a single owner.
6. **Types that implement `Copy` do so because they’re small and can be duplicated without performance issues.**
7. **If a type implements `Drop`, it cannot implement `Copy`**, so Rust forces explicit cloning.
8. **Tuples can be copied if all elements implement `Copy`,** otherwise, they follow move semantics.
9. **Moves are efficient as they only transfer metadata.**
10. **Cloning is an explicit operation** to prevent unintentional performance hits.

---

### Ownership and Functions in Rust

**Function Argument Types (Rule of Thumb)**

1) Need to store the argument somewhere? | -> Favor taking ownership (receive a value)
2) Need to do a calculation with the value? | -> Favor receiving a read-only reference
3) Need to change the value in some way? | -> Favor receiving a mutable reference

### **Ownership and Functions in Rust**

#### **How Ownership Works with Strings**

In Rust, ownership rules apply to strings differently based on how they are stored and referenced. Let’s explore **four key string types** and their behavior in functions.

---

#### **`String` – A Heap-Allocated, Dynamic String**
```rust
fn take_ownership(s: String) {
    println!("Owned string: {}", s);
} // `s` is dropped here

fn main() {
    let my_string = String::from("Hello, Rust!");
    take_ownership(my_string);
    // println!("{}", my_string); // Error: `my_string` is moved
}
```
#### **Behavior**
- `String` is an **owned heap-allocated string**.
- When `my_string` is **passed to the function**, ownership **moves**, and `main()` loses access.
- Trying to use `my_string` after the function call results in a **compile-time error**.

#### **Key Points**
✔ **Stored on the heap**, dynamically resizable.  
✔ **Ownership transfers** when passed to a function.  
✔ **Dropped** when the function exits unless returned.

---

#### **`&String` – A Reference to a Heap String**
```rust
fn borrow_string(s: &String) {
    println!("Borrowed string: {}", s);
}

fn main() {
    let my_string = String::from("Hello, Rust!");
    borrow_string(&my_string);
    println!("{}", my_string); // Still valid!
}
```
#### **Behavior**
- `&String` is a **reference to a heap-allocated `String`**.
- **Ownership is not transferred**, so `my_string` is still valid after the function call.
- This is **borrowing** and enables **safe, read-only access**.

#### **Key Points**
✔ Allows **read-only access** to heap data.  
✔ **Ownership is not moved**—original value remains valid.  
✔ Prevents unnecessary heap allocation and copying.

---

#### **`str` – A Hardcoded, Read-Only String in Binary**
```rust
fn print_literal(s: &str) {
    println!("String literal: {}", s);
}

fn main() {
    print_literal("Hello, world!");
}
```
#### **Behavior**
- `"Hello, world!"` is a **string literal**—a hardcoded **`str`** stored in the program's binary.
- The function accepts a `&str` reference, so ownership **never changes**.

#### **Key Points**
✔ **Immutable & hardcoded** into the compiled binary.  
✔ Fast access—no heap allocation required.  
✔ Can be safely passed around as `&str`.

---

#### **`&str` – A Reference to String Data in Memory**
```rust
fn display_message(message: &str) {
    println!("Message: {}", message);
}

fn main() {
    let my_string = String::from("Dynamic Rust!");
    display_message(&my_string); // Implicit conversion: `&String` → `&str`
    
    let my_literal = "Hardcoded Rust!";
    display_message(my_literal); // Direct `&str`
}
```
### **Behavior**
- `&str` is a **reference to string data** in memory.
- Works with both **string literals (`str`) and `String` (heap)**.
- **Implicit conversion**: `&String` automatically converts to `&str` when needed.

### **Key Points**
✔ **Does not take ownership**.  
✔ Works with both `String` and `str`.  
✔ **More flexible** than `&String`, making functions reusable.

---

## **Summary Table: Strings & Ownership in Rust**
| Type     | Stored In| Ownership | When Passed to a Function      |
|----------|----------|-----------|--------------------------------|
| `String` | Heap     | **Owned** | **Moved** (unless cloned)      |
| `&String`| Heap     | Borrowed  | **Not moved** (safe reference) |
| `str`    | Binary   | Immutable | Always available, fast access  |
| `&str`   | Memory   | Borrowed  | **Not moved** (safe reference) |

#### **Best Practices**
- Use **`String`** when you need a **dynamic, heap-allocated string**.
- Use **`&str`** when you just need to **read a string without taking ownership**.
- Prefer **`&str` over `&String`** in function signatures (`fn my_func(s: &str)`) for flexibility.
- **Avoid unnecessary ownership transfers** to optimize performance.

---

#### How Ownership Works with Functions
- **Explanation:**
  - When you pass a variable to a function, Rust follows the same rules as when assigning variables. For stack data (like integers), the data is copied. For heap data (like `String`), ownership is transferred (moved) unless you pass a reference.
  -` Stack data (i32)` is copied, while `heap data (String)` is moved.

**Move Example (Heap Data):**
When we pass a variable to a function, it behaves just like assigning it to another variable -> 

```rust
let s1 = String::from("hello"); 
let s2 = s1; //Ownership moves to s2, s1 is no longer valid.
```

In Functions

- Function parameters follow the same rules of ownership **(copying vs. moving)**.

- If a type **does not implement the `Copy` trait**, ownership **moves** to the parameter.

- If the type **implements the `Copy` trait**, Rust **creates a copy** of the value for the parameter.

- If ownership **moves**, the function must **return the value** to the caller to prevent the memory from being deallocated.

```rust
fn takes_ownership(some_string: String) {
    println!("{some_string}"); // some_string is dropped when the function scope ends. in this function, the parameter some_string takes ownership of the value passed into it. 
}

fn makes_copy(some_integer: i32) {
    println!("{some_integer}"); // some_integer is copied, so the original remains valid.
}

fn main() {
    let s = String::from("hello");
    takes_ownership(s);  // s is moved into the function and becomes invalid here.

    let x = 5;
    makes_copy(x);  // x is copied; x is still valid after the function call.
    println!("x is still valid: {x}");
}
```
- **Explanation:**  
  - This design ensures that functions have clear ownership over their inputs. When ownership is moved, it is clear that the caller can no longer use the value, preventing bugs related to use-after-free or double-free.

#### Returning Values and Ownership
- **How It Works:**
  - A function can return a value to transfer ownership back to the caller.

**Example:**
```rust
fn gives_ownership() -> String {
    String::from("hello") // Ownership moves to the caller.
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string // The ownership of a_string is returned.
}

fn main() {
    let s1 = gives_ownership(); // s1 now owns the returned String.
    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2); // s2 is moved; s3 now owns the String.
    // println!("{s2}"); // Error: s2 has been moved.
}
```

✔ **gives_ownership()** moves a String to s1.
✔ **takes_and_gives_back()** takes ownership of s2, then moves it back to s3.
✔ This process is necessary because Rust enforces ownership rules at compile time.

- **Explanation:**  
  - Returning values from functions allows you to manage ownership transfers explicitly. This is central to Rust’s design, ensuring that every piece of memory has a clear owner, which in turn is responsible for freeing it.

#### Alternative: Returning Multiple Values
- **Usage:**  
  - When you need to return more than one value, you can bundle them together in a tuple and Instead of manually returning ownership, we can return multiple values using a tuple.
  
**Example:**
```rust
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();  // Calculate the string length.
    (s, length)  // Return both the string and its length.
}

fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1); // s1 is moved and s2 now owns the String.
    println!("The length of '{s2}' is {len}.");
}
```
- **Explanation:**  
  - Using tuples to return multiple values is a common pattern in Rust. It prevents ownership from being lost while still providing multiple pieces of data to the caller.

**Summary**
Ownership moves to functions unless the type implements Copy.
To retain ownership, return values from functions.
Tuples allow returning multiple values at once.
This prevents memory leaks while enforcing safety.

---

#### Rules of Borrowing and Ownership

Rust’s ownership rules are the cornerstone of its memory safety.
- **Key Rules:**
  - Each value has a single owner.
  - When the owner goes out of scope, the value is dropped.
  - Ownership can be moved by default or explicitly cloned.
  - References allow you to borrow a value without taking ownership.
  - At any given time, you can have either multiple immutable references or one mutable reference, but not both.
- **Explanation:**  
  - These rules ensure that memory is automatically cleaned up when no longer needed and prevent many common programming errors such as dangling pointers or memory leaks.

---

### Referencing and Borrowing in Rust

- A **reference** is an address of a value in memory.  
  In other languages, it's called a **pointer**.

- A **reference** allows the program to use a value  
  **without transferring ownership**.

- We describe this action of creating a reference as **"borrowing"**.

- Create a reference with the **borrow operator (`&`)**.

#### The Problem with Moving Ownership
- **Explanation:**
  - Passing a value that owns heap data (e.g., a `String`) into a function normally transfers its ownership. This means that the original variable becomes invalid, and trying to use it later will result in a compile-time error.
  
**Example:**
```rust
fn calculate_length(s: String) -> usize {
    s.len()
} // s is dropped at the end of the function.

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(s1); // Ownership of s1 is moved.
    println!("The length of '{s1}' is {len}"); // Error: s1 is no longer valid.
}
```
- **Explanation:**  
  - This design forces you to think about ownership and resource management, which is crucial for preventing bugs in concurrent or low-level programming.

#### Solution Using References(&) to Borrow Data
- **How It Solves the Problem:**  
  - Instead of transferring ownership, you can pass a reference. This allows the function to access the data without taking ownership, leaving the original variable valid after the function call.
  
**Remember s1 examlpe this is how it will look. Example with &:**
```rust
fn calculate_length(s: &String) -> usize {
    s.len()
} // s is only borrowed, not moved.

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Pass a reference to s1.
    println!("The length of '{s1}' is {len}."); // s1 remains valid.
}
```
- **Expanded Explanation:**  
  - Borrowing via references is one of Rust’s most powerful features. It allows safe sharing of data between different parts of your program without duplicating it, thus maintaining performance while ensuring safety.

 **REFERENCES MUST NEVER OUTLIVE THEIR REFERANT**

#### What Is Borrowing?
- **Definition and Analogy:**
  - Borrowing means temporarily using a value without claiming ownership. Think of it like borrowing a book from a library: you can read it, but you must return it in the same condition.
- **Explanation:**  
  - This mechanism allows functions to work with data without having to worry about the overhead of copying or transferring ownership. It also helps prevent accidental modification or deallocation of data.

#### References Are Immutable by Default
- **Key Point:**  
  - By default, references in Rust are immutable, which means you cannot modify the borrowed data.
  
```rust
fn change(some_string: &String) {
    some_string.push_str(", world"); // Error: Cannot modify immutable reference.
}
```
⛔ Error:

error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference Why?

&s is an immutable reference, so we can’t modify the borrowed data.
By default, references do not allow mutation.

- **Explanation:**  
  - Immutable references allow multiple parts of your code to read the same data simultaneously without conflict. This is a key factor in ensuring thread safety and data consistency.

#### Mutable References (`&mut`)
- **Usage:**  
  - When you need to modify borrowed data, to use a mutable reference (`&mut`).
  
```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // Allowed modification.
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s); // Pass a mutable reference.
    println!("{s}"); // Prints "hello, world"
}
```
✔ `&mut` s creates a mutable reference to allow modification.
✔ some_string: &mut String takes a mutable reference instead of ownership.
✔ The original s can still be used after modification.
 Mutable references allow modifying borrowed data while still preventing ownership transfer.

- **Explanation:**  
  - Mutable references give you the flexibility to alter data without transferring ownership. However, to avoid conflicts and data races, Rust enforces that only one mutable reference exists at a time.
 Summary
References (&) allow functions to use a value without taking ownership.
Borrowing lets us pass data while keeping the original variable valid.
By default, references are immutable (&T).
To modify a reference, use mutable references (&mut T).
Rust prevents multiple mutable references to avoid data races.

---

#### Why Use Mutable References?
- **Expanded Explanation:**  
  - In scenarios where data needs to be changed, mutable references provide a controlled way to do so without losing ownership. This ensures that the data remains consistent throughout its lifecycle.

**Example:**
```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // The mutable reference allows modification.
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s); // s must be declared as mutable to allow a mutable reference.
    println!("{s}"); // Outputs "hello, world"
}
```

#### Rust Prevents Multiple Mutable References
- **Example:**
  ```rust
  fn main() {
      let mut s = String::from("hello");

      let r1 = &mut s; 
      let r2 = &mut s; // Error: Cannot have more than one mutable reference at a time.
      
      println!("{r1}, {r2}");
  }
  ```
- **Explanation:**  
  - Allowing more than one mutable reference could lead to data races, where two parts of your program try to modify data simultaneously. Rust’s compile-time checks ensure that only one mutable reference exists, which is crucial for thread safety.

#### Using Scopes to Manage Mutable References
- **How It Works:**  
  - By limiting the lifetime of a mutable reference using inner scopes, you can safely create new mutable references once the previous one goes out of scope.
  
**Example:**
```rust
fn main() {
    let mut s = String::from("hello");
Data Races
    { 
        let r1 = &mut s; 
        println!("{r1}"); // r1 is used and then goes out of scope.
    } 

    let r2 = &mut s; // Allowed because r1 is no longer in use.
    println!("{r2}");
}
```
- **Explanation:**  
  - Scoping is a powerful feature in Rust that lets you manage the lifetime of references. It ensures that mutable references do not overlap, thereby preventing data races while still allowing flexible modifications.

#### Mixing Mutable and Immutable References (What Not to Do)
- **Example of an Error:**
  ```rust
  fn main() {
      let mut s = String::from("hello");

      let r1 = &s; // Immutable reference.
      let r2 = &s; // Another immutable reference.
      let r3 = &mut s; // Error: Cannot have a mutable reference while immutable ones exist.
      
      println!("{r1}, {r2}, and {r3}");
  }
  ```
- **Explanation:**  
  - Rust’s safety rules forbid mixing mutable and immutable references simultaneously because it could lead to unpredictable behavior. Immutable references guarantee that the data will not change, which would be violated if a mutable reference were allowed concurrently.

#### Rust Allows Mutable References After Immutable References Are No Longer Used

The Rust compiler is smart enough to detect when an immutable reference is no longer needed before allowing a mutable reference.

- **Example:Allowed Since Immutable References Are No Longer Used**
  ```rust
  fn main() {
      let mut s = String::from("hello");

      let r1 = &s; 
      let r2 = &s; 
      println!("{r1} and {r2}"); // Immutable references r1 and r2 are last used here.

      let r3 = &mut s; // Allowed because r1 and r2 are no longer in use.
      println!("{r3}");
  }
  ```
- **Explanation:**  
  - The Rust compiler is smart enough to track when immutable references are no longer needed. Once their last use occurs, it permits a mutable reference to be created safely.

---

#### 🔹 What Are Data Races? (Why Rust Enforces These Rules)
A **data race** occurs when:
1️⃣ Two or more references access the same data at the same time.  
2️⃣ At least one of the references **modifies** the data.  
3️⃣ No synchronization exists to prevent conflicts.  

**Example: Data Race in Other Languages (Unsafe)**
In **C/C++**, this can happen:
```cpp
int shared_value = 10;

void modify() {
    shared_value += 1; // Unsafe: Multiple threads can modify `shared_value`
}

void read() {
    printf("%d\n", shared_value); // Unsafe: No guarantee of what value is read
}
```
- **If two threads modify `shared_value` at the same time**, the value can become **corrupted**.
- Rust **prevents this at compile time** by **restricting multiple references to mutable data**.

### Key Takeaways  
**Mutable references (`&mut`) allow modifying borrowed data without transferring ownership.**  
**Rust prevents multiple mutable references at the same time to avoid data races.**  
**Mutable references cannot coexist with immutable references to ensure data consistency.**  
**Using scopes (`{}`) lets us reuse mutable references safely.**  
**Rust prevents undefined behavior by enforcing safe memory access rules at compile time.**  

###  Dangling References in Rust

#### 🔹 What Is a Dangling Reference?
A **dangling reference** occurs when a pointer **refers to memory that has been freed**, leading to **undefined behavior**.  
In many languages (like **C/C++**), it’s possible to create a **dangling pointer** by referencing memory that **no longer exists**.

### **1️⃣ Example: Dangling Pointer in C**
```c
int* create_dangling_pointer() {
    int x = 42;  // x is a local variable
    return &x;   // ERROR: Returning a pointer to `x`, which will be destroyed!
}

int main() {
    int* p = create_dangling_pointer();  // `p` now points to invalid memory!
    printf("%d", *p); // Undefined behavior: memory has been freed!
}
```
✔ **This can lead to crashes or memory corruption** because the **memory is already freed**.  
✔ **Rust prevents this by enforcing safe references at compile time!**  

#### Rust **Prevents** Dangling References
In Rust, the **borrow checker ensures that references always point to valid data**.  
If a reference might become **invalid**, Rust **won’t compile the code**.

##### **Example: Dangling Reference in Rust (Compile-Time Error)**
```rust
fn dangle() -> &String { // ERROR: Returning a reference to a local variable
    let s = String::from("hello"); // `s` is created inside the function
    &s // ERROR: `s` will be dropped at the end of the function!
} // `s` is deallocated here!

fn main() {
    let reference_to_nothing = dangle(); // Compilation fails
}
```
⛔ **Error Message:**
```
error[E0106]: missing lifetime specifier
this function's return type contains a borrowed value,
but there is no value for it to be borrowed from
``` **Why Does This Happen?**  
- `s` is **created inside `dangle()`**, so when the function ends, **`s` is dropped**.  
- The reference `&s` would point to **freed memory**, which **Rust prevents**.  
- **Rust ensures that all references remain valid for their entire lifetime**.  

#### Correct Way: Return Ownership Instead of a Reference
Instead of returning a **reference to a local variable**, **move ownership out of the function**.

#### **3️⃣ Example: Fixing the Dangling Reference**
```rust
fn no_dangle() -> String {
    let s = String::from("hello"); // `s` is created
    s // Ownership is moved instead of returning a reference
}

fn main() {
    let valid_string = no_dangle(); // `valid_string` now owns the String
    println!("{valid_string}"); // Works fine!
}
```
✔ Instead of returning a **reference**, `s` is **moved out of the function**.  
✔ **Rust enforces ownership rules to ensure safe memory access.**  

#### The Rules of References in Rust
Rust follows **strict borrowing rules** to prevent **dangling references**:

#### **1. At any given time, you can have either:**
   - **One mutable reference (`&mut`)**  
   - **OR multiple immutable references (`&`)**  
   - **You cannot have both at the same time**  

#### **2. References Must Always Be Valid**
   - **You cannot return a reference to a variable that will be dropped.**
   - **Rust ensures references always point to live data.**

#### **Summary**
**Dangling references happen when a pointer refers to invalid memory.**  
**Rust prevents dangling references by enforcing ownership and borrowing rules.**  
**A function cannot return a reference to a local variable (use ownership instead).**  
**Rust ensures references always point to valid data at compile time.**  

### **Why This Matters**
- **Rust eliminates entire classes of memory safety bugs** by **enforcing strict reference rules**.  
- **Many languages allow dangling pointers, leading to crashes**—but Rust prevents them **before the program runs**.  
- **Mastering references is key** to writing **safe and efficient Rust programs**.

# The Slice Type in Rust

#### 🔹 What Is a Slice?
A **slice** is a **reference to a part of a collection** instead of the whole collection.  
Slices **do not take ownership**; they just provide a **view into the data**.

✔ **Why use slices?**  
- **Avoid unnecessary copying** of data.  
- **Ensure data remains valid** (prevents out-of-sync indexes).  
- **More efficient and safer** than using raw indexes.

#### Problem: Finding the First Word Without Slices
Imagine we need to **find the first word** in a string.

#### **1️⃣ Attempt Without Using Slices**
```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes(); // Convert String to bytes

    for (i, &item) in bytes.iter().enumerate() { // Iterate through characters
        if item == b' ' { // Check for space
            return i; // Return index of space
        }
    }
    s.len() // If no space found, return full length
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // word = 5

    s.clear(); // Now `word` is invalid because `s` is empty!
}
```
### ** Problem with This Approach**
- `word` stores **an index (5)**, but **if `s` is modified (`s.clear()`), the index becomes invalid**.
- **There is no guarantee** that `word` still refers to a valid part of `s`.
- **Bug-prone**: If `s` changes, `word` may reference non-existent data.
 **Solution: Use Slices to Return a Valid Part of the String!**  

#### 🔹 Solution: String Slices
A **string slice** is a reference **to a portion of a `String`**, ensuring **it remains valid**.

##### **2️⃣ Example: Using Slices**
```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5]; // Slice: "hello"
    let world = &s[6..11]; // Slice: "world"

    println!("{hello} {world}"); // Output: hello world
}
```
✔ `&s[0..5]` creates a **slice from index 0 to 5** (not including 5).  
✔ `&s[6..11]` creates a **slice from index 6 to 11** (not including 11).  
✔ **No ownership is taken**, so `s` remains valid.  
 **Slices solve the problem by returning a reference to part of `s`, keeping it in sync with the original string.**

#### Simplified Slice Syntax
Rust allows **shorter syntax** for common slice cases:

```rust
let s = String::from("hello");
let slice1 = &s[..2]; // Same as &s[0..2]
let slice2 = &s[3..]; // Same as &s[3..s.len()]
let slice3 = &s[..];  // Same as &s[0..s.len()]
```
✔ `[..2]` means **"from start to index 2"**.  
✔ `[3..]` means **"from index 3 to the end"**.  
✔ `[..]` means **"the whole string"**.  

#### Fixing `first_word()` with Slices
Using a slice ensures **the returned data remains valid**.

#### Correct Version of `first_word()`**
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes(); // Convert to bytes

    for (i, &item) in bytes.iter().enumerate() { // Loop through characters
        if item == b' ' {
            return &s[0..i]; // Return slice from start to space
        }
    }
    &s[..] // If no space, return entire string
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // "hello"

    // ERROR: This will now fail to compile!
    s.clear(); // We cannot clear `s` while `word` is in use
    println!("The first word is: {word}");
}
```
⛔ **Rust Compiler Error**
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
``` **Why is this error GOOD?**  
- The compiler **prevents modifying `s` while `word` exists**.  
- This **eliminates an entire class of bugs!**  

#### String Literals Are Slices
```rust
let s = "Hello, world!";
```
✔ **The type of `s` is `&str`, a string slice.**  
✔ **String literals are slices stored in the binary at compile time.**  
✔ This is why **string literals are immutable** (`&str` is an immutable reference).  

#### Making `first_word()` More Flexible
Rust allows **string slices (`&str`) to work with both `String` and string literals**.

#### **Improved `first_word()` (More Flexible)**
```rust
fn first_word(s: &str) -> &str { // Works for &String and &str
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

fn main() {
    let my_string = String::from("hello world");

    let word1 = first_word(&my_string[..]); // Pass slice of `String`
    let word2 = first_word(&my_string);     // Works with &String

    let my_string_literal = "hello world";

    let word3 = first_word(&my_string_literal[..]); // Pass slice of string literal
    let word4 = first_word(my_string_literal);     // Works directly with &str
}
```
✔ **Works for both `String` and `&str`**, making it more useful.  
✔ **Avoids unnecessary type conversions**.  
 **String slices make Rust APIs more flexible and safe!**  

#### Array Slices
Slices work for **arrays** too!

#### **Array Slice Example**
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let slice = &a[1..3]; // Slice of `[2, 3]`
    assert_eq!(slice, &[2, 3]);

    println!("{:?}", slice); // Output: [2, 3]
}
```
✔ `&a[1..3]` creates a **slice of elements 1 to 3** (excluding index 3).  
✔ The slice **remains tied to `a`**, preventing invalid access.  
 **Array slices work just like string slices and are just as safe!**  

#### Summary
**Slices (`&[start..end]`) reference a portion of a collection without taking ownership.**  
**String slices (`&str`) prevent out-of-sync indexes and borrowing errors.**  
**Rust ensures slices are always valid, preventing bugs at compile time.**  
**String literals (`&str`) are slices stored in the binary and are immutable.**  
**Slices work for both `String` and `&str`, making APIs more flexible.**  
**Array slices (`&[i32]`) work the same way as string slices.**  

### **Why This Matters**
- **Slices solve real-world memory safety issues** by keeping references **valid and in sync** with the data.  
- Rust **prevents indexing bugs at compile time**, unlike many other languages.  
- **Mastering slices is key to writing efficient Rust code**!  

#### Chapter 4 Summary: Ownership, Borrowing, References, and Slices

#### Why Is Ownership Important?
Rust **ensures memory safety at compile time** through **ownership rules**, preventing memory leaks, dangling pointers, and data races **without a garbage collector**.

✔ **Ownership defines how memory is managed in Rust.**  
✔ **Borrowing lets functions use data without taking ownership.**  
✔ **References allow safe and efficient access to variables.**  
✔ **Slices let us work with parts of collections without copying.**  

Ownership **affects all aspects of Rust programming**, making it essential to understand these concepts.

#### Ownership: The Foundation of Rust’s Memory Safety
#### Ownership Rules:
1️⃣ **Each value in Rust has one and only one owner.**  
2️⃣ **When the owner goes out of scope, Rust automatically cleans up the value.**  
3️⃣ **Transferring ownership (moving) invalidates the original variable.**

- new owner starts off just like a fresh variable initialization. It has its own set of permissions on what it can do with the data it represents.
- When ownership is moved from one variable to another in Rust, the mutability of the new owner is determined by how the new variable is declared—not by the original variable's mutability.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // Ownership moves from `s1` to `s2`
    
    println!("{s1}"); // ERROR: `s1` is invalid after move!
}
```
**Rust prevents multiple variables from freeing the same memory (double-free error).**  

#### Borrowing and References: Using Data Without Taking Ownership
Borrowing allows functions to **access** data **without taking ownership**.

#### References (`&T` - Immutable Borrowing)
```rust
fn calculate_length(s: &String) -> usize { 
    s.len()  // `s` is borrowed, not moved
}

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s); // Borrow `s` without ownership transfer
    println!("Length: {len}");
}
```
✔ `&s` **creates a reference** to `s` instead of transferring ownership.  
✔ The function **can read `s` but not modify it**.  
✔ **Multiple immutable references are allowed at the same time.**  

#### Mutable References (`&mut T` - Mutable Borrowing)
If we need to modify borrowed data, we use **mutable references (`&mut`)**.

```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // Allowed with `&mut`
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s); // Mutable borrow
    println!("{s}"); // Output: "hello, world"
}
```
✔ **Only one mutable reference is allowed at a time** to prevent data races.  
✔ **We cannot mix mutable and immutable references simultaneously.**  

#### Dangling References: Rust Prevents Invalid Memory Access
Rust **prevents dangling references** at compile time.

```rust
fn dangle() -> &String { 
    let s = String::from("hello"); // `s` is created inside the function
    &s // ERROR: `s` will be dropped at the end of the function!
}

fn main() {
    let ref_to_nothing = dangle(); // Compile-time error
}
```
✔ Rust **ensures all references are valid** before allowing compilation.  
✔ **Solution:** Return ownership instead of a reference.

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s // Ownership moves out instead of returning a reference
}
```

#### Referencing Parts of a Collection
A **slice** is a reference **to part of a collection**, preventing out-of-sync indexes.

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i]; // Return slice instead of index
        }
    }
    &s[..]
}
```
✔ **Slices fix bugs related to invalid indexes.**  
✔ **String slices (`&str`) reference part of a `String` without copying.**  

#### Array Slices Also Work the Same Way
```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3]; // Slice of `[2, 3]`
println!("{:?}", slice); // Output: [2, 3]
```
✔ **Slices ensure safe and efficient access to collections.**  

#### Summary of Chapter 4
**Ownership ensures each value has a single owner, preventing memory errors.**  
**Borrowing (`&T`) allows safe access to data without moving ownership.**  
**Mutable references (`&mut T`) allow controlled modification while preventing data races.**  
**Rust prevents dangling references at compile time.**  
**Slices (`&[T]`) provide safe, efficient access to parts of collections.**  
**Rust’s ownership model eliminates memory safety bugs without garbage collection.**  

#### **Why This Matters**
- **Ownership and borrowing are the foundation of Rust’s memory safety model.**  
- **Rust eliminates entire classes of memory bugs (use-after-free, data races, and null pointers).**  
- **Understanding these concepts is crucial for writing safe and efficient Rust programs.**  

# Chapter 5: Structs in Rust

## **What Are Structs?**
Structs in Rust are used to **group related data together**. They are similar to tuples but provide additional clarity and flexibility by **associating names** with each piece of data.Each struct you define is its own type, even though the fields within the struct might have the same or different types. Structs by default does not implement copy traid so ownership moves if not spacify.

### **Why Use Structs Instead of Tuples?**
✔ Structs **name** each field, making the code easier to understand.
✔ The order of fields **does not matter** when creating an instance.
✔ Structs allow for **self-documenting code**.
✔ Unlike tuples, fields are accessed by **name** instead of index.

---
## **Defining a Struct**
To define a struct in Rust, use the `struct` keyword, followed by the struct’s name and a set of **named fields**.

### **Example: Defining a `User` Struct**
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
✔ This struct groups together multiple **related** fields.
✔ Each field has a **name** and **type**.
✔ Unlike tuples, field names **clarify their meaning**.

---
## **Creating an Instance of a Struct**
Once a struct is defined, we can **instantiate** it by providing values for each field.

### **Example: Creating a `User` Instance**
```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```
✔ Field order **does not** have to match the struct definition.
✔ The struct instance is stored in memory with **each field identified by name**.

---
## **Accessing and Modifying Struct Fields**
To retrieve values from a struct, we use **dot notation** (`.`).

### **Example: Accessing a Field**
```rust
println!("User email: {}", user1.email);
```
✔ Dot notation allows direct access to **any field** of the struct.
✔ This is **clearer** than tuple indexing.

### **Making a Struct Mutable**
To modify struct fields, the **entire instance** must be marked `mut`.

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com"); // Allowed since `user1` is mutable
}
```
✔ **Rust does not allow partial mutability**—the whole instance must be `mut`.
✔ This prevents confusion about which fields can change.

---
## **Returning Structs from Functions**
Structs can be returned from functions just like other types.

### **Example: Function That Returns a Struct**
```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```
✔ The function **creates and returns** a `User` instance.
✔ Uses **field init shorthand**—instead of `email: email`, we can just write `email`.
✔ This makes code **more concise and readable**.

## **Using Struct Update Syntax**
Sometimes, we want to **create a new struct instance** based on an existing one, changing only some fields. Instead of manually copying each field, we can use **struct update syntax**.

### **Example: Creating a New Struct from an Existing One**
```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1 // Copies remaining fields from `user1` using struct update syntax
    };
}
```
✔ `..user2` **copies** all fields from `user1` except those explicitly set.
✔ The **username** field is moved from `user1`, making `user1` **invalid** after this operation.
✔ If only **copyable** fields (`bool`, `u64`) were used, `user1` would still be valid.
✔ This **reduces repetition** and ensures field consistency.

### **How Struct Update Syntax Affects Ownership**
- The **`=` operator moves the data** from the previous instance to the new one.
- If a struct contains **heap-allocated fields** (like `String`), those fields are **moved**.
- If a struct contains **Stack-only fields** those fileds are **copyed**.
- **After the move, the original instance is no longer valid** and cannot be used.

### **Example: Move in Struct Update Syntax**
```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("user1"),
        email: String::from("user1@example.com"),
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("user2@example.com"),
        ..user1 // Moves `username` field
    };
    
    // println!("{}", user1.username); // ERROR: user1.username was moved
}
```
✔ `user1.username` **was moved** into `user2`, making `user1` invalid.
✔ If `user2` had **new values for all `String` fields**, `user1` would still be usable.
✔ **Fields that implement `Copy` (`bool`, `u64`) are duplicated, not moved.**

## Tuple Structs and Unit-Like Structs in Rust

### **Tuple Structs: Structs Without Named Fields**
Tuple structs provide the ability to **group related data without explicitly naming each field**. They work similarly to tuples but are **distinct types**, preventing unintended type mix-ups.

### **Defining and Instantiating Tuple Structs**
Tuple structs follow this syntax:

```rust
struct StructName(Type1, Type2, Type3);
```

Example:
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```
#### **Key Takeaways**
**Each tuple struct is its own type**  
**Even if fields have the same types, instances are distinct**  
**Useful when naming individual fields is unnecessary**  
**Prevents mixing up different data types unintentionally**  

#### **Accessing Tuple Struct Fields**
Tuple struct instances behave like regular tuples, allowing:
- **Destructuring**:
  ```rust
  let Color(r, g, b) = black;
  ```
- **Index Access**:
  ```rust
  let red = black.0; // Accessing the first field
  ```

### **Unit-Like Structs: Structs Without Any Fields**
Rust allows **empty structs**, called **unit-like structs**, that have no fields but still behave as types.

#### **Defining and Instantiating Unit-Like Structs**
```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```
#### **Use Cases for Unit-Like Structs**
**Useful when implementing traits without storing data**  
**Can serve as markers or flags in the type system**  
**Acts similarly to `()` (unit type) but provides a unique type**  

---

## **Ownership of Struct Data**
When defining structs, **ownership of fields must be considered** to ensure **data validity** throughout the struct's lifetime.

### **Why Use `String` Instead of `&str`?**
In earlier examples, we used:
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
✔ **Each instance owns its data**  
✔ **Ensures struct data remains valid**  

Using references (`&str`) inside structs introduces **lifetime complexities**, requiring Rust to **ensure references remain valid**. Consider this incorrect example:

```rust
struct User {
    active: bool,
    username: &str,  // Error: Reference stored without specifying a lifetime
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: "someusername123",
        email: "someone@example.com",
        sign_in_count: 1,
    };
}
```

#### **Compiler Error**
```
error[E0106]: missing lifetime specifier
  --> src/main.rs:3:15
   |
3  |     username: &str,
   |               ^ expected named lifetime parameter
   |
   | help: consider introducing a named lifetime parameter
```

### **How to Fix This?**
✔ **Use `String` instead of `&str`** to ensure the struct owns its data.  
✔ **Use lifetimes (`'a`)** if references **must be stored** (covered in Chapter 10).  

For now, the simplest approach is:
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

### **Key Takeaways**
**Structs** allow grouping related data with named fields.
**Instances** are created using key-value pairs inside `{}`.
**Dot notation** provides easy access to struct fields.
**Entire instances must be mutable** to modify fields.
**Field init shorthand** simplifies struct instantiation.
**Struct update syntax (`..existing_instance`)** allows efficient creation of new instances.
**Heap-allocated fields (`String`) are moved, invalidating the original instance.**
**Stack-only fields (`bool`, `u64`) are copied, leaving the original instance usable.**


## **Debugging Structs in Rust**

By default, **structs cannot be printed** using `println!`. Rust does not assume how a struct should be formatted because it could have various fields and types. However, Rust provides **traits** that allow developers to customize the printing behavior of structs.

---

### * `Debug` Trait (`#[derive(Debug)]`)**
To enable Rust to print a struct for debugging, **derive the `Debug` trait**:

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String,
    active: bool,
}

fn main() {
    let user = User {
        username: String::from("rustacean"),
        email: String::from("rust@example.com"),
        active: true,
    };

    println!("{:?}", user); // Works because of `#[derive(Debug)]`
}
```
✔ `#[derive(Debug)]` **automatically implements** the `Debug` trait for a struct.  
✔ Allows **developer-friendly printing** using `println!`.  

### * Debug Formatting with `:?`**
Once the `Debug` trait is derived, **use `:?` inside `{}`** in `println!` to print the struct:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };

    println!("Rectangle data: {:?}", rect);
}
```
✔ **Prints the entire struct** in a **compact, developer-friendly format**.  
✔ Useful for **quick debugging** but not the most readable for complex structs.

#### * Output**
```
Rectangle data: Rectangle { width: 30, height: 50 }
```

---

### * Pretty Debug Formatting with `:#?`**
For **better readability**, use `:#?` inside `{}`:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };

    println!("Rectangle data (formatted): {:#?}", rect);
}
```
✔ **Formats the struct across multiple lines** for **clearer debugging output**.  
✔ Ideal for **large structs with multiple fields**.

#### * Output**
```
Rectangle data (formatted): Rectangle {
    width: 30,
    height: 50,
}
```

---

### * `dbg!` Macro for Debugging**
The `dbg!` macro is a **powerful debugging tool** that:
- Prints **both** the **expression** and its **result**.
- **Includes file and line number** where it's called.
- **Returns the value**, so it can be used inside expressions.

#### **Example: Debugging an Expression**
```rust
fn main() {
    let scale = 2;
    let width = dbg!(30 * scale);
}
```
✔ Useful for **debugging calculations and variable assignments**.  
✔ Unlike `println!`, `dbg!` prints to **stderr (standard error)** instead of stdout.  

#### * Output**
```
[src/main.rs:4] 30 * scale = 60
```

#### **Example: Debugging a Struct**
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };

    dbg!(&rect);
}
```

#### * Output**
```
[src/main.rs:10] &rect = Rectangle {
    width: 30,
    height: 50,
}
```
✔ The `dbg!` macro shows **file name, line number, and value**.  
✔ Since `dbg!` **takes ownership**, using `&rect` ensures the struct remains accessible after logging.

---

## ** Summary**
| Debugging Method            | Usage                      | Output Type                | Best For |
|-----------------------------|----------------------------|----------------------------|----------|
| **`#[derive(Debug)]`**      | Enables debug printing     | None (derives trait)       | Required for `:?` and `dbg!` |
| **`println!("{:?}", var)`** | Quick struct printing      | Compact single-line output | Simple debugging |
| **`println!("{:#?}", var)`**| Prettified struct printing | Multi-line output          | Large structs |
| **`dbg!(expression)`**      | Debugs expressions         | Shows file, line, and value| Tracing variable values |

# **Methods in Rust: Defining and Using Methods in Structs**

## * What Are Methods?**
- **Methods** are **similar to functions**, but they are defined within the context of a **struct**, **enum**, or **trait object**.
- The **first parameter** of a method is always `self`, which represents the **instance** of the struct the method is being called on.
- Methods allow **data encapsulation** and **better organization** of related functionality.

---

## * Defining Methods in Rust**
- Methods are defined **inside an `impl` block** (short for *implementation*).
- Methods use `self` to refer to the instance they are being called on.

### **Example: Adding an `area` Method to a `Rectangle` Struct**
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Defining a method for Rectangle
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("The area of the rectangle is {} square pixels.", rect1.area());
}
```

**Key Details:**
- **`impl Rectangle {}`** → Starts the implementation block for `Rectangle`.
- **`fn area(&self) -> u32 {}`** → Defines a method **inside** the `impl` block.
- **`&self`**:
  - `self` is a reference to the instance of `Rectangle`.
  - The `&` means this method **borrows the instance immutably** (does not modify it).
- **Calling the method** → `rect1.area()` instead of `area(rect1)`, making it **more intuitive**.

---

## * Understanding `self` in Methods**
The first parameter of a method is always **`self`**, but it can be used in different ways:


**self Declarations**

**Immutable Ownership**

self: MyStruct

self: Self

self

Mutable Ownership

mut self: MyStruct

mut self: Self

mut self

Immutable Reference

self: &MyStruct

self: &Self

&self

Mutable Reference

self: &mut MyStruct

self: &mut Self

&mut self

| `self` Type      | Meaning |
|------------------|---------|
| `self`          | Takes **ownership** of the instance. |
| `&self`         | Borrows **immutably** (read-only access). |
| `&mut self`     | Borrows **mutably** (allows modification). |

### **Example: Different `self` Usage**
```rust
impl Rectangle {
    fn consume(self) {
        println!("Rectangle consumed: {:?}", self);
    }

    fn update_width(&mut self, new_width: u32) {
        self.width = new_width;
    }

    fn get_width(&self) -> u32 {
        self.width
    }
}
```
✔ **`self` (Takes ownership)** → `consume(self)` moves the instance.  
✔ **`&mut self` (Mutable Borrow)** → `update_width(&mut self, new_width)` modifies the struct.  
✔ **`&self` (Immutable Borrow)** → `get_width(&self)` reads data without modifying it.

---

## * Why Use Methods Instead of Functions?**
**More Organized Code**: Methods keep related functionality inside the `impl` block, making the code **easier to read and maintain**.  
**Method Syntax is Cleaner**: Instead of `area(rect)`, we can simply call `rect.area()`.  
**Avoids Repeating Type Definitions**: No need to repeatedly specify `Rectangle` in function parameters.  

---

## * Methods vs Fields with the Same Name**
- Rust **allows methods to have the same name as struct fields**.
- When **using parentheses**, Rust treats it as a **method**.
- Without parentheses, Rust assumes it's the **field**.

### **Example**
```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
``` **Key Takeaways:**
✔ `rect1.width()` calls the **method** `width()`.  
✔ `rect1.width` accesses the **field** `width`.  
✔ Methods like this can be used as **getters** when struct fields are private.

---

## * Automatic Referencing & Dereferencing in Methods**
- In **C/C++**, you use `.` to call a method on an object and `->` when calling it on a pointer.
- **Rust simplifies this** with **automatic referencing and dereferencing**.

### **Example**
```rust
p1.distance(&p2);
(&p1).distance(&p2); // Rust automatically adds `&`
```
✔ Rust automatically **adds `&`, `&mut`, or `*`** so that `p1` matches the method signature.  
✔ This makes **method calls ergonomic** without needing explicit pointer syntax.  

---

## * Methods with More Parameters**
- Methods **can accept multiple parameters**, just like functions.
- Example: A method that checks if one `Rectangle` **can fit inside another**.

### **Example: `can_hold` Method**
```rust
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```
Now, we can call:
```rust
let rect1 = Rectangle { width: 30, height: 50 };
let rect2 = Rectangle { width: 10, height: 40 };

println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
``` **Key Takeaways:**
✔ `rect1.can_hold(&rect2)` passes `rect2` as a **reference** (to avoid ownership transfer).  
✔ The method returns `true` if `rect1` can completely contain `rect2`.  

---

## * Associated Functions (Methods Without `self`)**
- Methods inside `impl` that **don’t take `self`** are called **associated functions**.
- These are often used as **constructors**.

### **Example: Defining a Constructor**
```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self { width: size, height: size }
    }
}
```
✔ `Self` refers to `Rectangle` inside the `impl` block.  
✔ This allows us to call `Rectangle::square(10)`, returning a `Rectangle` with equal width and height.

### **Calling an Associated Function**
```rust
let sq = Rectangle::square(10);
``` **Uses `::` instead of `.`** → `Rectangle::square(10)` instead of `sq.square()`.  

---

## * Multiple `impl` Blocks**
Rust allows multiple `impl` blocks for a struct.

### **Example**
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
``` 
**Why Use Multiple `impl` Blocks?**
✔ Helps **organize** large structs with many methods.  
✔ Useful when implementing **traits** (covered in Chapter 10).  

---

## ** Summary**
| Feature | Description |
|---------|------------|
| **Methods** | Defined inside an `impl` block, always have `self` as the first parameter. |
| **`self` Usage** | `&self` (borrow immutably), `&mut self` (borrow mutably), `self` (move ownership). |
| **Method Syntax** | `instance.method()` is cleaner than `function(instance)`. |
| **Automatic Referencing** | Rust automatically adds `&`, `&mut`, or `*` when calling methods. |
| **Multiple Parameters** | Methods can accept multiple arguments, like `can_hold(&self, other: &Rectangle)`. |
| **Associated Functions** | Methods **without `self`**, often used as **constructors** (e.g., `Rectangle::square(10)`). |
| **Multiple `impl` Blocks** | Allowed, useful for organizing large structs. |

---
## Chapter 5: Summary

## **Key Takeaways from Structs in Rust**

### **1. What Are Structs?**
- Structs **group related data together** under a single name.
- Unlike tuples, struct fields are **named**, improving code clarity.
- Each struct is a **custom type**.
- Fields can have **different types** or the same type.

### **2. Why Use Structs Instead of Tuples?**
✔ Structs provide **named fields**, making code more readable.
✔ Field **order doesn’t matter** when creating instances.
✔ They allow **self-documenting** code.
✔ Fields are accessed by **name**, not index.

---
## **3. Defining and Instantiating Structs**
- Defined using the `struct` keyword, followed by named fields.
- Example:
  ```rust
  struct User {
      active: bool,
      username: String,
      email: String,
      sign_in_count: u64,
  }
  ```
- Instances are created using `{}`:
  ```rust
  let user1 = User {
      active: true,
      username: String::from("user1"),
      email: String::from("user1@example.com"),
      sign_in_count: 1,
  };
  ```
- Fields are accessed with **dot notation** (`user1.email`).
- **Mutability applies to the whole struct instance**, not individual fields.

  ```rust
  let mut user1 = User {
      active: true,
      username: String::from("user1"),
      email: String::from("user1@example.com"),
      sign_in_count: 1,
  };
  ```

---
## **4. Struct Update Syntax**
- Creates a new struct based on an existing one.
- Uses `..existing_instance` to copy remaining fields.
- Example:
  ```rust
  let user2 = User {
      email: String::from("user2@example.com"),
      ..user1
  };
  ```
- Moves non-`Copy` fields (like `String`), invalidating the original instance.
- `Copy` types (e.g., `bool`, `u64`) remain accessible.

---
## **5. Tuple Structs and Unit-Like Structs**
- **Tuple Structs**:
  ```rust
  struct Color(i32, i32, i32);
  struct Point(i32, i32, i32);
  ```
  ✔ Useful when naming fields isn't necessary.
  ✔ Different struct types, even if fields have the same types.
  
- **Unit-Like Structs**:
  ```rust
  struct AlwaysEqual;
  ```
  ✔ Used for **traits** or **marker types** without stored data.

---
## **6. Debugging Structs (`#[derive(Debug)]`)**
- Rust doesn’t allow printing structs by default.
- Add `#[derive(Debug)]` to enable `println!("{:?}", struct_instance);`.
- Use `:#?` for pretty formatting.
- The `dbg!` macro prints expressions **with file & line number**.

---
## **7. Methods in Structs**
- Defined in an `impl` block, using `self` to reference the instance.
- Example:
  ```rust
  impl Rectangle {
      fn area(&self) -> u32 {
          self.width * self.height
      }
  }
  ```
✔ `&self` means **borrow immutably**.
✔ `&mut self` allows **modification**.
✔ `self` **takes ownership**.

---
## **8. Associated Functions (`Self` & Constructors)**
- Methods without `self` are called **associated functions**.
- Example:
  ```rust
  impl Rectangle {
      fn square(size: u32) -> Self {
          Self { width: size, height: size }
      }
  }
  ```
- Called using `Rectangle::square(10);` (not on an instance).

---
## **9. Multiple `impl` Blocks**
✔ Methods can be split into multiple `impl` blocks for better organization.
✔ Useful when implementing **traits**.

---
## * Final Summary**
| Feature | Description |
|---------|------------|
| **Structs** | Custom types with named fields |
| **Tuple Structs** | Structs without named fields |
| **Unit-Like Structs** | Structs without fields, used as markers |
| **Dot Notation** | Access fields using `.` |
| **Struct Update Syntax** | Creates new instances by copying existing fields (`..`) |
| **Debugging (`Debug` Trait)** | Enables struct printing with `#[derive(Debug)]` |
| **Methods (`impl`)** | Define behavior associated with a struct |
| **Self in Methods** | `&self` (borrow), `&mut self` (mut borrow), `self` (ownership) |
| **Associated Functions** | Methods without `self` (e.g., constructors) |
| **Multiple `impl` Blocks** | Helps organize large structs |

 **Structs in Rust provide a powerful way to model real-world data while ensuring clarity, ownership, and safety.**


- **Organize related functionality inside structs**.
- **Improve readability & encapsulation**.
- **Method syntax is ergonomic and intuitive**.
- **Allow struct-specific behavior (e.g., constructors, calculations, checks)**.

Below is the enhanced README in markdown format. All of the original content is preserved and expanded with additional explanations and logical structuring for clarity.

---

# Chapter 6: Enums and Pattern Matching in Rust

## Defining and Using Enums

### Defining an Enum

Structs allow grouping related fields together, but **enums** allow defining a **type with a set of possible values**. An enum ensures that a value is **one of** a predefined set of options.

For example, an `IpAddr` can be either IPv4 or IPv6, but **not both at the same time**. Since an enum can only be one of its variants at a time, it is the perfect data structure for representing such cases.

#### Example: Defining an Enum for IP Addresses

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

- `IpAddrKind` is a **custom data type** with two variants: `V4` and `V6`.
- These variants are **namespaced** under `IpAddrKind`.

### Creating Instances of an Enum

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

✔ The `::` syntax **associates a variant with its enum type**. Both `four` and `six` are of type `IpAddrKind`.

### Using Enums in Functions

Since `IpAddrKind` is now a type, it can be used as a parameter to functions:

```rust
fn route(ip_kind: IpAddrKind) {}

route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

✔ This approach allows handling both variants within a **single function**.

---

## Enums vs Structs for Storing Data

The `IpAddrKind` enum only defines the **type** of IP address but does not store the actual IP address itself. One alternative is to use a **struct**.

### Using a Struct to Store IP Addresses

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

```rust
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

✔ In this approach, we store **both** the type (`kind`) and the actual address (`address`).  
✔ However, this approach is **redundant** because the `kind` field determines what type of address the `address` field contains.

---

## Key Differences Between Enums and Structs

Rust provides both **structs** and **enums** for defining custom types. While both can group related data, they serve different purposes.

| **Feature**         | **Structs**                                    | **Enums**                                          |
|---------------------|------------------------------------------------|----------------------------------------------------|
| **Purpose**         | Groups related fields together                 | Represents one of multiple predefined variants     |
| **Data Storage**    | Can hold multiple fields with different types  | Each variant can have different types of associated data |
| **Exhaustiveness**  | No need to check field types explicitly         | Must handle all possible variants in a `match` expression |
| **Flexibility**     | Best when all fields are always required        | Best when a value can only be one of several options |
| **Memory Usage**    | Always stores all fields                         | Stores only the data for the current variant       |

### When to Use Structs

Structs are best when:
- You need to **group related fields together** to form a composite type.
- Each instance **must contain all fields** (for example, a `Rectangle` with `width` and `height`).
- You want **clear field names** to improve documentation and readability.

#### Example: Using a Struct for a User Profile

```rust
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

let user1 = User {
    username: String::from("rustacean"),
    email: String::from("user@example.com"),
    active: true,
    sign_in_count: 1,
};
```

✔ **Why a Struct?**  
- `username` and `email` are **always required**.
- `sign_in_count` should be **tracked for all users**.

### When to Use Enums

Enums are best when:
- A value must be **one of a predefined set of options**.
- Different variants may store **different types of data**.
- You need **pattern matching** to enforce exhaustive handling.

#### Example: Representing an IP Address with an Enum

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

✔ **Why an Enum?**  
- An IP address can be **either** IPv4 or IPv6, never both.
- Different variants store **different types of data** (`V4` uses four `u8` values, while `V6` stores a `String`).
- The `match` construct ensures **all cases are handled** safely.

---

## Enum Variants: Tuple vs Struct Variants

Enums in Rust are versatile because **each variant** can store **different kinds of data**. There are two common styles for variants:

### Tuple Variant

```rust
enum PaymentMethodType {
    CreditCard(i32),
    DebitCard(String),
    PayPal(String),
}
```

- In this style, **each variant stores a single value** (or multiple unnamed values), similar to a tuple.
- For instance:
  - `CreditCard(String)` Will store a String.
  - `DebitCard(i32)` Will store a i32.
  - `PayPal(String)` Will store a String.

**When to use tuple variants:**  
Use tuple variants when **only a few values** need to be stored and you **do not need to name them** individually because their meaning is clear from context.

#### Example Use Case (Tuple Variant)

```rust
let payment = PaymentMethodType::CreditCard(String::from("1234-5678-9012-3456"));
```

The `String` clearly holds the card number without requiring a named field.

### Struct Variant

```rust
enum PaymentMethodType {
    CreditCard(String),
    DebitCard(String),
    PayPal { username: String, password: String },
}
```

- In this style, a variant can be defined with **named fields** inside it.
- `PayPal { username: String, password: String }` explicitly names the data it holds.

**When to use struct variants:**  
Use struct variants when **multiple pieces of data** need to be stored in a variant and naming them makes your code more **self-documenting** and clearer.

#### Example Use Case (Struct Variant)

```rust
let payment = PaymentMethodType::PayPal { 
    username: String::from("user@example.com"), 
    password: String::from("securepassword123") 
};
```

Here, both the username and password are explicitly labeled for clarity.

---

## Nested Enums

Nested enums are simply enums used inside other enums. This technique is useful for organizing complex data structures by breaking them into smaller, more manageable pieces.

### Example: Nested Enums

```rust
enum Meat {
    Chicken,
    Steak,
}

enum Beans {
    Pinto,
    Black,
}

enum RestaurantItem {
    Burrito { meat: Meat, beans: Beans },
    Bowl(Meat),
    VeganPlate,
}
```

#### What’s Happening Here?

- `Meat` is an enum with two options: `Chicken` or `Steak`.
- `Beans` is an enum with two options: `Pinto` or `Black`.
- `RestaurantItem` is the **main enum** that uses `Meat` and `Beans` in some of its variants:
  - `Burrito { meat: Meat, beans: Beans }`: a Burrito includes both **meat** and **beans**.
  - `Bowl(Meat)`: a Bowl contains only **meat**.
  - `VeganPlate`: a Vegan plate contains **neither meat nor beans**.

#### Why Use Nested Enums?

- **Cleaner Code:** Decomposes complex types into simpler, easily understandable parts.
- **Stronger Type Safety:** Rust enforces valid values at compile time.
- **Reusability:** The same enums (`Meat` and `Beans`) can be reused elsewhere in your code.

#### Practical Example (Using Nested Enums)

```rust
let order1 = RestaurantItem::Burrito {
    meat: Meat::Steak,
    beans: Beans::Pinto,
};

let order2 = RestaurantItem::Bowl(Meat::Chicken);

let order3 = RestaurantItem::VeganPlate;
```

And pattern matching on these values:

```rust
match order1 {
    RestaurantItem::Burrito { meat, beans } => {
        println!("Burrito with {:?} and {:?} beans!", meat, beans);
    },
    RestaurantItem::Bowl(meat) => {
        println!("Bowl with {:?}!", meat);
    },
    RestaurantItem::VeganPlate => {
        println!("Vegan Plate with no meat or beans!");
    },
}
```

*(Note: You would need to add `#[derive(Debug)]` to the `Meat` and `Beans` enums to print them with `{:?}`.)*

---

## Enums and `match`: Comprehensive Usage

### What is `match`?

Rust’s `match` is a control flow construct that checks a value against multiple patterns and executes code based on which pattern matches. It is **exhaustive** by design, meaning every possible case must be handled.

#### Matching on a Basic Enum

```rust
fn process_payment(method: PaymentMethod) {
    match method {
        PaymentMethod::CreditCard => println!("Paid with Credit Card"),
        PaymentMethod::DebitCard => println!("Paid with Debit Card"),
        PaymentMethod::PayPal => println!("Paid with PayPal"),
    }
}
```

- The `match` expression evaluates the value of `method`.
- Each arm handles one variant, and all variants must be covered.

### Tuple Enum Variants

Enums can store additional data using tuple variants.

```rust
enum PaymentMethod {
    CreditCard(String),
    DebitCard(String),
    PayPal(String),
}
```

Each variant carries a `String`. For example:

```rust
let method = PaymentMethod::CreditCard(String::from("1234-5678-9012-3456"));
```

#### Matching a Tuple Enum Variant

```rust
fn process_payment(method: PaymentMethod) {
    match method {
        PaymentMethod::CreditCard(number) => println!("Paid with Credit Card: {number}"),
        PaymentMethod::DebitCard(number) => println!("Paid with Debit Card: {number}"),
        PaymentMethod::PayPal(email) => println!("Paid with PayPal account: {email}"),
    }
}
```

- The inner value (e.g., `number` or `email`) is bound to a variable and used inside the match arm.

### Struct Enum Variants

Struct variants allow you to define named fields inside an enum.

```rust
enum PaymentMethod {
    CreditCard(String),
    DebitCard(String),
    PayPal { username: String, password: String },
}
```

Now, `PayPal` holds two named fields: `username` and `password`.

#### Matching a Struct Enum Variant

```rust
fn process_payment(method: PaymentMethod) {
    match method {
        PaymentMethod::CreditCard(number) => println!("Paid with Credit Card: {number}"),
        PaymentMethod::DebitCard(number) => println!("Paid with Debit Card: {number}"),
        PaymentMethod::PayPal { username, password } => {
            println!("Paid with PayPal username: {username} and password: {password}")
        },
    }
}
```

- This allows multiple values to be extracted by name inside the match arm.


### Using Enum in `if` / `else` (`if let`)

Instead of `match`, you can use **`if let`** with enums for cleaner code:

```rust
if let PaymentMethod::CreditCard(number) = &method {
    println!("Credit Card: {}", number);
} else {
    println!("Not a Credit Card.");
}
```

✔ `if let` checks one variant and automatically borrows the fields.

---

### Enum Methods

You can define methods on enums using an `impl` block, similar to structs.

```rust
enum PaymentMethod {
    CreditCard(String),
    DebitCard(String),
    PayPal { username: String, password: String },
}

impl PaymentMethod {
    fn pay(&self) {
        match self {
            PaymentMethod::CreditCard(number) => println!("Processing Credit Card: {number}"),
            PaymentMethod::DebitCard(number) => println!("Processing Debit Card: {number}"),
            PaymentMethod::PayPal { username, .. } => println!("Processing PayPal for {username}"),
        }
    }
}
```

Usage:

```rust
let method = PaymentMethod::DebitCard(String::from("8765-4321-0000-1111"));
method.pay();
```

- This defines an **associated method** on the enum that can be called directly on its instance.

### Using the Wildcard Pattern `_`

Sometimes you may not care about some variants. The wildcard `_` pattern matches all cases not handled explicitly:

```rust
match method {
    PaymentMethod::CreditCard(number) => println!("Credit Card: {number}"),
    _ => println!("Other payment method"),
}
```

- The `_` pattern ensures that any unmatched case is handled without listing every possible variant.

---

## The `Option<T>` Enum: Safety Over Null

### Why Does Rust Use `Option<T>` Instead of `null`?

Many languages use `null` to represent an absent value, but `null` is notorious for causing unexpected errors when a value is assumed to be present. Tony Hoare called it the **“billion-dollar mistake”** due to the significant impact of null-related bugs.

#### Problems with `null` in Other Languages

- **Unintentional Errors:** Using a `null` value where a valid value is expected often results in crashes.
- **Hidden Bugs:** Failing to check for `null` leads to runtime exceptions.
- **Lack of Type Safety:** `null` can be assigned to any type, making errors harder to catch at compile time.

Rust eliminates these issues by not having `null` at all. Instead, it uses the `Option<T>` enum, which explicitly represents the possibility of absence.

---

## Borrowing Rules in Enums

In Rust, **enums follow the same ownership and borrowing rules** as other types.

You can:
- **Own** an enum value.
- **Borrow** it immutably (`&`).
- **Borrow** it mutably (`&mut`).

---

### Owning an Enum

When you create an enum, you **own** the value.

```rust
enum PaymentMethod {
    CreditCard(String),
    PayPal(String),
}

let method = PaymentMethod::CreditCard(String::from("1234-5678-9012-3456"));
```

✔ Ownership is moved when passing the enum unless you borrow it.

---

### Immutable Borrowing (`&`)

Borrow the enum to read it without taking ownership.

```rust
fn describe(method: &PaymentMethod) {
    match method {
        PaymentMethod::CreditCard(number) => println!("Card ending: {}", &number[number.len()-4..]),
        PaymentMethod::PayPal(email) => println!("PayPal: {}", email),
    }
}

let method = PaymentMethod::PayPal(String::from("user@example.com"));
describe(&method); // Borrowed
```

✔ After calling, `method` is still usable because it was borrowed.

---

### Mutable Borrowing (`&mut`)

Borrow mutably to modify inner data.

```rust
fn update_email(method: &mut PaymentMethod) {
    if let PaymentMethod::PayPal(email) = method {
        email.push_str(".verified");
    }
}

let mut method = PaymentMethod::PayPal(String::from("user@example.com"));
update_email(&mut method);
```

✔ `&mut` gives write access to the inner data.

---

### Moving Out of an Enum

Moving inner data requires **owning** the enum.

```rust
fn extract_email(method: PaymentMethod) -> Option<String> {
    if let PaymentMethod::PayPal(email) = method {
        Some(email) // Moves email
    } else {
        None
    }
}
```

✔ Cannot move fields out of a borrowed enum.

---

### Borrowing in `match`

When matching:
- `match &enum` → borrows fields.
- `match &mut enum` → borrows fields mutably.

You can also use `ref` or `ref mut` inside `match`:

```rust
match method {
    PaymentMethod::CreditCard(ref number) => println!("Card: {}", number),
    _ => (),
}
```

---

## Quick Summary

| Action           | Behavior |
|------------------|----------|
| Own enum         | Can move data inside. |
| `&Enum`          | Read-only access. |
| `&mut Enum`      | Modify inner data. |
| Move inner field | Only with full ownership. |

Got it!  
You want two things added, **but still compact**:

1. **Mention common borrow/move error** (`cannot move out of borrowed content`).  
2. **Show how Enums work in `if` / `else` without `match`**, like using `if let` style.

Here’s the updated, small **Borrowing Rules in Enums** section ready for your README:

---

### Matching with Borrowed Enums

When matching:
- `match &enum` → borrows fields.
- `match &mut enum` → mutable borrows.

Use `ref` or `ref mut` if matching owned enums:

```rust
match method {
    PaymentMethod::CreditCard(ref number) => println!("Card: {}", number),
    _ => (),
}
```

---

### What is `Option<T>`?

`Option<T>` is defined as follows:

```rust
enum Option<T> {
    None,       // Represents the absence of a value
    Some(T),    // Holds a value of type T
}
```

✔ `Option<T>` is generic and can hold any type.  
✔ `Some(T)` stores an actual value of type `T`.  
✔ `None` represents the absence of a value.

#### Using `Option<T>` in Rust

```rust
let some_number = Some(5);
let some_char = Some('e');
let absent_number: Option<i32> = None;
```

- `some_number` is of type `Option<i32>` and contains `5`.
- `absent_number` is of type `Option<i32>`, representing no value.

#### Preventing `null`-Related Bugs

Rust forces you to explicitly handle the case where a value might be missing. For example, the following code results in a compiler error:

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);
let sum = x + y; // ERROR: Cannot add i8 and Option<i8>
```

Instead, you must handle both cases:

```rust
let value = y.unwrap_or(0); // If y is Some(5), value becomes 5; if None, value becomes 0.
println!("Value: {}", value);
```

✔ This explicit handling prevents accidental operations on non-existent values.

---

## The `match` Control Flow Construct in Rust

### What is `match`?

Rust’s `match` is a control flow tool that compares a value against multiple patterns and executes code based on the match. It is akin to a coin-sorting machine:
- Each coin (value) goes through different “tracks” (patterns).
- It falls into the first track that matches its size (pattern).
- The corresponding code block executes once a match is found.

### Using `match` with Enums

For example, consider determining the value of different US coins:

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

✔ The `match` expression checks `coin` against each variant and returns the corresponding value.

### Adding Multiple Lines in Match Arms

If you need multiple lines of code in a match arm, enclose the code in braces `{}`:

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

✔ When `Coin::Penny` matches, a message is printed before returning `1`.
✔ The last expression within the braces is the arm’s return value.

### Matching Enum Variants with Associated Data

Some enum variants carry associated data. `match` can extract this data for further use.

#### Example: Matching a Quarter’s State

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // ... other states
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

✔ If `Coin::Quarter(state)` is matched, `state` binds to the quarter’s state value.  
✔ This technique is useful for handling enums with associated data.

### Using `match` with `Option<T>`

`Option<T>` is commonly matched to safely extract a value:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
```

- If `x` is `Some(i)`, the function returns `Some(i + 1)`.
- If `x` is `None`, it returns `None`.

#### Example Calls:

```rust
let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

✔ `six` holds `Some(6)` while `none` remains `None`.

### Exhaustiveness and Catch-All Patterns

Rust requires that every possible case is handled in a match expression. If a variant is omitted, the compiler produces an error:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        // Compiler error if `None` is not covered.
    }
}
```

A catch-all pattern `_` can be used to handle all remaining cases:

```rust
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => move_player(dice_roll),
}
```

✔ The wildcard `_` pattern ensures that any unmatched value is processed.

---

## Concise Control Flow with `if let`

### What is `if let`?

Rust offers `if let` as a concise alternative to `match` when you only care about one specific pattern.

Instead of writing a full match expression when only one case matters, you can use `if let` to reduce boilerplate.

#### Example: Using `if let`

```rust
if let Some(n) = maybe_number {
    println!("Number: {}", n);
}
```

- If `maybe_number` is `Some(n)`, the block runs; otherwise, it is skipped.

### `match` vs. `if let`

#### Using `match` (Verbose Approach)

```rust
let config_max = Some(3u8);

match config_max {
    Some(max) => println!("The maximum is configured to be {max}"),
    _ => (), // Explicitly handle the None case
}
```

#### Using `if let` (Concise Approach)

```rust
let config_max = Some(3u8);

if let Some(max) = config_max {
    println!("The maximum is configured to be {max}");
}
```

✔ `if let` reduces the need for an explicit `_ => ()` case, making the code cleaner.

#### Handling Unmatched Cases with `if let ... else`

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

✔ If `coin` is not a `Quarter`, the else block is executed.

---

## Chapter 6 Summary - Enums, Option, match, and if let

### 1. What Are Enums?
- **Enums** allow defining a type with a fixed set of possible variants.
- An enum instance can hold **only one of its variants** at a time.

#### Example: Enum for IP Addresses

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
```

- `V4` stores four `u8` values; `V6` stores a `String`.
- Enums allow each variant to store **different types of data**.

✔ **Why Use an Enum?**
- Ensures a value is **only one of the possible variants**.
- Reduces redundancy by eliminating extra fields.

### 2. Enums vs Structs: When to Use Each

| **Feature**            | **Structs**                         | **Enums**                          |
|------------------------|-------------------------------------|------------------------------------|
| Grouping Data          | Related fields that must always be together | One of multiple predefined variants |
| Data Flexibility       | All fields are always required      | Each variant can store different data types |
| Pattern Matching       | Not applicable                      | Requires exhaustive handling of all cases |

✔ **Use Structs** when all fields are required together.  
✔ **Use Enums** when only **one variant** is active at a time.  
✔ **Use Structs inside Enums** when variants need additional structured data.

### 3. The `Option<T>` Enum and Why Rust Has No `null`
- `Option<T>` represents either `Some(T)` (a value) or `None` (no value).
- Eliminates many common null-related bugs seen in other languages.

#### Example: Using `Option<T>`

```rust
let some_number = Some(5);
let absent_number: Option<i32> = None;
```

✔ Rust forces you to **explicitly handle** the absence of a value.

### 4. The `match` Control Flow Construct
- `match` compares a value against multiple patterns.
- Rust enforces **exhaustiveness**, ensuring every case is handled.

#### Example: Matching on a Coin Enum

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

✔ The wildcard `_` can catch any unhandled cases.

### 5. `if let`: A Concise Alternative to `match`
- **`if let`** is useful when you only care about one pattern.
- It makes code more concise while still providing safe pattern matching.

#### Example: `match` vs. `if let`

```rust
// Using match (verbose)
match config_max {
    Some(max) => println!("Max is {max}"),
    _ => (),
}

// Using if let (concise)
if let Some(max) = config_max {
    println!("Max is {max}");
}
```

✔ `if let` reduces boilerplate when handling a single case.

### 6. Key Takeaways
✔ **Enums provide a way to define a type with multiple possible variants.**  
✔ **Each variant can store different types of data**, eliminating the need for extra structs.  
✔ Rust’s `Option<T>` replaces `null` by forcing explicit handling of absent values.  
✔ The `match` construct ensures that every case is handled safely and exhaustively.  
✔ **`if let` offers a concise syntax** for matching a single case.

---

# Chapter 7 - Managing Growing Projects with Packages, Crates, and Modules

## **1. Introduction: Why Organize Code?**
As Rust projects grow, managing code effectively becomes essential. Rust provides a **module system** that helps organize code into **packages, crates, and modules**. This system allows:
- **Better code organization** through modularization.
- **Encapsulation** of implementation details.
- **Efficient code reuse** across multiple files and projects.
- **Control over visibility and scope**, making APIs cleaner and safer.

### **Key Terms in Rust’s Module System**
| Feature | Purpose |
|---------|---------|
| **Package** | A collection of one or more crates with a `Cargo.toml` file. |
| **Crate** | A compilation unit in Rust that can be a **binary** (executable) or **library** (reusable code). |
| **Module** | A way to group related functionality within a crate. |
| **Paths** | A way to **refer** to functions, structs, and modules in the project. |
| **Scopes** | Control over what parts of the code can access certain items. |

---
## **2. Packages and Crates**
A **crate** is the smallest unit of code compilation in Rust. A **package** is a collection of **one or more crates**.

### **Binary Crates vs. Library Crates**
| Type | Description |
|------|-------------|
| **Binary Crate** | Compiles to an **executable** (must contain a `main` function). |
| **Library Crate** | A collection of reusable functions and modules (no `main` function). |

### **Creating a New Package with Cargo**
```sh
$ cargo new my_project
```
This creates the following structure:
```
my_project
├── Cargo.toml  # Package metadata
└── src
    └── main.rs # Crate root for the binary crate
```
- `Cargo.toml` defines **dependencies and settings**.
- `src/main.rs` is the **crate root** for a binary crate.
- To create a **library crate**, replace `main.rs` with `lib.rs`.

### **Adding Multiple Binary Crates**
To create multiple binary crates, add Rust files inside `src/bin/`:
```
my_project
├── src
│   ├── main.rs       # Binary crate (default)
│   ├── bin
│   │   ├── tool.rs   # Additional binary crate
│   │   └── cli.rs    # Another binary crate
```
Each file inside `src/bin/` becomes **a separate binary crate**.

---
## **3. Modules: Organizing Code Inside a Crate**
Modules allow **logical grouping** of functions, structs, and other definitions inside a crate.

### **Declaring a Module**
A module is declared using the `mod` keyword:
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}
```
- The **crate root** is `src/main.rs` (for binaries) or `src/lib.rs` (for libraries).
- The **module tree** represents a hierarchy of modules.

### **Module Tree Representation**
```
crate
└── front_of_house
    ├── hosting
    │   └── add_to_waitlist
    ├── seat_at_table
    └── serving
        ├── take_order
        ├── serve_order
        └── take_payment
```
- Modules act **like directories** inside a filesystem.
- Each module can contain **submodules, functions, structs, and constants**.

---
## **4. Module File Structure in Rust**
By default, Rust looks for module files in:
| Declaration | File Location |
|-------------|--------------|
| `mod garden;` (in `src/lib.rs`) | `src/garden.rs` OR `src/garden/mod.rs` |
| `mod vegetables;` (in `src/garden.rs`) | `src/garden/vegetables.rs` OR `src/garden/vegetables/mod.rs` |

### **Example: Structuring a Library Crate**
```
restaurant
├── Cargo.toml
└── src
    ├── lib.rs             # Crate root
    ├── front_of_house.rs  # Module
    ├── back_of_house
    │   ├── kitchen.rs     # Submodule
    │   └── staff.rs       # Submodule
```
Inside `lib.rs`, we declare:
```rust
mod front_of_house;
mod back_of_house;
```
Rust will then **look for `front_of_house.rs` and `back_of_house.rs` files**.

---
## **5. Controlling Scope & Visibility with `pub`**
By default, everything inside a module is **private**.

### **Making Modules and Functions Public**
```rust
pub mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```
✔ `pub mod` makes the **module itself** public.
✔ `pub fn` makes the **function** inside the module public.

### **Accessing Public Functions**
```rust
crate::front_of_house::hosting::add_to_waitlist();
```
✔ The function must be **public (`pub`)** in every parent module for this to work.

---
## **6. Paths and the `use` Keyword**
### **Absolute Paths** (From the crate root)
```rust
crate::front_of_house::hosting::add_to_waitlist();
```
✔ Starts with `crate::` and provides a **full path**.

### **Relative Paths** (From the current module)
```rust
front_of_house::hosting::add_to_waitlist();
```
✔ Shorter but works only within the **same module**.

### **Using `use` for Shorter Paths**
```rust
use crate::front_of_house::hosting;
hosting::add_to_waitlist();
```
✔ `use` brings a **module or item into scope**, reducing repetition.
✔ Instead of calling `crate::front_of_house::hosting::add_to_waitlist()`, we now call `hosting::add_to_waitlist()`.

---
## **Summary of Key Concepts**
✔ **A package** contains one or more **crates** (Rust projects).  
✔ **A crate** is the smallest compilation unit and can be a **binary or library**.  
✔ **Modules** help **organize code** into separate sections.  
✔ **Rust enforces privacy**: everything inside a module is private unless marked `pub`.  
✔ **Absolute paths start from `crate::`**, while **relative paths depend on the current module**.  
✔ **Use `use` to bring items into scope**, making paths shorter and cleaner.  

---

# **Paths for Referring to Items in the Module Tree**

In Rust, **paths** are how we navigate the **module system** to access functions, structs, enums, and other items. Paths in Rust work similarly to **file paths in a computer's filesystem**, where we use directories and subdirectories to organize files.

### **Two Types of Paths in Rust**
1. **Absolute Paths** – Always start from the **crate root** and specify the **full path**.
2. **Relative Paths** – Start from the **current module** and navigate using `self`, `super`, or direct names.

Both types of paths use the `::` (**double colons**) separatoI've expanded on this section with detailed explanations, real-world analogies, and examples to make it easier to understand. Let me know if anything is still unclear! 
r, similar to how files are structured with `/` in UNIX-like systems or `\` in Windows.

---

## **Absolute Paths in Rust**
An **absolute path** always starts from the **crate root** and specifies the full path to the item.
 **Example (Absolute Path)**
```rust
crate::front_of_house::hosting::add_to_waitlist();
```
✔ The path starts with `crate`, which means it refers to an item **starting from the root** of the crate.
✔ Then, it navigates through `front_of_house -> hosting -> add_to_waitlist`.

 **Filesystem analogy**:  
Imagine `crate::` is like `/` in Linux or `C:\` in Windows:
```
/front_of_house/hosting/add_to_waitlist
C:\front_of_house\hosting\add_to_waitlist
```
---

## **Relative Paths in Rust**
A **relative path** starts from the **current module** instead of the crate root. This is useful when working with items **in the same scope**.
 **Example (Relative Path)**
```rust
front_of_house::hosting::add_to_waitlist();
```
✔ This starts **from the current module** rather than the crate root.
✔ It assumes `front_of_house` exists **within the same module** where this function is called.

 **Filesystem analogy**:  
Imagine you’re inside `/front_of_house/` and want to access `hosting/add_to_waitlist`:
```
hosting/add_to_waitlist  (relative)
```
---

## **When to Use Absolute vs. Relative Paths?**
| **Scenario**                                  | **Use Absolute Path**| **Use Relative Path** |
|-------------                                  |----------------------|-----------------------|
| Referencing items from anywhere in the crate  |                   |                    |
| Moving code within the same module            |                   |                    |
| Code organization is stable and won’t move    |                   |                    |
| Code structure may change in the future       |                   |                    |

 **Best practice:**  
Use **absolute paths** when code is spread across different modules.  
Use **relative paths** when referencing nearby items that might move together.

---

## **Access Control**

 **Problem:** By default, all **modules, functions, structs, and enums are private** within their parent module.  
 **Rust’s rule:** **Child modules cannot be accessed from outside unless explicitly made public.**  

---

# **Exposing Paths with the `pub` Keyword**
To **allow access to private modules and functions**, we use the `pub` (**public**) keyword.
 **Fixing the Error: Making `hosting` Public**
```rust
mod front_of_house {
    pub mod hosting { // Now public
        pub fn add_to_waitlist() { // Now public
            println!("Added to waitlist");
        }
    }
}

pub fn eat_at_restaurant() {
    // Now works because hosting and add_to_waitlist are public!
    crate::front_of_house::hosting::add_to_waitlist();
}
```
✔ Now both `hosting` and `add_to_waitlist()` are `pub`, meaning they **can be accessed** from outside.

---

## **Important Rules About `pub`**
- **Making a module public (`pub mod`) only makes the module visible**, but not its contents.
- **Making an item public inside a public module (`pub fn`) allows external code to use it.**
- **By default, Rust follows "Encapsulation":**  
  - Modules **hide** their implementation details unless explicitly exposed.

 **Think of a restaurant kitchen:**  
- Customers can **see the menu** (public functions).
- They **cannot see how the food is prepared** (private functions inside a module).

---

## **Best Practices for Modules in a Package**
A **Rust package** often contains **both a library and a binary**. The **best practice** is to:
1. **Define all logic inside a `lib.rs`** (library crate).
2. **Use `main.rs`** (binary crate) **only to call library functions**.
 **Example**
```
my_project/
 ├── Cargo.toml
 ├── src/
 │   ├── main.rs  // Binary crate
 │   ├── lib.rs   // Library crate (main logic)
 │   ├── restaurant/
 │   │   ├── front_of_house.rs
 │   │   ├── back_of_house.rs
```
 This way, other projects can **reuse your library crate** without running the binary.

---

## **Using `super` to Refer to Parent Modules**
If a function **needs to call another function in its parent module**, we can use **`super`** (like `..` in file paths).
 **Example (Calling Parent Module Using `super`)**
```rust
fn deliver_order() {
    println!("Order delivered!");
}

mod back_of_house {
    pub fn fix_incorrect_order() {
        cook_order();
        super::deliver_order(); // Calls parent module function
    }

    fn cook_order() {
        println!("Cooking order!");
    }
}
```
✔ `super::deliver_order();` moves **one level up** in the module tree to find `deliver_order()`.

 **Filesystem analogy**:
```
../deliver_order  // Moves up one directory
```

---

# **Making Structs and Enums Public**
### **Structs (`pub struct`)**
- **Making a struct public (`pub struct`) does NOT make its fields public.**
- **You must use `pub` for each field individually.**
 **Example**
```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,       // Public field
        seasonal_fruit: String,  // Private field
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    let mut meal = back_of_house::Breakfast::summer("Rye");

    // Allowed: `toast` is public
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast", meal.toast);

    // Error: `seasonal_fruit` is private
    // meal.seasonal_fruit = String::from("blueberries");
}
```
✔ `toast` is public, so it can be changed.  
 `seasonal_fruit` is private, so **it cannot be accessed** outside the module.

---

### **Enums (`pub enum`)**
- **Making an enum public (`pub enum`) automatically makes all its variants public**.
 **Example**
```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```
✔ Since `Appetizer` is `pub`, **all of its variants** are also **public**.  
✔ **Unlike structs, enum variants do not need `pub` individually.**

---

## **Key Takeaways**
✔ **Absolute paths** start from `crate::` and always refer to a fixed location.  
✔ **Relative paths** start from the current module and allow flexible navigation.  
✔ **Private by default:** Functions, structs, and modules must be explicitly marked `pub` to be accessible.  
✔ **Struct fields require individual `pub` declarations**, but **enum variants are automatically public**.  
✔ **Use `super` to reference a parent module**, similar to `..` in file paths.  

### **Bringing Paths into Scope with `use`**  

When working with Rust modules, writing long paths repeatedly can be **tedious and error-prone**. The `use` keyword allows us to create **shortcuts** to these paths, making our code **cleaner and more readable**.

---

## **1. Why Use `use`?**  
Instead of writing **full module paths** every time, `use` lets us create a shortcut for easy access.

### **Example Without `use` (Verbose Approach)**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Added to waitlist!");
        }
    }
}

pub fn eat_at_restaurant() {
    // Calling function using full path (verbose)
    crate::front_of_house::hosting::add_to_waitlist();
}
```
✔ This works, but it's **repetitive** and **hard to read**.  

### **Example Using `use` (Cleaner Approach)**
```rust
use crate::front_of_house::hosting; // Create a shortcut

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Simpler!
}
```
✔ Now, **`hosting::add_to_waitlist()`** is shorter and cleaner.  

### **How `use` Works**
- `use` **creates a shortcut** to a path.
- The path **must still respect Rust’s privacy rules** (i.e., modules and functions must be `pub`).
- `use` **only applies to the scope it’s defined in**.

---

## **2. Scope Rules of `use`**  

`use` **only works within the scope where it is declared**. If another module needs the shortcut, it must declare `use` separately.

### **Example: `use` Scope Limitation**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting; // `use` applied at crate level

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist(); // ERROR! `use` is not in this module
    }
}
```
 **Error: `hosting` is not defined in `customer`**  
✔ To fix this, we **move `use` inside `customer`**:

```rust
mod customer {
    use crate::front_of_house::hosting; // Declare `use` inside `customer`
    
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist(); // Now it works!
    }
}
```
---

## **3. Idiomatic `use` Paths**  

When importing items, Rust follows certain **best practices**:

### **👎 Bad (Unclear Code)**
```rust
use crate::front_of_house::hosting::add_to_waitlist;
pub fn eat_at_restaurant() {
    add_to_waitlist(); // Where does this function come from?
}
```
 **Problem**: It’s not obvious that `add_to_waitlist` is from `hosting`.  

### **👍 Good (Idiomatic Rust)**
```rust
use crate::front_of_house::hosting; // Import the module
pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Clear where it comes from
}
```
✔ This **reduces repetition** while keeping clarity.  

### **Exception: Structs and Enums**
For **structs, enums, and traits**, it's common to import them **directly**:

```rust
use std::collections::HashMap; // Idiomatic for structs and enums

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```
✔ **Structs & enums** are usually used alone, so importing them directly is fine.

---

## **4. Handling Conflicts: Same Name Imports**
Rust **does not allow** importing two items with the **same name**.  

### **Example of Conflict**
```rust
use std::fmt::Result;
use std::io::Result; // ERROR: Two `Result`s in the same scope!

fn function1() -> Result {
    // std::fmt::Result
}

fn function2() -> Result<()> {
    // std::io::Result
}
```
✔ **Fix 1: Keep Parent Modules**
```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {}  // Clearly from `std::fmt`
fn function2() -> io::Result<()> {}  // Clearly from `std::io`
```

✔ **Fix 2: Rename Using `as`**
```rust
use std::fmt::Result;
use std::io::Result as IoResult; // Renamed to avoid conflict

fn function1() -> Result {} // `std::fmt::Result`
fn function2() -> IoResult<()> {} // `std::io::Result`
```
---

## **5. Re-exporting with `pub use`**  

Sometimes, we want to **make an imported module available for external use**.  
This is useful when you **want to provide a cleaner API** for users of your library.

### **Example Without `pub use`**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    crate::front_of_house::hosting::add_to_waitlist(); // External users must write full path!
}
```
 **Problem**: External users **must know** the internal structure (`front_of_house::hosting`).

---

### **Example With `pub use` (Re-exporting)**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting; // Expose `hosting` to external users

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Now accessible externally as `restaurant::hosting::add_to_waitlist`
}
```
✔ Now, external users can just **write**:
```rust
restaurant::hosting::add_to_waitlist();
```
✔ This **hides unnecessary internal structure**.

---

## **6. Importing External Crates**
If you use an **external crate**, you must:
1. **Add it to `Cargo.toml`**
2. **Use `use` to bring it into scope**

### **Example: Importing `rand`** **Cargo.toml**
```toml
[dependencies]
rand = "0.8.5"
```
 **Rust Code**
```rust
use rand::Rng; // Import the `Rng` trait

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```
✔ **External crates must always be prefixed with their name (`rand::`)**.

---

## **7. Using Nested Paths (`{}`)**
When importing **multiple items** from the same module, you can **group them**.

### **👎 Bad (Too Many `use` Lines)**
```rust
use std::cmp::Ordering;
use std::io;
```
✔ **Better Using Nested Paths**
```rust
use std::{cmp::Ordering, io}; // Cleaner and shorter
```

✔ **You Can Also Use `self` for Subpaths**
```rust
use std::io::{self, Write}; // Brings `std::io` and `std::io::Write`
```
✔ **Glob Import (`*`)**: Import **everything** from a module  Use carefully!)
```rust
use std::collections::*; // Can lead to conflicts!
```

---

## **Key Takeaways**
✔ **`use` allows importing modules for easier access.**  
✔ **`use` applies only in the scope it's declared.**  
✔ **Idiomatic Rust imports parent modules (`use module_name`), not individual functions.**  
✔ **Avoid conflicts by using parent modules or renaming imports with `as`.**  
✔ **Use `pub use` to re-export modules and simplify external APIs.**  
✔ **External crates require adding dependencies in `Cargo.toml` first.**  
✔ **Use nested paths (`{}`) to reduce redundancy in multiple imports.**  

### **Separating Modules into Different Files**  

So far, we've been defining **all modules within a single file** (like `lib.rs` or `main.rs`). However, as projects grow, keeping everything in one file makes navigation **difficult** and **clutters the code**.  

Rust allows you to **split modules into separate files** to keep the project organized and improve readability.

---

## **1. Moving a Module to a Separate File**  
Let’s take the **existing module structure** and extract `front_of_house` into its own file.  

### **Before (Everything in `lib.rs`):**  
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Added to waitlist!");
        }
    }
}

pub use crate::front_of_house::hosting; // Re-exporting for easier access

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Function call
}
```
✔ Everything works, but **all code is in one file**, making it harder to manage.

---

### **Step 1: Move `front_of_house` to `front_of_house.rs`**  
Instead of defining `front_of_house` inside `lib.rs`, we move it into a **separate file**.
 **Modify `src/lib.rs`:**  
```rust
mod front_of_house; // Declare module (Rust will look for `front_of_house.rs`)

pub use crate::front_of_house::hosting; // Re-exporting

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
``` **Create `src/front_of_house.rs`:**  
```rust
pub mod hosting {
    pub fn add_to_waitlist() {
        println!("Added to waitlist!");
    }
}
```
✔ **Now, `front_of_house` lives in its own file** (`src/front_of_house.rs`), but everything still works!

---

## **2. Moving a Submodule to Its Own File**  
Now, let’s move `hosting` **out of `front_of_house.rs`** into its own file.  

### **Step 2: Modify `src/front_of_house.rs` to Only Declare `hosting`** **New `src/front_of_house.rs`:**  
```rust
pub mod hosting; // Declare submodule (Rust will look for `front_of_house/hosting.rs`)
``` **Create `src/front_of_house/hosting.rs`:**  
```rust
pub fn add_to_waitlist() {
    println!("Added to waitlist!");
}
```
✔ **Now, both modules are in separate files**, improving organization.

---

## **3. How Rust Locates Modules in Files**  
Rust **automatically** searches for module files based on the `mod` keyword:  

| Module Declaration   | Rust Searches in These Locations |
|----------------------|--------------------------------|
| `mod front_of_house;` | `src/front_of_house.rs` |
| `pub mod hosting;` (inside `front_of_house.rs`) | `src/front_of_house/hosting.rs` |

 **Key Rule:** `mod` is **not** an "include" statement like in C or C++. It simply tells Rust **where to find the module**.

---

## **4. Alternative Module File Paths (`mod.rs` - Old Style)**  
Older Rust projects used **`mod.rs` files**, but this style is now discouraged.

| **Module**        | **New (Preferred) Style** | **Old Style (Still Supported)** |
|------------------|-------------------------|------------------------------|
| `front_of_house` | `src/front_of_house.rs` | `src/front_of_house/mod.rs` |
| `hosting` (child module) | `src/front_of_house/hosting.rs` | `src/front_of_house/hosting/mod.rs` |

 **Why Avoid `mod.rs`?**  
- You **end up with too many `mod.rs` files**, making navigation difficult.
- The new system makes the **directory structure more intuitive**.

✔ **Stick to `mod_name.rs` and `mod_name/submodule.rs` instead!**

---

## **5. Re-Exporting Modules with `pub use`**  
Sometimes, you want to **hide internal module structure** but still allow access to a module.

### **Example Without `pub use` (Complicated)**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    front_of_house::hosting::add_to_waitlist(); // Must specify full path
}
```
 **Problem:** External code must **navigate the full module structure**.

### **Better: Using `pub use`**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting; // Re-export

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Cleaner!
}
```
✔ **Now, external code can call `restaurant::hosting::add_to_waitlist()`** directly.  
✔ **Hides unnecessary internal structure** (`front_of_house` is no longer needed externally).

---

## **6. Summary: Organizing Large Projects with Modules**
✔ **Modules allow you to structure large Rust projects efficiently.**  
✔ **Use separate files for large modules** instead of keeping everything in `lib.rs` or `main.rs`.  
✔ **Declare modules using `mod module_name;`**, and Rust will look for the corresponding file.  
✔ **Use subdirectories for nested modules** (`src/module_name/submodule.rs`).  
✔ **Avoid the old `mod.rs` style**—use `mod_name.rs` instead.  
✔ **Use `pub use` to simplify external module access** and hide internal structure.  

---

```markdown
# Chapter 8 - Common Collections in Rust

Rust’s standard library includes several **collections**—data structures that store multiple values. Unlike arrays and tuples, collections **store data on the heap**, allowing them to grow or shrink dynamically.

This chapter focuses on **three essential collections** that you will use frequently in Rust:

- **Vectors (`Vec<T>`)** – Store a list of values of the same type.
- **Strings (`String`)** – A dynamically growing collection of characters.
- **Hash Maps (`HashMap<K, V>`)** – A key-value store.

## **1. Why Use Collections?**
Most basic data types store **only one value**. Collections, however, allow us to:
✔ Store multiple values in a single structure.  
✔ Grow or shrink dynamically at runtime.  
✔ Use powerful methods to modify and retrieve data efficiently.  

Rust ensures **memory safety** in collections using **ownership and borrowing rules**.

---

# **2. Storing Lists of Values with Vectors (`Vec<T>`)**
The **vector type (`Vec<T>`)** allows us to store multiple values of **the same type** in contiguous memory. This is useful when dealing with lists, such as:
- A list of **grades** in a report card.
- A list of **players** in a game.
- A **sequence of numbers** in a mathematical operation.

## **Creating a New Vector**
There are two main ways to create a vector:

### **Method 1: Using `Vec::new()`**
```rust
let v: Vec<i32> = Vec::new();
```
✔ Creates an **empty vector**.  
✔ We **must specify the type** (`i32` in this case) because Rust **cannot infer it** from an empty vector.  

### **Method 2: Using `vec!` Macro (Recommended)**
```rust
let v = vec![1, 2, 3];
```
✔ This creates a vector with **initial values**.  
✔ Rust **infers the type** (`Vec<i32>`) from the values.  
✔ More convenient than `Vec::new()`.

---

## **3. Modifying a Vector**
To add values to a vector **after creation**, we use the `.push()` method.

### **Example: Adding Elements**
```rust
let mut v = Vec::new(); // Mutable vector
v.push(5);
v.push(6);
v.push(7);
```
✔ **Vectors must be mutable** (`mut`) to allow modification.  
✔ The type is inferred as `Vec<i32>` because we added `i32` values.

---

## **4. Accessing Elements in a Vector**
There are **two ways** to read values from a vector:  
✔ **Indexing (`v[index]`)** – Panics if the index is out of bounds.  
✔ **`.get(index)` method** – Returns an `Option<&T>` to avoid panicking.

### **Example: Accessing Elements**
```rust
let v = vec![10, 20, 30, 40, 50];

let third: &i32 = &v[2]; // Using indexing
println!("The third element is {third}");

let third = v.get(2); // Using .get()
match third {
    Some(value) => println!("The third element is {value}"),
    None => println!("There is no third element."),
}
```
✔ **Indexing (`v[2]`)** returns a direct reference but **panics if out of range**.  
✔ **`.get(2)`** returns an `Option<&i32>` that we **must handle safely**. --prefered way

---

## Vector capacity

**with_capacity**

```rust
fn main() {
   let seasons: Vec<&str> = Vec::with_capacity(4);
   println!("Length {}. Capacity: {}", seasons.len(), seasons.capacity());
}
```
this will log - "Length 0. Capacity: 4"
when we added 4 elements and then log Lenght will be for and capacity 4.

## **5. Handling Out-of-Bounds Access**
If we try to **access an invalid index**, Rust reacts differently based on the method used:

### **Example: Invalid Index Access**
```rust
let v = vec![1, 2, 3, 4, 5];

// This will cause a runtime panic!
let does_not_exist = &v[100];

// This will return `None`, preventing a crash
let does_not_exist = v.get(100);
```
- **Using `v[100]` panics the program** if the index is out of range.
- **Using `v.get(100)` returns `None`**, allowing us to handle the error gracefully.

---

## **6. Borrowing and Mutability Issues in Vectors**
Rust **does not allow** modifying a vector while holding an immutable reference.

### **Example: Mutable Borrowing Conflict**
```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0]; // Immutable reference

v.push(6); // ERROR: Mutable borrow while an immutable reference exists

println!("First element is: {first}"); // This cannot happen
``` 
**Why does this fail?**  
- Rust **prevents modifying a vector (`v.push(6)`) while an immutable reference (`first`) exists**.
- **Solution**: Access elements **after** modifying the vector.

### **Example: Correct Order of Operations**
```rust
let mut v = vec![1, 2, 3, 4, 5];

v.push(6); // Modify the vector first

let first = &v[0]; // Immutable reference after modification

println!("First element is: {first}"); // This works now
```

## **7. Iterating Over a Vector**
Instead of accessing elements one by one, we can iterate over them.

### **Example: Immutable Iteration**
```rust
let v = vec![100, 32, 57];

for num in &v {
    println!("{num}");
}
```
✔ The **`&v`** gives an immutable reference to each item.

### **Example: Mutable Iteration**
```rust
let mut v = vec![100, 32, 57];

for num in &mut v {
    *num += 50; // Modify elements in-place
}
```
✔ The **`&mut v`** allows modifying each item.  
✔ We need to use **`*num` (dereferencing)** to change the actual value.

---

## **8. Storing Multiple Types in a Vector**
Vectors **only store values of the same type**.  
To store different types, we use **enums**.

### **Example: Using an Enum**
```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```
✔ **All elements belong to the same type (`SpreadsheetCell`)**.  
✔ Rust ensures type safety **at compile time**.  
✔ If the possible types are unknown at compile time, **use trait objects (covered in Chapter 17).**

---

## **9. Dropping a Vector (Memory Management)**
When a vector **goes out of scope**, Rust **automatically cleans up its memory**.

### **Example: Vector Drops Its Elements**
```rust
{
    let v = vec![1, 2, 3, 4];
    // Do something with v
} // <- v goes out of scope and is freed
```
✔ **All elements inside `v` are also dropped automatically.**  
✔ Rust's **ownership system** prevents memory leaks.

---

## **10. Summary of `Vec<T>`**
✔ **Vectors store multiple values in contiguous memory.**  
✔ **Use `.push()` to add elements** to a mutable vector.  
✔ **Use `v[index]` for direct access (panics if out of bounds).**  
✔ **Use `v.get(index)` for safe access (returns `Option<&T>`).**  
✔ **Cannot modify a vector while holding an immutable reference.**  
✔ **Use `for` loops to iterate over elements.**  
✔ **To store multiple types, use an `enum`.**  
✔ **Vectors are automatically freed when they go out of scope.**  

---

```markdown
# Storing UTF-8 Encoded Text with Strings

## **1. Understanding Strings in Rust**
Rust strings can be confusing at first because:
- **Rust prevents common string-related errors.**
- **Strings are more complex than they appear** in many programming languages.
- **Rust enforces UTF-8 encoding**, which makes handling different languages trickier.

Unlike other languages where strings are simple character arrays, Rust **stores strings as collections of bytes** and provides methods to interpret them as human-readable text.

### **Two Main String Types in Rust**
Rust provides **two primary string types**:
| **Type**   | **Description** |
|------------|----------------|
| `String`   | A **growable, mutable, heap-allocated** string. |
| `&str`     | A **string slice**—an immutable reference to part of a `String` or a string literal. |

✔ `String` is used when we **own and modify** text.  
✔ `&str` is used when we **borrow** a string (e.g., function arguments).  

---

## **2. Creating Strings**
A `String` is essentially **a wrapper around a vector of bytes (`Vec<u8>`)**.

### **Method 1: Creating an Empty String**
```rust
let mut s = String::new(); // Creates an empty, growable String
```
✔ This is useful when we **want to build a string dynamically**.

### **Method 2: Creating a String from a String Literal**
```rust
let s = String::from("Hello, world!");
```
✔ Equivalent to:
```rust
let s = "Hello, world!".to_string();
```
✔ `String::from()` and `.to_string()` **both create a `String` from a `&str`**.  
✔ Both methods **store the string in heap memory**.

### **UTF-8 Encoding in Strings**
Rust strings **support all Unicode characters**, so you can store text in different languages:
```rust
let hello = String::from("こんにちは");  // Japanese
let hello = String::from("Здравствуйте"); // Russian
let hello = String::from("안녕하세요");  // Korean
```
✔ Every `String` in Rust is **UTF-8 encoded by default**.

---

## **3. Updating a String**
Rust allows modifying a `String`, just like a `Vec<T>`, using **`push_str()`**, **`push()`**, or **concatenation**.

### **Method 1: Using `push_str()` to Append Text**
```rust
let mut s = String::from("Hello");
s.push_str(", world!");
println!("{}", s); // Output: "Hello, world!"
```
✔ `push_str()` **appends a string slice (`&str`)** without taking ownership.

### **Method 2: Using `push()` to Append a Single Character**
```rust
let mut s = String::from("Hi");
s.push('!');
println!("{}", s); // Output: "Hi!"
```
✔ `push()` **only adds a single character (`char`)**.

### **Method 3: Concatenation Using the `+` Operator**
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // `s1` is moved and cannot be used afterward
println!("{}", s3); // Output: "Hello, world!"
```
✔ **`s1` is moved**, meaning it cannot be used after the `+` operation.  
✔ **`s2` is borrowed (`&s2`)**, so it is still accessible.

### **Method 4: Concatenating Multiple Strings with `format!` (Recommended)**
Using `+` can be **unreadable** when concatenating multiple strings:
```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let result = s1 + "-" + &s2 + "-" + &s3;
```
A **better approach** is to use the `format!` macro:
```rust
let s = format!("{s1}-{s2}-{s3}");
println!("{}", s); // Output: "tic-tac-toe"
```
✔ `format!` **does not take ownership** of any string.  
✔ It is **more readable and efficient**.

---

## **4. Why Rust Strings Cannot Be Indexed**
In many programming languages, you can access characters in a string using an index (`s[0]`).  
 **Rust does not allow indexing into a `String` like this:**
```rust
let s = String::from("hello");
let h = s[0]; // ERROR: Strings cannot be indexed
```
 **Why?**
1. **Rust stores strings as a sequence of bytes (`Vec<u8>`)**, not individual characters.
2. **Different characters take different amounts of bytes in UTF-8.**
   - Example: The Russian word `"Здравствуйте"` takes **24 bytes**, not 12 characters!
   - Indexing into `"Здравствуйте"[0]` would return **only part of a character**, which is invalid.

---

## **5. Slicing Strings Safely**
Instead of indexing, Rust allows **string slicing**:
```rust
let hello = String::from("Здравствуйте");
let s = &hello[0..4]; // Valid slice
println!("{}", s); // Output: "Зд"
``` **Warning:**  
Slicing must **only cut at valid UTF-8 boundaries**. Otherwise, Rust **panics** at runtime.

---

## **6. Iterating Over Strings**
Since strings cannot be indexed, **iterating over characters or bytes** is the best way to process them.

### **Method 1: Iterating Over Unicode Characters**
```rust
for c in "Зд".chars() {
    println!("{}", c);
}
```
✔ Prints:
```
З
д
```
✔ `.chars()` **returns Unicode characters (`char`)**.

### **Method 2: Iterating Over Raw Bytes**
```rust
for b in "Зд".bytes() {
    println!("{}", b);
}
```
✔ Prints the **raw UTF-8 bytes**:
```
208
151
208
180
```
✔ `.bytes()` **returns each raw byte (`u8`)**.

---

## **7. Understanding UTF-8 Complexity**
Rust strings **store data as bytes**, but characters may be more complex:
| Representation | Meaning |
|---------------|---------|
| **Bytes (`u8`)** | Raw data stored in memory. |
| **Unicode Scalars (`char`)** | Single characters (may use multiple bytes). |
| **Grapheme Clusters** | A **user-perceived** character (e.g., "न" + "्" = "न्"). |

 **Example: Hindi Word "नमस्ते"**
- **Bytes:** `[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]`
- **Characters (`char`):** `['न', 'म', 'स', '्', 'त', 'े']`
- **Grapheme Clusters:** `["न", "म", "स्", "ते"]`

✔ **Rust does not provide built-in grapheme cluster handling**—use external crates if needed.

---

## **8. Summary of Strings in Rust**
✔ **Strings are UTF-8 encoded collections of bytes.**  
✔ **Two types exist:** `String` (owned) and `&str` (borrowed).  
✔ **Use `push_str()` or `push()` to modify strings.**  
✔ **Use `+` or `format!` to concatenate strings.**  i
✔ **Strings cannot be indexed due to UTF-8 complexity.**  
✔ **Use `.chars()` for characters and `.bytes()` for raw bytes.**  
✔ **Be careful when slicing strings—only use valid byte ranges.**  

---

```markdown
# Storing Keys with Associated Values in Hash Maps

## **1. What is a Hash Map?**
A **HashMap<K, V>** is a collection that **maps keys (`K`) to values (`V`)**. It is also known as:
- **Dictionary** (Python)
- **Hash table** (General CS term)
- **Object** (JavaScript)
- **Associative array** (PHP)

### **Why Use a Hash Map?**
Unlike vectors (`Vec<T>`) that use **index-based lookup**, hash maps allow you to:
✔ **Look up values using keys** instead of numerical indices.  
✔ **Quickly retrieve values** without scanning the entire collection.  
✔ **Store key-value pairs** efficiently.

 **Example Use Case**:  
In a game, you could use a hash map to **track team scores**:
```rust
let mut scores = HashMap::new();
scores.insert("Blue", 10);
scores.insert("Yellow", 50);
```
Now, `scores["Blue"]` gives us `10`.

---

## **2. Creating a Hash Map**
Since hash maps are **less commonly used** than vectors and strings, Rust **does not include them in the prelude**.  
You must explicitly **import them** from `std::collections`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```
✔ We **declare a mutable hash map (`mut scores`)**.  
✔ We **insert key-value pairs (`insert()`)**, associating `"Blue"` with `10` and `"Yellow"` with `50`.  

 **Important Notes**:
- All **keys must have the same type** (e.g., `String`).
- All **values must have the same type** (e.g., `i32`).
- Hash maps **store data on the heap**, just like vectors.

---

## **3. Accessing Values in a Hash Map**
To retrieve a value, use the `.get()` method:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);

println!("Score: {}", score);
```
✔ **`.get(&key)` returns `Option<&V>`** because the key may not exist.  
✔ **`.copied()` converts `Option<&i32>` to `Option<i32>`**.  
✔ **`.unwrap_or(0)` returns `0` if the key doesn't exist**.

---

## **4. Iterating Over Key-Value Pairs**
To loop through all key-value pairs in a hash map:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
}
```
✔ **Iterates over all key-value pairs** in an **arbitrary order** (hash maps don’t maintain insertion order).  
✔ Prints:
```
Yellow: 50
Blue: 10
```

---

## **5. Hash Maps and Ownership**
### **How Rust Handles Ownership in Hash Maps**
When inserting **values** into a hash map:
- If the type implements **Copy** (e.g., `i32`), Rust **copies** the value.
- If the type is **owned** (e.g., `String`), Rust **moves** the value into the hash map.

### **Example: Moving Ownership**
```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);

// field_name and field_value are now invalid!
// println!("{}", field_name); // ERROR: Ownership moved!
```
✔ **Ownership of `field_name` and `field_value` transfers to the hash map**.  
✔ We **cannot use `field_name` afterward** because it was moved.

### **Storing References Instead of Moving Data**
To **avoid ownership transfer**, store **references** instead of owned values:
```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(&field_name, &field_value); // Store references

println!("{}", field_name); // Still valid
``` 
**Warning**: The **referenced data must outlive the hash map**!

---

## **6. Updating a Hash Map**
Each key can only store **one value** at a time, but Rust offers **three ways to update values**.

### **Method 1: Overwriting an Existing Value**
If we insert a **new value for an existing key**, Rust **overwrites the old value**:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores); // Output: {"Blue": 25}
```
✔ `"Blue"` initially had `10`, but it was **replaced** with `25`.

---

### **Method 2: Only Inserting a Value If the Key Doesn’t Exist (`entry().or_insert()`)**
To **add a key-value pair only if the key is missing**, use `.entry().or_insert()`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores); // Output: {"Blue": 10, "Yellow": 50}
```
✔ `"Blue"` **was not updated** because it already had a value.  
✔ `"Yellow"` **was inserted** because it was missing.

---

### **Method 3: Updating a Value Based on the Old Value**
To **modify a value based on its previous value**, use `.entry().or_insert()` and **mutate it**:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";
let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map); // Output: {"world": 2, "hello": 1, "wonderful": 1}
```
✔ `.entry(word).or_insert(0)` initializes the count to `0` if the word is missing.  
✔ `*count += 1` **increments the existing count**.  

 **This is useful for counting word occurrences in text**.

---

## **7. How Hash Maps Work Internally**
Rust’s `HashMap<K, V>` **uses a hashing function** to store and retrieve values efficiently.

### **Default Hashing Algorithm: `SipHash`**
- **SipHash** is used for security **against DoS attacks**.
- **Other languages use faster but less secure hashing algorithms**.

### **Using a Custom Hashing Algorithm**
If you need **better performance**, you can use a different hashing function:

```rust
use std::collections::HashMap;
use std::hash::BuildHasherDefault;
use std::collections::hash_map::DefaultHasher;

let mut scores: HashMap<String, i32, BuildHasherDefault<DefaultHasher>> = HashMap::default();
```
 **This is an advanced optimization**—not needed for most applications.

---

## **8. Summary of Hash Maps in Rust**
✔ **Hash maps store key-value pairs and allow fast lookups**.  
✔ **Keys and values must be of the same type**.  
✔ **Rust takes ownership of inserted `String` keys/values unless referenced**.  
✔ **Use `.entry().or_insert()` to insert only if missing**.  
✔ **Use `.entry().or_insert_mut()` to update values based on existing ones**.  
✔ **Hash maps use `SipHash` for security but can be customized**.  

---

```markdown
# **Chapter 8 Summary - Common Collections in Rust**

Rust’s standard library provides **three essential collections**:
1. **Vectors (`Vec<T>`)** – Store a list of values in **contiguous memory**.
2. **Strings (`String`)** – A **growable, heap-allocated** UTF-8 text storage.
3. **Hash Maps (`HashMap<K, V>`)** – Store key-value pairs with **fast lookups**.

## **1. Vectors (`Vec<T>`)**
- **Used for**: Storing multiple values of the **same type**.
- **Key Methods**:
  - `Vec::new()` → Create an empty vector.
  - `vec![...]` → Initialize a vector with values.
  - `.push(value)` → Add an element.
  - `.get(index)` → Safely retrieve an element (`Option<T>`).
  - `.iter()` → Loop through elements.
- **Important Concepts**:
  - Cannot modify a vector while holding an **immutable reference**.
  - Use **enums** to store multiple types in a vector.

---

## **2. Strings (`String` and `&str`)**
- **Used for**: Storing and manipulating UTF-8 encoded text.
- **Key Methods**:
  - `String::new()` → Create an empty string.
  - `"text".to_string()` or `String::from("text")` → Convert a string literal to `String`.
  - `.push_str("text")` → Append a string slice.
  - `.push('c')` → Append a single character.
  - **Concatenation**:
    - `"str1".to_string() + "str2"` (Consumes `str1`).
    - `format!("{s1}-{s2}")` (Preferred, more readable).
  - `.chars()` → Iterate over characters.
  - `.bytes()` → Iterate over raw UTF-8 bytes.
- **Important Concepts**:
  - **Strings cannot be indexed (`s[0]` is invalid)** because Rust stores them as bytes.
  - **Slicing must be valid UTF-8 boundaries** to avoid runtime panics.

---

## **3. Hash Maps (`HashMap<K, V>`)**
- **Used for**: Storing key-value pairs for **fast lookups**.
- **Key Methods**:
  - `HashMap::new()` → Create an empty hash map.
  - `.insert(key, value)` → Add a key-value pair.
  - `.get(&key)` → Retrieve a value (`Option<V>`). 
  - `.entry(key).or_insert(value)` → Insert only if the key is missing.
  - **Iterating**: `for (key, value) in &hash_map { ... }`
- **Important Concepts**:
  - Hash maps **take ownership** of `String` keys and values unless referenced.
  - Uses **`SipHash` (secure but slower hashing function)** by default.
  - Supports **updating values** based on previous values (useful for word counting).

```markdown
# Chapter 9 - Error Handling in Rust

Errors happen in every program, so Rust provides tools to help you handle them **safely and clearly**.

Rust **forces you to handle errors**, which helps prevent bugs from sneaking into production code.

---

## Two Kinds of Errors in Rust

| **Error Type** | **What It Means** | **Rust Tool** |
|----------------|-------------------|----------------|
| **Unrecoverable** | A **serious bug** in your program (e.g., out-of-bounds access). | `panic!` |
| **Recoverable** | Something went wrong, but we can **try again** or report it (e.g., file not found). | `Result<T, E>` |

> This chapter focuses first on **unrecoverable errors** using `panic!`.

---

## Unrecoverable Errors with `panic!`

If something goes very wrong, Rust can **immediately stop execution** using the `panic!` macro.

```rust
fn main() {
    panic!("crash and burn");
}
```

✔ This code will **crash the program** and print:
```
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
```

### What happens when you panic?
- Rust **prints an error message**.
- It **starts unwinding the stack** (cleaning up).
- Then it **quits the program**.

You can also **see a backtrace** (a list of function calls leading to the error) by running:
```sh
RUST_BACKTRACE=1 cargo run
```

---

## Unwinding vs Aborting

When a panic occurs, you have two options:

| **Mode** | **What Happens** | **Use Case** |
|----------|------------------|--------------|
| **Unwind (default)** | Rust **cleans up** memory and shuts down | Good for debugging |
| **Abort** | Rust **immediately exits**, no cleanup | Good for small binaries or release builds |

### Switching to "abort" mode (in `Cargo.toml`)
```toml
[profile.release]
panic = 'abort'
```

---

## Example: Code That Panics Automatically

```rust
fn main() {
    let v = vec![1, 2, 3];
    let value = v[99]; // This will panic!
}
```

✔ You’re accessing **index 99**, but the vector only has 3 items (indexes 0, 1, 2).

Rust doesn’t silently allow this like C or C++.  
Instead, it **panics to protect you** from unsafe memory access (e.g., buffer overreads).

### What you’ll see:
```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
```

---

## 🕵️‍♂️ Reading a Backtrace

To debug, use:
```sh
RUST_BACKTRACE=1 cargo run
```

Sample output (simplified):
```
thread 'main' panicked at 'index out of bounds...', src/main.rs:4:5
stack backtrace:
0: rust_begin_unwind
1: core::panicking::panic_bounds_check
2: panic::main (src/main.rs:4:5)
```

✔ Look for the **first line with your code** (e.g., `src/main.rs:4:5`).  
✔ That’s where the problem started.

---

## Key Takeaways (So Far)

- Use `panic!` to **crash the program** if continuing would be unsafe.
- Rust panics **automatically** for critical issues like:
  - Indexing out of bounds
  - Integer overflow in debug mode
- You can enable detailed backtraces with `RUST_BACKTRACE=1`.
- Rust's design helps you catch **bugs early**, not ignore them.
- Wheb `panic!` Rust unwinds(clears) stack data, this is default behaviour or you can set `panic!` to abort.

---

## Summary: When to Panic

| **Use `panic!` When...** | |
|--------------------------|----|
| You hit an unrecoverable bug | ✔️ |
| The program is in a bad state | ✔️ |
| You’re writing a prototype or experimenting | ✔️ |
| You want to enforce a **debug assertion** | ✔️ |

## Recoverable Errors with `Result<T, E>`

Not all errors are fatal! Sometimes an operation might fail, but your program can continue.

Rust helps you handle such **recoverable errors** using the `Result` type.

---

## What Is `Result<T, E>`?

```rust
enum Result<T, E> {
    Ok(T),    // Successful result
    Err(E),  // Error occurred
}
```

- `T` is the type you get **if everything works**.
- `E` is the type of **error info** if something goes wrong.

> Think of `Result` as: "Do I get a value (`Ok`) or an error (`Err`)?"

---

## Example: Opening a File

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

- If the file exists, `File::open` returns `Ok(file)`.
- If it doesn’t (or there's another issue), it returns `Err(error)`.

---

## Handling `Result` with `match`

```rust
use std::fs::File;

fn main() {
    let greeting_file = match File::open("hello.txt") {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error);
        }
    };
}
```

- This handles both `Ok` and `Err` cases.
- If opening fails, the program crashes (not ideal in real apps).

---

## Match on Specific Error Kinds

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = match File::open("hello.txt") {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

- This checks **what kind of error** occurred.
- If the file is missing, it tries to **create it**.
- Otherwise, it panics.

---

#  Difference Between `Option<T>` and `Result<T, E>` in Rust

- **Purpose:**
  - `Option<T>` → "Is there a value or not?"
  - `Result<T, E>` → "Did the operation succeed or fail?"

- **Variants:**
  - `Option<T>` → `Some(T)` or `None`
  - `Result<T, E>` → `Ok(T)` or `Err(E)`

- **Use Case:**
  - `Option<T>` → Used for missing or optional values
  - `Result<T, E>` → Used for success or failure of an operation

- **Error Info:**
  - `Option<T>` →  No error information
  - `Result<T, E>` → Contains an error value (`E`)

- **Common With:**
  - `Option<T>` → Lookups (e.g., finding something), optional data
  - `Result<T, E>` → File input/output, parsing, network requests, and other operations that can fail

---

## Cleaner Handling with Closures
A closure is an anonymous function you can store in a variable or pass as a parameter. It can capture variables from its surrounding environment, which makes it more flexible than regular functions.

It’s like a function… but it can "remember" stuff from where it was created.
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

- Uses `unwrap_or_else`, which is **shorter than `match`**.

---

## Shortcuts: `unwrap` and `expect`

```rust
let file = File::open("hello.txt").unwrap();
```
- Crashes if file can’t be opened.

```rust
let file = File::open("hello.txt")
    .expect("hello.txt should be included in this project");
```
- Same behavior, but gives a **custom panic message**.

> Prefer `expect()` over `unwrap()` in production code for **better error context**.

---

## Propagating Errors

You don't have to handle errors immediately. Instead, **return them to the caller**:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

### What Does `?` Do?
- If `Ok`, continues and extracts the value.
- If `Err`, returns the error **from the current function**.

> Saves you from writing long match expressions.

---

## Even Shorter: fs::read_to_string

```rust
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

✔ Opens file, reads string, and returns it.
✔ Automatically handles errors via `Result`.

---

## Restrictions of `?`

You can only use `?` in functions that return:
- `Result<T, E>`
- `Option<T>`
- Or a type that implements `FromResidual`

```rust
fn main() {
    let f = File::open("hello.txt")?; // Error!
}
```

### ✔️ Fix: Change return type of `main`

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

✔ Now `main` supports returning `Result`, so `?` is allowed.

---

## `?` with Option<T>

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

- Stops and returns `None` if any part returns `None`.
- Works just like `?` with `Result`, but for `Option`.

> You can’t use `?` to convert between `Result` and `Option`. Use `.ok()` or `.ok_or()` for that.

---

## Key Takeaways

| Concept | Summary |
|--------|---------|
| `Result<T, E>` | Used for recoverable errors |
| `match` | Basic way to handle success or failure |
| `unwrap` / `expect` | Shortcuts that panic on error |
| `?` operator | Propagates errors easily without match |
| `Option<T>` | Works similarly with `?` for optional values |
| `main` function | Can return `Result` to support `?` |

---

## **To panic! or Not to panic!**

### When should you use `panic!` vs. `Result`?

Rust gives you two main ways to handle errors:

- `panic!` → for **unrecoverable** errors
- `Result<T, E>` → for **recoverable** errors

The general rule:
> Use `Result` when the caller might reasonably recover from the error.  
> Use `panic!` when continuing is unsafe or incorrect.

---

### `Result` gives control to the *caller*

If you return a `Result`, the caller of your function can decide:

- whether to try again,
- show a message to the user,
- or just give up and panic themselves.

Using `Result` is the more *flexible* and *friendly* choice.

---

### When is `panic!` acceptable?

There are a few valid cases for using `panic!`:

#### 1. **Examples, Prototypes, and Tests**
- `unwrap` and `expect` are fine when writing quick-and-dirty prototypes or examples.
- In tests, you *want* the program to panic when something goes wrong so the test fails.

#### 2. **You KNOW the error can’t happen**
Sometimes you *know* a `Result` will be `Ok`, but the compiler doesn’t.
In those cases, you can use `expect` and leave a clear reason:

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1"
    .parse()
    .expect("Hardcoded IP address should be valid");
```

This works because you’re 100% sure `"127.0.0.1"` is valid. The compiler can’t know that, but *you* do. Just make sure your assumption stays valid.

---

### 🧭 When to `panic!`: General Guidelines

Use `panic!` when:
- You enter a **bad state** (an assumption or invariant has been broken).
- You can't proceed safely without that assumption being true.
- You can't model the problem using types alone.

Example:
- Indexing outside a vector → could expose private memory → dangerous.
- Panic makes sure you don’t continue in an insecure state.

You should also:
- Document in your function’s API when it panics.
- Prefer encoding requirements in the **type system** when possible (see below).

---

### Encode Invariants with Types

Rust's **type system** can help you avoid bad states entirely!

For example, if your app only works with numbers from 1 to 100, instead of using a `u32`, create a `Guess` type that *only* allows valid values:

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess must be between 1 and 100, got {}.", value);
        }
        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

Now, other parts of your program never have to check whether a `Guess` is valid — because invalid values *can’t even be created*.

---

## Final Summary: Chapter 9 – Error Handling

Rust’s error-handling system is designed to encourage safe, robust, and clear code.

### Use `panic!` for unrecoverable errors:
- Things that should never happen
- Broken assumptions, bugs, or bad internal state
- Tests, examples, and early prototypes

### Use `Result<T, E>` for recoverable errors:
- File not found? Let the caller retry or handle it
- HTTP request failed? Let the caller decide what to do
- Parsing user input? Return `Err` if the input is malformed

Rust provides:
- `match`, `unwrap`, `expect`, `unwrap_or_else` for handling errors
- `?` operator to **propagate errors easily** up the call stack
- Integration with `main` so it can return `Result`, not just `()`
- Trait-based conversion via `From` for flexible error types

You can make your code **even safer** by:
- Using the type system to prevent invalid states (e.g., custom types like `Guess`)
- Documenting clearly when functions can panic
- Returning informative errors instead of panicking, unless it’s truly unrecoverable

# Chapter 10: Generic Types, Traits, and Lifetimes in Rust

## Introduction to Generics

Generics are a powerful feature in Rust that enable you to write flexible and reusable code without sacrificing performance. At their core, **generics allow you to write code that works for many different types**, abstracting over specific data types while still maintaining type safety.

Generics let you write code that works with many different types, instead of repeating the same logic for each type.

Think of generics as placeholders for future types. Instead of saying, “this function only will work with `i32`", you say, “this function will work with any type — you choose which!”

In many programming languages, similar tools exist to avoid duplication and boilerplate, but Rust's generics are known for their **zero-cost abstraction**—meaning there's no runtime performance penalty. Rust achieves this by compiling generic code into optimized versions specific to each concrete type used.

Generics are used in many parts of Rust, such as:
- `Option<T>` (Chapter 6)
- `Vec<T>` and `HashMap<K, V>` (Chapter 8)
- `Result<T, E>` (Chapter 9)

This chapter focuses on how to:
- Define your own generic functions, structs, enums, and methods
- Use **traits** to define shared behavior
- Use **lifetimes** to ensure reference validity

---

## Why Use Generics?

Without generics, you'd need to write the same logic multiple times for each type you want to support. For example:
```rust
fn largest_i32(list: &[i32]) -> &i32 { ... }
fn largest_char(list: &[char]) -> &char { ... }
```

With generics, we can unify this logic:
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T { ... }
```

Here:
- `T` is a type parameter.
- `PartialOrd` is a **trait bound** that restricts `T` to types that can be compared using `>`.

Generics eliminate duplication, improve readability, and make your code more flexible and abstract.

---

## **Generics in Function Definitions**

Generics let functions **work with different types** instead of being locked to just one type (like `i32`, `f64`, etc.).

### Example:
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

### Explanation:
- `T` is a **type parameter** — a placeholder for any type.
- `T: PartialOrd` is a **trait bound**, meaning `T` must be *comparable* (must support `<`, `>`).
- `list: &[T]` means the function accepts a **slice** (a reference to an array) of elements of type `T`.
- It returns a reference to the **largest** item in the list.

This function can now find the largest value in a list of `i32`, `f64`, or any other comparable type — super flexible!

---

## **Generics in Struct Definitions**

Structs can also use generics to store different kinds of data **without repeating code**.

### Basic Struct:
```rust
struct Point<T> {
    x: T,
    y: T,
}
```

Here, both `x` and `y` are of the **same type `T`** (could be `i32`, `f64`, `String`, etc.).

### Mixed-Type Struct:
```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

This version allows `x` and `y` to have **different types**. You could have:
```rust
let point = Point { x: 5, y: 4.0 }; // T = i32, U = f64
```

This makes your struct flexible and reusable.

---

## **Generics in Enum Definitions**

Just like structs, enums can also use generics. Rust’s **`Option`** and **`Result`** types are perfect examples.

### `Option<T>`:
```rust
enum Option<T> {
    Some(T),
    None,
}
```

- `Some(T)` — There is a data.
- `None` — There no a data.

This represents an optional value: either **Some value of type `T`**, or **None** (no value).

### `Result<T, E>`:
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

This represents a result of a computation:
- `Ok(T)` — everything went fine.
- `Err(E)` — something went wrong.

These enums can be reused for any type, thanks to generics.

---

## **Generics in Method Definitions**

Generic structs can also have generic **methods**.

### General Method (for all `T`):
```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

This defines a method `x()` that works on **any `Point<T>`**, regardless of what type `T` is.

---

### Turbofish operator ::

```rust
fn main() {
    println!("{}", identity(5)); // Here type of identity is i32 because we used 5
    println!("{}", identity::<i8>(5)); // With turbofish operator ::<8> we specify actual type and now Rust will treat 5 as i8
}

//we can use turbofish in other methods too

fn identity<T>(value: T) -> T {
    value
}
```

### Specialized Method (for a specific type only):
```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

This method is **only available** if `Point` is made with `f32` coordinates. Rust allows this kind of specialization!

---

## **Mixing Generics in Methods**

You can define methods that take **different generic types** than the struct itself.

### Example:
```rust
impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```

This method:
- Takes another `Point<X2, Y2>`.
- Returns a new `Point<X1, Y2>`, combining `self.x` and `other.y`.

This gives you a ton of flexibility to **combine different data types in creative ways**.

---

## Summary of Concepts:

| Concept                          | Description |
|----------------------------------|-------------|
| `T`, `U`, `E`                    | Generic type parameters |
| `<T>`                            | Declares a function, struct, or enum is generic |
| `T: Trait`                       | Constrains a generic type to only accept types with certain behaviors |
| `impl<T>`                        | Implements methods for all values of type `T` |
| `impl SpecificType`              | Specializes behavior for one specific type |
| `Option<T>`, `Result<T, E>`      | Real-life usage of generics in the standard library |

---

## Performance of Generics

Rust’s generics are **zero-cost** because of **monomorphization**.

Monomorphization is the process where the compiler:
1. Takes your generic code.
2. Generates specific versions of it for each concrete type you use.

### Example:
```rust
let integer = Some(5);
let float = Some(5.0);
```

Behind the scenes, Rust creates:
```rust
enum Option_i32 { Some(i32), None }
enum Option_f64 { Some(f64), None }
```

This means:
- No runtime type checks.
- No performance penalty.
- Code is as efficient as if you hand-wrote it for every type.

---

## Summary

- **Generics** let you abstract over types.
- You can use generics in **functions**, **structs**, **enums**, and **methods**.
- Thanks to **monomorphization**, generics are efficient with no runtime cost.

# Understanding Traits in Rust

Traits in Rust are one of the core features that make the language powerful and safe. If you're familiar with **interfaces** in other languages like Java, TypeScript, or Go, then you're already on the right track. But Rust's **traits** go even further, giving you control over shared behavior with **compile-time guarantees**.

---

## What is a Trait?

A **trait** defines **shared behavior**. It tells the compiler: “Any type that implements this trait must implement these methods.”

Think of a trait like a *contract* for behavior.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Here, any type that implements `Summary` must define a `summarize` method that returns a `String`. Simple as that.

---

## Why Use Traits?

- **Abstract shared behavior**
- **Write generic, reusable code**
- **Avoid duplication**
- **Enforce behavior at compile time**

For example, imagine two different data types: a `Tweet` and a `NewsArticle`. Both are different, but both can be summarized. You could write separate functions for each, but it’s better to define a shared trait that both types implement.

---

## Implementing a Trait

```rust
pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

Now you can use `tweet.summarize()` and Rust knows exactly what to call.

You can implement a trait on any type you own, as long as **either the trait or the type is defined in your crate** (the *orphan rule*).

---

## Default Implementations

You don’t always need every type to provide its own method implementation. Traits can include **default implementations**:

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

Now any type that implements `Summary` but doesn’t provide `summarize` will use this default.

You can even require one method but provide another that uses it:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

Implement only `summarize_author`, and you automatically get `summarize`!

---

## Traits as Parameters

You can require that a function accepts any type that implements a trait:

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

This lets you write flexible functions:
- Accept *any* type that implements `Summary`
- Enforce behavior at compile time

### Trait Bound Syntax
Same thing, written another way:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Useful when:
- You need multiple generic parameters
- You want more readable error messages

### Multiple Trait Bounds
Need a type to do *more* than one thing?

```rust
pub fn notify<T: Summary + Display>(item: &T) {
    println!("{}", item);
}
```

This says: only accept types that implement **both** `Summary` and `Display`.

---

## `where` Clauses for Cleaner Signatures

Trait bounds can get messy. Rust lets you clean them up:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

More readable, especially with multiple traits.

---

## Returning Types That Implement Traits

You can return `impl Trait` to hide the concrete type:

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

This is great for:
- Hiding complex types
- Returning closures or iterators

Note: You can’t return *different* types with the same `impl Trait`—you must return **one specific type**.

---

## Conditional Trait Implementations

Want a method only if a type meets certain conditions? Use trait bounds in the `impl` block:

```rust
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

Only types that implement `Display` and `PartialOrd` will have `cmp_display()`.

---

## Blanket Implementations

The Rust standard library uses **blanket implementations** to apply traits broadly:

```rust
impl<T: Display> ToString for T {
    // automatically implemented!
}
```

This means **any** type that implements `Display` also automatically implements `ToString`. That’s why you can do:

```rust
let s = 3.to_string();
```

---

## Traits – Advanced Concepts

### `impl` vs `dyn` – Static vs Dynamic Dispatch

Rust allows you to use traits in two ways: with **static dispatch** using `impl Trait` or generics, and with **dynamic dispatch** using `dyn Trait`.

---

### `impl Trait` (Static Dispatch)

When you use `impl Trait` (or generics like `T: Trait`), Rust generates concrete versions of your code at **compile time**. This is called **monomorphization**.

```rust
fn notify(item: &impl Summary) { ... }
// Equivalent:
fn notify<T: Summary>(item: &T) { ... }
```

✅ Fast: No runtime overhead

✅ Type-safe at compile time

✅ Used when the exact type is known

dyn Trait (Dynamic Dispatch)
If you don’t know the type until runtime, use a trait object:

rust
Copy
Edit
fn notify(item: &dyn Summary) {
    println!("Breaking news: {}", item.summarize());
}
⚠️ Slight runtime overhead

✅ Supports dynamic polymorphism

✅ Enables heterogeneous collections like Vec<Box<dyn Trait>>

This uses a vtable (a table of method pointers) for method dispatch at runtime.

Summary: impl vs dyn

Feature	impl Trait / Generics	dyn Trait
Dispatch	Static (compile-time)	Dynamic (runtime)
Performance	High (zero-cost)	Slightly slower
Use Case	Known types, performance	Dynamic behavior, plugins
Syntax	impl Trait, T: Trait	Box<dyn Trait>, &dyn Trait
Object Safe Required	❌	✅
Static vs Dynamic Dispatch
Static dispatch: Used with generics. Resolved at compile time. Faster.

Dynamic dispatch: Used with dyn Trait. Resolved at runtime using vtables.

Use static dispatch by default. Use dynamic dispatch if:

You need runtime polymorphism

You're building plugin systems or storing mixed trait objects



---

## Summary

| Concept | What it Means |
|--------|----------------|
| `trait` | Defines shared behavior |
| `impl Trait` | Accepts any type that implements a trait |
| `T: Trait` | Trait bound for generic types |
| Default impls | Optional built-in behavior |
| Trait + Trait | Require multiple behaviors |
| `where` | Clean syntax for many bounds |
| `impl<T: Trait>` | Constrain methods by behavior |
| Blanket impl | One trait implementation applies to many types |

Traits let you write code that’s:
- **Flexible**
- **Reusable**
- **Safe at compile time**
- **Fast at runtime**

They are a core pillar of how Rust enables powerful, abstract programming without the usual runtime cost of polymorphism.

---

# Rust Lifetimes – A Beginner-Friendly Guide

In Rust, **lifetimes** are a way to tell the compiler how long references are valid. They're a part of Rust's powerful borrow checker, which ensures memory safety without needing a garbage collector. Lifetimes are a kind of **generic**—but instead of abstracting over types or behavior, they abstract over **the scope in which a reference is valid**.

---

## Why Lifetimes Exist

Imagine this: you create a reference to a variable, but that variable goes out of scope. The reference is now **dangling**, pointing to nothing valid. Rust won’t let this happen. Lifetimes help the compiler catch such issues **at compile time**, before your program even runs.

**Goal of lifetimes:** Prevent dangling references by tracking how long references live and ensuring they're always valid.

---

## 🔍 What is a Lifetime?

A **lifetime** (') is the span during which a reference is valid. For example:

```rust
fn main() {
    let r;                  // 'a starts
    {
        let x = 5;          // 'b starts
        r = &x;             // r borrows x (invalid!)
    }                       // 'b ends (x is dropped)
    println!("r: {}", r);    // r is dangling - compiler error
}
```

The problem? `x` goes out of scope before `r` is used. The compiler prevents this because `r` would reference freed memory.

---

## Valid Example

```rust
fn main() {
    let x = 5;
    let r = &x; // both r and x exist within the same scope
    println!("r: {}", r); // This works!
}
```

---

## Lifetime Syntax

Just like types can be generic (`T`, `U`, etc.), lifetimes can be too. Rust uses names like `'a`, `'b` to label them.

Examples:
```rust
&'a i32       // reference to i32 valid for lifetime 'a
&'a mut i32   // mutable reference with lifetime 'a
```

---

## Lifetimes in Function Signatures

Let’s write a function to return the longer of two string slices:

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

 This will **not compile** because Rust doesn’t know how long the returned reference is valid.

### Fixing it with Lifetimes

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### What it means:
- There's some lifetime `'a`
- Both `x` and `y` must be valid for at least `'a`
- The returned reference is also valid for `'a`

Rust will ensure the returned reference does not outlive either input.

---

** lifetine to the returned reference (-> 'a something) will be the same as smallest lifetime of a arguments.
````rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
````

So if x has a smaller lifetime than y, then the lifetime of returned value will be x and vice versa.

---

## Real Example

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```
This works fine because both references live long enough.

### Invalid Case
```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    } // string2 is dropped here

    println!("The longest string is {}", result); // compiler error
}
```

Rust will complain because `string2` is dropped while `result` might reference it.

---

## Summary

- **Lifetimes** ensure references are always valid.
- Rust infers most lifetimes, but **you must specify them** when ambiguity arises.
- Lifetime annotations look like `'a`, `'b`, etc.
- They go in function signatures and describe how the lifetimes of inputs relate to outputs.
-**You are not changing how long data lives—you’re telling Rust how long references *must* be valid**.

---

## Why This Matters

In many languages, problems with dangling references show up **at runtime** (e.g., segmentation faults). Rust moves those problems to **compile time**, helping you write more reliable and safer code.

Once you get used to it, lifetimes become a powerful way to express ownership and borrowing clearly in your functions and structs.

---

# Lifetimes in Rust – Deeper Understanding

## Thinking in Terms of Lifetimes

Lifetimes describe how long references are valid. When writing functions that return references, you must explicitly tell Rust how the returned reference relates to the inputs.

### Example: One Parameter Needs a Lifetime
```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
Here, the returned reference has the same lifetime as `x`. `y`'s lifetime doesn’t matter because it’s not part of the return.

### Invalid Lifetime Example
```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
} // ⚠ Error: `result` goes out of scope here
```
You're returning a reference to a local variable—this would cause a dangling reference. Rust won’t allow that.

**Fix**: Return an owned type (`String`) instead of a reference.

---


---

## Lifetime Annotations in Structs

Structs can also hold references, but you must annotate them:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt { part: first_sentence };
}
```
- `'a` ensures that `ImportantExcerpt` can't outlive the string it borrows from.

---

## Lifetime Elision Rules
Rust has rules that help you skip explicit lifetimes when they’re obvious.

### Elision Rules:
1. Each reference gets its own lifetime.
2. If there is only one input lifetime, it's assigned to the output.
3. If there are multiple input lifetimes and one of them is `&self`, it’s assigned to the output.

### Example That Works Because of Elision:
```rust
fn first_word(s: &str) -> &str { // Lifetimes are elided
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```
---

## Lifetime Elision in Practice
### Example That Needs Annotations:
```rust
fn longest(x: &str, y: &str) -> &str // Doesn’t compile
```
Add lifetimes:
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str
```
This means the return reference is valid for the *shorter* of the two input lifetimes.

---

##  Lifetime Annotations in Methods

You add lifetimes just like type parameters:

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }

    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention: {announcement}");
        self.part
    }
}
```
The third elision rule kicks in here—`self` is assigned to the return value’s lifetime.

---

## The `'static` Lifetime
- `'static` means the reference is valid for the entire duration of the program.
```rust
let s: &'static str = "I have a static lifetime.";
```
Used for string literals and global data. Be cautious using it just to silence errors.

---

## Combining Generics, Trait Bounds, and Lifetimes

You can use generics and lifetimes together:

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement: {ann}");
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
- `'a` connects the input and output references
- `T: Display` ensures `ann` can be printed with `{}`

---

## Summary
- **Lifetimes** ensure references remain valid.
- Use **lifetime annotations** to describe how references relate.
- Rust’s **borrow checker** ensures safety at compile time.
- Rust can **elide** (omit) some lifetimes using predictable rules.
- You can mix **lifetimes, generics, and traits** to write powerful, reusable, and safe code.

# Chapter 11 - Writing Automated Tests in Rust

## Why Write Tests?

Even though Rust has a powerful type system and a compiler that prevents many common programming errors, **bugs can still occur**. Tests are essential to ensure that your code works as expected and continues to work as you make changes.

The goal of a test is to verify that your code:
1. Produces the correct output for a given input.
2. Handles edge cases and errors properly.
3. Continues to behave correctly as your codebase evolves.

Rust provides built-in tools to write, organize, and run tests efficiently.

---

## Anatomy of a Test Function

A basic test in Rust is a regular function marked with the `#[test]` attribute:
```rust
#[test]
fn it_works() {
    let result = 2 + 2;
    assert_eq!(result, 4);
}
```
When you run `cargo test`, Rust compiles your code and runs all functions marked with `#[test]`. Each test is executed in a separate thread.

### Assertions in Tests
- `assert!(condition)` → Fails if the condition is `false`
- `assert_eq!(left, right)` → Fails if the values are not equal
- `assert_ne!(left, right)` → Fails if the values are equal

---

## Writing Your First Tests

Create a new library project:
```sh
cargo new adder --lib
cd adder
```
Rust automatically creates a `tests` module in `src/lib.rs`:
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```
Run the test:
```sh
cargo test
```

You should see output showing the test passed. If you write another test and cause it to panic, it will fail:
```rust
#[test]
fn fails() {
    panic!("This test fails!");
}
```

---

## Using `assert!`, `assert_eq!`, and `assert_ne!`

### Example:
```rust
fn add_two(x: i32) -> i32 {
    x + 2
}

#[test]
fn it_adds_two() {
    assert_eq!(add_two(2), 4);
}
```
If the function returns 5 instead of 4, the test will fail and show a detailed message:
```sh
assertion failed: `(left == right)`
  left: `4`,
 right: `5`
```

---

## Custom Failure Messages

```rust
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```
This helps make failure messages clearer when debugging.

---

## Testing for Panics with `#[should_panic]`

Use this attribute when you expect code to `panic!` under certain conditions:
```rust
#[test]
#[should_panic(expected = "less than or equal to 100")]
fn greater_than_100() {
    Guess::new(200);
}
```
This test passes only if a panic occurs and the panic message contains the expected text.

### Example Struct:
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }
        Guess { value }
    }
}
```

---

## Using `Result<T, E>` in Tests

Tests can also return `Result<(), E>` for more flexibility:
```rust
#[test]
fn it_works() -> Result<(), String> {
    if 2 + 2 == 4 {
        Ok(())
    } else {
        Err(String::from("2 + 2 != 4"))
    }
}
```
You can now use `?` in your test functions, and handle errors without panics.

---

## Summary

Rust's testing framework is simple, expressive, and integrated with the language and build system. You can:
- Use assertions to validate logic.
- Handle panic scenarios.
- Return `Result` types.
- See clear output for failures.

# Chapter 12 - Writing Automated Tests in Rust

## Why Write Tests?

Even though Rust has a powerful type system and a compiler that prevents many common programming errors, **bugs can still occur**. Tests are essential to ensure that your code works as expected and continues to work as you make changes.

The goal of a test is to verify that your code:
1. Produces the correct output for a given input.
2. Handles edge cases and errors properly.
3. Continues to behave correctly as your codebase evolves.

Rust provides built-in tools to write, organize, and run tests efficiently.

---

## Anatomy of a Test Function

A basic test in Rust is a regular function marked with the `#[test]` attribute:
```rust
#[test]
fn it_works() {
    let result = 2 + 2;
    assert_eq!(result, 4);
}
```
When you run `cargo test`, Rust compiles your code and runs all functions marked with `#[test]`. Each test is executed in a separate thread.

### Assertions in Tests
- `assert!(condition)` → Fails if the condition is `false`
- `assert_eq!(left, right)` → Fails if the values are not equal
- `assert_ne!(left, right)` → Fails if the values are equal

---

## Writing Your First Tests

Create a new library project:
```sh
cargo new adder --lib
cd adder
```
Rust automatically creates a `tests` module in `src/lib.rs`:
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```
Run the test:
```sh
cargo test
```

You should see output showing the test passed. If you write another test and cause it to panic, it will fail:
```rust
#[test]
fn fails() {
    panic!("This test fails!");
}
```

---

## Using `assert!`, `assert_eq!`, and `assert_ne!`

### Example:
```rust
fn add_two(x: i32) -> i32 {
    x + 2
}

#[test]
fn it_adds_two() {
    assert_eq!(add_two(2), 4);
}
```
If the function returns 5 instead of 4, the test will fail and show a detailed message:
```sh
assertion failed: `(left == right)`
  left: `4`,
 right: `5`
```

---

## Custom Failure Messages

```rust
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```
This helps make failure messages clearer when debugging.

---

## Testing for Panics with `#[should_panic]`

Use this attribute when you expect code to `panic!` under certain conditions:
```rust
#[test]
#[should_panic(expected = "less than or equal to 100")]
fn greater_than_100() {
    Guess::new(200);
}
```
This test passes only if a panic occurs and the panic message contains the expected text.

### Example Struct:
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }
        Guess { value }
    }
}
```

---

## Using `Result<T, E>` in Tests

Tests can also return `Result<(), E>` for more flexibility:
```rust
#[test]
fn it_works() -> Result<(), String> {
    if 2 + 2 == 4 {
        Ok(())
    } else {
        Err(String::from("2 + 2 != 4"))
    }
}
```
You can now use `?` in your test functions, and handle errors without panics.

---

## Summary

Rust's testing framework is simple, expressive, and integrated with the language and build system. You can:
- Use assertions to validate logic.
- Handle panic scenarios.
- Return `Result` types.
- See clear output for failures.

# Controlling How Tests Are Run in Rust

Rust's `cargo test` command compiles your code in test mode and runs the resulting test binary. By default, all tests run in parallel, and output is captured for clarity. However, you can control test behavior through command-line options.

## Running Tests in Parallel or Sequentially

By default, Rust runs tests in **parallel**, using multiple threads to speed up test execution. This is great for performance but can cause issues if tests share state (e.g., writing to the same file).

### Example Problem
If multiple tests write to `test-output.txt`, one might overwrite the other's data, leading to incorrect test failures.

### Solution
Use different files per test or run tests **sequentially**:

```sh
cargo test -- --test-threads=1
```

## Controlling Output

### Default Behavior
Rust **suppresses output** (e.g., `println!`) from successful tests.

- Only output from **failing tests** is shown by default.

### Example:
```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {a}");
    10
}
```

To see output from **passing tests**, use:

```sh
cargo test -- --show-output
```

## Running a Subset of Tests

When working on a specific part of your code, it's efficient to run only related tests.

### Define Test Functions
```rust
#[test]
fn add_two_and_two() {
    assert_eq!(4, add_two(2));
}

#[test]
fn add_three_and_two() {
    assert_eq!(5, add_two(3));
}

#[test]
fn one_hundred() {
    assert_eq!(102, add_two(100));
}
```

### Run a Single Test
```sh
cargo test one_hundred
```

### Run Multiple Tests (Using Name Filtering)
```sh
cargo test add
```
This runs any test with `add` in its name (e.g., `add_two_and_two`, `add_three_and_two`).

## Ignoring Specific Tests

Some tests might be **slow** or run under specific conditions only. You can mark these with `#[ignore]`:

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // long-running task
}
```

### Run Default Tests (Ignored Tests Are Skipped)
```sh
cargo test
```
Output will indicate that `expensive_test` was ignored.

### Run Only Ignored Tests
```sh
cargo test -- --ignored
```

### Run All Tests, Including Ignored
```sh
cargo test -- --include-ignored
```

## Summary

You can control test execution in Rust by using:

| Command | Description |
|--------|-------------|
| `cargo test` | Run all tests in parallel |
| `cargo test -- --test-threads=1` | Run tests sequentially |
| `cargo test -- --show-output` | Show output from passing tests |
| `cargo test <name>` | Run specific test(s) |
| `cargo test -- --ignored` | Run only ignored tests |
| `cargo test -- --include-ignored` | Run all tests, including ignored ones |

**Rust Test Organization: Unit and Integration Tests**

Rust divides tests into two main categories: **unit tests** and **integration tests**. Both types help ensure that individual parts of your program work correctly on their own and when combined.

---

### Unit Tests

Unit tests are focused, small, and test a single module in isolation. They can test private functions and are defined in the same file as the code they test.

#### Where to Write Unit Tests
- Unit tests are placed in the `src` directory.
- Each file should have a `tests` module with `#[cfg(test)]`.
- This tells Rust to compile this module **only when running tests**.

#### Example Unit Test Setup
```rust
// src/lib.rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

#### Why #[cfg(test)]
It avoids compiling test code when running `cargo build`, saving time and binary size.

#### Testing Private Functions
Even private functions can be tested because the `tests` module is a child module and can access parent scope:

```rust
fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_internal_adder() {
        assert_eq!(internal_adder(2, 2), 4);
    }
}
```

---

### Integration Tests

Integration tests verify that many parts of your crate work together correctly. These tests use your library **like an external crate**.

#### Where to Write Integration Tests
- Create a `tests/` directory at the top level of your project (next to `src`).
- Each `.rs` file inside `tests/` is compiled as its **own crate**.

#### Example Directory Layout
```
adder/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── integration_test.rs
```

#### Example Integration Test
```rust
// tests/integration_test.rs
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(adder::add_two(2), 4);
}
```

Note: `use adder;` is required to bring the library crate into scope.

---

### Running Tests

- Run all tests: `cargo test`
- Run only one integration test file: `cargo test --test integration_test`

---

### Reusing Code in Integration Tests

You might need helper functions shared across test files. To do this:

1. Create a `tests/common/mod.rs` file:
```rust
// tests/common/mod.rs
pub fn setup() {
    // setup code here
}
```

2. Use the module in test files:
```rust
// tests/integration_test.rs
use adder;
mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(adder::add_two(2), 4);
}
```

**Important:** Do NOT place helpers in `tests/common.rs`, or it will be treated as a test crate and show up as a test with 0 functions. Using `tests/common/mod.rs` avoids this.

---

### Integration Tests for Binary Crates

If your project only has `src/main.rs`, you can’t test functions using `use` because binaries don’t expose public APIs.

**Best practice:**
- Move logic into `src/lib.rs`
- Keep `src/main.rs` minimal and call into the library

This allows:
- Reuse of logic
- Easy unit and integration testing via the library

---

### Summary
- Use **unit tests** for isolated module-level testing
- Use **integration tests** for testing public APIs and multi-module behavior
- Structure your project to enable testing by splitting logic into `lib.rs`
- Share test setup code using `tests/common/mod.rs`



# Closures in Rust: Explained Simply

Rust closures are a powerful way to write concise, flexible code. Let's break them down step by step so they're easy to understand.

---

## What Are Closures?
Closures are **anonymous functions** you can:
- Store in a variable.
- Pass as arguments to other functions.
- Define inline and use elsewhere.

Closures can **capture variables from the scope** in which they're defined — something normal functions can't do.

---

## Real-World Example: T-Shirt Giveaway
Imagine a company giving away t-shirts. If a user picked a favorite color, they get that. If not, they get the color that's most in stock. Here's the logic:

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }

        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}
```

### Key Idea:
- The `unwrap_or_else` method accepts a **closure**.
- If the user didn’t choose a shirt color, the closure runs to determine the most-stocked color.
- `|| self.most_stocked()` is a closure that captures `self` from the environment.

---

## Syntax and Type Inference
Closures can be written in many ways:

```rust
// Function
fn add_one(x: u32) -> u32 { x + 1 }

// Fully annotated closure
let add_one = |x: u32| -> u32 { x + 1 };

// Inferred types
let add_one = |x| x + 1;
```

Closures are flexible because Rust often infers parameter and return types.

---

## Closures Capture Environment Variables
Closures can automatically access variables outside their body.

### 1. Immutable Reference:
```rust
let list = vec![1, 2, 3];
let print_list = || println!("{:?}", list);
print_list();
```
The closure uses `list` immutably — so it can be reused and doesn’t restrict access.

### 2. Mutable Reference:
```rust
let mut list = vec![1, 2, 3];
let mut modify_list = || list.push(4);
modify_list();
```
The closure mutates the `list`, so it captures it mutably.

### 3. Taking Ownership:
```rust
let list = vec![1, 2, 3];
let handle = thread::spawn(move || println!("{:?}", list));
handle.join().unwrap();
```
Using `move`, the closure takes **ownership** of `list`, necessary when working across threads.

---

## The Fn Traits
Closures in Rust automatically implement one or more of the following traits:

| Trait     | Description |
|-----------|-------------|
| `FnOnce`  | Closure can be called **once**. It **moves** values out. |
| `FnMut`   | Closure can be called multiple times and **mutates** values. |
| `Fn`      | Closure can be called multiple times and only uses **immutable references**. |

### Example: `unwrap_or_else`
```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T,
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```
`unwrap_or_else` uses `FnOnce` because it might only call the closure once.

### Example: `sort_by_key`
```rust
list.sort_by_key(|r| r.width);
```
The closure is used multiple times, so `sort_by_key` requires `FnMut`.

---

## Closures and Type Restrictions
Once a closure is called with a specific type, Rust locks in that type:

```rust
let example = |x| x;
example(String::from("hi"));
example(5); // ERROR: types don’t match
```

---

## Summary
Closures are:
- Flexible anonymous functions.
- Can borrow, mutate, or own captured variables.
- Useful for callbacks, event handlers, and functional patterns.
- Differ from functions because they capture their **environment**.

# Iterators in Rust: Processing a Series of Items

## What Are Iterators?

Iterators in Rust are a powerful tool that lets you process a sequence of items one at a time. They're designed to abstract the logic of traversing items and are *lazy*, meaning they do nothing until you ask them to. Instead of writing repetitive loop logic, you can write clean, expressive code that works on collections.

---

## Creating an Iterator

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
```

- `v1.iter()` creates an iterator over references to the items in `v1`.
- At this point, the iterator does nothing. It needs to be *used* (consumed) to have any effect.

---

## Using Iterators

The `for` loop automatically consumes the iterator:

```rust
for val in v1_iter {
    println!("Got: {val}");
}
```

This behaves like a traditional loop, but all iteration logic is handled by the iterator.

---

## The Iterator Trait

All iterators implement the `Iterator` trait, which requires:

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

- `type Item`: the type of item being iterated over
- `next`: returns `Some(item)` or `None` when the sequence ends

### Example:

```rust
let mut v1_iter = vec![1, 2, 3].iter();
assert_eq!(v1_iter.next(), Some(&1));
assert_eq!(v1_iter.next(), Some(&2));
assert_eq!(v1_iter.next(), Some(&3));
assert_eq!(v1_iter.next(), None);
```

> Note: Iterators are stateful, so they must be mutable to call `next()`.

---

## Consumers and Adapters

In Rust, iterators are powerful tools for working with sequences of values. Two key concepts related to iterators are **adapters** and **consumers**.

---

### Iterator Adapters

**Adapters** are methods that transform an iterator into another iterator. They are *lazy*, meaning they do not perform any work until a **consumer** calls for the values.
**Adapters** create a step in a proccessing pipeline, but don't actually cause any iteration because they are *lazy*

#### Common Examples:

- `.map(f)` – transforms each element
- `.filter(pred)` – keeps only elements that match a condition
- `.take(n)` – takes the first `n` elements
- `.skip(n)` – skips the first `n` elements
- `.enumerate()` – adds index numbers to items

Adapters return **new iterator types** and can be **chained together**.

```rust
let nums = vec![1, 2, 3, 4];
let iter = nums.iter()
               .filter(|x| **x % 2 == 0)
               .map(|x| x * 10); // Adapter chain
```

---

### Iterator Consumers

**Consumers** are methods that use up the iterator and produce a final value. They are **eager** – they trigger the iteration and consume all or part of the sequence.
**Consumers** are methods that automatically call the next() method on an iterator to consume its values.
#### Examples of Consumers:

- `.collect()` – gathers items into a collection (e.g., `Vec`, `HashMap`, `List`). if from function we are returning -> Vec<String> it will gather items into Vec colletcion. Same for others. Also collect knows what collection to return depdening on *type anotation* too!
**Example** 
````rust
    let uppercase = elements
    .iter()
    .map(|el| el.to_uppercase())
    .collect::<Vec<String>>();
````    
- `.sum()` – sums up all elements
- `.count()` – counts elements
- `.any()`, `.all()` – logical tests
- `.find()`, `.position()` – searches
- `.for_each()` – applies an action to each item

```rust
let nums = vec![1, 2, 3, 4];
let doubled_even: Vec<_> = nums.iter()
    .filter(|x| **x % 2 == 0) // adapter
    .map(|x| x * 2)           // adapter
    .collect();               // consumer
```

---

### Summary

- **Adapters** = Lazy transformers of iterators  
- **Consumers** = Eager evaluators that finalize iteration

You can chain many adapters together and finish with a consumer to get the final result.

---

## Consuming Adaptors

Some methods consume the iterator and return a final result:

```rust
let total: i32 = vec![1, 2, 3].iter().sum();
```

- `sum()` consumes the iterator and returns the sum of all elements.
- Once consumed, the iterator can’t be used again.

---

## Iterator Adapters (Non-Consuming)

Adapter methods transform an iterator into another. They're lazy and must be *consumed* with a method like `collect()`.

### Example: `map`

```rust
let v1 = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

> `map` applies the closure to each item.
> `collect` consumes the iterator and returns a new collection.

---

## Closures in Iterators

Many adapters like `map`, `filter`, etc., take closures.

Closures can **capture values** from their environment:

### Example: `filter` with a captured variable

```rust
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
```

- The closure captures `shoe_size` from its environment.
- `filter` uses the closure to select only matching items.

---

## Choosing the Right Iterator Type

- `iter()`: yields **immutable references** (`&T`)
- `iter_mut()`: yields **mutable references** (`&mut T`)
- `into_iter()`: yields **owned values** (`T`) -> call into_inter only on a calue not referecne!

Choose based on whether you want to read, modify, or consume the collection.

---

## Iterator Summary

| Concept | Description |
|--------|-------------|
| Lazy | Iterator does nothing until consumed |
| `next()` | Core method to step through items |
| Consuming adapters | Methods like `sum()`, `collect()` that use up the iterator |
| Non-consuming adapters | Methods like `map()`, `filter()` that return new iterators |
| Closures | Passed to adapters to customize behavior |

