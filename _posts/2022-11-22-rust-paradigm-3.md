---
title: 'Rust paradigm'
date: 2022-11-22
permalink: /posts/2022/11/rust-paradigm/
tags:
  - rust
---

Rust is a modern systems programming language that emphasizes safety, performance, and concurrency.
Let's dive into the key aspects of the Rust paradigm...

Key aspects
======
**Ownership and Borrowing**:
   One of the most distinctive features of Rust is its ownership system. It enables memory safety without the need for garbage collection. In Rust, each value has a unique owner, and there can only be one owner at a time. When a value goes out of scope, Rust automatically frees the memory associated with it.
Let's take a simple example:
<pre>
{
    let my_var = "rust_is_fun";   // s is valid from this point forward
    // use my_var
}                      // scope is now over, and my_var is no longer valid
</pre>

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless the data has been moved to be owned by another variable. But you have to known that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. That means there’s no reason to invalid an integers after a move.
Let's take an example:
<pre>
fn main() {
    let my_var = String::from("rust_is_fun");  // my_var comes into scope (warning: heap allocation)

    my_func(my_var);             // my_var's value moves into the function my_func...
                                         // ... and so is no longer valid here

    let my_int = 5;                      // x comes into scope

    makes_copy(my_int);             // my_int would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // you can use my_int after

} // Both variable goes out of scope
</pre>

To handle situations where you need to temporarily access a value without taking ownership, Rust introduces borrowing. Borrowing allows multiple references to a value but with certain restrictions to prevent data races and memory issues. This concept is enforced at compile time by the borrow checker.

