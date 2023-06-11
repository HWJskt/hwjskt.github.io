+++
title = "18-21"
weight = 180
sort_by = "weight"
template ="book_section_rust.html"

+++





# 18

# [Patterns and Matching](https://doc.rust-lang.org/book/ch18-00-patterns.html#patterns-and-matching)

*Patterns* are a special syntax in Rust for matching against the structure of types, both complex and simple. Using patterns in conjunction with "match" expressions and other constructs gives you more control over a program’s control flow. A pattern consists of some combination of the following:

- Literals
- Destructured arrays, enums, structs, or tuples
- Variables
- Wildcards
- Placeholders

Some example patterns include "x", "(a, 3)", and "Some(Color::Red)". In the contexts in which patterns are valid, these components describe the shape of data. Our program then matches values against the patterns to determine whether it has the correct shape of data to continue running a particular piece of code.

To use a pattern, we compare it to some value. If the pattern matches the value, we use the value parts in our code. Recall the "match" expressions in Chapter 6 that used patterns, such as the coin-sorting machine example. If the value fits the shape of the pattern, we can use the named pieces. If it doesn’t, the code associated with the pattern won’t run.

This chapter is a reference on all things related to patterns. We’ll cover the valid places to use patterns, the difference between refutable and irrefutable patterns, and the different kinds of pattern syntax that you might see. By the end of the chapter, you’ll know how to use patterns to express many concepts in a clear way.



---





## [All the Places Patterns Can Be Used](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#all-the-places-patterns-can-be-used)

Patterns pop up in a number of places in Rust, and you’ve been using them a lot without realizing it! This section discusses all the places where patterns are valid.

### ["match" Arms](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#match-arms)

As discussed in Chapter 6, we use patterns in the arms of "match" expressions. Formally, "match" expressions are defined as the keyword "match", a value to match on, and one or more match arms that consist of a pattern and an expression to run if the value matches that arm’s pattern, like this:

```
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

For example, here"s the "match" expression from Listing 6-5 that matches on an "Option<i32>" value in the variable "x":

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

The patterns in this "match" expression are the "None" and "Some(i)" on the left of each arrow.

One requirement for "match" expressions is that they need to be *exhaustive* in the sense that all possibilities for the value in the "match" expression must be accounted for. One way to ensure you’ve covered every possibility is to have a catchall pattern for the last arm: for example, a variable name matching any value can never fail and thus covers every remaining case.

The particular pattern "_" will match anything, but it never binds to a variable, so it’s often used in the last match arm. The "_" pattern can be useful when you want to ignore any value not specified, for example. We’ll cover the "_" pattern in more detail in the [“Ignoring Values in a Pattern”](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern) section later in this chapter.

### [Conditional "if let" Expressions](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#conditional-if-let-expressions)

In Chapter 6 we discussed how to use "if let" expressions mainly as a shorter way to write the equivalent of a "match" that only matches one case. Optionally, "if let" can have a corresponding "else" containing code to run if the pattern in the "if let" doesn’t match.

Listing 18-1 shows that it’s also possible to mix and match "if let", "else if", and "else if let" expressions. Doing so gives us more flexibility than a "match" expression in which we can express only one value to compare with the patterns. Also, Rust doesn"t require that the conditions in a series of "if let", "else if", "else if let" arms relate to each other.

The code in Listing 18-1 determines what color to make your background based on a series of checks for several conditions. For this example, we’ve created variables with hardcoded values that a real program might receive from user input.

Filename: src/main.rs

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

Listing 18-1: Mixing "if let", "else if", "else if let", and "else"

If the user specifies a favorite color, that color is used as the background. If no favorite color is specified and today is Tuesday, the background color is green. Otherwise, if the user specifies their age as a string and we can parse it as a number successfully, the color is either purple or orange depending on the value of the number. If none of these conditions apply, the background color is blue.

This conditional structure lets us support complex requirements. With the hardcoded values we have here, this example will print "Using purple as the background color".

You can see that "if let" can also introduce shadowed variables in the same way that "match" arms can: the line "if let Ok(age) = age" introduces a new shadowed "age" variable that contains the value inside the "Ok" variant. This means we need to place the "if age > 30" condition within that block: we can’t combine these two conditions into "if let Ok(age) = age && age > 30". The shadowed "age" we want to compare to 30 isn’t valid until the new scope starts with the curly bracket.

The downside of using "if let" expressions is that the compiler doesn’t check for exhaustiveness, whereas with "match" expressions it does. If we omitted the last "else" block and therefore missed handling some cases, the compiler would not alert us to the possible logic bug.

### ["while let" Conditional Loops](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#while-let-conditional-loops)

Similar in construction to "if let", the "while let" conditional loop allows a "while" loop to run for as long as a pattern continues to match. In Listing 18-2 we code a "while let" loop that uses a vector as a stack and prints the values in the vector in the opposite order in which they were pushed.

```rust
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
```

Listing 18-2: Using a "while let" loop to print values for as long as "stack.pop()" returns "Some"

This example prints 3, 2, and then 1. The "pop" method takes the last element out of the vector and returns "Some(value)". If the vector is empty, "pop" returns "None". The "while" loop continues running the code in its block as long as "pop" returns "Some". When "pop" returns "None", the loop stops. We can use "while let" to pop every element off our stack.

### ["for" Loops](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#for-loops)

In a "for" loop, the value that directly follows the keyword "for" is a pattern. For example, in "for x in y" the "x" is the pattern. Listing 18-3 demonstrates how to use a pattern in a "for" loop to destructure, or break apart, a tuple as part of the "for" loop.

```rust
    let v = vec!["a", "b", "c"];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
```

Listing 18-3: Using a pattern in a "for" loop to destructure a tuple

The code in Listing 18-3 will print the following:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running "target/debug/patterns"
a is at index 0
b is at index 1
c is at index 2
```

We adapt an iterator using the "enumerate" method so it produces a value and the index for that value, placed into a tuple. The first value produced is the tuple "(0, "a")". When this value is matched to the pattern "(index, value)", "index" will be "0" and "value" will be ""a"", printing the first line of the output.

### ["let" Statements](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#let-statements)

Prior to this chapter, we had only explicitly discussed using patterns with "match" and "if let", but in fact, we’ve used patterns in other places as well, including in "let" statements. For example, consider this straightforward variable assignment with "let":

```rust
let x = 5;
```

Every time you"ve used a "let" statement like this you"ve been using patterns, although you might not have realized it! More formally, a "let" statement looks like this:

```
let PATTERN = EXPRESSION;
```

In statements like "let x = 5;" with a variable name in the "PATTERN" slot, the variable name is just a particularly simple form of a pattern. Rust compares the expression against the pattern and assigns any names it finds. So in the "let x = 5;" example, "x" is a pattern that means “bind what matches here to the variable "x".” Because the name "x" is the whole pattern, this pattern effectively means “bind everything to the variable "x", whatever the value is.”

To see the pattern matching aspect of "let" more clearly, consider Listing 18-4, which uses a pattern with "let" to destructure a tuple.

```rust
    let (x, y, z) = (1, 2, 3);
```

Listing 18-4: Using a pattern to destructure a tuple and create three variables at once

Here, we match a tuple against a pattern. Rust compares the value "(1, 2, 3)" to the pattern "(x, y, z)" and sees that the value matches the pattern, so Rust binds "1" to "x", "2" to "y", and "3" to "z". You can think of this tuple pattern as nesting three individual variable patterns inside it.

If the number of elements in the pattern doesn’t match the number of elements in the tuple, the overall type won’t match and we’ll get a compiler error. For example, Listing 18-5 shows an attempt to destructure a tuple with three elements into two variables, which won’t work.

```rust
    let (x, y) = (1, 2, 3);
```

Listing 18-5: Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple

Attempting to compile this code results in this type error:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type "({integer}, {integer}, {integer})"
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple "({integer}, {integer}, {integer})"
             found tuple "(_, _)"

For more information about this error, try "rustc --explain E0308".
error: could not compile "patterns" due to previous error
```

To fix the error, we could ignore one or more of the values in the tuple using "_" or "..", as you’ll see in the [“Ignoring Values in a Pattern”](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern) section. If the problem is that we have too many variables in the pattern, the solution is to make the types match by removing variables so the number of variables equals the number of elements in the tuple.

### [Function Parameters](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#function-parameters)

Function parameters can also be patterns. The code in Listing 18-6, which declares a function named "foo" that takes one parameter named "x" of type "i32", should by now look familiar.

```rust
fn foo(x: i32) {
    // code goes here
}
```

Listing 18-6: A function signature uses patterns in the parameters

The "x" part is a pattern! As we did with "let", we could match a tuple in a function’s arguments to the pattern. Listing 18-7 splits the values in a tuple as we pass it to a function.

Filename: src/main.rs

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

Listing 18-7: A function with parameters that destructure a tuple

This code prints "Current location: (3, 5)". The values "&(3, 5)" match the pattern "&(x, y)", so "x" is the value "3" and "y" is the value "5".

We can also use patterns in closure parameter lists in the same way as in function parameter lists, because closures are similar to functions, as discussed in Chapter 13.

At this point, you’ve seen several ways of using patterns, but patterns don’t work the same in every place we can use them. In some places, the patterns must be irrefutable; in other circumstances, they can be refutable. We’ll discuss these two concepts next.





---





## [Refutability: Whether a Pattern Might Fail to Match](https://doc.rust-lang.org/book/ch18-02-refutability.html#refutability-whether-a-pattern-might-fail-to-match)

Patterns come in two forms: refutable and irrefutable. Patterns that will match for any possible value passed are *irrefutable*. An example would be "x" in the statement "let x = 5;" because "x" matches anything and therefore cannot fail to match. Patterns that can fail to match for some possible value are *refutable*. An example would be "Some(x)" in the expression "if let Some(x) = a_value" because if the value in the "a_value" variable is "None" rather than "Some", the "Some(x)" pattern will not match.

Function parameters, "let" statements, and "for" loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match. The "if let" and "while let" expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns because by definition they’re intended to handle possible failure: the functionality of a conditional is in its ability to perform differently depending on success or failure.

In general, you shouldn’t have to worry about the distinction between refutable and irrefutable patterns; however, you do need to be familiar with the concept of refutability so you can respond when you see it in an error message. In those cases, you’ll need to change either the pattern or the construct you’re using the pattern with, depending on the intended behavior of the code.

Let’s look at an example of what happens when we try to use a refutable pattern where Rust requires an irrefutable pattern and vice versa. Listing 18-8 shows a "let" statement, but for the pattern we’ve specified "Some(x)", a refutable pattern. As you might expect, this code will not compile.

```rust
    let Some(x) = some_option_value;
```

Listing 18-8: Attempting to use a refutable pattern with "let"

If "some_option_value" was a "None" value, it would fail to match the pattern "Some(x)", meaning the pattern is refutable. However, the "let" statement can only accept an irrefutable pattern because there is nothing valid the code can do with a "None" value. At compile time, Rust will complain that we’ve tried to use a refutable pattern where an irrefutable pattern is required:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: "None" not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern "None" not covered
  |
  = note: "let" bindings require an "irrefutable pattern", like a "struct" or an "enum" with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
note: "Option<i32>" defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type "Option<i32>"
help: you might want to use "if let" to ignore the variant that isn"t matched
  |
3 |     let x = if let Some(x) = some_option_value { x } else { todo!() };
  |     ++++++++++                                 ++++++++++++++++++++++
help: alternatively, you might want to use let else to handle the variant that isn"t matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try "rustc --explain E0005".
error: could not compile "patterns" due to previous error
```

Because we didn’t cover (and couldn’t cover!) every valid value with the pattern "Some(x)", Rust rightfully produces a compiler error.

If we have a refutable pattern where an irrefutable pattern is needed, we can fix it by changing the code that uses the pattern: instead of using "let", we can use "if let". Then if the pattern doesn’t match, the code will just skip the code in the curly brackets, giving it a way to continue validly. Listing 18-9 shows how to fix the code in Listing 18-8.

```rust
    if let Some(x) = some_option_value {
        println!("{}", x);
    }
```

Listing 18-9: Using "if let" and a block with refutable patterns instead of "let"

We’ve given the code an out! This code is perfectly valid, although it means we cannot use an irrefutable pattern without receiving an error. If we give "if let" a pattern that will always match, such as "x", as shown in Listing 18-10, the compiler will give a warning.

```rust
    if let x = 5 {
        println!("{}", x);
    };
```

Listing 18-10: Attempting to use an irrefutable pattern with "if let"

Rust complains that it doesn’t make sense to use "if let" with an irrefutable pattern:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable "if let" pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the "if let" is useless
  = help: consider replacing the "if let" with a "let"
  = note: "#[warn(irrefutable_let_patterns)]" on by default

warning: "patterns" (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running "target/debug/patterns"
5
```

For this reason, match arms must use refutable patterns, except for the last arm, which should match any remaining values with an irrefutable pattern. Rust allows us to use an irrefutable pattern in a "match" with only one arm, but this syntax isn’t particularly useful and could be replaced with a simpler "let" statement.

Now that you know where to use patterns and the difference between refutable and irrefutable patterns, let’s cover all the syntax we can use to create patterns.





---





## [Pattern Syntax](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#pattern-syntax)

In this section, we gather all the syntax valid in patterns and discuss why and when you might want to use each one.

### [Matching Literals](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-literals)

As you saw in Chapter 6, you can match patterns against literals directly. The following code gives some examples:

```rust
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
```

This code prints "one" because the value in "x" is 1. This syntax is useful when you want your code to take an action if it gets a particular concrete value.

### [Matching Named Variables](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-named-variables)

Named variables are irrefutable patterns that match any value, and we’ve used them many times in the book. However, there is a complication when you use named variables in "match" expressions. Because "match" starts a new scope, variables declared as part of a pattern inside the "match" expression will shadow those with the same name outside the "match" construct, as is the case with all variables. In Listing 18-11, we declare a variable named "x" with the value "Some(5)" and a variable "y" with the value "10". We then create a "match" expression on the value "x". Look at the patterns in the match arms and "println!" at the end, and try to figure out what the code will print before running this code or reading further.

Filename: src/main.rs

```rust
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }
    
    println!("at the end: x = {:?}, y = {y}", x);
```

Listing 18-11: A "match" expression with an arm that introduces a shadowed variable "y"

Let’s walk through what happens when the "match" expression runs. The pattern in the first match arm doesn’t match the defined value of "x", so the code continues.

The pattern in the second match arm introduces a new variable named "y" that will match any value inside a "Some" value. Because we’re in a new scope inside the "match" expression, this is a new "y" variable, not the "y" we declared at the beginning with the value 10. This new "y" binding will match any value inside a "Some", which is what we have in "x". Therefore, this new "y" binds to the inner value of the "Some" in "x". That value is "5", so the expression for that arm executes and prints "Matched, y = 5".

If "x" had been a "None" value instead of "Some(5)", the patterns in the first two arms wouldn’t have matched, so the value would have matched to the underscore. We didn’t introduce the "x" variable in the pattern of the underscore arm, so the "x" in the expression is still the outer "x" that hasn’t been shadowed. In this hypothetical case, the "match" would print "Default case, x = None".

When the "match" expression is done, its scope ends, and so does the scope of the inner "y". The last "println!" produces "at the end: x = Some(5), y = 10".

To create a "match" expression that compares the values of the outer "x" and "y", rather than introducing a shadowed variable, we would need to use a match guard conditional instead. We’ll talk about match guards later in the [“Extra Conditionals with Match Guards”](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards) section.

### [Multiple Patterns](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#multiple-patterns)

In "match" expressions, you can match multiple patterns using the "|" syntax, which is the pattern *or* operator. For example, in the following code we match the value of "x" against the match arms, the first of which has an *or* option, meaning if the value of "x" matches either of the values in that arm, that arm’s code will run:

```rust
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
```

This code prints "one or two".

### [Matching Ranges of Values with "..="](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-ranges-of-values-with-)

The "..=" syntax allows us to match to an inclusive range of values. In the following code, when a pattern matches any of the values within the given range, that arm will execute:

```rust
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
```

If "x" is 1, 2, 3, 4, or 5, the first arm will match. This syntax is more convenient for multiple match values than using the "|" operator to express the same idea; if we were to use "|" we would have to specify "1 | 2 | 3 | 4 | 5". Specifying a range is much shorter, especially if we want to match, say, any number between 1 and 1,000!

The compiler checks that the range isn’t empty at compile time, and because the only types for which Rust can tell if a range is empty or not are "char" and numeric values, ranges are only allowed with numeric or "char" values.

Here is an example using ranges of "char" values:

```rust
    let x = "c";

    match x {
        "a"..="j" => println!("early ASCII letter"),
        "k"..="z" => println!("late ASCII letter"),
        _ => println!("something else"),
    }
```

Rust can tell that ""c"" is within the first pattern’s range and prints "early ASCII letter".

### [Destructuring to Break Apart Values](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-to-break-apart-values)

We can also use patterns to destructure structs, enums, and tuples to use different parts of these values. Let’s walk through each value.

#### [Destructuring Structs](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs)

Listing 18-12 shows a "Point" struct with two fields, "x" and "y", that we can break apart using a pattern with a "let" statement.

Filename: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

Listing 18-12: Destructuring a struct’s fields into separate variables

This code creates the variables "a" and "b" that match the values of the "x" and "y" fields of the "p" struct. This example shows that the names of the variables in the pattern don’t have to match the field names of the struct. However, it’s common to match the variable names to the field names to make it easier to remember which variables came from which fields. Because of this common usage, and because writing "let Point { x: x, y: y } = p;" contains a lot of duplication, Rust has a shorthand for patterns that match struct fields: you only need to list the name of the struct field, and the variables created from the pattern will have the same names. Listing 18-13 behaves in the same way as the code in Listing 18-12, but the variables created in the "let" pattern are "x" and "y" instead of "a" and "b".

Filename: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

Listing 18-13: Destructuring struct fields using struct field shorthand

This code creates the variables "x" and "y" that match the "x" and "y" fields of the "p" variable. The outcome is that the variables "x" and "y" contain the values from the "p" struct.

We can also destructure with literal values as part of the struct pattern rather than creating variables for all the fields. Doing so allows us to test some of the fields for particular values while creating variables to destructure the other fields.

In Listing 18-14, we have a "match" expression that separates "Point" values into three cases: points that lie directly on the "x" axis (which is true when "y = 0"), on the "y" axis ("x = 0"), or neither.

Filename: src/main.rs

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

Listing 18-14: Destructuring and matching literal values in one pattern

The first arm will match any point that lies on the "x" axis by specifying that the "y" field matches if its value matches the literal "0". The pattern still creates an "x" variable that we can use in the code for this arm.

Similarly, the second arm matches any point on the "y" axis by specifying that the "x" field matches if its value is "0" and creates a variable "y" for the value of the "y" field. The third arm doesn’t specify any literals, so it matches any other "Point" and creates variables for both the "x" and "y" fields.

In this example, the value "p" matches the second arm by virtue of "x" containing a 0, so this code will print "On the y axis at 7".

Remember that a "match" expression stops checking arms once it has found the first matching pattern, so even though "Point { x: 0, y: 0}" is on the "x" axis and the "y" axis, this code would only print "On the x axis at 0".

#### [Destructuring Enums](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-enums)

We"ve destructured enums in this book (for example, Listing 6-5 in Chapter 6), but haven’t yet explicitly discussed that the pattern to destructure an enum corresponds to the way the data stored within the enum is defined. As an example, in Listing 18-15 we use the "Message" enum from Listing 6-2 and write a "match" with patterns that will destructure each inner value.

Filename: src/main.rs

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
```

Listing 18-15: Destructuring enum variants that hold different kinds of values

This code will print "Change the color to red 0, green 160, and blue 255". Try changing the value of "msg" to see the code from the other arms run.

For enum variants without any data, like "Message::Quit", we can’t destructure the value any further. We can only match on the literal "Message::Quit" value, and no variables are in that pattern.

For struct-like enum variants, such as "Message::Move", we can use a pattern similar to the pattern we specify to match structs. After the variant name, we place curly brackets and then list the fields with variables so we break apart the pieces to use in the code for this arm. Here we use the shorthand form as we did in Listing 18-13.

For tuple-like enum variants, like "Message::Write" that holds a tuple with one element and "Message::ChangeColor" that holds a tuple with three elements, the pattern is similar to the pattern we specify to match tuples. The number of variables in the pattern must match the number of elements in the variant we’re matching.

#### [Destructuring Nested Structs and Enums](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-nested-structs-and-enums)

So far, our examples have all been matching structs or enums one level deep, but matching can work on nested items too! For example, we can refactor the code in Listing 18-15 to support RGB and HSV colors in the "ChangeColor" message, as shown in Listing 18-16.

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

Listing 18-16: Matching on nested enums

The pattern of the first arm in the "match" expression matches a "Message::ChangeColor" enum variant that contains a "Color::Rgb" variant; then the pattern binds to the three inner "i32" values. The pattern of the second arm also matches a "Message::ChangeColor" enum variant, but the inner enum matches "Color::Hsv" instead. We can specify these complex conditions in one "match" expression, even though two enums are involved.

#### [Destructuring Structs and Tuples](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs-and-tuples)

We can mix, match, and nest destructuring patterns in even more complex ways. The following example shows a complicated destructure where we nest structs and tuples inside a tuple and destructure all the primitive values out:

```rust
    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

This code lets us break complex types into their component parts so we can use the values we’re interested in separately.

Destructuring with patterns is a convenient way to use pieces of values, such as the value from each field in a struct, separately from each other.

### [Ignoring Values in a Pattern](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern)

You’ve seen that it’s sometimes useful to ignore values in a pattern, such as in the last arm of a "match", to get a catchall that doesn’t actually do anything but does account for all remaining possible values. There are a few ways to ignore entire values or parts of values in a pattern: using the "_" pattern (which you’ve seen), using the "_" pattern within another pattern, using a name that starts with an underscore, or using ".." to ignore remaining parts of a value. Let’s explore how and why to use each of these patterns.

#### [Ignoring an Entire Value with "_"](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-entire-value-with-_)

We’ve used the underscore as a wildcard pattern that will match any value but not bind to the value. This is especially useful as the last arm in a "match" expression, but we can also use it in any pattern, including function parameters, as shown in Listing 18-17.

Filename: src/main.rs

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

Listing 18-17: Using "_" in a function signature

This code will completely ignore the value "3" passed as the first argument, and will print "This code only uses the y parameter: 4".

In most cases when you no longer need a particular function parameter, you would change the signature so it doesn’t include the unused parameter. Ignoring a function parameter can be especially useful in cases when, for example, you"re implementing a trait when you need a certain type signature but the function body in your implementation doesn’t need one of the parameters. You then avoid getting a compiler warning about unused function parameters, as you would if you used a name instead.

#### [Ignoring Parts of a Value with a Nested "_"](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-parts-of-a-value-with-a-nested-_)

We can also use "_" inside another pattern to ignore just part of a value, for example, when we want to test for only part of a value but have no use for the other parts in the corresponding code we want to run. Listing 18-18 shows code responsible for managing a setting’s value. The business requirements are that the user should not be allowed to overwrite an existing customization of a setting but can unset the setting and give it a value if it is currently unset.

```rust
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can"t overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }
    
    println!("setting is {:?}", setting_value);
```

Listing 18-18: Using an underscore within patterns that match "Some" variants when we don’t need to use the value inside the "Some"

This code will print "Can"t overwrite an existing customized value" and then "setting is Some(5)". In the first match arm, we don’t need to match on or use the values inside either "Some" variant, but we do need to test for the case when "setting_value" and "new_setting_value" are the "Some" variant. In that case, we print the reason for not changing "setting_value", and it doesn’t get changed.

In all other cases (if either "setting_value" or "new_setting_value" are "None") expressed by the "_" pattern in the second arm, we want to allow "new_setting_value" to become "setting_value".

We can also use underscores in multiple places within one pattern to ignore particular values. Listing 18-19 shows an example of ignoring the second and fourth values in a tuple of five items.

```rust
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}")
        }
    }
```

Listing 18-19: Ignoring multiple parts of a tuple

This code will print "Some numbers: 2, 8, 32", and the values 4 and 16 will be ignored.

#### [Ignoring an Unused Variable by Starting Its Name with "_"](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-unused-variable-by-starting-its-name-with-_)

If you create a variable but don’t use it anywhere, Rust will usually issue a warning because an unused variable could be a bug. However, sometimes it’s useful to be able to create a variable you won’t use yet, such as when you’re prototyping or just starting a project. In this situation, you can tell Rust not to warn you about the unused variable by starting the name of the variable with an underscore. In Listing 18-20, we create two unused variables, but when we compile this code, we should only get a warning about one of them.

Filename: src/main.rs

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

Listing 18-20: Starting a variable name with an underscore to avoid getting unused variable warnings

Here we get a warning about not using the variable "y", but we don’t get a warning about not using "_x".

Note that there is a subtle difference between using only "_" and using a name that starts with an underscore. The syntax "_x" still binds the value to the variable, whereas "_" doesn’t bind at all. To show a case where this distinction matters, Listing 18-21 will provide us with an error.

```rust
    let s = Some(String::from("Hello!"));

    if let Some(_s) = s {
        println!("found a string");
    }
    
    println!("{:?}", s);
```

Listing 18-21: An unused variable starting with an underscore still binds the value, which might take ownership of the value

We’ll receive an error because the "s" value will still be moved into "_s", which prevents us from using "s" again. However, using the underscore by itself doesn’t ever bind to the value. Listing 18-22 will compile without any errors because "s" doesn’t get moved into "_".

```rust
    let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }
    
    println!("{:?}", s);
```

Listing 18-22: Using an underscore does not bind the value

This code works just fine because we never bind "s" to anything; it isn’t moved.

#### [Ignoring Remaining Parts of a Value with ".."](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-remaining-parts-of-a-value-with-)

With values that have many parts, we can use the ".." syntax to use specific parts and ignore the rest, avoiding the need to list underscores for each ignored value. The ".." pattern ignores any parts of a value that we haven’t explicitly matched in the rest of the pattern. In Listing 18-23, we have a "Point" struct that holds a coordinate in three-dimensional space. In the "match" expression, we want to operate only on the "x" coordinate and ignore the values in the "y" and "z" fields.

```rust
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }

    let origin = Point { x: 0, y: 0, z: 0 };
    
    match origin {
        Point { x, .. } => println!("x is {}", x),
    }
```

Listing 18-23: Ignoring all fields of a "Point" except for "x" by using ".."

We list the "x" value and then just include the ".." pattern. This is quicker than having to list "y: _" and "z: _", particularly when we’re working with structs that have lots of fields in situations where only one or two fields are relevant.

The syntax ".." will expand to as many values as it needs to be. Listing 18-24 shows how to use ".." with a tuple.

Filename: src/main.rs

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

Listing 18-24: Matching only the first and last values in a tuple and ignoring all other values

In this code, the first and last value are matched with "first" and "last". The ".." will match and ignore everything in the middle.

However, using ".." must be unambiguous. If it is unclear which values are intended for matching and which should be ignored, Rust will give us an error. Listing 18-25 shows an example of using ".." ambiguously, so it will not compile.

Filename: src/main.rs

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}
```

Listing 18-25: An attempt to use ".." in an ambiguous way

When we compile this example, we get this error:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: ".." can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile "patterns" due to previous error
```

It’s impossible for Rust to determine how many values in the tuple to ignore before matching a value with "second" and then how many further values to ignore thereafter. This code could mean that we want to ignore "2", bind "second" to "4", and then ignore "8", "16", and "32"; or that we want to ignore "2" and "4", bind "second" to "8", and then ignore "16" and "32"; and so forth. The variable name "second" doesn’t mean anything special to Rust, so we get a compiler error because using ".." in two places like this is ambiguous.

### [Extra Conditionals with Match Guards](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards)

A *match guard* is an additional "if" condition, specified after the pattern in a "match" arm, that must also match for that arm to be chosen. Match guards are useful for expressing more complex ideas than a pattern alone allows.

The condition can use variables created in the pattern. Listing 18-26 shows a "match" where the first arm has the pattern "Some(x)" and also has a match guard of "if x % 2 == 0" (which will be true if the number is even).

```rust
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {} is even", x),
        Some(x) => println!("The number {} is odd", x),
        None => (),
    }
```

Listing 18-26: Adding a match guard to a pattern

This example will print "The number 4 is even". When "num" is compared to the pattern in the first arm, it matches, because "Some(4)" matches "Some(x)". Then the match guard checks whether the remainder of dividing "x" by 2 is equal to 0, and because it is, the first arm is selected.

If "num" had been "Some(5)" instead, the match guard in the first arm would have been false because the remainder of 5 divided by 2 is 1, which is not equal to 0. Rust would then go to the second arm, which would match because the second arm doesn’t have a match guard and therefore matches any "Some" variant.

There is no way to express the "if x % 2 == 0" condition within a pattern, so the match guard gives us the ability to express this logic. The downside of this additional expressiveness is that the compiler doesn"t try to check for exhaustiveness when match guard expressions are involved.

In Listing 18-11, we mentioned that we could use match guards to solve our pattern-shadowing problem. Recall that we created a new variable inside the pattern in the "match" expression instead of using the variable outside the "match". That new variable meant we couldn’t test against the value of the outer variable. Listing 18-27 shows how we can use a match guard to fix this problem.

Filename: src/main.rs

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }
    
    println!("at the end: x = {:?}, y = {y}", x);
}
```

Listing 18-27: Using a match guard to test for equality with an outer variable

This code will now print "Default case, x = Some(5)". The pattern in the second match arm doesn’t introduce a new variable "y" that would shadow the outer "y", meaning we can use the outer "y" in the match guard. Instead of specifying the pattern as "Some(y)", which would have shadowed the outer "y", we specify "Some(n)". This creates a new variable "n" that doesn’t shadow anything because there is no "n" variable outside the "match".

The match guard "if n == y" is not a pattern and therefore doesn’t introduce new variables. This "y" *is* the outer "y" rather than a new shadowed "y", and we can look for a value that has the same value as the outer "y" by comparing "n" to "y".

You can also use the *or* operator "|" in a match guard to specify multiple patterns; the match guard condition will apply to all the patterns. Listing 18-28 shows the precedence when combining a pattern that uses "|" with a match guard. The important part of this example is that the "if y" match guard applies to "4", "5", *and* "6", even though it might look like "if y" only applies to "6".

```rust
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
```

Listing 18-28: Combining multiple patterns with a match guard

The match condition states that the arm only matches if the value of "x" is equal to "4", "5", or "6" *and* if "y" is "true". When this code runs, the pattern of the first arm matches because "x" is "4", but the match guard "if y" is false, so the first arm is not chosen. The code moves on to the second arm, which does match, and this program prints "no". The reason is that the "if" condition applies to the whole pattern "4 | 5 | 6", not only to the last value "6". In other words, the precedence of a match guard in relation to a pattern behaves like this:

```
(4 | 5 | 6) if y => ...
```

rather than this:

```
4 | 5 | (6 if y) => ...
```

After running the code, the precedence behavior is evident: if the match guard were applied only to the final value in the list of values specified using the "|" operator, the arm would have matched and the program would have printed "yes".

### ["@" Bindings](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#-bindings)

The *at* operator "@" lets us create a variable that holds a value at the same time as we’re testing that value for a pattern match. In Listing 18-29, we want to test that a "Message::Hello" "id" field is within the range "3..=7". We also want to bind the value to the variable "id_variable" so we can use it in the code associated with the arm. We could name this variable "id", the same as the field, but for this example we’ll use a different name.

```rust
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };
    
    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
```

Listing 18-29: Using "@" to bind to a value in a pattern while also testing it

This example will print "Found an id in range: 5". By specifying "id_variable @" before the range "3..=7", we’re capturing whatever value matched the range while also testing that the value matched the range pattern.

In the second arm, where we only have a range specified in the pattern, the code associated with the arm doesn’t have a variable that contains the actual value of the "id" field. The "id" field’s value could have been 10, 11, or 12, but the code that goes with that pattern doesn’t know which it is. The pattern code isn’t able to use the value from the "id" field, because we haven’t saved the "id" value in a variable.

In the last arm, where we’ve specified a variable without a range, we do have the value available to use in the arm’s code in a variable named "id". The reason is that we’ve used the struct field shorthand syntax. But we haven’t applied any test to the value in the "id" field in this arm, as we did with the first two arms: any value would match this pattern.

Using "@" lets us test a value and save it in a variable within one pattern.

## [Summary](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#summary)

Rust’s patterns are very useful in distinguishing between different kinds of data. When used in "match" expressions, Rust ensures your patterns cover every possible value, or your program won’t compile. Patterns in "let" statements and function parameters make those constructs more useful, enabling the destructuring of values into smaller parts at the same time as assigning to variables. We can create simple or complex patterns to suit our needs.

Next, for the penultimate chapter of the book, we’ll look at some advanced aspects of a variety of Rust’s features.





---





# 19



# [Advanced Features](https://doc.rust-lang.org/book/ch19-00-advanced-features.html#advanced-features)

By now, you’ve learned the most commonly used parts of the Rust programming language. Before we do one more project in Chapter 20, we’ll look at a few aspects of the language you might run into every once in a while, but may not use every day. You can use this chapter as a reference for when you encounter any unknowns. The features covered here are useful in very specific situations. Although you might not reach for them often, we want to make sure you have a grasp of all the features Rust has to offer.

In this chapter, we’ll cover:

- Unsafe Rust: how to opt out of some of Rust’s guarantees and take responsibility for manually upholding those guarantees
- Advanced traits: associated types, default type parameters, fully qualified syntax, supertraits, and the newtype pattern in relation to traits
- Advanced types: more about the newtype pattern, type aliases, the never type, and dynamically sized types
- Advanced functions and closures: function pointers and returning closures
- Macros: ways to define code that defines more code at compile time

It’s a panoply of Rust features with something for everyone! Let’s dive in!





---





## [Unsafe Rust](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#unsafe-rust)

All the code we’ve discussed so far has had Rust’s memory safety guarantees enforced at compile time. However, Rust has a second language hidden inside it that doesn’t enforce these memory safety guarantees: it’s called *unsafe Rust* and works just like regular Rust, but gives us extra superpowers.

Unsafe Rust exists because, by nature, static analysis is conservative. When the compiler tries to determine whether or not code upholds the guarantees, it’s better for it to reject some valid programs than to accept some invalid programs. Although the code *might* be okay, if the Rust compiler doesn’t have enough information to be confident, it will reject the code. In these cases, you can use unsafe code to tell the compiler, “Trust me, I know what I’m doing.” Be warned, however, that you use unsafe Rust at your own risk: if you use unsafe code incorrectly, problems can occur due to memory unsafety, such as null pointer dereferencing.

Another reason Rust has an unsafe alter ego is that the underlying computer hardware is inherently unsafe. If Rust didn’t let you do unsafe operations, you couldn’t do certain tasks. Rust needs to allow you to do low-level systems programming, such as directly interacting with the operating system or even writing your own operating system. Working with low-level systems programming is one of the goals of the language. Let’s explore what we can do with unsafe Rust and how to do it.

### [Unsafe Superpowers](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#unsafe-superpowers)

To switch to unsafe Rust, use the "unsafe" keyword and then start a new block that holds the unsafe code. You can take five actions in unsafe Rust that you can’t in safe Rust, which we call *unsafe superpowers*. Those superpowers include the ability to:

- Dereference a raw pointer
- Call an unsafe function or method
- Access or modify a mutable static variable
- Implement an unsafe trait
- Access fields of "union"s

It’s important to understand that "unsafe" doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use a reference in unsafe code, it will still be checked. The "unsafe" keyword only gives you access to these five features that are then not checked by the compiler for memory safety. You’ll still get some degree of safety inside of an unsafe block.

In addition, "unsafe" does not mean the code inside the block is necessarily dangerous or that it will definitely have memory safety problems: the intent is that as the programmer, you’ll ensure the code inside an "unsafe" block will access memory in a valid way.

People are fallible, and mistakes will happen, but by requiring these five unsafe operations to be inside blocks annotated with "unsafe" you’ll know that any errors related to memory safety must be within an "unsafe" block. Keep "unsafe" blocks small; you’ll be thankful later when you investigate memory bugs.

To isolate unsafe code as much as possible, it’s best to enclose unsafe code within a safe abstraction and provide a safe API, which we’ll discuss later in the chapter when we examine unsafe functions and methods. Parts of the standard library are implemented as safe abstractions over unsafe code that has been audited. Wrapping unsafe code in a safe abstraction prevents uses of "unsafe" from leaking out into all the places that you or your users might want to use the functionality implemented with "unsafe" code, because using a safe abstraction is safe.

Let’s look at each of the five unsafe superpowers in turn. We’ll also look at some abstractions that provide a safe interface to unsafe code.

### [Dereferencing a Raw Pointer](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer)

In Chapter 4, in the [“Dangling References”](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references) section, we mentioned that the compiler ensures references are always valid. Unsafe Rust has two new types called *raw pointers* that are similar to references. As with references, raw pointers can be immutable or mutable and are written as "*const T" and "*mut T", respectively. The asterisk isn’t the dereference operator; it’s part of the type name. In the context of raw pointers, *immutable* means that the pointer can’t be directly assigned to after being dereferenced.

Different from references and smart pointers, raw pointers:

- Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
- Aren’t guaranteed to point to valid memory
- Are allowed to be null
- Don’t implement any automatic cleanup

By opting out of having Rust enforce these guarantees, you can give up guaranteed safety in exchange for greater performance or the ability to interface with another language or hardware where Rust’s guarantees don’t apply.

Listing 19-1 shows how to create an immutable and a mutable raw pointer from references.

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
```

Listing 19-1: Creating raw pointers from references

Notice that we don’t include the "unsafe" keyword in this code. We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block, as you’ll see in a bit.

We’ve created raw pointers by using "as" to cast an immutable and a mutable reference into their corresponding raw pointer types. Because we created them directly from references guaranteed to be valid, we know these particular raw pointers are valid, but we can’t make that assumption about just any raw pointer.

To demonstrate this, next we’ll create a raw pointer whose validity we can’t be so certain of. Listing 19-2 shows how to create a raw pointer to an arbitrary location in memory. Trying to use arbitrary memory is undefined: there might be data at that address or there might not, the compiler might optimize the code so there is no memory access, or the program might error with a segmentation fault. Usually, there is no good reason to write code like this, but it is possible.

```rust
    let address = 0x012345usize;
    let r = address as *const i32;
```

Listing 19-2: Creating a raw pointer to an arbitrary memory address

Recall that we can create raw pointers in safe code, but we can’t *dereference* raw pointers and read the data being pointed to. In Listing 19-3, we use the dereference operator "*" on a raw pointer that requires an "unsafe" block.

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
```

Listing 19-3: Dereferencing raw pointers within an "unsafe" block

Creating a pointer does no harm; it’s only when we try to access the value that it points at that we might end up dealing with an invalid value.

Note also that in Listing 19-1 and 19-3, we created "*const i32" and "*mut i32" raw pointers that both pointed to the same memory location, where "num" is stored. If we instead tried to create an immutable and a mutable reference to "num", the code would not have compiled because Rust’s ownership rules don’t allow a mutable reference at the same time as any immutable references. With raw pointers, we can create a mutable pointer and an immutable pointer to the same location and change data through the mutable pointer, potentially creating a data race. Be careful!

With all of these dangers, why would you ever use raw pointers? One major use case is when interfacing with C code, as you’ll see in the next section, [“Calling an Unsafe Function or Method.”](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-an-unsafe-function-or-method) Another case is when building up safe abstractions that the borrow checker doesn’t understand. We’ll introduce unsafe functions and then look at an example of a safe abstraction that uses unsafe code.

### [Calling an Unsafe Function or Method](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-an-unsafe-function-or-method)

The second type of operation you can perform in an unsafe block is calling unsafe functions. Unsafe functions and methods look exactly like regular functions and methods, but they have an extra "unsafe" before the rest of the definition. The "unsafe" keyword in this context indicates the function has requirements we need to uphold when we call this function, because Rust can’t guarantee we’ve met these requirements. By calling an unsafe function within an "unsafe" block, we’re saying that we’ve read this function’s documentation and take responsibility for upholding the function’s contracts.

Here is an unsafe function named "dangerous" that doesn’t do anything in its body:

```rust
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
```

We must call the "dangerous" function within a separate "unsafe" block. If we try to call "dangerous" without the "unsafe" block, we’ll get an error:

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function is unsafe and requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function"s documentation for information on how to avoid undefined behavior

For more information about this error, try "rustc --explain E0133".
error: could not compile "unsafe-example" due to previous error
```

With the "unsafe" block, we’re asserting to Rust that we’ve read the function’s documentation, we understand how to use it properly, and we’ve verified that we’re fulfilling the contract of the function.

Bodies of unsafe functions are effectively "unsafe" blocks, so to perform other unsafe operations within an unsafe function, we don’t need to add another "unsafe" block.

#### [Creating a Safe Abstraction over Unsafe Code](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#creating-a-safe-abstraction-over-unsafe-code)

Just because a function contains unsafe code doesn’t mean we need to mark the entire function as unsafe. In fact, wrapping unsafe code in a safe function is a common abstraction. As an example, let’s study the "split_at_mut" function from the standard library, which requires some unsafe code. We’ll explore how we might implement it. This safe method is defined on mutable slices: it takes one slice and makes it two by splitting the slice at the index given as an argument. Listing 19-4 shows how to use "split_at_mut".

```rust
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];
    
    let (a, b) = r.split_at_mut(3);
    
    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
```

Listing 19-4: Using the safe "split_at_mut" function

We can’t implement this function using only safe Rust. An attempt might look something like Listing 19-5, which won’t compile. For simplicity, we’ll implement "split_at_mut" as a function rather than a method and only for slices of "i32" values rather than for a generic type "T".

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);
    
    (&mut values[..mid], &mut values[mid..])
}
```

Listing 19-5: An attempted implementation of "split_at_mut" using only safe Rust

This function first gets the total length of the slice. Then it asserts that the index given as a parameter is within the slice by checking whether it’s less than or equal to the length. The assertion means that if we pass an index that is greater than the length to split the slice at, the function will panic before it attempts to use that index.

Then we return two mutable slices in a tuple: one from the start of the original slice to the "mid" index and another from "mid" to the end of the slice.

When we try to compile the code in Listing 19-5, we’ll get an error.

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow "*values" as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let"s call the lifetime of this reference ""1"
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that "*values" is borrowed for ""1"

For more information about this error, try "rustc --explain E0499".
error: could not compile "unsafe-example" due to previous error
```

Rust’s borrow checker can’t understand that we’re borrowing different parts of the slice; it only knows that we’re borrowing from the same slice twice. Borrowing different parts of a slice is fundamentally okay because the two slices aren’t overlapping, but Rust isn’t smart enough to know this. When we know code is okay, but Rust doesn’t, it’s time to reach for unsafe code.

Listing 19-6 shows how to use an "unsafe" block, a raw pointer, and some calls to unsafe functions to make the implementation of "split_at_mut" work.

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);
    
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

Listing 19-6: Using unsafe code in the implementation of the "split_at_mut" function

Recall from [“The Slice Type”](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type) section in Chapter 4 that slices are a pointer to some data and the length of the slice. We use the "len" method to get the length of a slice and the "as_mut_ptr" method to access the raw pointer of a slice. In this case, because we have a mutable slice to "i32" values, "as_mut_ptr" returns a raw pointer with the type "*mut i32", which we’ve stored in the variable "ptr".

We keep the assertion that the "mid" index is within the slice. Then we get to the unsafe code: the "slice::from_raw_parts_mut" function takes a raw pointer and a length, and it creates a slice. We use this function to create a slice that starts from "ptr" and is "mid" items long. Then we call the "add" method on "ptr" with "mid" as an argument to get a raw pointer that starts at "mid", and we create a slice using that pointer and the remaining number of items after "mid" as the length.

The function "slice::from_raw_parts_mut" is unsafe because it takes a raw pointer and must trust that this pointer is valid. The "add" method on raw pointers is also unsafe, because it must trust that the offset location is also a valid pointer. Therefore, we had to put an "unsafe" block around our calls to "slice::from_raw_parts_mut" and "add" so we could call them. By looking at the code and by adding the assertion that "mid" must be less than or equal to "len", we can tell that all the raw pointers used within the "unsafe" block will be valid pointers to data within the slice. This is an acceptable and appropriate use of "unsafe".

Note that we don’t need to mark the resulting "split_at_mut" function as "unsafe", and we can call this function from safe Rust. We’ve created a safe abstraction to the unsafe code with an implementation of the function that uses "unsafe" code in a safe way, because it creates only valid pointers from the data this function has access to.

In contrast, the use of "slice::from_raw_parts_mut" in Listing 19-7 would likely crash when the slice is used. This code takes an arbitrary memory location and creates a slice 10,000 items long.

```rust
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;
    
    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
```

Listing 19-7: Creating a slice from an arbitrary memory location

We don’t own the memory at this arbitrary location, and there is no guarantee that the slice this code creates contains valid "i32" values. Attempting to use "values" as though it’s a valid slice results in undefined behavior.

#### [Using "extern" Functions to Call External Code](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code)

Sometimes, your Rust code might need to interact with code written in another language. For this, Rust has the keyword "extern" that facilitates the creation and use of a *Foreign Function Interface (FFI)*. An FFI is a way for a programming language to define functions and enable a different (foreign) programming language to call those functions.

Listing 19-8 demonstrates how to set up an integration with the "abs" function from the C standard library. Functions declared within "extern" blocks are always unsafe to call from Rust code. The reason is that other languages don’t enforce Rust’s rules and guarantees, and Rust can’t check them, so responsibility falls on the programmer to ensure safety.

Filename: src/main.rs

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

Listing 19-8: Declaring and calling an "extern" function defined in another language

Within the "extern "C"" block, we list the names and signatures of external functions from another language we want to call. The ""C"" part defines which *application binary interface (ABI)* the external function uses: the ABI defines how to call the function at the assembly level. The ""C"" ABI is the most common and follows the C programming language’s ABI.

> #### [Calling Rust Functions from Other Languages](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-rust-functions-from-other-languages)
>
> We can also use "extern" to create an interface that allows other languages to call Rust functions. Instead of creating a whole "extern" block, we add the "extern" keyword and specify the ABI to use just before the "fn" keyword for the relevant function. We also need to add a "#[no_mangle]" annotation to tell the Rust compiler not to mangle the name of this function. *Mangling* is when a compiler changes the name we’ve given a function to a different name that contains more information for other parts of the compilation process to consume but is less human readable. Every programming language compiler mangles names slightly differently, so for a Rust function to be nameable by other languages, we must disable the Rust compiler’s name mangling.
>
> In the following example, we make the "call_from_c" function accessible from C code, after it’s compiled to a shared library and linked from C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> This usage of "extern" does not require "unsafe".

### [Accessing or Modifying a Mutable Static Variable](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#accessing-or-modifying-a-mutable-static-variable)

In this book, we’ve not yet talked about *global variables*, which Rust does support but can be problematic with Rust’s ownership rules. If two threads are accessing the same mutable global variable, it can cause a data race.

In Rust, global variables are called *static* variables. Listing 19-9 shows an example declaration and use of a static variable with a string slice as a value.

Filename: src/main.rs

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

Listing 19-9: Defining and using an immutable static variable

Static variables are similar to constants, which we discussed in the [“Differences Between Variables and Constants”](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#constants) section in Chapter 3. The names of static variables are in "SCREAMING_SNAKE_CASE" by convention. Static variables can only store references with the ""static" lifetime, which means the Rust compiler can figure out the lifetime and we aren’t required to annotate it explicitly. Accessing an immutable static variable is safe.

A subtle difference between constants and immutable static variables is that values in a static variable have a fixed address in memory. Using the value will always access the same data. Constants, on the other hand, are allowed to duplicate their data whenever they’re used. Another difference is that static variables can be mutable. Accessing and modifying mutable static variables is *unsafe*. Listing 19-10 shows how to declare, access, and modify a mutable static variable named "COUNTER".

Filename: src/main.rs

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

Listing 19-10: Reading from or writing to a mutable static variable is unsafe

As with regular variables, we specify mutability using the "mut" keyword. Any code that reads or writes from "COUNTER" must be within an "unsafe" block. This code compiles and prints "COUNTER: 3" as we would expect because it’s single threaded. Having multiple threads access "COUNTER" would likely result in data races.

With mutable data that is globally accessible, it’s difficult to ensure there are no data races, which is why Rust considers mutable static variables to be unsafe. Where possible, it’s preferable to use the concurrency techniques and thread-safe smart pointers we discussed in Chapter 16 so the compiler checks that data accessed from different threads is done safely.

### [Implementing an Unsafe Trait](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#implementing-an-unsafe-trait)

We can use "unsafe" to implement an unsafe trait. A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify. We declare that a trait is "unsafe" by adding the "unsafe" keyword before "trait" and marking the implementation of the trait as "unsafe" too, as shown in Listing 19-11.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
```

Listing 19-11: Defining and implementing an unsafe trait

By using "unsafe impl", we’re promising that we’ll uphold the invariants that the compiler can’t verify.

As an example, recall the "Sync" and "Send" marker traits we discussed in the [“Extensible Concurrency with the "Sync" and "Send" Traits”](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits) section in Chapter 16: the compiler implements these traits automatically if our types are composed entirely of "Send" and "Sync" types. If we implement a type that contains a type that is not "Send" or "Sync", such as raw pointers, and we want to mark that type as "Send" or "Sync", we must use "unsafe". Rust can’t verify that our type upholds the guarantees that it can be safely sent across threads or accessed from multiple threads; therefore, we need to do those checks manually and indicate as such with "unsafe".

### [Accessing Fields of a Union](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#accessing-fields-of-a-union)

The final action that works only with "unsafe" is accessing fields of a *union*. A "union" is similar to a "struct", but only one declared field is used in a particular instance at one time. Unions are primarily used to interface with unions in C code. Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance. You can learn more about unions in [the Rust Reference](https://doc.rust-lang.org/reference/items/unions.html).

### [When to Use Unsafe Code](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#when-to-use-unsafe-code)

Using "unsafe" to take one of the five actions (superpowers) just discussed isn’t wrong or even frowned upon. But it is trickier to get "unsafe" code correct because the compiler can’t help uphold memory safety. When you have a reason to use "unsafe" code, you can do so, and having the explicit "unsafe" annotation makes it easier to track down the source of problems when they occur.





---





## [Advanced Traits](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#advanced-traits)

We first covered traits in the [“Traits: Defining Shared Behavior”](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior) section of Chapter 10, but we didn’t discuss the more advanced details. Now that you know more about Rust, we can get into the nitty-gritty.

### [Specifying Placeholder Types in Trait Definitions with Associated Types](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)

*Associated types* connect a type placeholder with a trait such that the trait method definitions can use these placeholder types in their signatures. The implementor of a trait will specify the concrete type to be used instead of the placeholder type for the particular implementation. That way, we can define a trait that uses some types without needing to know exactly what those types are until the trait is implemented.

We’ve described most of the advanced features in this chapter as being rarely needed. Associated types are somewhere in the middle: they’re used more rarely than features explained in the rest of the book but more commonly than many of the other features discussed in this chapter.

One example of a trait with an associated type is the "Iterator" trait that the standard library provides. The associated type is named "Item" and stands in for the type of the values the type implementing the "Iterator" trait is iterating over. The definition of the "Iterator" trait is as shown in Listing 19-12.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

Listing 19-12: The definition of the "Iterator" trait that has an associated type "Item"

The type "Item" is a placeholder, and the "next" method’s definition shows that it will return values of type "Option<Self::Item>". Implementors of the "Iterator" trait will specify the concrete type for "Item", and the "next" method will return an "Option" containing a value of that concrete type.

Associated types might seem like a similar concept to generics, in that the latter allow us to define a function without specifying what types it can handle. To examine the difference between the two concepts, we’ll look at an implementation of the "Iterator" trait on a type named "Counter" that specifies the "Item" type is "u32":

Filename: src/lib.rs

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

This syntax seems comparable to that of generics. So why not just define the "Iterator" trait with generics, as shown in Listing 19-13?

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

Listing 19-13: A hypothetical definition of the "Iterator" trait using generics

The difference is that when using generics, as in Listing 19-13, we must annotate the types in each implementation; because we can also implement "Iterator<String> for Counter" or any other type, we could have multiple implementations of "Iterator" for "Counter". In other words, when a trait has a generic parameter, it can be implemented for a type multiple times, changing the concrete types of the generic type parameters each time. When we use the "next" method on "Counter", we would have to provide type annotations to indicate which implementation of "Iterator" we want to use.

With associated types, we don’t need to annotate types because we can’t implement a trait on a type multiple times. In Listing 19-12 with the definition that uses associated types, we can only choose what the type of "Item" will be once, because there can only be one "impl Iterator for Counter". We don’t have to specify that we want an iterator of "u32" values everywhere that we call "next" on "Counter".

Associated types also become part of the trait’s contract: implementors of the trait must provide a type to stand in for the associated type placeholder. Associated types often have a name that describes how the type will be used, and documenting the associated type in the API documentation is good practice.

### [Default Generic Type Parameters and Operator Overloading](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#default-generic-type-parameters-and-operator-overloading)

When we use generic type parameters, we can specify a default concrete type for the generic type. This eliminates the need for implementors of the trait to specify a concrete type if the default type works. You specify a default type when declaring a generic type with the "<PlaceholderType=ConcreteType>" syntax.

A great example of a situation where this technique is useful is with *operator overloading*, in which you customize the behavior of an operator (such as "+") in particular situations.

Rust doesn’t allow you to create your own operators or overload arbitrary operators. But you can overload the operations and corresponding traits listed in "std::ops" by implementing the traits associated with the operator. For example, in Listing 19-14 we overload the "+" operator to add two "Point" instances together. We do this by implementing the "Add" trait on a "Point" struct:

Filename: src/main.rs

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

Listing 19-14: Implementing the "Add" trait to overload the "+" operator for "Point" instances

The "add" method adds the "x" values of two "Point" instances and the "y" values of two "Point" instances to create a new "Point". The "Add" trait has an associated type named "Output" that determines the type returned from the "add" method.

The default generic type in this code is within the "Add" trait. Here is its definition:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

This code should look generally familiar: a trait with one method and an associated type. The new part is "Rhs=Self": this syntax is called *default type parameters*. The "Rhs" generic type parameter (short for “right hand side”) defines the type of the "rhs" parameter in the "add" method. If we don’t specify a concrete type for "Rhs" when we implement the "Add" trait, the type of "Rhs" will default to "Self", which will be the type we’re implementing "Add" on.

When we implemented "Add" for "Point", we used the default for "Rhs" because we wanted to add two "Point" instances. Let’s look at an example of implementing the "Add" trait where we want to customize the "Rhs" type rather than using the default.

We have two structs, "Millimeters" and "Meters", holding values in different units. This thin wrapping of an existing type in another struct is known as the *newtype pattern*, which we describe in more detail in the [“Using the Newtype Pattern to Implement External Traits on External Types”](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types) section. We want to add values in millimeters to values in meters and have the implementation of "Add" do the conversion correctly. We can implement "Add" for "Millimeters" with "Meters" as the "Rhs", as shown in Listing 19-15.

Filename: src/lib.rs

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

Listing 19-15: Implementing the "Add" trait on "Millimeters" to add "Millimeters" to "Meters"

To add "Millimeters" and "Meters", we specify "impl Add<Meters>" to set the value of the "Rhs" type parameter instead of using the default of "Self".

You’ll use default type parameters in two main ways:

- To extend a type without breaking existing code
- To allow customization in specific cases most users won’t need

The standard library’s "Add" trait is an example of the second purpose: usually, you’ll add two like types, but the "Add" trait provides the ability to customize beyond that. Using a default type parameter in the "Add" trait definition means you don’t have to specify the extra parameter most of the time. In other words, a bit of implementation boilerplate isn’t needed, making it easier to use the trait.

The first purpose is similar to the second but in reverse: if you want to add a type parameter to an existing trait, you can give it a default to allow extension of the functionality of the trait without breaking the existing implementation code.

### [Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name)

Nothing in Rust prevents a trait from having a method with the same name as another trait’s method, nor does Rust prevent you from implementing both traits on one type. It’s also possible to implement a method directly on the type with the same name as methods from traits.

When calling methods with the same name, you’ll need to tell Rust which one you want to use. Consider the code in Listing 19-16 where we’ve defined two traits, "Pilot" and "Wizard", that both have a method called "fly". We then implement both traits on a type "Human" that already has a method named "fly" implemented on it. Each "fly" method does something different.

Filename: src/main.rs

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

Listing 19-16: Two traits are defined to have a "fly" method and are implemented on the "Human" type, and a "fly" method is implemented on "Human" directly

When we call "fly" on an instance of "Human", the compiler defaults to calling the method that is directly implemented on the type, as shown in Listing 19-17.

Filename: src/main.rs

```rust
fn main() {
    let person = Human;
    person.fly();
}
```

Listing 19-17: Calling "fly" on an instance of "Human"

Running this code will print "*waving arms furiously*", showing that Rust called the "fly" method implemented on "Human" directly.

To call the "fly" methods from either the "Pilot" trait or the "Wizard" trait, we need to use more explicit syntax to specify which "fly" method we mean. Listing 19-18 demonstrates this syntax.

Filename: src/main.rs

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

Listing 19-18: Specifying which trait’s "fly" method we want to call

Specifying the trait name before the method name clarifies to Rust which implementation of "fly" we want to call. We could also write "Human::fly(&person)", which is equivalent to the "person.fly()" that we used in Listing 19-18, but this is a bit longer to write if we don’t need to disambiguate.

Running this code prints the following:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running "target/debug/traits-example"
This is your captain speaking.
Up!
*waving arms furiously*
```

Because the "fly" method takes a "self" parameter, if we had two *types* that both implement one *trait*, Rust could figure out which implementation of a trait to use based on the type of "self".

However, associated functions that are not methods don’t have a "self" parameter. When there are multiple types or traits that define non-method functions with the same function name, Rust doesn"t always know which type you mean unless you use *fully qualified syntax*. For example, in Listing 19-19 we create a trait for an animal shelter that wants to name all baby dogs *Spot*. We make an "Animal" trait with an associated non-method function "baby_name". The "Animal" trait is implemented for the struct "Dog", on which we also provide an associated non-method function "baby_name" directly.

Filename: src/main.rs

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

Listing 19-19: A trait with an associated function and a type with an associated function of the same name that also implements the trait

We implement the code for naming all puppies Spot in the "baby_name" associated function that is defined on "Dog". The "Dog" type also implements the trait "Animal", which describes characteristics that all animals have. Baby dogs are called puppies, and that is expressed in the implementation of the "Animal" trait on "Dog" in the "baby_name" function associated with the "Animal" trait.

In "main", we call the "Dog::baby_name" function, which calls the associated function defined on "Dog" directly. This code prints the following:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.54s
     Running "target/debug/traits-example"
A baby dog is called a Spot
```

This output isn’t what we wanted. We want to call the "baby_name" function that is part of the "Animal" trait that we implemented on "Dog" so the code prints "A baby dog is called a puppy". The technique of specifying the trait name that we used in Listing 19-18 doesn’t help here; if we change "main" to the code in Listing 19-20, we’ll get a compilation error.

Filename: src/main.rs

```rust
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```

Listing 19-20: Attempting to call the "baby_name" function from the "Animal" trait, but Rust doesn’t know which implementation to use

Because "Animal::baby_name" doesn’t have a "self" parameter, and there could be other types that implement the "Animal" trait, Rust can’t figure out which implementation of "Animal::baby_name" we want. We’ll get this compiler error:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding "impl" type
  --> src/main.rs:20:43
   |
2  |     fn baby_name() -> String;
   |     ------------------------- "Animal::baby_name" defined here
...
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
   |                                           +++++++       +

For more information about this error, try "rustc --explain E0790".
error: could not compile "traits-example" due to previous error
```

To disambiguate and tell Rust that we want to use the implementation of "Animal" for "Dog" as opposed to the implementation of "Animal" for some other type, we need to use fully qualified syntax. Listing 19-21 demonstrates how to use fully qualified syntax.

Filename: src/main.rs

```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

Listing 19-21: Using fully qualified syntax to specify that we want to call the "baby_name" function from the "Animal" trait as implemented on "Dog"

We’re providing Rust with a type annotation within the angle brackets, which indicates we want to call the "baby_name" method from the "Animal" trait as implemented on "Dog" by saying that we want to treat the "Dog" type as an "Animal" for this function call. This code will now print what we want:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running "target/debug/traits-example"
A baby dog is called a puppy
```

In general, fully qualified syntax is defined as follows:

```rust
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

For associated functions that aren’t methods, there would not be a "receiver": there would only be the list of other arguments. You could use fully qualified syntax everywhere that you call functions or methods. However, you’re allowed to omit any part of this syntax that Rust can figure out from other information in the program. You only need to use this more verbose syntax in cases where there are multiple implementations that use the same name and Rust needs help to identify which implementation you want to call.

### [Using Supertraits to Require One Trait’s Functionality Within Another Trait](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-supertraits-to-require-one-traits-functionality-within-another-trait)

Sometimes, you might write a trait definition that depends on another trait: for a type to implement the first trait, you want to require that type to also implement the second trait. You would do this so that your trait definition can make use of the associated items of the second trait. The trait your trait definition is relying on is called a *supertrait* of your trait.

For example, let’s say we want to make an "OutlinePrint" trait with an "outline_print" method that will print a given value formatted so that it"s framed in asterisks. That is, given a "Point" struct that implements the standard library trait "Display" to result in "(x, y)", when we call "outline_print" on a "Point" instance that has "1" for "x" and "3" for "y", it should print the following:

```
**********
*        *
* (1, 3) *
*        *
**********
```

In the implementation of the "outline_print" method, we want to use the "Display" trait’s functionality. Therefore, we need to specify that the "OutlinePrint" trait will work only for types that also implement "Display" and provide the functionality that "OutlinePrint" needs. We can do that in the trait definition by specifying "OutlinePrint: Display". This technique is similar to adding a trait bound to the trait. Listing 19-22 shows an implementation of the "OutlinePrint" trait.

Filename: src/main.rs

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

Listing 19-22: Implementing the "OutlinePrint" trait that requires the functionality from "Display"

Because we’ve specified that "OutlinePrint" requires the "Display" trait, we can use the "to_string" function that is automatically implemented for any type that implements "Display". If we tried to use "to_string" without adding a colon and specifying the "Display" trait after the trait name, we’d get an error saying that no method named "to_string" was found for the type "&Self" in the current scope.

Let’s see what happens when we try to implement "OutlinePrint" on a type that doesn’t implement "Display", such as the "Point" struct:

Filename: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
```

We get an error saying that "Display" is required but not implemented:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0277]: "Point" doesn"t implement "std::fmt::Display"
  --> src/main.rs:20:6
   |
20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ "Point" cannot be formatted with the default formatter
   |
   = help: the trait "std::fmt::Display" is not implemented for "Point"
   = note: in format strings you may be able to use "{:?}" (or {:#?} for pretty-print) instead
note: required by a bound in "OutlinePrint"
  --> src/main.rs:3:21
   |
3  | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in "OutlinePrint"

For more information about this error, try "rustc --explain E0277".
error: could not compile "traits-example" due to previous error
```

To fix this, we implement "Display" on "Point" and satisfy the constraint that "OutlinePrint" requires, like so:

Filename: src/main.rs

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

Then implementing the "OutlinePrint" trait on "Point" will compile successfully, and we can call "outline_print" on a "Point" instance to display it within an outline of asterisks.

### [Using the Newtype Pattern to Implement External Traits on External Types](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)

In Chapter 10 in the [“Implementing a Trait on a Type”](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type) section, we mentioned the orphan rule that states we’re only allowed to implement a trait on a type if either the trait or the type are local to our crate. It’s possible to get around this restriction using the *newtype pattern*, which involves creating a new type in a tuple struct. (We covered tuple structs in the [“Using Tuple Structs without Named Fields to Create Different Types”](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types) section of Chapter 5.) The tuple struct will have one field and be a thin wrapper around the type we want to implement a trait for. Then the wrapper type is local to our crate, and we can implement the trait on the wrapper. *Newtype* is a term that originates from the Haskell programming language. There is no runtime performance penalty for using this pattern, and the wrapper type is elided at compile time.

As an example, let’s say we want to implement "Display" on "Vec<T>", which the orphan rule prevents us from doing directly because the "Display" trait and the "Vec<T>" type are defined outside our crate. We can make a "Wrapper" struct that holds an instance of "Vec<T>"; then we can implement "Display" on "Wrapper" and use the "Vec<T>" value, as shown in Listing 19-23.

Filename: src/main.rs

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

Listing 19-23: Creating a "Wrapper" type around "Vec<String>" to implement "Display"

The implementation of "Display" uses "self.0" to access the inner "Vec<T>", because "Wrapper" is a tuple struct and "Vec<T>" is the item at index 0 in the tuple. Then we can use the functionality of the "Display" type on "Wrapper".

The downside of using this technique is that "Wrapper" is a new type, so it doesn’t have the methods of the value it’s holding. We would have to implement all the methods of "Vec<T>" directly on "Wrapper" such that the methods delegate to "self.0", which would allow us to treat "Wrapper" exactly like a "Vec<T>". If we wanted the new type to have every method the inner type has, implementing the "Deref" trait (discussed in Chapter 15 in the [“Treating Smart Pointers Like Regular References with the "Deref" Trait”](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait) section) on the "Wrapper" to return the inner type would be a solution. If we don’t want the "Wrapper" type to have all the methods of the inner type—for example, to restrict the "Wrapper" type’s behavior—we would have to implement just the methods we do want manually.

This newtype pattern is also useful even when traits are not involved. Let’s switch focus and look at some advanced ways to interact with Rust’s type system.





---





## [Advanced Types](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#advanced-types)

The Rust type system has some features that we’ve so far mentioned but haven’t yet discussed. We’ll start by discussing newtypes in general as we examine why newtypes are useful as types. Then we’ll move on to type aliases, a feature similar to newtypes but with slightly different semantics. We’ll also discuss the "!" type and dynamically sized types.

### [Using the Newtype Pattern for Type Safety and Abstraction](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction)

> Note: This section assumes you’ve read the earlier section [“Using the Newtype Pattern to Implement External Traits on External Types.”](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)

The newtype pattern is also useful for tasks beyond those we’ve discussed so far, including statically enforcing that values are never confused and indicating the units of a value. You saw an example of using newtypes to indicate units in Listing 19-15: recall that the "Millimeters" and "Meters" structs wrapped "u32" values in a newtype. If we wrote a function with a parameter of type "Millimeters", we couldn’t compile a program that accidentally tried to call that function with a value of type "Meters" or a plain "u32".

We can also use the newtype pattern to abstract away some implementation details of a type: the new type can expose a public API that is different from the API of the private inner type.

Newtypes can also hide internal implementation. For example, we could provide a "People" type to wrap a "HashMap<i32, String>" that stores a person’s ID associated with their name. Code using "People" would only interact with the public API we provide, such as a method to add a name string to the "People" collection; that code wouldn’t need to know that we assign an "i32" ID to names internally. The newtype pattern is a lightweight way to achieve encapsulation to hide implementation details, which we discussed in the [“Encapsulation that Hides Implementation Details”](https://doc.rust-lang.org/book/ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details) section of Chapter 17.

### [Creating Type Synonyms with Type Aliases](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases)

Rust provides the ability to declare a *type alias* to give an existing type another name. For this we use the "type" keyword. For example, we can create the alias "Kilometers" to "i32" like so:

```rust
    type Kilometers = i32;
```

Now, the alias "Kilometers" is a *synonym* for "i32"; unlike the "Millimeters" and "Meters" types we created in Listing 19-15, "Kilometers" is not a separate, new type. Values that have the type "Kilometers" will be treated the same as values of type "i32":

```rust
    type Kilometers = i32;

    let x: i32 = 5;
    let y: Kilometers = 5;
    
    println!("x + y = {}", x + y);
```

Because "Kilometers" and "i32" are the same type, we can add values of both types and we can pass "Kilometers" values to functions that take "i32" parameters. However, using this method, we don’t get the type checking benefits that we get from the newtype pattern discussed earlier. In other words, if we mix up "Kilometers" and "i32" values somewhere, the compiler will not give us an error.

The main use case for type synonyms is to reduce repetition. For example, we might have a lengthy type like this:

```rust
Box<dyn Fn() + Send + "static>
```

Writing this lengthy type in function signatures and as type annotations all over the code can be tiresome and error prone. Imagine having a project full of code like that in Listing 19-24.

```rust
    let f: Box<dyn Fn() + Send + "static> = Box::new(|| println!("hi"));

    fn takes_long_type(f: Box<dyn Fn() + Send + "static>) {
        // --snip--
    }
    
    fn returns_long_type() -> Box<dyn Fn() + Send + "static> {
        // --snip--
    }
```

Listing 19-24: Using a long type in many places

A type alias makes this code more manageable by reducing the repetition. In Listing 19-25, we’ve introduced an alias named "Thunk" for the verbose type and can replace all uses of the type with the shorter alias "Thunk".

```rust
    type Thunk = Box<dyn Fn() + Send + "static>;

    let f: Thunk = Box::new(|| println!("hi"));
    
    fn takes_long_type(f: Thunk) {
        // --snip--
    }
    
    fn returns_long_type() -> Thunk {
        // --snip--
    }
```

Listing 19-25: Introducing a type alias "Thunk" to reduce repetition

This code is much easier to read and write! Choosing a meaningful name for a type alias can help communicate your intent as well (*thunk* is a word for code to be evaluated at a later time, so it’s an appropriate name for a closure that gets stored).

Type aliases are also commonly used with the "Result<T, E>" type for reducing repetition. Consider the "std::io" module in the standard library. I/O operations often return a "Result<T, E>" to handle situations when operations fail to work. This library has a "std::io::Error" struct that represents all possible I/O errors. Many of the functions in "std::io" will be returning "Result<T, E>" where the "E" is "std::io::Error", such as these functions in the "Write" trait:

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

The "Result<..., Error>" is repeated a lot. As such, "std::io" has this type alias declaration:

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

Because this declaration is in the "std::io" module, we can use the fully qualified alias "std::io::Result<T>"; that is, a "Result<T, E>" with the "E" filled in as "std::io::Error". The "Write" trait function signatures end up looking like this:

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

The type alias helps in two ways: it makes code easier to write *and* it gives us a consistent interface across all of "std::io". Because it’s an alias, it’s just another "Result<T, E>", which means we can use any methods that work on "Result<T, E>" with it, as well as special syntax like the "?" operator.

### [The Never Type that Never Returns](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#the-never-type-that-never-returns)

Rust has a special type named "!" that’s known in type theory lingo as the *empty type* because it has no values. We prefer to call it the *never type* because it stands in the place of the return type when a function will never return. Here is an example:

```rust
fn bar() -> ! {
    // --snip--
}
```

This code is read as “the function "bar" returns never.” Functions that return never are called *diverging functions*. We can’t create values of the type "!" so "bar" can never possibly return.

But what use is a type you can never create values for? Recall the code from Listing 2-5, part of the number guessing game; we’ve reproduced a bit of it here in Listing 19-26.

```rust
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```

Listing 19-26: A "match" with an arm that ends in "continue"

At the time, we skipped over some details in this code. In Chapter 6 in [“The "match" Control Flow Operator”](https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-operator) section, we discussed that "match" arms must all return the same type. So, for example, the following code doesn’t work:

```rust
    let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => "hello",
    };
```

The type of "guess" in this code would have to be an integer *and* a string, and Rust requires that "guess" have only one type. So what does "continue" return? How were we allowed to return a "u32" from one arm and have another arm that ends with "continue" in Listing 19-26?

As you might have guessed, "continue" has a "!" value. That is, when Rust computes the type of "guess", it looks at both match arms, the former with a value of "u32" and the latter with a "!" value. Because "!" can never have a value, Rust decides that the type of "guess" is "u32".

The formal way of describing this behavior is that expressions of type "!" can be coerced into any other type. We’re allowed to end this "match" arm with "continue" because "continue" doesn’t return a value; instead, it moves control back to the top of the loop, so in the "Err" case, we never assign a value to "guess".

The never type is useful with the "panic!" macro as well. Recall the "unwrap" function that we call on "Option<T>" values to produce a value or panic with this definition:

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called "Option::unwrap()" on a "None" value"),
        }
    }
}
```

In this code, the same thing happens as in the "match" in Listing 19-26: Rust sees that "val" has the type "T" and "panic!" has the type "!", so the result of the overall "match" expression is "T". This code works because "panic!" doesn’t produce a value; it ends the program. In the "None" case, we won’t be returning a value from "unwrap", so this code is valid.

One final expression that has the type "!" is a "loop":

```rust
    print!("forever ");

    loop {
        print!("and ever ");
    }
```

Here, the loop never ends, so "!" is the value of the expression. However, this wouldn’t be true if we included a "break", because the loop would terminate when it got to the "break".

### [Dynamically Sized Types and the "Sized" Trait](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait)

Rust needs to know certain details about its types, such as how much space to allocate for a value of a particular type. This leaves one corner of its type system a little confusing at first: the concept of *dynamically sized types*. Sometimes referred to as *DSTs* or *unsized types*, these types let us write code using values whose size we can know only at runtime.

Let’s dig into the details of a dynamically sized type called "str", which we’ve been using throughout the book. That’s right, not "&str", but "str" on its own, is a DST. We can’t know how long the string is until runtime, meaning we can’t create a variable of type "str", nor can we take an argument of type "str". Consider the following code, which does not work:

```rust
    let s1: str = "Hello there!";
    let s2: str = "How"s it going?";
```

Rust needs to know how much memory to allocate for any value of a particular type, and all values of a type must use the same amount of memory. If Rust allowed us to write this code, these two "str" values would need to take up the same amount of space. But they have different lengths: "s1" needs 12 bytes of storage and "s2" needs 15. This is why it’s not possible to create a variable holding a dynamically sized type.

So what do we do? In this case, you already know the answer: we make the types of "s1" and "s2" a "&str" rather than a "str". Recall from the [“String Slices”](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices) section of Chapter 4 that the slice data structure just stores the starting position and the length of the slice. So although a "&T" is a single value that stores the memory address of where the "T" is located, a "&str" is *two* values: the address of the "str" and its length. As such, we can know the size of a "&str" value at compile time: it’s twice the length of a "usize". That is, we always know the size of a "&str", no matter how long the string it refers to is. In general, this is the way in which dynamically sized types are used in Rust: they have an extra bit of metadata that stores the size of the dynamic information. The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a pointer of some kind.

We can combine "str" with all kinds of pointers: for example, "Box<str>" or "Rc<str>". In fact, you’ve seen this before but with a different dynamically sized type: traits. Every trait is a dynamically sized type we can refer to by using the name of the trait. In Chapter 17 in the [“Using Trait Objects That Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) section, we mentioned that to use traits as trait objects, we must put them behind a pointer, such as "&dyn Trait" or "Box<dyn Trait>" ("Rc<dyn Trait>" would work too).

To work with DSTs, Rust provides the "Sized" trait to determine whether or not a type’s size is known at compile time. This trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on "Sized" to every generic function. That is, a generic function definition like this:

```rust
fn generic<T>(t: T) {
    // --snip--
}
```

is actually treated as though we had written this:

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

By default, generic functions will work only on types that have a known size at compile time. However, you can use the following special syntax to relax this restriction:

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

A trait bound on "?Sized" means “"T" may or may not be "Sized"” and this notation overrides the default that generic types must have a known size at compile time. The "?Trait" syntax with this meaning is only available for "Sized", not any other traits.

Also note that we switched the type of the "t" parameter from "T" to "&T". Because the type might not be "Sized", we need to use it behind some kind of pointer. In this case, we’ve chosen a reference.

Next, we’ll talk about functions and closures!





---





## [Advanced Functions and Closures](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#advanced-functions-and-closures)

This section explores some advanced features related to functions and closures, including function pointers and returning closures.

### [Function Pointers](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#function-pointers)

We’ve talked about how to pass closures to functions; you can also pass regular functions to functions! This technique is useful when you want to pass a function you’ve already defined rather than defining a new closure. Functions coerce to the type "fn" (with a lowercase f), not to be confused with the "Fn" closure trait. The "fn" type is called a *function pointer*. Passing functions with function pointers will allow you to use functions as arguments to other functions.

The syntax for specifying that a parameter is a function pointer is similar to that of closures, as shown in Listing 19-27, where we’ve defined a function "add_one" that adds one to its parameter. The function "do_twice" takes two parameters: a function pointer to any function that takes an "i32" parameter and returns an "i32", and one "i32 value". The "do_twice" function calls the function "f" twice, passing it the "arg" value, then adds the two function call results together. The "main" function calls "do_twice" with the arguments "add_one" and "5".

Filename: src/main.rs

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

Listing 19-27: Using the "fn" type to accept a function pointer as an argument

This code prints "The answer is: 12". We specify that the parameter "f" in "do_twice" is an "fn" that takes one parameter of type "i32" and returns an "i32". We can then call "f" in the body of "do_twice". In "main", we can pass the function name "add_one" as the first argument to "do_twice".

Unlike closures, "fn" is a type rather than a trait, so we specify "fn" as the parameter type directly rather than declaring a generic type parameter with one of the "Fn" traits as a trait bound.

Function pointers implement all three of the closure traits ("Fn", "FnMut", and "FnOnce"), meaning you can always pass a function pointer as an argument for a function that expects a closure. It’s best to write functions using a generic type and one of the closure traits so your functions can accept either functions or closures.

That said, one example of where you would want to only accept "fn" and not closures is when interfacing with external code that doesn’t have closures: C functions can accept functions as arguments, but C doesn’t have closures.

As an example of where you could use either a closure defined inline or a named function, let’s look at a use of the "map" method provided by the "Iterator" trait in the standard library. To use the "map" function to turn a vector of numbers into a vector of strings, we could use a closure, like this:

```rust
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
```

Or we could name a function as the argument to "map" instead of the closure, like this:

```rust
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
```

Note that we must use the fully qualified syntax that we talked about earlier in the [“Advanced Traits”](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#advanced-traits) section because there are multiple functions available named "to_string". Here, we’re using the "to_string" function defined in the "ToString" trait, which the standard library has implemented for any type that implements "Display".

Recall from the [“Enum values”](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values) section of Chapter 6 that the name of each enum variant that we define also becomes an initializer function. We can use these initializer functions as function pointers that implement the closure traits, which means we can specify the initializer functions as arguments for methods that take closures, like so:

```rust
    enum Status {
        Value(u32),
        Stop,
    }

    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

Here we create "Status::Value" instances using each "u32" value in the range that "map" is called on by using the initializer function of "Status::Value". Some people prefer this style, and some people prefer to use closures. They compile to the same code, so use whichever style is clearer to you.

### [Returning Closures](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#returning-closures)

Closures are represented by traits, which means you can’t return closures directly. In most cases where you might want to return a trait, you can instead use the concrete type that implements the trait as the return value of the function. However, you can’t do that with closures because they don’t have a concrete type that is returnable; you’re not allowed to use the function pointer "fn" as a return type, for example.

The following code tries to return a closure directly, but it won’t compile:

```rust
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}
```

The compiler error is as follows:

```bash
$ cargo build
   Compiling functions-example v0.1.0 (file:///projects/functions-example)
error[E0746]: return type cannot have an unboxed trait object
 --> src/lib.rs:1:25
  |
1 | fn returns_closure() -> dyn Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^^^^^ doesn"t have a size known at compile-time
  |
  = note: for information on "impl Trait", see <https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits>
help: use "impl Fn(i32) -> i32" as the return type, as all return paths are of type "[closure@src/lib.rs:2:5: 2:8]", which implements "Fn(i32) -> i32"
  |
1 | fn returns_closure() -> impl Fn(i32) -> i32 {
  |                         ~~~~~~~~~~~~~~~~~~~

For more information about this error, try "rustc --explain E0746".
error: could not compile "functions-example" due to previous error
```

The error references the "Sized" trait again! Rust doesn’t know how much space it will need to store the closure. We saw a solution to this problem earlier. We can use a trait object:

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

This code will compile just fine. For more about trait objects, refer to the section [“Using Trait Objects That Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) in Chapter 17.

Next, let’s look at macros!





---





## [Macros](https://doc.rust-lang.org/book/ch19-06-macros.html#macros)

We’ve used macros like "println!" throughout this book, but we haven’t fully explored what a macro is and how it works. The term *macro* refers to a family of features in Rust: *declarative* macros with "macro_rules!" and three kinds of *procedural* macros:

- Custom "#[derive]" macros that specify code added with the "derive" attribute used on structs and enums
- Attribute-like macros that define custom attributes usable on any item
- Function-like macros that look like function calls but operate on the tokens specified as their argument

We’ll talk about each of these in turn, but first, let’s look at why we even need macros when we already have functions.

### [The Difference Between Macros and Functions](https://doc.rust-lang.org/book/ch19-06-macros.html#the-difference-between-macros-and-functions)

Fundamentally, macros are a way of writing code that writes other code, which is known as *metaprogramming*. In Appendix C, we discuss the "derive" attribute, which generates an implementation of various traits for you. We’ve also used the "println!" and "vec!" macros throughout the book. All of these macros *expand* to produce more code than the code you’ve written manually.

Metaprogramming is useful for reducing the amount of code you have to write and maintain, which is also one of the roles of functions. However, macros have some additional powers that functions don’t.

A function signature must declare the number and type of parameters the function has. Macros, on the other hand, can take a variable number of parameters: we can call "println!("hello")" with one argument or "println!("hello {}", name)" with two arguments. Also, macros are expanded before the compiler interprets the meaning of the code, so a macro can, for example, implement a trait on a given type. A function can’t, because it gets called at runtime and a trait needs to be implemented at compile time.

The downside to implementing a macro instead of a function is that macro definitions are more complex than function definitions because you’re writing Rust code that writes Rust code. Due to this indirection, macro definitions are generally more difficult to read, understand, and maintain than function definitions.

Another important difference between macros and functions is that you must define macros or bring them into scope *before* you call them in a file, as opposed to functions you can define anywhere and call anywhere.

### [Declarative Macros with "macro_rules!" for General Metaprogramming](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming)

The most widely used form of macros in Rust is the *declarative macro*. These are also sometimes referred to as “macros by example,” “"macro_rules!" macros,” or just plain “macros.” At their core, declarative macros allow you to write something similar to a Rust "match" expression. As discussed in Chapter 6, "match" expressions are control structures that take an expression, compare the resulting value of the expression to patterns, and then run the code associated with the matching pattern. Macros also compare a value to patterns that are associated with particular code: in this situation, the value is the literal Rust source code passed to the macro; the patterns are compared with the structure of that source code; and the code associated with each pattern, when matched, replaces the code passed to the macro. This all happens during compilation.

To define a macro, you use the "macro_rules!" construct. Let’s explore how to use "macro_rules!" by looking at how the "vec!" macro is defined. Chapter 8 covered how we can use the "vec!" macro to create a new vector with particular values. For example, the following macro creates a new vector containing three integers:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

We could also use the "vec!" macro to make a vector of two integers or a vector of five string slices. We wouldn’t be able to use a function to do the same because we wouldn’t know the number or type of values up front.

Listing 19-28 shows a slightly simplified definition of the "vec!" macro.

Filename: src/lib.rs

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

Listing 19-28: A simplified version of the "vec!" macro definition

> Note: The actual definition of the "vec!" macro in the standard library includes code to preallocate the correct amount of memory up front. That code is an optimization that we don’t include here to make the example simpler.

The "#[macro_export]" annotation indicates that this macro should be made available whenever the crate in which the macro is defined is brought into scope. Without this annotation, the macro can’t be brought into scope.

We then start the macro definition with "macro_rules!" and the name of the macro we’re defining *without* the exclamation mark. The name, in this case "vec", is followed by curly brackets denoting the body of the macro definition.

The structure in the "vec!" body is similar to the structure of a "match" expression. Here we have one arm with the pattern "( $( $x:expr ),* )", followed by "=>" and the block of code associated with this pattern. If the pattern matches, the associated block of code will be emitted. Given that this is the only pattern in this macro, there is only one valid way to match; any other pattern will result in an error. More complex macros will have more than one arm.

Valid pattern syntax in macro definitions is different than the pattern syntax covered in Chapter 18 because macro patterns are matched against Rust code structure rather than values. Let’s walk through what the pattern pieces in Listing 19-28 mean; for the full macro pattern syntax, see the [Rust Reference](https://doc.rust-lang.org/reference/macros-by-example.html).

First, we use a set of parentheses to encompass the whole pattern. We use a dollar sign ("$") to declare a variable in the macro system that will contain the Rust code matching the pattern. The dollar sign makes it clear this is a macro variable as opposed to a regular Rust variable. Next comes a set of parentheses that captures values that match the pattern within the parentheses for use in the replacement code. Within "$()" is "$x:expr", which matches any Rust expression and gives the expression the name "$x".

The comma following "$()" indicates that a literal comma separator character could optionally appear after the code that matches the code in "$()". The "*" specifies that the pattern matches zero or more of whatever precedes the "*".

When we call this macro with "vec![1, 2, 3];", the "$x" pattern matches three times with the three expressions "1", "2", and "3".

Now let’s look at the pattern in the body of the code associated with this arm: "temp_vec.push()" within "$()*" is generated for each part that matches "$()" in the pattern zero or more times depending on how many times the pattern matches. The "$x" is replaced with each expression matched. When we call this macro with "vec![1, 2, 3];", the code generated that replaces this macro call will be the following:

```rust
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

We’ve defined a macro that can take any number of arguments of any type and can generate code to create a vector containing the specified elements.

To learn more about how to write macros, consult the online documentation or other resources, such as [“The Little Book of Rust Macros”](https://veykril.github.io/tlborm/) started by Daniel Keep and continued by Lukas Wirth.

### [Procedural Macros for Generating Code from Attributes](https://doc.rust-lang.org/book/ch19-06-macros.html#procedural-macros-for-generating-code-from-attributes)

The second form of macros is the *procedural macro*, which acts more like a function (and is a type of procedure). Procedural macros accept some code as an input, operate on that code, and produce some code as an output rather than matching against patterns and replacing the code with other code as declarative macros do. The three kinds of procedural macros are custom derive, attribute-like, and function-like, and all work in a similar fashion.

When creating procedural macros, the definitions must reside in their own crate with a special crate type. This is for complex technical reasons that we hope to eliminate in the future. In Listing 19-29, we show how to define a procedural macro, where "some_attribute" is a placeholder for using a specific macro variety.

Filename: src/lib.rs

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

Listing 19-29: An example of defining a procedural macro

The function that defines a procedural macro takes a "TokenStream" as an input and produces a "TokenStream" as an output. The "TokenStream" type is defined by the "proc_macro" crate that is included with Rust and represents a sequence of tokens. This is the core of the macro: the source code that the macro is operating on makes up the input "TokenStream", and the code the macro produces is the output "TokenStream". The function also has an attribute attached to it that specifies which kind of procedural macro we’re creating. We can have multiple kinds of procedural macros in the same crate.

Let’s look at the different kinds of procedural macros. We’ll start with a custom derive macro and then explain the small dissimilarities that make the other forms different.

### [How to Write a Custom "derive" Macro](https://doc.rust-lang.org/book/ch19-06-macros.html#how-to-write-a-custom-derive-macro)

Let’s create a crate named "hello_macro" that defines a trait named "HelloMacro" with one associated function named "hello_macro". Rather than making our users implement the "HelloMacro" trait for each of their types, we’ll provide a procedural macro so users can annotate their type with "#[derive(HelloMacro)]" to get a default implementation of the "hello_macro" function. The default implementation will print "Hello, Macro! My name is TypeName!" where "TypeName" is the name of the type on which this trait has been defined. In other words, we’ll write a crate that enables another programmer to write code like Listing 19-30 using our crate.

Filename: src/main.rs

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

Listing 19-30: The code a user of our crate will be able to write when using our procedural macro

This code will print "Hello, Macro! My name is Pancakes!" when we’re done. The first step is to make a new library crate, like this:

```bash
$ cargo new hello_macro --lib
```

Next, we’ll define the "HelloMacro" trait and its associated function:

Filename: src/lib.rs

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

We have a trait and its function. At this point, our crate user could implement the trait to achieve the desired functionality, like so:

```rust
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}
```

However, they would need to write the implementation block for each type they wanted to use with "hello_macro"; we want to spare them from having to do this work.

Additionally, we can’t yet provide the "hello_macro" function with default implementation that will print the name of the type the trait is implemented on: Rust doesn’t have reflection capabilities, so it can’t look up the type’s name at runtime. We need a macro to generate code at compile time.

The next step is to define the procedural macro. At the time of this writing, procedural macros need to be in their own crate. Eventually, this restriction might be lifted. The convention for structuring crates and macro crates is as follows: for a crate named "foo", a custom derive procedural macro crate is called "foo_derive". Let’s start a new crate called "hello_macro_derive" inside our "hello_macro" project:

```bash
$ cargo new hello_macro_derive --lib
```

Our two crates are tightly related, so we create the procedural macro crate within the directory of our "hello_macro" crate. If we change the trait definition in "hello_macro", we’ll have to change the implementation of the procedural macro in "hello_macro_derive" as well. The two crates will need to be published separately, and programmers using these crates will need to add both as dependencies and bring them both into scope. We could instead have the "hello_macro" crate use "hello_macro_derive" as a dependency and re-export the procedural macro code. However, the way we’ve structured the project makes it possible for programmers to use "hello_macro" even if they don’t want the "derive" functionality.

We need to declare the "hello_macro_derive" crate as a procedural macro crate. We’ll also need functionality from the "syn" and "quote" crates, as you’ll see in a moment, so we need to add them as dependencies. Add the following to the *Cargo.toml* file for "hello_macro_derive":

Filename: hello_macro_derive/Cargo.toml

```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

To start defining the procedural macro, place the code in Listing 19-31 into your *src/lib.rs* file for the "hello_macro_derive" crate. Note that this code won’t compile until we add a definition for the "impl_hello_macro" function.

Filename: hello_macro_derive/src/lib.rs

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

Listing 19-31: Code that most procedural macro crates will require in order to process Rust code

Notice that we’ve split the code into the "hello_macro_derive" function, which is responsible for parsing the "TokenStream", and the "impl_hello_macro" function, which is responsible for transforming the syntax tree: this makes writing a procedural macro more convenient. The code in the outer function ("hello_macro_derive" in this case) will be the same for almost every procedural macro crate you see or create. The code you specify in the body of the inner function ("impl_hello_macro" in this case) will be different depending on your procedural macro’s purpose.

We’ve introduced three new crates: "proc_macro", ["syn"](https://crates.io/crates/syn), and ["quote"](https://crates.io/crates/quote). The "proc_macro" crate comes with Rust, so we didn’t need to add that to the dependencies in *Cargo.toml*. The "proc_macro" crate is the compiler’s API that allows us to read and manipulate Rust code from our code.

The "syn" crate parses Rust code from a string into a data structure that we can perform operations on. The "quote" crate turns "syn" data structures back into Rust code. These crates make it much simpler to parse any sort of Rust code we might want to handle: writing a full parser for Rust code is no simple task.

The "hello_macro_derive" function will be called when a user of our library specifies "#[derive(HelloMacro)]" on a type. This is possible because we’ve annotated the "hello_macro_derive" function here with "proc_macro_derive" and specified the name "HelloMacro", which matches our trait name; this is the convention most procedural macros follow.

The "hello_macro_derive" function first converts the "input" from a "TokenStream" to a data structure that we can then interpret and perform operations on. This is where "syn" comes into play. The "parse" function in "syn" takes a "TokenStream" and returns a "DeriveInput" struct representing the parsed Rust code. Listing 19-32 shows the relevant parts of the "DeriveInput" struct we get from parsing the "struct Pancakes;" string:

```rust
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

Listing 19-32: The "DeriveInput" instance we get when parsing the code that has the macro’s attribute in Listing 19-30

The fields of this struct show that the Rust code we’ve parsed is a unit struct with the "ident" (identifier, meaning the name) of "Pancakes". There are more fields on this struct for describing all sorts of Rust code; check the ["syn" documentation for "DeriveInput"](https://docs.rs/syn/1.0/syn/struct.DeriveInput.html) for more information.

Soon we’ll define the "impl_hello_macro" function, which is where we’ll build the new Rust code we want to include. But before we do, note that the output for our derive macro is also a "TokenStream". The returned "TokenStream" is added to the code that our crate users write, so when they compile their crate, they’ll get the extra functionality that we provide in the modified "TokenStream".

You might have noticed that we’re calling "unwrap" to cause the "hello_macro_derive" function to panic if the call to the "syn::parse" function fails here. It’s necessary for our procedural macro to panic on errors because "proc_macro_derive" functions must return "TokenStream" rather than "Result" to conform to the procedural macro API. We’ve simplified this example by using "unwrap"; in production code, you should provide more specific error messages about what went wrong by using "panic!" or "expect".

Now that we have the code to turn the annotated Rust code from a "TokenStream" into a "DeriveInput" instance, let’s generate the code that implements the "HelloMacro" trait on the annotated type, as shown in Listing 19-33.

Filename: hello_macro_derive/src/lib.rs

```rust
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

Listing 19-33: Implementing the "HelloMacro" trait using the parsed Rust code

We get an "Ident" struct instance containing the name (identifier) of the annotated type using "ast.ident". The struct in Listing 19-32 shows that when we run the "impl_hello_macro" function on the code in Listing 19-30, the "ident" we get will have the "ident" field with a value of ""Pancakes"". Thus, the "name" variable in Listing 19-33 will contain an "Ident" struct instance that, when printed, will be the string ""Pancakes"", the name of the struct in Listing 19-30.

The "quote!" macro lets us define the Rust code that we want to return. The compiler expects something different to the direct result of the "quote!" macro’s execution, so we need to convert it to a "TokenStream". We do this by calling the "into" method, which consumes this intermediate representation and returns a value of the required "TokenStream" type.

The "quote!" macro also provides some very cool templating mechanics: we can enter "#name", and "quote!" will replace it with the value in the variable "name". You can even do some repetition similar to the way regular macros work. Check out [the "quote" crate’s docs](https://docs.rs/quote) for a thorough introduction.

We want our procedural macro to generate an implementation of our "HelloMacro" trait for the type the user annotated, which we can get by using "#name". The trait implementation has the one function "hello_macro", whose body contains the functionality we want to provide: printing "Hello, Macro! My name is" and then the name of the annotated type.

The "stringify!" macro used here is built into Rust. It takes a Rust expression, such as "1 + 2", and at compile time turns the expression into a string literal, such as ""1 + 2"". This is different than "format!" or "println!", macros which evaluate the expression and then turn the result into a "String". There is a possibility that the "#name" input might be an expression to print literally, so we use "stringify!". Using "stringify!" also saves an allocation by converting "#name" to a string literal at compile time.

At this point, "cargo build" should complete successfully in both "hello_macro" and "hello_macro_derive". Let’s hook up these crates to the code in Listing 19-30 to see the procedural macro in action! Create a new binary project in your *projects* directory using "cargo new pancakes". We need to add "hello_macro" and "hello_macro_derive" as dependencies in the "pancakes" crate’s *Cargo.toml*. If you’re publishing your versions of "hello_macro" and "hello_macro_derive" to [crates.io](https://crates.io/), they would be regular dependencies; if not, you can specify them as "path" dependencies as follows:

```toml
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```

Put the code in Listing 19-30 into *src/main.rs*, and run "cargo run": it should print "Hello, Macro! My name is Pancakes!" The implementation of the "HelloMacro" trait from the procedural macro was included without the "pancakes" crate needing to implement it; the "#[derive(HelloMacro)]" added the trait implementation.

Next, let’s explore how the other kinds of procedural macros differ from custom derive macros.

### [Attribute-like macros](https://doc.rust-lang.org/book/ch19-06-macros.html#attribute-like-macros)

Attribute-like macros are similar to custom derive macros, but instead of generating code for the "derive" attribute, they allow you to create new attributes. They’re also more flexible: "derive" only works for structs and enums; attributes can be applied to other items as well, such as functions. Here’s an example of using an attribute-like macro: say you have an attribute named "route" that annotates functions when using a web application framework:

```rust
#[route(GET, "/")]
fn index() {
```

This "#[route]" attribute would be defined by the framework as a procedural macro. The signature of the macro definition function would look like this:

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Here, we have two parameters of type "TokenStream". The first is for the contents of the attribute: the "GET, "/"" part. The second is the body of the item the attribute is attached to: in this case, "fn index() {}" and the rest of the function’s body.

Other than that, attribute-like macros work the same way as custom derive macros: you create a crate with the "proc-macro" crate type and implement a function that generates the code you want!

### [Function-like macros](https://doc.rust-lang.org/book/ch19-06-macros.html#function-like-macros)

Function-like macros define macros that look like function calls. Similarly to "macro_rules!" macros, they’re more flexible than functions; for example, they can take an unknown number of arguments. However, "macro_rules!" macros can be defined only using the match-like syntax we discussed in the section [“Declarative Macros with "macro_rules!" for General Metaprogramming”](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming) earlier. Function-like macros take a "TokenStream" parameter and their definition manipulates that "TokenStream" using Rust code as the other two types of procedural macros do. An example of a function-like macro is an "sql!" macro that might be called like so:

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

This macro would parse the SQL statement inside it and check that it’s syntactically correct, which is much more complex processing than a "macro_rules!" macro can do. The "sql!" macro would be defined like this:

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

This definition is similar to the custom derive macro’s signature: we receive the tokens that are inside the parentheses and return the code we wanted to generate.

## [Summary](https://doc.rust-lang.org/book/ch19-06-macros.html#summary)

Whew! Now you have some Rust features in your toolbox that you likely won’t use often, but you’ll know they’re available in very particular circumstances. We’ve introduced several complex topics so that when you encounter them in error message suggestions or in other peoples’ code, you’ll be able to recognize these concepts and syntax. Use this chapter as a reference to guide you to solutions.

Next, we’ll put everything we’ve discussed throughout the book into practice and do one more project!





---



# 20



# [Final Project: Building a Multithreaded Web Server](https://doc.rust-lang.org/book/ch20-00-final-project-a-web-server.html#final-project-building-a-multithreaded-web-server)

It’s been a long journey, but we’ve reached the end of the book. In this chapter, we’ll build one more project together to demonstrate some of the concepts we covered in the final chapters, as well as recap some earlier lessons.

For our final project, we’ll make a web server that says “hello” and looks like Figure 20-1 in a web browser.

![hello from rust](https://doc.rust-lang.org/book/img/trpl20-01.png)

Figure 20-1: Our final shared project

Here is our plan for building the web server:

1. Learn a bit about TCP and HTTP.
2. Listen for TCP connections on a socket.
3. Parse a small number of HTTP requests.
4. Create a proper HTTP response.
5. Improve the throughput of our server with a thread pool.

Before we get started, we should mention one detail: the method we’ll use won’t be the best way to build a web server with Rust. Community members have published a number of production-ready crates available on [crates.io](https://crates.io/) that provide more complete web server and thread pool implementations than we’ll build. However, our intention in this chapter is to help you learn, not to take the easy route. Because Rust is a systems programming language, we can choose the level of abstraction we want to work with and can go to a lower level than is possible or practical in other languages. We’ll therefore write the basic HTTP server and thread pool manually so you can learn the general ideas and techniques behind the crates you might use in the future.





---





## [Building a Single-Threaded Web Server](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#building-a-single-threaded-web-server)

We’ll start by getting a single-threaded web server working. Before we begin, let’s look at a quick overview of the protocols involved in building web servers. The details of these protocols are beyond the scope of this book, but a brief overview will give you the information you need.

The two main protocols involved in web servers are *Hypertext Transfer Protocol* *(HTTP)* and *Transmission Control Protocol* *(TCP)*. Both protocols are *request-response* protocols, meaning a *client* initiates requests and a *server* listens to the requests and provides a response to the client. The contents of those requests and responses are defined by the protocols.

TCP is the lower-level protocol that describes the details of how information gets from one server to another but doesn’t specify what that information is. HTTP builds on top of TCP by defining the contents of the requests and responses. It’s technically possible to use HTTP with other protocols, but in the vast majority of cases, HTTP sends its data over TCP. We’ll work with the raw bytes of TCP and HTTP requests and responses.

### [Listening to the TCP Connection](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#listening-to-the-tcp-connection)

Our web server needs to listen to a TCP connection, so that’s the first part we’ll work on. The standard library offers a "std::net" module that lets us do this. Let’s make a new project in the usual fashion:

```bash
$ cargo new hello
     Created binary (application) "hello" project
$ cd hello
```

Now enter the code in Listing 20-1 in *src/main.rs* to start. This code will listen at the local address "127.0.0.1:7878" for incoming TCP streams. When it gets an incoming stream, it will print "Connection established!".

Filename: src/main.rs

```rust
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        println!("Connection established!");
    }
}
```

Listing 20-1: Listening for incoming streams and printing a message when we receive a stream

Using "TcpListener", we can listen for TCP connections at the address "127.0.0.1:7878". In the address, the section before the colon is an IP address representing your computer (this is the same on every computer and doesn’t represent the authors’ computer specifically), and "7878" is the port. We’ve chosen this port for two reasons: HTTP isn’t normally accepted on this port so our server is unlikely to conflict with any other web server you might have running on your machine, and 7878 is *rust* typed on a telephone.

The "bind" function in this scenario works like the "new" function in that it will return a new "TcpListener" instance. The function is called "bind" because, in networking, connecting to a port to listen to is known as “binding to a port.”

The "bind" function returns a "Result<T, E>", which indicates that it’s possible for binding to fail. For example, connecting to port 80 requires administrator privileges (nonadministrators can listen only on ports higher than 1023), so if we tried to connect to port 80 without being an administrator, binding wouldn’t work. Binding also wouldn’t work, for example, if we ran two instances of our program and so had two programs listening to the same port. Because we’re writing a basic server just for learning purposes, we won’t worry about handling these kinds of errors; instead, we use "unwrap" to stop the program if errors happen.

The "incoming" method on "TcpListener" returns an iterator that gives us a sequence of streams (more specifically, streams of type "TcpStream"). A single *stream* represents an open connection between the client and the server. A *connection* is the name for the full request and response process in which a client connects to the server, the server generates a response, and the server closes the connection. As such, we will read from the "TcpStream" to see what the client sent and then write our response to the stream to send data back to the client. Overall, this "for" loop will process each connection in turn and produce a series of streams for us to handle.

For now, our handling of the stream consists of calling "unwrap" to terminate our program if the stream has any errors; if there aren’t any errors, the program prints a message. We’ll add more functionality for the success case in the next listing. The reason we might receive errors from the "incoming" method when a client connects to the server is that we’re not actually iterating over connections. Instead, we’re iterating over *connection attempts*. The connection might not be successful for a number of reasons, many of them operating system specific. For example, many operating systems have a limit to the number of simultaneous open connections they can support; new connection attempts beyond that number will produce an error until some of the open connections are closed.

Let’s try running this code! Invoke "cargo run" in the terminal and then load *127.0.0.1:7878* in a web browser. The browser should show an error message like “Connection reset,” because the server isn’t currently sending back any data. But when you look at your terminal, you should see several messages that were printed when the browser connected to the server!

```
     Running "target/debug/hello"
Connection established!
Connection established!
Connection established!
```

Sometimes, you’ll see multiple messages printed for one browser request; the reason might be that the browser is making a request for the page as well as a request for other resources, like the *favicon.ico* icon that appears in the browser tab.

It could also be that the browser is trying to connect to the server multiple times because the server isn’t responding with any data. When "stream" goes out of scope and is dropped at the end of the loop, the connection is closed as part of the "drop" implementation. Browsers sometimes deal with closed connections by retrying, because the problem might be temporary. The important factor is that we’ve successfully gotten a handle to a TCP connection!

Remember to stop the program by pressing ctrl-c when you’re done running a particular version of the code. Then restart the program by invoking the "cargo run" command after you’ve made each set of code changes to make sure you’re running the newest code.

### [Reading the Request](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#reading-the-request)

Let’s implement the functionality to read the request from the browser! To separate the concerns of first getting a connection and then taking some action with the connection, we’ll start a new function for processing connections. In this new "handle_connection" function, we’ll read data from the TCP stream and print it so we can see the data being sent from the browser. Change the code to look like Listing 20-2.

Filename: src/main.rs

```rust
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

Listing 20-2: Reading from the "TcpStream" and printing the data

We bring "std::io::prelude" and "std::io::BufReader" into scope to get access to traits and types that let us read from and write to the stream. In the "for" loop in the "main" function, instead of printing a message that says we made a connection, we now call the new "handle_connection" function and pass the "stream" to it.

In the "handle_connection" function, we create a new "BufReader" instance that wraps a mutable reference to the "stream". "BufReader" adds buffering by managing calls to the "std::io::Read" trait methods for us.

We create a variable named "http_request" to collect the lines of the request the browser sends to our server. We indicate that we want to collect these lines in a vector by adding the "Vec<_>" type annotation.

"BufReader" implements the "std::io::BufRead" trait, which provides the "lines" method. The "lines" method returns an iterator of "Result<String, std::io::Error>" by splitting the stream of data whenever it sees a newline byte. To get each "String", we map and "unwrap" each "Result". The "Result" might be an error if the data isn’t valid UTF-8 or if there was a problem reading from the stream. Again, a production program should handle these errors more gracefully, but we’re choosing to stop the program in the error case for simplicity.

The browser signals the end of an HTTP request by sending two newline characters in a row, so to get one request from the stream, we take lines until we get a line that is the empty string. Once we’ve collected the lines into the vector, we’re printing them out using pretty debug formatting so we can take a look at the instructions the web browser is sending to our server.

Let’s try this code! Start the program and make a request in a web browser again. Note that we’ll still get an error page in the browser, but our program’s output in the terminal will now look similar to this:

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running "target/debug/hello"
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Depending on your browser, you might get slightly different output. Now that we’re printing the request data, we can see why we get multiple connections from one browser request by looking at the path after "GET" in the first line of the request. If the repeated connections are all requesting */*, we know the browser is trying to fetch */* repeatedly because it’s not getting a response from our program.

Let’s break down this request data to understand what the browser is asking of our program.

### [A Closer Look at an HTTP Request](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#a-closer-look-at-an-http-request)

HTTP is a text-based protocol, and a request takes this format:

```
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

The first line is the *request line* that holds information about what the client is requesting. The first part of the request line indicates the *method* being used, such as "GET" or "POST", which describes how the client is making this request. Our client used a "GET" request, which means it is asking for information.

The next part of the request line is */*, which indicates the *Uniform Resource Identifier* *(URI)* the client is requesting: a URI is almost, but not quite, the same as a *Uniform Resource Locator* *(URL)*. The difference between URIs and URLs isn’t important for our purposes in this chapter, but the HTTP spec uses the term URI, so we can just mentally substitute URL for URI here.

The last part is the HTTP version the client uses, and then the request line ends in a *CRLF sequence*. (CRLF stands for *carriage return* and *line feed*, which are terms from the typewriter days!) The CRLF sequence can also be written as "\r\n", where "\r" is a carriage return and "\n" is a line feed. The CRLF sequence separates the request line from the rest of the request data. Note that when the CRLF is printed, we see a new line start rather than "\r\n".

Looking at the request line data we received from running our program so far, we see that "GET" is the method, */* is the request URI, and "HTTP/1.1" is the version.

After the request line, the remaining lines starting from "Host:" onward are headers. "GET" requests have no body.

Try making a request from a different browser or asking for a different address, such as *127.0.0.1:7878/test*, to see how the request data changes.

Now that we know what the browser is asking for, let’s send back some data!

### [Writing a Response](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#writing-a-response)

We’re going to implement sending data in response to a client request. Responses have the following format:

```
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

The first line is a *status line* that contains the HTTP version used in the response, a numeric status code that summarizes the result of the request, and a reason phrase that provides a text description of the status code. After the CRLF sequence are any headers, another CRLF sequence, and the body of the response.

Here is an example response that uses HTTP version 1.1, has a status code of 200, an OK reason phrase, no headers, and no body:

```
HTTP/1.1 200 OK\r\n\r\n
```

The status code 200 is the standard success response. The text is a tiny successful HTTP response. Let’s write this to the stream as our response to a successful request! From the "handle_connection" function, remove the "println!" that was printing the request data and replace it with the code in Listing 20-3.

Filename: src/main.rs

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let response = "HTTP/1.1 200 OK\r\n\r\n";
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

Listing 20-3: Writing a tiny successful HTTP response to the stream

The first new line defines the "response" variable that holds the success message’s data. Then we call "as_bytes" on our "response" to convert the string data to bytes. The "write_all" method on "stream" takes a "&[u8]" and sends those bytes directly down the connection. Because the "write_all" operation could fail, we use "unwrap" on any error result as before. Again, in a real application you would add error handling here.

With these changes, let’s run our code and make a request. We’re no longer printing any data to the terminal, so we won’t see any output other than the output from Cargo. When you load *127.0.0.1:7878* in a web browser, you should get a blank page instead of an error. You’ve just hand-coded receiving an HTTP request and sending a response!

### [Returning Real HTML](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#returning-real-html)

Let’s implement the functionality for returning more than a blank page. Create the new file *hello.html* in the root of your project directory, not in the *src* directory. You can input any HTML you want; Listing 20-4 shows one possibility.

Filename: hello.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

Listing 20-4: A sample HTML file to return in a response

This is a minimal HTML5 document with a heading and some text. To return this from the server when a request is received, we’ll modify "handle_connection" as shown in Listing 20-5 to read the HTML file, add it to the response as a body, and send it.

Filename: src/main.rs

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();
    
    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

Listing 20-5: Sending the contents of *hello.html* as the body of the response

We’ve added "fs" to the "use" statement to bring the standard library’s filesystem module into scope. The code for reading the contents of a file to a string should look familiar; we used it in Chapter 12 when we read the contents of a file for our I/O project in Listing 12-4.

Next, we use "format!" to add the file’s contents as the body of the success response. To ensure a valid HTTP response, we add the "Content-Length" header which is set to the size of our response body, in this case the size of "hello.html".

Run this code with "cargo run" and load *127.0.0.1:7878* in your browser; you should see your HTML rendered!

Currently, we’re ignoring the request data in "http_request" and just sending back the contents of the HTML file unconditionally. That means if you try requesting *127.0.0.1:7878/something-else* in your browser, you’ll still get back this same HTML response. At the moment, our server is very limited and does not do what most web servers do. We want to customize our responses depending on the request and only send back the HTML file for a well-formed request to */*.

### [Validating the Request and Selectively Responding](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#validating-the-request-and-selectively-responding)

Right now, our web server will return the HTML in the file no matter what the client requested. Let’s add functionality to check that the browser is requesting */* before returning the HTML file and return an error if the browser requests anything else. For this we need to modify "handle_connection", as shown in Listing 20-6. This new code checks the content of the request received against what we know a request for */* looks like and adds "if" and "else" blocks to treat requests differently.

Filename: src/main.rs

```rust
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();
    
        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );
    
        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // some other request
    }
}
```

Listing 20-6: Handling requests to */* differently from other requests

We’re only going to be looking at the first line of the HTTP request, so rather than reading the entire request into a vector, we’re calling "next" to get the first item from the iterator. The first "unwrap" takes care of the "Option" and stops the program if the iterator has no items. The second "unwrap" handles the "Result" and has the same effect as the "unwrap" that was in the "map" added in Listing 20-2.

Next, we check the "request_line" to see if it equals the request line of a GET request to the */* path. If it does, the "if" block returns the contents of our HTML file.

If the "request_line" does *not* equal the GET request to the */* path, it means we’ve received some other request. We’ll add code to the "else" block in a moment to respond to all other requests.

Run this code now and request *127.0.0.1:7878*; you should get the HTML in *hello.html*. If you make any other request, such as *127.0.0.1:7878/something-else*, you’ll get a connection error like those you saw when running the code in Listing 20-1 and Listing 20-2.

Now let’s add the code in Listing 20-7 to the "else" block to return a response with the status code 404, which signals that the content for the request was not found. We’ll also return some HTML for a page to render in the browser indicating the response to the end user.

Filename: src/main.rs

```rust
    // --snip--
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );
    
        stream.write_all(response.as_bytes()).unwrap();
    }
```

Listing 20-7: Responding with status code 404 and an error page if anything other than */* was requested

Here, our response has a status line with status code 404 and the reason phrase "NOT FOUND". The body of the response will be the HTML in the file *404.html*. You’ll need to create a *404.html* file next to *hello.html* for the error page; again feel free to use any HTML you want or use the example HTML in Listing 20-8.

Filename: 404.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don"t know what you"re asking for.</p>
  </body>
</html>
```

Listing 20-8: Sample content for the page to send back with any 404 response

With these changes, run your server again. Requesting *127.0.0.1:7878* should return the contents of *hello.html*, and any other request, like *127.0.0.1:7878/foo*, should return the error HTML from *404.html*.

### [A Touch of Refactoring](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#a-touch-of-refactoring)

At the moment the "if" and "else" blocks have a lot of repetition: they’re both reading files and writing the contents of the files to the stream. The only differences are the status line and the filename. Let’s make the code more concise by pulling out those differences into separate "if" and "else" lines that will assign the values of the status line and the filename to variables; we can then use those variables unconditionally in the code to read the file and write the response. Listing 20-9 shows the resulting code after replacing the large "if" and "else" blocks.

Filename: src/main.rs

```rust
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
    
    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();
    
    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

Listing 20-9: Refactoring the "if" and "else" blocks to contain only the code that differs between the two cases

Now the "if" and "else" blocks only return the appropriate values for the status line and filename in a tuple; we then use destructuring to assign these two values to "status_line" and "filename" using a pattern in the "let" statement, as discussed in Chapter 18.

The previously duplicated code is now outside the "if" and "else" blocks and uses the "status_line" and "filename" variables. This makes it easier to see the difference between the two cases, and it means we have only one place to update the code if we want to change how the file reading and response writing work. The behavior of the code in Listing 20-9 will be the same as that in Listing 20-8.

Awesome! We now have a simple web server in approximately 40 lines of Rust code that responds to one request with a page of content and responds to all other requests with a 404 response.

Currently, our server runs in a single thread, meaning it can only serve one request at a time. Let’s examine how that can be a problem by simulating some slow requests. Then we’ll fix it so our server can handle multiple requests at once.





---





## [Turning Our Single-Threaded Server into a Multithreaded Server](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#turning-our-single-threaded-server-into-a-multithreaded-server)

Right now, the server will process each request in turn, meaning it won’t process a second connection until the first is finished processing. If the server received more and more requests, this serial execution would be less and less optimal. If the server receives a request that takes a long time to process, subsequent requests will have to wait until the long request is finished, even if the new requests can be processed quickly. We’ll need to fix this, but first, we’ll look at the problem in action.

### [Simulating a Slow Request in the Current Server Implementation](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#simulating-a-slow-request-in-the-current-server-implementation)

We’ll look at how a slow-processing request can affect other requests made to our current server implementation. Listing 20-10 implements handling a request to */sleep* with a simulated slow response that will cause the server to sleep for 5 seconds before responding.

Filename: src/main.rs

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };
    
    // --snip--
}
```

Listing 20-10: Simulating a slow request by sleeping for 5 seconds

We switched from "if" to "match" now that we have three cases. We need to explicitly match on a slice of "request_line" to pattern match against the string literal values; "match" doesn’t do automatic referencing and dereferencing like the equality method does.

The first arm is the same as the "if" block from Listing 20-9. The second arm matches a request to */sleep*. When that request is received, the server will sleep for 5 seconds before rendering the successful HTML page. The third arm is the same as the "else" block from Listing 20-9.

You can see how primitive our server is: real libraries would handle the recognition of multiple requests in a much less verbose way!

Start the server using "cargo run". Then open two browser windows: one for *http://127.0.0.1:7878/* and the other for *http://127.0.0.1:7878/sleep*. If you enter the */* URI a few times, as before, you’ll see it respond quickly. But if you enter */sleep* and then load */*, you’ll see that */* waits until "sleep" has slept for its full 5 seconds before loading.

There are multiple techniques we could use to avoid requests backing up behind a slow request; the one we’ll implement is a thread pool.

### [Improving Throughput with a Thread Pool](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#improving-throughput-with-a-thread-pool)

A *thread pool* is a group of spawned threads that are waiting and ready to handle a task. When the program receives a new task, it assigns one of the threads in the pool to the task, and that thread will process the task. The remaining threads in the pool are available to handle any other tasks that come in while the first thread is processing. When the first thread is done processing its task, it’s returned to the pool of idle threads, ready to handle a new task. A thread pool allows you to process connections concurrently, increasing the throughput of your server.

We’ll limit the number of threads in the pool to a small number to protect us from Denial of Service (DoS) attacks; if we had our program create a new thread for each request as it came in, someone making 10 million requests to our server could create havoc by using up all our server’s resources and grinding the processing of requests to a halt.

Rather than spawning unlimited threads, then, we’ll have a fixed number of threads waiting in the pool. Requests that come in are sent to the pool for processing. The pool will maintain a queue of incoming requests. Each of the threads in the pool will pop off a request from this queue, handle the request, and then ask the queue for another request. With this design, we can process up to "N" requests concurrently, where "N" is the number of threads. If each thread is responding to a long-running request, subsequent requests can still back up in the queue, but we’ve increased the number of long-running requests we can handle before reaching that point.

This technique is just one of many ways to improve the throughput of a web server. Other options you might explore are the *fork/join model*, the *single-threaded async I/O model*, or the *multi-threaded async I/O model*. If you’re interested in this topic, you can read more about other solutions and try to implement them; with a low-level language like Rust, all of these options are possible.

Before we begin implementing a thread pool, let’s talk about what using the pool should look like. When you’re trying to design code, writing the client interface first can help guide your design. Write the API of the code so it’s structured in the way you want to call it; then implement the functionality within that structure rather than implementing the functionality and then designing the public API.

Similar to how we used test-driven development in the project in Chapter 12, we’ll use compiler-driven development here. We’ll write the code that calls the functions we want, and then we’ll look at errors from the compiler to determine what we should change next to get the code to work. Before we do that, however, we’ll explore the technique we’re not going to use as a starting point.



#### [Spawning a Thread for Each Request](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#spawning-a-thread-for-each-request)

First, let’s explore how our code might look if it did create a new thread for every connection. As mentioned earlier, this isn’t our final plan due to the problems with potentially spawning an unlimited number of threads, but it is a starting point to get a working multithreaded server first. Then we’ll add the thread pool as an improvement, and contrasting the two solutions will be easier. Listing 20-11 shows the changes to make to "main" to spawn a new thread to handle each stream within the "for" loop.

Filename: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

Listing 20-11: Spawning a new thread for each stream

As you learned in Chapter 16, "thread::spawn" will create a new thread and then run the code in the closure in the new thread. If you run this code and load */sleep* in your browser, then */* in two more browser tabs, you’ll indeed see that the requests to */* don’t have to wait for */sleep* to finish. However, as we mentioned, this will eventually overwhelm the system because you’d be making new threads without any limit.



#### [Creating a Finite Number of Threads](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-a-finite-number-of-threads)

We want our thread pool to work in a similar, familiar way so switching from threads to a thread pool doesn’t require large changes to the code that uses our API. Listing 20-12 shows the hypothetical interface for a "ThreadPool" struct we want to use instead of "thread::spawn".

Filename: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

Listing 20-12: Our ideal "ThreadPool" interface

We use "ThreadPool::new" to create a new thread pool with a configurable number of threads, in this case four. Then, in the "for" loop, "pool.execute" has a similar interface as "thread::spawn" in that it takes a closure the pool should run for each stream. We need to implement "pool.execute" so it takes the closure and gives it to a thread in the pool to run. This code won’t yet compile, but we’ll try so the compiler can guide us in how to fix it.



#### [Building "ThreadPool" Using Compiler Driven Development](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#building-threadpool-using-compiler-driven-development)

Make the changes in Listing 20-12 to *src/main.rs*, and then let’s use the compiler errors from "cargo check" to drive our development. Here is the first error we get:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type "ThreadPool"
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type "ThreadPool"

For more information about this error, try "rustc --explain E0433".
error: could not compile "hello" due to previous error
```

Great! This error tells us we need a "ThreadPool" type or module, so we’ll build one now. Our "ThreadPool" implementation will be independent of the kind of work our web server is doing. So, let’s switch the "hello" crate from a binary crate to a library crate to hold our "ThreadPool" implementation. After we change to a library crate, we could also use the separate thread pool library for any work we want to do using a thread pool, not just for serving web requests.

Create a *src/lib.rs* that contains the following, which is the simplest definition of a "ThreadPool" struct that we can have for now:

Filename: src/lib.rs

```rust
pub struct ThreadPool;
```

Then edit *main.rs* file to bring "ThreadPool" into scope from the library crate by adding the following code to the top of *src/main.rs*:

Filename: src/main.rs

```rust
use hello::ThreadPool;
```

This code still won’t work, but let’s check it again to get the next error that we need to address:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named "new" found for struct "ThreadPool" in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in "ThreadPool"

For more information about this error, try "rustc --explain E0599".
error: could not compile "hello" due to previous error
```

This error indicates that next we need to create an associated function named "new" for "ThreadPool". We also know that "new" needs to have one parameter that can accept "4" as an argument and should return a "ThreadPool" instance. Let’s implement the simplest "new" function that will have those characteristics:

Filename: src/lib.rs

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

We chose "usize" as the type of the "size" parameter, because we know that a negative number of threads doesn’t make any sense. We also know we’ll use this 4 as the number of elements in a collection of threads, which is what the "usize" type is for, as discussed in the [“Integer Types”](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types) section of Chapter 3.

Let’s check the code again:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named "execute" found for struct "ThreadPool" in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in "ThreadPool"

For more information about this error, try "rustc --explain E0599".
error: could not compile "hello" due to previous error
```

Now the error occurs because we don’t have an "execute" method on "ThreadPool". Recall from the [“Creating a Finite Number of Threads”](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-a-finite-number-of-threads) section that we decided our thread pool should have an interface similar to "thread::spawn". In addition, we’ll implement the "execute" function so it takes the closure it’s given and gives it to an idle thread in the pool to run.

We’ll define the "execute" method on "ThreadPool" to take a closure as a parameter. Recall from the [“Moving Captured Values Out of the Closure and the "Fn" Traits”](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits) section in Chapter 13 that we can take closures as parameters with three different traits: "Fn", "FnMut", and "FnOnce". We need to decide which kind of closure to use here. We know we’ll end up doing something similar to the standard library "thread::spawn" implementation, so we can look at what bounds the signature of "thread::spawn" has on its parameter. The documentation shows us the following:

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + "static,
        T: Send + "static,
```

The "F" type parameter is the one we’re concerned with here; the "T" type parameter is related to the return value, and we’re not concerned with that. We can see that "spawn" uses "FnOnce" as the trait bound on "F". This is probably what we want as well, because we’ll eventually pass the argument we get in "execute" to "spawn". We can be further confident that "FnOnce" is the trait we want to use because the thread for running a request will only execute that request’s closure one time, which matches the "Once" in "FnOnce".

The "F" type parameter also has the trait bound "Send" and the lifetime bound ""static", which are useful in our situation: we need "Send" to transfer the closure from one thread to another and ""static" because we don’t know how long the thread will take to execute. Let’s create an "execute" method on "ThreadPool" that will take a generic parameter of type "F" with these bounds:

Filename: src/lib.rs

```rust
impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + "static,
    {
    }
}
```

We still use the "()" after "FnOnce" because this "FnOnce" represents a closure that takes no parameters and returns the unit type "()". Just like function definitions, the return type can be omitted from the signature, but even if we have no parameters, we still need the parentheses.

Again, this is the simplest implementation of the "execute" method: it does nothing, but we’re trying only to make our code compile. Let’s check it again:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
```

It compiles! But note that if you try "cargo run" and make a request in the browser, you’ll see the errors in the browser that we saw at the beginning of the chapter. Our library isn’t actually calling the closure passed to "execute" yet!

> Note: A saying you might hear about languages with strict compilers, such as Haskell and Rust, is “if the code compiles, it works.” But this saying is not universally true. Our project compiles, but it does absolutely nothing! If we were building a real, complete project, this would be a good time to start writing unit tests to check that the code compiles *and* has the behavior we want.

#### [Validating the Number of Threads in "new"](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#validating-the-number-of-threads-in-new)

We aren’t doing anything with the parameters to "new" and "execute". Let’s implement the bodies of these functions with the behavior we want. To start, let’s think about "new". Earlier we chose an unsigned type for the "size" parameter, because a pool with a negative number of threads makes no sense. However, a pool with zero threads also makes no sense, yet zero is a perfectly valid "usize". We’ll add code to check that "size" is greater than zero before we return a "ThreadPool" instance and have the program panic if it receives a zero by using the "assert!" macro, as shown in Listing 20-13.

Filename: src/lib.rs

```rust
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The "new" function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }
    
    // --snip--
}
```

Listing 20-13: Implementing "ThreadPool::new" to panic if "size" is zero

We’ve also added some documentation for our "ThreadPool" with doc comments. Note that we followed good documentation practices by adding a section that calls out the situations in which our function can panic, as discussed in Chapter 14. Try running "cargo doc --open" and clicking the "ThreadPool" struct to see what the generated docs for "new" look like!

Instead of adding the "assert!" macro as we’ve done here, we could change "new" into "build" and return a "Result" like we did with "Config::build" in the I/O project in Listing 12-9. But we’ve decided in this case that trying to create a thread pool without any threads should be an unrecoverable error. If you’re feeling ambitious, try to write a function named "build" with the following signature to compare with the "new" function:

```rust
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### [Creating Space to Store the Threads](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-space-to-store-the-threads)

Now that we have a way to know we have a valid number of threads to store in the pool, we can create those threads and store them in the "ThreadPool" struct before returning the struct. But how do we “store” a thread? Let’s take another look at the "thread::spawn" signature:

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + "static,
        T: Send + "static,
```

The "spawn" function returns a "JoinHandle<T>", where "T" is the type that the closure returns. Let’s try using "JoinHandle" too and see what happens. In our case, the closures we’re passing to the thread pool will handle the connection and not return anything, so "T" will be the unit type "()".

The code in Listing 20-14 will compile but doesn’t create any threads yet. We’ve changed the definition of "ThreadPool" to hold a vector of "thread::JoinHandle<()>" instances, initialized the vector with a capacity of "size", set up a "for" loop that will run some code to create the threads, and returned a "ThreadPool" instance containing them.

Filename: src/lib.rs

```rust
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);
    
        for _ in 0..size {
            // create some threads and store them in the vector
        }
    
        ThreadPool { threads }
    }
    // --snip--
}
```

Listing 20-14: Creating a vector for "ThreadPool" to hold the threads

We’ve brought "std::thread" into scope in the library crate, because we’re using "thread::JoinHandle" as the type of the items in the vector in "ThreadPool".

Once a valid size is received, our "ThreadPool" creates a new vector that can hold "size" items. The "with_capacity" function performs the same task as "Vec::new" but with an important difference: it preallocates space in the vector. Because we know we need to store "size" elements in the vector, doing this allocation up front is slightly more efficient than using "Vec::new", which resizes itself as elements are inserted.

When you run "cargo check" again, it should succeed.

#### [A "Worker" Struct Responsible for Sending Code from the "ThreadPool" to a Thread](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread)

We left a comment in the "for" loop in Listing 20-14 regarding the creation of threads. Here, we’ll look at how we actually create threads. The standard library provides "thread::spawn" as a way to create threads, and "thread::spawn" expects to get some code the thread should run as soon as the thread is created. However, in our case, we want to create the threads and have them *wait* for code that we’ll send later. The standard library’s implementation of threads doesn’t include any way to do that; we have to implement it manually.

We’ll implement this behavior by introducing a new data structure between the "ThreadPool" and the threads that will manage this new behavior. We’ll call this data structure *Worker*, which is a common term in pooling implementations. The Worker picks up code that needs to be run and runs the code in the Worker’s thread. Think of people working in the kitchen at a restaurant: the workers wait until orders come in from customers, and then they’re responsible for taking those orders and fulfilling them.

Instead of storing a vector of "JoinHandle<()>" instances in the thread pool, we’ll store instances of the "Worker" struct. Each "Worker" will store a single "JoinHandle<()>" instance. Then we’ll implement a method on "Worker" that will take a closure of code to run and send it to the already running thread for execution. We’ll also give each worker an "id" so we can distinguish between the different workers in the pool when logging or debugging.

Here is the new process that will happen when we create a "ThreadPool". We’ll implement the code that sends the closure to the thread after we have "Worker" set up in this way:

1. Define a "Worker" struct that holds an "id" and a "JoinHandle<()>".
2. Change "ThreadPool" to hold a vector of "Worker" instances.
3. Define a "Worker::new" function that takes an "id" number and returns a "Worker" instance that holds the "id" and a thread spawned with an empty closure.
4. In "ThreadPool::new", use the "for" loop counter to generate an "id", create a new "Worker" with that "id", and store the worker in the vector.

If you’re up for a challenge, try implementing these changes on your own before looking at the code in Listing 20-15.

Ready? Here is Listing 20-15 with one way to make the preceding modifications.

Filename: src/lib.rs

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id));
        }
    
        ThreadPool { workers }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

Listing 20-15: Modifying "ThreadPool" to hold "Worker" instances instead of holding threads directly

We’ve changed the name of the field on "ThreadPool" from "threads" to "workers" because it’s now holding "Worker" instances instead of "JoinHandle<()>" instances. We use the counter in the "for" loop as an argument to "Worker::new", and we store each new "Worker" in the vector named "workers".

External code (like our server in *src/main.rs*) doesn’t need to know the implementation details regarding using a "Worker" struct within "ThreadPool", so we make the "Worker" struct and its "new" function private. The "Worker::new" function uses the "id" we give it and stores a "JoinHandle<()>" instance that is created by spawning a new thread using an empty closure.

> Note: If the operating system can’t create a thread because there aren’t enough system resources, "thread::spawn" will panic. That will cause our whole server to panic, even though the creation of some threads might succeed. For simplicity’s sake, this behavior is fine, but in a production thread pool implementation, you’d likely want to use ["std::thread::Builder"](https://doc.rust-lang.org/std/thread/struct.Builder.html) and its ["spawn"](https://doc.rust-lang.org/std/thread/struct.Builder.html#method.spawn) method that returns "Result" instead.

This code will compile and will store the number of "Worker" instances we specified as an argument to "ThreadPool::new". But we’re *still* not processing the closure that we get in "execute". Let’s look at how to do that next.

#### [Sending Requests to Threads via Channels](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#sending-requests-to-threads-via-channels)

The next problem we’ll tackle is that the closures given to "thread::spawn" do absolutely nothing. Currently, we get the closure we want to execute in the "execute" method. But we need to give "thread::spawn" a closure to run when we create each "Worker" during the creation of the "ThreadPool".

We want the "Worker" structs that we just created to fetch the code to run from a queue held in the "ThreadPool" and send that code to its thread to run.

The channels we learned about in Chapter 16—a simple way to communicate between two threads—would be perfect for this use case. We’ll use a channel to function as the queue of jobs, and "execute" will send a job from the "ThreadPool" to the "Worker" instances, which will send the job to its thread. Here is the plan:

1. The "ThreadPool" will create a channel and hold on to the sender.
2. Each "Worker" will hold on to the receiver.
3. We’ll create a new "Job" struct that will hold the closures we want to send down the channel.
4. The "execute" method will send the job it wants to execute through the sender.
5. In its thread, the "Worker" will loop over its receiver and execute the closures of any jobs it receives.

Let’s start by creating a channel in "ThreadPool::new" and holding the sender in the "ThreadPool" instance, as shown in Listing 20-16. The "Job" struct doesn’t hold anything for now but will be the type of item we’re sending down the channel.

Filename: src/lib.rs

```rust
use std::{sync::mpsc, thread};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id));
        }
    
        ThreadPool { workers, sender }
    }
    // --snip--
}
```

Listing 20-16: Modifying "ThreadPool" to store the sender of a channel that transmits "Job" instances

In "ThreadPool::new", we create our new channel and have the pool hold the sender. This will successfully compile.

Let’s try passing a receiver of the channel into each worker as the thread pool creates the channel. We know we want to use the receiver in the thread that the workers spawn, so we’ll reference the "receiver" parameter in the closure. The code in Listing 20-17 won’t quite compile yet.

Filename: src/lib.rs

```rust
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }
    
        ThreadPool { workers, sender }
    }
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

Listing 20-17: Passing the receiver to the workers

We’ve made some small and straightforward changes: we pass the receiver into "Worker::new", and then we use it inside the closure.

When we try to check this code, we get this error:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: "receiver"
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because "receiver" has type "std::sync::mpsc::Receiver<Job>", which does not implement the "Copy" trait
...
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop

For more information about this error, try "rustc --explain E0382".
error: could not compile "hello" due to previous error
```

The code is trying to pass "receiver" to multiple "Worker" instances. This won’t work, as you’ll recall from Chapter 16: the channel implementation that Rust provides is multiple *producer*, single *consumer*. This means we can’t just clone the consuming end of the channel to fix this code. We also don’t want to send a message multiple times to multiple consumers; we want one list of messages with multiple workers such that each message gets processed once.

Additionally, taking a job off the channel queue involves mutating the "receiver", so the threads need a safe way to share and modify "receiver"; otherwise, we might get race conditions (as covered in Chapter 16).

Recall the thread-safe smart pointers discussed in Chapter 16: to share ownership across multiple threads and allow the threads to mutate the value, we need to use "Arc<Mutex<T>>". The "Arc" type will let multiple workers own the receiver, and "Mutex" will ensure that only one worker gets a job from the receiver at a time. Listing 20-18 shows the changes we need to make.

Filename: src/lib.rs

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};
// --snip--

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let receiver = Arc::new(Mutex::new(receiver));
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
    
        ThreadPool { workers, sender }
    }
    
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}
```

Listing 20-18: Sharing the receiver among the workers using "Arc" and "Mutex"

In "ThreadPool::new", we put the receiver in an "Arc" and a "Mutex". For each new worker, we clone the "Arc" to bump the reference count so the workers can share ownership of the receiver.

With these changes, the code compiles! We’re getting there!

#### [Implementing the "execute" Method](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#implementing-the-execute-method)

Let’s finally implement the "execute" method on "ThreadPool". We’ll also change "Job" from a struct to a type alias for a trait object that holds the type of closure that "execute" receives. As discussed in the [“Creating Type Synonyms with Type Aliases”](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases) section of Chapter 19, type aliases allow us to make long types shorter for ease of use. Look at Listing 20-19.

Filename: src/lib.rs

```rust
// --snip--

type Job = Box<dyn FnOnce() + Send + "static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + "static,
    {
        let job = Box::new(f);
    
        self.sender.send(job).unwrap();
    }
}

// --snip--
```

Listing 20-19: Creating a "Job" type alias for a "Box" that holds each closure and then sending the job down the channel

After creating a new "Job" instance using the closure we get in "execute", we send that job down the sending end of the channel. We’re calling "unwrap" on "send" for the case that sending fails. This might happen if, for example, we stop all our threads from executing, meaning the receiving end has stopped receiving new messages. At the moment, we can’t stop our threads from executing: our threads continue executing as long as the pool exists. The reason we use "unwrap" is that we know the failure case won’t happen, but the compiler doesn’t know that.

But we’re not quite done yet! In the worker, our closure being passed to "thread::spawn" still only *references* the receiving end of the channel. Instead, we need the closure to loop forever, asking the receiving end of the channel for a job and running the job when it gets one. Let’s make the change shown in Listing 20-20 to "Worker::new".

Filename: src/lib.rs

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Worker {id} got a job; executing.");
    
            job();
        });
    
        Worker { id, thread }
    }
}
```

Listing 20-20: Receiving and executing the jobs in the worker’s thread

Here, we first call "lock" on the "receiver" to acquire the mutex, and then we call "unwrap" to panic on any errors. Acquiring a lock might fail if the mutex is in a *poisoned* state, which can happen if some other thread panicked while holding the lock rather than releasing the lock. In this situation, calling "unwrap" to have this thread panic is the correct action to take. Feel free to change this "unwrap" to an "expect" with an error message that is meaningful to you.

If we get the lock on the mutex, we call "recv" to receive a "Job" from the channel. A final "unwrap" moves past any errors here as well, which might occur if the thread holding the sender has shut down, similar to how the "send" method returns "Err" if the receiver shuts down.

The call to "recv" blocks, so if there is no job yet, the current thread will wait until a job becomes available. The "Mutex<T>" ensures that only one "Worker" thread at a time is trying to request a job.

Our thread pool is now in a working state! Give it a "cargo run" and make some requests:

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: "workers"
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: "#[warn(dead_code)]" on by default

warning: field is never read: "id"
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: "thread"
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: "hello" (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running "target/debug/hello"
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Success! We now have a thread pool that executes connections asynchronously. There are never more than four threads created, so our system won’t get overloaded if the server receives a lot of requests. If we make a request to */sleep*, the server will be able to serve other requests by having another thread run them.

> Note: if you open */sleep* in multiple browser windows simultaneously, they might load one at a time in 5 second intervals. Some web browsers execute multiple instances of the same request sequentially for caching reasons. This limitation is not caused by our web server.

After learning about the "while let" loop in Chapter 18, you might be wondering why we didn’t write the worker thread code as shown in Listing 20-21.

Filename: src/lib.rs

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!("Worker {id} got a job; executing.");

                job();
            }
        });
    
        Worker { id, thread }
    }
}
```

Listing 20-21: An alternative implementation of "Worker::new" using "while let"

This code compiles and runs but doesn’t result in the desired threading behavior: a slow request will still cause other requests to wait to be processed. The reason is somewhat subtle: the "Mutex" struct has no public "unlock" method because the ownership of the lock is based on the lifetime of the "MutexGuard<T>" within the "LockResult<MutexGuard<T>>" that the "lock" method returns. At compile time, the borrow checker can then enforce the rule that a resource guarded by a "Mutex" cannot be accessed unless we hold the lock. However, this implementation can also result in the lock being held longer than intended if we aren’t mindful of the lifetime of the "MutexGuard<T>".

The code in Listing 20-20 that uses "let job = receiver.lock().unwrap().recv().unwrap();" works because with "let", any temporary values used in the expression on the right hand side of the equals sign are immediately dropped when the "let" statement ends. However, "while let" (and "if let" and "match") does not drop temporary values until the end of the associated block. In Listing 20-21, the lock remains held for the duration of the call to "job()", meaning other workers cannot receive jobs.





---





## [Graceful Shutdown and Cleanup](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#graceful-shutdown-and-cleanup)

The code in Listing 20-20 is responding to requests asynchronously through the use of a thread pool, as we intended. We get some warnings about the "workers", "id", and "thread" fields that we’re not using in a direct way that reminds us we’re not cleaning up anything. When we use the less elegant ctrl-c method to halt the main thread, all other threads are stopped immediately as well, even if they’re in the middle of serving a request.

Next, then, we’ll implement the "Drop" trait to call "join" on each of the threads in the pool so they can finish the requests they’re working on before closing. Then we’ll implement a way to tell the threads they should stop accepting new requests and shut down. To see this code in action, we’ll modify our server to accept only two requests before gracefully shutting down its thread pool.

### [Implementing the "Drop" Trait on "ThreadPool"](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#implementing-the-drop-trait-on-threadpool)

Let’s start with implementing "Drop" on our thread pool. When the pool is dropped, our threads should all join to make sure they finish their work. Listing 20-22 shows a first attempt at a "Drop" implementation; this code won’t quite work yet.

Filename: src/lib.rs

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

Listing 20-22: Joining each thread when the thread pool goes out of scope

First, we loop through each of the thread pool "workers". We use "&mut" for this because "self" is a mutable reference, and we also need to be able to mutate "worker". For each worker, we print a message saying that this particular worker is shutting down, and then we call "join" on that worker’s thread. If the call to "join" fails, we use "unwrap" to make Rust panic and go into an ungraceful shutdown.

Here is the error we get when we compile this code:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of "worker.thread" which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ ------ "worker.thread" moved due to this method call
   |             |
   |             move occurs because "worker.thread" has type "JoinHandle<()>", which does not implement the "Copy" trait
   |
note: this function takes ownership of the receiver "self", which moves "worker.thread"
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:17

For more information about this error, try "rustc --explain E0507".
error: could not compile "hello" due to previous error
```

The error tells us we can’t call "join" because we only have a mutable borrow of each "worker" and "join" takes ownership of its argument. To solve this issue, we need to move the thread out of the "Worker" instance that owns "thread" so "join" can consume the thread. We did this in Listing 17-15: if "Worker" holds an "Option<thread::JoinHandle<()>>" instead, we can call the "take" method on the "Option" to move the value out of the "Some" variant and leave a "None" variant in its place. In other words, a "Worker" that is running will have a "Some" variant in "thread", and when we want to clean up a "Worker", we’ll replace "Some" with "None" so the "Worker" doesn’t have a thread to run.

So we know we want to update the definition of "Worker" like this:

Filename: src/lib.rs

```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

Now let’s lean on the compiler to find the other places that need to change. Checking this code, we get two errors:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named "join" found for enum "Option" in the current scope
  --> src/lib.rs:52:27
   |
52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in "Option<JoinHandle<()>>"
   |
note: the method "join" exists on the type "JoinHandle<()>"
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:5
help: consider using "Option::expect" to unwrap the "JoinHandle<()>" value, panicking if the value is an "Option::None"
   |
52 |             worker.thread.expect("REASON").join().unwrap();
   |                          +++++++++++++++++

error[E0308]: mismatched types
  --> src/lib.rs:72:22
   |
72 |         Worker { id, thread }
   |                      ^^^^^^ expected enum "Option", found struct "JoinHandle"
   |
   = note: expected enum "Option<JoinHandle<()>>"
            found struct "JoinHandle<_>"
help: try wrapping the expression in "Some"
   |
72 |         Worker { id, thread: Some(thread) }
   |                      +++++++++++++      +

Some errors have detailed explanations: E0308, E0599.
For more information about an error, try "rustc --explain E0308".
error: could not compile "hello" due to 2 previous errors
```

Let’s address the second error, which points to the code at the end of "Worker::new"; we need to wrap the "thread" value in "Some" when we create a new "Worker". Make the following changes to fix this error:

Filename: src/lib.rs

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

The first error is in our "Drop" implementation. We mentioned earlier that we intended to call "take" on the "Option" value to move "thread" out of "worker". The following changes will do so:

Filename: src/lib.rs

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

As discussed in Chapter 17, the "take" method on "Option" takes the "Some" variant out and leaves "None" in its place. We’re using "if let" to destructure the "Some" and get the thread; then we call "join" on the thread. If a worker’s thread is already "None", we know that worker has already had its thread cleaned up, so nothing happens in that case.

### [Signaling to the Threads to Stop Listening for Jobs](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#signaling-to-the-threads-to-stop-listening-for-jobs)

With all the changes we’ve made, our code compiles without any warnings. However, the bad news is this code doesn’t function the way we want it to yet. The key is the logic in the closures run by the threads of the "Worker" instances: at the moment, we call "join", but that won’t shut down the threads because they "loop" forever looking for jobs. If we try to drop our "ThreadPool" with our current implementation of "drop", the main thread will block forever waiting for the first thread to finish.

To fix this problem, we’ll need a change in the "ThreadPool" "drop" implementation and then a change in the "Worker" loop.

First, we’ll change the "ThreadPool" "drop" implementation to explicitly drop the "sender" before waiting for the threads to finish. Listing 20-23 shows the changes to "ThreadPool" to explicitly drop "sender". We use the same "Option" and "take" technique as we did with the thread to be able to move "sender" out of "ThreadPool":

Filename: src/lib.rs

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --snip--
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --snip--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }
    
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + "static,
    {
        let job = Box::new(f);
    
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
    
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Listing 20-23: Explicitly drop "sender" before joining the worker threads

Dropping "sender" closes the channel, which indicates no more messages will be sent. When that happens, all the calls to "recv" that the workers do in the infinite loop will return an error. In Listing 20-24, we change the "Worker" loop to gracefully exit the loop in that case, which means the threads will finish when the "ThreadPool" "drop" implementation calls "join" on them.

Filename: src/lib.rs

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");
    
                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });
    
        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

Listing 20-24: Explicitly break out of the loop when "recv" returns an error

To see this code in action, let’s modify "main" to accept only two requests before gracefully shutting down the server, as shown in Listing 20-25.

Filename: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
    
    println!("Shutting down.");
}
```

Listing 20-25: Shut down the server after serving two requests by exiting the loop

You wouldn’t want a real-world web server to shut down after serving only two requests. This code just demonstrates that the graceful shutdown and cleanup is in working order.

The "take" method is defined in the "Iterator" trait and limits the iteration to the first two items at most. The "ThreadPool" will go out of scope at the end of "main", and the "drop" implementation will run.

Start the server with "cargo run", and make three requests. The third request should error, and in your terminal you should see output similar to this:

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running "target/debug/hello"
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

You might see a different ordering of workers and messages printed. We can see how this code works from the messages: workers 0 and 3 got the first two requests. The server stopped accepting connections after the second connection, and the "Drop" implementation on "ThreadPool" starts executing before worker 3 even starts its job. Dropping the "sender" disconnects all the workers and tells them to shut down. The workers each print a message when they disconnect, and then the thread pool calls "join" to wait for each worker thread to finish.

Notice one interesting aspect of this particular execution: the "ThreadPool" dropped the "sender", and before any worker received an error, we tried to join worker 0. Worker 0 had not yet gotten an error from "recv", so the main thread blocked waiting for worker 0 to finish. In the meantime, worker 3 received a job and then all threads received an error. When worker 0 finished, the main thread waited for the rest of the workers to finish. At that point, they had all exited their loops and stopped.

Congrats! We’ve now completed our project; we have a basic web server that uses a thread pool to respond asynchronously. We’re able to perform a graceful shutdown of the server, which cleans up all the threads in the pool.

Here’s the full code for reference:

Filename: src/main.rs

```rust
use hello::ThreadPool;
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
    
    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";
    
    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
    
    let contents = fs::read_to_string(filename).unwrap();
    
    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );
    
    stream.write_all(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

Filename: src/lib.rs

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}

type Job = Box<dyn FnOnce() + Send + "static>;

impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The "new" function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let receiver = Arc::new(Mutex::new(receiver));
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
    
        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }
    
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + "static,
    {
        let job = Box::new(f);
    
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
    
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");
    
                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });
    
        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

We could do more here! If you want to continue enhancing this project, here are some ideas:

- Add more documentation to "ThreadPool" and its public methods.
- Add tests of the library’s functionality.
- Change calls to "unwrap" to more robust error handling.
- Use "ThreadPool" to perform some task other than serving web requests.
- Find a thread pool crate on [crates.io](https://crates.io/) and implement a similar web server using the crate instead. Then compare its API and robustness to the thread pool we implemented.

## [Summary](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#summary)

Well done! You’ve made it to the end of the book! We want to thank you for joining us on this tour of Rust. You’re now ready to implement your own Rust projects and help with other peoples’ projects. Keep in mind that there is a welcoming community of other Rustaceans who would love to help you with any challenges you encounter on your Rust journey.







---





# [Appendix](https://doc.rust-lang.org/book/appendix-00.html#appendix)

The following sections contain reference material you may find useful in your Rust journey.





---





## [Appendix A: Keywords](https://doc.rust-lang.org/book/appendix-01-keywords.html#appendix-a-keywords)

The following list contains keywords that are reserved for current or future use by the Rust language. As such, they cannot be used as identifiers (except as raw identifiers as we’ll discuss in the “[Raw Identifiers](https://doc.rust-lang.org/book/appendix-01-keywords.html#raw-identifiers)” section). Identifiers are names of functions, variables, parameters, struct fields, modules, crates, constants, macros, static values, attributes, types, traits, or lifetimes.

### [Keywords Currently in Use](https://doc.rust-lang.org/book/appendix-01-keywords.html#keywords-currently-in-use)

The following is a list of keywords currently in use, with their functionality described.

- "as" - perform primitive casting, disambiguate the specific trait containing an item, or rename items in "use" statements
- "async" - return a "Future" instead of blocking the current thread
- "await" - suspend execution until the result of a "Future" is ready
- "break" - exit a loop immediately
- "const" - define constant items or constant raw pointers
- "continue" - continue to the next loop iteration
- "crate" - in a module path, refers to the crate root
- "dyn" - dynamic dispatch to a trait object
- "else" - fallback for "if" and "if let" control flow constructs
- "enum" - define an enumeration
- "extern" - link an external function or variable
- "false" - Boolean false literal
- "fn" - define a function or the function pointer type
- "for" - loop over items from an iterator, implement a trait, or specify a higher-ranked lifetime
- "if" - branch based on the result of a conditional expression
- "impl" - implement inherent or trait functionality
- "in" - part of "for" loop syntax
- "let" - bind a variable
- "loop" - loop unconditionally
- "match" - match a value to patterns
- "mod" - define a module
- "move" - make a closure take ownership of all its captures
- "mut" - denote mutability in references, raw pointers, or pattern bindings
- "pub" - denote public visibility in struct fields, "impl" blocks, or modules
- "ref" - bind by reference
- "return" - return from function
- "Self" - a type alias for the type we are defining or implementing
- "self" - method subject or current module
- "static" - global variable or lifetime lasting the entire program execution
- "struct" - define a structure
- "super" - parent module of the current module
- "trait" - define a trait
- "true" - Boolean true literal
- "type" - define a type alias or associated type
- "union" - define a [union](https://doc.rust-lang.org/reference/items/unions.html); is only a keyword when used in a union declaration
- "unsafe" - denote unsafe code, functions, traits, or implementations
- "use" - bring symbols into scope
- "where" - denote clauses that constrain a type
- "while" - loop conditionally based on the result of an expression

### [Keywords Reserved for Future Use](https://doc.rust-lang.org/book/appendix-01-keywords.html#keywords-reserved-for-future-use)

The following keywords do not yet have any functionality but are reserved by Rust for potential future use.

- "abstract"
- "become"
- "box"
- "do"
- "final"
- "macro"
- "override"
- "priv"
- "try"
- "typeof"
- "unsized"
- "virtual"
- "yield"

### [Raw Identifiers](https://doc.rust-lang.org/book/appendix-01-keywords.html#raw-identifiers)

*Raw identifiers* are the syntax that lets you use keywords where they wouldn’t normally be allowed. You use a raw identifier by prefixing a keyword with "r#".

For example, "match" is a keyword. If you try to compile the following function that uses "match" as its name:

Filename: src/main.rs

```rust
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

you’ll get this error:

```
error: expected identifier, found keyword "match"
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

The error shows that you can’t use the keyword "match" as the function identifier. To use "match" as a function name, you need to use the raw identifier syntax, like this:

Filename: src/main.rs

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

This code will compile without any errors. Note the "r#" prefix on the function name in its definition as well as where the function is called in "main".

Raw identifiers allow you to use any word you choose as an identifier, even if that word happens to be a reserved keyword. This gives us more freedom to choose identifier names, as well as lets us integrate with programs written in a language where these words aren’t keywords. In addition, raw identifiers allow you to use libraries written in a different Rust edition than your crate uses. For example, "try" isn’t a keyword in the 2015 edition but is in the 2018 edition. If you depend on a library that’s written using the 2015 edition and has a "try" function, you’ll need to use the raw identifier syntax, "r#try" in this case, to call that function from your 2018 edition code. See [Appendix E](https://doc.rust-lang.org/book/appendix-05-editions.html) for more information on editions.





---





## [Appendix B: Operators and Symbols](https://doc.rust-lang.org/book/appendix-02-operators.html#appendix-b-operators-and-symbols)

This appendix contains a glossary of Rust’s syntax, including operators and other symbols that appear by themselves or in the context of paths, generics, trait bounds, macros, attributes, comments, tuples, and brackets.

### [Operators](https://doc.rust-lang.org/book/appendix-02-operators.html#operators)

Table B-1 contains the operators in Rust, an example of how the operator would appear in context, a short explanation, and whether that operator is overloadable. If an operator is overloadable, the relevant trait to use to overload that operator is listed.

Table B-1: Operators

| Operator | Example                                          | Explanation                                                  | Overloadable?  |
| -------- | ------------------------------------------------ | ------------------------------------------------------------ | -------------- |
| "!"      | "ident!(...)", "ident!{...}", "ident![...]"      | Macro expansion                                              |                |
| "!"      | "!expr"                                          | Bitwise or logical complement                                | "Not"          |
| "!="     | "expr != expr"                                   | Nonequality comparison                                       | "PartialEq"    |
| "%"      | "expr % expr"                                    | Arithmetic remainder                                         | "Rem"          |
| "%="     | "var %= expr"                                    | Arithmetic remainder and assignment                          | "RemAssign"    |
| "&"      | "&expr", "&mut expr"                             | Borrow                                                       |                |
| "&"      | "&type", "&mut type", "&"a type", "&"a mut type" | Borrowed pointer type                                        |                |
| "&"      | "expr & expr"                                    | Bitwise AND                                                  | "BitAnd"       |
| "&="     | "var &= expr"                                    | Bitwise AND and assignment                                   | "BitAndAssign" |
| "&&"     | "expr && expr"                                   | Short-circuiting logical AND                                 |                |
| "*"      | "expr * expr"                                    | Arithmetic multiplication                                    | "Mul"          |
| "*="     | "var *= expr"                                    | Arithmetic multiplication and assignment                     | "MulAssign"    |
| "*"      | "*expr"                                          | Dereference                                                  | "Deref"        |
| "*"      | "*const type", "*mut type"                       | Raw pointer                                                  |                |
| "+"      | "trait + trait", ""a + trait"                    | Compound type constraint                                     |                |
| "+"      | "expr + expr"                                    | Arithmetic addition                                          | "Add"          |
| "+="     | "var += expr"                                    | Arithmetic addition and assignment                           | "AddAssign"    |
| ","      | "expr, expr"                                     | Argument and element separator                               |                |
| "-"      | "- expr"                                         | Arithmetic negation                                          | "Neg"          |
| "-"      | "expr - expr"                                    | Arithmetic subtraction                                       | "Sub"          |
| "-="     | "var -= expr"                                    | Arithmetic subtraction and assignment                        | "SubAssign"    |
| "->"     | "fn(...) -> type", "|...| -> type"               |
| "."      | "expr.ident"                                     | Member access                                                |                |
| ".."     | "..", "expr..", "..expr", "expr..expr"           | Right-exclusive range literal                                | "PartialOrd"   |
| "..="    | "..=expr", "expr..=expr"                         | Right-inclusive range literal                                | "PartialOrd"   |
| ".."     | "..expr"                                         | Struct literal update syntax                                 |                |
| ".."     | "variant(x, ..)", "struct_type { x, .. }"        | “And the rest” pattern binding                               |                |
| "..."    | "expr...expr"                                    | (Deprecated, use "..=" instead) In a pattern: inclusive range pattern |                |
| "/"      | "expr / expr"                                    | Arithmetic division                                          | "Div"          |
| "/="     | "var /= expr"                                    | Arithmetic division and assignment                           | "DivAssign"    |
| ":"      | "pat: type", "ident: type"                       | Constraints                                                  |                |
| ":"      | "ident: expr"                                    | Struct field initializer                                     |                |
| ":"      | ""a: loop {...}"                                 | Loop label                                                   |                |
| ";"      | "expr;"                                          | Statement and item terminator                                |                |
| ";"      | "[...; len]"                                     | Part of fixed-size array syntax                              |                |
| "<<"     | "expr << expr"                                   | Left-shift                                                   | "Shl"          |
| "<<="    | "var <<= expr"                                   | Left-shift and assignment                                    | "ShlAssign"    |
| "<"      | "expr < expr"                                    | Less than comparison                                         | "PartialOrd"   |
| "<="     | "expr <= expr"                                   | Less than or equal to comparison                             | "PartialOrd"   |
| "="      | "var = expr", "ident = type"                     | Assignment/equivalence                                       |                |
| "=="     | "expr == expr"                                   | Equality comparison                                          | "PartialEq"    |
| "=>"     | "pat => expr"                                    | Part of match arm syntax                                     |                |
| ">"      | "expr > expr"                                    | Greater than comparison                                      | "PartialOrd"   |
| ">="     | "expr >= expr"                                   | Greater than or equal to comparison                          | "PartialOrd"   |
| ">>"     | "expr >> expr"                                   | Right-shift                                                  | "Shr"          |
| ">>="    | "var >>= expr"                                   | Right-shift and assignment                                   | "ShrAssign"    |
| "@"      | "ident @ pat"                                    | Pattern binding                                              |                |
| "^"      | "expr ^ expr"                                    | Bitwise exclusive OR                                         | "BitXor"       |
| "^="     | "var ^= expr"                                    | Bitwise exclusive OR and assignment                          | "BitXorAssign" |
| "|"      | "pat | pat"                                      |
| "|"      | "expr | expr"                                    |
| "|="     | "var |= expr"                                    |
| "||"     | "expr |
| "?"      | "expr?"                                          | Error propagation                                            |                |

### [Non-operator Symbols](https://doc.rust-lang.org/book/appendix-02-operators.html#non-operator-symbols)

The following list contains all symbols that don’t function as operators; that is, they don’t behave like a function or method call.

Table B-2 shows symbols that appear on their own and are valid in a variety of locations.

Table B-2: Stand-Alone Syntax

| Symbol                                        | Explanation                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| ""ident"                                      | Named lifetime or loop label                                 |
| "...u8", "...i32", "...f64", "...usize", etc. | Numeric literal of specific type                             |
| ""...""                                       | String literal                                               |
| "r"..."", "r#"..."#", "r##"..."##", etc.      | Raw string literal, escape characters not processed          |
| "b"...""                                      | Byte string literal; constructs an array of bytes instead of a string |
| "br"..."", "br#"..."#", "br##"..."##", etc.   | Raw byte string literal, combination of raw and byte string literal |
| ""...""                                       | Character literal                                            |
| "b"...""                                      | ASCII byte literal                                           |
| "|...| expr"                                  | Closure                                                      |
| "!"                                           | Always empty bottom type for diverging functions             |
| "_"                                           | “Ignored” pattern binding; also used to make integer literals readable |

Table B-3 shows symbols that appear in the context of a path through the module hierarchy to an item.

Table B-3: Path-Related Syntax

| Symbol                                  | Explanation                                                  |
| --------------------------------------- | ------------------------------------------------------------ |
| "ident::ident"                          | Namespace path                                               |
| "::path"                                | Path relative to the crate root (i.e., an explicitly absolute path) |
| "self::path"                            | Path relative to the current module (i.e., an explicitly relative path). |
| "super::path"                           | Path relative to the parent of the current module            |
| "type::ident", "<type as trait>::ident" | Associated constants, functions, and types                   |
| "<type>::..."                           | Associated item for a type that cannot be directly named (e.g., "<&T>::...", "<[T]>::...", etc.) |
| "trait::method(...)"                    | Disambiguating a method call by naming the trait that defines it |
| "type::method(...)"                     | Disambiguating a method call by naming the type for which it’s defined |
| "<type as trait>::method(...)"          | Disambiguating a method call by naming the trait and type    |

Table B-4 shows symbols that appear in the context of using generic type parameters.

Table B-4: Generics

| Symbol                         | Explanation                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| "path<...>"                    | Specifies parameters to generic type in a type (e.g., "Vec<u8>") |
| "path::<...>", "method::<...>" | Specifies parameters to generic type, function, or method in an expression; often referred to as turbofish (e.g., ""42".parse::<i32>()") |
| "fn ident<...> ..."            | Define generic function                                      |
| "struct ident<...> ..."        | Define generic structure                                     |
| "enum ident<...> ..."          | Define generic enumeration                                   |
| "impl<...> ..."                | Define generic implementation                                |
| "for<...> type"                | Higher-ranked lifetime bounds                                |
| "type<ident=type>"             | A generic type where one or more associated types have specific assignments (e.g., "Iterator<Item=T>") |

Table B-5 shows symbols that appear in the context of constraining generic type parameters with trait bounds.

Table B-5: Trait Bound Constraints

| Symbol                        | Explanation                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| "T: U"                        | Generic parameter "T" constrained to types that implement "U" |
| "T: "a"                       | Generic type "T" must outlive lifetime ""a" (meaning the type cannot transitively contain any references with lifetimes shorter than ""a") |
| "T: "static"                  | Generic type "T" contains no borrowed references other than ""static" ones |
| ""b: "a"                      | Generic lifetime ""b" must outlive lifetime ""a"             |
| "T: ?Sized"                   | Allow generic type parameter to be a dynamically sized type  |
| ""a + trait", "trait + trait" | Compound type constraint                                     |

Table B-6 shows symbols that appear in the context of calling or defining macros and specifying attributes on an item.

Table B-6: Macros and Attributes

| Symbol                                      | Explanation        |
| ------------------------------------------- | ------------------ |
| "#[meta]"                                   | Outer attribute    |
| "#![meta]"                                  | Inner attribute    |
| "$ident"                                    | Macro substitution |
| "$ident:kind"                               | Macro capture      |
| "$(…)…"                                     | Macro repetition   |
| "ident!(...)", "ident!{...}", "ident![...]" | Macro invocation   |

Table B-7 shows symbols that create comments.

Table B-7: Comments

| Symbol     | Explanation             |
| ---------- | ----------------------- |
| "//"       | Line comment            |
| "//!"      | Inner line doc comment  |
| "///"      | Outer line doc comment  |
| "/*...*/"  | Block comment           |
| "/*!...*/" | Inner block doc comment |
| "/**...*/" | Outer block doc comment |

Table B-8 shows symbols that appear in the context of using tuples.

Table B-8: Tuples

| Symbol                   | Explanation                                                  |
| ------------------------ | ------------------------------------------------------------ |
| "()"                     | Empty tuple (aka unit), both literal and type                |
| "(expr)"                 | Parenthesized expression                                     |
| "(expr,)"                | Single-element tuple expression                              |
| "(type,)"                | Single-element tuple type                                    |
| "(expr, ...)"            | Tuple expression                                             |
| "(type, ...)"            | Tuple type                                                   |
| "expr(expr, ...)"        | Function call expression; also used to initialize tuple "struct"s and tuple "enum" variants |
| "expr.0", "expr.1", etc. | Tuple indexing                                               |

Table B-9 shows the contexts in which curly braces are used.

Table B-9: Curly Brackets

| Context      | Explanation      |
| ------------ | ---------------- |
| "{...}"      | Block expression |
| "Type {...}" | "struct" literal |

Table B-10 shows the contexts in which square brackets are used.

Table B-10: Square Brackets

| Context                                            | Explanation                                                  |
| -------------------------------------------------- | ------------------------------------------------------------ |
| "[...]"                                            | Array literal                                                |
| "[expr; len]"                                      | Array literal containing "len" copies of "expr"              |
| "[type; len]"                                      | Array type containing "len" instances of "type"              |
| "expr[expr]"                                       | Collection indexing. Overloadable ("Index", "IndexMut")      |
| "expr[..]", "expr[a..]", "expr[..b]", "expr[a..b]" | Collection indexing pretending to be collection slicing, using "Range", "RangeFrom", "RangeTo", or "RangeFull" as the “index” |





---





## [Appendix C: Derivable Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#appendix-c-derivable-traits)

In various places in the book, we’ve discussed the "derive" attribute, which you can apply to a struct or enum definition. The "derive" attribute generates code that will implement a trait with its own default implementation on the type you’ve annotated with the "derive" syntax.

In this appendix, we provide a reference of all the traits in the standard library that you can use with "derive". Each section covers:

- What operators and methods deriving this trait will enable
- What the implementation of the trait provided by "derive" does
- What implementing the trait signifies about the type
- The conditions in which you’re allowed or not allowed to implement the trait
- Examples of operations that require the trait

If you want different behavior from that provided by the "derive" attribute, consult the [standard library documentation](https://doc.rust-lang.org/std/index.html) for each trait for details of how to manually implement them.

These traits listed here are the only ones defined by the standard library that can be implemented on your types using "derive". Other traits defined in the standard library don’t have sensible default behavior, so it’s up to you to implement them in the way that makes sense for what you’re trying to accomplish.

An example of a trait that can’t be derived is "Display", which handles formatting for end users. You should always consider the appropriate way to display a type to an end user. What parts of the type should an end user be allowed to see? What parts would they find relevant? What format of the data would be most relevant to them? The Rust compiler doesn’t have this insight, so it can’t provide appropriate default behavior for you.

The list of derivable traits provided in this appendix is not comprehensive: libraries can implement "derive" for their own traits, making the list of traits you can use "derive" with truly open-ended. Implementing "derive" involves using a procedural macro, which is covered in the [“Macros”](https://doc.rust-lang.org/book/ch19-06-macros.html#macros) section of Chapter 19.

### ["Debug" for Programmer Output](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#debug-for-programmer-output)

The "Debug" trait enables debug formatting in format strings, which you indicate by adding ":?" within "{}" placeholders.

The "Debug" trait allows you to print instances of a type for debugging purposes, so you and other programmers using your type can inspect an instance at a particular point in a program’s execution.

The "Debug" trait is required, for example, in use of the "assert_eq!" macro. This macro prints the values of instances given as arguments if the equality assertion fails so programmers can see why the two instances weren’t equal.

### ["PartialEq" and "Eq" for Equality Comparisons](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#partialeq-and-eq-for-equality-comparisons)

The "PartialEq" trait allows you to compare instances of a type to check for equality and enables use of the "==" and "!=" operators.

Deriving "PartialEq" implements the "eq" method. When "PartialEq" is derived on structs, two instances are equal only if *all* fields are equal, and the instances are not equal if any fields are not equal. When derived on enums, each variant is equal to itself and not equal to the other variants.

The "PartialEq" trait is required, for example, with the use of the "assert_eq!" macro, which needs to be able to compare two instances of a type for equality.

The "Eq" trait has no methods. Its purpose is to signal that for every value of the annotated type, the value is equal to itself. The "Eq" trait can only be applied to types that also implement "PartialEq", although not all types that implement "PartialEq" can implement "Eq". One example of this is floating point number types: the implementation of floating point numbers states that two instances of the not-a-number ("NaN") value are not equal to each other.

An example of when "Eq" is required is for keys in a "HashMap<K, V>" so the "HashMap<K, V>" can tell whether two keys are the same.

### ["PartialOrd" and "Ord" for Ordering Comparisons](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#partialord-and-ord-for-ordering-comparisons)

The "PartialOrd" trait allows you to compare instances of a type for sorting purposes. A type that implements "PartialOrd" can be used with the "<", ">", "<=", and ">=" operators. You can only apply the "PartialOrd" trait to types that also implement "PartialEq".

Deriving "PartialOrd" implements the "partial_cmp" method, which returns an "Option<Ordering>" that will be "None" when the values given don’t produce an ordering. An example of a value that doesn’t produce an ordering, even though most values of that type can be compared, is the not-a-number ("NaN") floating point value. Calling "partial_cmp" with any floating point number and the "NaN" floating point value will return "None".

When derived on structs, "PartialOrd" compares two instances by comparing the value in each field in the order in which the fields appear in the struct definition. When derived on enums, variants of the enum declared earlier in the enum definition are considered less than the variants listed later.

The "PartialOrd" trait is required, for example, for the "gen_range" method from the "rand" crate that generates a random value in the range specified by a range expression.

The "Ord" trait allows you to know that for any two values of the annotated type, a valid ordering will exist. The "Ord" trait implements the "cmp" method, which returns an "Ordering" rather than an "Option<Ordering>" because a valid ordering will always be possible. You can only apply the "Ord" trait to types that also implement "PartialOrd" and "Eq" (and "Eq" requires "PartialEq"). When derived on structs and enums, "cmp" behaves the same way as the derived implementation for "partial_cmp" does with "PartialOrd".

An example of when "Ord" is required is when storing values in a "BTreeSet<T>", a data structure that stores data based on the sort order of the values.

### ["Clone" and "Copy" for Duplicating Values](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#clone-and-copy-for-duplicating-values)

The "Clone" trait allows you to explicitly create a deep copy of a value, and the duplication process might involve running arbitrary code and copying heap data. See the [“Ways Variables and Data Interact: Clone”](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ways-variables-and-data-interact-clone) section in Chapter 4 for more information on "Clone".

Deriving "Clone" implements the "clone" method, which when implemented for the whole type, calls "clone" on each of the parts of the type. This means all the fields or values in the type must also implement "Clone" to derive "Clone".

An example of when "Clone" is required is when calling the "to_vec" method on a slice. The slice doesn’t own the type instances it contains, but the vector returned from "to_vec" will need to own its instances, so "to_vec" calls "clone" on each item. Thus, the type stored in the slice must implement "Clone".

The "Copy" trait allows you to duplicate a value by only copying bits stored on the stack; no arbitrary code is necessary. See the [“Stack-Only Data: Copy”](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy) section in Chapter 4 for more information on "Copy".

The "Copy" trait doesn’t define any methods to prevent programmers from overloading those methods and violating the assumption that no arbitrary code is being run. That way, all programmers can assume that copying a value will be very fast.

You can derive "Copy" on any type whose parts all implement "Copy". A type that implements "Copy" must also implement "Clone", because a type that implements "Copy" has a trivial implementation of "Clone" that performs the same task as "Copy".

The "Copy" trait is rarely required; types that implement "Copy" have optimizations available, meaning you don’t have to call "clone", which makes the code more concise.

Everything possible with "Copy" you can also accomplish with "Clone", but the code might be slower or have to use "clone" in places.

### ["Hash" for Mapping a Value to a Value of Fixed Size](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#hash-for-mapping-a-value-to-a-value-of-fixed-size)

The "Hash" trait allows you to take an instance of a type of arbitrary size and map that instance to a value of fixed size using a hash function. Deriving "Hash" implements the "hash" method. The derived implementation of the "hash" method combines the result of calling "hash" on each of the parts of the type, meaning all fields or values must also implement "Hash" to derive "Hash".

An example of when "Hash" is required is in storing keys in a "HashMap<K, V>" to store data efficiently.

### ["Default" for Default Values](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#default-for-default-values)

The "Default" trait allows you to create a default value for a type. Deriving "Default" implements the "default" function. The derived implementation of the "default" function calls the "default" function on each part of the type, meaning all fields or values in the type must also implement "Default" to derive "Default".

The "Default::default" function is commonly used in combination with the struct update syntax discussed in the [“Creating Instances From Other Instances With Struct Update Syntax”](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax) section in Chapter 5. You can customize a few fields of a struct and then set and use a default value for the rest of the fields by using "..Default::default()".

The "Default" trait is required when you use the method "unwrap_or_default" on "Option<T>" instances, for example. If the "Option<T>" is "None", the method "unwrap_or_default" will return the result of "Default::default" for the type "T" stored in the "Option<T>".





---





## [Appendix D - Useful Development Tools](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#appendix-d---useful-development-tools)

In this appendix, we talk about some useful development tools that the Rust project provides. We’ll look at automatic formatting, quick ways to apply warning fixes, a linter, and integrating with IDEs.

### [Automatic Formatting with "rustfmt"](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#automatic-formatting-with-rustfmt)

The "rustfmt" tool reformats your code according to the community code style. Many collaborative projects use "rustfmt" to prevent arguments about which style to use when writing Rust: everyone formats their code using the tool.

To install "rustfmt", enter the following:

```bash
$ rustup component add rustfmt
```

This command gives you "rustfmt" and "cargo-fmt", similar to how Rust gives you both "rustc" and "cargo". To format any Cargo project, enter the following:

```bash
$ cargo fmt
```

Running this command reformats all the Rust code in the current crate. This should only change the code style, not the code semantics. For more information on "rustfmt", see [its documentation](https://github.com/rust-lang/rustfmt).

### [Fix Your Code with "rustfix"](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#fix-your-code-with-rustfix)

The rustfix tool is included with Rust installations and can automatically fix compiler warnings that have a clear way to correct the problem that’s likely what you want. It’s likely you’ve seen compiler warnings before. For example, consider this code:

Filename: src/main.rs

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

Here, we’re calling the "do_something" function 100 times, but we never use the variable "i" in the body of the "for" loop. Rust warns us about that:

```bash
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: "i"
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using "_i" instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

The warning suggests that we use "_i" as a name instead: the underscore indicates that we intend for this variable to be unused. We can automatically apply that suggestion using the "rustfix" tool by running the command "cargo fix":

```bash
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

When we look at *src/main.rs* again, we’ll see that "cargo fix" has changed the code:

Filename: src/main.rs

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

The "for" loop variable is now named "_i", and the warning no longer appears.

You can also use the "cargo fix" command to transition your code between different Rust editions. Editions are covered in Appendix E.

### [More Lints with Clippy](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#more-lints-with-clippy)

The Clippy tool is a collection of lints to analyze your code so you can catch common mistakes and improve your Rust code.

To install Clippy, enter the following:

```bash
$ rustup component add clippy
```

To run Clippy’s lints on any Cargo project, enter the following:

```bash
$ cargo clippy
```

For example, say you write a program that uses an approximation of a mathematical constant, such as pi, as this program does:

Filename: src/main.rs

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Running "cargo clippy" on this project results in this error:

```
error: approximate value of "f{32, 64}::consts::PI" found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: "#[deny(clippy::approx_constant)]" on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

This error lets you know that Rust already has a more precise "PI" constant defined, and that your program would be more correct if you used the constant instead. You would then change your code to use the "PI" constant. The following code doesn’t result in any errors or warnings from Clippy:

Filename: src/main.rs

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

For more information on Clippy, see [its documentation](https://github.com/rust-lang/rust-clippy).

### [IDE Integration Using "rust-analyzer"](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#ide-integration-using-rust-analyzer)

To help IDE integration, the Rust community recommends using ["rust-analyzer"](https://rust-analyzer.github.io/). This tool is a set of compiler-centric utilities that speaks the [Language Server Protocol](http://langserver.org/), which is a specification for IDEs and programming languages to communicate with each other. Different clients can use "rust-analyzer", such as [the Rust analyzer plug-in for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer).

Visit the "rust-analyzer" project’s [home page](https://rust-analyzer.github.io/) for installation instructions, then install the language server support in your particular IDE. Your IDE will gain abilities such as autocompletion, jump to definition, and inline errors.





---





## [Appendix E - Editions](https://doc.rust-lang.org/book/appendix-05-editions.html#appendix-e---editions)

In Chapter 1, you saw that "cargo new" adds a bit of metadata to your *Cargo.toml* file about an edition. This appendix talks about what that means!

The Rust language and compiler have a six-week release cycle, meaning users get a constant stream of new features. Other programming languages release larger changes less often; Rust releases smaller updates more frequently. After a while, all of these tiny changes add up. But from release to release, it can be difficult to look back and say, “Wow, between Rust 1.10 and Rust 1.31, Rust has changed a lot!”

Every two or three years, the Rust team produces a new Rust *edition*. Each edition brings together the features that have landed into a clear package with fully updated documentation and tooling. New editions ship as part of the usual six-week release process.

Editions serve different purposes for different people:

- For active Rust users, a new edition brings together incremental changes into an easy-to-understand package.
- For non-users, a new edition signals that some major advancements have landed, which might make Rust worth another look.
- For those developing Rust, a new edition provides a rallying point for the project as a whole.

At the time of this writing, three Rust editions are available: Rust 2015, Rust 2018, and Rust 2021. This book is written using Rust 2021 edition idioms.

The "edition" key in *Cargo.toml* indicates which edition the compiler should use for your code. If the key doesn’t exist, Rust uses "2015" as the edition value for backward compatibility reasons.

Each project can opt in to an edition other than the default 2015 edition. Editions can contain incompatible changes, such as including a new keyword that conflicts with identifiers in code. However, unless you opt in to those changes, your code will continue to compile even as you upgrade the Rust compiler version you use.

All Rust compiler versions support any edition that existed prior to that compiler’s release, and they can link crates of any supported editions together. Edition changes only affect the way the compiler initially parses code. Therefore, if you’re using Rust 2015 and one of your dependencies uses Rust 2018, your project will compile and be able to use that dependency. The opposite situation, where your project uses Rust 2018 and a dependency uses Rust 2015, works as well.

To be clear: most features will be available on all editions. Developers using any Rust edition will continue to see improvements as new stable releases are made. However, in some cases, mainly when new keywords are added, some new features might only be available in later editions. You will need to switch editions if you want to take advantage of such features.

For more details, the [*Edition Guide*](https://doc.rust-lang.org/stable/edition-guide/) is a complete book about editions that enumerates the differences between editions and explains how to automatically upgrade your code to a new edition via "cargo fix".





---





## [Appendix F: Translations of the Book](https://doc.rust-lang.org/book/appendix-06-translation.html#appendix-f-translations-of-the-book)

For resources in languages other than English. Most are still in progress; see [the Translations label](https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations) to help or let us know about a new translation!

- [Português](https://github.com/rust-br/rust-book-pt-br) (BR)
- [Português](https://github.com/nunojesus/rust-book-pt-pt) (PT)
- [简体中文](https://github.com/KaiserY/trpl-zh-cn)
- [正體中文](https://github.com/rust-tw/book-tw)
- [Українська](https://github.com/pavloslav/rust-book-uk-ua)
- [Español](https://github.com/thecodix/book), [alternate](https://github.com/ManRR/rust-book-es)
- [Italiano](https://github.com/EmanueleGurini/book_it)
- [Русский](https://github.com/rust-lang-ru/book)
- [한국어](https://github.com/rinthel/rust-lang-book-ko)
- [日本語](https://github.com/rust-lang-ja/book-ja)
- [Français](https://github.com/Jimskapt/rust-book-fr)
- [Polski](https://github.com/paytchoo/book-pl)
- [Cebuano](https://github.com/agentzero1/book)
- [Tagalog](https://github.com/josephace135/book)
- [Esperanto](https://github.com/psychoslave/Rust-libro)
- [ελληνική](https://github.com/TChatzigiannakis/rust-book-greek)
- [Svenska](https://github.com/sebras/book)
- [Farsi](https://github.com/pomokhtari/rust-book-fa)
- [Deutsch](https://github.com/rust-lang-de/rustbook-de)
- [हिंदी](https://github.com/venkatarun95/rust-book-hindi)
- [ไทย](https://github.com/rust-lang-th/book-th)
- [Danske](https://github.com/DanKHansen/book-dk)



---





## [Appendix G - How Rust is Made and “Nightly Rust”](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#appendix-g---how-rust-is-made-and-nightly-rust)

This appendix is about how Rust is made and how that affects you as a Rust developer.

### [Stability Without Stagnation](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#stability-without-stagnation)

As a language, Rust cares a *lot* about the stability of your code. We want Rust to be a rock-solid foundation you can build on, and if things were constantly changing, that would be impossible. At the same time, if we can’t experiment with new features, we may not find out important flaws until after their release, when we can no longer change things.

Our solution to this problem is what we call “stability without stagnation”, and our guiding principle is this: you should never have to fear upgrading to a new version of stable Rust. Each upgrade should be painless, but should also bring you new features, fewer bugs, and faster compile times.

### [Choo, Choo! Release Channels and Riding the Trains](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#choo-choo-release-channels-and-riding-the-trains)

Rust development operates on a *train schedule*. That is, all development is done on the "master" branch of the Rust repository. Releases follow a software release train model, which has been used by Cisco IOS and other software projects. There are three *release channels* for Rust:

- Nightly
- Beta
- Stable

Most Rust developers primarily use the stable channel, but those who want to try out experimental new features may use nightly or beta.

Here’s an example of how the development and release process works: let’s assume that the Rust team is working on the release of Rust 1.5. That release happened in December of 2015, but it will provide us with realistic version numbers. A new feature is added to Rust: a new commit lands on the "master" branch. Each night, a new nightly version of Rust is produced. Every day is a release day, and these releases are created by our release infrastructure automatically. So as time passes, our releases look like this, once a night:

```
nightly: * - - * - - *
```

Every six weeks, it’s time to prepare a new release! The "beta" branch of the Rust repository branches off from the "master" branch used by nightly. Now, there are two releases:

```
nightly: * - - * - - *
                     |
beta:                *
```

Most Rust users do not use beta releases actively, but test against beta in their CI system to help Rust discover possible regressions. In the meantime, there’s still a nightly release every night:

```
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Let’s say a regression is found. Good thing we had some time to test the beta release before the regression snuck into a stable release! The fix is applied to "master", so that nightly is fixed, and then the fix is backported to the "beta" branch, and a new release of beta is produced:

```
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Six weeks after the first beta was created, it’s time for a stable release! The "stable" branch is produced from the "beta" branch:

```
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hooray! Rust 1.5 is done! However, we’ve forgotten one thing: because the six weeks have gone by, we also need a new beta of the *next* version of Rust, 1.6. So after "stable" branches off of "beta", the next version of "beta" branches off of "nightly" again:

```
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

This is called the “train model” because every six weeks, a release “leaves the station”, but still has to take a journey through the beta channel before it arrives as a stable release.

Rust releases every six weeks, like clockwork. If you know the date of one Rust release, you can know the date of the next one: it’s six weeks later. A nice aspect of having releases scheduled every six weeks is that the next train is coming soon. If a feature happens to miss a particular release, there’s no need to worry: another one is happening in a short time! This helps reduce pressure to sneak possibly unpolished features in close to the release deadline.

Thanks to this process, you can always check out the next build of Rust and verify for yourself that it’s easy to upgrade to: if a beta release doesn’t work as expected, you can report it to the team and get it fixed before the next stable release happens! Breakage in a beta release is relatively rare, but "rustc" is still a piece of software, and bugs do exist.

### [Unstable Features](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#unstable-features)

There’s one more catch with this release model: unstable features. Rust uses a technique called “feature flags” to determine what features are enabled in a given release. If a new feature is under active development, it lands on "master", and therefore, in nightly, but behind a *feature flag*. If you, as a user, wish to try out the work-in-progress feature, you can, but you must be using a nightly release of Rust and annotate your source code with the appropriate flag to opt in.

If you’re using a beta or stable release of Rust, you can’t use any feature flags. This is the key that allows us to get practical use with new features before we declare them stable forever. Those who wish to opt into the bleeding edge can do so, and those who want a rock-solid experience can stick with stable and know that their code won’t break. Stability without stagnation.

This book only contains information about stable features, as in-progress features are still changing, and surely they’ll be different between when this book was written and when they get enabled in stable builds. You can find documentation for nightly-only features online.

### [Rustup and the Role of Rust Nightly](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#rustup-and-the-role-of-rust-nightly)

Rustup makes it easy to change between different release channels of Rust, on a global or per-project basis. By default, you’ll have stable Rust installed. To install nightly, for example:

```bash
$ rustup toolchain install nightly
```

You can see all of the *toolchains* (releases of Rust and associated components) you have installed with "rustup" as well. Here’s an example on one of your authors’ Windows computer:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

As you can see, the stable toolchain is the default. Most Rust users use stable most of the time. You might want to use stable most of the time, but use nightly on a specific project, because you care about a cutting-edge feature. To do so, you can use "rustup override" in that project’s directory to set the nightly toolchain as the one "rustup" should use when you’re in that directory:

```bash
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Now, every time you call "rustc" or "cargo" inside of *~/projects/needs-nightly*, "rustup" will make sure that you are using nightly Rust, rather than your default of stable Rust. This comes in handy when you have a lot of Rust projects!

### [The RFC Process and Teams](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#the-rfc-process-and-teams)

So how do you learn about these new features? Rust’s development model follows a *Request For Comments (RFC) process*. If you’d like an improvement in Rust, you can write up a proposal, called an RFC.

Anyone can write RFCs to improve Rust, and the proposals are reviewed and discussed by the Rust team, which is comprised of many topic subteams. There’s a full list of the teams [on Rust’s website](https://www.rust-lang.org/governance), which includes teams for each area of the project: language design, compiler implementation, infrastructure, documentation, and more. The appropriate team reads the proposal and the comments, writes some comments of their own, and eventually, there’s consensus to accept or reject the feature.

If the feature is accepted, an issue is opened on the Rust repository, and someone can implement it. The person who implements it very well may not be the person who proposed the feature in the first place! When the implementation is ready, it lands on the "master" branch behind a feature gate, as we discussed in the [“Unstable Features”](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#unstable-features) section.

After some time, once Rust developers who use nightly releases have been able to try out the new feature, team members will discuss the feature, how it’s worked out on nightly, and decide if it should make it into stable Rust or not. If the decision is to move forward, the feature gate is removed, and the feature is now considered stable! It rides the trains into a new stable release of Rust.





