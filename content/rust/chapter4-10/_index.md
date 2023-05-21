+++
title = "4-10"
weight = 40
sort_by = "weight"
template ="book_section_rust.html"

+++





# [Understanding Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html#understanding-ownership)

Ownership is Rust’s most unique feature and has deep implications for the rest of the language. It enables Rust to make memory safety guarantees without needing a garbage collector, so it’s important to understand how ownership works. In this chapter, we’ll talk about ownership as well as several related features: borrowing, slices, and how Rust lays data out in memory.



---



## [What Is Ownership?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#what-is-ownership)

*Ownership* is a set of rules that govern how a Rust program manages memory. All programs have to manage the way they use a computer’s memory while running. Some languages have garbage collection that regularly looks for no-longer-used memory as the program runs; in other languages, the programmer must explicitly allocate and free the memory. Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks. If any of the rules are violated, the program won’t compile. None of the features of ownership will slow down your program while it’s running.

Because ownership is a new concept for many programmers, it does take some time to get used to. The good news is that the more experienced you become with Rust and the rules of the ownership system, the easier you’ll find it to naturally develop code that is safe and efficient. Keep at it!

When you understand ownership, you’ll have a solid foundation for understanding the features that make Rust unique. In this chapter, you’ll learn ownership by working through some examples that focus on a very common data structure: strings.

> ### [The Stack and the Heap](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap)
>
> Many programming languages don’t require you to think about the stack and the heap very often. But in a systems programming language like Rust, whether a value is on the stack or the heap affects how the language behaves and why you have to make certain decisions. Parts of ownership will be described in relation to the stack and the heap later in this chapter, so here is a brief explanation in preparation.
>
> Both the stack and the heap are parts of memory available to your code to use at runtime, but they are structured in different ways. The stack stores values in the order it gets them and removes the values in the opposite order. This is referred to as *last in, first out*. Think of a stack of plates: when you add more plates, you put them on top of the pile, and when you need a plate, you take one off the top. Adding or removing plates from the middle or bottom wouldn’t work as well! Adding data is called *pushing onto the stack*, and removing data is called *popping off the stack*. All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead.
>
> The heap is less organized: when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a *pointer*, which is the address of that location. This process is called *allocating on the heap* and is sometimes abbreviated as just *allocating* (pushing values onto the stack is not considered allocating). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer. Think of being seated at a restaurant. When you enter, you state the number of people in your group, and the host finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.
>
> Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack. Comparatively, allocating space on the heap requires more work because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.
>
> Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. Continuing the analogy, consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap).
>
> When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.
>
> Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that ownership addresses. Once you understand ownership, you won’t need to think about the stack and the heap very often, but knowing that the main purpose of ownership is to manage heap data can help explain why it works the way it does.

### [Ownership Rules](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-rules)

First, let’s take a look at the ownership rules. Keep these rules in mind as we work through the examples that illustrate them:

- Each value in Rust has an *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### [Variable Scope](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variable-scope)

Now that we’re past basic Rust syntax, we won’t include all the "fn main() {" code in examples, so if you’re following along, make sure to put the following examples inside a "main" function manually. As a result, our examples will be a bit more concise, letting us focus on the actual details rather than boilerplate code.

As a first example of ownership, we’ll look at the *scope* of some variables. A scope is the range within a program for which an item is valid. Take the following variable:

```rust
let s = "hello";
```

The variable "s" refers to a string literal, where the value of the string is hardcoded into the text of our program. The variable is valid from the point at which it’s declared until the end of the current *scope*. Listing 4-1 shows a program with comments annotating where the variable "s" would be valid.

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

Listing 4-1: A variable and the scope in which it is valid

In other words, there are two important points in time here:

- When "s" comes *into* scope, it is valid.
- It remains valid until it goes *out of* scope.

At this point, the relationship between scopes and when variables are valid is similar to that in other programming languages. Now we’ll build on top of this understanding by introducing the "String" type.

### [The "String" Type](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-string-type)

To illustrate the rules of ownership, we need a data type that is more complex than those we covered in the [“Data Types”](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) section of Chapter 3. The types covered previously are of a known size, can be stored on the stack and popped off the stack when their scope is over, and can be quickly and trivially copied to make a new, independent instance if another part of code needs to use the same value in a different scope. But we want to look at data that is stored on the heap and explore how Rust knows when to clean up that data, and the "String" type is a great example.

We’ll concentrate on the parts of "String" that relate to ownership. These aspects also apply to other complex data types, whether they are provided by the standard library or created by you. We’ll discuss "String" in more depth in [Chapter 8](https://doc.rust-lang.org/book/ch08-02-strings.html).

We’ve already seen string literals, where a string value is hardcoded into our program. String literals are convenient, but they aren’t suitable for every situation in which we may want to use text. One reason is that they’re immutable. Another is that not every string value can be known when we write our code: for example, what if we want to take user input and store it? For these situations, Rust has a second string type, "String". This type manages data allocated on the heap and as such is able to store an amount of text that is unknown to us at compile time. You can create a "String" from a string literal using the "from" function, like so:

```rust
let s = String::from("hello");
```

The double colon "::" operator allows us to namespace this particular "from" function under the "String" type rather than using some sort of name like "string_from". We’ll discuss this syntax more in the [“Method Syntax”](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax) section of Chapter 5, and when we talk about namespacing with modules in [“Paths for Referring to an Item in the Module Tree”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) in Chapter 7.

This kind of string *can* be mutated:

```rust
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() appends a literal to a String

    println!("{}", s); // This will print "hello, world!"
```

So, what’s the difference here? Why can "String" be mutated but literals cannot? The difference is in how these two types deal with memory.

### [Memory and Allocation](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#memory-and-allocation)

In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s immutability. Unfortunately, we can’t put a blob of memory into the binary for each piece of text whose size is unknown at compile time and whose size might change while running the program.

With the "String" type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents. This means:

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we’re done with our "String".

That first part is done by us: when we call "String::from", its implementation requests the memory it needs. This is pretty much universal in programming languages.

However, the second part is different. In languages with a *garbage collector (GC)*, the GC keeps track of and cleans up memory that isn’t being used anymore, and we don’t need to think about it. In most languages without a GC, it’s our responsibility to identify when memory is no longer being used and to call code to explicitly free it, just as we did to request it. Doing this correctly has historically been a difficult programming problem. If we forget, we’ll waste memory. If we do it too early, we’ll have an invalid variable. If we do it twice, that’s a bug too. We need to pair exactly one "allocate" with exactly one "free".

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. Here’s a version of our scope example from Listing 4-1 using a "String" instead of a string literal:

```rust
    {
        let s = String::from("hello"); // s is valid from this point forward

        // do stuff with s
    }                                  // this scope is now over, and s is no
                                       // longer valid
```

There is a natural point at which we can return the memory our "String" needs to the allocator: when "s" goes out of scope. When a variable goes out of scope, Rust calls a special function for us. This function is called ["drop"](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop), and it’s where the author of "String" can put the code to return the memory. Rust calls "drop" automatically at the closing curly bracket.

> Note: In C++, this pattern of deallocating resources at the end of an item’s lifetime is sometimes called *Resource Acquisition Is Initialization (RAII)*. The "drop" function in Rust will be familiar to you if you’ve used RAII patterns.

This pattern has a profound impact on the way Rust code is written. It may seem simple right now, but the behavior of code can be unexpected in more complicated situations when we want to have multiple variables use the data we’ve allocated on the heap. Let’s explore some of those situations now.



#### [Variables and Data Interacting with Move](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move)

Multiple variables can interact with the same data in different ways in Rust. Let’s look at an example using an integer in Listing 4-2.

```rust
    let x = 5;
    let y = x;
```

Listing 4-2: Assigning the integer value of variable "x" to "y"

We can probably guess what this is doing: “bind the value "5" to "x"; then make a copy of the value in "x" and bind it to "y".” We now have two variables, "x" and "y", and both equal "5". This is indeed what is happening, because integers are simple values with a known, fixed size, and these two "5" values are pushed onto the stack.

Now let’s look at the "String" version:

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

This looks very similar, so we might assume that the way it works would be the same: that is, the second line would make a copy of the value in "s1" and bind it to "s2". But this isn’t quite what happens.

Take a look at Figure 4-1 to see what is happening to "String" under the covers. A "String" is made up of three parts, shown on the left: a pointer to the memory that holds the contents of the string, a length, and a capacity. This group of data is stored on the stack. On the right is the memory on the heap that holds the contents.

![Two tables: the first table contains the representation of s1 on the stack, consisting of its length (5), capacity (5), and a pointer to the first value in the second table. The second table contains the representation of the string data on the heap, byte by byte.](https://doc.rust-lang.org/book/img/trpl04-01.svg)

Figure 4-1: Representation in memory of a "String" holding the value ""hello"" bound to "s1"

The length is how much memory, in bytes, the contents of the "String" are currently using. The capacity is the total amount of memory, in bytes, that the "String" has received from the allocator. The difference between length and capacity matters, but not in this context, so for now, it’s fine to ignore the capacity.

When we assign "s1" to "s2", the "String" data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to. In other words, the data representation in memory looks like Figure 4-2.

![Three tables: tables s1 and s2 representing those strings on the stack, respectively, and both pointing to the same string data on the heap.](https://doc.rust-lang.org/book/img/trpl04-02.svg)

Figure 4-2: Representation in memory of the variable "s2" that has a copy of the pointer, length, and capacity of "s1"

The representation does *not* look like Figure 4-3, which is what memory would look like if Rust instead copied the heap data as well. If Rust did this, the operation "s2 = s1" could be very expensive in terms of runtime performance if the data on the heap were large.

![Four tables: two tables representing the stack data for s1 and s2, and each points to its own copy of string data on the heap.](https://doc.rust-lang.org/book/img/trpl04-03.svg)

Figure 4-3: Another possibility for what "s2 = s1" might do if Rust copied the heap data as well

Earlier, we said that when a variable goes out of scope, Rust automatically calls the "drop" function and cleans up the heap memory for that variable. But Figure 4-2 shows both data pointers pointing to the same location. This is a problem: when "s2" and "s1" go out of scope, they will both try to free the same memory. This is known as a *double free* error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, after the line "let s2 = s1;", Rust considers "s1" as no longer valid. Therefore, Rust doesn’t need to free anything when "s1" goes out of scope. Check out what happens when you try to use "s1" after "s2" is created; it won’t work:

```rust
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
```

You’ll get an error like this because Rust prevents you from using the invalidated reference:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: "s1"
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because "s1" has type "String", which does not implement the "Copy" trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
  |
  = note: this error originates in the macro "$crate::format_args_nl" which comes from the expansion of the macro "println" (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let s2 = s1.clone();
  |                ++++++++

For more information about this error, try "rustc --explain E0382".
error: could not compile "ownership" due to previous error
```

If you’ve heard the terms *shallow copy* and *deep copy* while working with other languages, the concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. But because Rust also invalidates the first variable, instead of being called a shallow copy, it’s known as a *move*. In this example, we would say that "s1" was *moved* into "s2". So, what actually happens is shown in Figure 4-4.

![Three tables: tables s1 and s2 representing those strings on the stack, respectively, and both pointing to the same string data on the heap. Table s1 is grayed out be-cause s1 is no longer valid; only s2 can be used to access the heap data.](https://doc.rust-lang.org/book/img/trpl04-04.svg)

Figure 4-4: Representation in memory after "s1" has been invalidated

That solves our problem! With only "s2" valid, when it goes out of scope it alone will free the memory, and we’re done.

In addition, there’s a design choice that’s implied by this: Rust will never automatically create “deep” copies of your data. Therefore, any *automatic* copying can be assumed to be inexpensive in terms of runtime performance.



#### [Variables and Data Interacting with Clone](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone)

If we *do* want to deeply copy the heap data of the "String", not just the stack data, we can use a common method called "clone". We’ll discuss method syntax in Chapter 5, but because methods are a common feature in many programming languages, you’ve probably seen them before.

Here’s an example of the "clone" method in action:

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```

This works just fine and explicitly produces the behavior shown in Figure 4-3, where the heap data *does* get copied.

When you see a call to "clone", you know that some arbitrary code is being executed and that code may be expensive. It’s a visual indicator that something different is going on.

#### [Stack-Only Data: Copy](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy)

There’s another wrinkle we haven’t talked about yet. This code using integers—part of which was shown in Listing 4-2—works and is valid:

```rust
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
```

But this code seems to contradict what we just learned: we don’t have a call to "clone", but "x" is still valid and wasn’t moved into "y".

The reason is that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. That means there’s no reason we would want to prevent "x" from being valid after we create the variable "y". In other words, there’s no difference between deep and shallow copying here, so calling "clone" wouldn’t do anything different from the usual shallow copying, and we can leave it out.

Rust has a special annotation called the "Copy" trait that we can place on types that are stored on the stack, as integers are (we’ll talk more about traits in [Chapter 10](https://doc.rust-lang.org/book/ch10-02-traits.html)). If a type implements the "Copy" trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable.

Rust won’t let us annotate a type with "Copy" if the type, or any of its parts, has implemented the "Drop" trait. If the type needs something special to happen when the value goes out of scope and we add the "Copy" annotation to that type, we’ll get a compile-time error. To learn about how to add the "Copy" annotation to your type to implement the trait, see [“Derivable Traits”](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) in Appendix C.

So, what types implement the "Copy" trait? You can check the documentation for the given type to be sure, but as a general rule, any group of simple scalar values can implement "Copy", and nothing that requires allocation or is some form of resource can implement "Copy". Here are some of the types that implement "Copy":

- All the integer types, such as "u32".
- The Boolean type, "bool", with values "true" and "false".
- All the floating-point types, such as "f64".
- The character type, "char".
- Tuples, if they only contain types that also implement "Copy". For example, "(i32, i32)" implements "Copy", but "(i32, String)" does not.

### [Ownership and Functions](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-and-functions)

The mechanics of passing a value to a function are similar to those when assigning a value to a variable. Passing a variable to a function will move or copy, just as assignment does. Listing 4-3 has an example with some annotations showing where variables go into and out of scope.

Filename: src/main.rs

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and "drop" is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

Listing 4-3: Functions with ownership and scope annotated

If we tried to use "s" after the call to "takes_ownership", Rust would throw a compile-time error. These static checks protect us from mistakes. Try adding code to "main" that uses "s" and "x" to see where you can use them and where the ownership rules prevent you from doing so.

### [Return Values and Scope](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#return-values-and-scope)

Returning values can also transfer ownership. Listing 4-4 shows an example of a function that returns some value, with similar annotations as those in Listing 4-3.

Filename: src/main.rs

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

Listing 4-4: Transferring ownership of return values

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by "drop" unless ownership of the data has been moved to another variable.

While this works, taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? It’s quite annoying that anything we pass in also needs to be passed back if we want to use it again, in addition to any data resulting from the body of the function that we might want to return as well.

Rust does let us return multiple values using a tuple, as shown in Listing 4-5.

Filename: src/main.rs

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

Listing 4-5: Returning ownership of parameters

But this is too much ceremony and a lot of work for a concept that should be common. Luckily for us, Rust has a feature for using a value without transferring ownership, called *references*.





---



## [References and Borrowing](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing)

The issue with the tuple code in Listing 4-5 is that we have to return the "String" to the calling function so we can still use the "String" after the call to "calculate_length", because the "String" was moved into "calculate_length". Instead, we can provide a reference to the "String" value. A *reference* is like a pointer in that it’s an address we can follow to access the data stored at that address; that data is owned by some other variable. Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type for the life of that reference.

Here is how you would define and use a "calculate_length" function that has a reference to an object as a parameter instead of taking ownership of the value:

Filename: src/main.rs

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

First, notice that all the tuple code in the variable declaration and the function return value is gone. Second, note that we pass "&s1" into "calculate_length" and, in its definition, we take "&String" rather than "String". These ampersands represent *references*, and they allow you to refer to some value without taking ownership of it. Figure 4-5 depicts this concept.

![Three tables: the table for s contains only a pointer to the table for s1. The table for s1 contains the stack data for s1 and points to the string data on the heap.](https://doc.rust-lang.org/book/img/trpl04-05.svg)

Figure 4-5: A diagram of "&String s" pointing at "String s1"

> Note: The opposite of referencing by using "&" is *dereferencing*, which is accomplished with the dereference operator, "*". We’ll see some uses of the dereference operator in Chapter 8 and discuss details of dereferencing in Chapter 15.

Let’s take a closer look at the function call here:

```rust
    let s1 = String::from("hello");

    let len = calculate_length(&s1);
```

The "&s1" syntax lets us create a reference that *refers* to the value of "s1" but does not own it. Because it does not own it, the value it points to will not be dropped when the reference stops being used.

Likewise, the signature of the function uses "&" to indicate that the type of the parameter "s" is a reference. Let’s add some explanatory annotations:

```rust
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

The scope in which the variable "s" is valid is the same as any function parameter’s scope, but the value pointed to by the reference is not dropped when "s" stops being used, because "s" doesn’t have ownership. When functions have references as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.

We call the action of creating a reference *borrowing*. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back. You don’t own it.

So, what happens if we try to modify something we’re borrowing? Try the code in Listing 4-6. Spoiler alert: it doesn’t work!

Filename: src/main.rs

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

Listing 4-6: Attempting to modify a borrowed value

Here’s the error:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow "*some_string" as mutable, as it is behind a "&" reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: "&mut String"
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ "some_string" is a "&" reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try "rustc --explain E0596".
error: could not compile "ownership" due to previous error
```

Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.

### [Mutable References](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references)

We can fix the code from Listing 4-6 to allow us to modify a borrowed value with just a few small tweaks that use, instead, a *mutable reference*:

Filename: src/main.rs

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

First we change "s" to be "mut". Then we create a mutable reference with "&mut s" where we call the "change" function, and update the function signature to accept a mutable reference with "some_string: &mut String". This makes it very clear that the "change" function will mutate the value it borrows.

Mutable references have one big restriction: if you have a mutable reference to a value, you can have no other references to that value. This code that attempts to create two mutable references to "s" will fail:

Filename: src/main.rs

```rust
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
```

Here’s the error:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow "s" as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here

For more information about this error, try "rustc --explain E0499".
error: could not compile "ownership" due to previous error
```

This error says that this code is invalid because we cannot borrow "s" as mutable more than once at a time. The first mutable borrow is in "r1" and must last until it’s used in the "println!", but between the creation of that mutable reference and its usage, we tried to create another mutable reference in "r2" that borrows the same data as "r1".

The restriction preventing multiple mutable references to the same data at the same time allows for mutation but in a very controlled fashion. It’s something that new Rustaceans struggle with because most languages let you mutate whenever you’d like. The benefit of having this restriction is that Rust can prevent data races at compile time. A *data race* is similar to a race condition and happens when these three behaviors occur:

- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There’s no mechanism being used to synchronize access to the data.

Data races cause undefined behavior and can be difficult to diagnose and fix when you’re trying to track them down at runtime; Rust prevents this problem by refusing to compile code with data races!

As always, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not *simultaneous* ones:

```rust
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```

Rust enforces a similar rule for combining mutable and immutable references. This code results in an error:

```rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
```

Here’s the error:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow "s" as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try "rustc --explain E0502".
error: could not compile "ownership" due to previous error
```

Whew! We *also* cannot have a mutable reference while we have an immutable one to the same value.

Users of an immutable reference don’t expect the value to suddenly change out from under them! However, multiple immutable references are allowed because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used. For instance, this code will compile because the last usage of the immutable references, the "println!", occurs before the mutable reference is introduced:

```rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
```

The scopes of the immutable references "r1" and "r2" end after the "println!" where they are last used, which is before the mutable reference "r3" is created. These scopes don’t overlap, so this code is allowed: the compiler can tell that the reference is no longer being used at a point before the end of the scope.

Even though borrowing errors may be frustrating at times, remember that it’s the Rust compiler pointing out a potential bug early (at compile time rather than at runtime) and showing you exactly where the problem is. Then you don’t have to track down why your data isn’t what you thought it was.

### [Dangling References](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references)

In languages with pointers, it’s easy to erroneously create a *dangling pointer*—a pointer that references a location in memory that may have been given to someone else—by freeing some memory while preserving a pointer to that memory. In Rust, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

Let’s try to create a dangling reference to see how Rust prevents them with a compile-time error:

Filename: src/main.rs

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Here’s the error:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the "'static" lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try "rustc --explain E0106".
error: could not compile "ownership" due to previous error
```

This error message refers to a feature we haven’t covered yet: lifetimes. We’ll discuss lifetimes in detail in Chapter 10. But, if you disregard the parts about lifetimes, the message does contain the key to why this code is a problem:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Let’s take a closer look at exactly what’s happening at each stage of our "dangle" code:

Filename: src/main.rs

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

Because "s" is created inside "dangle", when the code of "dangle" is finished, "s" will be deallocated. But we tried to return a reference to it. That means this reference would be pointing to an invalid "String". That’s no good! Rust won’t let us do this.

The solution here is to return the "String" directly:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

This works without any problems. Ownership is moved out, and nothing is deallocated.

### [The Rules of References](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#the-rules-of-references)

Let’s recap what we’ve discussed about references:

- At any given time, you can have *either* one mutable reference *or* any number of immutable references.
- References must always be valid.

Next, we’ll look at a different kind of reference: slices.





---





## [The Slice Type](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type)

*Slices* let you reference a contiguous sequence of elements in a collection rather than the whole collection. A slice is a kind of reference, so it does not have ownership.

Here’s a small programming problem: write a function that takes a string of words separated by spaces and returns the first word it finds in that string. If the function doesn’t find a space in the string, the whole string must be one word, so the entire string should be returned.

Let’s work through how we’d write the signature of this function without using slices, to understand the problem that slices will solve:

```rust
fn first_word(s: &String) -> ?
```

The "first_word" function has a "&String" as a parameter. We don’t want ownership, so this is fine. But what should we return? We don’t really have a way to talk about *part* of a string. However, we could return the index of the end of the word, indicated by a space. Let’s try that, as shown in Listing 4-7.

Filename: src/main.rs

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

Listing 4-7: The "first_word" function that returns a byte index value into the "String" parameter

Because we need to go through the "String" element by element and check whether a value is a space, we’ll convert our "String" to an array of bytes using the "as_bytes" method.

```rust
    let bytes = s.as_bytes();
```

Next, we create an iterator over the array of bytes using the "iter" method:

```rust
    for (i, &item) in bytes.iter().enumerate() {
```

We’ll discuss iterators in more detail in [Chapter 13](https://doc.rust-lang.org/book/ch13-02-iterators.html). For now, know that "iter" is a method that returns each element in a collection and that "enumerate" wraps the result of "iter" and returns each element as part of a tuple instead. The first element of the tuple returned from "enumerate" is the index, and the second element is a reference to the element. This is a bit more convenient than calculating the index ourselves.

Because the "enumerate" method returns a tuple, we can use patterns to destructure that tuple. We’ll be discussing patterns more in [Chapter 6](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values). In the "for" loop, we specify a pattern that has "i" for the index in the tuple and "&item" for the single byte in the tuple. Because we get a reference to the element from ".iter().enumerate()", we use "&" in the pattern.

Inside the "for" loop, we search for the byte that represents the space by using the byte literal syntax. If we find a space, we return the position. Otherwise, we return the length of the string by using "s.len()".

```rust
        if item == b' ' {
            return i;
        }
    }

    s.len()
```

We now have a way to find out the index of the end of the first word in the string, but there’s a problem. We’re returning a "usize" on its own, but it’s only a meaningful number in the context of the "&String". In other words, because it’s a separate value from the "String", there’s no guarantee that it will still be valid in the future. Consider the program in Listing 4-8 that uses the "first_word" function from Listing 4-7.

Filename: src/main.rs

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ""

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```

Listing 4-8: Storing the result from calling the "first_word" function and then changing the "String" contents

This program compiles without any errors and would also do so if we used "word" after calling "s.clear()". Because "word" isn’t connected to the state of "s" at all, "word" still contains the value "5". We could use that value "5" with the variable "s" to try to extract the first word out, but this would be a bug because the contents of "s" have changed since we saved "5" in "word".

Having to worry about the index in "word" getting out of sync with the data in "s" is tedious and error prone! Managing these indices is even more brittle if we write a "second_word" function. Its signature would have to look like this:

```rust
fn second_word(s: &String) -> (usize, usize) {
```

Now we’re tracking a starting *and* an ending index, and we have even more values that were calculated from data in a particular state but aren’t tied to that state at all. We have three unrelated variables floating around that need to be kept in sync.

Luckily, Rust has a solution to this problem: string slices.

### [String Slices](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices)

A *string slice* is a reference to part of a "String", and it looks like this:

```rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

Rather than a reference to the entire "String", "hello" is a reference to a portion of the "String", specified in the extra "[0..5]" bit. We create slices using a range within brackets by specifying "[starting_index..ending_index]", where "starting_index" is the first position in the slice and "ending_index" is one more than the last position in the slice. Internally, the slice data structure stores the starting position and the length of the slice, which corresponds to "ending_index" minus "starting_index". So, in the case of "let world = &s[6..11];", "world" would be a slice that contains a pointer to the byte at index 6 of "s" with a length value of "5".

Figure 4-6 shows this in a diagram.

![Three tables: a table representing the stack data of s, which points to the byte at index 0 in a table of the string data "hello world" on the heap. The third table rep-resents the stack data of the slice world, which has a length value of 5 and points to byte 6 of the heap data table.](https://doc.rust-lang.org/book/img/trpl04-06.svg)

Figure 4-6: String slice referring to part of a "String"

With Rust’s ".." range syntax, if you want to start at index 0, you can drop the value before the two periods. In other words, these are equal:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

By the same token, if your slice includes the last byte of the "String", you can drop the trailing number. That means these are equal:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

You can also drop both values to take a slice of the entire string. So these are equal:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Note: String slice range indices must occur at valid UTF-8 character boundaries. If you attempt to create a string slice in the middle of a multibyte character, your program will exit with an error. For the purposes of introducing string slices, we are assuming ASCII only in this section; a more thorough discussion of UTF-8 handling is in the [“Storing UTF-8 Encoded Text with Strings”](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings) section of Chapter 8.

With all this information in mind, let’s rewrite "first_word" to return a slice. The type that signifies “string slice” is written as "&str":

Filename: src/main.rs

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

We get the index for the end of the word the same way we did in Listing 4-7, by looking for the first occurrence of a space. When we find a space, we return a string slice using the start of the string and the index of the space as the starting and ending indices.

Now when we call "first_word", we get back a single value that is tied to the underlying data. The value is made up of a reference to the starting point of the slice and the number of elements in the slice.

Returning a slice would also work for a "second_word" function:

```rust
fn second_word(s: &String) -> &str {
```

We now have a straightforward API that’s much harder to mess up because the compiler will ensure the references into the "String" remain valid. Remember the bug in the program in Listing 4-8, when we got the index to the end of the first word but then cleared the string so our index was invalid? That code was logically incorrect but didn’t show any immediate errors. The problems would show up later if we kept trying to use the first word index with an emptied string. Slices make this bug impossible and let us know we have a problem with our code much sooner. Using the slice version of "first_word" will throw a compile-time error:

Filename: src/main.rs

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
```

Here’s the compiler error:

```console
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow "s" as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here

For more information about this error, try "rustc --explain E0502".
error: could not compile "ownership" due to previous error
```

Recall from the borrowing rules that if we have an immutable reference to something, we cannot also take a mutable reference. Because "clear" needs to truncate the "String", it needs to get a mutable reference. The "println!" after the call to "clear" uses the reference in "word", so the immutable reference must still be active at that point. Rust disallows the mutable reference in "clear" and the immutable reference in "word" from existing at the same time, and compilation fails. Not only has Rust made our API easier to use, but it has also eliminated an entire class of errors at compile time!



#### [String Literals as Slices](https://doc.rust-lang.org/book/ch04-03-slices.html#string-literals-as-slices)

Recall that we talked about string literals being stored inside the binary. Now that we know about slices, we can properly understand string literals:

```rust
let s = "Hello, world!";
```

The type of "s" here is "&str": it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; "&str" is an immutable reference.

#### [String Slices as Parameters](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters)

Knowing that you can take slices of literals and "String" values leads us to one more improvement on "first_word", and that’s its signature:

```rust
fn first_word(s: &String) -> &str {
```

A more experienced Rustacean would write the signature shown in Listing 4-9 instead because it allows us to use the same function on both "&String" values and "&str" values.

```rust
fn first_word(s: &str) -> &str {
```

Listing 4-9: Improving the "first_word" function by using a string slice for the type of the "s" parameter

If we have a string slice, we can pass that directly. If we have a "String", we can pass a slice of the "String" or a reference to the "String". This flexibility takes advantage of *deref coercions*, a feature we will cover in [“Implicit Deref Coercions with Functions and Methods”](https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods) section of Chapter 15.

Defining a function to take a string slice instead of a reference to a "String" makes our API more general and useful without losing any functionality:

Filename: src/main.rs

```rust
fn main() {
    let my_string = String::from("hello world");

    // "first_word" works on slices of "String"s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // "first_word" also works on references to "String"s, which are equivalent
    // to whole slices of "String"s
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // "first_word" works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### [Other Slices](https://doc.rust-lang.org/book/ch04-03-slices.html#other-slices)

String slices, as you might imagine, are specific to strings. But there’s a more general slice type too. Consider this array:

```rust
let a = [1, 2, 3, 4, 5];
```

Just as we might want to refer to part of a string, we might want to refer to part of an array. We’d do so like this:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

This slice has the type "&[i32]". It works the same way as string slices do, by storing a reference to the first element and a length. You’ll use this kind of slice for all sorts of other collections. We’ll discuss these collections in detail when we talk about vectors in Chapter 8.

## [Summary](https://doc.rust-lang.org/book/ch04-03-slices.html#summary)

The concepts of ownership, borrowing, and slices ensure memory safety in Rust programs at compile time. The Rust language gives you control over your memory usage in the same way as other systems programming languages, but having the owner of data automatically clean up that data when the owner goes out of scope means you don’t have to write and debug extra code to get this control.

Ownership affects how lots of other parts of Rust work, so we’ll talk about these concepts further throughout the rest of the book. Let’s move on to Chapter 5 and look at grouping pieces of data together in a "struct".



---





# [Using Structs to Structure Related Data](https://doc.rust-lang.org/book/ch05-00-structs.html#using-structs-to-structure-related-data)

A *struct*, or *structure*, is a custom data type that lets you package together and name multiple related values that make up a meaningful group. If you’re familiar with an object-oriented language, a *struct* is like an object’s data attributes. In this chapter, we’ll compare and contrast tuples with structs to build on what you already know and demonstrate when structs are a better way to group data.

We’ll demonstrate how to define and instantiate structs. We’ll discuss how to define associated functions, especially the kind of associated functions called *methods*, to specify behavior associated with a struct type. Structs and enums (discussed in Chapter 6) are the building blocks for creating new types in your program’s domain to take full advantage of Rust’s compile-time type checking.



---



## [Defining and Instantiating Structs](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#defining-and-instantiating-structs)

Structs are similar to tuples, discussed in [“The Tuple Type”](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) section, in that both hold multiple related values. Like tuples, the pieces of a struct can be different types. Unlike with tuples, in a struct you’ll name each piece of data so it’s clear what the values mean. Adding these names means that structs are more flexible than tuples: you don’t have to rely on the order of the data to specify or access the values of an instance.

To define a struct, we enter the keyword "struct" and name the entire struct. A struct’s name should describe the significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of the pieces of data, which we call *fields*. For example, Listing 5-1 shows a struct that stores information about a user account.

Filename: src/main.rs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

Listing 5-1: A "User" struct definition

To use a struct after we’ve defined it, we create an *instance* of that struct by specifying concrete values for each of the fields. We create an instance by stating the name of the struct and then add curly brackets containing *key: value* pairs, where the keys are the names of the fields and the values are the data we want to store in those fields. We don’t have to specify the fields in the same order in which we declared them in the struct. In other words, the struct definition is like a general template for the type, and instances fill in that template with particular data to create values of the type. For example, we can declare a particular user as shown in Listing 5-2.

Filename: src/main.rs

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

Listing 5-2: Creating an instance of the "User" struct

To get a specific value from a struct, we use dot notation. For example, to access this user’s email address, we use "user1.email". If the instance is mutable, we can change a value by using the dot notation and assigning into a particular field. Listing 5-3 shows how to change the value in the "email" field of a mutable "User" instance.

Filename: src/main.rs

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

Listing 5-3: Changing the value in the "email" field of a "User" instance

Note that the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable. As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance.

Listing 5-4 shows a "build_user" function that returns a "User" instance with the given email and username. The "active" field gets the value of "true", and the "sign_in_count" gets a value of "1".

Filename: src/main.rs

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

Listing 5-4: A "build_user" function that takes an email and username and returns a "User" instance

It makes sense to name the function parameters with the same name as the struct fields, but having to repeat the "email" and "username" field names and variables is a bit tedious. If the struct had more fields, repeating each name would get even more annoying. Luckily, there’s a convenient shorthand!



### [Using the Field Init Shorthand](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-the-field-init-shorthand)

Because the parameter names and the struct field names are exactly the same in Listing 5-4, we can use the *field init shorthand* syntax to rewrite "build_user" so it behaves exactly the same but doesn’t have the repetition of "username" and "email", as shown in Listing 5-5.

Filename: src/main.rs

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

Listing 5-5: A "build_user" function that uses field init shorthand because the "username" and "email" parameters have the same name as struct fields

Here, we’re creating a new instance of the "User" struct, which has a field named "email". We want to set the "email" field’s value to the value in the "email" parameter of the "build_user" function. Because the "email" field and the "email" parameter have the same name, we only need to write "email" rather than "email: email".

### [Creating Instances from Other Instances with Struct Update Syntax](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax)

It’s often useful to create a new instance of a struct that includes most of the values from another instance, but changes some. You can do this using *struct update syntax*.

First, in Listing 5-6 we show how to create a new "User" instance in "user2" regularly, without the update syntax. We set a new value for "email" but otherwise use the same values from "user1" that we created in Listing 5-2.

Filename: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```

Listing 5-6: Creating a new "User" instance using one of the values from "user1"

Using struct update syntax, we can achieve the same effect with less code, as shown in Listing 5-7. The syntax ".." specifies that the remaining fields not explicitly set should have the same value as the fields in the given instance.

Filename: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

Listing 5-7: Using struct update syntax to set a new "email" value for a "User" instance but to use the rest of the values from "user1"

The code in Listing 5-7 also creates an instance in "user2" that has a different value for "email" but has the same values for the "username", "active", and "sign_in_count" fields from "user1". The "..user1" must come last to specify that any remaining fields should get their values from the corresponding fields in "user1", but we can choose to specify values for as many fields as we want in any order, regardless of the order of the fields in the struct’s definition.

Note that the struct update syntax uses "=" like an assignment; this is because it moves the data, just as we saw in the [“Variables and Data Interacting with Move”](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move) section. In this example, we can no longer use "user1" as a whole after creating "user2" because the "String" in the "username" field of "user1" was moved into "user2". If we had given "user2" new "String" values for both "email" and "username", and thus only used the "active" and "sign_in_count" values from "user1", then "user1" would still be valid after creating "user2". Both "active" and "sign_in_count" are types that implement the "Copy" trait, so the behavior we discussed in the [“Stack-Only Data: Copy”](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy) section would apply.

### [Using Tuple Structs Without Named Fields to Create Different Types](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types)

Rust also supports structs that look similar to tuples, called *tuple structs*. Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields. Tuple structs are useful when you want to give the whole tuple a name and make the tuple a different type from other tuples, and when naming each field as in a regular struct would be verbose or redundant.

To define a tuple struct, start with the "struct" keyword and the struct name followed by the types in the tuple. For example, here we define and use two tuple structs named "Color" and "Point":

Filename: src/main.rs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

Note that the "black" and "origin" values are different types because they’re instances of different tuple structs. Each struct you define is its own type, even though the fields within the struct might have the same types. For example, a function that takes a parameter of type "Color" cannot take a "Point" as an argument, even though both types are made up of three "i32" values. Otherwise, tuple struct instances are similar to tuples in that you can destructure them into their individual pieces, and you can use a "." followed by the index to access an individual value.

### [Unit-Like Structs Without Any Fields](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#unit-like-structs-without-any-fields)

You can also define structs that don’t have any fields! These are called *unit-like structs* because they behave similarly to "()", the unit type that we mentioned in [“The Tuple Type”](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) section. Unit-like structs can be useful when you need to implement a trait on some type but don’t have any data that you want to store in the type itself. We’ll discuss traits in Chapter 10. Here’s an example of declaring and instantiating a unit struct named "AlwaysEqual":

Filename: src/main.rs

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

To define "AlwaysEqual", we use the "struct" keyword, the name we want, and then a semicolon. No need for curly brackets or parentheses! Then we can get an instance of "AlwaysEqual" in the "subject" variable in a similar way: using the name we defined, without any curly brackets or parentheses. Imagine that later we’ll implement behavior for this type such that every instance of "AlwaysEqual" is always equal to every instance of any other type, perhaps to have a known result for testing purposes. We wouldn’t need any data to implement that behavior! You’ll see in Chapter 10 how to define traits and implement them on any type, including unit-like structs.

> ### [Ownership of Struct Data](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#ownership-of-struct-data)
>
> In the "User" struct definition in Listing 5-1, we used the owned "String" type rather than the "&str" string slice type. This is a deliberate choice because we want each instance of this struct to own all of its data and for that data to be valid for as long as the entire struct is valid.
>
> It’s also possible for structs to store references to data owned by something else, but to do so requires the use of *lifetimes*, a Rust feature that we’ll discuss in Chapter 10. Lifetimes ensure that the data referenced by a struct is valid for as long as the struct is. Let’s say you try to store a reference in a struct without specifying lifetimes, like the following; this won’t work:
>
> Filename: src/main.rs
>
> ```rust
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
> 
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> The compiler will complain that it needs lifetime specifiers:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
> 
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
> 
> For more information about this error, try "rustc --explain E0106".
> error: could not compile "structs" due to 2 previous errors
> ```
>
> In Chapter 10, we’ll discuss how to fix these errors so you can store references in structs, but for now, we’ll fix errors like these using owned types like "String" instead of references like "&str".



---



## [An Example Program Using Structs](https://doc.rust-lang.org/book/ch05-02-example-structs.html#an-example-program-using-structs)

To understand when we might want to use structs, let’s write a program that calculates the area of a rectangle. We’ll start by using single variables, and then refactor the program until we’re using structs instead.

Let’s make a new binary project with Cargo called *rectangles* that will take the width and height of a rectangle specified in pixels and calculate the area of the rectangle. Listing 5-8 shows a short program with one way of doing exactly that in our project’s *src/main.rs*.

Filename: src/main.rs

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

Listing 5-8: Calculating the area of a rectangle specified by separate width and height variables

Now, run this program using "cargo run":

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running "target/debug/rectangles"
The area of the rectangle is 1500 square pixels.
```

This code succeeds in figuring out the area of the rectangle by calling the "area" function with each dimension, but we can do more to make this code clear and readable.

The issue with this code is evident in the signature of "area":

```rust
fn area(width: u32, height: u32) -> u32 {
```

The "area" function is supposed to calculate the area of one rectangle, but the function we wrote has two parameters, and it’s not clear anywhere in our program that the parameters are related. It would be more readable and more manageable to group width and height together. We’ve already discussed one way we might do that in [“The Tuple Type”](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) section of Chapter 3: by using tuples.

### [Refactoring with Tuples](https://doc.rust-lang.org/book/ch05-02-example-structs.html#refactoring-with-tuples)

Listing 5-9 shows another version of our program that uses tuples.

Filename: src/main.rs

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

Listing 5-9: Specifying the width and height of the rectangle with a tuple

In one way, this program is better. Tuples let us add a bit of structure, and we’re now passing just one argument. But in another way, this version is less clear: tuples don’t name their elements, so we have to index into the parts of the tuple, making our calculation less obvious.

Mixing up the width and height wouldn’t matter for the area calculation, but if we want to draw the rectangle on the screen, it would matter! We would have to keep in mind that "width" is the tuple index "0" and "height" is the tuple index "1". This would be even harder for someone else to figure out and keep in mind if they were to use our code. Because we haven’t conveyed the meaning of our data in our code, it’s now easier to introduce errors.

### [Refactoring with Structs: Adding More Meaning](https://doc.rust-lang.org/book/ch05-02-example-structs.html#refactoring-with-structs-adding-more-meaning)

We use structs to add meaning by labeling the data. We can transform the tuple we’re using into a struct with a name for the whole as well as names for the parts, as shown in Listing 5-10.

Filename: src/main.rs

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

Listing 5-10: Defining a "Rectangle" struct

Here we’ve defined a struct and named it "Rectangle". Inside the curly brackets, we defined the fields as "width" and "height", both of which have type "u32". Then, in "main", we created a particular instance of "Rectangle" that has a width of "30" and a height of "50".

Our "area" function is now defined with one parameter, which we’ve named "rectangle", whose type is an immutable borrow of a struct "Rectangle" instance. As mentioned in Chapter 4, we want to borrow the struct rather than take ownership of it. This way, "main" retains its ownership and can continue using "rect1", which is the reason we use the "&" in the function signature and where we call the function.

The "area" function accesses the "width" and "height" fields of the "Rectangle" instance (note that accessing fields of a borrowed struct instance does not move the field values, which is why you often see borrows of structs). Our function signature for "area" now says exactly what we mean: calculate the area of "Rectangle", using its "width" and "height" fields. This conveys that the width and height are related to each other, and it gives descriptive names to the values rather than using the tuple index values of "0" and "1". This is a win for clarity.

### [Adding Useful Functionality with Derived Traits](https://doc.rust-lang.org/book/ch05-02-example-structs.html#adding-useful-functionality-with-derived-traits)

It’d be useful to be able to print an instance of "Rectangle" while we’re debugging our program and see the values for all its fields. Listing 5-11 tries using the ["println!" macro](https://doc.rust-lang.org/std/macro.println.html) as we have used in previous chapters. This won’t work, however.

Filename: src/main.rs

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {}", rect1);
}
```

Listing 5-11: Attempting to print a "Rectangle" instance

When we compile this code, we get an error with this core message:

```text
error[E0277]: "Rectangle" doesn't implement "std::fmt::Display"
```

The "println!" macro can do many kinds of formatting, and by default, the curly brackets tell "println!" to use formatting known as "Display": output intended for direct end user consumption. The primitive types we’ve seen so far implement "Display" by default because there’s only one way you’d want to show a "1" or any other primitive type to a user. But with structs, the way "println!" should format the output is less clear because there are more display possibilities: Do you want commas or not? Do you want to print the curly brackets? Should all the fields be shown? Due to this ambiguity, Rust doesn’t try to guess what we want, and structs don’t have a provided implementation of "Display" to use with "println!" and the "{}" placeholder.

If we continue reading the errors, we’ll find this helpful note:

```text
   = help: the trait "std::fmt::Display" is not implemented for "Rectangle"
   = note: in format strings you may be able to use "{:?}" (or {:#?} for pretty-print) instead
```

Let’s try it! The "println!" macro call will now look like "println!("rect1 is {:?}", rect1);". Putting the specifier ":?" inside the curly brackets tells "println!" we want to use an output format called "Debug". The "Debug" trait enables us to print our struct in a way that is useful for developers so we can see its value while we’re debugging our code.

Compile the code with this change. Drat! We still get an error:

```text
error[E0277]: "Rectangle" doesn't implement "Debug"
```

But again, the compiler gives us a helpful note:

```text
   = help: the trait "Debug" is not implemented for "Rectangle"
   = note: add "#[derive(Debug)]" to "Rectangle" or manually "impl Debug for Rectangle"
```

Rust *does* include functionality to print out debugging information, but we have to explicitly opt in to make that functionality available for our struct. To do that, we add the outer attribute "#[derive(Debug)]" just before the struct definition, as shown in Listing 5-12.

Filename: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```

Listing 5-12: Adding the attribute to derive the "Debug" trait and printing the "Rectangle" instance using debug formatting

Now when we run the program, we won’t get any errors, and we’ll see the following output:

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running "target/debug/rectangles"
rect1 is Rectangle { width: 30, height: 50 }
```

Nice! It’s not the prettiest output, but it shows the values of all the fields for this instance, which would definitely help during debugging. When we have larger structs, it’s useful to have output that’s a bit easier to read; in those cases, we can use "{:#?}" instead of "{:?}" in the "println!" string. In this example, using the "{:#?}" style will output the following:

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running "target/debug/rectangles"
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

Another way to print out a value using the "Debug" format is to use the ["dbg!" macro](https://doc.rust-lang.org/std/macro.dbg.html), which takes ownership of an expression (as opposed to "println!", which takes a reference), prints the file and line number of where that "dbg!" macro call occurs in your code along with the resultant value of that expression, and returns ownership of the value.

> Note: Calling the "dbg!" macro prints to the standard error console stream ("stderr"), as opposed to "println!", which prints to the standard output console stream ("stdout"). We’ll talk more about "stderr" and "stdout" in the [“Writing Error Messages to Standard Error Instead of Standard Output” section in Chapter 12](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html).

Here’s an example where we’re interested in the value that gets assigned to the "width" field, as well as the value of the whole struct in "rect1":

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

We can put "dbg!" around the expression "30 * scale" and, because "dbg!" returns ownership of the expression’s value, the "width" field will get the same value as if we didn’t have the "dbg!" call there. We don’t want "dbg!" to take ownership of "rect1", so we use a reference to "rect1" in the next call. Here’s what the output of this example looks like:

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running "target/debug/rectangles"
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

We can see the first bit of output came from *src/main.rs* line 10 where we’re debugging the expression "30 * scale", and its resultant value is "60" (the "Debug" formatting implemented for integers is to print only their value). The "dbg!" call on line 14 of *src/main.rs* outputs the value of "&rect1", which is the "Rectangle" struct. This output uses the pretty "Debug" formatting of the "Rectangle" type. The "dbg!" macro can be really helpful when you’re trying to figure out what your code is doing!

In addition to the "Debug" trait, Rust has provided a number of traits for us to use with the "derive" attribute that can add useful behavior to our custom types. Those traits and their behaviors are listed in [Appendix C](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html). We’ll cover how to implement these traits with custom behavior as well as how to create your own traits in Chapter 10. There are also many attributes other than "derive"; for more information, see [the “Attributes” section of the Rust Reference](https://doc.rust-lang.org/reference/attributes.html).

Our "area" function is very specific: it only computes the area of rectangles. It would be helpful to tie this behavior more closely to our "Rectangle" struct because it won’t work with any other type. Let’s look at how we can continue to refactor this code by turning the "area" function into an "area" *method* defined on our "Rectangle" type.







---





## [Method Syntax](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax)

*Methods* are similar to functions: we declare them with the "fn" keyword and a name, they can have parameters and a return value, and they contain some code that’s run when the method is called from somewhere else. Unlike functions, methods are defined within the context of a struct (or an enum or a trait object, which we cover in [Chapter 6](https://doc.rust-lang.org/book/ch06-00-enums.html) and [Chapter 17](https://doc.rust-lang.org/book/ch17-02-trait-objects.html), respectively), and their first parameter is always "self", which represents the instance of the struct the method is being called on.

### [Defining Methods](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#defining-methods)

Let’s change the "area" function that has a "Rectangle" instance as a parameter and instead make an "area" method defined on the "Rectangle" struct, as shown in Listing 5-13.

Filename: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

Listing 5-13: Defining an "area" method on the "Rectangle" struct

To define the function within the context of "Rectangle", we start an "impl" (implementation) block for "Rectangle". Everything within this "impl" block will be associated with the "Rectangle" type. Then we move the "area" function within the "impl" curly brackets and change the first (and in this case, only) parameter to be "self" in the signature and everywhere within the body. In "main", where we called the "area" function and passed "rect1" as an argument, we can instead use *method syntax* to call the "area" method on our "Rectangle" instance. The method syntax goes after an instance: we add a dot followed by the method name, parentheses, and any arguments.

In the signature for "area", we use "&self" instead of "rectangle: &Rectangle". The "&self" is actually short for "self: &Self". Within an "impl" block, the type "Self" is an alias for the type that the "impl" block is for. Methods must have a parameter named "self" of type "Self" for their first parameter, so Rust lets you abbreviate this with only the name "self" in the first parameter spot. Note that we still need to use the "&" in front of the "self" shorthand to indicate that this method borrows the "Self" instance, just as we did in "rectangle: &Rectangle". Methods can take ownership of "self", borrow "self" immutably, as we’ve done here, or borrow "self" mutably, just as they can any other parameter.

We chose "&self" here for the same reason we used "&Rectangle" in the function version: we don’t want to take ownership, and we just want to read the data in the struct, not write to it. If we wanted to change the instance that we’ve called the method on as part of what the method does, we’d use "&mut self" as the first parameter. Having a method that takes ownership of the instance by using just "self" as the first parameter is rare; this technique is usually used when the method transforms "self" into something else and you want to prevent the caller from using the original instance after the transformation.

The main reason for using methods instead of functions, in addition to providing method syntax and not having to repeat the type of "self" in every method’s signature, is for organization. We’ve put all the things we can do with an instance of a type in one "impl" block rather than making future users of our code search for capabilities of "Rectangle" in various places in the library we provide.

Note that we can choose to give a method the same name as one of the struct’s fields. For example, we can define a method on "Rectangle" that is also named "width":

Filename: src/main.rs

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```

Here, we’re choosing to make the "width" method return "true" if the value in the instance’s "width" field is greater than "0" and "false" if the value is "0": we can use a field within a method of the same name for any purpose. In "main", when we follow "rect1.width" with parentheses, Rust knows we mean the method "width". When we don’t use parentheses, Rust knows we mean the field "width".

Often, but not always, when we give a method the same name as a field we want it to only return the value in the field and do nothing else. Methods like this are called *getters*, and Rust does not implement them automatically for struct fields as some other languages do. Getters are useful because you can make the field private but the method public, and thus enable read-only access to that field as part of the type’s public API. We will discuss what public and private are and how to designate a field or method as public or private in [Chapter 7](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword).

> ### [Where’s the "->" Operator?](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator)
>
> In C and C++, two different operators are used for calling methods: you use "." if you’re calling a method on the object directly and "->" if you’re calling the method on a pointer to the object and need to dereference the pointer first. In other words, if "object" is a pointer, "object->something()" is similar to "(*object).something()".
>
> Rust doesn’t have an equivalent to the "->" operator; instead, Rust has a feature called *automatic referencing and dereferencing*. Calling methods is one of the few places in Rust that has this behavior.
>
> Here’s how it works: when you call a method with "object.something()", Rust automatically adds in "&", "&mut", or "*" so "object" matches the signature of the method. In other words, the following are the same:
>
> ```rust
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> The first one looks much cleaner. This automatic referencing behavior works because methods have a clear receiver—the type of "self". Given the receiver and name of a method, Rust can figure out definitively whether the method is reading ("&self"), mutating ("&mut self"), or consuming ("self"). The fact that Rust makes borrowing implicit for method receivers is a big part of making ownership ergonomic in practice.

### [Methods with More Parameters](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#methods-with-more-parameters)

Let’s practice using methods by implementing a second method on the "Rectangle" struct. This time we want an instance of "Rectangle" to take another instance of "Rectangle" and return "true" if the second "Rectangle" can fit completely within "self" (the first "Rectangle"); otherwise, it should return "false". That is, once we’ve defined the "can_hold" method, we want to be able to write the program shown in Listing 5-14.

Filename: src/main.rs

```rust
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

Listing 5-14: Using the as-yet-unwritten "can_hold" method

The expected output would look like the following because both dimensions of "rect2" are smaller than the dimensions of "rect1", but "rect3" is wider than "rect1":

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

We know we want to define a method, so it will be within the "impl Rectangle" block. The method name will be "can_hold", and it will take an immutable borrow of another "Rectangle" as a parameter. We can tell what the type of the parameter will be by looking at the code that calls the method: "rect1.can_hold(&rect2)" passes in "&rect2", which is an immutable borrow to "rect2", an instance of "Rectangle". This makes sense because we only need to read "rect2" (rather than write, which would mean we’d need a mutable borrow), and we want "main" to retain ownership of "rect2" so we can use it again after calling the "can_hold" method. The return value of "can_hold" will be a Boolean, and the implementation will check whether the width and height of "self" are greater than the width and height of the other "Rectangle", respectively. Let’s add the new "can_hold" method to the "impl" block from Listing 5-13, shown in Listing 5-15.

Filename: src/main.rs

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 5-15: Implementing the "can_hold" method on "Rectangle" that takes another "Rectangle" instance as a parameter

When we run this code with the "main" function in Listing 5-14, we’ll get our desired output. Methods can take multiple parameters that we add to the signature after the "self" parameter, and those parameters work just like parameters in functions.

### [Associated Functions](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#associated-functions)

All functions defined within an "impl" block are called *associated functions* because they’re associated with the type named after the "impl". We can define associated functions that don’t have "self" as their first parameter (and thus are not methods) because they don’t need an instance of the type to work with. We’ve already used one function like this: the "String::from" function that’s defined on the "String" type.

Associated functions that aren’t methods are often used for constructors that will return a new instance of the struct. These are often called "new", but "new" isn’t a special name and isn’t built into the language. For example, we could choose to provide an associated function named "square" that would have one dimension parameter and use that as both width and height, thus making it easier to create a square "Rectangle" rather than having to specify the same value twice:

Filename: src/main.rs

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

The "Self" keywords in the return type and in the body of the function are aliases for the type that appears after the "impl" keyword, which in this case is "Rectangle".

To call this associated function, we use the "::" syntax with the struct name; "let sq = Rectangle::square(3);" is an example. This function is namespaced by the struct: the "::" syntax is used for both associated functions and namespaces created by modules. We’ll discuss modules in [Chapter 7](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html).

### [Multiple "impl" Blocks](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#multiple-impl-blocks)

Each struct is allowed to have multiple "impl" blocks. For example, Listing 5-15 is equivalent to the code shown in Listing 5-16, which has each method in its own "impl" block.

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

Listing 5-16: Rewriting Listing 5-15 using multiple "impl" blocks

There’s no reason to separate these methods into multiple "impl" blocks here, but this is valid syntax. We’ll see a case in which multiple "impl" blocks are useful in Chapter 10, where we discuss generic types and traits.

## [Summary](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#summary)

Structs let you create custom types that are meaningful for your domain. By using structs, you can keep associated pieces of data connected to each other and name each piece to make your code clear. In "impl" blocks, you can define functions that are associated with your type, and methods are a kind of associated function that let you specify the behavior that instances of your structs have.

But structs aren’t the only way you can create custom types: let’s turn to Rust’s enum feature to add another tool to your toolbox.









---







# [Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html#enums-and-pattern-matching)

In this chapter, we’ll look at *enumerations*, also referred to as *enums*. Enums allow you to define a type by enumerating its possible *variants*. First we’ll define and use an enum to show how an enum can encode meaning along with data. Next, we’ll explore a particularly useful enum, called "Option", which expresses that a value can be either something or nothing. Then we’ll look at how pattern matching in the "match" expression makes it easy to run different code for different values of an enum. Finally, we’ll cover how the "if let" construct is another convenient and concise idiom available to handle enums in your code.









---









## [Defining an Enum](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#defining-an-enum)

Where structs give you a way of grouping together related fields and data, like a "Rectangle" with its "width" and "height", enums give you a way of saying a value is one of a possible set of values. For example, we may want to say that "Rectangle" is one of a set of possible shapes that also includes "Circle" and "Triangle". To do this, Rust allows us to encode these possibilities as an enum.

Let’s look at a situation we might want to express in code and see why enums are useful and more appropriate than structs in this case. Say we need to work with IP addresses. Currently, two major standards are used for IP addresses: version four and version six. Because these are the only possibilities for an IP address that our program will come across, we can *enumerate* all possible variants, which is where enumeration gets its name.

Any IP address can be either a version four or a version six address, but not both at the same time. That property of IP addresses makes the enum data structure appropriate because an enum value can only be one of its variants. Both version four and version six addresses are still fundamentally IP addresses, so they should be treated as the same type when the code is handling situations that apply to any kind of IP address.

We can express this concept in code by defining an "IpAddrKind" enumeration and listing the possible kinds an IP address can be, "V4" and "V6". These are the variants of the enum:

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

"IpAddrKind" is now a custom data type that we can use elsewhere in our code.

### [Enum Values](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values)

We can create instances of each of the two variants of "IpAddrKind" like this:

```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

Note that the variants of the enum are namespaced under its identifier, and we use a double colon to separate the two. This is useful because now both values "IpAddrKind::V4" and "IpAddrKind::V6" are of the same type: "IpAddrKind". We can then, for instance, define a function that takes any "IpAddrKind":

```rust
fn route(ip_kind: IpAddrKind) {}
```

And we can call this function with either variant:

```rust
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
```

Using enums has even more advantages. Thinking more about our IP address type, at the moment we don’t have a way to store the actual IP address *data*; we only know what *kind* it is. Given that you just learned about structs in Chapter 5, you might be tempted to tackle this problem with structs as shown in Listing 6-1.

```rust
    enum IpAddrKind {
        V4,
        V6,
    }

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

Listing 6-1: Storing the data and "IpAddrKind" variant of an IP address using a "struct"

Here, we’ve defined a struct "IpAddr" that has two fields: a "kind" field that is of type "IpAddrKind" (the enum we defined previously) and an "address" field of type "String". We have two instances of this struct. The first is "home", and it has the value "IpAddrKind::V4" as its "kind" with associated address data of "127.0.0.1". The second instance is "loopback". It has the other variant of "IpAddrKind" as its "kind" value, "V6", and has address "::1" associated with it. We’ve used a struct to bundle the "kind" and "address" values together, so now the variant is associated with the value.

However, representing the same concept using just an enum is more concise: rather than an enum inside a struct, we can put data directly into each enum variant. This new definition of the "IpAddr" enum says that both "V4" and "V6" variants will have associated "String" values:

```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
```

We attach data to each variant of the enum directly, so there is no need for an extra struct. Here, it’s also easier to see another detail of how enums work: the name of each enum variant that we define also becomes a function that constructs an instance of the enum. That is, "IpAddr::V4()" is a function call that takes a "String" argument and returns an instance of the "IpAddr" type. We automatically get this constructor function defined as a result of defining the enum.

There’s another advantage to using an enum rather than a struct: each variant can have different types and amounts of associated data. Version four IP addresses will always have four numeric components that will have values between 0 and 255. If we wanted to store "V4" addresses as four "u8" values but still express "V6" addresses as one "String" value, we wouldn’t be able to with a struct. Enums handle this case with ease:

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

We’ve shown several different ways to define data structures to store version four and version six IP addresses. However, as it turns out, wanting to store IP addresses and encode which kind they are is so common that [the standard library has a definition we can use!](https://doc.rust-lang.org/std/net/enum.IpAddr.html) Let’s look at how the standard library defines "IpAddr": it has the exact enum and variants that we’ve defined and used, but it embeds the address data inside the variants in the form of two different structs, which are defined differently for each variant:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

This code illustrates that you can put any kind of data inside an enum variant: strings, numeric types, or structs, for example. You can even include another enum! Also, standard library types are often not much more complicated than what you might come up with.

Note that even though the standard library contains a definition for "IpAddr", we can still create and use our own definition without conflict because we haven’t brought the standard library’s definition into our scope. We’ll talk more about bringing types into scope in Chapter 7.

Let’s look at another example of an enum in Listing 6-2: this one has a wide variety of types embedded in its variants.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Listing 6-2: A "Message" enum whose variants each store different amounts and types of values

This enum has four variants with different types:

- "Quit" has no data associated with it at all.
- "Move" has named fields, like a struct does.
- "Write" includes a single "String".
- "ChangeColor" includes three "i32" values.

Defining an enum with variants such as the ones in Listing 6-2 is similar to defining different kinds of struct definitions, except the enum doesn’t use the "struct" keyword and all the variants are grouped together under the "Message" type. The following structs could hold the same data that the preceding enum variants hold:

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

But if we used the different structs, each of which has its own type, we couldn’t as easily define a function to take any of these kinds of messages as we could with the "Message" enum defined in Listing 6-2, which is a single type.

There is one more similarity between enums and structs: just as we’re able to define methods on structs using "impl", we’re also able to define methods on enums. Here’s a method named "call" that we could define on our "Message" enum:

```rust
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
```

The body of the method would use "self" to get the value that we called the method on. In this example, we’ve created a variable "m" that has the value "Message::Write(String::from("hello"))", and that is what "self" will be in the body of the "call" method when "m.call()" runs.

Let’s look at another enum in the standard library that is very common and useful: "Option".

### [The "Option" Enum and Its Advantages Over Null Values](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values)

This section explores a case study of "Option", which is another enum defined by the standard library. The "Option" type encodes the very common scenario in which a value could be something or it could be nothing.

For example, if you request the first item in a non-empty list, you would get a value. If you request the first item in an empty list, you would get nothing. Expressing this concept in terms of the type system means the compiler can check whether you’ve handled all the cases you should be handling; this functionality can prevent bugs that are extremely common in other programming languages.

Programming language design is often thought of in terms of which features you include, but the features you exclude are important too. Rust doesn’t have the null feature that many other languages have. *Null* is a value that means there is no value there. In languages with null, variables can always be in one of two states: null or not-null.

In his 2009 presentation “Null References: The Billion Dollar Mistake,” Tony Hoare, the inventor of null, has this to say:

> I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.

However, the concept that null is trying to express is still a useful one: a null is a value that is currently invalid or absent for some reason.

The problem isn’t really with the concept but with the particular implementation. As such, Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is "Option<T>", and it is [defined by the standard library](https://doc.rust-lang.org/std/option/enum.Option.html) as follows:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

The "Option<T>" enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. Its variants are also included in the prelude: you can use "Some" and "None" directly without the "Option::" prefix. The "Option<T>" enum is still just a regular enum, and "Some(T)" and "None" are still variants of type "Option<T>".

The "<T>" syntax is a feature of Rust we haven’t talked about yet. It’s a generic type parameter, and we’ll cover generics in more detail in Chapter 10. For now, all you need to know is that "<T>" means that the "Some" variant of the "Option" enum can hold one piece of data of any type, and that each concrete type that gets used in place of "T" makes the overall "Option<T>" type a different type. Here are some examples of using "Option" values to hold number types and string types:

```rust
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
```

The type of "some_number" is "Option<i32>". The type of "some_char" is "Option<char>", which is a different type. Rust can infer these types because we’ve specified a value inside the "Some" variant. For "absent_number", Rust requires us to annotate the overall "Option" type: the compiler can’t infer the type that the corresponding "Some" variant will hold by looking only at a "None" value. Here, we tell Rust that we mean for "absent_number" to be of type "Option<i32>".

When we have a "Some" value, we know that a value is present and the value is held within the "Some". When we have a "None" value, in some sense it means the same thing as null: we don’t have a valid value. So why is having "Option<T>" any better than having null?

In short, because "Option<T>" and "T" (where "T" can be any type) are different types, the compiler won’t let us use an "Option<T>" value as if it were definitely a valid value. For example, this code won’t compile, because it’s trying to add an "i8" to an "Option<i8>":

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

If we run this code, we get an error message like this one:

```console
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add "Option<i8>" to "i8"
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for "i8 + Option<i8>"
  |
  = help: the trait "Add<Option<i8>>" is not implemented for "i8"
  = help: the following other types implement trait "Add<Rhs>":
            <&'a i8 as Add<i8>>
            <&i8 as Add<&i8>>
            <i8 as Add<&i8>>
            <i8 as Add>

For more information about this error, try "rustc --explain E0277".
error: could not compile "enums" due to previous error
```

Intense! In effect, this error message means that Rust doesn’t understand how to add an "i8" and an "Option<i8>", because they’re different types. When we have a value of a type like "i8" in Rust, the compiler will ensure that we always have a valid value. We can proceed confidently without having to check for null before using that value. Only when we have an "Option<i8>" (or whatever type of value we’re working with) do we have to worry about possibly not having a value, and the compiler will make sure we handle that case before using the value.

In other words, you have to convert an "Option<T>" to a "T" before you can perform "T" operations with it. Generally, this helps catch one of the most common issues with null: assuming that something isn’t null when it actually is.

Eliminating the risk of incorrectly assuming a not-null value helps you to be more confident in your code. In order to have a value that can possibly be null, you must explicitly opt in by making the type of that value "Option<T>". Then, when you use that value, you are required to explicitly handle the case when the value is null. Everywhere that a value has a type that isn’t an "Option<T>", you *can* safely assume that the value isn’t null. This was a deliberate design decision for Rust to limit null’s pervasiveness and increase the safety of Rust code.

So how do you get the "T" value out of a "Some" variant when you have a value of type "Option<T>" so that you can use that value? The "Option<T>" enum has a large number of methods that are useful in a variety of situations; you can check them out in [its documentation](https://doc.rust-lang.org/std/option/enum.Option.html). Becoming familiar with the methods on "Option<T>" will be extremely useful in your journey with Rust.

In general, in order to use an "Option<T>" value, you want to have code that will handle each variant. You want some code that will run only when you have a "Some(T)" value, and this code is allowed to use the inner "T". You want some other code to run only if you have a "None" value, and that code doesn’t have a "T" value available. The "match" expression is a control flow construct that does just this when used with enums: it will run different code depending on which variant of the enum it has, and that code can use the data inside the matching value.









---







## [The "match" Control Flow Construct](https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-construct)

Rust has an extremely powerful control flow construct called "match" that allows you to compare a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of literal values, variable names, wildcards, and many other things; [Chapter 18](https://doc.rust-lang.org/book/ch18-00-patterns.html) covers all the different kinds of patterns and what they do. The power of "match" comes from the expressiveness of the patterns and the fact that the compiler confirms that all possible cases are handled.

Think of a "match" expression as being like a coin-sorting machine: coins slide down a track with variously sized holes along it, and each coin falls through the first hole it encounters that it fits into. In the same way, values go through each pattern in a "match", and at the first pattern the value “fits,” the value falls into the associated code block to be used during execution.

Speaking of coins, let’s use them as an example using "match"! We can write a function that takes an unknown US coin and, in a similar way as the counting machine, determines which coin it is and returns its value in cents, as shown in Listing 6-3.

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

Listing 6-3: An enum and a "match" expression that has the variants of the enum as its patterns

Let’s break down the "match" in the "value_in_cents" function. First we list the "match" keyword followed by an expression, which in this case is the value "coin". This seems very similar to a conditional expression used with "if", but there’s a big difference: with "if", the condition needs to evaluate to a Boolean value, but here it can be any type. The type of "coin" in this example is the "Coin" enum that we defined on the first line.

Next are the "match" arms. An arm has two parts: a pattern and some code. The first arm here has a pattern that is the value "Coin::Penny" and then the "=>" operator that separates the pattern and the code to run. The code in this case is just the value "1". Each arm is separated from the next with a comma.

When the "match" expression executes, it compares the resultant value against the pattern of each arm, in order. If a pattern matches the value, the code associated with that pattern is executed. If that pattern doesn’t match the value, execution continues to the next arm, much as in a coin-sorting machine. We can have as many arms as we need: in Listing 6-3, our "match" has four arms.

The code associated with each arm is an expression, and the resultant value of the expression in the matching arm is the value that gets returned for the entire "match" expression.

We don’t typically use curly brackets if the match arm code is short, as it is in Listing 6-3 where each arm just returns a value. If you want to run multiple lines of code in a match arm, you must use curly brackets, and the comma following the arm is then optional. For example, the following code prints “Lucky penny!” every time the method is called with a "Coin::Penny", but still returns the last value of the block, "1":

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### [Patterns That Bind to Values](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values)

Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is how we can extract values out of enum variants.

As an example, let’s change one of our enum variants to hold data inside it. From 1999 through 2008, the United States minted quarters with different designs for each of the 50 states on one side. No other coins got state designs, so only quarters have this extra value. We can add this information to our "enum" by changing the "Quarter" variant to include a "UsState" value stored inside it, which we’ve done in Listing 6-4.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

Listing 6-4: A "Coin" enum in which the "Quarter" variant also holds a "UsState" value

Let’s imagine that a friend is trying to collect all 50 state quarters. While we sort our loose change by coin type, we’ll also call out the name of the state associated with each quarter so that if it’s one our friend doesn’t have, they can add it to their collection.

In the match expression for this code, we add a variable called "state" to the pattern that matches values of the variant "Coin::Quarter". When a "Coin::Quarter" matches, the "state" variable will bind to the value of that quarter’s state. Then we can use "state" in the code for that arm, like so:

```rust
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

If we were to call "value_in_cents(Coin::Quarter(UsState::Alaska))", "coin" would be "Coin::Quarter(UsState::Alaska)". When we compare that value with each of the match arms, none of them match until we reach "Coin::Quarter(state)". At that point, the binding for "state" will be the value "UsState::Alaska". We can then use that binding in the "println!" expression, thus getting the inner state value out of the "Coin" enum variant for "Quarter".

### [Matching with "Option"](https://doc.rust-lang.org/book/ch06-02-match.html#matching-with-optiont)

In the previous section, we wanted to get the inner "T" value out of the "Some" case when using "Option<T>"; we can also handle "Option<T>" using "match", as we did with the "Coin" enum! Instead of comparing coins, we’ll compare the variants of "Option<T>", but the way the "match" expression works remains the same.

Let’s say we want to write a function that takes an "Option<i32>" and, if there’s a value inside, adds 1 to that value. If there isn’t a value inside, the function should return the "None" value and not attempt to perform any operations.

This function is very easy to write, thanks to "match", and will look like Listing 6-5.

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

Listing 6-5: A function that uses a "match" expression on an "Option<i32>"

Let’s examine the first execution of "plus_one" in more detail. When we call "plus_one(five)", the variable "x" in the body of "plus_one" will have the value "Some(5)". We then compare that against each match arm:

```rust
            None => None,
```

The "Some(5)" value doesn’t match the pattern "None", so we continue to the next arm:

```rust
            Some(i) => Some(i + 1),
```

Does "Some(5)" match "Some(i)"? It does! We have the same variant. The "i" binds to the value contained in "Some", so "i" takes the value "5". The code in the match arm is then executed, so we add 1 to the value of "i" and create a new "Some" value with our total "6" inside.

Now let’s consider the second call of "plus_one" in Listing 6-5, where "x" is "None". We enter the "match" and compare to the first arm:

```rust
            None => None,
```

It matches! There’s no value to add to, so the program stops and returns the "None" value on the right side of "=>". Because the first arm matched, no other arms are compared.

Combining "match" and enums is useful in many situations. You’ll see this pattern a lot in Rust code: "match" against an enum, bind a variable to the data inside, and then execute code based on it. It’s a bit tricky at first, but once you get used to it, you’ll wish you had it in all languages. It’s consistently a user favorite.

### [Matches Are Exhaustive](https://doc.rust-lang.org/book/ch06-02-match.html#matches-are-exhaustive)

There’s one other aspect of "match" we need to discuss: the arms’ patterns must cover all possibilities. Consider this version of our "plus_one" function, which has a bug and won’t compile:

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```

We didn’t handle the "None" case, so this code will cause a bug. Luckily, it’s a bug Rust knows how to catch. If we try to compile this code, we’ll get this error:

```console
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0004]: non-exhaustive patterns: "None" not covered
 --> src/main.rs:3:15
  |
3 |         match x {
  |               ^ pattern "None" not covered
  |
note: "Option<i32>" defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type "Option<i32>"
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
4 ~             Some(i) => Some(i + 1),
5 ~             None => todo!(),
  |

For more information about this error, try "rustc --explain E0004".
error: could not compile "enums" due to previous error
```

Rust knows that we didn’t cover every possible case, and even knows which pattern we forgot! Matches in Rust are *exhaustive*: we must exhaust every last possibility in order for the code to be valid. Especially in the case of "Option<T>", when Rust prevents us from forgetting to explicitly handle the "None" case, it protects us from assuming that we have a value when we might have null, thus making the billion-dollar mistake discussed earlier impossible.

### [Catch-all Patterns and the "_" Placeholder](https://doc.rust-lang.org/book/ch06-02-match.html#catch-all-patterns-and-the-_-placeholder)

Using enums, we can also take special actions for a few particular values, but for all other values take one default action. Imagine we’re implementing a game where, if you roll a 3 on a dice roll, your player doesn’t move, but instead gets a new fancy hat. If you roll a 7, your player loses a fancy hat. For all other values, your player moves that number of spaces on the game board. Here’s a "match" that implements that logic, with the result of the dice roll hardcoded rather than a random value, and all other logic represented by functions without bodies because actually implementing them is out of scope for this example:

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

For the first two arms, the patterns are the literal values "3" and "7". For the last arm that covers every other possible value, the pattern is the variable we’ve chosen to name "other". The code that runs for the "other" arm uses the variable by passing it to the "move_player" function.

This code compiles, even though we haven’t listed all the possible values a "u8" can have, because the last pattern will match all values not specifically listed. This catch-all pattern meets the requirement that "match" must be exhaustive. Note that we have to put the catch-all arm last because the patterns are evaluated in order. If we put the catch-all arm earlier, the other arms would never run, so Rust will warn us if we add arms after a catch-all!

Rust also has a pattern we can use when we want a catch-all but don’t want to *use* the value in the catch-all pattern: "_" is a special pattern that matches any value and does not bind to that value. This tells Rust we aren’t going to use the value, so Rust won’t warn us about an unused variable.

Let’s change the rules of the game: now, if you roll anything other than a 3 or a 7, you must roll again. We no longer need to use the catch-all value, so we can change our code to use "_" instead of the variable named "other":

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

This example also meets the exhaustiveness requirement because we’re explicitly ignoring all other values in the last arm; we haven’t forgotten anything.

Finally, we’ll change the rules of the game one more time so that nothing else happens on your turn if you roll anything other than a 3 or a 7. We can express that by using the unit value (the empty tuple type we mentioned in [“The Tuple Type”](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) section) as the code that goes with the "_" arm:

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```

Here, we’re telling Rust explicitly that we aren’t going to use any other value that doesn’t match a pattern in an earlier arm, and we don’t want to run any code in this case.

There’s more about patterns and matching that we’ll cover in [Chapter 18](https://doc.rust-lang.org/book/ch18-00-patterns.html). For now, we’re going to move on to the "if let" syntax, which can be useful in situations where the "match" expression is a bit wordy.









---









## [Concise Control Flow with "if let"](https://doc.rust-lang.org/book/ch06-03-if-let.html#concise-control-flow-with-if-let)

The "if let" syntax lets you combine "if" and "let" into a less verbose way to handle values that match one pattern while ignoring the rest. Consider the program in Listing 6-6 that matches on an "Option<u8>" value in the "config_max" variable but only wants to execute code if the value is the "Some" variant.

```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
```

Listing 6-6: A "match" that only cares about executing code when the value is "Some"

If the value is "Some", we print out the value in the "Some" variant by binding the value to the variable "max" in the pattern. We don’t want to do anything with the "None" value. To satisfy the "match" expression, we have to add "_ => ()" after processing just one variant, which is annoying boilerplate code to add.

Instead, we could write this in a shorter way using "if let". The following code behaves the same as the "match" in Listing 6-6:

```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```

The syntax "if let" takes a pattern and an expression separated by an equal sign. It works the same way as a "match", where the expression is given to the "match" and the pattern is its first arm. In this case, the pattern is "Some(max)", and the "max" binds to the value inside the "Some". We can then use "max" in the body of the "if let" block in the same way we used "max" in the corresponding "match" arm. The code in the "if let" block isn’t run if the value doesn’t match the pattern.

Using "if let" means less typing, less indentation, and less boilerplate code. However, you lose the exhaustive checking that "match" enforces. Choosing between "match" and "if let" depends on what you’re doing in your particular situation and whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.

In other words, you can think of "if let" as syntax sugar for a "match" that runs code when the value matches one pattern and then ignores all other values.

We can include an "else" with an "if let". The block of code that goes with the "else" is the same as the block of code that would go with the "_" case in the "match" expression that is equivalent to the "if let" and "else". Recall the "Coin" enum definition in Listing 6-4, where the "Quarter" variant also held a "UsState" value. If we wanted to count all non-quarter coins we see while also announcing the state of the quarters, we could do that with a "match" expression, like this:

```rust
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
```

Or we could use an "if let" and "else" expression, like this:

```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
```

If you have a situation in which your program has logic that is too verbose to express using a "match", remember that "if let" is in your Rust toolbox as well.

## [Summary](https://doc.rust-lang.org/book/ch06-03-if-let.html#summary)

We’ve now covered how to use enums to create custom types that can be one of a set of enumerated values. We’ve shown how the standard library’s "Option<T>" type helps you use the type system to prevent errors. When enum values have data inside them, you can use "match" or "if let" to extract and use those values, depending on how many cases you need to handle.

Your Rust programs can now express concepts in your domain using structs and enums. Creating custom types to use in your API ensures type safety: the compiler will make certain your functions only get values of the type each function expects.

In order to provide a well-organized API to your users that is straightforward to use and only exposes exactly what your users will need, let’s now turn to Rust’s modules.





---





# [Managing Growing Projects with Packages, Crates, and Modules](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html#managing-growing-projects-with-packages-crates-and-modules)

As you write large programs, organizing your code will become increasingly important. By grouping related functionality and separating code with distinct features, you’ll clarify where to find code that implements a particular feature and where to go to change how a feature works.

The programs we’ve written so far have been in one module in one file. As a project grows, you should organize code by splitting it into multiple modules and then multiple files. A package can contain multiple binary crates and optionally one library crate. As a package grows, you can extract parts into separate crates that become external dependencies. This chapter covers all these techniques. For very large projects comprising a set of interrelated packages that evolve together, Cargo provides *workspaces*, which we’ll cover in the [“Cargo Workspaces”](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) section in Chapter 14.

We’ll also discuss encapsulating implementation details, which lets you reuse code at a higher level: once you’ve implemented an operation, other code can call your code via its public interface without having to know how the implementation works. The way you write code defines which parts are public for other code to use and which parts are private implementation details that you reserve the right to change. This is another way to limit the amount of detail you have to keep in your head.

A related concept is scope: the nested context in which code is written has a set of names that are defined as “in scope.” When reading, writing, and compiling code, programmers and compilers need to know whether a particular name at a particular spot refers to a variable, function, struct, enum, module, constant, or other item and what that item means. You can create scopes and change which names are in or out of scope. You can’t have two items with the same name in the same scope; tools are available to resolve name conflicts.

Rust has a number of features that allow you to manage your code’s organization, including which details are exposed, which details are private, and what names are in each scope in your programs. These features, sometimes collectively referred to as the *module system*, include:

- **Packages:** A Cargo feature that lets you build, test, and share crates
- **Crates:** A tree of modules that produces a library or executable
- **Modules** and **use:** Let you control the organization, scope, and privacy of paths
- **Paths:** A way of naming an item, such as a struct, function, or module

In this chapter, we’ll cover all these features, discuss how they interact, and explain how to use them to manage scope. By the end, you should have a solid understanding of the module system and be able to work with scopes like a pro!







---







## [Packages and Crates](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html#packages-and-crates)

The first parts of the module system we’ll cover are packages and crates.

A *crate* is the smallest amount of code that the Rust compiler considers at a time. Even if you run "rustc" rather than "cargo" and pass a single source code file (as we did all the way back in the “Writing and Running a Rust Program” section of Chapter 1), the compiler considers that file to be a crate. Crates can contain modules, and the modules may be defined in other files that get compiled with the crate, as we’ll see in the coming sections.

A crate can come in one of two forms: a binary crate or a library crate. *Binary crates* are programs you can compile to an executable that you can run, such as a command-line program or a server. Each must have a function called "main" that defines what happens when the executable runs. All the crates we’ve created so far have been binary crates.

*Library crates* don’t have a "main" function, and they don’t compile to an executable. Instead, they define functionality intended to be shared with multiple projects. For example, the "rand" crate we used in [Chapter 2](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) provides functionality that generates random numbers. Most of the time when Rustaceans say “crate”, they mean library crate, and they use “crate” interchangeably with the general programming concept of a “library".

The *crate root* is a source file that the Rust compiler starts from and makes up the root module of your crate (we’ll explain modules in depth in the [“Defining Modules to Control Scope and Privacy”](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html) section).

A *package* is a bundle of one or more crates that provides a set of functionality. A package contains a *Cargo.toml* file that describes how to build those crates. Cargo is actually a package that contains the binary crate for the command-line tool you’ve been using to build your code. The Cargo package also contains a library crate that the binary crate depends on. Other projects can depend on the Cargo library crate to use the same logic the Cargo command-line tool uses.

A package can contain as many binary crates as you like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.

Let’s walk through what happens when we create a package. First, we enter the command "cargo new":

```console
$ cargo new my-project
     Created binary (application) "my-project" package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

After we run "cargo new", we use "ls" to see what Cargo creates. In the project directory, there’s a *Cargo.toml* file, giving us a package. There’s also a *src* directory that contains *main.rs*. Open *Cargo.toml* in your text editor, and note there’s no mention of *src/main.rs*. Cargo follows a convention that *src/main.rs* is the crate root of a binary crate with the same name as the package. Likewise, Cargo knows that if the package directory contains *src/lib.rs*, the package contains a library crate with the same name as the package, and *src/lib.rs* is its crate root. Cargo passes the crate root files to "rustc" to build the library or binary.

Here, we have a package that only contains *src/main.rs*, meaning it only contains a binary crate named "my-project". If a package contains *src/main.rs* and *src/lib.rs*, it has two crates: a binary and a library, both with the same name as the package. A package can have multiple binary crates by placing files in the *src/bin* directory: each file will be a separate binary crate.







---







## [Defining Modules to Control Scope and Privacy](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#defining-modules-to-control-scope-and-privacy)

In this section, we’ll talk about modules and other parts of the module system, namely *paths* that allow you to name items; the "use" keyword that brings a path into scope; and the "pub" keyword to make items public. We’ll also discuss the "as" keyword, external packages, and the glob operator.

First, we’re going to start with a list of rules for easy reference when you’re organizing your code in the future. Then we’ll explain each of the rules in detail.

### [Modules Cheat Sheet](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet)

Here we provide a quick reference on how modules, paths, the "use" keyword, and the "pub" keyword work in the compiler, and how most developers organize their code. We’ll be going through examples of each of these rules throughout this chapter, but this is a great place to refer to as a reminder of how modules work.

- **Start from the crate root**: When compiling a crate, the compiler first looks in the crate root file (usually *src/lib.rs* for a library crate or *src/main.rs* for a binary crate) for code to compile.

- Declaring modules

  : In the crate root file, you can declare new modules; say, you declare a “garden” module with

   

  ```
  mod garden;
  ```

  . The compiler will look for the module’s code in these places:

  - Inline, within curly brackets that replace the semicolon following "mod garden"
  - In the file *src/garden.rs*
  - In the file *src/garden/mod.rs*

- Declaring submodules

  : In any file other than the crate root, you can declare submodules. For example, you might declare

   

  ```
  mod vegetables;
  ```

   

  in

   

  src/garden.rs

  . The compiler will look for the submodule’s code within the directory named for the parent module in these places:

  - Inline, directly following "mod vegetables", within curly brackets instead of the semicolon
  - In the file *src/garden/vegetables.rs*
  - In the file *src/garden/vegetables/mod.rs*

- **Paths to code in modules**: Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate, as long as the privacy rules allow, using the path to the code. For example, an "Asparagus" type in the garden vegetables module would be found at "crate::garden::vegetables::Asparagus".

- **Private vs public**: Code within a module is private from its parent modules by default. To make a module public, declare it with "pub mod" instead of "mod". To make items within a public module public as well, use "pub" before their declarations.

- **The "use" keyword**: Within a scope, the "use" keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to "crate::garden::vegetables::Asparagus", you can create a shortcut with "use crate::garden::vegetables::Asparagus;" and from then on you only need to write "Asparagus" to make use of that type in the scope.

Here we create a binary crate named "backyard" that illustrates these rules. The crate’s directory, also named "backyard", contains these files and directories:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

The crate root file in this case is *src/main.rs*, and it contains:

Filename: src/main.rs

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

The "pub mod garden;" line tells the compiler to include the code it finds in *src/garden.rs*, which is:

Filename: src/garden.rs

```rust
pub mod vegetables;
```

Here, "pub mod vegetables;" means the code in *src/garden/vegetables.rs* is included too. That code is:

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

Now let’s get into the details of these rules and demonstrate them in action!

### [Grouping Related Code in Modules](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#grouping-related-code-in-modules)

*Modules* let us organize code within a crate for readability and easy reuse. Modules also allow us to control the *privacy* of items, because code within a module is private by default. Private items are internal implementation details not available for outside use. We can choose to make modules and the items within them public, which exposes them to allow external code to use and depend on them.

As an example, let’s write a library crate that provides the functionality of a restaurant. We’ll define the signatures of functions but leave their bodies empty to concentrate on the organization of the code, rather than the implementation of a restaurant.

In the restaurant industry, some parts of a restaurant are referred to as *front of house* and others as *back of house*. Front of house is where customers are; this encompasses where the hosts seat customers, servers take orders and payment, and bartenders make drinks. Back of house is where the chefs and cooks work in the kitchen, dishwashers clean up, and managers do administrative work.

To structure our crate in this way, we can organize its functions into nested modules. Create a new library named "restaurant" by running "cargo new restaurant --lib"; then enter the code in Listing 7-1 into *src/lib.rs* to define some modules and function signatures. Here’s the front of house section:

Filename: src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

Listing 7-1: A "front_of_house" module containing other modules that then contain functions

We define a module with the "mod" keyword followed by the name of the module (in this case, "front_of_house"). The body of the module then goes inside curly brackets. Inside modules, we can place other modules, as in this case with the modules "hosting" and "serving". Modules can also hold definitions for other items, such as structs, enums, constants, traits, and—as in Listing 7-1—functions.

By using modules, we can group related definitions together and name why they’re related. Programmers using this code can navigate the code based on the groups rather than having to read through all the definitions, making it easier to find the definitions relevant to them. Programmers adding new functionality to this code would know where to place the code to keep the program organized.

Earlier, we mentioned that *src/main.rs* and *src/lib.rs* are called crate roots. The reason for their name is that the contents of either of these two files form a module named "crate" at the root of the crate’s module structure, known as the *module tree*.

Listing 7-2 shows the module tree for the structure in Listing 7-1.

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

Listing 7-2: The module tree for the code in Listing 7-1

This tree shows how some of the modules nest inside one another; for example, "hosting" nests inside "front_of_house". The tree also shows that some modules are *siblings* to each other, meaning they’re defined in the same module; "hosting" and "serving" are siblings defined within "front_of_house". If module A is contained inside module B, we say that module A is the *child* of module B and that module B is the *parent* of module A. Notice that the entire module tree is rooted under the implicit module named "crate".

The module tree might remind you of the filesystem’s directory tree on your computer; this is a very apt comparison! Just like directories in a filesystem, you use modules to organize your code. And just like files in a directory, we need a way to find our modules.





---





## [Paths for Referring to an Item in the Module Tree](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#paths-for-referring-to-an-item-in-the-module-tree)

To show Rust where to find an item in a module tree, we use a path in the same way we use a path when navigating a filesystem. To call a function, we need to know its path.

A path can take two forms:

- An *absolute path* is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal "crate".
- A *relative path* starts from the current module and uses "self", "super", or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons ("::").

Returning to Listing 7-1, say we want to call the "add_to_waitlist" function. This is the same as asking: what’s the path of the "add_to_waitlist" function? Listing 7-3 contains Listing 7-1 with some of the modules and functions removed.

We’ll show two ways to call the "add_to_waitlist" function from a new function "eat_at_restaurant" defined in the crate root. These paths are correct, but there’s another problem remaining that will prevent this example from compiling as-is. We’ll explain why in a bit.

The "eat_at_restaurant" function is part of our library crate’s public API, so we mark it with the "pub" keyword. In the [“Exposing Paths with the "pub" Keyword”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword) section, we’ll go into more detail about "pub".

Filename: src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Listing 7-3: Calling the "add_to_waitlist" function using absolute and relative paths

The first time we call the "add_to_waitlist" function in "eat_at_restaurant", we use an absolute path. The "add_to_waitlist" function is defined in the same crate as "eat_at_restaurant", which means we can use the "crate" keyword to start an absolute path. We then include each of the successive modules until we make our way to "add_to_waitlist". You can imagine a filesystem with the same structure: we’d specify the path "/front_of_house/hosting/add_to_waitlist" to run the "add_to_waitlist" program; using the "crate" name to start from the crate root is like using "/" to start from the filesystem root in your shell.

The second time we call "add_to_waitlist" in "eat_at_restaurant", we use a relative path. The path starts with "front_of_house", the name of the module defined at the same level of the module tree as "eat_at_restaurant". Here the filesystem equivalent would be using the path "front_of_house/hosting/add_to_waitlist". Starting with a module name means that the path is relative.

Choosing whether to use a relative or absolute path is a decision you’ll make based on your project, and depends on whether you’re more likely to move item definition code separately from or together with the code that uses the item. For example, if we move the "front_of_house" module and the "eat_at_restaurant" function into a module named "customer_experience", we’d need to update the absolute path to "add_to_waitlist", but the relative path would still be valid. However, if we moved the "eat_at_restaurant" function separately into a module named "dining", the absolute path to the "add_to_waitlist" call would stay the same, but the relative path would need to be updated. Our preference in general is to specify absolute paths because it’s more likely we’ll want to move code definitions and item calls independently of each other.

Let’s try to compile Listing 7-3 and find out why it won’t compile yet! The error we get is shown in Listing 7-4.

```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module "hosting" is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module "hosting" is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module "hosting" is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module "hosting" is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try "rustc --explain E0603".
error: could not compile "restaurant" due to 2 previous errors
```

Listing 7-4: Compiler errors from building the code in Listing 7-3

The error messages say that module "hosting" is private. In other words, we have the correct paths for the "hosting" module and the "add_to_waitlist" function, but Rust won’t let us use them because it doesn’t have access to the private sections. In Rust, all items (functions, methods, structs, enums, modules, and constants) are private to parent modules by default. If you want to make an item like a function or struct private, you put it in a module.

Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules. This is because child modules wrap and hide their implementation details, but the child modules can see the context in which they’re defined. To continue with our metaphor, think of the privacy rules as being like the back office of a restaurant: what goes on in there is private to restaurant customers, but office managers can see and do everything in the restaurant they operate.

Rust chose to have the module system function this way so that hiding inner implementation details is the default. That way, you know which parts of the inner code you can change without breaking outer code. However, Rust does give you the option to expose inner parts of child modules’ code to outer ancestor modules by using the "pub" keyword to make an item public.

### [Exposing Paths with the "pub" Keyword](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword)

Let’s return to the error in Listing 7-4 that told us the "hosting" module is private. We want the "eat_at_restaurant" function in the parent module to have access to the "add_to_waitlist" function in the child module, so we mark the "hosting" module with the "pub" keyword, as shown in Listing 7-5.

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Listing 7-5: Declaring the "hosting" module as "pub" to use it from "eat_at_restaurant"

Unfortunately, the code in Listing 7-5 still results in an error, as shown in Listing 7-6.

```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function "add_to_waitlist" is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^ private function
  |
note: the function "add_to_waitlist" is defined here
 --> src/lib.rs:3:9
  |
3 |         fn add_to_waitlist() {}
  |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function "add_to_waitlist" is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function "add_to_waitlist" is defined here
  --> src/lib.rs:3:9
   |
3  |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try "rustc --explain E0603".
error: could not compile "restaurant" due to 2 previous errors
```

Listing 7-6: Compiler errors from building the code in Listing 7-5

What happened? Adding the "pub" keyword in front of "mod hosting" makes the module public. With this change, if we can access "front_of_house", we can access "hosting". But the *contents* of "hosting" are still private; making the module public doesn’t make its contents public. The "pub" keyword on a module only lets code in its ancestor modules refer to it, not access its inner code. Because modules are containers, there’s not much we can do by only making the module public; we need to go further and choose to make one or more of the items within the module public as well.

The errors in Listing 7-6 say that the "add_to_waitlist" function is private. The privacy rules apply to structs, enums, functions, and methods as well as modules.

Let’s also make the "add_to_waitlist" function public by adding the "pub" keyword before its definition, as in Listing 7-7.

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Listing 7-7: Adding the "pub" keyword to "mod hosting" and "fn add_to_waitlist" lets us call the function from "eat_at_restaurant"

Now the code will compile! To see why adding the "pub" keyword lets us use these paths in "add_to_waitlist" with respect to the privacy rules, let’s look at the absolute and the relative paths.

In the absolute path, we start with "crate", the root of our crate’s module tree. The "front_of_house" module is defined in the crate root. While "front_of_house" isn’t public, because the "eat_at_restaurant" function is defined in the same module as "front_of_house" (that is, "eat_at_restaurant" and "front_of_house" are siblings), we can refer to "front_of_house" from "eat_at_restaurant". Next is the "hosting" module marked with "pub". We can access the parent module of "hosting", so we can access "hosting". Finally, the "add_to_waitlist" function is marked with "pub" and we can access its parent module, so this function call works!

In the relative path, the logic is the same as the absolute path except for the first step: rather than starting from the crate root, the path starts from "front_of_house". The "front_of_house" module is defined within the same module as "eat_at_restaurant", so the relative path starting from the module in which "eat_at_restaurant" is defined works. Then, because "hosting" and "add_to_waitlist" are marked with "pub", the rest of the path works, and this function call is valid!

If you plan on sharing your library crate so other projects can use your code, your public API is your contract with users of your crate that determines how they can interact with your code. There are many considerations around managing changes to your public API to make it easier for people to depend on your crate. These considerations are out of the scope of this book; if you’re interested in this topic, see [The Rust API Guidelines](https://rust-lang.github.io/api-guidelines/).

> #### [Best Practices for Packages with a Binary and a Library](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#best-practices-for-packages-with-a-binary-and-a-library)
>
> We mentioned a package can contain both a *src/main.rs* binary crate root as well as a *src/lib.rs* library crate root, and both crates will have the package name by default. Typically, packages with this pattern of containing both a library and a binary crate will have just enough code in the binary crate to start an executable that calls code with the library crate. This lets other projects benefit from the most functionality that the package provides, because the library crate’s code can be shared.
>
> The module tree should be defined in *src/lib.rs*. Then, any public items can be used in the binary crate by starting paths with the name of the package. The binary crate becomes a user of the library crate just like a completely external crate would use the library crate: it can only use the public API. This helps you design a good API; not only are you the author, you’re also a client!
>
> In [Chapter 12](https://doc.rust-lang.org/book/ch12-00-an-io-project.html), we’ll demonstrate this organizational practice with a command-line program that will contain both a binary crate and a library crate.

### [Starting Relative Paths with "super"](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#starting-relative-paths-with-super)

We can construct relative paths that begin in the parent module, rather than the current module or the crate root, by using "super" at the start of the path. This is like starting a filesystem path with the ".." syntax. Using "super" allows us to reference an item that we know is in the parent module, which can make rearranging the module tree easier when the module is closely related to the parent, but the parent might be moved elsewhere in the module tree someday.

Consider the code in Listing 7-8 that models the situation in which a chef fixes an incorrect order and personally brings it out to the customer. The function "fix_incorrect_order" defined in the "back_of_house" module calls the function "deliver_order" defined in the parent module by specifying the path to "deliver_order" starting with "super":

Filename: src/lib.rs

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

Listing 7-8: Calling a function using a relative path starting with "super"

The "fix_incorrect_order" function is in the "back_of_house" module, so we can use "super" to go to the parent module of "back_of_house", which in this case is "crate", the root. From there, we look for "deliver_order" and find it. Success! We think the "back_of_house" module and the "deliver_order" function are likely to stay in the same relationship to each other and get moved together should we decide to reorganize the crate’s module tree. Therefore, we used "super" so we’ll have fewer places to update code in the future if this code gets moved to a different module.

### [Making Structs and Enums Public](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#making-structs-and-enums-public)

We can also use "pub" to designate structs and enums as public, but there are a few details extra to the usage of "pub" with structs and enums. If we use "pub" before a struct definition, we make the struct public, but the struct’s fields will still be private. We can make each field public or not on a case-by-case basis. In Listing 7-9, we’ve defined a public "back_of_house::Breakfast" struct with a public "toast" field but a private "seasonal_fruit" field. This models the case in a restaurant where the customer can pick the type of bread that comes with a meal, but the chef decides which fruit accompanies the meal based on what’s in season and in stock. The available fruit changes quickly, so customers can’t choose the fruit or even see which fruit they’ll get.

Filename: src/lib.rs

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
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
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

Listing 7-9: A struct with some public fields and some private fields

Because the "toast" field in the "back_of_house::Breakfast" struct is public, in "eat_at_restaurant" we can write and read to the "toast" field using dot notation. Notice that we can’t use the "seasonal_fruit" field in "eat_at_restaurant" because "seasonal_fruit" is private. Try uncommenting the line modifying the "seasonal_fruit" field value to see what error you get!

Also, note that because "back_of_house::Breakfast" has a private field, the struct needs to provide a public associated function that constructs an instance of "Breakfast" (we’ve named it "summer" here). If "Breakfast" didn’t have such a function, we couldn’t create an instance of "Breakfast" in "eat_at_restaurant" because we couldn’t set the value of the private "seasonal_fruit" field in "eat_at_restaurant".

In contrast, if we make an enum public, all of its variants are then public. We only need the "pub" before the "enum" keyword, as shown in Listing 7-10.

Filename: src/lib.rs

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

Listing 7-10: Designating an enum as public makes all its variants public

Because we made the "Appetizer" enum public, we can use the "Soup" and "Salad" variants in "eat_at_restaurant".

Enums aren’t very useful unless their variants are public; it would be annoying to have to annotate all enum variants with "pub" in every case, so the default for enum variants is to be public. Structs are often useful without their fields being public, so struct fields follow the general rule of everything being private by default unless annotated with "pub".

There’s one more situation involving "pub" that we haven’t covered, and that is our last module system feature: the "use" keyword. We’ll cover "use" by itself first, and then we’ll show how to combine "pub" and "use".





---







## [Bringing Paths into Scope with the "use" Keyword](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#bringing-paths-into-scope-with-the-use-keyword)

Having to write out the paths to call functions can feel inconvenient and repetitive. In Listing 7-7, whether we chose the absolute or relative path to the "add_to_waitlist" function, every time we wanted to call "add_to_waitlist" we had to specify "front_of_house" and "hosting" too. Fortunately, there’s a way to simplify this process: we can create a shortcut to a path with the "use" keyword once, and then use the shorter name everywhere else in the scope.

In Listing 7-11, we bring the "crate::front_of_house::hosting" module into the scope of the "eat_at_restaurant" function so we only have to specify "hosting::add_to_waitlist" to call the "add_to_waitlist" function in "eat_at_restaurant".

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Listing 7-11: Bringing a module into scope with "use"

Adding "use" and a path in a scope is similar to creating a symbolic link in the filesystem. By adding "use crate::front_of_house::hosting" in the crate root, "hosting" is now a valid name in that scope, just as though the "hosting" module had been defined in the crate root. Paths brought into scope with "use" also check privacy, like any other paths.

Note that "use" only creates the shortcut for the particular scope in which the "use" occurs. Listing 7-12 moves the "eat_at_restaurant" function into a new child module named "customer", which is then a different scope than the "use" statement, so the function body won’t compile:

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

Listing 7-12: A "use" statement only applies in the scope it’s in

The compiler error shows that the shortcut no longer applies within the "customer" module:

```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
warning: unused import: "crate::front_of_house::hosting"
 --> src/lib.rs:7:5
  |
7 | use crate::front_of_house::hosting;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: "#[warn(unused_imports)]" on by default

error[E0433]: failed to resolve: use of undeclared crate or module "hosting"
  --> src/lib.rs:11:9
   |
11 |         hosting::add_to_waitlist();
   |         ^^^^^^^ use of undeclared crate or module "hosting"

For more information about this error, try "rustc --explain E0433".
warning: "restaurant" (lib) generated 1 warning
error: could not compile "restaurant" due to previous error; 1 warning emitted
```

Notice there’s also a warning that the "use" is no longer used in its scope! To fix this problem, move the "use" within the "customer" module too, or reference the shortcut in the parent module with "super::hosting" within the child "customer" module.

### [Creating Idiomatic "use" Paths](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths)

In Listing 7-11, you might have wondered why we specified "use crate::front_of_house::hosting" and then called "hosting::add_to_waitlist" in "eat_at_restaurant" rather than specifying the "use" path all the way out to the "add_to_waitlist" function to achieve the same result, as in Listing 7-13.

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

Listing 7-13: Bringing the "add_to_waitlist" function into scope with "use", which is unidiomatic

Although both Listing 7-11 and 7-13 accomplish the same task, Listing 7-11 is the idiomatic way to bring a function into scope with "use". Bringing the function’s parent module into scope with "use" means we have to specify the parent module when calling the function. Specifying the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path. The code in Listing 7-13 is unclear as to where "add_to_waitlist" is defined.

On the other hand, when bringing in structs, enums, and other items with "use", it’s idiomatic to specify the full path. Listing 7-14 shows the idiomatic way to bring the standard library’s "HashMap" struct into the scope of a binary crate.

Filename: src/main.rs

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

Listing 7-14: Bringing "HashMap" into scope in an idiomatic way

There’s no strong reason behind this idiom: it’s just the convention that has emerged, and folks have gotten used to reading and writing Rust code this way.

The exception to this idiom is if we’re bringing two items with the same name into scope with "use" statements, because Rust doesn’t allow that. Listing 7-15 shows how to bring two "Result" types into scope that have the same name but different parent modules and how to refer to them.

Filename: src/lib.rs

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

Listing 7-15: Bringing two types with the same name into the same scope requires using their parent modules.

As you can see, using the parent modules distinguishes the two "Result" types. If instead we specified "use std::fmt::Result" and "use std::io::Result", we’d have two "Result" types in the same scope and Rust wouldn’t know which one we meant when we used "Result".

### [Providing New Names with the "as" Keyword](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#providing-new-names-with-the-as-keyword)

There’s another solution to the problem of bringing two types of the same name into the same scope with "use": after the path, we can specify "as" and a new local name, or *alias*, for the type. Listing 7-16 shows another way to write the code in Listing 7-15 by renaming one of the two "Result" types using "as".

Filename: src/lib.rs

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

Listing 7-16: Renaming a type when it’s brought into scope with the "as" keyword

In the second "use" statement, we chose the new name "IoResult" for the "std::io::Result" type, which won’t conflict with the "Result" from "std::fmt" that we’ve also brought into scope. Listing 7-15 and Listing 7-16 are considered idiomatic, so the choice is up to you!

### [Re-exporting Names with "pub use"](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#re-exporting-names-with-pub-use)

When we bring a name into scope with the "use" keyword, the name available in the new scope is private. To enable the code that calls our code to refer to that name as if it had been defined in that code’s scope, we can combine "pub" and "use". This technique is called *re-exporting* because we’re bringing an item into scope but also making that item available for others to bring into their scope.

Listing 7-17 shows the code in Listing 7-11 with "use" in the root module changed to "pub use".

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Listing 7-17: Making a name available for any code to use from a new scope with "pub use"

Before this change, external code would have to call the "add_to_waitlist" function by using the path "restaurant::front_of_house::hosting::add_to_waitlist()". Now that this "pub use" has re-exported the "hosting" module from the root module, external code can now use the path "restaurant::hosting::add_to_waitlist()" instead.

Re-exporting is useful when the internal structure of your code is different from how programmers calling your code would think about the domain. For example, in this restaurant metaphor, the people running the restaurant think about “front of house” and “back of house.” But customers visiting a restaurant probably won’t think about the parts of the restaurant in those terms. With "pub use", we can write our code with one structure but expose a different structure. Doing so makes our library well organized for programmers working on the library and programmers calling the library. We’ll look at another example of "pub use" and how it affects your crate’s documentation in the [“Exporting a Convenient Public API with "pub use"”](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use) section of Chapter 14.

### [Using External Packages](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-external-packages)

In Chapter 2, we programmed a guessing game project that used an external package called "rand" to get random numbers. To use "rand" in our project, we added this line to *Cargo.toml*:

Filename: Cargo.toml

```toml
rand = "0.8.5"
```

Adding "rand" as a dependency in *Cargo.toml* tells Cargo to download the "rand" package and any dependencies from [crates.io](https://crates.io/) and make "rand" available to our project.

Then, to bring "rand" definitions into the scope of our package, we added a "use" line starting with the name of the crate, "rand", and listed the items we wanted to bring into scope. Recall that in the [“Generating a Random Number”](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) section in Chapter 2, we brought the "Rng" trait into scope and called the "rand::thread_rng" function:

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

Members of the Rust community have made many packages available at [crates.io](https://crates.io/), and pulling any of them into your package involves these same steps: listing them in your package’s *Cargo.toml* file and using "use" to bring items from their crates into scope.

Note that the standard "std" library is also a crate that’s external to our package. Because the standard library is shipped with the Rust language, we don’t need to change *Cargo.toml* to include "std". But we do need to refer to it with "use" to bring items from there into our package’s scope. For example, with "HashMap" we would use this line:

```rust
use std::collections::HashMap;
```

This is an absolute path starting with "std", the name of the standard library crate.

### [Using Nested Paths to Clean Up Large "use" Lists](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-nested-paths-to-clean-up-large-use-lists)

If we’re using multiple items defined in the same crate or same module, listing each item on its own line can take up a lot of vertical space in our files. For example, these two "use" statements we had in the Guessing Game in Listing 2-4 bring items from "std" into scope:

Filename: src/main.rs

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--
```

Instead, we can use nested paths to bring the same items into scope in one line. We do this by specifying the common part of the path, followed by two colons, and then curly brackets around a list of the parts of the paths that differ, as shown in Listing 7-18.

Filename: src/main.rs

```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

Listing 7-18: Specifying a nested path to bring multiple items with the same prefix into scope

In bigger programs, bringing many items into scope from the same crate or module using nested paths can reduce the number of separate "use" statements needed by a lot!

We can use a nested path at any level in a path, which is useful when combining two "use" statements that share a subpath. For example, Listing 7-19 shows two "use" statements: one that brings "std::io" into scope and one that brings "std::io::Write" into scope.

Filename: src/lib.rs

```rust
use std::io;
use std::io::Write;
```

Listing 7-19: Two "use" statements where one is a subpath of the other

The common part of these two paths is "std::io", and that’s the complete first path. To merge these two paths into one "use" statement, we can use "self" in the nested path, as shown in Listing 7-20.

Filename: src/lib.rs

```rust
use std::io::{self, Write};
```

Listing 7-20: Combining the paths in Listing 7-19 into one "use" statement

This line brings "std::io" and "std::io::Write" into scope.

### [The Glob Operator](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#the-glob-operator)

If we want to bring *all* public items defined in a path into scope, we can specify that path followed by the "*" glob operator:

```rust
use std::collections::*;
```

This "use" statement brings all public items defined in "std::collections" into the current scope. Be careful when using the glob operator! Glob can make it harder to tell what names are in scope and where a name used in your program was defined.

The glob operator is often used when testing to bring everything under test into the "tests" module; we’ll talk about that in the [“How to Write Tests”](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#how-to-write-tests) section in Chapter 11. The glob operator is also sometimes used as part of the prelude pattern: see [the standard library documentation](https://doc.rust-lang.org/std/prelude/index.html#other-preludes) for more information on that pattern.





---









## [Separating Modules into Different Files](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#separating-modules-into-different-files)

So far, all the examples in this chapter defined multiple modules in one file. When modules get large, you might want to move their definitions to a separate file to make the code easier to navigate.

For example, let’s start from the code in Listing 7-17 that had multiple restaurant modules. We’ll extract modules into files instead of having all the modules defined in the crate root file. In this case, the crate root file is *src/lib.rs*, but this procedure also works with binary crates whose crate root file is *src/main.rs*.

First, we’ll extract the "front_of_house" module to its own file. Remove the code inside the curly brackets for the "front_of_house" module, leaving only the "mod front_of_house;" declaration, so that *src/lib.rs* contains the code shown in Listing 7-21. Note that this won’t compile until we create the *src/front_of_house.rs* file in Listing 7-22.

Filename: src/lib.rs

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Listing 7-21: Declaring the "front_of_house" module whose body will be in *src/front_of_house.rs*

Next, place the code that was in the curly brackets into a new file named *src/front_of_house.rs*, as shown in Listing 7-22. The compiler knows to look in this file because it came across the module declaration in the crate root with the name "front_of_house".

Filename: src/front_of_house.rs

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

Listing 7-22: Definitions inside the "front_of_house" module in *src/front_of_house.rs*

Note that you only need to load a file using a "mod" declaration *once* in your module tree. Once the compiler knows the file is part of the project (and knows where in the module tree the code resides because of where you’ve put the "mod" statement), other files in your project should refer to the loaded file’s code using a path to where it was declared, as covered in the [“Paths for Referring to an Item in the Module Tree”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) section. In other words, "mod" is *not* an “include” operation that you may have seen in other programming languages.

Next, we’ll extract the "hosting" module to its own file. The process is a bit different because "hosting" is a child module of "front_of_house", not of the root module. We’ll place the file for "hosting" in a new directory that will be named for its ancestors in the module tree, in this case *src/front_of_house/*.

To start moving "hosting", we change *src/front_of_house.rs* to contain only the declaration of the "hosting" module:

Filename: src/front_of_house.rs

```rust
pub mod hosting;
```

Then we create a *src/front_of_house* directory and a file *hosting.rs* to contain the definitions made in the "hosting" module:

Filename: src/front_of_house/hosting.rs

```rust
pub fn add_to_waitlist() {}
```

If we instead put *hosting.rs* in the *src* directory, the compiler would expect the *hosting.rs* code to be in a "hosting" module declared in the crate root, and not declared as a child of the "front_of_house" module. The compiler’s rules for which files to check for which modules’ code means the directories and files more closely match the module tree.

> ### [Alternate File Paths](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths)
>
> So far we’ve covered the most idiomatic file paths the Rust compiler uses, but Rust also supports an older style of file path. For a module named "front_of_house" declared in the crate root, the compiler will look for the module’s code in:
>
> - *src/front_of_house.rs* (what we covered)
> - *src/front_of_house/mod.rs* (older style, still supported path)
>
> For a module named "hosting" that is a submodule of "front_of_house", the compiler will look for the module’s code in:
>
> - *src/front_of_house/hosting.rs* (what we covered)
> - *src/front_of_house/hosting/mod.rs* (older style, still supported path)
>
> If you use both styles for the same module, you’ll get a compiler error. Using a mix of both styles for different modules in the same project is allowed, but might be confusing for people navigating your project.
>
> The main downside to the style that uses files named *mod.rs* is that your project can end up with many files named *mod.rs*, which can get confusing when you have them open in your editor at the same time.

We’ve moved each module’s code to a separate file, and the module tree remains the same. The function calls in "eat_at_restaurant" will work without any modification, even though the definitions live in different files. This technique lets you move modules to new files as they grow in size.

Note that the "pub use crate::front_of_house::hosting" statement in *src/lib.rs* also hasn’t changed, nor does "use" have any impact on what files are compiled as part of the crate. The "mod" keyword declares modules, and Rust looks in a file with the same name as the module for the code that goes into that module.

## [Summary](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#summary)

Rust lets you split a package into multiple crates and a crate into modules so you can refer to items defined in one module from another module. You can do this by specifying absolute or relative paths. These paths can be brought into scope with a "use" statement so you can use a shorter path for multiple uses of the item in that scope. Module code is private by default, but you can make definitions public by adding the "pub" keyword.

In the next chapter, we’ll look at some collection data structures in the standard library that you can use in your neatly organized code.





---





# [Common Collections](https://doc.rust-lang.org/book/ch08-00-common-collections.html#common-collections)

Rust’s standard library includes a number of very useful data structures called *collections*. Most other data types represent one specific value, but collections can contain multiple values. Unlike the built-in array and tuple types, the data these collections point to is stored on the heap, which means the amount of data does not need to be known at compile time and can grow or shrink as the program runs. Each kind of collection has different capabilities and costs, and choosing an appropriate one for your current situation is a skill you’ll develop over time. In this chapter, we’ll discuss three collections that are used very often in Rust programs:

- A *vector* allows you to store a variable number of values next to each other.
- A *string* is a collection of characters. We’ve mentioned the "String" type previously, but in this chapter we’ll talk about it in depth.
- A *hash map* allows you to associate a value with a particular key. It’s a particular implementation of the more general data structure called a *map*.

To learn about the other kinds of collections provided by the standard library, see [the documentation](https://doc.rust-lang.org/std/collections/index.html).

We’ll discuss how to create and update vectors, strings, and hash maps, as well as what makes each special.





---





## [Storing Lists of Values with Vectors](https://doc.rust-lang.org/book/ch08-01-vectors.html#storing-lists-of-values-with-vectors)

The first collection type we’ll look at is "Vec<T>", also known as a *vector*. Vectors allow you to store more than one value in a single data structure that puts all the values next to each other in memory. Vectors can only store values of the same type. They are useful when you have a list of items, such as the lines of text in a file or the prices of items in a shopping cart.

### [Creating a New Vector](https://doc.rust-lang.org/book/ch08-01-vectors.html#creating-a-new-vector)

To create a new empty vector, we call the "Vec::new" function, as shown in Listing 8-1.

```rust
    let v: Vec<i32> = Vec::new();
```

Listing 8-1: Creating a new, empty vector to hold values of type "i32"

Note that we added a type annotation here. Because we aren’t inserting any values into this vector, Rust doesn’t know what kind of elements we intend to store. This is an important point. Vectors are implemented using generics; we’ll cover how to use generics with your own types in Chapter 10. For now, know that the "Vec<T>" type provided by the standard library can hold any type. When we create a vector to hold a specific type, we can specify the type within angle brackets. In Listing 8-1, we’ve told Rust that the "Vec<T>" in "v" will hold elements of the "i32" type.

More often, you’ll create a "Vec<T>" with initial values and Rust will infer the type of value you want to store, so you rarely need to do this type annotation. Rust conveniently provides the "vec!" macro, which will create a new vector that holds the values you give it. Listing 8-2 creates a new "Vec<i32>" that holds the values "1", "2", and "3". The integer type is "i32" because that’s the default integer type, as we discussed in the [“Data Types”](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) section of Chapter 3.

```rust
    let v = vec![1, 2, 3];
```

Listing 8-2: Creating a new vector containing values

Because we’ve given initial "i32" values, Rust can infer that the type of "v" is "Vec<i32>", and the type annotation isn’t necessary. Next, we’ll look at how to modify a vector.

### [Updating a Vector](https://doc.rust-lang.org/book/ch08-01-vectors.html#updating-a-vector)

To create a vector and then add elements to it, we can use the "push" method, as shown in Listing 8-3.

```rust
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
```

Listing 8-3: Using the "push" method to add values to a vector

As with any variable, if we want to be able to change its value, we need to make it mutable using the "mut" keyword, as discussed in Chapter 3. The numbers we place inside are all of type "i32", and Rust infers this from the data, so we don’t need the "Vec<i32>" annotation.

### [Reading Elements of Vectors](https://doc.rust-lang.org/book/ch08-01-vectors.html#reading-elements-of-vectors)

There are two ways to reference a value stored in a vector: via indexing or using the "get" method. In the following examples, we’ve annotated the types of the values that are returned from these functions for extra clarity.

Listing 8-4 shows both methods of accessing a value in a vector, with indexing syntax and the "get" method.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
```

Listing 8-4: Using indexing syntax or the "get" method to access an item in a vector

Note a few details here. We use the index value of "2" to get the third element because vectors are indexed by number, starting at zero. Using "&" and "[]" gives us a reference to the element at the index value. When we use the "get" method with the index passed as an argument, we get an "Option<&T>" that we can use with "match".

The reason Rust provides these two ways to reference an element is so you can choose how the program behaves when you try to use an index value outside the range of existing elements. As an example, let’s see what happens when we have a vector of five elements and then we try to access an element at index 100 with each technique, as shown in Listing 8-5.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
```

Listing 8-5: Attempting to access the element at index 100 in a vector containing five elements

When we run this code, the first "[]" method will cause the program to panic because it references a nonexistent element. This method is best used when you want your program to crash if there’s an attempt to access an element past the end of the vector.

When the "get" method is passed an index that is outside the vector, it returns "None" without panicking. You would use this method if accessing an element beyond the range of the vector may happen occasionally under normal circumstances. Your code will then have logic to handle having either "Some(&element)" or "None", as discussed in Chapter 6. For example, the index could be coming from a person entering a number. If they accidentally enter a number that’s too large and the program gets a "None" value, you could tell the user how many items are in the current vector and give them another chance to enter a valid value. That would be more user-friendly than crashing the program due to a typo!

When the program has a valid reference, the borrow checker enforces the ownership and borrowing rules (covered in Chapter 4) to ensure this reference and any other references to the contents of the vector remain valid. Recall the rule that states you can’t have mutable and immutable references in the same scope. That rule applies in Listing 8-6, where we hold an immutable reference to the first element in a vector and try to add an element to the end. This program won’t work if we also try to refer to that element later in the function:

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
```

Listing 8-6: Attempting to add an element to a vector while holding a reference to an item

Compiling this code will result in this error:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow "v" as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {first}");
  |                                      ----- immutable borrow later used here

For more information about this error, try "rustc --explain E0502".
error: could not compile "collections" due to previous error
```

The code in Listing 8-6 might look like it should work: why should a reference to the first element care about changes at the end of the vector? This error is due to the way vectors work: because vectors put the values next to each other in memory, adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space, if there isn’t enough room to put all the elements next to each other where the vector is currently stored. In that case, the reference to the first element would be pointing to deallocated memory. The borrowing rules prevent programs from ending up in that situation.

> Note: For more on the implementation details of the "Vec<T>" type, see [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/vec/vec.html).

### [Iterating over the Values in a Vector](https://doc.rust-lang.org/book/ch08-01-vectors.html#iterating-over-the-values-in-a-vector)

To access each element in a vector in turn, we would iterate through all of the elements rather than use indices to access one at a time. Listing 8-7 shows how to use a "for" loop to get immutable references to each element in a vector of "i32" values and print them.

```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
```

Listing 8-7: Printing each element in a vector by iterating over the elements using a "for" loop

We can also iterate over mutable references to each element in a mutable vector in order to make changes to all the elements. The "for" loop in Listing 8-8 will add "50" to each element.

```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
```

Listing 8-8: Iterating over mutable references to elements in a vector

To change the value that the mutable reference refers to, we have to use the "*" dereference operator to get to the value in "i" before we can use the "+=" operator. We’ll talk more about the dereference operator in the [“Following the Pointer to the Value with the Dereference Operator”](https://doc.rust-lang.org/book/ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator) section of Chapter 15.

Iterating over a vector, whether immutably or mutably, is safe because of the borrow checker's rules. If we attempted to insert or remove items in the "for" loop bodies in Listing 8-7 and Listing 8-8, we would get a compiler error similar to the one we got with the code in Listing 8-6. The reference to the vector that the "for" loop holds prevents simultaneous modification of the whole vector.

### [Using an Enum to Store Multiple Types](https://doc.rust-lang.org/book/ch08-01-vectors.html#using-an-enum-to-store-multiple-types)

Vectors can only store values that are the same type. This can be inconvenient; there are definitely use cases for needing to store a list of items of different types. Fortunately, the variants of an enum are defined under the same enum type, so when we need one type to represent elements of different types, we can define and use an enum!

For example, say we want to get values from a row in a spreadsheet in which some of the columns in the row contain integers, some floating-point numbers, and some strings. We can define an enum whose variants will hold the different value types, and all the enum variants will be considered the same type: that of the enum. Then we can create a vector to hold that enum and so, ultimately, holds different types. We’ve demonstrated this in Listing 8-9.

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

Listing 8-9: Defining an "enum" to store values of different types in one vector

Rust needs to know what types will be in the vector at compile time so it knows exactly how much memory on the heap will be needed to store each element. We must also be explicit about what types are allowed in this vector. If Rust allowed a vector to hold any type, there would be a chance that one or more of the types would cause errors with the operations performed on the elements of the vector. Using an enum plus a "match" expression means that Rust will ensure at compile time that every possible case is handled, as discussed in Chapter 6.

If you don’t know the exhaustive set of types a program will get at runtime to store in a vector, the enum technique won’t work. Instead, you can use a trait object, which we’ll cover in Chapter 17.

Now that we’ve discussed some of the most common ways to use vectors, be sure to review [the API documentation](https://doc.rust-lang.org/std/vec/struct.Vec.html) for all the many useful methods defined on "Vec<T>" by the standard library. For example, in addition to "push", a "pop" method removes and returns the last element.

### [Dropping a Vector Drops Its Elements](https://doc.rust-lang.org/book/ch08-01-vectors.html#dropping-a-vector-drops-its-elements)

Like any other "struct", a vector is freed when it goes out of scope, as annotated in Listing 8-10.

```rust
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
```

Listing 8-10: Showing where the vector and its elements are dropped

When the vector gets dropped, all of its contents are also dropped, meaning the integers it holds will be cleaned up. The borrow checker ensures that any references to contents of a vector are only used while the vector itself is valid.

Let’s move on to the next collection type: "String"!





---





## [Storing UTF-8 Encoded Text with Strings](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings)

We talked about strings in Chapter 4, but we’ll look at them in more depth now. New Rustaceans commonly get stuck on strings for a combination of three reasons: Rust’s propensity for exposing possible errors, strings being a more complicated data structure than many programmers give them credit for, and UTF-8. These factors combine in a way that can seem difficult when you’re coming from other programming languages.

We discuss strings in the context of collections because strings are implemented as a collection of bytes, plus some methods to provide useful functionality when those bytes are interpreted as text. In this section, we’ll talk about the operations on "String" that every collection type has, such as creating, updating, and reading. We’ll also discuss the ways in which "String" is different from the other collections, namely how indexing into a "String" is complicated by the differences between how people and computers interpret "String" data.

### [What Is a String?](https://doc.rust-lang.org/book/ch08-02-strings.html#what-is-a-string)

We’ll first define what we mean by the term *string*. Rust has only one string type in the core language, which is the string slice "str" that is usually seen in its borrowed form "&str". In Chapter 4, we talked about *string slices*, which are references to some UTF-8 encoded string data stored elsewhere. String literals, for example, are stored in the program’s binary and are therefore string slices.

The "String" type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type. When Rustaceans refer to “strings” in Rust, they might be referring to either the "String" or the string slice "&str" types, not just one of those types. Although this section is largely about "String", both types are used heavily in Rust’s standard library, and both "String" and string slices are UTF-8 encoded.

### [Creating a New String](https://doc.rust-lang.org/book/ch08-02-strings.html#creating-a-new-string)

Many of the same operations available with "Vec<T>" are available with "String" as well, because "String" is actually implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities. An example of a function that works the same way with "Vec<T>" and "String" is the "new" function to create an instance, shown in Listing 8-11.

```rust
    let mut s = String::new();
```

Listing 8-11: Creating a new, empty "String"

This line creates a new empty string called "s", which we can then load data into. Often, we’ll have some initial data that we want to start the string with. For that, we use the "to_string" method, which is available on any type that implements the "Display" trait, as string literals do. Listing 8-12 shows two examples.

```rust
    let data = "initial contents";

    let s = data.to_string();

    // the method also works on a literal directly:
    let s = "initial contents".to_string();
```

Listing 8-12: Using the "to_string" method to create a "String" from a string literal

This code creates a string containing "initial contents".

We can also use the function "String::from" to create a "String" from a string literal. The code in Listing 8-13 is equivalent to the code from Listing 8-12 that uses "to_string".

```rust
    let s = String::from("initial contents");
```

Listing 8-13: Using the "String::from" function to create a "String" from a string literal

Because strings are used for so many things, we can use many different generic APIs for strings, providing us with a lot of options. Some of them can seem redundant, but they all have their place! In this case, "String::from" and "to_string" do the same thing, so which you choose is a matter of style and readability.

Remember that strings are UTF-8 encoded, so we can include any properly encoded data in them, as shown in Listing 8-14.

```rust
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
```

Listing 8-14: Storing greetings in different languages in strings

All of these are valid "String" values.

### [Updating a String](https://doc.rust-lang.org/book/ch08-02-strings.html#updating-a-string)

A "String" can grow in size and its contents can change, just like the contents of a "Vec<T>", if you push more data into it. In addition, you can conveniently use the "+" operator or the "format!" macro to concatenate "String" values.

#### [Appending to a String with "push_str" and "push"](https://doc.rust-lang.org/book/ch08-02-strings.html#appending-to-a-string-with-push_str-and-push)

We can grow a "String" by using the "push_str" method to append a string slice, as shown in Listing 8-15.

```rust
    let mut s = String::from("foo");
    s.push_str("bar");
```

Listing 8-15: Appending a string slice to a "String" using the "push_str" method

After these two lines, "s" will contain "foobar". The "push_str" method takes a string slice because we don’t necessarily want to take ownership of the parameter. For example, in the code in Listing 8-16, we want to be able to use "s2" after appending its contents to "s1".

```rust
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
```

Listing 8-16: Using a string slice after appending its contents to a "String"

If the "push_str" method took ownership of "s2", we wouldn’t be able to print its value on the last line. However, this code works as we’d expect!

The "push" method takes a single character as a parameter and adds it to the "String". Listing 8-17 adds the letter “l” to a "String" using the "push" method.

```rust
    let mut s = String::from("lo");
    s.push('l');
```

Listing 8-17: Adding one character to a "String" value using "push"

As a result, "s" will contain "lol".

#### [Concatenation with the "+" Operator or the "format!" Macro](https://doc.rust-lang.org/book/ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro)

Often, you’ll want to combine two existing strings. One way to do so is to use the "+" operator, as shown in Listing 8-18.

```rust
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

Listing 8-18: Using the "+" operator to combine two "String" values into a new "String" value

The string "s3" will contain "Hello, world!". The reason "s1" is no longer valid after the addition, and the reason we used a reference to "s2", has to do with the signature of the method that’s called when we use the "+" operator. The "+" operator uses the "add" method, whose signature looks something like this:

```rust
fn add(self, s: &str) -> String {
```

In the standard library, you'll see "add" defined using generics and associated types. Here, we’ve substituted in concrete types, which is what happens when we call this method with "String" values. We’ll discuss generics in Chapter 10. This signature gives us the clues we need to understand the tricky bits of the "+" operator.

First, "s2" has an "&", meaning that we’re adding a *reference* of the second string to the first string. This is because of the "s" parameter in the "add" function: we can only add a "&str" to a "String"; we can’t add two "String" values together. But wait—the type of "&s2" is "&String", not "&str", as specified in the second parameter to "add". So why does Listing 8-18 compile?

The reason we’re able to use "&s2" in the call to "add" is that the compiler can *coerce* the "&String" argument into a "&str". When we call the "add" method, Rust uses a *deref coercion*, which here turns "&s2" into "&s2[..]". We’ll discuss deref coercion in more depth in Chapter 15. Because "add" does not take ownership of the "s" parameter, "s2" will still be a valid "String" after this operation.

Second, we can see in the signature that "add" takes ownership of "self", because "self" does *not* have an "&". This means "s1" in Listing 8-18 will be moved into the "add" call and will no longer be valid after that. So although "let s3 = s1 + &s2;" looks like it will copy both strings and create a new one, this statement actually takes ownership of "s1", appends a copy of the contents of "s2", and then returns ownership of the result. In other words, it looks like it’s making a lot of copies but isn’t; the implementation is more efficient than copying.

If we need to concatenate multiple strings, the behavior of the "+" operator gets unwieldy:

```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
```

At this point, "s" will be "tic-tac-toe". With all of the "+" and ``` characters, it’s difficult to see what’s going on. For more complicated string combining, we can instead use the "format!" macro:

```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
```

This code also sets "s" to "tic-tac-toe". The "format!" macro works like "println!", but instead of printing the output to the screen, it returns a "String" with the contents. The version of the code using "format!" is much easier to read, and the code generated by the "format!" macro uses references so that this call doesn’t take ownership of any of its parameters.

### [Indexing into Strings](https://doc.rust-lang.org/book/ch08-02-strings.html#indexing-into-strings)

In many other programming languages, accessing individual characters in a string by referencing them by index is a valid and common operation. However, if you try to access parts of a "String" using indexing syntax in Rust, you’ll get an error. Consider the invalid code in Listing 8-19.

```rust
    let s1 = String::from("hello");
    let h = s1[0];
```

Listing 8-19: Attempting to use indexing syntax with a String

This code will result in the following error:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type "String" cannot be indexed by "{integer}"
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ "String" cannot be indexed by "{integer}"
  |
  = help: the trait "Index<{integer}>" is not implemented for "String"
  = help: the following other types implement trait "Index<Idx>":
            <String as Index<RangeFrom<usize>>>
            <String as Index<RangeFull>>
            <String as Index<RangeInclusive<usize>>>
            <String as Index<RangeTo<usize>>>
            <String as Index<RangeToInclusive<usize>>>
            <String as Index<std::ops::Range<usize>>>

For more information about this error, try "rustc --explain E0277".
error: could not compile "collections" due to previous error
```

The error and the note tell the story: Rust strings don’t support indexing. But why not? To answer that question, we need to discuss how Rust stores strings in memory.

#### [Internal Representation](https://doc.rust-lang.org/book/ch08-02-strings.html#internal-representation)

A "String" is a wrapper over a "Vec<u8>". Let’s look at some of our properly encoded UTF-8 example strings from Listing 8-14. First, this one:

```rust
    let hello = String::from("Hola");
```

In this case, "len" will be 4, which means the vector storing the string “Hola” is 4 bytes long. Each of these letters takes 1 byte when encoded in UTF-8. The following line, however, may surprise you. (Note that this string begins with the capital Cyrillic letter Ze, not the Arabic number 3.)

```rust
    let hello = String::from("Здравствуйте");
```

Asked how long the string is, you might say 12. In fact, Rust’s answer is 24: that’s the number of bytes it takes to encode “Здравствуйте” in UTF-8, because each Unicode scalar value in that string takes 2 bytes of storage. Therefore, an index into the string’s bytes will not always correlate to a valid Unicode scalar value. To demonstrate, consider this invalid Rust code:

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

You already know that "answer" will not be "З", the first letter. When encoded in UTF-8, the first byte of "З" is "208" and the second is "151", so it would seem that "answer" should in fact be "208", but "208" is not a valid character on its own. Returning "208" is likely not what a user would want if they asked for the first letter of this string; however, that’s the only data that Rust has at byte index 0. Users generally don’t want the byte value returned, even if the string contains only Latin letters: if "&"hello"[0]" were valid code that returned the byte value, it would return "104", not "h".

The answer, then, is that to avoid returning an unexpected value and causing bugs that might not be discovered immediately, Rust doesn’t compile this code at all and prevents misunderstandings early in the development process.

#### [Bytes and Scalar Values and Grapheme Clusters! Oh My!](https://doc.rust-lang.org/book/ch08-02-strings.html#bytes-and-scalar-values-and-grapheme-clusters-oh-my)

Another point about UTF-8 is that there are actually three relevant ways to look at strings from Rust’s perspective: as bytes, scalar values, and grapheme clusters (the closest thing to what we would call *letters*).

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is stored as a vector of "u8" values that looks like this:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

That’s 18 bytes and is how computers ultimately store this data. If we look at them as Unicode scalar values, which are what Rust’s "char" type is, those bytes look like this:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

There are six "char" values here, but the fourth and sixth are not letters: they’re diacritics that don’t make sense on their own. Finally, if we look at them as grapheme clusters, we’d get what a person would call the four letters that make up the Hindi word:

```text
["न", "म", "स्", "ते"]
```

Rust provides different ways of interpreting the raw string data that computers store so that each program can choose the interpretation it needs, no matter what human language the data is in.

A final reason Rust doesn’t allow us to index into a "String" to get a character is that indexing operations are expected to always take constant time (O(1)). But it isn’t possible to guarantee that performance with a "String", because Rust would have to walk through the contents from the beginning to the index to determine how many valid characters there were.

### [Slicing Strings](https://doc.rust-lang.org/book/ch08-02-strings.html#slicing-strings)

Indexing into a string is often a bad idea because it’s not clear what the return type of the string-indexing operation should be: a byte value, a character, a grapheme cluster, or a string slice. If you really need to use indices to create string slices, therefore, Rust asks you to be more specific.

Rather than indexing using "[]" with a single number, you can use "[]" with a range to create a string slice containing particular bytes:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Here, "s" will be a "&str" that contains the first 4 bytes of the string. Earlier, we mentioned that each of these characters was 2 bytes, which means "s" will be "Зд".

If we were to try to slice only part of a character’s bytes with something like "&hello[0..1]", Rust would panic at runtime in the same way as if an invalid index were accessed in a vector:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running "target/debug/collections"
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of "Здравствуйте"', src/main.rs:4:14
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

You should use ranges to create string slices with caution, because doing so can crash your program.

### [Methods for Iterating Over Strings](https://doc.rust-lang.org/book/ch08-02-strings.html#methods-for-iterating-over-strings)

The best way to operate on pieces of strings is to be explicit about whether you want characters or bytes. For individual Unicode scalar values, use the "chars" method. Calling "chars" on “Зд” separates out and returns two values of type "char", and you can iterate over the result to access each element:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

This code will print the following:

```text
З
д
```

Alternatively, the "bytes" method returns each raw byte, which might be appropriate for your domain:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

This code will print the four bytes that make up this string:

```text
208
151
208
180
```

But be sure to remember that valid Unicode scalar values may be made up of more than 1 byte.

Getting grapheme clusters from strings as with the Devanagari script is complex, so this functionality is not provided by the standard library. Crates are available on [crates.io](https://crates.io/) if this is the functionality you need.

### [Strings Are Not So Simple](https://doc.rust-lang.org/book/ch08-02-strings.html#strings-are-not-so-simple)

To summarize, strings are complicated. Different programming languages make different choices about how to present this complexity to the programmer. Rust has chosen to make the correct handling of "String" data the default behavior for all Rust programs, which means programmers have to put more thought into handling UTF-8 data upfront. This trade-off exposes more of the complexity of strings than is apparent in other programming languages, but it prevents you from having to handle errors involving non-ASCII characters later in your development life cycle.

The good news is that the standard library offers a lot of functionality built off the "String" and "&str" types to help handle these complex situations correctly. Be sure to check out the documentation for useful methods like "contains" for searching in a string and "replace" for substituting parts of a string with another string.

Let’s switch to something a bit less complex: hash maps!





---





## [Storing Keys with Associated Values in Hash Maps](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#storing-keys-with-associated-values-in-hash-maps)

The last of our common collections is the *hash map*. The type "HashMap<K, V>" stores a mapping of keys of type "K" to values of type "V" using a *hashing function*, which determines how it places these keys and values into memory. Many programming languages support this kind of data structure, but they often use a different name, such as hash, map, object, hash table, dictionary, or associative array, just to name a few.

Hash maps are useful when you want to look up data not by using an index, as you can with vectors, but by using a key that can be of any type. For example, in a game, you could keep track of each team’s score in a hash map in which each key is a team’s name and the values are each team’s score. Given a team name, you can retrieve its score.

We’ll go over the basic API of hash maps in this section, but many more goodies are hiding in the functions defined on "HashMap<K, V>" by the standard library. As always, check the standard library documentation for more information.

### [Creating a New Hash Map](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#creating-a-new-hash-map)

One way to create an empty hash map is using "new" and adding elements with "insert". In Listing 8-20, we’re keeping track of the scores of two teams whose names are *Blue* and *Yellow*. The Blue team starts with 10 points, and the Yellow team starts with 50.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
```

Listing 8-20: Creating a new hash map and inserting some keys and values

Note that we need to first "use" the "HashMap" from the collections portion of the standard library. Of our three common collections, this one is the least often used, so it’s not included in the features brought into scope automatically in the prelude. Hash maps also have less support from the standard library; there’s no built-in macro to construct them, for example.

Just like vectors, hash maps store their data on the heap. This "HashMap" has keys of type "String" and values of type "i32". Like vectors, hash maps are homogeneous: all of the keys must have the same type as each other, and all of the values must have the same type.

### [Accessing Values in a Hash Map](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#accessing-values-in-a-hash-map)

We can get a value out of the hash map by providing its key to the "get" method, as shown in Listing 8-21.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
```

Listing 8-21: Accessing the score for the Blue team stored in the hash map

Here, "score" will have the value that’s associated with the Blue team, and the result will be "10". The "get" method returns an "Option<&V>"; if there’s no value for that key in the hash map, "get" will return "None". This program handles the "Option" by calling "copied" to get an "Option<i32>" rather than an "Option<&i32>", then "unwrap_or" to set "score" to zero if "scores" doesn't have an entry for the key.

We can iterate over each key/value pair in a hash map in a similar manner as we do with vectors, using a "for" loop:

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
```

This code will print each pair in an arbitrary order:

```text
Yellow: 50
Blue: 10
```

### [Hash Maps and Ownership](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hash-maps-and-ownership)

For types that implement the "Copy" trait, like "i32", the values are copied into the hash map. For owned values like "String", the values will be moved and the hash map will be the owner of those values, as demonstrated in Listing 8-22.

```rust
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

Listing 8-22: Showing that keys and values are owned by the hash map once they’re inserted

We aren’t able to use the variables "field_name" and "field_value" after they’ve been moved into the hash map with the call to "insert".

If we insert references to values into the hash map, the values won’t be moved into the hash map. The values that the references point to must be valid for at least as long as the hash map is valid. We’ll talk more about these issues in the [“Validating References with Lifetimes”](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) section in Chapter 10.

### [Updating a Hash Map](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#updating-a-hash-map)

Although the number of key and value pairs is growable, each unique key can only have one value associated with it at a time (but not vice versa: for example, both the Blue team and the Yellow team could have value 10 stored in the "scores" hash map).

When you want to change the data in a hash map, you have to decide how to handle the case when a key already has a value assigned. You could replace the old value with the new value, completely disregarding the old value. You could keep the old value and ignore the new value, only adding the new value if the key *doesn’t* already have a value. Or you could combine the old value and the new value. Let’s look at how to do each of these!

#### [Overwriting a Value](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#overwriting-a-value)

If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced. Even though the code in Listing 8-23 calls "insert" twice, the hash map will only contain one key/value pair because we’re inserting the value for the Blue team’s key both times.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores);
```

Listing 8-23: Replacing a value stored with a particular key

This code will print "{"Blue": 25}". The original value of "10" has been overwritten.



#### [Adding a Key and Value Only If a Key Isn’t Present](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#adding-a-key-and-value-only-if-a-key-isnt-present)

It’s common to check whether a particular key already exists in the hash map with a value then take the following actions: if the key does exist in the hash map, the existing value should remain the way it is. If the key doesn’t exist, insert it and a value for it.

Hash maps have a special API for this called "entry" that takes the key you want to check as a parameter. The return value of the "entry" method is an enum called "Entry" that represents a value that might or might not exist. Let’s say we want to check whether the key for the Yellow team has a value associated with it. If it doesn’t, we want to insert the value 50, and the same for the Blue team. Using the "entry" API, the code looks like Listing 8-24.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
```

Listing 8-24: Using the "entry" method to only insert if the key does not already have a value

The "or_insert" method on "Entry" is defined to return a mutable reference to the value for the corresponding "Entry" key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. This technique is much cleaner than writing the logic ourselves and, in addition, plays more nicely with the borrow checker.

Running the code in Listing 8-24 will print "{"Yellow": 50, "Blue": 10}". The first call to "entry" will insert the key for the Yellow team with the value 50 because the Yellow team doesn’t have a value already. The second call to "entry" will not change the hash map because the Blue team already has the value 10.

#### [Updating a Value Based on the Old Value](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#updating-a-value-based-on-the-old-value)

Another common use case for hash maps is to look up a key’s value and then update it based on the old value. For instance, Listing 8-25 shows code that counts how many times each word appears in some text. We use a hash map with the words as keys and increment the value to keep track of how many times we’ve seen that word. If it’s the first time we’ve seen a word, we’ll first insert the value 0.

```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
```

Listing 8-25: Counting occurrences of words using a hash map that stores words and counts

This code will print "{"world": 2, "hello": 1, "wonderful": 1}". You might see the same key/value pairs printed in a different order: recall from the [“Accessing Values in a Hash Map”](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#accessing-values-in-a-hash-map) section that iterating over a hash map happens in an arbitrary order.

The "split_whitespace" method returns an iterator over sub-slices, separated by whitespace, of the value in "text". The "or_insert" method returns a mutable reference ("&mut V") to the value for the specified key. Here we store that mutable reference in the "count" variable, so in order to assign to that value, we must first dereference "count" using the asterisk ("*"). The mutable reference goes out of scope at the end of the "for" loop, so all of these changes are safe and allowed by the borrowing rules.

### [Hashing Functions](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hashing-functions)

By default, "HashMap" uses a hashing function called *SipHash* that can provide resistance to Denial of Service (DoS) attacks involving hash tables[1](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#siphash). This is not the fastest hashing algorithm available, but the trade-off for better security that comes with the drop in performance is worth it. If you profile your code and find that the default hash function is too slow for your purposes, you can switch to another function by specifying a different hasher. A *hasher* is a type that implements the "BuildHasher" trait. We’ll talk about traits and how to implement them in Chapter 10. You don’t necessarily have to implement your own hasher from scratch; [crates.io](https://crates.io/) has libraries shared by other Rust users that provide hashers implementing many common hashing algorithms.

1 

https://en.wikipedia.org/wiki/SipHash

## [Summary](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#summary)

Vectors, strings, and hash maps will provide a large amount of functionality necessary in programs when you need to store, access, and modify data. Here are some exercises you should now be equipped to solve:

- Given a list of integers, use a vector and return the median (when sorted, the value in the middle position) and mode (the value that occurs most often; a hash map will be helpful here) of the list.
- Convert strings to pig latin. The first consonant of each word is moved to the end of the word and “ay” is added, so “first” becomes “irst-fay.” Words that start with a vowel have “hay” added to the end instead (“apple” becomes “apple-hay”). Keep in mind the details about UTF-8 encoding!
- Using a hash map and vectors, create a text interface to allow a user to add employee names to a department in a company. For example, “Add Sally to Engineering” or “Add Amir to Sales.” Then let the user retrieve a list of all people in a department or all people in the company by department, sorted alphabetically.

The standard library API documentation describes methods that vectors, strings, and hash maps have that will be helpful for these exercises!

We’re getting into more complex programs in which operations can fail, so, it’s a perfect time to discuss error handling. We’ll do that next!





---





# [Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html#error-handling)

Errors are a fact of life in software, so Rust has a number of features for handling situations in which something goes wrong. In many cases, Rust requires you to acknowledge the possibility of an error and take some action before your code will compile. This requirement makes your program more robust by ensuring that you’ll discover errors and handle them appropriately before you’ve deployed your code to production!

Rust groups errors into two major categories: *recoverable* and *unrecoverable* errors. For a recoverable error, such as a *file not found* error, we most likely just want to report the problem to the user and retry the operation. Unrecoverable errors are always symptoms of bugs, like trying to access a location beyond the end of an array, and so we want to immediately stop the program.

Most languages don’t distinguish between these two kinds of errors and handle both in the same way, using mechanisms such as exceptions. Rust doesn’t have exceptions. Instead, it has the type "Result<T, E>" for recoverable errors and the "panic!" macro that stops execution when the program encounters an unrecoverable error. This chapter covers calling "panic!" first and then talks about returning "Result<T, E>" values. Additionally, we’ll explore considerations when deciding whether to try to recover from an error or to stop execution.





## [Unrecoverable Errors with "panic!"](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unrecoverable-errors-with-panic)

Sometimes, bad things happen in your code, and there’s nothing you can do about it. In these cases, Rust has the "panic!" macro. There are two ways to cause a panic in practice: by taking an action that causes our code to panic (such as accessing an array past the end) or by explicitly calling the "panic!" macro. In both cases, we cause a panic in our program. By default, these panics will print a failure message, unwind, clean up the stack, and quit. Via an environment variable, you can also have Rust display the call stack when a panic occurs to make it easier to track down the source of the panic.

> ### [Unwinding the Stack or Aborting in Response to a Panic](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic)
>
> By default, when a panic occurs, the program starts *unwinding*, which means Rust walks back up the stack and cleans up the data from each function it encounters. However, this walking back and cleanup is a lot of work. Rust, therefore, allows you to choose the alternative of immediately *aborting*, which ends the program without cleaning up.
>
> Memory that the program was using will then need to be cleaned up by the operating system. If in your project you need to make the resulting binary as small as possible, you can switch from unwinding to aborting upon a panic by adding "panic = 'abort'" to the appropriate "[profile]" sections in your *Cargo.toml* file. For example, if you want to abort on panic in release mode, add this:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Let’s try calling "panic!" in a simple program:

Filename: src/main.rs

```rust
fn main() {
    panic!("crash and burn");
}
```

When you run the program, you’ll see something like this:

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running "target/debug/panic"
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

The call to "panic!" causes the error message contained in the last two lines. The first line shows our panic message and the place in our source code where the panic occurred: *src/main.rs:2:5* indicates that it’s the second line, fifth character of our *src/main.rs* file.

In this case, the line indicated is part of our code, and if we go to that line, we see the "panic!" macro call. In other cases, the "panic!" call might be in code that our code calls, and the filename and line number reported by the error message will be someone else’s code where the "panic!" macro is called, not the line of our code that eventually led to the "panic!" call. We can use the backtrace of the functions the "panic!" call came from to figure out the part of our code that is causing the problem. We’ll discuss backtraces in more detail next.

### [Using a "panic!" Backtrace](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#using-a-panic-backtrace)

Let’s look at another example to see what it’s like when a "panic!" call comes from a library because of a bug in our code instead of from our code calling the macro directly. Listing 9-1 has some code that attempts to access an index in a vector beyond the range of valid indexes.

Filename: src/main.rs

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

Listing 9-1: Attempting to access an element beyond the end of a vector, which will cause a call to "panic!"

Here, we’re attempting to access the 100th element of our vector (which is at index 99 because indexing starts at zero), but the vector has only 3 elements. In this situation, Rust will panic. Using "[]" is supposed to return an element, but if you pass an invalid index, there’s no element that Rust could return here that would be correct.

In C, attempting to read beyond the end of a data structure is undefined behavior. You might get whatever is at the location in memory that would correspond to that element in the data structure, even though the memory doesn’t belong to that structure. This is called a *buffer overread* and can lead to security vulnerabilities if an attacker is able to manipulate the index in such a way as to read data they shouldn’t be allowed to that is stored after the data structure.

To protect your program from this sort of vulnerability, if you try to read an element at an index that doesn’t exist, Rust will stop execution and refuse to continue. Let’s try it and see:

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running "target/debug/panic"
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

This error points at line 4 of our "main.rs" where we attempt to access index 99. The next note line tells us that we can set the "RUST_BACKTRACE" environment variable to get a backtrace of exactly what happened to cause the error. A *backtrace* is a list of all the functions that have been called to get to this point. Backtraces in Rust work as they do in other languages: the key to reading the backtrace is to start from the top and read until you see files you wrote. That’s the spot where the problem originated. The lines above that spot are code that your code has called; the lines below are code that called your code. These before-and-after lines might include core Rust code, standard library code, or crates that you’re using. Let’s try getting a backtrace by setting the "RUST_BACKTRACE" environment variable to any value except 0. Listing 9-2 shows output similar to what you’ll see.

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with "RUST_BACKTRACE=full" for a verbose backtrace.
```

Listing 9-2: The backtrace generated by a call to "panic!" displayed when the environment variable "RUST_BACKTRACE" is set

That’s a lot of output! The exact output you see might be different depending on your operating system and Rust version. In order to get backtraces with this information, debug symbols must be enabled. Debug symbols are enabled by default when using "cargo build" or "cargo run" without the "--release" flag, as we have here.

In the output in Listing 9-2, line 6 of the backtrace points to the line in our project that’s causing the problem: line 4 of *src/main.rs*. If we don’t want our program to panic, we should start our investigation at the location pointed to by the first line mentioning a file we wrote. In Listing 9-1, where we deliberately wrote code that would panic, the way to fix the panic is to not request an element beyond the range of the vector indexes. When your code panics in the future, you’ll need to figure out what action the code is taking with what values to cause the panic and what the code should do instead.

We’ll come back to "panic!" and when we should and should not use "panic!" to handle error conditions in the [“To "panic!" or Not to "panic!"”](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic) section later in this chapter. Next, we’ll look at how to recover from an error using "Result".





---





## [Recoverable Errors with "Result"](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#recoverable-errors-with-result)

Most errors aren’t serious enough to require the program to stop entirely. Sometimes, when a function fails, it’s for a reason that you can easily interpret and respond to. For example, if you try to open a file and that operation fails because the file doesn’t exist, you might want to create the file instead of terminating the process.

Recall from [“Handling Potential Failure with "Result"”](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result) in Chapter 2 that the "Result" enum is defined as having two variants, "Ok" and "Err", as follows:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The "T" and "E" are generic type parameters: we’ll discuss generics in more detail in Chapter 10. What you need to know right now is that "T" represents the type of the value that will be returned in a success case within the "Ok" variant, and "E" represents the type of the error that will be returned in a failure case within the "Err" variant. Because "Result" has these generic type parameters, we can use the "Result" type and the functions defined on it in many different situations where the successful value and error value we want to return may differ.

Let’s call a function that returns a "Result" value because the function could fail. In Listing 9-3 we try to open a file.

Filename: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

Listing 9-3: Opening a file

The return type of "File::open" is a "Result<T, E>". The generic parameter "T" has been filled in by the implementation of "File::open" with the type of the success value, "std::fs::File", which is a file handle. The type of "E" used in the error value is "std::io::Error". This return type means the call to "File::open" might succeed and return a file handle that we can read from or write to. The function call also might fail: for example, the file might not exist, or we might not have permission to access the file. The "File::open" function needs to have a way to tell us whether it succeeded or failed and at the same time give us either the file handle or error information. This information is exactly what the "Result" enum conveys.

In the case where "File::open" succeeds, the value in the variable "greeting_file_result" will be an instance of "Ok" that contains a file handle. In the case where it fails, the value in "greeting_file_result" will be an instance of "Err" that contains more information about the kind of error that happened.

We need to add to the code in Listing 9-3 to take different actions depending on the value "File::open" returns. Listing 9-4 shows one way to handle the "Result" using a basic tool, the "match" expression that we discussed in Chapter 6.

Filename: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

Listing 9-4: Using a "match" expression to handle the "Result" variants that might be returned

Note that, like the "Option" enum, the "Result" enum and its variants have been brought into scope by the prelude, so we don’t need to specify "Result::" before the "Ok" and "Err" variants in the "match" arms.

When the result is "Ok", this code will return the inner "file" value out of the "Ok" variant, and we then assign that file handle value to the variable "greeting_file". After the "match", we can use the file handle for reading or writing.

The other arm of the "match" handles the case where we get an "Err" value from "File::open". In this example, we’ve chosen to call the "panic!" macro. If there’s no file named *hello.txt* in our current directory and we run this code, we’ll see the following output from the "panic!" macro:

```console
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running "target/debug/error-handling"
thread 'main' panicked at 'Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:8:23
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

As usual, this output tells us exactly what has gone wrong.

### [Matching on Different Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#matching-on-different-errors)

The code in Listing 9-4 will "panic!" no matter why "File::open" failed. However, we want to take different actions for different failure reasons: if "File::open" failed because the file doesn’t exist, we want to create the file and return the handle to the new file. If "File::open" failed for any other reason—for example, because we didn’t have permission to open the file—we still want the code to "panic!" in the same way as it did in Listing 9-4. For this we add an inner "match" expression, shown in Listing 9-5.

Filename: src/main.rs

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

Listing 9-5: Handling different kinds of errors in different ways

The type of the value that "File::open" returns inside the "Err" variant is "io::Error", which is a struct provided by the standard library. This struct has a method "kind" that we can call to get an "io::ErrorKind" value. The enum "io::ErrorKind" is provided by the standard library and has variants representing the different kinds of errors that might result from an "io" operation. The variant we want to use is "ErrorKind::NotFound", which indicates the file we’re trying to open doesn’t exist yet. So we match on "greeting_file_result", but we also have an inner match on "error.kind()".

The condition we want to check in the inner match is whether the value returned by "error.kind()" is the "NotFound" variant of the "ErrorKind" enum. If it is, we try to create the file with "File::create". However, because "File::create" could also fail, we need a second arm in the inner "match" expression. When the file can’t be created, a different error message is printed. The second arm of the outer "match" stays the same, so the program panics on any error besides the missing file error.

> ### [Alternatives to Using "match" with "Result"](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#alternatives-to-using-match-with-resultt-e)
>
> That’s a lot of "match"! The "match" expression is very useful but also very much a primitive. In Chapter 13, you’ll learn about closures, which are used with many of the methods defined on "Result<T, E>". These methods can be more concise than using "match" when handling "Result<T, E>" values in your code.
>
> For example, here’s another way to write the same logic as shown in Listing 9-5, this time using closures and the "unwrap_or_else" method:
>
> ```rust
> use std::fs::File;
> use std::io::ErrorKind;
> 
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {:?}", error);
>             })
>         } else {
>             panic!("Problem opening the file: {:?}", error);
>         }
>     });
> }
> ```
>
> Although this code has the same behavior as Listing 9-5, it doesn’t contain any "match" expressions and is cleaner to read. Come back to this example after you’ve read Chapter 13, and look up the "unwrap_or_else" method in the standard library documentation. Many more of these methods can clean up huge nested "match" expressions when you’re dealing with errors.

### [Shortcuts for Panic on Error: "unwrap" and "expect"](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#shortcuts-for-panic-on-error-unwrap-and-expect)

Using "match" works well enough, but it can be a bit verbose and doesn’t always communicate intent well. The "Result<T, E>" type has many helper methods defined on it to do various, more specific tasks. The "unwrap" method is a shortcut method implemented just like the "match" expression we wrote in Listing 9-4. If the "Result" value is the "Ok" variant, "unwrap" will return the value inside the "Ok". If the "Result" is the "Err" variant, "unwrap" will call the "panic!" macro for us. Here is an example of "unwrap" in action:

Filename: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

If we run this code without a *hello.txt* file, we’ll see an error message from the "panic!" call that the "unwrap" method makes:

```text
thread 'main' panicked at 'called "Result::unwrap()" on an "Err" value: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:4:49
```

Similarly, the "expect" method lets us also choose the "panic!" error message. Using "expect" instead of "unwrap" and providing good error messages can convey your intent and make tracking down the source of a panic easier. The syntax of "expect" looks like this:

Filename: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

We use "expect" in the same way as "unwrap": to return the file handle or call the "panic!" macro. The error message used by "expect" in its call to "panic!" will be the parameter that we pass to "expect", rather than the default "panic!" message that "unwrap" uses. Here’s what it looks like:

```text
thread 'main' panicked at 'hello.txt should be included in this project: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:5:10
```

In production-quality code, most Rustaceans choose "expect" rather than "unwrap" and give more context about why the operation is expected to always succeed. That way, if your assumptions are ever proven wrong, you have more information to use in debugging.

### [Propagating Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#propagating-errors)

When a function’s implementation calls something that might fail, instead of handling the error within the function itself, you can return the error to the calling code so that it can decide what to do. This is known as *propagating* the error and gives more control to the calling code, where there might be more information or logic that dictates how the error should be handled than what you have available in the context of your code.

For example, Listing 9-6 shows a function that reads a username from a file. If the file doesn’t exist or can’t be read, this function will return those errors to the code that called the function.

Filename: src/main.rs

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

Listing 9-6: A function that returns errors to the calling code using "match"

This function can be written in a much shorter way, but we’re going to start by doing a lot of it manually in order to explore error handling; at the end, we’ll show the shorter way. Let’s look at the return type of the function first: "Result<String, io::Error>". This means the function is returning a value of the type "Result<T, E>" where the generic parameter "T" has been filled in with the concrete type "String", and the generic type "E" has been filled in with the concrete type "io::Error".

If this function succeeds without any problems, the code that calls this function will receive an "Ok" value that holds a "String"—the username that this function read from the file. If this function encounters any problems, the calling code will receive an "Err" value that holds an instance of "io::Error" that contains more information about what the problems were. We chose "io::Error" as the return type of this function because that happens to be the type of the error value returned from both of the operations we’re calling in this function’s body that might fail: the "File::open" function and the "read_to_string" method.

The body of the function starts by calling the "File::open" function. Then we handle the "Result" value with a "match" similar to the "match" in Listing 9-4. If "File::open" succeeds, the file handle in the pattern variable "file" becomes the value in the mutable variable "username_file" and the function continues. In the "Err" case, instead of calling "panic!", we use the "return" keyword to return early out of the function entirely and pass the error value from "File::open", now in the pattern variable "e", back to the calling code as this function’s error value.

So if we have a file handle in "username_file", the function then creates a new "String" in variable "username" and calls the "read_to_string" method on the file handle in "username_file" to read the contents of the file into "username". The "read_to_string" method also returns a "Result" because it might fail, even though "File::open" succeeded. So we need another "match" to handle that "Result": if "read_to_string" succeeds, then our function has succeeded, and we return the username from the file that’s now in "username" wrapped in an "Ok". If "read_to_string" fails, we return the error value in the same way that we returned the error value in the "match" that handled the return value of "File::open". However, we don’t need to explicitly say "return", because this is the last expression in the function.

The code that calls this code will then handle getting either an "Ok" value that contains a username or an "Err" value that contains an "io::Error". It’s up to the calling code to decide what to do with those values. If the calling code gets an "Err" value, it could call "panic!" and crash the program, use a default username, or look up the username from somewhere other than a file, for example. We don’t have enough information on what the calling code is actually trying to do, so we propagate all the success or error information upward for it to handle appropriately.

This pattern of propagating errors is so common in Rust that Rust provides the question mark operator "?" to make this easier.

#### [A Shortcut for Propagating Errors: the "?" Operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator)

Listing 9-7 shows an implementation of "read_username_from_file" that has the same functionality as in Listing 9-6, but this implementation uses the "?" operator.

Filename: src/main.rs

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

Listing 9-7: A function that returns errors to the calling code using the "?" operator

The "?" placed after a "Result" value is defined to work in almost the same way as the "match" expressions we defined to handle the "Result" values in Listing 9-6. If the value of the "Result" is an "Ok", the value inside the "Ok" will get returned from this expression, and the program will continue. If the value is an "Err", the "Err" will be returned from the whole function as if we had used the "return" keyword so the error value gets propagated to the calling code.

There is a difference between what the "match" expression from Listing 9-6 does and what the "?" operator does: error values that have the "?" operator called on them go through the "from" function, defined in the "From" trait in the standard library, which is used to convert values from one type into another. When the "?" operator calls the "from" function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons.

For example, we could change the "read_username_from_file" function in Listing 9-7 to return a custom error type named "OurError" that we define. If we also define "impl From<io::Error> for OurError" to construct an instance of "OurError" from an "io::Error", then the "?" operator calls in the body of "read_username_from_file" will call "from" and convert the error types without needing to add any more code to the function.

In the context of Listing 9-7, the "?" at the end of the "File::open" call will return the value inside an "Ok" to the variable "username_file". If an error occurs, the "?" operator will return early out of the whole function and give any "Err" value to the calling code. The same thing applies to the "?" at the end of the "read_to_string" call.

The "?" operator eliminates a lot of boilerplate and makes this function’s implementation simpler. We could even shorten this code further by chaining method calls immediately after the "?", as shown in Listing 9-8.

Filename: src/main.rs

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```

Listing 9-8: Chaining method calls after the "?" operator

We’ve moved the creation of the new "String" in "username" to the beginning of the function; that part hasn’t changed. Instead of creating a variable "username_file", we’ve chained the call to "read_to_string" directly onto the result of "File::open("hello.txt")?". We still have a "?" at the end of the "read_to_string" call, and we still return an "Ok" value containing "username" when both "File::open" and "read_to_string" succeed rather than returning errors. The functionality is again the same as in Listing 9-6 and Listing 9-7; this is just a different, more ergonomic way to write it.

Listing 9-9 shows a way to make this even shorter using "fs::read_to_string".

Filename: src/main.rs

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

Listing 9-9: Using "fs::read_to_string" instead of opening and then reading the file

Reading a file into a string is a fairly common operation, so the standard library provides the convenient "fs::read_to_string" function that opens the file, creates a new "String", reads the contents of the file, puts the contents into that "String", and returns it. Of course, using "fs::read_to_string" doesn’t give us the opportunity to explain all the error handling, so we did it the longer way first.

#### [Where The "?" Operator Can Be Used](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#where-the--operator-can-be-used)

The "?" operator can only be used in functions whose return type is compatible with the value the "?" is used on. This is because the "?" operator is defined to perform an early return of a value out of the function, in the same manner as the "match" expression we defined in Listing 9-6. In Listing 9-6, the "match" was using a "Result" value, and the early return arm returned an "Err(e)" value. The return type of the function has to be a "Result" so that it’s compatible with this "return".

In Listing 9-10, let’s look at the error we’ll get if we use the "?" operator in a "main" function with a return type incompatible with the type of the value we use "?" on:

Filename: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

Listing 9-10: Attempting to use the "?" in the "main" function that returns "()" won’t compile

This code opens a file, which might fail. The "?" operator follows the "Result" value returned by "File::open", but this "main" function has the return type of "()", not "Result". When we compile this code, we get the following error message:

```console
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
error[E0277]: the "?" operator can only be used in a function that returns "Result" or "Option" (or another type that implements "FromResidual")
 --> src/main.rs:4:48
  |
3 | fn main() {
  | --------- this function should return "Result" or "Option" to accept "?"
4 |     let greeting_file = File::open("hello.txt")?;
  |                                                ^ cannot use the "?" operator in a function that returns "()"
  |
  = help: the trait "FromResidual<Result<Infallible, std::io::Error>>" is not implemented for "()"

For more information about this error, try "rustc --explain E0277".
error: could not compile "error-handling" due to previous error
```

This error points out that we’re only allowed to use the "?" operator in a function that returns "Result", "Option", or another type that implements "FromResidual".

To fix the error, you have two choices. One choice is to change the return type of your function to be compatible with the value you’re using the "?" operator on as long as you have no restrictions preventing that. The other technique is to use a "match" or one of the "Result<T, E>` methods to handle the `Result<T, E>` in whatever way is appropriate.

The error message also mentioned that `?` can be used with `Option<T>` values as well. As with using `?` on `Result`, you can only use `?` on `Option` in a function that returns an `Option`. The behavior of the `?` operator when called on an `Option<T>` is similar to its behavior when called on a `Result<T, E>`: if the value is `None`, the `None` will be returned early from the function at that point. If the value is `Some`, the value inside the `Some` is the resulting value of the expression and the function continues. Listing 9-11 has an example of a function that finds the last character of the first line in the given text:

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

Listing 9-11: Using the `?` operator on an `Option<T>` value

This function returns `Option<char>` because it’s possible that there is a character there, but it’s also possible that there isn’t. This code takes the `text` string slice argument and calls the `lines` method on it, which returns an iterator over the lines in the string. Because this function wants to examine the first line, it calls `next` on the iterator to get the first value from the iterator. If `text` is the empty string, this call to `next` will return `None`, in which case we use `?` to stop and return `None` from `last_char_of_first_line`. If `text` is not the empty string, `next` will return a `Some` value containing a string slice of the first line in `text`.

The `?` extracts the string slice, and we can call `chars` on that string slice to get an iterator of its characters. We’re interested in the last character in this first line, so we call `last` to return the last item in the iterator. This is an `Option` because it’s possible that the first line is the empty string, for example if `text` starts with a blank line but has characters on other lines, as in `"\nhi"`. However, if there is a last character on the first line, it will be returned in the `Some` variant. The `?` operator in the middle gives us a concise way to express this logic, allowing us to implement the function in one line. If we couldn’t use the `?` operator on `Option`, we’d have to implement this logic using more method calls or a `match` expression.

Note that you can use the `?` operator on a `Result` in a function that returns `Result`, and you can use the `?` operator on an `Option` in a function that returns `Option`, but you can’t mix and match. The `?` operator won’t automatically convert a `Result` to an `Option` or vice versa; in those cases, you can use methods like the `ok` method on `Result` or the `ok_or` method on `Option` to do the conversion explicitly.

So far, all the `main` functions we’ve used return `()`. The `main` function is special because it’s the entry and exit point of executable programs, and there are restrictions on what its return type can be for the programs to behave as expected.

Luckily, `main` can also return a `Result<(), E>`. Listing 9-12 has the code from Listing 9-10 but we’ve changed the return type of `main` to be `Result<(), Box<dyn Error>>` and added a return value `Ok(())` to the end. This code will now compile:

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

Listing 9-12: Changing `main` to return `Result<(), E>` allows the use of the `?` operator on `Result` values

The `Box<dyn Error>` type is a *trait object*, which we’ll talk about in the [“Using Trait Objects that Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) section in Chapter 17. For now, you can read `Box<dyn Error>` to mean “any kind of error.” Using `?` on a `Result` value in a `main` function with the error type `Box<dyn Error>` is allowed, because it allows any `Err` value to be returned early. Even though the body of this `main` function will only ever return errors of type `std::io::Error`, by specifying `Box<dyn Error>`, this signature will continue to be correct even if more code that returns other errors is added to the body of `main`.

When a `main` function returns a `Result<(), E>`, the executable will exit with a value of `0` if `main` returns `Ok(())` and will exit with a nonzero value if `main` returns an `Err` value. Executables written in C return integers when they exit: programs that exit successfully return the integer `0`, and programs that error return some integer other than `0`. Rust also returns integers from executables to be compatible with this convention.

The `main` function may return any types that implement [the `std::process::Termination` trait](https://doc.rust-lang.org/std/process/trait.Termination.html), which contains a function `report` that returns an `ExitCode`. Consult the standard library documentation for more information on implementing the `Termination` trait for your own types.

Now that we’ve discussed the details of calling `panic!` or returning `Result`, let’s return to the topic of how to decide which is appropriate to use in which cases.





---





## [To `panic!` or Not to `panic!`](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic)

So how do you decide when you should call `panic!` and when you should return `Result`? When code panics, there’s no way to recover. You could call `panic!` for any error situation, whether there’s a possible way to recover or not, but then you’re making the decision that a situation is unrecoverable on behalf of the calling code. When you choose to return a `Result` value, you give the calling code options. The calling code could choose to attempt to recover in a way that’s appropriate for its situation, or it could decide that an `Err` value in this case is unrecoverable, so it can call `panic!` and turn your recoverable error into an unrecoverable one. Therefore, returning `Result` is a good default choice when you’re defining a function that might fail.

In situations such as examples, prototype code, and tests, it’s more appropriate to write code that panics instead of returning a `Result`. Let’s explore why, then discuss situations in which the compiler can’t tell that failure is impossible, but you as a human can. The chapter will conclude with some general guidelines on how to decide whether to panic in library code.

### [Examples, Prototype Code, and Tests](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#examples-prototype-code-and-tests)

When you’re writing an example to illustrate some concept, also including robust error-handling code can make the example less clear. In examples, it’s understood that a call to a method like `unwrap` that could panic is meant as a placeholder for the way you’d want your application to handle errors, which can differ based on what the rest of your code is doing.

Similarly, the `unwrap` and `expect` methods are very handy when prototyping, before you’re ready to decide how to handle errors. They leave clear markers in your code for when you’re ready to make your program more robust.

If a method call fails in a test, you’d want the whole test to fail, even if that method isn’t the functionality under test. Because `panic!` is how a test is marked as a failure, calling `unwrap` or `expect` is exactly what should happen.

### [Cases in Which You Have More Information Than the Compiler](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler)

It would also be appropriate to call `unwrap` or `expect` when you have some other logic that ensures the `Result` will have an `Ok` value, but the logic isn’t something the compiler understands. You’ll still have a `Result` value that you need to handle: whatever operation you’re calling still has the possibility of failing in general, even though it’s logically impossible in your particular situation. If you can ensure by manually inspecting the code that you’ll never have an `Err` variant, it’s perfectly acceptable to call `unwrap`, and even better to document the reason you think you’ll never have an `Err` variant in the `expect` text. Here’s an example:

```rust
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
```

We’re creating an `IpAddr` instance by parsing a hardcoded string. We can see that `127.0.0.1` is a valid IP address, so it’s acceptable to use `expect` here. However, having a hardcoded, valid string doesn’t change the return type of the `parse` method: we still get a `Result` value, and the compiler will still make us handle the `Result` as if the `Err` variant is a possibility because the compiler isn’t smart enough to see that this string is always a valid IP address. If the IP address string came from a user rather than being hardcoded into the program and therefore *did* have a possibility of failure, we’d definitely want to handle the `Result` in a more robust way instead. Mentioning the assumption that this IP address is hardcoded will prompt us to change `expect` to better error handling code if in the future, we need to get the IP address from some other source instead.

### [Guidelines for Error Handling](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling)

It’s advisable to have your code panic when it’s possible that your code could end up in a bad state. In this context, a *bad state* is when some assumption, guarantee, contract, or invariant has been broken, such as when invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:

- The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format.
- Your code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step.
- There’s not a good way to encode this information in the types you use. We’ll work through an example of what we mean in the [“Encoding States and Behavior as Types”](https://doc.rust-lang.org/book/ch17-03-oo-design-patterns.html#encoding-states-and-behavior-as-types) section of Chapter 17.

If someone calls your code and passes in values that don’t make sense, it’s best to return an error if you can so the user of the library can decide what they want to do in that case. However, in cases where continuing could be insecure or harmful, the best choice might be to call `panic!` and alert the person using your library to the bug in their code so they can fix it during development. Similarly, `panic!` is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.

However, when failure is expected, it’s more appropriate to return a `Result` than to make a `panic!` call. Examples include a parser being given malformed data or an HTTP request returning a status that indicates you have hit a rate limit. In these cases, returning a `Result` indicates that failure is an expected possibility that the calling code must decide how to handle.

When your code performs an operation that could put a user at risk if it’s called using invalid values, your code should verify the values are valid first and panic if the values aren’t valid. This is mostly for safety reasons: attempting to operate on invalid data can expose your code to vulnerabilities. This is the main reason the standard library will call `panic!` if you attempt an out-of-bounds memory access: trying to access memory that doesn’t belong to the current data structure is a common security problem. Functions often have *contracts*: their behavior is only guaranteed if the inputs meet particular requirements. Panicking when the contract is violated makes sense because a contract violation always indicates a caller-side bug and it’s not a kind of error you want the calling code to have to explicitly handle. In fact, there’s no reasonable way for calling code to recover; the calling *programmers* need to fix the code. Contracts for a function, especially when a violation will cause a panic, should be explained in the API documentation for the function.

However, having lots of error checks in all of your functions would be verbose and annoying. Fortunately, you can use Rust’s type system (and thus the type checking done by the compiler) to do many of the checks for you. If your function has a particular type as a parameter, you can proceed with your code’s logic knowing that the compiler has already ensured you have a valid value. For example, if you have a type rather than an `Option`, your program expects to have *something* rather than *nothing*. Your code then doesn’t have to handle two cases for the `Some` and `None` variants: it will only have one case for definitely having a value. Code trying to pass nothing to your function won’t even compile, so your function doesn’t have to check for that case at runtime. Another example is using an unsigned integer type such as `u32`, which ensures the parameter is never negative.

### [Creating Custom Types for Validation](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation)

Let’s take the idea of using Rust’s type system to ensure we have a valid value one step further and look at creating a custom type for validation. Recall the guessing game in Chapter 2 in which our code asked the user to guess a number between 1 and 100. We never validated that the user’s guess was between those numbers before checking it against our secret number; we only validated that the guess was positive. In this case, the consequences were not very dire: our output of “Too high” or “Too low” would still be correct. But it would be a useful enhancement to guide the user toward valid guesses and have different behavior when a user guesses a number that’s out of range versus when a user types, for example, letters instead.

One way to do this would be to parse the guess as an `i32` instead of only a `u32` to allow potentially negative numbers, and then add a check for the number being in range, like so:

```rust
    loop {
        // --snip--

        let guess: i32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < 1 || guess > 100 {
            println!("The secret number will be between 1 and 100.");
            continue;
        }

        match guess.cmp(&secret_number) {
            // --snip--
    }
```

The `if` expression checks whether our value is out of range, tells the user about the problem, and calls `continue` to start the next iteration of the loop and ask for another guess. After the `if` expression, we can proceed with the comparisons between `guess` and the secret number knowing that `guess` is between 1 and 100.

However, this is not an ideal solution: if it was absolutely critical that the program only operated on values between 1 and 100, and it had many functions with this requirement, having a check like this in every function would be tedious (and might impact performance).

Instead, we can make a new type and put the validations in a function to create an instance of the type rather than repeating the validations everywhere. That way, it’s safe for functions to use the new type in their signatures and confidently use the values they receive. Listing 9-13 shows one way to define a `Guess` type that will only create an instance of `Guess` if the `new` function receives a value between 1 and 100.

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

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

Listing 9-13: A `Guess` type that will only continue with values between 1 and 100

First, we define a struct named `Guess` that has a field named `value` that holds an `i32`. This is where the number will be stored.

Then we implement an associated function named `new` on `Guess` that creates instances of `Guess` values. The `new` function is defined to have one parameter named `value` of type `i32` and to return a `Guess`. The code in the body of the `new` function tests `value` to make sure it’s between 1 and 100. If `value` doesn’t pass this test, we make a `panic!` call, which will alert the programmer who is writing the calling code that they have a bug they need to fix, because creating a `Guess` with a `value` outside this range would violate the contract that `Guess::new` is relying on. The conditions in which `Guess::new` might panic should be discussed in its public-facing API documentation; we’ll cover documentation conventions indicating the possibility of a `panic!` in the API documentation that you create in Chapter 14. If `value` does pass the test, we create a new `Guess` with its `value` field set to the `value` parameter and return the `Guess`.

Next, we implement a method named `value` that borrows `self`, doesn’t have any other parameters, and returns an `i32`. This kind of method is sometimes called a *getter*, because its purpose is to get some data from its fields and return it. This public method is necessary because the `value` field of the `Guess` struct is private. It’s important that the `value` field be private so code using the `Guess` struct is not allowed to set `value` directly: code outside the module *must* use the `Guess::new` function to create an instance of `Guess`, thereby ensuring there’s no way for a `Guess` to have a `value` that hasn’t been checked by the conditions in the `Guess::new` function.

A function that has a parameter or returns only numbers between 1 and 100 could then declare in its signature that it takes or returns a `Guess` rather than an `i32` and wouldn’t need to do any additional checks in its body.

## [Summary](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#summary)

Rust’s error handling features are designed to help you write more robust code. The `panic!` macro signals that your program is in a state it can’t handle and lets you tell the process to stop instead of trying to proceed with invalid or incorrect values. The `Result` enum uses Rust’s type system to indicate that operations might fail in a way that your code could recover from. You can use `Result` to tell code that calls your code that it needs to handle potential success or failure as well. Using `panic!` and `Result` in the appropriate situations will make your code more reliable in the face of inevitable problems.

Now that you’ve seen useful ways that the standard library uses generics with the `Option` and `Result` enums, we’ll talk about how generics work and how you can use them in your code.





---





# [Generic Types, Traits, and Lifetimes](https://doc.rust-lang.org/book/ch10-00-generics.html#generic-types-traits-and-lifetimes)

Every programming language has tools for effectively handling the duplication of concepts. In Rust, one such tool is *generics*: abstract stand-ins for concrete types or other properties. We can express the behavior of generics or how they relate to other generics without knowing what will be in their place when compiling and running the code.

Functions can take parameters of some generic type, instead of a concrete type like `i32` or `String`, in the same way a function takes parameters with unknown values to run the same code on multiple concrete values. In fact, we’ve already used generics in Chapter 6 with `Option<T>`, Chapter 8 with `Vec<T>` and `HashMap<K, V>`, and Chapter 9 with `Result<T, E>`. In this chapter, you’ll explore how to define your own types, functions, and methods with generics!

First, we’ll review how to extract a function to reduce code duplication. We’ll then use the same technique to make a generic function from two functions that differ only in the types of their parameters. We’ll also explain how to use generic types in struct and enum definitions.

Then you’ll learn how to use *traits* to define behavior in a generic way. You can combine traits with generic types to constrain a generic type to accept only those types that have a particular behavior, as opposed to just any type.

Finally, we’ll discuss *lifetimes*: a variety of generics that give the compiler information about how references relate to each other. Lifetimes allow us to give the compiler enough information about borrowed values so that it can ensure references will be valid in more situations than it could without our help.

## [Removing Duplication by Extracting a Function](https://doc.rust-lang.org/book/ch10-00-generics.html#removing-duplication-by-extracting-a-function)

Generics allow us to replace specific types with a placeholder that represents multiple types to remove code duplication. Before diving into generics syntax, then, let’s first look at how to remove duplication in a way that doesn’t involve generic types by extracting a function that replaces specific values with a placeholder that represents multiple values. Then we’ll apply the same technique to extract a generic function! By looking at how to recognize duplicated code you can extract into a function, you’ll start to recognize duplicated code that can use generics.

We begin with the short program in Listing 10-1 that finds the largest number in a list.

Filename: src/main.rs

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

Listing 10-1: Finding the largest number in a list of numbers

We store a list of integers in the variable `number_list` and place a reference to the first number in the list in a variable named `largest`. We then iterate through all the numbers in the list, and if the current number is greater than the number stored in `largest`, replace the reference in that variable. However, if the current number is less than or equal to the largest number seen so far, the variable doesn’t change, and the code moves on to the next number in the list. After considering all the numbers in the list, `largest` should refer to the largest number, which in this case is 100.

We've now been tasked with finding the largest number in two different lists of numbers. To do so, we can choose to duplicate the code in Listing 10-1 and use the same logic at two different places in the program, as shown in Listing 10-2.

Filename: src/main.rs

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

Listing 10-2: Code to find the largest number in *two* lists of numbers

Although this code works, duplicating code is tedious and error prone. We also have to remember to update the code in multiple places when we want to change it.

To eliminate this duplication, we’ll create an abstraction by defining a function that operates on any list of integers passed in a parameter. This solution makes our code clearer and lets us express the concept of finding the largest number in a list abstractly.

In Listing 10-3, we extract the code that finds the largest number into a function named `largest`. Then we call the function to find the largest number in the two lists from Listing 10-2. We could also use the function on any other list of `i32` values we might have in the future.

Filename: src/main.rs

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
}
```

Listing 10-3: Abstracted code to find the largest number in two lists

The `largest` function has a parameter called `list`, which represents any concrete slice of `i32` values we might pass into the function. As a result, when we call the function, the code runs on the specific values that we pass in.

In summary, here are the steps we took to change the code from Listing 10-2 to Listing 10-3:

1. Identify duplicate code.
2. Extract the duplicate code into the body of the function and specify the inputs and return values of that code in the function signature.
3. Update the two instances of duplicated code to call the function instead.

Next, we’ll use these same steps with generics to reduce code duplication. In the same way that the function body can operate on an abstract `list` instead of specific values, generics allow code to operate on abstract types.

For example, say we had two functions: one that finds the largest item in a slice of `i32` values and one that finds the largest item in a slice of `char` values. How would we eliminate that duplication? Let’s find out!





---





## [Generic Data Types](https://doc.rust-lang.org/book/ch10-01-syntax.html#generic-data-types)

We use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types. Let’s first look at how to define functions, structs, enums, and methods using generics. Then we’ll discuss how generics affect code performance.

### [In Function Definitions](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-function-definitions)

When defining a function that uses generics, we place the generics in the signature of the function where we would usually specify the data types of the parameters and return value. Doing so makes our code more flexible and provides more functionality to callers of our function while preventing code duplication.

Continuing with our `largest` function, Listing 10-4 shows two functions that both find the largest value in a slice. We'll then combine these into a single function that uses generics.

Filename: src/main.rs

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

Listing 10-4: Two functions that differ only in their names and the types in their signatures

The `largest_i32` function is the one we extracted in Listing 10-3 that finds the largest `i32` in a slice. The `largest_char` function finds the largest `char` in a slice. The function bodies have the same code, so let’s eliminate the duplication by introducing a generic type parameter in a single function.

To parameterize the types in a new single function, we need to name the type parameter, just as we do for the value parameters to a function. You can use any identifier as a type parameter name. But we’ll use `T` because, by convention, type parameter names in Rust are short, often just a letter, and Rust’s type-naming convention is UpperCamelCase. Short for “type,” `T` is the default choice of most Rust programmers.

When we use a parameter in the body of the function, we have to declare the parameter name in the signature so the compiler knows what that name means. Similarly, when we use a type parameter name in a function signature, we have to declare the type parameter name before we use it. To define the generic `largest` function, place type name declarations inside angle brackets, `<>`, between the name of the function and the parameter list, like this:

```rust
fn largest<T>(list: &[T]) -> &T {
```

We read this definition as: the function `largest` is generic over some type `T`. This function has one parameter named `list`, which is a slice of values of type `T`. The `largest` function will return a reference to a value of the same type `T`.

Listing 10-5 shows the combined `largest` function definition using the generic data type in its signature. The listing also shows how we can call the function with either a slice of `i32` values or `char` values. Note that this code won’t compile yet, but we’ll fix it later in this chapter.

Filename: src/main.rs

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

Listing 10-5: The `largest` function using generic type parameters; this doesn’t yet compile

If we compile this code right now, we’ll get this error:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
```

The help text mentions `std::cmp::PartialOrd`, which is a *trait*, and we’re going to talk about traits in the next section. For now, know that this error states that the body of `largest` won’t work for all possible types that `T` could be. Because we want to compare values of type `T` in the body, we can only use types whose values can be ordered. To enable comparisons, the standard library has the `std::cmp::PartialOrd` trait that you can implement on types (see Appendix C for more on this trait). By following the help text's suggestion, we restrict the types valid for `T` to only those that implement `PartialOrd` and this example will compile, because the standard library implements `PartialOrd` on both `i32` and `char`.

### [In Struct Definitions](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-struct-definitions)

We can also define structs to use a generic type parameter in one or more fields using the `<>` syntax. Listing 10-6 defines a `Point<T>` struct to hold `x` and `y` coordinate values of any type.

Filename: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

Listing 10-6: A `Point<T>` struct that holds `x` and `y` values of type `T`

The syntax for using generics in struct definitions is similar to that used in function definitions. First, we declare the name of the type parameter inside angle brackets just after the name of the struct. Then we use the generic type in the struct definition where we would otherwise specify concrete data types.

Note that because we’ve used only one generic type to define `Point<T>`, this definition says that the `Point<T>` struct is generic over some type `T`, and the fields `x` and `y` are *both* that same type, whatever that type may be. If we create an instance of a `Point<T>` that has values of different types, as in Listing 10-7, our code won’t compile.

Filename: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

Listing 10-7: The fields `x` and `y` must be the same type because both have the same generic data type `T`.

In this example, when we assign the integer value 5 to `x`, we let the compiler know that the generic type `T` will be an integer for this instance of `Point<T>`. Then when we specify 4.0 for `y`, which we’ve defined to have the same type as `x`, we’ll get a type mismatch error like this:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` due to previous error
```

To define a `Point` struct where `x` and `y` are both generics but could have different types, we can use multiple generic type parameters. For example, in Listing 10-8, we change the definition of `Point` to be generic over types `T` and `U` where `x` is of type `T` and `y` is of type `U`.

Filename: src/main.rs

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

Listing 10-8: A `Point<T, U>` generic over two types so that `x` and `y` can be values of different types

Now all the instances of `Point` shown are allowed! You can use as many generic type parameters in a definition as you want, but using more than a few makes your code hard to read. If you're finding you need lots of generic types in your code, it could indicate that your code needs restructuring into smaller pieces.

### [In Enum Definitions](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-enum-definitions)

As we did with structs, we can define enums to hold generic data types in their variants. Let’s take another look at the `Option<T>` enum that the standard library provides, which we used in Chapter 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

This definition should now make more sense to you. As you can see, the `Option<T>` enum is generic over type `T` and has two variants: `Some`, which holds one value of type `T`, and a `None` variant that doesn’t hold any value. By using the `Option<T>` enum, we can express the abstract concept of an optional value, and because `Option<T>` is generic, we can use this abstraction no matter what the type of the optional value is.

Enums can use multiple generic types as well. The definition of the `Result` enum that we used in Chapter 9 is one example:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The `Result` enum is generic over two types, `T` and `E`, and has two variants: `Ok`, which holds a value of type `T`, and `Err`, which holds a value of type `E`. This definition makes it convenient to use the `Result` enum anywhere we have an operation that might succeed (return a value of some type `T`) or fail (return an error of some type `E`). In fact, this is what we used to open a file in Listing 9-3, where `T` was filled in with the type `std::fs::File` when the file was opened successfully and `E` was filled in with the type `std::io::Error` when there were problems opening the file.

When you recognize situations in your code with multiple struct or enum definitions that differ only in the types of the values they hold, you can avoid duplication by using generic types instead.

### [In Method Definitions](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-method-definitions)

We can implement methods on structs and enums (as we did in Chapter 5) and use generic types in their definitions, too. Listing 10-9 shows the `Point<T>` struct we defined in Listing 10-6 with a method named `x` implemented on it.

Filename: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

Listing 10-9: Implementing a method named `x` on the `Point<T>` struct that will return a reference to the `x` field of type `T`

Here, we’ve defined a method named `x` on `Point<T>` that returns a reference to the data in the field `x`.

Note that we have to declare `T` just after `impl` so we can use `T` to specify that we’re implementing methods on the type `Point<T>`. By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete type. We could have chosen a different name for this generic parameter than the generic parameter declared in the struct definition, but using the same name is conventional. Methods written within an `impl` that declares the generic type will be defined on any instance of the type, no matter what concrete type ends up substituting for the generic type.

We can also specify constraints on generic types when defining methods on the type. We could, for example, implement methods only on `Point<f32>` instances rather than on `Point<T>` instances with any generic type. In Listing 10-10 we use the concrete type `f32`, meaning we don’t declare any types after `impl`.

Filename: src/main.rs

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

Listing 10-10: An `impl` block that only applies to a struct with a particular concrete type for the generic type parameter `T`

This code means the type `Point<f32>` will have a `distance_from_origin` method; other instances of `Point<T>` where `T` is not of type `f32` will not have this method defined. The method measures how far our point is from the point at coordinates (0.0, 0.0) and uses mathematical operations that are available only for floating point types.

Generic type parameters in a struct definition aren’t always the same as those you use in that same struct’s method signatures. Listing 10-11 uses the generic types `X1` and `Y1` for the `Point` struct and `X2` `Y2` for the `mixup` method signature to make the example clearer. The method creates a new `Point` instance with the `x` value from the `self` `Point` (of type `X1`) and the `y` value from the passed-in `Point` (of type `Y2`).

Filename: src/main.rs

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

Listing 10-11: A method that uses generic types different from its struct’s definition

In `main`, we’ve defined a `Point` that has an `i32` for `x` (with value `5`) and an `f64` for `y` (with value `10.4`). The `p2` variable is a `Point` struct that has a string slice for `x` (with value `"Hello"`) and a `char` for `y` (with value `c`). Calling `mixup` on `p1` with the argument `p2` gives us `p3`, which will have an `i32` for `x`, because `x` came from `p1`. The `p3` variable will have a `char` for `y`, because `y` came from `p2`. The `println!` macro call will print `p3.x = 5, p3.y = c`.

The purpose of this example is to demonstrate a situation in which some generic parameters are declared with `impl` and some are declared with the method definition. Here, the generic parameters `X1` and `Y1` are declared after `impl` because they go with the struct definition. The generic parameters `X2` and `Y2` are declared after `fn mixup`, because they’re only relevant to the method.

### [Performance of Code Using Generics](https://doc.rust-lang.org/book/ch10-01-syntax.html#performance-of-code-using-generics)

You might be wondering whether there is a runtime cost when using generic type parameters. The good news is that using generic types won't make your program run any slower than it would with concrete types.

Rust accomplishes this by performing monomorphization of the code using generics at compile time. *Monomorphization* is the process of turning generic code into specific code by filling in the concrete types that are used when compiled. In this process, the compiler does the opposite of the steps we used to create the generic function in Listing 10-5: the compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.

Let’s look at how this works by using the standard library’s generic `Option<T>` enum:

```rust
let integer = Some(5);
let float = Some(5.0);
```

When Rust compiles this code, it performs monomorphization. During that process, the compiler reads the values that have been used in `Option<T>` instances and identifies two kinds of `Option<T>`: one is `i32` and the other is `f64`. As such, it expands the generic definition of `Option<T>` into two definitions specialized to `i32` and `f64`, thereby replacing the generic definition with the specific ones.

The monomorphized version of the code looks similar to the following (the compiler uses different names than what we’re using here for illustration):

Filename: src/main.rs

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

The generic `Option<T>` is replaced with the specific definitions created by the compiler. Because Rust compiles generic code into code that specifies the type in each instance, we pay no runtime cost for using generics. When the code runs, it performs just as it would if we had duplicated each definition by hand. The process of monomorphization makes Rust’s generics extremely efficient at runtime.





---





## [Traits: Defining Shared Behavior](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior)

A *trait* defines functionality a particular type has and can share with other types. We can use traits to define shared behavior in an abstract way. We can use *trait bounds* to specify that a generic type can be any type that has certain behavior.

> Note: Traits are similar to a feature often called *interfaces* in other languages, although with some differences.

### [Defining a Trait](https://doc.rust-lang.org/book/ch10-02-traits.html#defining-a-trait)

A type’s behavior consists of the methods we can call on that type. Different types share the same behavior if we can call the same methods on all of those types. Trait definitions are a way to group method signatures together to define a set of behaviors necessary to accomplish some purpose.

For example, let’s say we have multiple structs that hold various kinds and amounts of text: a `NewsArticle` struct that holds a news story filed in a particular location and a `Tweet` that can have at most 280 characters along with metadata that indicates whether it was a new tweet, a retweet, or a reply to another tweet.

We want to make a media aggregator library crate named `aggregator` that can display summaries of data that might be stored in a `NewsArticle` or `Tweet` instance. To do this, we need a summary from each type, and we’ll request that summary by calling a `summarize` method on an instance. Listing 10-12 shows the definition of a public `Summary` trait that expresses this behavior.

Filename: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Listing 10-12: A `Summary` trait that consists of the behavior provided by a `summarize` method

Here, we declare a trait using the `trait` keyword and then the trait’s name, which is `Summary` in this case. We’ve also declared the trait as `pub` so that crates depending on this crate can make use of this trait too, as we’ll see in a few examples. Inside the curly brackets, we declare the method signatures that describe the behaviors of the types that implement this trait, which in this case is `fn summarize(&self) -> String`.

After the method signature, instead of providing an implementation within curly brackets, we use a semicolon. Each type implementing this trait must provide its own custom behavior for the body of the method. The compiler will enforce that any type that has the `Summary` trait will have the method `summarize` defined with this signature exactly.

A trait can have multiple methods in its body: the method signatures are listed one per line and each line ends in a semicolon.

### [Implementing a Trait on a Type](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type)

Now that we’ve defined the desired signatures of the `Summary` trait’s methods, we can implement it on the types in our media aggregator. Listing 10-13 shows an implementation of the `Summary` trait on the `NewsArticle` struct that uses the headline, the author, and the location to create the return value of `summarize`. For the `Tweet` struct, we define `summarize` as the username followed by the entire text of the tweet, assuming that tweet content is already limited to 280 characters.

Filename: src/lib.rs

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

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

Listing 10-13: Implementing the `Summary` trait on the `NewsArticle` and `Tweet` types

Implementing a trait on a type is similar to implementing regular methods. The difference is that after `impl`, we put the trait name we want to implement, then use the `for` keyword, and then specify the name of the type we want to implement the trait for. Within the `impl` block, we put the method signatures that the trait definition has defined. Instead of adding a semicolon after each signature, we use curly brackets and fill in the method body with the specific behavior that we want the methods of the trait to have for the particular type.

Now that the library has implemented the `Summary` trait on `NewsArticle` and `Tweet`, users of the crate can call the trait methods on instances of `NewsArticle` and `Tweet` in the same way we call regular methods. The only difference is that the user must bring the trait into scope as well as the types. Here’s an example of how a binary crate could use our `aggregator` library crate:

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

This code prints `1 new tweet: horse_ebooks: of course, as you probably already know, people`.

Other crates that depend on the `aggregator` crate can also bring the `Summary` trait into scope to implement `Summary` on their own types. One restriction to note is that we can implement a trait on a type only if at least one of the trait or the type is local to our crate. For example, we can implement standard library traits like `Display` on a custom type like `Tweet` as part of our `aggregator` crate functionality, because the type `Tweet` is local to our `aggregator` crate. We can also implement `Summary` on `Vec<T>` in our `aggregator` crate, because the trait `Summary` is local to our `aggregator` crate.

But we can’t implement external traits on external types. For example, we can’t implement the `Display` trait on `Vec<T>` within our `aggregator` crate, because `Display` and `Vec<T>` are both defined in the standard library and aren’t local to our `aggregator` crate. This restriction is part of a property called *coherence*, and more specifically the *orphan rule*, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa. Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.

### [Default Implementations](https://doc.rust-lang.org/book/ch10-02-traits.html#default-implementations)

Sometimes it’s useful to have default behavior for some or all of the methods in a trait instead of requiring implementations for all methods on every type. Then, as we implement the trait on a particular type, we can keep or override each method’s default behavior.

In Listing 10-14 we specify a default string for the `summarize` method of the `Summary` trait instead of only defining the method signature, as we did in Listing 10-12.

Filename: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

Listing 10-14: Defining a `Summary` trait with a default implementation of the `summarize` method

To use a default implementation to summarize instances of `NewsArticle`, we specify an empty `impl` block with `impl Summary for NewsArticle {}`.

Even though we’re no longer defining the `summarize` method on `NewsArticle` directly, we’ve provided a default implementation and specified that `NewsArticle` implements the `Summary` trait. As a result, we can still call the `summarize` method on an instance of `NewsArticle`, like this:

```rust
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
```

This code prints `New article available! (Read more...)`.

Creating a default implementation doesn’t require us to change anything about the implementation of `Summary` on `Tweet` in Listing 10-13. The reason is that the syntax for overriding a default implementation is the same as the syntax for implementing a trait method that doesn’t have a default implementation.

Default implementations can call other methods in the same trait, even if those other methods don’t have a default implementation. In this way, a trait can provide a lot of useful functionality and only require implementors to specify a small part of it. For example, we could define the `Summary` trait to have a `summarize_author` method whose implementation is required, and then define a `summarize` method that has a default implementation that calls the `summarize_author` method:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

To use this version of `Summary`, we only need to define `summarize_author` when we implement the trait on a type:

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

After we define `summarize_author`, we can call `summarize` on instances of the `Tweet` struct, and the default implementation of `summarize` will call the definition of `summarize_author` that we’ve provided. Because we’ve implemented `summarize_author`, the `Summary` trait has given us the behavior of the `summarize` method without requiring us to write any more code.

```rust
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
```

This code prints `1 new tweet: (Read more from @horse_ebooks...)`.

Note that it isn’t possible to call the default implementation from an overriding implementation of that same method.

### [Traits as Parameters](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters)

Now that you know how to define and implement traits, we can explore how to use traits to define functions that accept many different types. We'll use the `Summary` trait we implemented on the `NewsArticle` and `Tweet` types in Listing 10-13 to define a `notify` function that calls the `summarize` method on its `item` parameter, which is of some type that implements the `Summary` trait. To do this, we use the `impl Trait` syntax, like this:

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

Instead of a concrete type for the `item` parameter, we specify the `impl` keyword and the trait name. This parameter accepts any type that implements the specified trait. In the body of `notify`, we can call any methods on `item` that come from the `Summary` trait, such as `summarize`. We can call `notify` and pass in any instance of `NewsArticle` or `Tweet`. Code that calls the function with any other type, such as a `String` or an `i32`, won’t compile because those types don’t implement `Summary`.



#### [Trait Bound Syntax](https://doc.rust-lang.org/book/ch10-02-traits.html#trait-bound-syntax)

The `impl Trait` syntax works for straightforward cases but is actually syntax sugar for a longer form known as a *trait bound*; it looks like this:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

This longer form is equivalent to the example in the previous section but is more verbose. We place trait bounds with the declaration of the generic type parameter after a colon and inside angle brackets.

The `impl Trait` syntax is convenient and makes for more concise code in simple cases, while the fuller trait bound syntax can express more complexity in other cases. For example, we can have two parameters that implement `Summary`. Doing so with the `impl Trait` syntax looks like this:

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Using `impl Trait` is appropriate if we want this function to allow `item1` and `item2` to have different types (as long as both types implement `Summary`). If we want to force both parameters to have the same type, however, we must use a trait bound, like this:

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

The generic type `T` specified as the type of the `item1` and `item2` parameters constrains the function such that the concrete type of the value passed as an argument for `item1` and `item2` must be the same.

#### [Specifying Multiple Trait Bounds with the `+` Syntax](https://doc.rust-lang.org/book/ch10-02-traits.html#specifying-multiple-trait-bounds-with-the--syntax)

We can also specify more than one trait bound. Say we wanted `notify` to use display formatting as well as `summarize` on `item`: we specify in the `notify` definition that `item` must implement both `Display` and `Summary`. We can do so using the `+` syntax:

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

The `+` syntax is also valid with trait bounds on generic types:

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

With the two trait bounds specified, the body of `notify` can call `summarize` and use `{}` to format `item`.

#### [Clearer Trait Bounds with `where` Clauses](https://doc.rust-lang.org/book/ch10-02-traits.html#clearer-trait-bounds-with-where-clauses)

Using too many trait bounds has its downsides. Each generic has its own trait bounds, so functions with multiple generic type parameters can contain lots of trait bound information between the function’s name and its parameter list, making the function signature hard to read. For this reason, Rust has alternate syntax for specifying trait bounds inside a `where` clause after the function signature. So instead of writing this:

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

we can use a `where` clause, like this:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

This function’s signature is less cluttered: the function name, parameter list, and return type are close together, similar to a function without lots of trait bounds.

### [Returning Types that Implement Traits](https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits)

We can also use the `impl Trait` syntax in the return position to return a value of some type that implements a trait, as shown here:

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

By using `impl Summary` for the return type, we specify that the `returns_summarizable` function returns some type that implements the `Summary` trait without naming the concrete type. In this case, `returns_summarizable` returns a `Tweet`, but the code calling this function doesn’t need to know that.

The ability to specify a return type only by the trait it implements is especially useful in the context of closures and iterators, which we cover in Chapter 13. Closures and iterators create types that only the compiler knows or types that are very long to specify. The `impl Trait` syntax lets you concisely specify that a function returns some type that implements the `Iterator` trait without needing to write out a very long type.

However, you can only use `impl Trait` if you’re returning a single type. For example, this code that returns either a `NewsArticle` or a `Tweet` with the return type specified as `impl Summary` wouldn’t work:

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

Returning either a `NewsArticle` or a `Tweet` isn’t allowed due to restrictions around how the `impl Trait` syntax is implemented in the compiler. We’ll cover how to write a function with this behavior in the [“Using Trait Objects That Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) section of Chapter 17.

### [Using Trait Bounds to Conditionally Implement Methods](https://doc.rust-lang.org/book/ch10-02-traits.html#using-trait-bounds-to-conditionally-implement-methods)

By using a trait bound with an `impl` block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits. For example, the type `Pair<T>` in Listing 10-15 always implements the `new` function to return a new instance of `Pair<T>` (recall from the [“Defining Methods”](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#defining-methods) section of Chapter 5 that `Self` is a type alias for the type of the `impl` block, which in this case is `Pair<T>`). But in the next `impl` block, `Pair<T>` only implements the `cmp_display` method if its inner type `T` implements the `PartialOrd` trait that enables comparison *and* the `Display` trait that enables printing.

Filename: src/lib.rs

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

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

Listing 10-15: Conditionally implementing methods on a generic type depending on trait bounds

We can also conditionally implement a trait for any type that implements another trait. Implementations of a trait on any type that satisfies the trait bounds are called *blanket implementations* and are extensively used in the Rust standard library. For example, the standard library implements the `ToString` trait on any type that implements the `Display` trait. The `impl` block in the standard library looks similar to this code:

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

Because the standard library has this blanket implementation, we can call the `to_string` method defined by the `ToString` trait on any type that implements the `Display` trait. For example, we can turn integers into their corresponding `String` values like this because integers implement `Display`:

```rust
let s = 3.to_string();
```

Blanket implementations appear in the documentation for the trait in the “Implementors” section.

Traits and trait bounds let us write code that uses generic type parameters to reduce duplication but also specify to the compiler that we want the generic type to have particular behavior. The compiler can then use the trait bound information to check that all the concrete types used with our code provide the correct behavior. In dynamically typed languages, we would get an error at runtime if we called a method on a type which didn’t define the method. But Rust moves these errors to compile time so we’re forced to fix the problems before our code is even able to run. Additionally, we don’t have to write code that checks for behavior at runtime because we’ve already checked at compile time. Doing so improves performance without having to give up the flexibility of generics.





---





## [Validating References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes)

Lifetimes are another kind of generic that we’ve already been using. Rather than ensuring that a type has the behavior we want, lifetimes ensure that references are valid as long as we need them to be.

One detail we didn’t discuss in the [“References and Borrowing”](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing) section in Chapter 4 is that every reference in Rust has a *lifetime*, which is the scope for which that reference is valid. Most of the time, lifetimes are implicit and inferred, just like most of the time, types are inferred. We only must annotate types when multiple types are possible. In a similar way, we must annotate lifetimes when the lifetimes of references could be related in a few different ways. Rust requires us to annotate the relationships using generic lifetime parameters to ensure the actual references used at runtime will definitely be valid.

Annotating lifetimes is not even a concept most other programming languages have, so this is going to feel unfamiliar. Although we won’t cover lifetimes in their entirety in this chapter, we’ll discuss common ways you might encounter lifetime syntax so you can get comfortable with the concept.

### [Preventing Dangling References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#preventing-dangling-references-with-lifetimes)

The main aim of lifetimes is to prevent *dangling references*, which cause a program to reference data other than the data it’s intended to reference. Consider the program in Listing 10-16, which has an outer scope and an inner scope.

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

Listing 10-16: An attempt to use a reference whose value has gone out of scope

> Note: The examples in Listings 10-16, 10-17, and 10-23 declare variables without giving them an initial value, so the variable name exists in the outer scope. At first glance, this might appear to be in conflict with Rust’s having no null values. However, if we try to use a variable before giving it a value, we’ll get a compile-time error, which shows that Rust indeed does not allow null values.

The outer scope declares a variable named `r` with no initial value, and the inner scope declares a variable named `x` with the initial value of 5. Inside the inner scope, we attempt to set the value of `r` as a reference to `x`. Then the inner scope ends, and we attempt to print the value in `r`. This code won’t compile because the value `r` is referring to has gone out of scope before we try to use it. Here is the error message:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {}", r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

The variable `x` doesn’t “live long enough.” The reason is that `x` will be out of scope when the inner scope ends on line 7. But `r` is still valid for the outer scope; because its scope is larger, we say that it “lives longer.” If Rust allowed this code to work, `r` would be referencing memory that was deallocated when `x` went out of scope, and anything we tried to do with `r` wouldn’t work correctly. So how does Rust determine that this code is invalid? It uses a borrow checker.

### [The Borrow Checker](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-borrow-checker)

The Rust compiler has a *borrow checker* that compares scopes to determine whether all borrows are valid. Listing 10-17 shows the same code as Listing 10-16 but with annotations showing the lifetimes of the variables.

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

Listing 10-17: Annotations of the lifetimes of `r` and `x`, named `'a` and `'b`, respectively

Here, we’ve annotated the lifetime of `r` with `'a` and the lifetime of `x` with `'b`. As you can see, the inner `'b` block is much smaller than the outer `'a` lifetime block. At compile time, Rust compares the size of the two lifetimes and sees that `r` has a lifetime of `'a` but that it refers to memory with a lifetime of `'b`. The program is rejected because `'b` is shorter than `'a`: the subject of the reference doesn’t live as long as the reference.

Listing 10-18 fixes the code so it doesn’t have a dangling reference and compiles without any errors.

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

Listing 10-18: A valid reference because the data has a longer lifetime than the reference

Here, `x` has the lifetime `'b`, which in this case is larger than `'a`. This means `r` can reference `x` because Rust knows that the reference in `r` will always be valid while `x` is valid.

Now that you know where the lifetimes of references are and how Rust analyzes lifetimes to ensure references will always be valid, let’s explore generic lifetimes of parameters and return values in the context of functions.

### [Generic Lifetimes in Functions](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-lifetimes-in-functions)

We’ll write a function that returns the longer of two string slices. This function will take two string slices and return a single string slice. After we’ve implemented the `longest` function, the code in Listing 10-19 should print `The longest string is abcd`.

Filename: src/main.rs

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

Listing 10-19: A `main` function that calls the `longest` function to find the longer of two string slices

Note that we want the function to take string slices, which are references, rather than strings, because we don’t want the `longest` function to take ownership of its parameters. Refer to the [“String Slices as Parameters”](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters) section in Chapter 4 for more discussion about why the parameters we use in Listing 10-19 are the ones we want.

If we try to implement the `longest` function as shown in Listing 10-20, it won’t compile.

Filename: src/main.rs

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Listing 10-20: An implementation of the `longest` function that returns the longer of two string slices but does not yet compile

Instead, we get the following error that talks about lifetimes:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
```

The help text reveals that the return type needs a generic lifetime parameter on it because Rust can’t tell whether the reference being returned refers to `x` or `y`. Actually, we don’t know either, because the `if` block in the body of this function returns a reference to `x` and the `else` block returns a reference to `y`!

When we’re defining this function, we don’t know the concrete values that will be passed into this function, so we don’t know whether the `if` case or the `else` case will execute. We also don’t know the concrete lifetimes of the references that will be passed in, so we can’t look at the scopes as we did in Listings 10-17 and 10-18 to determine whether the reference we return will always be valid. The borrow checker can’t determine this either, because it doesn’t know how the lifetimes of `x` and `y` relate to the lifetime of the return value. To fix this error, we’ll add generic lifetime parameters that define the relationship between the references so the borrow checker can perform its analysis.

### [Lifetime Annotation Syntax](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotation-syntax)

Lifetime annotations don’t change how long any of the references live. Rather, they describe the relationships of the lifetimes of multiple references to each other without affecting the lifetimes. Just as functions can accept any type when the signature specifies a generic type parameter, functions can accept references with any lifetime by specifying a generic lifetime parameter.

Lifetime annotations have a slightly unusual syntax: the names of lifetime parameters must start with an apostrophe (`'`) and are usually all lowercase and very short, like generic types. Most people use the name `'a` for the first lifetime annotation. We place lifetime parameter annotations after the `&` of a reference, using a space to separate the annotation from the reference’s type.

Here are some examples: a reference to an `i32` without a lifetime parameter, a reference to an `i32` that has a lifetime parameter named `'a`, and a mutable reference to an `i32` that also has the lifetime `'a`.

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

One lifetime annotation by itself doesn’t have much meaning, because the annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other. Let’s examine how the lifetime annotations relate to each other in the context of the `longest` function.

### [Lifetime Annotations in Function Signatures](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-function-signatures)

To use lifetime annotations in function signatures, we need to declare the generic *lifetime* parameters inside angle brackets between the function name and the parameter list, just as we did with generic *type* parameters.

We want the signature to express the following constraint: the returned reference will be valid as long as both the parameters are valid. This is the relationship between lifetimes of the parameters and the return value. We’ll name the lifetime `'a` and then add it to each reference, as shown in Listing 10-21.

Filename: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Listing 10-21: The `longest` function definition specifying that all the references in the signature must have the same lifetime `'a`

This code should compile and produce the result we want when we use it with the `main` function in Listing 10-19.

The function signature now tells Rust that for some lifetime `'a`, the function takes two parameters, both of which are string slices that live at least as long as lifetime `'a`. The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`. In practice, it means that the lifetime of the reference returned by the `longest` function is the same as the smaller of the lifetimes of the values referred to by the function arguments. These relationships are what we want Rust to use when analyzing this code.

Remember, when we specify the lifetime parameters in this function signature, we’re not changing the lifetimes of any values passed in or returned. Rather, we’re specifying that the borrow checker should reject any values that don’t adhere to these constraints. Note that the `longest` function doesn’t need to know exactly how long `x` and `y` will live, only that some scope can be substituted for `'a` that will satisfy this signature.

When annotating lifetimes in functions, the annotations go in the function signature, not in the function body. The lifetime annotations become part of the contract of the function, much like the types in the signature. Having function signatures contain the lifetime contract means the analysis the Rust compiler does can be simpler. If there’s a problem with the way a function is annotated or the way it is called, the compiler errors can point to the part of our code and the constraints more precisely. If, instead, the Rust compiler made more inferences about what we intended the relationships of the lifetimes to be, the compiler might only be able to point to a use of our code many steps away from the cause of the problem.

When we pass concrete references to `longest`, the concrete lifetime that is substituted for `'a` is the part of the scope of `x` that overlaps with the scope of `y`. In other words, the generic lifetime `'a` will get the concrete lifetime that is equal to the smaller of the lifetimes of `x` and `y`. Because we’ve annotated the returned reference with the same lifetime parameter `'a`, the returned reference will also be valid for the length of the smaller of the lifetimes of `x` and `y`.

Let’s look at how the lifetime annotations restrict the `longest` function by passing in references that have different concrete lifetimes. Listing 10-22 is a straightforward example.

Filename: src/main.rs

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

Listing 10-22: Using the `longest` function with references to `String` values that have different concrete lifetimes

In this example, `string1` is valid until the end of the outer scope, `string2` is valid until the end of the inner scope, and `result` references something that is valid until the end of the inner scope. Run this code, and you’ll see that the borrow checker approves; it will compile and print `The longest string is long string is long`.

Next, let’s try an example that shows that the lifetime of the reference in `result` must be the smaller lifetime of the two arguments. We’ll move the declaration of the `result` variable outside the inner scope but leave the assignment of the value to the `result` variable inside the scope with `string2`. Then we’ll move the `println!` that uses `result` to outside the inner scope, after the inner scope has ended. The code in Listing 10-23 will not compile.

Filename: src/main.rs

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

Listing 10-23: Attempting to use `result` after `string2` has gone out of scope

When we try to compile this code, we get this error:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

The error shows that for `result` to be valid for the `println!` statement, `string2` would need to be valid until the end of the outer scope. Rust knows this because we annotated the lifetimes of the function parameters and return values using the same lifetime parameter `'a`.

As humans, we can look at this code and see that `string1` is longer than `string2` and therefore `result` will contain a reference to `string1`. Because `string1` has not gone out of scope yet, a reference to `string1` will still be valid for the `println!` statement. However, the compiler can’t see that the reference is valid in this case. We’ve told Rust that the lifetime of the reference returned by the `longest` function is the same as the smaller of the lifetimes of the references passed in. Therefore, the borrow checker disallows the code in Listing 10-23 as possibly having an invalid reference.

Try designing more experiments that vary the values and lifetimes of the references passed in to the `longest` function and how the returned reference is used. Make hypotheses about whether or not your experiments will pass the borrow checker before you compile; then check to see if you’re right!

### [Thinking in Terms of Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#thinking-in-terms-of-lifetimes)

The way in which you need to specify lifetime parameters depends on what your function is doing. For example, if we changed the implementation of the `longest` function to always return the first parameter rather than the longest string slice, we wouldn’t need to specify a lifetime on the `y` parameter. The following code will compile:

Filename: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

We’ve specified a lifetime parameter `'a` for the parameter `x` and the return type, but not for the parameter `y`, because the lifetime of `y` does not have any relationship with the lifetime of `x` or the return value.

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters. If the reference returned does *not* refer to one of the parameters, it must refer to a value created within this function. However, this would be a dangling reference because the value will go out of scope at the end of the function. Consider this attempted implementation of the `longest` function that won’t compile:

Filename: src/main.rs

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Here, even though we’ve specified a lifetime parameter `'a` for the return type, this implementation will fail to compile because the return value lifetime is not related to the lifetime of the parameters at all. Here is the error message we get:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
```

The problem is that `result` goes out of scope and gets cleaned up at the end of the `longest` function. We’re also trying to return a reference to `result` from the function. There is no way we can specify lifetime parameters that would change the dangling reference, and Rust won’t let us create a dangling reference. In this case, the best fix would be to return an owned data type rather than a reference so the calling function is then responsible for cleaning up the value.

Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions. Once they’re connected, Rust has enough information to allow memory-safe operations and disallow operations that would create dangling pointers or otherwise violate memory safety.

### [Lifetime Annotations in Struct Definitions](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-struct-definitions)

So far, the structs we’ve defined all hold owned types. We can define structs to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition. Listing 10-24 has a struct named `ImportantExcerpt` that holds a string slice.

Filename: src/main.rs

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

Listing 10-24: A struct that holds a reference, requiring a lifetime annotation

This struct has the single field `part` that holds a string slice, which is a reference. As with generic data types, we declare the name of the generic lifetime parameter inside angle brackets after the name of the struct so we can use the lifetime parameter in the body of the struct definition. This annotation means an instance of `ImportantExcerpt` can’t outlive the reference it holds in its `part` field.

The `main` function here creates an instance of the `ImportantExcerpt` struct that holds a reference to the first sentence of the `String` owned by the variable `novel`. The data in `novel` exists before the `ImportantExcerpt` instance is created. In addition, `novel` doesn’t go out of scope until after the `ImportantExcerpt` goes out of scope, so the reference in the `ImportantExcerpt` instance is valid.

### [Lifetime Elision](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision)

You’ve learned that every reference has a lifetime and that you need to specify lifetime parameters for functions or structs that use references. However, in Chapter 4 we had a function in Listing 4-9, shown again in Listing 10-25, that compiled without lifetime annotations.

Filename: src/lib.rs

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Listing 10-25: A function we defined in Listing 4-9 that compiled without lifetime annotations, even though the parameter and return type are references

The reason this function compiles without lifetime annotations is historical: in early versions (pre-1.0) of Rust, this code wouldn’t have compiled because every reference needed an explicit lifetime. At that time, the function signature would have been written like this:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

After writing a lot of Rust code, the Rust team found that Rust programmers were entering the same lifetime annotations over and over in particular situations. These situations were predictable and followed a few deterministic patterns. The developers programmed these patterns into the compiler’s code so the borrow checker could infer the lifetimes in these situations and wouldn’t need explicit annotations.

This piece of Rust history is relevant because it’s possible that more deterministic patterns will emerge and be added to the compiler. In the future, even fewer lifetime annotations might be required.

The patterns programmed into Rust’s analysis of references are called the *lifetime elision rules*. These aren’t rules for programmers to follow; they’re a set of particular cases that the compiler will consider, and if your code fits these cases, you don’t need to write the lifetimes explicitly.

The elision rules don’t provide full inference. If Rust deterministically applies the rules but there is still ambiguity as to what lifetimes the references have, the compiler won’t guess what the lifetime of the remaining references should be. Instead of guessing, the compiler will give you an error that you can resolve by adding the lifetime annotations.

Lifetimes on function or method parameters are called *input lifetimes*, and lifetimes on return values are called *output lifetimes*.

The compiler uses three rules to figure out the lifetimes of the references when there aren’t explicit annotations. The first rule applies to input lifetimes, and the second and third rules apply to output lifetimes. If the compiler gets to the end of the three rules and there are still references for which it can’t figure out lifetimes, the compiler will stop with an error. These rules apply to `fn` definitions as well as `impl` blocks.

The first rule is that the compiler assigns a lifetime parameter to each parameter that’s a reference. In other words, a function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32)`; a function with two parameters gets two separate lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; and so on.

The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.

The third rule is that, if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters. This third rule makes methods much nicer to read and write because fewer symbols are necessary.

Let’s pretend we’re the compiler. We’ll apply these rules to figure out the lifetimes of the references in the signature of the `first_word` function in Listing 10-25. The signature starts without any lifetimes associated with the references:

```rust
fn first_word(s: &str) -> &str {
```

Then the compiler applies the first rule, which specifies that each parameter gets its own lifetime. We’ll call it `'a` as usual, so now the signature is this:

```rust
fn first_word<'a>(s: &'a str) -> &str {
```

The second rule applies because there is exactly one input lifetime. The second rule specifies that the lifetime of the one input parameter gets assigned to the output lifetime, so the signature is now this:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Now all the references in this function signature have lifetimes, and the compiler can continue its analysis without needing the programmer to annotate the lifetimes in this function signature.

Let’s look at another example, this time using the `longest` function that had no lifetime parameters when we started working with it in Listing 10-20:

```rust
fn longest(x: &str, y: &str) -> &str {
```

Let’s apply the first rule: each parameter gets its own lifetime. This time we have two parameters instead of one, so we have two lifetimes:

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

You can see that the second rule doesn’t apply because there is more than one input lifetime. The third rule doesn’t apply either, because `longest` is a function rather than a method, so none of the parameters are `self`. After working through all three rules, we still haven’t figured out what the return type’s lifetime is. This is why we got an error trying to compile the code in Listing 10-20: the compiler worked through the lifetime elision rules but still couldn’t figure out all the lifetimes of the references in the signature.

Because the third rule really only applies in method signatures, we’ll look at lifetimes in that context next to see why the third rule means we don’t have to annotate lifetimes in method signatures very often.

### [Lifetime Annotations in Method Definitions](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-method-definitions)

When we implement methods on a struct with lifetimes, we use the same syntax as that of generic type parameters shown in Listing 10-11. Where we declare and use the lifetime parameters depends on whether they’re related to the struct fields or the method parameters and return values.

Lifetime names for struct fields always need to be declared after the `impl` keyword and then used after the struct’s name, because those lifetimes are part of the struct’s type.

In method signatures inside the `impl` block, references might be tied to the lifetime of references in the struct’s fields, or they might be independent. In addition, the lifetime elision rules often make it so that lifetime annotations aren’t necessary in method signatures. Let’s look at some examples using the struct named `ImportantExcerpt` that we defined in Listing 10-24.

First, we’ll use a method named `level` whose only parameter is a reference to `self` and whose return value is an `i32`, which is not a reference to anything:

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

The lifetime parameter declaration after `impl` and its use after the type name are required, but we’re not required to annotate the lifetime of the reference to `self` because of the first elision rule.

Here is an example where the third lifetime elision rule applies:

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

There are two input lifetimes, so Rust applies the first lifetime elision rule and gives both `&self` and `announcement` their own lifetimes. Then, because one of the parameters is `&self`, the return type gets the lifetime of `&self`, and all lifetimes have been accounted for.

### [The Static Lifetime](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-static-lifetime)

One special lifetime we need to discuss is `'static`, which denotes that the affected reference *can* live for the entire duration of the program. All string literals have the `'static` lifetime, which we can annotate as follows:

```rust
let s: &'static str = "I have a static lifetime.";
```

The text of this string is stored directly in the program’s binary, which is always available. Therefore, the lifetime of all string literals is `'static`.

You might see suggestions to use the `'static` lifetime in error messages. But before specifying `'static` as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to. Most of the time, an error message suggesting the `'static` lifetime results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is fixing those problems, not specifying the `'static` lifetime.

## [Generic Type Parameters, Trait Bounds, and Lifetimes Together](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-type-parameters-trait-bounds-and-lifetimes-together)

Let’s briefly look at the syntax of specifying generic type parameters, trait bounds, and lifetimes all in one function!

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
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

This is the `longest` function from Listing 10-21 that returns the longer of two string slices. But now it has an extra parameter named `ann` of the generic type `T`, which can be filled in by any type that implements the `Display` trait as specified by the `where` clause. This extra parameter will be printed using `{}`, which is why the `Display` trait bound is necessary. Because lifetimes are a type of generic, the declarations of the lifetime parameter `'a` and the generic type parameter `T` go in the same list inside the angle brackets after the function name.

## [Summary](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#summary)

We covered a lot in this chapter! Now that you know about generic type parameters, traits and trait bounds, and generic lifetime parameters, you’re ready to write code without repetition that works in many different situations. Generic type parameters let you apply the code to different types. Traits and trait bounds ensure that even though the types are generic, they’ll have the behavior the code needs. You learned how to use lifetime annotations to ensure that this flexible code won’t have any dangling references. And all of this analysis happens at compile time, which doesn’t affect runtime performance!

Believe it or not, there is much more to learn on the topics we discussed in this chapter: Chapter 17 discusses trait objects, which are another way to use traits. There are also more complex scenarios involving lifetime annotations that you will only need in very advanced scenarios; for those, you should read the [Rust Reference](https://doc.rust-lang.org/reference/index.html). But next, you’ll learn how to write tests in Rust so you can make sure your code is working the way it should.







---















