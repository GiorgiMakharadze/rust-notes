# Rust Programming Notes

# Chapter 1: Rust Basics

### File Naming
Use snake_case when creating files: `my_file.rs` (✅) instead of `myFile.rs` (❌).

### Formatting Code
Use `rustfmt` to format Rust code automatically:
```bash
rustfmt main.rs
```

### Macros
Macros (with `!` suffix) are not normal functions and behave differently:
```rust
println!("Hello, world!");  // println! is a macro
```

### Cargo - Rust’s Package Manager
Similar to `package.json` in JavaScript. Manages dependencies and builds code.

### Crates vs Packages
- **Crate**: A single library or executable (e.g., bigdecimal).
- **Package**: A collection of one or more crates.

### Common Cargo Commands
```bash
cargo new my_app      # Creates a new Rust project
cargo build           # Compiles the project
cargo run             # Compiles & runs the project
cargo check           # Checks for errors without producing a binary
cargo build --release # Builds an optimized binary for production
```

### Project Structure (cargo new my_app)
```
my_app/
├── Cargo.toml  # Project metadata
├── Cargo.lock  # Dependency versions (auto-managed)
└── src/
    └── main.rs  # Main Rust file
```

**Cargo.toml:**
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

Compiled output is stored in:
- `target/debug/` (default)
- `target/release/` (after `cargo build --release`)

# Chapter 2: Variables and Mutability

## Variables & Mutability
By default, variables are immutable:
```rust
let x = 5; // Immutable
```
To make a variable mutable, use `mut`:
```rust
let mut x = 5; // Mutable
```

### Constants
Constants are always immutable. Use `const` (not mut) and must annotate the type. They can only store compile-time values.  
Naming convention: ALLCAPSWITH_UNDERSCORES.

**Example:**
```rust
const SECONDS_IN_MINUTE: u32 = 60;
```

### Shadowing
Allows redeclaring a variable with the same name. Each shadowed variable replaces the previous one within its scope. Use `let` again to shadow a variable. Unlike `mut`, shadowing allows type changes.

**Example:**
```rust
fn main() {
    let x = 5;  
    let x = x + 1; // x = 6

    {
        let x = x * 2; // x = 12 (only inside this block)
        println!("Inner x: {x}"); 
    }

    println!("Outer x: {x}"); // x = 6
}
```

### Shadowing vs mut

**Shadowing:**
```rust
let spaces = "   ";  // String type
let spaces = spaces.len();  // Now it's a number
```
✅ Works because shadowing creates a new variable.

**Using mut (❌ Error):**
```rust
let mut spaces = "   ";  
spaces = spaces.len();  // ❌ Error: Cannot change type
```

**When to Use Shadowing?**
- When modifying a value while keeping it immutable afterward.
- When changing types but reusing the same variable name.

# DATA TYPES

## 1) Scalar Types (Single Value)
Rust has four primary scalar types:
- **Integers** (i8, u8, i16, u16, etc.)
- **Floating-point numbers** (f32, f64)
- **Booleans** (true, false)
- **Characters** (char)

### Integer Types
- **Signed**: Can store both positive and negative numbers.
- **Unsigned**: Can store only positive numbers.

| Length  | Signed  | Unsigned |
|---------|---------|----------|
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| Arch    | `isize` | `usize`  |

### Integer Literals in Rust

| Number Type      | Example      |
|------------------|--------------|
| Decimal          | `98_222`     |
| Hex              | `0xff`       |
| Octal            | `0o77`       |
| Binary           | `0b1111_0000`|
| Byte (u8 only)   | `b'A'`       |

## 2. Floating-Point Types
Rust has two floating-point types:
- `f32` (32-bit)
- `f64` (64-bit, default)

```rust
fn main() {
    let x = 2.0; // f64
}
```
Floating-point numbers are always signed.

### 3. The Character Type
`char` is a single Unicode character (e.g., letters, symbols, emojis).
```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // Explicit type annotation
}
```

## 2) Compound Types
Rust provides two compound types:
- **Tuple**
- **Array**

### The Tuple Type (Stack)
A tuple groups multiple values of different types. Fixed length: Cannot grow or shrink.
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```
Accessing Tuple Values:
- **Destructuring:**
  ```rust
  let (x, y, z) = (500, 6.4, 1);
  ```
- **Indexing:**
  ```rust
  let tup: (i32, f64, u8) = (500, 6.4, 1);
  let five_hundred = tup.0; // Access first value
  ```

### The Array Type (Stack)
All elements must have the same type. Fixed length (unlike vectors). Stored on the stack (faster than heap allocation).
```rust
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
}
```
Accessing Array Values:
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let first = a[0];  // Access first element
}
```
**Mutable Arrays & Tuples:**
- To make an array mutable:
  ```rust
  let mut numbers = [1, 2, 3, 4, 5];
  numbers[0] = 10; // Allowed
  ```
- To make a tuple mutable:
  ```rust
  let mut tup = (1, 2, 3);
  tup.0 = 100; // Allowed
  ```

### The Vector Type (Heap)
A vector (`Vec<T>`) is a resizable array-like data structure in Rust. Unlike arrays and tuples, vectors can grow and shrink dynamically. All elements must be the same type. Vectors are stored in the heap.
```rust
fn main() {
    let mut numbers: Vec<i32> = Vec::new(); // Empty vector
    numbers.push(10);  // Add elements
    numbers.push(20);
    println!("{:?}", numbers); // Output: [10, 20]
}
```

# FUNCTIONS

### Functions & Parameters
Functions are defined using `fn` and can take parameters.

**Example:**
```rust
fn greet(name: &str) {
    println!("Hello, {name}!");
}
```

### 2. Statements & Expressions
- **Statements:** Perform an action but do not return a value.
- **Expressions:** Evaluate to a value and are used within functions.

**Example:**
```rust
let x = 5; // Statement (assigns a value)
let y = {
    let z = 3;
    z + 1 // Expression (evaluates to 4)
}; // y = 4
```

### 3. Functions with Return Values
Return values use an arrow (`->`), and the last expression in the function body is implicitly returned.

**Example:**
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b // No semicolon, so this value is returned
}
```
If you add a semicolon, it becomes a statement, causing an error:
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b; // ❌ ERROR: This does not return a value
}
```
To return early, use `return`:
```rust
fn always_five() -> i32 {
    return 5;
}
```

## 4. Control Flow

#### if Statement
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
*Note:* `if` statements must evaluate to a Boolean (`true` or `false`).

### 5. Repetition with Loops
Rust has three types of loops:
- `loop` (infinite loop)
- `while` (condition-based loop)
- `for` (iterates over a collection)

## `loop` (Infinite Loop)
Runs forever unless you explicitly break out.
```rust
fn main() {
    loop {
        println!("Running forever!");
        break; // Stops the loop
    }
}
```

## Loop Labels (`'label`)
Useful when dealing with nested loops. Allows breaking or continuing a specific loop.

**Example:**
```rust
fn main() {
    let mut count = 0;

    'outer: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break; // Breaks only the inner loop
            }
            if count == 2 {
                break 'outer; // Breaks the outer loop
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {count}");
}
```

#### 2. `while` Loop
Runs while a condition is true.
```rust
fn main() {
    let mut number = 3;

    while number > 0 {
        println!("{number}");
        number -= 1;
    }
}
```
*Downside:* It's slower than `for` loops when iterating over collections.

#### 3. `for` Loop (Best for Iteration)
Efficient way to loop over arrays and collections.
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("The value is: {element}");
    }
}
```

# Chapter 4 - Borrowing and Ownership

# The Stack and Heap
**3️⃣ Stack vs. Heap - Key Differences**.

## Stack vs. Heap - Understanding Memory in Rust 🧠⚡

### 1️⃣ What is the Stack()?
**Definition:**  
A fast and organized region of memory that follows a Last In, First Out (LIFO) order.

**How It Works:**  
New values are pushed onto the stack. Values are popped off in reverse order (last added, first removed). Think of a stack of plates: you add and remove from the top.

**Characteristics:**
- ✔ Stores fixed-size values like integers, floating points, booleans, characters, and arrays. values must be known at compile time.
- ✔ Fast allocation and deallocation—no searching required.
- ✔ Data is stored close together, making it CPU-efficient.

### 2️⃣ What is the Heap?
**Definition:**  
A slower but flexible region of memory where values can grow dynamically.

**How It Works:**  
When storing data, you request memory from the allocator. The allocator finds available space, marks it as used, and returns a pointer to that location. Think of a restaurant: a host finds an available table and gives you directions (a pointer).

**Characteristics:**
- ✔ Stores dynamic-sized data (size not known at compile time).
- ✔ The Heap is for data whose size is not known at compile(user input, file content and so on)   
- ✔ Requires manual management (Rust’s ownership system helps with this).
- ✔ Accessing heap data is slower because the CPU must follow a pointer.

### 3️⃣ Stack vs. Heap - Key Differences
- **Structure:**  
  - Stack: Ordered, follows LIFO (Last-In, First-Out).  
  - Heap: Unstructured, scattered memory allocations.
- **Size:**  
  - Stack: Fixed at compile time.  
  - Heap: Can grow and shrink dynamically.
- **Speed:**  
  - Stack: Fast, since it only involves push/pop operations.  
  - Heap: Slower, because it requires searching for space and managing memory.
- **Access:**  
  - Stack: Direct access—CPU operations are very fast.  
  - Heap: Pointer-based access, requiring extra lookup time.
- **Use Cases:**  
  - Stack: Function calls, fixed-size data (integers, floats, booleans).  
  - Heap: Dynamic data (e.g., Strings, Vectors, complex objects).

### 4️⃣ Why Does Rust Care About Stack & Heap?
- **Function calls use the stack:**  
  Arguments and local variables are pushed onto the stack. When the function ends, they are popped off automatically.
- **Heap memory needs careful management:**  
  The ownership system ensures that data is cleaned up when no longer needed. This prevents memory leaks and duplicate heap data.
- **Stack is faster, heap is flexible:**  
  Rust optimizes for performance by keeping data on the stack whenever possible. If data size is unknown or needs to change, Rust safely uses the heap with ownership rules.

### 5️⃣ Key Takeaways
- ✅ Stack = Fast, simple, fixed-size values (pushed and popped).
- ✅ Heap = Flexible, dynamic data, but slower (uses pointers).
- ✅ Rust’s Ownership System automatically manages heap memory.
- ✅ Understanding this helps explain why Rust has ownership and borrowing rules!

## Ownership and Ownership Rules
Set of Rust Rules. In Rust, each value has a sole owner responsible for its disposal when it's not needed, eliminating the need for garbage collection or reference counting. ownership is Rust's solution for solving memory problems, for making sure that you clean up memory when it is no longer needed.

The purpose of ownership is to assign responsibility for deallocating memory, particularly heap memory.

1. **First rule:** Each value in Rust has an owner. There can only be one owner at a time. When the owner goes out of scope, the value will be dropped.
2. **Variable Scope:**
```rust
fn main() {
    // s is not valid here, since it's not yet declared
    let s = "my owner is s"; // s is valid from this point forward
    // we are doing some stuff here
} // here scope is now over, and s is no longer valid
```
3. **The owne is usually a name**
A `variable` can be an owner.
A `parameter` can be an owner.
A `compisite types` composite types, which are types comprised of multiple elements. So for example, a `tuple`, an `array` are composite types that hold other elements inside them, and a tuple and array are considered the owners of their values.

4. **String Example**

### Revised Notes - The String Type & Ownership

**Why Do We Need String?**
```rust
let s = "Hello";
```
String literals (`"hello"`) are immutable and stored on the stack.

Now as it turns out, this type is actually not stored on either the stack or the heap.
Rather, it is embedded directly into the binary executable that the Rust compiler produces.
And the reason that is done is because the value is already known at compile time. In this program, this string will always be the text "pasta".

**Creating a String**
```rust
let s = String::from("hello");
```
The String type is stored on the heap, allowing dynamic growth. String is useful when working with user input or text unknown at compile time.  
`String::from("hello")` creates a new, heap-allocated string. The `::` (double colon) operator is used to call an associated function on a type.

**Modifying a String**
```rust
{
   let mut s = String::from("hello");
   s.push_str(", world!"); // Appends text
   println!("{s}"); // Prints "hello, world!"
}
```
Unlike string literals, a String can be mutated because it's stored on the heap.

**What is a String Literal?**
- A string literal (`"hello"`) is:
  - Whenever we use a pair of double quotes and that syntax is called a string literal.
  - Stored on the stack (fixed size, known at compile time).
  - Immutable (cannot change).
- A String (`String::from("hello")`) is:
  - Stored on the heap (dynamic size).
  - Mutable (can be modified).

**Why Does This Matter?**  
Ownership rules apply to String, unlike string literals. The heap requires explicit memory management, which Rust handles via ownership and borrowing. Understanding this helps prevent memory leaks and invalid memory access.

# Memory and Allocation

**String Example -**

**Why Are String Literals Fast?**
- Stored directly in the binary at compile time (known size).
- Immutable, making them efficient and lightweight.

**Why Does String Need the Heap?**
- Dynamic size → Cannot be stored in the binary. Memory must be allocated at runtime to support growth.

**Memory Allocation & Cleanup**
- **Allocation:** `String::from("text")` requests memory from the heap.
- **Deallocation:** In garbage-collected (GC) languages, memory is cleaned up automatically.  
  Rust’s approach: No GC → Memory is freed when the owner goes out of scope. This prevents memory leaks, dangling references, and double frees.
```rust
{
   let s = String::from("hello"); // s is valid from this point forward
   // do stuff with s
} // here scope ends and s is no longer valid
```
When a variable goes out of scope, Rust calls a special function called `drop` automatically at the closing curly bracket to free the memory.

Here `s` save in Stack and points ponier to `hello ` that is stored in the Heap

✔ Stack stores: The `s` (pointer, length, capacity).
✔ Heap stores: The actual string data ("hello").
✔ Automatic Cleanup: Rust frees heap memory when s goes out of scope (RAII).

# Variables and Data: Move, Copy, and Clone in Rust

## **📌 Understanding Move, Copy, and Clone in Rust**
Rust handles variables differently depending on whether they store **stack** or **heap** data. The way Rust transfers data between variables prevents **memory errors** while keeping performance efficient.

## **1️⃣ Stack Data (Fixed Size) → Copy**
```rust
let x = 5;
let y = x;  // x is copied, both x and y are valid.
```
✔ Numbers (`i32`, `bool`, `char`) **implement the `Copy` trait** because they are small, fixed-size, and live entirely on the stack.  
✔ Copying stack values is **cheap** because it involves only simple duplication of memory.  
✔ Both `x` and `y` exist independently, and modifying one does **not** affect the other.  
✔ Useful for Simple Calculations: When you need to pass values into functions without worrying about ownership transfer, Copy types are ideal.
✔ Safe for Multiple Uses: Since copying does not affect the original variable, it is perfect for cases where the same value is used in multiple places.

## **2️⃣ Heap Data (Dynamic Size) → Move**
```rust
let s1 = String::from("hello");
let s2 = s1;  // Ownership moves to s2, s1 is no longer valid.
```

✔ A `String` **stores only a pointer, length, and capacity on the stack**.  
✔ **`s1` is on the stack**, while its **actual contents ("hello") live on the heap**.  
✔ When ownership moves from `s1` to `s2` (**which gets a new stack address**), **Rust only moves the stack data** (pointer, length, and capacity).  
✔ The **heap data remains unchanged**—Rust **does not copy** the actual string `"hello"`.  
✔ **`s1` is now invalid**, ensuring **only `s2` has ownership** and preventing **double free errors**.  

### **What Actually Happens During Move?**
```rust
let s1 = String::from("genius");
let s2 = s1;  // Ownership moves from `s1` to `s2`
```
1️⃣ **Rust copies the stack data** (pointer, length, capacity) from `s1` to a **new stack entry for `s2`**.  
2️⃣ **The heap data remains unchanged** (it is **not duplicated**).  
3️⃣ **`s1` is invalidated** so that only `s2` owns the heap memory.  

Rust does not copy the String's text content on the heap. The heap data remains in place, but ownership is transferred to `s2`, making `s1` invalid.

📌 **Memory Representation**
```plaintext
Before Move:
Stack:                   Heap:
s1 -> ptr -----------> ["hello"]

After Move:
Stack:                   Heap:
s2 -> ptr -----------> ["hello"]  // s1 is now invalid
```

## **3️⃣ If We Want a Copy of Heap Data → Use `clone()`**
```rust
let s1 = String::from("hello");
let s2 = s1.clone();  // Deep copy: Heap data is duplicated.
println!("s1 = {s1}, s2 = {s2}");
```
✔ `clone()` **explicitly duplicates** the heap data, so both `s1` and `s2` have **separate** memory allocations.  
✔ Unlike a **move**, `s1` remains valid after `s1.clone()`.  
✔ **Cloning is expensive** (it performs a full copy of heap data), so Rust makes it **explicit** to avoid unnecessary performance costs.  

## **📝 Notes (10 Sentences)**
1. Stack variables (like `i32`, `bool`) are **copied** because they have a fixed size and live entirely on the stack.  
2. Heap variables (like `String`) **cannot be copied automatically** because they require memory allocation.  
3. When assigning a heap variable (`let s2 = s1;`), Rust **moves ownership** instead of copying.  
4. After a move, the original variable (`s1`) **becomes invalid** to prevent memory errors.  
5. **Moving ownership prevents double-free errors**, where two variables could try freeing the same heap memory.  
6. **Use `clone()` if you need an independent copy** of heap-allocated data.  
7. **`clone()` is explicit** because deep copying can be expensive.  
8. **Types that implement the `Copy` trait (like `i32`) are copied automatically instead of being moved**.  
9. **If a type implements `Drop` (requires special cleanup), it cannot implement `Copy`**.  
10. **Tuples can be `Copy` if all their elements are `Copy`, but not if they contain heap data like `String`**.  

# **Ownership and Functions in Rust**

## **📌 How Ownership Works with Functions**
- When passing variables to functions, **Rust applies move or copy just like variable assignments**.  
- **Stack data (`i32`) is copied**, while **heap data (`String`) is moved**.  

## **🔹 Move Example (Heap Data)**
When we pass a variable to a function, it behaves just like assigning it to another variable -> let s1 = String::from("hello"); let s2 = s1; //Ownership moves to s2, s1 is no longer valid.

```rust
fn takes_ownership(some_string: String) {
    println!("{some_string}"); // `some_string` is dropped here
}

fn makes_copy(some_integer: i32) {
    println!("{some_integer}"); // `some_integer` is copied
}

fn main() {
    let s = String::from("hello");
    takes_ownership(s);  // Ownership moves to the function, `s` is now invalid.

    let x = 5;
    makes_copy(x);  // `x` is copied, still valid after function call.
    println!("x is still valid: {x}");
}
```
✔ **`String` moves ownership**, so it **cannot be used after the function call**.  
✔ **`i32` is copied**, so the original variable remains valid after the function executes.  

## **🔹 Returning Values and Ownership**
- A function **can return a value to transfer ownership back** to the caller.

```rust
fn gives_ownership() -> String {
    String::from("hello") // Ownership moves to caller
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string // Ownership is returned
}

fn main() {
    let s1 = gives_ownership(); // `s1` gets ownership from the function
    let s2 = String::from("hello"); // `s2` comes into scope
    let s3 = takes_and_gives_back(s2); // `s2` is moved to function, function returns it to `s3`
    // println!("{s2}"); // ❌ ERROR: `s2` was moved
}
```
✔ **`gives_ownership()` moves a `String` to `s1`**.  
✔ **`takes_and_gives_back()` takes ownership of `s2`, then moves it back to `s3`**.  
✔ **This process is necessary because Rust enforces ownership rules at compile time**.  

## **🔹 Alternative: Returning Multiple Values**
- Instead of manually returning ownership, we can return **multiple values** using a tuple.

```rust
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();  // Get the length of the string
    (s, length)  // Return ownership + length
}

fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1); // `s1` is moved, `s2` takes ownership
    println!("The length of '{s2}' is {len}.");
}
```
✔ **We return both the string and its length** in a tuple.  
✔ **This prevents loss of ownership while still allowing access to data**.  

## **📌 Summary**
✅ **Ownership moves to functions unless the type implements `Copy`**.  
✅ **To retain ownership, return values from functions**.  
✅ **Tuples allow returning multiple values at once**.  
✅ **This prevents memory leaks while enforcing safety**.  

# RULES OF BORROWING AND OWNERSHIP

✅ 1. Each value has a single owner.
✅ 2. When the owner goes out of scope, the value is dropped.
✅ 3. Ownership can be moved (default) or cloned (explicitly).
✅ 4. References allow borrowing without taking ownership.
✅ 5. At any time, either multiple immutable borrows OR one mutable borrow is allowed.
✅ 6. Mutable references (&mut T) must be unique.
✅ 7. References must always be valid (no dangling references).

```md
# 📌 Referencing and Borrowing in Rust

## 🔹 The Problem with Moving Ownership
In Rust, when we pass a **heap-allocated value** (like `String`) to a function, ownership moves, making the original variable **invalid** after the function call.

Example:
```rust
fn calculate_length(s: String) -> usize {
    s.len()
} // `s` is dropped here because ownership was moved.

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(s1); // s1 is moved!
    println!("The length of '{s1}' is {len}"); // ❌ ERROR: s1 is no longer valid!
}
```
- Since `s1` was moved into `calculate_length()`, **we can’t use `s1` after the function call**.
- To fix this, we need a **way to use `s1` in the function without transferring ownership**.

## 🔹 Solution: Using **References (`&`)**
Instead of moving ownership, we can **pass a reference** (`&s1`) to the function.

```rust
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // `s` is dropped, but the original `String` is NOT affected.

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Pass a reference instead of moving ownership
    println!("The length of '{s1}' is {len}."); // ✅ s1 is still valid!
}
```

✔ `&s1` **creates a reference** to `s1` instead of transferring ownership.  
✔ `calculate_length(s: &String)` **takes a reference instead of owning the value**.  
✔ `s1` is still **valid after the function call** because **ownership was never transferred**.

📌 **References allow functions to use a value without taking ownership of it.**  

## 🔹 What Is Borrowing?
- **Borrowing** means **using a reference** instead of taking ownership.  
- The function **borrows the value temporarily** but does **not own it**, so the **original variable remains valid**.  
- **Analogy:** Borrowing a book 📖 → You can read it but must return it without modifying it.  

## 🔹 References Are Immutable by Default
References work like variables: they are **immutable by default**.

Example (❌ Fails because we try to modify a borrowed value):
```rust
fn change(some_string: &String) {
    some_string.push_str(", world"); // ❌ ERROR: Cannot modify an immutable reference!
}

fn main() {
    let s = String::from("hello");
    change(&s);
}
```
⛔ **Error:**
```
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
```
📌 **Why?**
- **`&s` is an immutable reference**, so we **can’t modify the borrowed data**.
- **By default, references do not allow mutation**.

## 🔹 Mutable References (`&mut`)
To allow modification, we need to use **a mutable reference (`&mut`)**.

```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // ✅ Allowed because it's a mutable reference
}

fn main() {
    let mut s = String::from("hello"); // ✅ `s` must be mutable
    change(&mut s); // ✅ Passing a mutable reference
    println!("{s}"); // ✅ Prints "hello, world"
}
```
✔ `&mut s` creates a **mutable reference** to allow modification.  
✔ `some_string: &mut String` takes a **mutable reference** instead of ownership.  
✔ The original `s` **can still be used after modification**.

📌 **Mutable references allow modifying borrowed data while still preventing ownership transfer.**  

## 🔹 Key Rules of References
1️⃣ **A reference does not take ownership**, so the original value **remains valid**.  
2️⃣ **References are immutable by default**. To modify a reference, use **`&mut`**.  
3️⃣ **Only one mutable reference is allowed at a time** (prevents data races).  
4️⃣ **You can have multiple immutable references at the same time** (safe for reading).  
5️⃣ **Mutable and immutable references cannot coexist at the same time**.

## 📌 Summary
✅ **References (`&`) allow functions to use a value without taking ownership**.  
✅ **Borrowing lets us pass data while keeping the original variable valid**.  
✅ **By default, references are immutable (`&T`)**.  
✅ **To modify a reference, use mutable references (`&mut T`)**.  
✅ **Rust prevents multiple mutable references to avoid data races**.  

### **Why This Matters**
- Without references, we **must return values** every time we call a function to **give back ownership**.
- **References allow us to keep ownership while letting functions use our data**.
- This is a **key feature in Rust’s memory safety model**, avoiding **dangling pointers** and **data races**.

# 📌 Mutable References and Borrowing Rules in Rust

## 🔹 Why Do We Need Mutable References?
By default, **references are immutable** in Rust. This means we cannot modify a borrowed value unless we explicitly allow it using **mutable references (`&mut`)**.

### **1️⃣ Example: Using a Mutable Reference**
```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // ✅ Allowed: `some_string` is a mutable reference
}

fn main() {
    let mut s = String::from("hello"); // ✅ `s` must be mutable
    change(&mut s); // ✅ Passing a mutable reference
    println!("{s}"); // ✅ Prints "hello, world"
}
```
✔ `&mut s` **creates a mutable reference**, allowing modification.  
✔ `some_string: &mut String` **takes a mutable reference** instead of moving ownership.  
✔ The original `s` **can still be used after modification**.  

📌 **Mutable references let us modify borrowed data while keeping ownership intact.**  

## 🔹 Rust Prevents Multiple Mutable References
Rust **does not allow multiple mutable references** to the same data at the same time to prevent **data races**.

### **2️⃣ Example: ❌ Multiple Mutable References (Error)**
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s; 
    let r2 = &mut s; // ❌ ERROR: Cannot borrow `s` as mutable more than once!
    
    println!("{r1}, {r2}");
}
```
⛔ **Error Message:**
```
error[E0499]: cannot borrow `s` as mutable more than once at a time
```
📌 **Why Does This Happen?**  
- `r1` **mutably borrows `s`**, meaning **no other reference can exist** until `r1` is used and goes out of scope.  
- `r2` tries to **borrow `s` mutably again**, but `r1` still exists, causing a **conflict**.  
- **Rust prevents this at compile time to ensure safe, controlled mutation.**  

## 🔹 Preventing Borrowing Errors: Use Scopes
Rust allows **multiple mutable references, just not at the same time**.  
We can use **scopes `{}` to control reference lifetimes**.

### **3️⃣ Example: ✅ Using Scopes to Allow Multiple Mutable References**
```rust
fn main() {
    let mut s = String::from("hello");

    { 
        let r1 = &mut s; 
        println!("{r1}"); // ✅ `r1` goes out of scope here
    } 

    let r2 = &mut s; // ✅ Allowed because `r1` is no longer used
    println!("{r2}"); 
}
```
✔ **`r1` is created inside a scope** `{}`.  
✔ **Once `r1` goes out of scope, Rust allows `r2` to exist**.  
✔ **Rust ensures no two mutable references exist at the same time.**  

📌 **Using scopes lets us avoid multiple mutable references while still modifying values.**  

## 🔹 Mixing Mutable and Immutable References
Rust **also prevents a mutable reference from coexisting with immutable references**.

### **4️⃣ Example: ❌ Mutable & Immutable References Together (Error)**
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // ✅ Immutable reference
    let r2 = &s; // ✅ Another immutable reference
    let r3 = &mut s; // ❌ ERROR: Cannot borrow `s` mutably while it's borrowed immutably!
    
    println!("{r1}, {r2}, and {r3}");
}
```
⛔ **Error Message:**
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```
📌 **Why Does This Happen?**  
- **Rust allows multiple immutable references** (`r1`, `r2`) because they do not modify data.  
- **However, `r3` tries to take a mutable reference while immutable references still exist**, which **Rust forbids**.  
- This prevents **unexpected changes while reading data**.  

## 🔹 Rust Allows Mutable References **After** Immutable References Are No Longer Used
The Rust compiler is smart enough to **detect when an immutable reference is no longer needed** before allowing a mutable reference.

### **5️⃣ Example: ✅ Allowed Since Immutable References Are No Longer Used**
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; 
    let r2 = &s; 
    println!("{r1} and {r2}"); // ✅ `r1` and `r2` are last used here

    let r3 = &mut s; // ✅ Allowed: `r1` and `r2` are no longer in use
    println!("{r3}");
}
```
✔ **The immutable references (`r1` and `r2`) are last used before `r3` is created**.  
✔ **Rust allows `r3` to exist because no immutable references exist beyond their last use**.  
✔ **This ensures safety while still allowing flexible reference management.**  

📌 **Rust's borrowing rules prevent data races by ensuring references do not overlap in unsafe ways.**  

## 🔹 What Are Data Races? (Why Rust Enforces These Rules)
A **data race** occurs when:
1️⃣ Two or more references access the same data at the same time.  
2️⃣ At least one of the references **modifies** the data.  
3️⃣ No synchronization exists to prevent conflicts.  

### **Example: Data Race in Other Languages (Unsafe)**
In **C/C++**, this can happen:
```cpp
int shared_value = 10;

void modify() {
    shared_value += 1; // ⚠️ Unsafe: Multiple threads can modify `shared_value`
}

void read() {
    printf("%d\n", shared_value); // ⚠️ Unsafe: No guarantee of what value is read
}
```
- **If two threads modify `shared_value` at the same time**, the value can become **corrupted**.
- Rust **prevents this at compile time** by **restricting multiple references to mutable data**.

## 📌 Key Takeaways  
✅ **Mutable references (`&mut`) allow modifying borrowed data without transferring ownership.**  
✅ **Rust prevents multiple mutable references at the same time to avoid data races.**  
✅ **Mutable references cannot coexist with immutable references to ensure data consistency.**  
✅ **Using scopes (`{}`) lets us reuse mutable references safely.**  
✅ **Rust prevents undefined behavior by enforcing safe memory access rules at compile time.**  

### **Why This Matters**
- **Rust’s borrowing rules eliminate entire classes of memory safety issues** (like data races and dangling pointers).  
- These restrictions **may feel strict at first**, but they ensure **safe and efficient memory management**.  
- **Mastering references is essential** for writing **fast and safe Rust code**!  

# 📌 Dangling References in Rust

## 🔹 What Is a Dangling Reference?
A **dangling reference** occurs when a pointer **refers to memory that has been freed**, leading to **undefined behavior**.  
In many languages (like **C/C++**), it’s possible to create a **dangling pointer** by referencing memory that **no longer exists**.

### **1️⃣ Example: Dangling Pointer in C**
```c
int* create_dangling_pointer() {
    int x = 42;  // x is a local variable
    return &x;   // ❌ ERROR: Returning a pointer to `x`, which will be destroyed!
}

int main() {
    int* p = create_dangling_pointer();  // `p` now points to invalid memory!
    printf("%d", *p); // ❌ Undefined behavior: memory has been freed!
}
```
✔ **This can lead to crashes or memory corruption** because the **memory is already freed**.  
✔ **Rust prevents this by enforcing safe references at compile time!**  

## 🔹 Rust **Prevents** Dangling References
In Rust, the **borrow checker ensures that references always point to valid data**.  
If a reference might become **invalid**, Rust **won’t compile the code**.

### **2️⃣ Example: Dangling Reference in Rust (Compile-Time Error)**
```rust
fn dangle() -> &String { // ❌ ERROR: Returning a reference to a local variable
    let s = String::from("hello"); // `s` is created inside the function
    &s // ❌ ERROR: `s` will be dropped at the end of the function!
} // `s` is deallocated here!

fn main() {
    let reference_to_nothing = dangle(); // ❌ Compilation fails
}
```
⛔ **Error Message:**
```
error[E0106]: missing lifetime specifier
this function's return type contains a borrowed value,
but there is no value for it to be borrowed from
```
📌 **Why Does This Happen?**  
- `s` is **created inside `dangle()`**, so when the function ends, **`s` is dropped**.  
- The reference `&s` would point to **freed memory**, which **Rust prevents**.  
- **Rust ensures that all references remain valid for their entire lifetime**.  

## 🔹 ✅ Correct Way: Return Ownership Instead of a Reference
Instead of returning a **reference to a local variable**, **move ownership out of the function**.

### **3️⃣ Example: Fixing the Dangling Reference**
```rust
fn no_dangle() -> String {
    let s = String::from("hello"); // ✅ `s` is created
    s // ✅ Ownership is moved instead of returning a reference
}

fn main() {
    let valid_string = no_dangle(); // ✅ `valid_string` now owns the String
    println!("{valid_string}"); // ✅ Works fine!
}
```
✔ Instead of returning a **reference**, `s` is **moved out of the function**.  
✔ **Rust enforces ownership rules to ensure safe memory access.**  

## 🔹 The Rules of References in Rust
Rust follows **strict borrowing rules** to prevent **dangling references**:

### ✅ **1. At any given time, you can have either:**
   - **One mutable reference (`&mut`)**  
   - **OR multiple immutable references (`&`)**  
   - ❌ **You cannot have both at the same time**  

### ✅ **2. References Must Always Be Valid**
   - **You cannot return a reference to a variable that will be dropped.**
   - **Rust ensures references always point to live data.**

## 📌 **Summary**
✅ **Dangling references happen when a pointer refers to invalid memory.**  
✅ **Rust prevents dangling references by enforcing ownership and borrowing rules.**  
✅ **A function cannot return a reference to a local variable (use ownership instead).**  
✅ **Rust ensures references always point to valid data at compile time.**  

### **Why This Matters**
- **Rust eliminates entire classes of memory safety bugs** by **enforcing strict reference rules**.  
- **Many languages allow dangling pointers, leading to crashes**—but Rust prevents them **before the program runs**.  
- **Mastering references is key** to writing **safe and efficient Rust programs**.

# 📌 The Slice Type in Rust

## 🔹 What Is a Slice?
A **slice** is a **reference to a part of a collection** instead of the whole collection.  
Slices **do not take ownership**; they just provide a **view into the data**.

✔ **Why use slices?**  
- **Avoid unnecessary copying** of data.  
- **Ensure data remains valid** (prevents out-of-sync indexes).  
- **More efficient and safer** than using raw indexes.

## 🔹 Problem: Finding the First Word Without Slices
Imagine we need to **find the first word** in a string.

### **1️⃣ Attempt Without Using Slices**
```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes(); // ✅ Convert String to bytes

    for (i, &item) in bytes.iter().enumerate() { // ✅ Iterate through characters
        if item == b' ' { // ✅ Check for space
            return i; // ✅ Return index of space
        }
    }
    s.len() // ✅ If no space found, return full length
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // word = 5

    s.clear(); // ❌ Now `word` is invalid because `s` is empty!
}
```
### **❌ Problem with This Approach**
- `word` stores **an index (5)**, but **if `s` is modified (`s.clear()`), the index becomes invalid**.
- **There is no guarantee** that `word` still refers to a valid part of `s`.
- **Bug-prone**: If `s` changes, `word` may reference non-existent data.

📌 **Solution: Use Slices to Return a Valid Part of the String!**  

## 🔹 ✅ Solution: String Slices
A **string slice** is a reference **to a portion of a `String`**, ensuring **it remains valid**.

### **2️⃣ Example: Using Slices**
```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5]; // ✅ Slice: "hello"
    let world = &s[6..11]; // ✅ Slice: "world"

    println!("{hello} {world}"); // Output: hello world
}
```
✔ `&s[0..5]` creates a **slice from index 0 to 5** (not including 5).  
✔ `&s[6..11]` creates a **slice from index 6 to 11** (not including 11).  
✔ **No ownership is taken**, so `s` remains valid.  

📌 **Slices solve the problem by returning a reference to part of `s`, keeping it in sync with the original string.**

## 🔹 Simplified Slice Syntax
Rust allows **shorter syntax** for common slice cases:

```rust
let s = String::from("hello");
let slice1 = &s[..2]; // ✅ Same as &s[0..2]
let slice2 = &s[3..]; // ✅ Same as &s[3..s.len()]
let slice3 = &s[..];  // ✅ Same as &s[0..s.len()]
```
✔ `[..2]` means **"from start to index 2"**.  
✔ `[3..]` means **"from index 3 to the end"**.  
✔ `[..]` means **"the whole string"**.  

## 🔹 ✅ Fixing `first_word()` with Slices
Using a slice ensures **the returned data remains valid**.

### **3️⃣ Correct Version of `first_word()`**
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes(); // ✅ Convert to bytes

    for (i, &item) in bytes.iter().enumerate() { // ✅ Loop through characters
        if item == b' ' {
            return &s[0..i]; // ✅ Return slice from start to space
        }
    }
    &s[..] // ✅ If no space, return entire string
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // ✅ "hello"

    // ❌ ERROR: This will now fail to compile!
    s.clear(); // ❌ We cannot clear `s` while `word` is in use
    println!("The first word is: {word}");
}
```
⛔ **Rust Compiler Error**
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```
📌 **Why is this error GOOD?**  
- The compiler **prevents modifying `s` while `word` exists**.  
- This **eliminates an entire class of bugs!**  

## 🔹 String Literals Are Slices
```rust
let s = "Hello, world!";
```
✔ **The type of `s` is `&str`, a string slice.**  
✔ **String literals are slices stored in the binary at compile time.**  
✔ This is why **string literals are immutable** (`&str` is an immutable reference).  

## 🔹 Making `first_word()` More Flexible
Rust allows **string slices (`&str`) to work with both `String` and string literals**.

### **4️⃣ Improved `first_word()` (More Flexible)**
```rust
fn first_word(s: &str) -> &str { // ✅ Works for &String and &str
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

    let word1 = first_word(&my_string[..]); // ✅ Pass slice of `String`
    let word2 = first_word(&my_string);     // ✅ Works with &String

    let my_string_literal = "hello world";

    let word3 = first_word(&my_string_literal[..]); // ✅ Pass slice of string literal
    let word4 = first_word(my_string_literal);     // ✅ Works directly with &str
}
```
✔ **Works for both `String` and `&str`**, making it more useful.  
✔ **Avoids unnecessary type conversions**.  

📌 **String slices make Rust APIs more flexible and safe!**  

## 🔹 Array Slices
Slices work for **arrays** too!

### **5️⃣ Array Slice Example**
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let slice = &a[1..3]; // ✅ Slice of `[2, 3]`
    assert_eq!(slice, &[2, 3]);

    println!("{:?}", slice); // Output: [2, 3]
}
```
✔ `&a[1..3]` creates a **slice of elements 1 to 3** (excluding index 3).  
✔ The slice **remains tied to `a`**, preventing invalid access.  

📌 **Array slices work just like string slices and are just as safe!**  

## 📌 Summary
✅ **Slices (`&[start..end]`) reference a portion of a collection without taking ownership.**  
✅ **String slices (`&str`) prevent out-of-sync indexes and borrowing errors.**  
✅ **Rust ensures slices are always valid, preventing bugs at compile time.**  
✅ **String literals (`&str`) are slices stored in the binary and are immutable.**  
✅ **Slices work for both `String` and `&str`, making APIs more flexible.**  
✅ **Array slices (`&[i32]`) work the same way as string slices.**  

### **Why This Matters**
- **Slices solve real-world memory safety issues** by keeping references **valid and in sync** with the data.  
- Rust **prevents indexing bugs at compile time**, unlike many other languages.  
- **Mastering slices is key to writing efficient Rust code**!  

# 📌 Chapter 4 Summary: Ownership, Borrowing, References, and Slices

## 🔹 Why Is Ownership Important?
Rust **ensures memory safety at compile time** through **ownership rules**, preventing memory leaks, dangling pointers, and data races **without a garbage collector**.

✔ **Ownership defines how memory is managed in Rust.**  
✔ **Borrowing lets functions use data without taking ownership.**  
✔ **References allow safe and efficient access to variables.**  
✔ **Slices let us work with parts of collections without copying.**  

Ownership **affects all aspects of Rust programming**, making it essential to understand these concepts.

## 🔹 1️⃣ Ownership: The Foundation of Rust’s Memory Safety
**Ownership rules ensure that every value has a single owner.**

### ✅ Ownership Rules:
1️⃣ **Each value in Rust has one and only one owner.**  
2️⃣ **When the owner goes out of scope, Rust automatically cleans up the value.**  
3️⃣ **Transferring ownership (moving) invalidates the original variable.**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // ✅ Ownership moves from `s1` to `s2`
    
    println!("{s1}"); // ❌ ERROR: `s1` is invalid after move!
}
```
📌 **Rust prevents multiple variables from freeing the same memory (double-free error).**  

## 🔹 2️⃣ Borrowing and References: Using Data Without Taking Ownership
Borrowing allows functions to **access** data **without taking ownership**.

### ✅ References (`&T` - Immutable Borrowing)
```rust
fn calculate_length(s: &String) -> usize { 
    s.len()  // ✅ `s` is borrowed, not moved
}

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s); // ✅ Borrow `s` without ownership transfer
    println!("Length: {len}");
}
```
✔ `&s` **creates a reference** to `s` instead of transferring ownership.  
✔ The function **can read `s` but not modify it**.  
✔ **Multiple immutable references are allowed at the same time.**  

### ✅ Mutable References (`&mut T` - Mutable Borrowing)
If we need to modify borrowed data, we use **mutable references (`&mut`)**.

```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world"); // ✅ Allowed with `&mut`
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s); // ✅ Mutable borrow
    println!("{s}"); // Output: "hello, world"
}
```
✔ **Only one mutable reference is allowed at a time** to prevent data races.  
✔ **We cannot mix mutable and immutable references simultaneously.**  

## 🔹 3️⃣ Dangling References: Rust Prevents Invalid Memory Access
Rust **prevents dangling references** at compile time.

```rust
fn dangle() -> &String { 
    let s = String::from("hello"); // `s` is created inside the function
    &s // ❌ ERROR: `s` will be dropped at the end of the function!
}

fn main() {
    let ref_to_nothing = dangle(); // ❌ Compile-time error
}
```
✔ Rust **ensures all references are valid** before allowing compilation.  
✔ **Solution:** Return ownership instead of a reference.

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s // ✅ Ownership moves out instead of returning a reference
}
```

## 🔹 4️⃣ Slices: Referencing Parts of a Collection
A **slice** is a reference **to part of a collection**, preventing out-of-sync indexes.

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i]; // ✅ Return slice instead of index
        }
    }
    &s[..]
}
```
✔ **Slices fix bugs related to invalid indexes.**  
✔ **String slices (`&str`) reference part of a `String` without copying.**  

### ✅ Array Slices Also Work the Same Way
```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3]; // ✅ Slice of `[2, 3]`
println!("{:?}", slice); // Output: [2, 3]
```
✔ **Slices ensure safe and efficient access to collections.**  

## 📌 Summary of Chapter 4
✅ **Ownership ensures each value has a single owner, preventing memory errors.**  
✅ **Borrowing (`&T`) allows safe access to data without moving ownership.**  
✅ **Mutable references (`&mut T`) allow controlled modification while preventing data races.**  
✅ **Rust prevents dangling references at compile time.**  
✅ **Slices (`&[T]`) provide safe, efficient access to parts of collections.**  
✅ **Rust’s ownership model eliminates memory safety bugs without garbage collection.**  

### **Why This Matters**
- **Ownership and borrowing are the foundation of Rust’s memory safety model.**  
- **Rust eliminates entire classes of memory bugs (use-after-free, data races, and null pointers).**  
- **Understanding these concepts is crucial for writing safe and efficient Rust programs.**  