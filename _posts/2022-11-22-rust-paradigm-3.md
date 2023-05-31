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
    let my_var = "rust_is_fun";   // my_var is valid from this point forward
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

    let my_int = 5;                      // my_int comes into scope

    makes_copy(my_int);             // my_int would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // you can use my_int after

} // Both variable goes out of scope
</pre>

To handle situations where you need to temporarily access a value without taking ownership, Rust introduces borrowing. Borrowing allows multiple references to a value but with certain restrictions to prevent data races and memory issues.
Let's illustrate borrowing with a simple example
```
let my_var = String::from("rust_is_fun");
let len = calculate_length(&my_var);
```
The &my_var syntax lets us create a reference that refers to the value of s1 but does not own it. Because it does not own it, the value it points to will not be dropped when the reference goes out of scope.

**Mutability and Immutability**:
By default, variables in Rust are immutable, meaning they cannot be changed once assigned. This ensures thread safety and enables more predictable code.
However, Rust also allows mutable bindings, denoted with the `mut` keyword, when you need to modify a value.
We can use mutable reference for example:
```
fn main() {
    let mut my_var = String::from("rust_is_fun");

    change(&mut my_var);
}

fn change(some_string: &mut String) {
    some_string.push_str("mutable_borrowing_is_fun");
}
```
But mutable references have **one big restriction**: you can have only one mutable reference to a particular piece of data in a particular scope. The benefit of having this restriction is that Rust can prevent data races. We also cannot have a mutable reference while we have an immutable one. Users of an immutable reference don’t expect the values to suddenly change.

**Pattern Matching**:
Pattern matching is a powerful feature in Rust that allows you to destructure and match on different data structures, including enums, structs, tuples, and more. It enables concise and expressive code when dealing with complex data structures.
Pattern matching is often used in combination with Rust's `match` expression, which compares a value against a series of patterns and executes the corresponding branch based on the matched pattern.
Example: 
<pre>
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // ... other states
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    let coin1 = Coin::Penny;
    let coin2 = Coin::Quarter(UsState::Alaska);

    println!("Value: {} cents", value_in_cents(coin1));
    println!("Value: {} cents", value_in_cents(coin2));
}
</pre>

Resources
======
[Better resource to learn rust, the official doc](https://doc.rust-lang.org/book/)



