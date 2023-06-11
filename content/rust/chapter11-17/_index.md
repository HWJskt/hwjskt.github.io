+++
title = "11-17"
weight = 110
sort_by = "weight"
template ="book_section_rust.html"

+++



# [Writing Automated Tests](https://doc.rust-lang.org/book/ch11-00-testing.html#writing-automated-tests)

In his 1972 essay “The Humble Programmer,” Edsger W. Dijkstra said that “Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.” That doesn’t mean we shouldn’t try to test as much as we can!

Correctness in our programs is the extent to which our code does what we intend it to do. Rust is designed with a high degree of concern about the correctness of programs, but correctness is complex and not easy to prove. Rust’s type system shoulders a huge part of this burden, but the type system cannot catch everything. As such, Rust includes support for writing automated software tests.

Say we write a function "add_two" that adds 2 to whatever number is passed to it. This function’s signature accepts an integer as a parameter and returns an integer as a result. When we implement and compile that function, Rust does all the type checking and borrow checking that you’ve learned so far to ensure that, for instance, we aren’t passing a "String" value or an invalid reference to this function. But Rust *can’t* check that this function will do precisely what we intend, which is return the parameter plus 2 rather than, say, the parameter plus 10 or the parameter minus 50! That’s where tests come in.

We can write tests that assert, for example, that when we pass "3" to the "add_two" function, the returned value is "5". We can run these tests whenever we make changes to our code to make sure any existing correct behavior has not changed.

Testing is a complex skill: although we can’t cover every detail about how to write good tests in one chapter, we’ll discuss the mechanics of Rust’s testing facilities. We’ll talk about the annotations and macros available to you when writing your tests, the default behavior and options provided for running your tests, and how to organize tests into unit tests and integration tests.





---





## [How to Write Tests](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#how-to-write-tests)

Tests are Rust functions that verify that the non-test code is functioning in the expected manner. The bodies of test functions typically perform these three actions:

1. Set up any needed data or state.
2. Run the code you want to test.
3. Assert the results are what you expect.

Let’s look at the features Rust provides specifically for writing tests that take these actions, which include the "test" attribute, a few macros, and the "should_panic" attribute.

### [The Anatomy of a Test Function](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#the-anatomy-of-a-test-function)

At its simplest, a test in Rust is a function that’s annotated with the "test" attribute. Attributes are metadata about pieces of Rust code; one example is the "derive" attribute we used with structs in Chapter 5. To change a function into a test function, add "#[test]" on the line before "fn". When you run your tests with the "cargo test" command, Rust builds a test runner binary that runs the annotated functions and reports on whether each test function passes or fails.

Whenever we make a new library project with Cargo, a test module with a test function in it is automatically generated for us. This module gives you a template for writing your tests so you don’t have to look up the exact structure and syntax every time you start a new project. You can add as many additional test functions and as many test modules as you want!

We’ll explore some aspects of how tests work by experimenting with the template test before we actually test any code. Then we’ll write some real-world tests that call some code that we’ve written and assert that its behavior is correct.

Let’s create a new library project called "adder" that will add two numbers:

```bash
$ cargo new adder --lib
     Created library "adder" project
$ cd adder
```

The contents of the *src/lib.rs* file in your "adder" library should look like Listing 11-1.

Filename: src/lib.rs

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

Listing 11-1: The test module and function generated automatically by "cargo new"

For now, let’s ignore the top two lines and focus on the function. Note the "#[test]" annotation: this attribute indicates this is a test function, so the test runner knows to treat this function as a test. We might also have non-test functions in the "tests" module to help set up common scenarios or perform common operations, so we always need to indicate which functions are tests.

The example function body uses the "assert_eq!" macro to assert that "result", which contains the result of adding 2 and 2, equals 4. This assertion serves as an example of the format for a typical test. Let’s run it to see that this test passes.

The "cargo test" command runs all tests in our project, as shown in Listing 11-2.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Listing 11-2: The output from running the automatically generated test

Cargo compiled and ran the test. We see the line "running 1 test". The next line shows the name of the generated test function, called "it_works", and that the result of running that test is "ok". The overall summary "test result: ok." means that all the tests passed, and the portion that reads "1 passed; 0 failed" totals the number of tests that passed or failed.

It’s possible to mark a test as ignored so it doesn’t run in a particular instance; we’ll cover that in the [“Ignoring Some Tests Unless Specifically Requested”](https://doc.rust-lang.org/book/ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested) section later in this chapter. Because we haven’t done that here, the summary shows "0 ignored". We can also pass an argument to the "cargo test" command to run only tests whose name matches a string; this is called *filtering* and we’ll cover that in the [“Running a Subset of Tests by Name”](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-a-subset-of-tests-by-name) section. We also haven’t filtered the tests being run, so the end of the summary shows "0 filtered out".

The "0 measured" statistic is for benchmark tests that measure performance. Benchmark tests are, as of this writing, only available in nightly Rust. See [the documentation about benchmark tests](https://doc.rust-lang.org/unstable-book/library-features/test.html) to learn more.

The next part of the test output starting at "Doc-tests adder" is for the results of any documentation tests. We don’t have any documentation tests yet, but Rust can compile any code examples that appear in our API documentation. This feature helps keep your docs and your code in sync! We’ll discuss how to write documentation tests in the [“Documentation Comments as Tests”](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests) section of Chapter 14. For now, we’ll ignore the "Doc-tests" output.

Let’s start to customize the test to our own needs. First change the name of the "it_works" function to a different name, such as "exploration", like so:

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Then run "cargo test" again. The output now shows "exploration" instead of "it_works":

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.59s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Now we’ll add another test, but this time we’ll make a test that fails! Tests fail when something in the test function panics. Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed. In Chapter 9, we talked about how the simplest way to panic is to call the "panic!" macro. Enter the new test as a function named "another", so your *src/lib.rs* file looks like Listing 11-3.

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

Listing 11-3: Adding a second test that will fail because we call the "panic!" macro

Run the tests again using "cargo test". The output should look like Listing 11-4, which shows that our "exploration" test passed and "another" failed.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----
thread "tests::another" panicked at "Make this test fail", src/lib.rs:10:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Listing 11-4: Test results when one test passes and one test fails

Instead of "ok", the line "test tests::another" shows "FAILED". Two new sections appear between the individual results and the summary: the first displays the detailed reason for each test failure. In this case, we get the details that "another" failed because it "panicked at "Make this test fail"" on line 10 in the *src/lib.rs* file. The next section lists just the names of all the failing tests, which is useful when there are lots of tests and lots of detailed failing test output. We can use the name of a failing test to run just that test to more easily debug it; we’ll talk more about ways to run tests in the [“Controlling How Tests Are Run”](https://doc.rust-lang.org/book/ch11-02-running-tests.html#controlling-how-tests-are-run) section.

The summary line displays at the end: overall, our test result is "FAILED". We had one test pass and one test fail.

Now that you’ve seen what the test results look like in different scenarios, let’s look at some macros other than "panic!" that are useful in tests.

### [Checking Results with the "assert!" Macro](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#checking-results-with-the-assert-macro)

The "assert!" macro, provided by the standard library, is useful when you want to ensure that some condition in a test evaluates to "true". We give the "assert!" macro an argument that evaluates to a Boolean. If the value is "true", nothing happens and the test passes. If the value is "false", the "assert!" macro calls "panic!" to cause the test to fail. Using the "assert!" macro helps us check that our code is functioning in the way we intend.

In Chapter 5, Listing 5-15, we used a "Rectangle" struct and a "can_hold" method, which are repeated here in Listing 11-5. Let’s put this code in the *src/lib.rs* file, then write some tests for it using the "assert!" macro.

Filename: src/lib.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 11-5: Using the "Rectangle" struct and its "can_hold" method from Chapter 5

The "can_hold" method returns a Boolean, which means it’s a perfect use case for the "assert!" macro. In Listing 11-6, we write a test that exercises the "can_hold" method by creating a "Rectangle" instance that has a width of 8 and a height of 7 and asserting that it can hold another "Rectangle" instance that has a width of 5 and a height of 1.

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```

Listing 11-6: A test for "can_hold" that checks whether a larger rectangle can indeed hold a smaller rectangle

Note that we’ve added a new line inside the "tests" module: "use super::*;". The "tests" module is a regular module that follows the usual visibility rules we covered in Chapter 7 in the [“Paths for Referring to an Item in the Module Tree”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) section. Because the "tests" module is an inner module, we need to bring the code under test in the outer module into the scope of the inner module. We use a glob here so anything we define in the outer module is available to this "tests" module.

We’ve named our test "larger_can_hold_smaller", and we’ve created the two "Rectangle" instances that we need. Then we called the "assert!" macro and passed it the result of calling "larger.can_hold(&smaller)". This expression is supposed to return "true", so our test should pass. Let’s find out!

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

It does pass! Let’s add another test, this time asserting that a smaller rectangle cannot hold a larger rectangle:

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}
```

Because the correct result of the "can_hold" function in this case is "false", we need to negate that result before we pass it to the "assert!" macro. As a result, our test will pass if "can_hold" returns "false":

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... ok
test tests::smaller_cannot_hold_larger ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Two tests that pass! Now let’s see what happens to our test results when we introduce a bug in our code. We’ll change the implementation of the "can_hold" method by replacing the greater-than sign with a less-than sign when it compares the widths:

```rust
// --snip--
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}
```

Running the tests now produces the following:

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... FAILED
test tests::smaller_cannot_hold_larger ... ok

failures:

---- tests::larger_can_hold_smaller stdout ----
thread "tests::larger_can_hold_smaller" panicked at "assertion failed: larger.can_hold(&smaller)", src/lib.rs:28:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Our tests caught the bug! Because "larger.width" is 8 and "smaller.width" is 5, the comparison of the widths in "can_hold" now returns "false": 8 is not less than 5.

### [Testing Equality with the "assert_eq!" and "assert_ne!" Macros](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#testing-equality-with-the-assert_eq-and-assert_ne-macros)

A common way to verify functionality is to test for equality between the result of the code under test and the value you expect the code to return. You could do this using the "assert!" macro and passing it an expression using the "==" operator. However, this is such a common test that the standard library provides a pair of macros—"assert_eq!" and "assert_ne!"—to perform this test more conveniently. These macros compare two arguments for equality or inequality, respectively. They’ll also print the two values if the assertion fails, which makes it easier to see *why* the test failed; conversely, the "assert!" macro only indicates that it got a "false" value for the "==" expression, without printing the values that led to the "false" value.

In Listing 11-7, we write a function named "add_two" that adds "2" to its parameter, then we test this function using the "assert_eq!" macro.

Filename: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

Listing 11-7: Testing the function "add_two" using the "assert_eq!" macro

Let’s check that it passes!

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

We pass "4" as the argument to "assert_eq!", which is equal to the result of calling "add_two(2)". The line for this test is "test tests::it_adds_two ... ok", and the "ok" text indicates that our test passed!

Let’s introduce a bug into our code to see what "assert_eq!" looks like when it fails. Change the implementation of the "add_two" function to instead add "3":

```rust
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Run the tests again:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread "tests::it_adds_two" panicked at "assertion failed: "(left == right)"
  left: "4",
 right: "5"", src/lib.rs:11:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Our test caught the bug! The "it_adds_two" test failed, and the message tells us that the assertion that fails was "assertion failed: "(left == right)"" and what the "left" and "right" values are. This message helps us start debugging: the "left" argument was "4" but the "right" argument, where we had "add_two(2)", was "5". You can imagine that this would be especially helpful when we have a lot of tests going on.

Note that in some languages and test frameworks, the parameters to equality assertion functions are called "expected" and "actual", and the order in which we specify the arguments matters. However, in Rust, they’re called "left" and "right", and the order in which we specify the value we expect and the value the code produces doesn’t matter. We could write the assertion in this test as "assert_eq!(add_two(2), 4)", which would result in the same failure message that displays "assertion failed: "(left == right)"".

The "assert_ne!" macro will pass if the two values we give it are not equal and fail if they’re equal. This macro is most useful for cases when we’re not sure what a value *will* be, but we know what the value definitely *shouldn’t* be. For example, if we’re testing a function that is guaranteed to change its input in some way, but the way in which the input is changed depends on the day of the week that we run our tests, the best thing to assert might be that the output of the function is not equal to the input.

Under the surface, the "assert_eq!" and "assert_ne!" macros use the operators "==" and "!=", respectively. When the assertions fail, these macros print their arguments using debug formatting, which means the values being compared must implement the "PartialEq" and "Debug" traits. All primitive types and most of the standard library types implement these traits. For structs and enums that you define yourself, you’ll need to implement "PartialEq" to assert equality of those types. You’ll also need to implement "Debug" to print the values when the assertion fails. Because both traits are derivable traits, as mentioned in Listing 5-12 in Chapter 5, this is usually as straightforward as adding the "#[derive(PartialEq, Debug)]" annotation to your struct or enum definition. See Appendix C, [“Derivable Traits,”](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) for more details about these and other derivable traits.

### [Adding Custom Failure Messages](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#adding-custom-failure-messages)

You can also add a custom message to be printed with the failure message as optional arguments to the "assert!", "assert_eq!", and "assert_ne!" macros. Any arguments specified after the required arguments are passed along to the "format!" macro (discussed in Chapter 8 in the [“Concatenation with the "+" Operator or the "format!" Macro”](https://doc.rust-lang.org/book/ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro) section), so you can pass a format string that contains "{}" placeholders and values to go in those placeholders. Custom messages are useful for documenting what an assertion means; when a test fails, you’ll have a better idea of what the problem is with the code.

For example, let’s say we have a function that greets people by name and we want to test that the name we pass into the function appears in the output:

Filename: src/lib.rs

```rust
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

The requirements for this program haven’t been agreed upon yet, and we’re pretty sure the "Hello" text at the beginning of the greeting will change. We decided we don’t want to have to update the test when the requirements change, so instead of checking for exact equality to the value returned from the "greeting" function, we’ll just assert that the output contains the text of the input parameter.

Now let’s introduce a bug into this code by changing "greeting" to exclude "name" to see what the default test failure looks like:

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Running this test produces the following:

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread "tests::greeting_contains_name" panicked at "assertion failed: result.contains(\"Carol\")", src/lib.rs:12:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

This result just indicates that the assertion failed and which line the assertion is on. A more useful failure message would print the value from the "greeting" function. Let’s add a custom failure message composed of a format string with a placeholder filled in with the actual value we got from the "greeting" function:

```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was "{}"",
            result
        );
    }
```

Now when we run the test, we’ll get a more informative error message:

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread "tests::greeting_contains_name" panicked at "Greeting did not contain name, value was "Hello!"", src/lib.rs:12:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

We can see the value we actually got in the test output, which would help us debug what happened instead of what we were expecting to happen.

### [Checking for Panics with "should_panic"](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#checking-for-panics-with-should_panic)

In addition to checking return values, it’s important to check that our code handles error conditions as we expect. For example, consider the "Guess" type that we created in Chapter 9, Listing 9-13. Other code that uses "Guess" depends on the guarantee that "Guess" instances will contain only values between 1 and 100. We can write a test that ensures that attempting to create a "Guess" instance with a value outside that range panics.

We do this by adding the attribute "should_panic" to our test function. The test passes if the code inside the function panics; the test fails if the code inside the function doesn’t panic.

Listing 11-8 shows a test that checks that the error conditions of "Guess::new" happen when we expect them to.

Filename: src/lib.rs

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

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

Listing 11-8: Testing that a condition will cause a "panic!"

We place the "#[should_panic]" attribute after the "#[test]" attribute and before the test function it applies to. Let’s look at the result when this test passes:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Looks good! Now let’s introduce a bug in our code by removing the condition that the "new" function will panic if the value is greater than 100:

```rust
// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}
```

When we run the test in Listing 11-8, it will fail:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

We don’t get a very helpful message in this case, but when we look at the test function, we see that it’s annotated with "#[should_panic]". The failure we got means that the code in the test function did not cause a panic.

Tests that use "should_panic" can be imprecise. A "should_panic" test would pass even if the test panics for a different reason from the one we were expecting. To make "should_panic" tests more precise, we can add an optional "expected" parameter to the "should_panic" attribute. The test harness will make sure that the failure message contains the provided text. For example, consider the modified code for "Guess" in Listing 11-9 where the "new" function panics with different messages depending on whether the value is too small or too large.

Filename: src/lib.rs

```rust
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

Listing 11-9: Testing for a "panic!" with a panic message containing a specified substring

This test will pass because the value we put in the "should_panic" attribute’s "expected" parameter is a substring of the message that the "Guess::new" function panics with. We could have specified the entire panic message that we expect, which in this case would be "Guess value must be less than or equal to 100, got 200." What you choose to specify depends on how much of the panic message is unique or dynamic and how precise you want your test to be. In this case, a substring of the panic message is enough to ensure that the code in the test function executes the "else if value > 100" case.

To see what happens when a "should_panic" test with an "expected" message fails, let’s again introduce a bug into our code by swapping the bodies of the "if value < 1" and the "else if value > 100" blocks:

```rust
        if value < 1 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        }
```

This time when we run the "should_panic" test, it will fail:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread "tests::greater_than_100" panicked at "Guess value must be greater than or equal to 1, got 200.", src/lib.rs:13:13
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
note: panic did not contain expected string
      panic message: ""Guess value must be greater than or equal to 1, got 200."",
 expected substring: ""less than or equal to 100""

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

The failure message indicates that this test did indeed panic as we expected, but the panic message did not include the expected string ""Guess value must be less than or equal to 100"". The panic message that we did get in this case was "Guess value must be greater than or equal to 1, got 200." Now we can start figuring out where our bug is!

### [Using "Result" in Tests](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#using-resultt-e-in-tests)

Our tests so far all panic when they fail. We can also write tests that use "Result<T, E>"! Here’s the test from Listing 11-1, rewritten to use "Result<T, E>" and return an "Err" instead of panicking:

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

The "it_works" function now has the "Result<(), String>" return type. In the body of the function, rather than calling the "assert_eq!" macro, we return "Ok(())" when the test passes and an "Err" with a "String" inside when the test fails.

Writing tests so they return a "Result<T, E>" enables you to use the question mark operator in the body of tests, which can be a convenient way to write tests that should fail if any operation within them returns an "Err" variant.

You can’t use the "#[should_panic]" annotation on tests that use "Result<T, E>". To assert that an operation returns an "Err" variant, *don’t* use the question mark operator on the "Result<T, E>" value. Instead, use "assert!(value.is_err())".

Now that you know several ways to write tests, let’s look at what is happening when we run our tests and explore the different options we can use with "cargo test".



---





## [Controlling How Tests Are Run](https://doc.rust-lang.org/book/ch11-02-running-tests.html#controlling-how-tests-are-run)

Just as "cargo run" compiles your code and then runs the resulting binary, "cargo test" compiles your code in test mode and runs the resulting test binary. The default behavior of the binary produced by "cargo test" is to run all the tests in parallel and capture output generated during test runs, preventing the output from being displayed and making it easier to read the output related to the test results. You can, however, specify command line options to change this default behavior.

Some command line options go to "cargo test", and some go to the resulting test binary. To separate these two types of arguments, you list the arguments that go to "cargo test" followed by the separator "--" and then the ones that go to the test binary. Running "cargo test --help" displays the options you can use with "cargo test", and running "cargo test -- --help" displays the options you can use after the separator.

### [Running Tests in Parallel or Consecutively](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-tests-in-parallel-or-consecutively)

When you run multiple tests, by default they run in parallel using threads, meaning they finish running faster and you get feedback quicker. Because the tests are running at the same time, you must make sure your tests don’t depend on each other or on any shared state, including a shared environment, such as the current working directory or environment variables.

For example, say each of your tests runs some code that creates a file on disk named *test-output.txt* and writes some data to that file. Then each test reads the data in that file and asserts that the file contains a particular value, which is different in each test. Because the tests run at the same time, one test might overwrite the file in the time between another test writing and reading the file. The second test will then fail, not because the code is incorrect but because the tests have interfered with each other while running in parallel. One solution is to make sure each test writes to a different file; another solution is to run the tests one at a time.

If you don’t want to run the tests in parallel or if you want more fine-grained control over the number of threads used, you can send the "--test-threads" flag and the number of threads you want to use to the test binary. Take a look at the following example:

```bash
$ cargo test -- --test-threads=1
```

We set the number of test threads to "1", telling the program not to use any parallelism. Running the tests using one thread will take longer than running them in parallel, but the tests won’t interfere with each other if they share state.

### [Showing Function Output](https://doc.rust-lang.org/book/ch11-02-running-tests.html#showing-function-output)

By default, if a test passes, Rust’s test library captures anything printed to standard output. For example, if we call "println!" in a test and the test passes, we won’t see the "println!" output in the terminal; we’ll see only the line that indicates the test passed. If a test fails, we’ll see whatever was printed to standard output with the rest of the failure message.

As an example, Listing 11-10 has a silly function that prints the value of its parameter and returns 10, as well as a test that passes and a test that fails.

Filename: src/lib.rs

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

Listing 11-10: Tests for a function that calls "println!"

When we run these tests with "cargo test", we’ll see the following output:

```bash
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread "tests::this_test_will_fail" panicked at "assertion failed: "(left == right)"
  left: "5",
 right: "10"", src/lib.rs:19:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Note that nowhere in this output do we see "I got the value 4", which is what is printed when the test that passes runs. That output has been captured. The output from the test that failed, "I got the value 8", appears in the section of the test summary output, which also shows the cause of the test failure.

If we want to see printed values for passing tests as well, we can tell Rust to also show the output of successful tests with "--show-output".

```bash
$ cargo test -- --show-output
```

When we run the tests in Listing 11-10 again with the "--show-output" flag, we see the following output:

```bash
$ cargo test -- --show-output
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread "tests::this_test_will_fail" panicked at "assertion failed: "(left == right)"
  left: "5",
 right: "10"", src/lib.rs:19:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

### [Running a Subset of Tests by Name](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-a-subset-of-tests-by-name)

Sometimes, running a full test suite can take a long time. If you’re working on code in a particular area, you might want to run only the tests pertaining to that code. You can choose which tests to run by passing "cargo test" the name or names of the test(s) you want to run as an argument.

To demonstrate how to run a subset of tests, we’ll first create three tests for our "add_two" function, as shown in Listing 11-11, and choose which ones to run.

Filename: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

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
}
```

Listing 11-11: Three tests with three different names

If we run the tests without passing any arguments, as we saw earlier, all the tests will run in parallel:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 3 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

#### [Running Single Tests](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-single-tests)

We can pass the name of any test function to "cargo test" to run only that test:

```bash
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

Only the test with the name "one_hundred" ran; the other two tests didn’t match that name. The test output lets us know we had more tests that didn’t run by displaying "2 filtered out" at the end.

We can’t specify the names of multiple tests in this way; only the first value given to "cargo test" will be used. But there is a way to run multiple tests.

#### [Filtering to Run Multiple Tests](https://doc.rust-lang.org/book/ch11-02-running-tests.html#filtering-to-run-multiple-tests)

We can specify part of a test name, and any test whose name matches that value will be run. For example, because two of our tests’ names contain "add", we can run those two by running "cargo test add":

```bash
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

This command ran all tests with "add" in the name and filtered out the test named "one_hundred". Also note that the module in which a test appears becomes part of the test’s name, so we can run all the tests in a module by filtering on the module’s name.

### [Ignoring Some Tests Unless Specifically Requested](https://doc.rust-lang.org/book/ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested)

Sometimes a few specific tests can be very time-consuming to execute, so you might want to exclude them during most runs of "cargo test". Rather than listing as arguments all tests you do want to run, you can instead annotate the time-consuming tests using the "ignore" attribute to exclude them, as shown here:

Filename: src/lib.rs

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

After "#[test]" we add the "#[ignore]" line to the test we want to exclude. Now when we run our tests, "it_works" runs, but "expensive_test" doesn’t:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The "expensive_test" function is listed as "ignored". If we want to run only the ignored tests, we can use "cargo test -- --ignored":

```bash
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

By controlling which tests run, you can make sure your "cargo test" results will be fast. When you’re at a point where it makes sense to check the results of the "ignored" tests and you have time to wait for the results, you can run "cargo test -- --ignored" instead. If you want to run all tests whether they’re ignored or not, you can run "cargo test -- --include-ignored".





---





## [Test Organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html#test-organization)

As mentioned at the start of the chapter, testing is a complex discipline, and different people use different terminology and organization. The Rust community thinks about tests in terms of two main categories: unit tests and integration tests. *Unit tests* are small and more focused, testing one module in isolation at a time, and can test private interfaces. *Integration tests* are entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

Writing both kinds of tests is important to ensure that the pieces of your library are doing what you expect them to, separately and together.

### [Unit Tests](https://doc.rust-lang.org/book/ch11-03-test-organization.html#unit-tests)

The purpose of unit tests is to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn’t working as expected. You’ll put unit tests in the *src* directory in each file with the code that they’re testing. The convention is to create a module named "tests" in each file to contain the test functions and to annotate the module with "cfg(test)".

#### [The Tests Module and "#[cfg(test)\]"](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest)

The "#[cfg(test)]" annotation on the tests module tells Rust to compile and run the test code only when you run "cargo test", not when you run "cargo build". This saves compile time when you only want to build the library and saves space in the resulting compiled artifact because the tests are not included. You’ll see that because integration tests go in a different directory, they don’t need the "#[cfg(test)]" annotation. However, because unit tests go in the same files as the code, you’ll use "#[cfg(test)]" to specify that they shouldn’t be included in the compiled result.

Recall that when we generated the new "adder" project in the first section of this chapter, Cargo generated this code for us:

Filename: src/lib.rs

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

This code is the automatically generated test module. The attribute "cfg" stands for *configuration* and tells Rust that the following item should only be included given a certain configuration option. In this case, the configuration option is "test", which is provided by Rust for compiling and running tests. By using the "cfg" attribute, Cargo compiles our test code only if we actively run the tests with "cargo test". This includes any helper functions that might be within this module, in addition to the functions annotated with "#[test]".

#### [Testing Private Functions](https://doc.rust-lang.org/book/ch11-03-test-organization.html#testing-private-functions)

There’s debate within the testing community about whether or not private functions should be tested directly, and other languages make it difficult or impossible to test private functions. Regardless of which testing ideology you adhere to, Rust’s privacy rules do allow you to test private functions. Consider the code in Listing 11-12 with the private function "internal_adder".

Filename: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

Listing 11-12: Testing a private function

Note that the "internal_adder" function is not marked as "pub". Tests are just Rust code, and the "tests" module is just another module. As we discussed in the [“Paths for Referring to an Item in the Module Tree”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) section, items in child modules can use the items in their ancestor modules. In this test, we bring all of the "test" module’s parent’s items into scope with "use super::*", and then the test can call "internal_adder". If you don’t think private functions should be tested, there’s nothing in Rust that will compel you to do so.

### [Integration Tests](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests)

In Rust, integration tests are entirely external to your library. They use your library in the same way any other code would, which means they can only call functions that are part of your library’s public API. Their purpose is to test whether many parts of your library work together correctly. Units of code that work correctly on their own could have problems when integrated, so test coverage of the integrated code is important as well. To create integration tests, you first need a *tests* directory.

#### [The *tests* Directory](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-directory)

We create a *tests* directory at the top level of our project directory, next to *src*. Cargo knows to look for integration test files in this directory. We can then make as many test files as we want, and Cargo will compile each of the files as an individual crate.

Let’s create an integration test. With the code in Listing 11-12 still in the *src/lib.rs* file, make a *tests* directory, and create a new file named *tests/integration_test.rs*. Your directory structure should look like this:

```
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Enter the code in Listing 11-13 into the *tests/integration_test.rs* file:

Filename: tests/integration_test.rs

```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

Listing 11-13: An integration test of a function in the "adder" crate

Each file in the "tests" directory is a separate crate, so we need to bring our library into each test crate’s scope. For that reason we add "use adder" at the top of the code, which we didn’t need in the unit tests.

We don’t need to annotate any code in *tests/integration_test.rs* with "#[cfg(test)]". Cargo treats the "tests" directory specially and compiles files in this directory only when we run "cargo test". Run "cargo test" now:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests src/lib.rs (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The three sections of output include the unit tests, the integration test, and the doc tests. Note that if any test in a section fails, the following sections will not be run. For example, if a unit test fails, there won’t be any output for integration and doc tests because those tests will only be run if all unit tests are passing.

The first section for the unit tests is the same as we’ve been seeing: one line for each unit test (one named "internal" that we added in Listing 11-12) and then a summary line for the unit tests.

The integration tests section starts with the line "Running tests/integration_test.rs". Next, there is a line for each test function in that integration test and a summary line for the results of the integration test just before the "Doc-tests adder" section starts.

Each integration test file has its own section, so if we add more files in the *tests* directory, there will be more integration test sections.

We can still run a particular integration test function by specifying the test function’s name as an argument to "cargo test". To run all the tests in a particular integration test file, use the "--test" argument of "cargo test" followed by the name of the file:

```bash
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

This command runs only the tests in the *tests/integration_test.rs* file.

#### [Submodules in Integration Tests](https://doc.rust-lang.org/book/ch11-03-test-organization.html#submodules-in-integration-tests)

As you add more integration tests, you might want to make more files in the *tests* directory to help organize them; for example, you can group the test functions by the functionality they’re testing. As mentioned earlier, each file in the *tests* directory is compiled as its own separate crate, which is useful for creating separate scopes to more closely imitate the way end users will be using your crate. However, this means files in the *tests* directory don’t share the same behavior as files in *src* do, as you learned in Chapter 7 regarding how to separate code into modules and files.

The different behavior of *tests* directory files is most noticeable when you have a set of helper functions to use in multiple integration test files and you try to follow the steps in the [“Separating Modules into Different Files”](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html) section of Chapter 7 to extract them into a common module. For example, if we create *tests/common.rs* and place a function named "setup" in it, we can add some code to "setup" that we want to call from multiple test functions in multiple test files:

Filename: tests/common.rs

```rust
pub fn setup() {
    // setup code specific to your library"s tests would go here
}
```

When we run the tests again, we’ll see a new section in the test output for the *common.rs* file, even though this file doesn’t contain any test functions nor did we call the "setup" function from anywhere:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Having "common" appear in the test results with "running 0 tests" displayed for it is not what we wanted. We just wanted to share some code with the other integration test files.

To avoid having "common" appear in the test output, instead of creating *tests/common.rs*, we’ll create *tests/common/mod.rs*. The project directory now looks like this:

```
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

This is the older naming convention that Rust also understands that we mentioned in the [“Alternate File Paths”](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths) section of Chapter 7. Naming the file this way tells Rust not to treat the "common" module as an integration test file. When we move the "setup" function code into *tests/common/mod.rs* and delete the *tests/common.rs* file, the section in the test output will no longer appear. Files in subdirectories of the *tests* directory don’t get compiled as separate crates or have sections in the test output.

After we’ve created *tests/common/mod.rs*, we can use it from any of the integration test files as a module. Here’s an example of calling the "setup" function from the "it_adds_two" test in *tests/integration_test.rs*:

Filename: tests/integration_test.rs

```rust
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Note that the "mod common;" declaration is the same as the module declaration we demonstrated in Listing 7-21. Then in the test function, we can call the "common::setup()" function.

#### [Integration Tests for Binary Crates](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests-for-binary-crates)

If our project is a binary crate that only contains a *src/main.rs* file and doesn’t have a *src/lib.rs* file, we can’t create integration tests in the *tests* directory and bring functions defined in the *src/main.rs* file into scope with a "use" statement. Only library crates expose functions that other crates can use; binary crates are meant to be run on their own.

This is one of the reasons Rust projects that provide a binary have a straightforward *src/main.rs* file that calls logic that lives in the *src/lib.rs* file. Using that structure, integration tests *can* test the library crate with "use" to make the important functionality available. If the important functionality works, the small amount of code in the *src/main.rs* file will work as well, and that small amount of code doesn’t need to be tested.

## [Summary](https://doc.rust-lang.org/book/ch11-03-test-organization.html#summary)

Rust’s testing features provide a way to specify how code should function to ensure it continues to work as you expect, even as you make changes. Unit tests exercise different parts of a library separately and can test private implementation details. Integration tests check that many parts of the library work together correctly, and they use the library’s public API to test the code in the same way external code will use it. Even though Rust’s type system and ownership rules help prevent some kinds of bugs, tests are still important to reduce logic bugs having to do with how your code is expected to behave.

Let’s combine the knowledge you learned in this chapter and in previous chapters to work on a project!





---





# 12



# [An I/O Project: Building a Command Line Program](https://doc.rust-lang.org/book/ch12-00-an-io-project.html#an-io-project-building-a-command-line-program)

This chapter is a recap of the many skills you’ve learned so far and an exploration of a few more standard library features. We’ll build a command line tool that interacts with file and command line input/output to practice some of the Rust concepts you now have under your belt.

Rust’s speed, safety, single binary output, and cross-platform support make it an ideal language for creating command line tools, so for our project, we’ll make our own version of the classic command line search tool "grep" (**g**lobally search a **r**egular **e**xpression and **p**rint). In the simplest use case, "grep" searches a specified file for a specified string. To do so, "grep" takes as its arguments a file path and a string. Then it reads the file, finds lines in that file that contain the string argument, and prints those lines.

Along the way, we’ll show how to make our command line tool use the terminal features that many other command line tools use. We’ll read the value of an environment variable to allow the user to configure the behavior of our tool. We’ll also print error messages to the standard error bash stream ("stderr") instead of standard output ("stdout"), so, for example, the user can redirect successful output to a file while still seeing error messages onscreen.

One Rust community member, Andrew Gallant, has already created a fully featured, very fast version of "grep", called "ripgrep". By comparison, our version will be fairly simple, but this chapter will give you some of the background knowledge you need to understand a real-world project such as "ripgrep".

Our "grep" project will combine a number of concepts you’ve learned so far:

- Organizing code (using what you learned about modules in [Chapter 7](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html))
- Using vectors and strings (collections, [Chapter 8](https://doc.rust-lang.org/book/ch08-00-common-collections.html))
- Handling errors ([Chapter 9](https://doc.rust-lang.org/book/ch09-00-error-handling.html))
- Using traits and lifetimes where appropriate ([Chapter 10](https://doc.rust-lang.org/book/ch10-00-generics.html))
- Writing tests ([Chapter 11](https://doc.rust-lang.org/book/ch11-00-testing.html))

We’ll also briefly introduce closures, iterators, and trait objects, which Chapters [13](https://doc.rust-lang.org/book/ch13-00-functional-features.html) and [17](https://doc.rust-lang.org/book/ch17-00-oop.html) will cover in detail.





---





## [Accepting Command Line Arguments](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#accepting-command-line-arguments)

Let’s create a new project with, as always, "cargo new". We’ll call our project "minigrep" to distinguish it from the "grep" tool that you might already have on your system.

```bash
$ cargo new minigrep
     Created binary (application) "minigrep" project
$ cd minigrep
```

The first task is to make "minigrep" accept its two command line arguments: the file path and a string to search for. That is, we want to be able to run our program with "cargo run", two hyphens to indicate the following arguments are for our program rather than for "cargo", a string to search for, and a path to a file to search in, like so:

```bash
$ cargo run -- searchstring example-filename.txt
```

Right now, the program generated by "cargo new" cannot process arguments we give it. Some existing libraries on [crates.io](https://crates.io/) can help with writing a program that accepts command line arguments, but because you’re just learning this concept, let’s implement this capability ourselves.

### [Reading the Argument Values](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#reading-the-argument-values)

To enable "minigrep" to read the values of command line arguments we pass to it, we’ll need the "std::env::args" function provided in Rust’s standard library. This function returns an iterator of the command line arguments passed to "minigrep". We’ll cover iterators fully in [Chapter 13](https://doc.rust-lang.org/book/ch13-00-functional-features.html). For now, you only need to know two details about iterators: iterators produce a series of values, and we can call the "collect" method on an iterator to turn it into a collection, such as a vector, that contains all the elements the iterator produces.

The code in Listing 12-1 allows your "minigrep" program to read any command line arguments passed to it and then collect the values into a vector.

Filename: src/main.rs

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    dbg!(args);
}
```

Listing 12-1: Collecting the command line arguments into a vector and printing them

First, we bring the "std::env" module into scope with a "use" statement so we can use its "args" function. Notice that the "std::env::args" function is nested in two levels of modules. As we discussed in [Chapter 7](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths), in cases where the desired function is nested in more than one module, we’ve chosen to bring the parent module into scope rather than the function. By doing so, we can easily use other functions from "std::env". It’s also less ambiguous than adding "use std::env::args" and then calling the function with just "args", because "args" might easily be mistaken for a function that’s defined in the current module.

> ### [The "args" Function and Invalid Unicode](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#the-args-function-and-invalid-unicode)
>
> Note that "std::env::args" will panic if any argument contains invalid Unicode. If your program needs to accept arguments containing invalid Unicode, use "std::env::args_os" instead. That function returns an iterator that produces "OsString" values instead of "String" values. We’ve chosen to use "std::env::args" here for simplicity, because "OsString" values differ per platform and are more complex to work with than "String" values.

On the first line of "main", we call "env::args", and we immediately use "collect" to turn the iterator into a vector containing all the values produced by the iterator. We can use the "collect" function to create many kinds of collections, so we explicitly annotate the type of "args" to specify that we want a vector of strings. Although we very rarely need to annotate types in Rust, "collect" is one function you do often need to annotate because Rust isn’t able to infer the kind of collection you want.

Finally, we print the vector using the debug macro. Let’s try running the code first with no arguments and then with two arguments:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running "target/debug/minigrep"
[src/main.rs:5] args = [
    "target/debug/minigrep",
]
$ cargo run -- needle haystack
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 1.57s
     Running "target/debug/minigrep needle haystack"
[src/main.rs:5] args = [
    "target/debug/minigrep",
    "needle",
    "haystack",
]
```

Notice that the first value in the vector is ""target/debug/minigrep"", which is the name of our binary. This matches the behavior of the arguments list in C, letting programs use the name by which they were invoked in their execution. It’s often convenient to have access to the program name in case you want to print it in messages or change behavior of the program based on what command line alias was used to invoke the program. But for the purposes of this chapter, we’ll ignore it and save only the two arguments we need.

### [Saving the Argument Values in Variables](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#saving-the-argument-values-in-variables)

The program is currently able to access the values specified as command line arguments. Now we need to save the values of the two arguments in variables so we can use the values throughout the rest of the program. We do that in Listing 12-2.

Filename: src/main.rs

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", file_path);
}
```

Listing 12-2: Creating variables to hold the query argument and file path argument

As we saw when we printed the vector, the program’s name takes up the first value in the vector at "args[0]", so we’re starting arguments at index "1". The first argument "minigrep" takes is the string we’re searching for, so we put a reference to the first argument in the variable "query". The second argument will be the file path, so we put a reference to the second argument in the variable "file_path".

We temporarily print the values of these variables to prove that the code is working as we intend. Let’s run this program again with the arguments "test" and "sample.txt":

```bash
$ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep test sample.txt"
Searching for test
In file sample.txt
```

Great, the program is working! The values of the arguments we need are being saved into the right variables. Later we’ll add some error handling to deal with certain potential erroneous situations, such as when the user provides no arguments; for now, we’ll ignore that situation and work on adding file-reading capabilities instead.





---





## [Reading a File](https://doc.rust-lang.org/book/ch12-02-reading-a-file.html#reading-a-file)

Now we’ll add functionality to read the file specified in the "file_path" argument. First, we need a sample file to test it with: we’ll use a file with a small amount of text over multiple lines with some repeated words. Listing 12-3 has an Emily Dickinson poem that will work well! Create a file called *poem.txt* at the root level of your project, and enter the poem “I’m Nobody! Who are you?”

Filename: poem.txt

```
I"m nobody! Who are you?
Are you nobody, too?
Then there"s a pair of us - don"t tell!
They"d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Listing 12-3: A poem by Emily Dickinson makes a good test case

With the text in place, edit *src/main.rs* and add code to read the file, as shown in Listing 12-4.

Filename: src/main.rs

```rust
use std::env;
use std::fs;

fn main() {
    // --snip--
    println!("In file {}", file_path);

    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}
```

Listing 12-4: Reading the contents of the file specified by the second argument

First, we bring in a relevant part of the standard library with a "use" statement: we need "std::fs" to handle files.

In "main", the new statement "fs::read_to_string" takes the "file_path", opens that file, and returns a "std::io::Result<String>" of the file’s contents.

After that, we again add a temporary "println!" statement that prints the value of "contents" after the file is read, so we can check that the program is working so far.

Let’s run this code with any string as the first command line argument (because we haven’t implemented the searching part yet) and the *poem.txt* file as the second argument:

```bash
$ cargo run -- the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep the poem.txt"
Searching for the
In file poem.txt
With text:
I"m nobody! Who are you?
Are you nobody, too?
Then there"s a pair of us - don"t tell!
They"d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Great! The code read and then printed the contents of the file. But the code has a few flaws. At the moment, the "main" function has multiple responsibilities: generally, functions are clearer and easier to maintain if each function is responsible for only one idea. The other problem is that we’re not handling errors as well as we could. The program is still small, so these flaws aren’t a big problem, but as the program grows, it will be harder to fix them cleanly. It’s good practice to begin refactoring early on when developing a program, because it’s much easier to refactor smaller amounts of code. We’ll do that next.





---





## [Refactoring to Improve Modularity and Error Handling](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#refactoring-to-improve-modularity-and-error-handling)

To improve our program, we’ll fix four problems that have to do with the program’s structure and how it’s handling potential errors. First, our "main" function now performs two tasks: it parses arguments and reads files. As our program grows, the number of separate tasks the "main" function handles will increase. As a function gains responsibilities, it becomes more difficult to reason about, harder to test, and harder to change without breaking one of its parts. It’s best to separate functionality so each function is responsible for one task.

This issue also ties into the second problem: although "query" and "file_path" are configuration variables to our program, variables like "contents" are used to perform the program’s logic. The longer "main" becomes, the more variables we’ll need to bring into scope; the more variables we have in scope, the harder it will be to keep track of the purpose of each. It’s best to group the configuration variables into one structure to make their purpose clear.

The third problem is that we’ve used "expect" to print an error message when reading the file fails, but the error message just prints "Should have been able to read the file". Reading a file can fail in a number of ways: for example, the file could be missing, or we might not have permission to open it. Right now, regardless of the situation, we’d print the same error message for everything, which wouldn’t give the user any information!

Fourth, we use "expect" repeatedly to handle different errors, and if the user runs our program without specifying enough arguments, they’ll get an "index out of bounds" error from Rust that doesn’t clearly explain the problem. It would be best if all the error-handling code were in one place so future maintainers had only one place to consult the code if the error-handling logic needed to change. Having all the error-handling code in one place will also ensure that we’re printing messages that will be meaningful to our end users.

Let’s address these four problems by refactoring our project.

### [Separation of Concerns for Binary Projects](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#separation-of-concerns-for-binary-projects)

The organizational problem of allocating responsibility for multiple tasks to the "main" function is common to many binary projects. As a result, the Rust community has developed guidelines for splitting the separate concerns of a binary program when "main" starts getting large. This process has the following steps:

- Split your program into a *main.rs* and a *lib.rs* and move your program’s logic to *lib.rs*.
- As long as your command line parsing logic is small, it can remain in *main.rs*.
- When the command line parsing logic starts getting complicated, extract it from *main.rs* and move it to *lib.rs*.

The responsibilities that remain in the "main" function after this process should be limited to the following:

- Calling the command line parsing logic with the argument values
- Setting up any other configuration
- Calling a "run" function in *lib.rs*
- Handling the error if "run" returns an error

This pattern is about separating concerns: *main.rs* handles running the program, and *lib.rs* handles all the logic of the task at hand. Because you can’t test the "main" function directly, this structure lets you test all of your program’s logic by moving it into functions in *lib.rs*. The code that remains in *main.rs* will be small enough to verify its correctness by reading it. Let’s rework our program by following this process.

#### [Extracting the Argument Parser](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#extracting-the-argument-parser)

We’ll extract the functionality for parsing arguments into a function that "main" will call to prepare for moving the command line parsing logic to *src/lib.rs*. Listing 12-5 shows the new start of "main" that calls a new function "parse_config", which we’ll define in *src/main.rs* for the moment.

Filename: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, file_path) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];

    (query, file_path)
}
```

Listing 12-5: Extracting a "parse_config" function from "main"

We’re still collecting the command line arguments into a vector, but instead of assigning the argument value at index 1 to the variable "query" and the argument value at index 2 to the variable "file_path" within the "main" function, we pass the whole vector to the "parse_config" function. The "parse_config" function then holds the logic that determines which argument goes in which variable and passes the values back to "main". We still create the "query" and "file_path" variables in "main", but "main" no longer has the responsibility of determining how the command line arguments and variables correspond.

This rework may seem like overkill for our small program, but we’re refactoring in small, incremental steps. After making this change, run the program again to verify that the argument parsing still works. It’s good to check your progress often, to help identify the cause of problems when they occur.

#### [Grouping Configuration Values](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#grouping-configuration-values)

We can take another small step to improve the "parse_config" function further. At the moment, we’re returning a tuple, but then we immediately break that tuple into individual parts again. This is a sign that perhaps we don’t have the right abstraction yet.

Another indicator that shows there’s room for improvement is the "config" part of "parse_config", which implies that the two values we return are related and are both part of one configuration value. We’re not currently conveying this meaning in the structure of the data other than by grouping the two values into a tuple; we’ll instead put the two values into one struct and give each of the struct fields a meaningful name. Doing so will make it easier for future maintainers of this code to understand how the different values relate to each other and what their purpose is.

Listing 12-6 shows the improvements to the "parse_config" function.

Filename: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    // --snip--
}

struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();

    Config { query, file_path }
}
```

Listing 12-6: Refactoring "parse_config" to return an instance of a "Config" struct

We’ve added a struct named "Config" defined to have fields named "query" and "file_path". The signature of "parse_config" now indicates that it returns a "Config" value. In the body of "parse_config", where we used to return string slices that reference "String" values in "args", we now define "Config" to contain owned "String" values. The "args" variable in "main" is the owner of the argument values and is only letting the "parse_config" function borrow them, which means we’d violate Rust’s borrowing rules if "Config" tried to take ownership of the values in "args".

There are a number of ways we could manage the "String" data; the easiest, though somewhat inefficient, route is to call the "clone" method on the values. This will make a full copy of the data for the "Config" instance to own, which takes more time and memory than storing a reference to the string data. However, cloning the data also makes our code very straightforward because we don’t have to manage the lifetimes of the references; in this circumstance, giving up a little performance to gain simplicity is a worthwhile trade-off.

> ### [The Trade-Offs of Using "clone"](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#the-trade-offs-of-using-clone)
>
> There’s a tendency among many Rustaceans to avoid using "clone" to fix ownership problems because of its runtime cost. In [Chapter 13](https://doc.rust-lang.org/book/ch13-00-functional-features.html), you’ll learn how to use more efficient methods in this type of situation. But for now, it’s okay to copy a few strings to continue making progress because you’ll make these copies only once and your file path and query string are very small. It’s better to have a working program that’s a bit inefficient than to try to hyperoptimize code on your first pass. As you become more experienced with Rust, it’ll be easier to start with the most efficient solution, but for now, it’s perfectly acceptable to call "clone".

We’ve updated "main" so it places the instance of "Config" returned by "parse_config" into a variable named "config", and we updated the code that previously used the separate "query" and "file_path" variables so it now uses the fields on the "Config" struct instead.

Now our code more clearly conveys that "query" and "file_path" are related and that their purpose is to configure how the program will work. Any code that uses these values knows to find them in the "config" instance in the fields named for their purpose.

#### [Creating a Constructor for "Config"](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#creating-a-constructor-for-config)

So far, we’ve extracted the logic responsible for parsing the command line arguments from "main" and placed it in the "parse_config" function. Doing so helped us to see that the "query" and "file_path" values were related and that relationship should be conveyed in our code. We then added a "Config" struct to name the related purpose of "query" and "file_path" and to be able to return the values’ names as struct field names from the "parse_config" function.

So now that the purpose of the "parse_config" function is to create a "Config" instance, we can change "parse_config" from a plain function to a function named "new" that is associated with the "Config" struct. Making this change will make the code more idiomatic. We can create instances of types in the standard library, such as "String", by calling "String::new". Similarly, by changing "parse_config" into a "new" function associated with "Config", we’ll be able to create instances of "Config" by calling "Config::new". Listing 12-7 shows the changes we need to make.

Filename: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

Listing 12-7: Changing "parse_config" into "Config::new"

We’ve updated "main" where we were calling "parse_config" to instead call "Config::new". We’ve changed the name of "parse_config" to "new" and moved it within an "impl" block, which associates the "new" function with "Config". Try compiling this code again to make sure it works.

### [Fixing the Error Handling](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#fixing-the-error-handling)

Now we’ll work on fixing our error handling. Recall that attempting to access the values in the "args" vector at index 1 or index 2 will cause the program to panic if the vector contains fewer than three items. Try running the program without any arguments; it will look like this:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep"
thread "main" panicked at "index out of bounds: the len is 1 but the index is 1", src/main.rs:27:21
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

The line "index out of bounds: the len is 1 but the index is 1" is an error message intended for programmers. It won’t help our end users understand what they should do instead. Let’s fix that now.

#### [Improving the Error Message](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#improving-the-error-message)

In Listing 12-8, we add a check in the "new" function that will verify that the slice is long enough before accessing index 1 and 2. If the slice isn’t long enough, the program panics and displays a better error message.

Filename: src/main.rs

```rust
    // --snip--
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!("not enough arguments");
        }
        // --snip--
```

Listing 12-8: Adding a check for the number of arguments

This code is similar to [the "Guess::new" function we wrote in Listing 9-13](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation), where we called "panic!" when the "value" argument was out of the range of valid values. Instead of checking for a range of values here, we’re checking that the length of "args" is at least 3 and the rest of the function can operate under the assumption that this condition has been met. If "args" has fewer than three items, this condition will be true, and we call the "panic!" macro to end the program immediately.

With these extra few lines of code in "new", let’s run the program without any arguments again to see what the error looks like now:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep"
thread "main" panicked at "not enough arguments", src/main.rs:26:13
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace
```

This output is better: we now have a reasonable error message. However, we also have extraneous information we don’t want to give to our users. Perhaps using the technique we used in Listing 9-13 isn’t the best to use here: a call to "panic!" is more appropriate for a programming problem than a usage problem, [as discussed in Chapter 9](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling). Instead, we’ll use the other technique you learned about in Chapter 9—[returning a "Result"](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html) that indicates either success or an error.



#### [Returning a "Result" Instead of Calling "panic!"](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#returning-a-result-instead-of-calling-panic)

We can instead return a "Result" value that will contain a "Config" instance in the successful case and will describe the problem in the error case. We’re also going to change the function name from "new" to "build" because many programmers expect "new" functions to never fail. When "Config::build" is communicating to "main", we can use the "Result" type to signal there was a problem. Then we can change "main" to convert an "Err" variant into a more practical error for our users without the surrounding text about "thread "main"" and "RUST_BACKTRACE" that a call to "panic!" causes.

Listing 12-9 shows the changes we need to make to the return value of the function we’re now calling "Config::build" and the body of the function needed to return a "Result". Note that this won’t compile until we update "main" as well, which we’ll do in the next listing.

Filename: src/main.rs

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &"static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

Listing 12-9: Returning a "Result" from "Config::build"

Our "build" function returns a "Result" with a "Config" instance in the success case and a "&"static str" in the error case. Our error values will always be string literals that have the ""static" lifetime.

We’ve made two changes in the body of the function: instead of calling "panic!" when the user doesn’t pass enough arguments, we now return an "Err" value, and we’ve wrapped the "Config" return value in an "Ok". These changes make the function conform to its new type signature.

Returning an "Err" value from "Config::build" allows the "main" function to handle the "Result" value returned from the "build" function and exit the process more cleanly in the error case.



#### [Calling "Config::build" and Handling Errors](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#calling-configbuild-and-handling-errors)

To handle the error case and print a user-friendly message, we need to update "main" to handle the "Result" being returned by "Config::build", as shown in Listing 12-10. We’ll also take the responsibility of exiting the command line tool with a nonzero error code away from "panic!" and instead implement it by hand. A nonzero exit status is a convention to signal to the process that called our program that the program exited with an error state.

Filename: src/main.rs

```rust
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
```

Listing 12-10: Exiting with an error code if building a "Config" fails

In this listing, we’ve used a method we haven’t covered in detail yet: "unwrap_or_else", which is defined on "Result<T, E>" by the standard library. Using "unwrap_or_else" allows us to define some custom, non-"panic!" error handling. If the "Result" is an "Ok" value, this method’s behavior is similar to "unwrap": it returns the inner value "Ok" is wrapping. However, if the value is an "Err" value, this method calls the code in the *closure*, which is an anonymous function we define and pass as an argument to "unwrap_or_else". We’ll cover closures in more detail in [Chapter 13](https://doc.rust-lang.org/book/ch13-00-functional-features.html). For now, you just need to know that "unwrap_or_else" will pass the inner value of the "Err", which in this case is the static string ""not enough arguments"" that we added in Listing 12-9, to our closure in the argument "err" that appears between the vertical pipes. The code in the closure can then use the "err" value when it runs.

We’ve added a new "use" line to bring "process" from the standard library into scope. The code in the closure that will be run in the error case is only two lines: we print the "err" value and then call "process::exit". The "process::exit" function will stop the program immediately and return the number that was passed as the exit status code. This is similar to the "panic!"-based handling we used in Listing 12-8, but we no longer get all the extra output. Let’s try it:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running "target/debug/minigrep"
Problem parsing arguments: not enough arguments
```

Great! This output is much friendlier for our users.

### [Extracting Logic from "main"](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#extracting-logic-from-main)

Now that we’ve finished refactoring the configuration parsing, let’s turn to the program’s logic. As we stated in [“Separation of Concerns for Binary Projects”](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#separation-of-concerns-for-binary-projects), we’ll extract a function named "run" that will hold all the logic currently in the "main" function that isn’t involved with setting up configuration or handling errors. When we’re done, "main" will be concise and easy to verify by inspection, and we’ll be able to write tests for all the other logic.

Listing 12-11 shows the extracted "run" function. For now, we’re just making the small, incremental improvement of extracting the function. We’re still defining the function in *src/main.rs*.

Filename: src/main.rs

```rust
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

// --snip--
```

Listing 12-11: Extracting a "run" function containing the rest of the program logic

The "run" function now contains all the remaining logic from "main", starting from reading the file. The "run" function takes the "Config" instance as an argument.

#### [Returning Errors from the "run" Function](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#returning-errors-from-the-run-function)

With the remaining program logic separated into the "run" function, we can improve the error handling, as we did with "Config::build" in Listing 12-9. Instead of allowing the program to panic by calling "expect", the "run" function will return a "Result<T, E>" when something goes wrong. This will let us further consolidate the logic around handling errors into "main" in a user-friendly way. Listing 12-12 shows the changes we need to make to the signature and body of "run".

Filename: src/main.rs

```rust
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{contents}");

    Ok(())
}
```

Listing 12-12: Changing the "run" function to return "Result"

We’ve made three significant changes here. First, we changed the return type of the "run" function to "Result<(), Box<dyn Error>>". This function previously returned the unit type, "()", and we keep that as the value returned in the "Ok" case.

For the error type, we used the *trait object* "Box<dyn Error>" (and we’ve brought "std::error::Error" into scope with a "use" statement at the top). We’ll cover trait objects in [Chapter 17](https://doc.rust-lang.org/book/ch17-00-oop.html). For now, just know that "Box<dyn Error>" means the function will return a type that implements the "Error" trait, but we don’t have to specify what particular type the return value will be. This gives us flexibility to return error values that may be of different types in different error cases. The "dyn" keyword is short for “dynamic.”

Second, we’ve removed the call to "expect" in favor of the "?" operator, as we talked about in [Chapter 9](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator). Rather than "panic!" on an error, "?" will return the error value from the current function for the caller to handle.

Third, the "run" function now returns an "Ok" value in the success case. We’ve declared the "run" function’s success type as "()" in the signature, which means we need to wrap the unit type value in the "Ok" value. This "Ok(())" syntax might look a bit strange at first, but using "()" like this is the idiomatic way to indicate that we’re calling "run" for its side effects only; it doesn’t return a value we need.

When you run this code, it will compile but will display a warning:

```bash
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
warning: unused "Result" that must be used
  --> src/main.rs:19:5
   |
19 |     run(config);
   |     ^^^^^^^^^^^
   |
   = note: this "Result" may be an "Err" variant, which should be handled
   = note: "#[warn(unused_must_use)]" on by default

warning: "minigrep" (bin "minigrep") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.71s
     Running "target/debug/minigrep the poem.txt"
Searching for the
In file poem.txt
With text:
I"m nobody! Who are you?
Are you nobody, too?
Then there"s a pair of us - don"t tell!
They"d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Rust tells us that our code ignored the "Result" value and the "Result" value might indicate that an error occurred. But we’re not checking to see whether or not there was an error, and the compiler reminds us that we probably meant to have some error-handling code here! Let’s rectify that problem now.

#### [Handling Errors Returned from "run" in "main"](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#handling-errors-returned-from-run-in-main)

We’ll check for errors and handle them using a technique similar to one we used with "Config::build" in Listing 12-10, but with a slight difference:

Filename: src/main.rs

```rust
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    if let Err(e) = run(config) {
        println!("Application error: {e}");
        process::exit(1);
    }
}
```

We use "if let" rather than "unwrap_or_else" to check whether "run" returns an "Err" value and call "process::exit(1)" if it does. The "run" function doesn’t return a value that we want to "unwrap" in the same way that "Config::build" returns the "Config" instance. Because "run" returns "()" in the success case, we only care about detecting an error, so we don’t need "unwrap_or_else" to return the unwrapped value, which would only be "()".

The bodies of the "if let" and the "unwrap_or_else" functions are the same in both cases: we print the error and exit.

### [Splitting Code into a Library Crate](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#splitting-code-into-a-library-crate)

Our "minigrep" project is looking good so far! Now we’ll split the *src/main.rs* file and put some code into the *src/lib.rs* file. That way we can test the code and have a *src/main.rs* file with fewer responsibilities.

Let’s move all the code that isn’t the "main" function from *src/main.rs* to *src/lib.rs*:

- The "run" function definition
- The relevant "use" statements
- The definition of "Config"
- The "Config::build" function definition

The contents of *src/lib.rs* should have the signatures shown in Listing 12-13 (we’ve omitted the bodies of the functions for brevity). Note that this won’t compile until we modify *src/main.rs* in Listing 12-14.

Filename: src/lib.rs

```rust
use std::error::Error;
use std::fs;

pub struct Config {
    pub query: String,
    pub file_path: String,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &"static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    // --snip--
}
```

Listing 12-13: Moving "Config" and "run" into *src/lib.rs*

We’ve made liberal use of the "pub" keyword: on "Config", on its fields and its "build" method, and on the "run" function. We now have a library crate that has a public API we can test!

Now we need to bring the code we moved to *src/lib.rs* into the scope of the binary crate in *src/main.rs*, as shown in Listing 12-14.

Filename: src/main.rs

```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

Listing 12-14: Using the "minigrep" library crate in *src/main.rs*

We add a "use minigrep::Config" line to bring the "Config" type from the library crate into the binary crate’s scope, and we prefix the "run" function with our crate name. Now all the functionality should be connected and should work. Run the program with "cargo run" and make sure everything works correctly.

Whew! That was a lot of work, but we’ve set ourselves up for success in the future. Now it’s much easier to handle errors, and we’ve made the code more modular. Almost all of our work will be done in *src/lib.rs* from here on out.

Let’s take advantage of this newfound modularity by doing something that would have been difficult with the old code but is easy with the new code: we’ll write some tests!





---





## [Developing the Library’s Functionality with Test-Driven Development](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#developing-the-librarys-functionality-with-test-driven-development)

Now that we’ve extracted the logic into *src/lib.rs* and left the argument collecting and error handling in *src/main.rs*, it’s much easier to write tests for the core functionality of our code. We can call functions directly with various arguments and check return values without having to call our binary from the command line.

In this section, we’ll add the searching logic to the "minigrep" program using the test-driven development (TDD) process with the following steps:

1. Write a test that fails and run it to make sure it fails for the reason you expect.
2. Write or modify just enough code to make the new test pass.
3. Refactor the code you just added or changed and make sure the tests continue to pass.
4. Repeat from step 1!

Though it’s just one of many ways to write software, TDD can help drive code design. Writing the test before you write the code that makes the test pass helps to maintain high test coverage throughout the process.

We’ll test drive the implementation of the functionality that will actually do the searching for the query string in the file contents and produce a list of lines that match the query. We’ll add this functionality in a function called "search".

### [Writing a Failing Test](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#writing-a-failing-test)

Because we don’t need them anymore, let’s remove the "println!" statements from *src/lib.rs* and *src/main.rs* that we used to check the program’s behavior. Then, in *src/lib.rs*, add a "tests" module with a test function, as we did in [Chapter 11](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#the-anatomy-of-a-test-function). The test function specifies the behavior we want the "search" function to have: it will take a query and the text to search, and it will return only the lines from the text that contain the query. Listing 12-15 shows this test, which won’t compile yet.

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
}
```

Listing 12-15: Creating a failing test for the "search" function we wish we had

This test searches for the string ""duct"". The text we’re searching is three lines, only one of which contains ""duct"" (Note that the backslash after the opening double quote tells Rust not to put a newline character at the beginning of the contents of this string literal). We assert that the value returned from the "search" function contains only the line we expect.

We aren’t yet able to run this test and watch it fail because the test doesn’t even compile: the "search" function doesn’t exist yet! In accordance with TDD principles, we’ll add just enough code to get the test to compile and run by adding a definition of the "search" function that always returns an empty vector, as shown in Listing 12-16. Then the test should compile and fail because an empty vector doesn’t match a vector containing the line ""safe, fast, productive.""

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    vec![]
}
```

Listing 12-16: Defining just enough of the "search" function so our test will compile

Notice that we need to define an explicit lifetime ""a" in the signature of "search" and use that lifetime with the "contents" argument and the return value. Recall in [Chapter 10](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html) that the lifetime parameters specify which argument lifetime is connected to the lifetime of the return value. In this case, we indicate that the returned vector should contain string slices that reference slices of the argument "contents" (rather than the argument "query").

In other words, we tell Rust that the data returned by the "search" function will live as long as the data passed into the "search" function in the "contents" argument. This is important! The data referenced *by* a slice needs to be valid for the reference to be valid; if the compiler assumes we’re making string slices of "query" rather than "contents", it will do its safety checking incorrectly.

If we forget the lifetime annotations and try to compile this function, we’ll get this error:

```bash
$ cargo build
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
error[E0106]: missing lifetime specifier
  --> src/lib.rs:28:51
   |
28 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
   |                      ----            ----         ^ expected named lifetime parameter
   |
   = help: this function"s return type contains a borrowed value, but the signature does not say whether it is borrowed from "query" or "contents"
help: consider introducing a named lifetime parameter
   |
28 | pub fn search<"a>(query: &"a str, contents: &"a str) -> Vec<&"a str> {
   |              ++++         ++                 ++              ++

For more information about this error, try "rustc --explain E0106".
error: could not compile "minigrep" due to previous error
```

Rust can’t possibly know which of the two arguments we need, so we need to tell it explicitly. Because "contents" is the argument that contains all of our text and we want to return the parts of that text that match, we know "contents" is the argument that should be connected to the return value using the lifetime syntax.

Other programming languages don’t require you to connect arguments to return values in the signature, but this practice will get easier over time. You might want to compare this example with the [“Validating References with Lifetimes”](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) section in Chapter 10.

Now let’s run the test:

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 0.97s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 1 test
test tests::one_result ... FAILED

failures:

---- tests::one_result stdout ----
thread "tests::one_result" panicked at "assertion failed: "(left == right)"
  left: "["safe, fast, productive."]",
 right: "[]"", src/lib.rs:44:9
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Great, the test fails, exactly as we expected. Let’s get the test to pass!

### [Writing Code to Pass the Test](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#writing-code-to-pass-the-test)

Currently, our test is failing because we always return an empty vector. To fix that and implement "search", our program needs to follow these steps:

- Iterate through each line of the contents.
- Check whether the line contains our query string.
- If it does, add it to the list of values we’re returning.
- If it doesn’t, do nothing.
- Return the list of results that match.

Let’s work through each step, starting with iterating through lines.

#### [Iterating Through Lines with the "lines" Method](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#iterating-through-lines-with-the-lines-method)

Rust has a helpful method to handle line-by-line iteration of strings, conveniently named "lines", that works as shown in Listing 12-17. Note this won’t compile yet.

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

Listing 12-17: Iterating through each line in "contents"

The "lines" method returns an iterator. We’ll talk about iterators in depth in [Chapter 13](https://doc.rust-lang.org/book/ch13-02-iterators.html), but recall that you saw this way of using an iterator in [Listing 3-5](https://doc.rust-lang.org/book/ch03-05-control-flow.html#looping-through-a-collection-with-for), where we used a "for" loop with an iterator to run some code on each item in a collection.

#### [Searching Each Line for the Query](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#searching-each-line-for-the-query)

Next, we’ll check whether the current line contains our query string. Fortunately, strings have a helpful method named "contains" that does this for us! Add a call to the "contains" method in the "search" function, as shown in Listing 12-18. Note this still won’t compile yet.

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

Listing 12-18: Adding functionality to see whether the line contains the string in "query"

At the moment, we’re building up functionality. To get it to compile, we need to return a value from the body as we indicated we would in the function signature.

#### [Storing Matching Lines](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#storing-matching-lines)

To finish this function, we need a way to store the matching lines that we want to return. For that, we can make a mutable vector before the "for" loop and call the "push" method to store a "line" in the vector. After the "for" loop, we return the vector, as shown in Listing 12-19.

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

Listing 12-19: Storing the lines that match so we can return them

Now the "search" function should return only the lines that contain "query", and our test should pass. Let’s run the test:

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 1.22s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 1 test
test tests::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Our test passed, so we know it works!

At this point, we could consider opportunities for refactoring the implementation of the search function while keeping the tests passing to maintain the same functionality. The code in the search function isn’t too bad, but it doesn’t take advantage of some useful features of iterators. We’ll return to this example in [Chapter 13](https://doc.rust-lang.org/book/ch13-02-iterators.html), where we’ll explore iterators in detail, and look at how to improve it.

#### [Using the "search" Function in the "run" Function](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#using-the-search-function-in-the-run-function)

Now that the "search" function is working and tested, we need to call "search" from our "run" function. We need to pass the "config.query" value and the "contents" that "run" reads from the file to the "search" function. Then "run" will print each line returned from "search":

Filename: src/lib.rs

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    for line in search(&config.query, &contents) {
        println!("{line}");
    }

    Ok(())
}
```

We’re still using a "for" loop to return each line from "search" and print it.

Now the entire program should work! Let’s try it out, first with a word that should return exactly one line from the Emily Dickinson poem, “frog”:

```bash
$ cargo run -- frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38s
     Running "target/debug/minigrep frog poem.txt"
How public, like a frog
```

Cool! Now let’s try a word that will match multiple lines, like “body”:

```bash
$ cargo run -- body poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep body poem.txt"
I"m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

And finally, let’s make sure that we don’t get any lines when we search for a word that isn’t anywhere in the poem, such as “monomorphization”:

```bash
$ cargo run -- monomorphization poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep monomorphization poem.txt"
```

Excellent! We’ve built our own mini version of a classic tool and learned a lot about how to structure applications. We’ve also learned a bit about file input and output, lifetimes, testing, and command line parsing.

To round out this project, we’ll briefly demonstrate how to work with environment variables and how to print to standard error, both of which are useful when you’re writing command line programs.





---





## [Working with Environment Variables](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#working-with-environment-variables)

We’ll improve "minigrep" by adding an extra feature: an option for case-insensitive searching that the user can turn on via an environment variable. We could make this feature a command line option and require that users enter it each time they want it to apply, but by instead making it an environment variable, we allow our users to set the environment variable once and have all their searches be case insensitive in that terminal session.

### [Writing a Failing Test for the Case-Insensitive "search" Function](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#writing-a-failing-test-for-the-case-insensitive-search-function)

We first add a new "search_case_insensitive" function that will be called when the environment variable has a value. We’ll continue to follow the TDD process, so the first step is again to write a failing test. We’ll add a new test for the new "search_case_insensitive" function and rename our old test from "one_result" to "case_sensitive" to clarify the differences between the two tests, as shown in Listing 12-20.

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

Listing 12-20: Adding a new failing test for the case-insensitive function we’re about to add

Note that we’ve edited the old test’s "contents" too. We’ve added a new line with the text ""Duct tape."" using a capital D that shouldn’t match the query ""duct"" when we’re searching in a case-sensitive manner. Changing the old test in this way helps ensure that we don’t accidentally break the case-sensitive search functionality that we’ve already implemented. This test should pass now and should continue to pass as we work on the case-insensitive search.

The new test for the case-*insensitive* search uses ""rUsT"" as its query. In the "search_case_insensitive" function we’re about to add, the query ""rUsT"" should match the line containing ""Rust:"" with a capital R and match the line ""Trust me."" even though both have different casing from the query. This is our failing test, and it will fail to compile because we haven’t yet defined the "search_case_insensitive" function. Feel free to add a skeleton implementation that always returns an empty vector, similar to the way we did for the "search" function in Listing 12-16 to see the test compile and fail.

### [Implementing the "search_case_insensitive" Function](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#implementing-the-search_case_insensitive-function)

The "search_case_insensitive" function, shown in Listing 12-21, will be almost the same as the "search" function. The only difference is that we’ll lowercase the "query" and each "line" so whatever the case of the input arguments, they’ll be the same case when we check whether the line contains the query.

Filename: src/lib.rs

```rust
pub fn search_case_insensitive<"a>(
    query: &str,
    contents: &"a str,
) -> Vec<&"a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

Listing 12-21: Defining the "search_case_insensitive" function to lowercase the query and the line before comparing them

First, we lowercase the "query" string and store it in a shadowed variable with the same name. Calling "to_lowercase" on the query is necessary so no matter whether the user’s query is ""rust"", ""RUST"", ""Rust"", or ""rUsT"", we’ll treat the query as if it were ""rust"" and be insensitive to the case. While "to_lowercase" will handle basic Unicode, it won’t be 100% accurate. If we were writing a real application, we’d want to do a bit more work here, but this section is about environment variables, not Unicode, so we’ll leave it at that here.

Note that "query" is now a "String" rather than a string slice, because calling "to_lowercase" creates new data rather than referencing existing data. Say the query is ""rUsT"", as an example: that string slice doesn’t contain a lowercase "u" or "t" for us to use, so we have to allocate a new "String" containing ""rust"". When we pass "query" as an argument to the "contains" method now, we need to add an ampersand because the signature of "contains" is defined to take a string slice.

Next, we add a call to "to_lowercase" on each "line" to lowercase all characters. Now that we’ve converted "line" and "query" to lowercase, we’ll find matches no matter what the case of the query is.

Let’s see if this implementation passes the tests:

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 1.33s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 2 tests
test tests::case_insensitive ... ok
test tests::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Great! They passed. Now, let’s call the new "search_case_insensitive" function from the "run" function. First, we’ll add a configuration option to the "Config" struct to switch between case-sensitive and case-insensitive search. Adding this field will cause compiler errors because we aren’t initializing this field anywhere yet:

Filename: src/lib.rs

```rust
pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}
```

We added the "ignore_case" field that holds a Boolean. Next, we need the "run" function to check the "ignore_case" field’s value and use that to decide whether to call the "search" function or the "search_case_insensitive" function, as shown in Listing 12-22. This still won’t compile yet.

Filename: src/lib.rs

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}
```

Listing 12-22: Calling either "search" or "search_case_insensitive" based on the value in "config.ignore_case"

Finally, we need to check for the environment variable. The functions for working with environment variables are in the "env" module in the standard library, so we bring that module into scope at the top of *src/lib.rs*. Then we’ll use the "var" function from the "env" module to check to see if any value has been set for an environment variable named "IGNORE_CASE", as shown in Listing 12-23.

Filename: src/lib.rs

```rust
use std::env;
// --snip--

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &"static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

Listing 12-23: Checking for any value in an environment variable named "IGNORE_CASE"

Here, we create a new variable "ignore_case". To set its value, we call the "env::var" function and pass it the name of the "IGNORE_CASE" environment variable. The "env::var" function returns a "Result" that will be the successful "Ok" variant that contains the value of the environment variable if the environment variable is set to any value. It will return the "Err" variant if the environment variable is not set.

We’re using the "is_ok" method on the "Result" to check whether the environment variable is set, which means the program should do a case-insensitive search. If the "IGNORE_CASE" environment variable isn’t set to anything, "is_ok" will return false and the program will perform a case-sensitive search. We don’t care about the *value* of the environment variable, just whether it’s set or unset, so we’re checking "is_ok" rather than using "unwrap", "expect", or any of the other methods we’ve seen on "Result".

We pass the value in the "ignore_case" variable to the "Config" instance so the "run" function can read that value and decide whether to call "search_case_insensitive" or "search", as we implemented in Listing 12-22.

Let’s give it a try! First, we’ll run our program without the environment variable set and with the query "to", which should match any line that contains the word “to” in all lowercase:

```bash
$ cargo run -- to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/minigrep to poem.txt"
Are you nobody, too?
How dreary to be somebody!
```

Looks like that still works! Now, let’s run the program with "IGNORE_CASE" set to "1" but with the same query "to".

```bash
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

If you’re using PowerShell, you will need to set the environment variable and run the program as separate commands:

```bash
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

This will make "IGNORE_CASE" persist for the remainder of your shell session. It can be unset with the "Remove-Item" cmdlet:

```bash
PS> Remove-Item Env:IGNORE_CASE
```

We should get lines that contain “to” that might have uppercase letters:

```bash
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excellent, we also got lines containing “To”! Our "minigrep" program can now do case-insensitive searching controlled by an environment variable. Now you know how to manage options set using either command line arguments or environment variables.

Some programs allow arguments *and* environment variables for the same configuration. In those cases, the programs decide that one or the other takes precedence. For another exercise on your own, try controlling case sensitivity through either a command line argument or an environment variable. Decide whether the command line argument or the environment variable should take precedence if the program is run with one set to case sensitive and one set to ignore case.

The "std::env" module contains many more useful features for dealing with environment variables: check out its documentation to see what is available.





---





## [Writing Error Messages to Standard Error Instead of Standard Output](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#writing-error-messages-to-standard-error-instead-of-standard-output)

At the moment, we’re writing all of our output to the terminal using the "println!" macro. In most terminals, there are two kinds of output: *standard output* ("stdout") for general information and *standard error* ("stderr") for error messages. This distinction enables users to choose to direct the successful output of a program to a file but still print error messages to the screen.

The "println!" macro is only capable of printing to standard output, so we have to use something else to print to standard error.

### [Checking Where Errors Are Written](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#checking-where-errors-are-written)

First, let’s observe how the content printed by "minigrep" is currently being written to standard output, including any error messages we want to write to standard error instead. We’ll do that by redirecting the standard output stream to a file while intentionally causing an error. We won’t redirect the standard error stream, so any content sent to standard error will continue to display on the screen.

Command line programs are expected to send error messages to the standard error stream so we can still see error messages on the screen even if we redirect the standard output stream to a file. Our program is not currently well-behaved: we’re about to see that it saves the error message output to a file instead!

To demonstrate this behavior, we’ll run the program with ">" and the file path, *output.txt*, that we want to redirect the standard output stream to. We won’t pass any arguments, which should cause an error:

```bash
$ cargo run > output.txt
```

The ">" syntax tells the shell to write the contents of standard output to *output.txt* instead of the screen. We didn’t see the error message we were expecting printed to the screen, so that means it must have ended up in the file. This is what *output.txt* contains:

```
Problem parsing arguments: not enough arguments
```

Yup, our error message is being printed to standard output. It’s much more useful for error messages like this to be printed to standard error so only data from a successful run ends up in the file. We’ll change that.

### [Printing Errors to Standard Error](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#printing-errors-to-standard-error)

We’ll use the code in Listing 12-24 to change how error messages are printed. Because of the refactoring we did earlier in this chapter, all the code that prints error messages is in one function, "main". The standard library provides the "eprintln!" macro that prints to the standard error stream, so let’s change the two places we were calling "println!" to print errors to use "eprintln!" instead.

Filename: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

Listing 12-24: Writing error messages to standard error instead of standard output using "eprintln!"

Let’s now run the program again in the same way, without any arguments and redirecting standard output with ">":

```bash
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Now we see the error onscreen and *output.txt* contains nothing, which is the behavior we expect of command line programs.

Let’s run the program again with arguments that don’t cause an error but still redirect standard output to a file, like so:

```bash
$ cargo run -- to poem.txt > output.txt
```

We won’t see any output to the terminal, and *output.txt* will contain our results:

Filename: output.txt

```
Are you nobody, too?
How dreary to be somebody!
```

This demonstrates that we’re now using standard output for successful output and standard error for error output as appropriate.

## [Summary](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#summary)

This chapter recapped some of the major concepts you’ve learned so far and covered how to perform common I/O operations in Rust. By using command line arguments, files, environment variables, and the "eprintln!" macro for printing errors, you’re now prepared to write command line applications. Combined with the concepts in previous chapters, your code will be well organized, store data effectively in the appropriate data structures, handle errors nicely, and be well tested.

Next, we’ll explore some Rust features that were influenced by functional languages: closures and iterators.





---





# 13



# [Functional Language Features: Iterators and Closures](https://doc.rust-lang.org/book/ch13-00-functional-features.html#functional-language-features-iterators-and-closures)

Rust’s design has taken inspiration from many existing languages and techniques, and one significant influence is *functional programming*. Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, assigning them to variables for later execution, and so forth.

In this chapter, we won’t debate the issue of what functional programming is or isn’t but will instead discuss some features of Rust that are similar to features in many languages often referred to as functional.

More specifically, we’ll cover:

- *Closures*, a function-like construct you can store in a variable
- *Iterators*, a way of processing a series of elements
- How to use closures and iterators to improve the I/O project in Chapter 12
- The performance of closures and iterators (Spoiler alert: they’re faster than you might think!)

We’ve already covered some other Rust features, such as pattern matching and enums, that are also influenced by the functional style. Because mastering closures and iterators is an important part of writing idiomatic, fast Rust code, we’ll devote this entire chapter to them.





---





## [Closures: Anonymous Functions that Capture Their Environment](https://doc.rust-lang.org/book/ch13-01-closures.html#closures-anonymous-functions-that-capture-their-environment)

Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. You can create the closure in one place and then call the closure elsewhere to evaluate it in a different context. Unlike functions, closures can capture values from the scope in which they’re defined. We’ll demonstrate how these closure features allow for code reuse and behavior customization.



### [Capturing the Environment with Closures](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-the-environment-with-closures)

We’ll first examine how we can use closures to capture values from the environment they’re defined in for later use. Here’s the scenario: Every so often, our t-shirt company gives away an exclusive, limited-edition shirt to someone on our mailing list as a promotion. People on the mailing list can optionally add their favorite color to their profile. If the person chosen for a free shirt has their favorite color set, they get that color shirt. If the person hasn’t specified a favorite color, they get whatever color the company currently has the most of.

There are many ways to implement this. For this example, we’re going to use an enum called "ShirtColor" that has the variants "Red" and "Blue" (limiting the number of colors available for simplicity). We represent the company’s inventory with an "Inventory" struct that has a field named "shirts" that contains a "Vec<ShirtColor>" representing the shirt colors currently in stock. The method "giveaway" defined on "Inventory" gets the optional shirt color preference of the free shirt winner, and returns the shirt color the person will get. This setup is shown in Listing 13-1:

Filename: src/main.rs

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

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```

Listing 13-1: Shirt company giveaway situation

The "store" defined in "main" has two blue shirts and one red shirt remaining to distribute for this limited-edition promotion. We call the "giveaway" method for a user with a preference for a red shirt and a user without any preference.

Again, this code could be implemented in many ways, and here, to focus on closures, we’ve stuck to concepts you’ve already learned except for the body of the "giveaway" method that uses a closure. In the "giveaway" method, we get the user preference as a parameter of type "Option<ShirtColor>" and call the "unwrap_or_else" method on "user_preference". The ["unwrap_or_else" method on "Option"](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else) is defined by the standard library. It takes one argument: a closure without any arguments that returns a value "T" (the same type stored in the "Some" variant of the "Option<T>", in this case "ShirtColor"). If the "Option<T>" is the "Some" variant, "unwrap_or_else" returns the value from within the "Some". If the "Option<T>" is the "None" variant, "unwrap_or_else" calls the closure and returns the value returned by the closure.

We specify the closure expression "|| self.most_stocked()" as the argument to "unwrap_or_else". This is a closure that takes no parameters itself (if the closure had parameters, they would appear between the two vertical bars). The body of the closure calls "self.most_stocked()". We’re defining the closure here, and the implementation of "unwrap_or_else" will evaluate the closure later if the result is needed.

Running this code prints:

```bash
$ cargo run
   Compiling shirt-company v0.1.0 (file:///projects/shirt-company)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running "target/debug/shirt-company"
The user with preference Some(Red) gets Red
The user with preference None gets Blue
```

One interesting aspect here is that we’ve passed a closure that calls "self.most_stocked()" on the current "Inventory" instance. The standard library didn’t need to know anything about the "Inventory" or "ShirtColor" types we defined, or the logic we want to use in this scenario. The closure captures an immutable reference to the "self" "Inventory" instance and passes it with the code we specify to the "unwrap_or_else" method. Functions, on the other hand, are not able to capture their environment in this way.

### [Closure Type Inference and Annotation](https://doc.rust-lang.org/book/ch13-01-closures.html#closure-type-inference-and-annotation)

There are more differences between functions and closures. Closures don’t usually require you to annotate the types of the parameters or the return value like "fn" functions do. Type annotations are required on functions because the types are part of an explicit interface exposed to your users. Defining this interface rigidly is important for ensuring that everyone agrees on what types of values a function uses and returns. Closures, on the other hand, aren’t used in an exposed interface like this: they’re stored in variables and used without naming them and exposing them to users of our library.

Closures are typically short and relevant only within a narrow context rather than in any arbitrary scenario. Within these limited contexts, the compiler can infer the types of the parameters and the return type, similar to how it’s able to infer the types of most variables (there are rare cases where the compiler needs closure type annotations too).

As with variables, we can add type annotations if we want to increase explicitness and clarity at the cost of being more verbose than is strictly necessary. Annotating the types for a closure would look like the definition shown in Listing 13-2. In this example, we’re defining a closure and storing it in a variable rather than defining the closure in the spot we pass it as an argument as we did in Listing 13-1.

Filename: src/main.rs

```rust
    let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };
```

Listing 13-2: Adding optional type annotations of the parameter and return value types in the closure

With type annotations added, the syntax of closures looks more similar to the syntax of functions. Here we define a function that adds 1 to its parameter and a closure that has the same behavior, for comparison. We’ve added some spaces to line up the relevant parts. This illustrates how closure syntax is similar to function syntax except for the use of pipes and the amount of syntax that is optional:

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

The first line shows a function definition, and the second line shows a fully annotated closure definition. In the third line, we remove the type annotations from the closure definition. In the fourth line, we remove the brackets, which are optional because the closure body has only one expression. These are all valid definitions that will produce the same behavior when they’re called. The "add_one_v3" and "add_one_v4" lines require the closures to be evaluated to be able to compile because the types will be inferred from their usage. This is similar to "let v = Vec::new();" needing either type annotations or values of some type to be inserted into the "Vec" for Rust to be able to infer the type.

For closure definitions, the compiler will infer one concrete type for each of their parameters and for their return value. For instance, Listing 13-3 shows the definition of a short closure that just returns the value it receives as a parameter. This closure isn’t very useful except for the purposes of this example. Note that we haven’t added any type annotations to the definition. Because there are no type annotations, we can call the closure with any type, which we’ve done here with "String" the first time. If we then try to call "example_closure" with an integer, we’ll get an error.

Filename: src/main.rs

```rust
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
```

Listing 13-3: Attempting to call a closure whose types are inferred with two different types

The compiler gives us this error:

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |             --------------- ^- help: try using a conversion method: ".to_string()"
  |             |               |
  |             |               expected struct "String", found integer
  |             arguments to this function are incorrect
  |
note: closure parameter defined here
 --> src/main.rs:2:28
  |
2 |     let example_closure = |x| x;
  |                            ^

For more information about this error, try "rustc --explain E0308".
error: could not compile "closure-example" due to previous error
```

The first time we call "example_closure" with the "String" value, the compiler infers the type of "x" and the return type of the closure to be "String". Those types are then locked into the closure in "example_closure", and we get a type error when we next try to use a different type with the same closure.

### [Capturing References or Moving Ownership](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership)

Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: borrowing immutably, borrowing mutably, and taking ownership. The closure will decide which of these to use based on what the body of the function does with the captured values.

In Listing 13-4, we define a closure that captures an immutable reference to the vector named "list" because it only needs an immutable reference to print the value:

Filename: src/main.rs

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let only_borrows = || println!("From closure: {:?}", list);

    println!("Before calling closure: {:?}", list);
    only_borrows();
    println!("After calling closure: {:?}", list);
}
```

Listing 13-4: Defining and calling a closure that captures an immutable reference

This example also illustrates that a variable can bind to a closure definition, and we can later call the closure by using the variable name and parentheses as if the variable name were a function name.

Because we can have multiple immutable references to "list" at the same time, "list" is still accessible from the code before the closure definition, after the closure definition but before the closure is called, and after the closure is called. This code compiles, runs, and prints:

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running "target/debug/closure-example"
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
```

Next, in Listing 13-5, we change the closure body so that it adds an element to the "list" vector. The closure now captures a mutable reference:

Filename: src/main.rs

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {:?}", list);
}
```

Listing 13-5: Defining and calling a closure that captures a mutable reference

This code compiles, runs, and prints:

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running "target/debug/closure-example"
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
```

Note that there’s no longer a "println!" between the definition and the call of the "borrows_mutably" closure: when "borrows_mutably" is defined, it captures a mutable reference to "list". We don’t use the closure again after the closure is called, so the mutable borrow ends. Between the closure definition and the closure call, an immutable borrow to print isn’t allowed because no other borrows are allowed when there’s a mutable borrow. Try adding a "println!" there to see what error message you get!

If you want to force the closure to take ownership of the values it uses in the environment even though the body of the closure doesn’t strictly need ownership, you can use the "move" keyword before the parameter list.

This technique is mostly useful when passing a closure to a new thread to move the data so that it’s owned by the new thread. We’ll discuss threads and why you would want to use them in detail in Chapter 16 when we talk about concurrency, but for now, let’s briefly explore spawning a new thread using a closure that needs the "move" keyword. Listing 13-6 shows Listing 13-4 modified to print the vector in a new thread rather than in the main thread:

Filename: src/main.rs

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```

Listing 13-6: Using "move" to force the closure for the thread to take ownership of "list"

We spawn a new thread, giving the thread a closure to run as an argument. The closure body prints out the list. In Listing 13-4, the closure only captured "list" using an immutable reference because that"s the least amount of access to "list" needed to print it. In this example, even though the closure body still only needs an immutable reference, we need to specify that "list" should be moved into the closure by putting the "move" keyword at the beginning of the closure definition. The new thread might finish before the rest of the main thread finishes, or the main thread might finish first. If the main thread maintained ownership of "list" but ended before the new thread did and dropped "list", the immutable reference in the thread would be invalid. Therefore, the compiler requires that "list" be moved into the closure given to the new thread so the reference will be valid. Try removing the "move" keyword or using "list" in the main thread after the closure is defined to see what compiler errors you get!



### [Moving Captured Values Out of Closures and the "Fn" Traits](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits)

Once a closure has captured a reference or captured ownership of a value from the environment where the closure is defined (thus affecting what, if anything, is moved *into* the closure), the code in the body of the closure defines what happens to the references or values when the closure is evaluated later (thus affecting what, if anything, is moved *out of* the closure). A closure body can do any of the following: move a captured value out of the closure, mutate the captured value, neither move nor mutate the value, or capture nothing from the environment to begin with.

The way a closure captures and handles values from the environment affects which traits the closure implements, and traits are how functions and structs can specify what kinds of closures they can use. Closures will automatically implement one, two, or all three of these "Fn" traits, in an additive fashion, depending on how the closure’s body handles the values:

1. "FnOnce" applies to closures that can be called once. All closures implement at least this trait, because all closures can be called. A closure that moves captured values out of its body will only implement "FnOnce" and none of the other "Fn" traits, because it can only be called once.
2. "FnMut" applies to closures that don’t move captured values out of their body, but that might mutate the captured values. These closures can be called more than once.
3. "Fn" applies to closures that don’t move captured values out of their body and that don’t mutate captured values, as well as closures that capture nothing from their environment. These closures can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.

Let’s look at the definition of the "unwrap_or_else" method on "Option<T>" that we used in Listing 13-1:

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Recall that "T" is the generic type representing the type of the value in the "Some" variant of an "Option". That type "T" is also the return type of the "unwrap_or_else" function: code that calls "unwrap_or_else" on an "Option<String>", for example, will get a "String".

Next, notice that the "unwrap_or_else" function has the additional generic type parameter "F". The "F" type is the type of the parameter named "f", which is the closure we provide when calling "unwrap_or_else".

The trait bound specified on the generic type "F" is "FnOnce() -> T", which means "F" must be able to be called once, take no arguments, and return a "T". Using "FnOnce" in the trait bound expresses the constraint that "unwrap_or_else" is only going to call "f" at most one time. In the body of "unwrap_or_else", we can see that if the "Option" is "Some", "f" won’t be called. If the "Option" is "None", "f" will be called once. Because all closures implement "FnOnce", "unwrap_or_else" accepts the most different kinds of closures and is as flexible as it can be.

> Note: Functions can implement all three of the "Fn" traits too. If what we want to do doesn’t require capturing a value from the environment, we can use the name of a function rather than a closure where we need something that implements one of the "Fn" traits. For example, on an "Option<Vec<T>>" value, we could call "unwrap_or_else(Vec::new)" to get a new, empty vector if the value is "None".

Now let’s look at the standard library method "sort_by_key" defined on slices, to see how that differs from "unwrap_or_else" and why "sort_by_key" uses "FnMut" instead of "FnOnce" for the trait bound. The closure gets one argument in the form of a reference to the current item in the slice being considered, and returns a value of type "K" that can be ordered. This function is useful when you want to sort a slice by a particular attribute of each item. In Listing 13-7, we have a list of "Rectangle" instances and we use "sort_by_key" to order them by their "width" attribute from low to high:

Filename: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
```

Listing 13-7: Using "sort_by_key" to order rectangles by width

This code prints:

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running "target/debug/rectangles"
[
    Rectangle {
        width: 3,
        height: 5,
    },
    Rectangle {
        width: 7,
        height: 12,
    },
    Rectangle {
        width: 10,
        height: 1,
    },
]
```

The reason "sort_by_key" is defined to take an "FnMut" closure is that it calls the closure multiple times: once for each item in the slice. The closure "|r| r.width" doesn’t capture, mutate, or move out anything from its environment, so it meets the trait bound requirements.

In contrast, Listing 13-8 shows an example of a closure that implements just the "FnOnce" trait, because it moves a value out of the environment. The compiler won’t let us use this closure with "sort_by_key":

Filename: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("by key called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{:#?}", list);
}
```

Listing 13-8: Attempting to use an "FnOnce" closure with "sort_by_key"

This is a contrived, convoluted way (that doesn’t work) to try and count the number of times "sort_by_key" gets called when sorting "list". This code attempts to do this counting by pushing "value"—a "String" from the closure’s environment—into the "sort_operations" vector. The closure captures "value" then moves "value" out of the closure by transferring ownership of "value" to the "sort_operations" vector. This closure can be called once; trying to call it a second time wouldn’t work because "value" would no longer be in the environment to be pushed into "sort_operations" again! Therefore, this closure only implements "FnOnce". When we try to compile this code, we get this error that "value" can’t be moved out of the closure because the closure must implement "FnMut":

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
error[E0507]: cannot move out of "value", a captured variable in an "FnMut" closure
  --> src/main.rs:18:30
   |
15 |     let value = String::from("by key called");
   |         ----- captured outer variable
16 |
17 |     list.sort_by_key(|r| {
   |                      --- captured by this "FnMut" closure
18 |         sort_operations.push(value);
   |                              ^^^^^ move occurs because "value" has type "String", which does not implement the "Copy" trait

For more information about this error, try "rustc --explain E0507".
error: could not compile "rectangles" due to previous error
```

The error points to the line in the closure body that moves "value" out of the environment. To fix this, we need to change the closure body so that it doesn’t move values out of the environment. To count the number of times "sort_by_key" is called, keeping a counter in the environment and incrementing its value in the closure body is a more straightforward way to calculate that. The closure in Listing 13-9 works with "sort_by_key" because it is only capturing a mutable reference to the "num_sort_operations" counter and can therefore be called more than once:

Filename: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}
```

Listing 13-9: Using an "FnMut" closure with "sort_by_key" is allowed

The "Fn" traits are important when defining or using functions or types that make use of closures. In the next section, we’ll discuss iterators. Many iterator methods take closure arguments, so keep these closure details in mind as we continue!





---





## [Processing a Series of Items with Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html#processing-a-series-of-items-with-iterators)

The iterator pattern allows you to perform some task on a sequence of items in turn. An iterator is responsible for the logic of iterating over each item and determining when the sequence has finished. When you use iterators, you don’t have to reimplement that logic yourself.

In Rust, iterators are *lazy*, meaning they have no effect until you call methods that consume the iterator to use it up. For example, the code in Listing 13-10 creates an iterator over the items in the vector "v1" by calling the "iter" method defined on "Vec<T>". This code by itself doesn’t do anything useful.

```rust
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();
```

Listing 13-10: Creating an iterator

The iterator is stored in the "v1_iter" variable. Once we’ve created an iterator, we can use it in a variety of ways. In Listing 3-5 in Chapter 3, we iterated over an array using a "for" loop to execute some code on each of its items. Under the hood this implicitly created and then consumed an iterator, but we glossed over how exactly that works until now.

In the example in Listing 13-11, we separate the creation of the iterator from the use of the iterator in the "for" loop. When the "for" loop is called using the iterator in "v1_iter", each element in the iterator is used in one iteration of the loop, which prints out each value.

```rust
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {}", val);
    }
```

Listing 13-11: Using an iterator in a "for" loop

In languages that don’t have iterators provided by their standard libraries, you would likely write this same functionality by starting a variable at index 0, using that variable to index into the vector to get a value, and incrementing the variable value in a loop until it reached the total number of items in the vector.

Iterators handle all that logic for you, cutting down on repetitive code you could potentially mess up. Iterators give you more flexibility to use the same logic with many different kinds of sequences, not just data structures you can index into, like vectors. Let’s examine how iterators do that.

### [The "Iterator" Trait and the "next" Method](https://doc.rust-lang.org/book/ch13-02-iterators.html#the-iterator-trait-and-the-next-method)

All iterators implement a trait named "Iterator" that is defined in the standard library. The definition of the trait looks like this:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Notice this definition uses some new syntax: "type Item" and "Self::Item", which are defining an *associated type* with this trait. We’ll talk about associated types in depth in Chapter 19. For now, all you need to know is that this code says implementing the "Iterator" trait requires that you also define an "Item" type, and this "Item" type is used in the return type of the "next" method. In other words, the "Item" type will be the type returned from the iterator.

The "Iterator" trait only requires implementors to define one method: the "next" method, which returns one item of the iterator at a time wrapped in "Some" and, when iteration is over, returns "None".

We can call the "next" method on iterators directly; Listing 13-12 demonstrates what values are returned from repeated calls to "next" on the iterator created from the vector.

Filename: src/lib.rs

```rust
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
```

Listing 13-12: Calling the "next" method on an iterator

Note that we needed to make "v1_iter" mutable: calling the "next" method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. In other words, this code *consumes*, or uses up, the iterator. Each call to "next" eats up an item from the iterator. We didn’t need to make "v1_iter" mutable when we used a "for" loop because the loop took ownership of "v1_iter" and made it mutable behind the scenes.

Also note that the values we get from the calls to "next" are immutable references to the values in the vector. The "iter" method produces an iterator over immutable references. If we want to create an iterator that takes ownership of "v1" and returns owned values, we can call "into_iter" instead of "iter". Similarly, if we want to iterate over mutable references, we can call "iter_mut" instead of "iter".

### [Methods that Consume the Iterator](https://doc.rust-lang.org/book/ch13-02-iterators.html#methods-that-consume-the-iterator)

The "Iterator" trait has a number of different methods with default implementations provided by the standard library; you can find out about these methods by looking in the standard library API documentation for the "Iterator" trait. Some of these methods call the "next" method in their definition, which is why you’re required to implement the "next" method when implementing the "Iterator" trait.

Methods that call "next" are called *consuming adaptors*, because calling them uses up the iterator. One example is the "sum" method, which takes ownership of the iterator and iterates through the items by repeatedly calling "next", thus consuming the iterator. As it iterates through, it adds each item to a running total and returns the total when iteration is complete. Listing 13-13 has a test illustrating a use of the "sum" method:

Filename: src/lib.rs

```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
```

Listing 13-13: Calling the "sum" method to get the total of all items in the iterator

We aren’t allowed to use "v1_iter" after the call to "sum" because "sum" takes ownership of the iterator we call it on.

### [Methods that Produce Other Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html#methods-that-produce-other-iterators)

*Iterator adaptors* are methods defined on the "Iterator" trait that don’t consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator.

Listing 13-14 shows an example of calling the iterator adaptor method "map", which takes a closure to call on each item as the items are iterated through. The "map" method returns a new iterator that produces the modified items. The closure here creates a new iterator in which each item from the vector will be incremented by 1:

Filename: src/main.rs

```rust
    let v1: Vec<i32> = vec![1, 2, 3];

    v1.iter().map(|x| x + 1);
```

Listing 13-14: Calling the iterator adaptor "map" to create a new iterator

However, this code produces a warning:

```bash
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused "Map" that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: iterators are lazy and do nothing unless consumed
  = note: "#[warn(unused_must_use)]" on by default

warning: "iterators" (bin "iterators") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.47s
     Running "target/debug/iterators"
```

The code in Listing 13-14 doesn’t do anything; the closure we’ve specified never gets called. The warning reminds us why: iterator adaptors are lazy, and we need to consume the iterator here.

To fix this warning and consume the iterator, we’ll use the "collect" method, which we used in Chapter 12 with "env::args" in Listing 12-1. This method consumes the iterator and collects the resulting values into a collection data type.

In Listing 13-15, we collect the results of iterating over the iterator that’s returned from the call to "map" into a vector. This vector will end up containing each item from the original vector incremented by 1.

Filename: src/main.rs

```rust
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
```

Listing 13-15: Calling the "map" method to create a new iterator and then calling the "collect" method to consume the new iterator and create a vector

Because "map" takes a closure, we can specify any operation we want to perform on each item. This is a great example of how closures let you customize some behavior while reusing the iteration behavior that the "Iterator" trait provides.

You can chain multiple calls to iterator adaptors to perform complex actions in a readable way. But because all iterators are lazy, you have to call one of the consuming adaptor methods to get results from calls to iterator adaptors.

### [Using Closures that Capture Their Environment](https://doc.rust-lang.org/book/ch13-02-iterators.html#using-closures-that-capture-their-environment)

Many iterator adapters take closures as arguments, and commonly the closures we’ll specify as arguments to iterator adapters will be closures that capture their environment.

For this example, we’ll use the "filter" method that takes a closure. The closure gets an item from the iterator and returns a "bool". If the closure returns "true", the value will be included in the iteration produced by "filter". If the closure returns "false", the value won’t be included.

In Listing 13-16, we use "filter" with a closure that captures the "shoe_size" variable from its environment to iterate over a collection of "Shoe" struct instances. It will return only shoes that are the specified size.

Filename: src/lib.rs

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
```

Listing 13-16: Using the "filter" method with a closure that captures "shoe_size"

The "shoes_in_size" function takes ownership of a vector of shoes and a shoe size as parameters. It returns a vector containing only shoes of the specified size.

In the body of "shoes_in_size", we call "into_iter" to create an iterator that takes ownership of the vector. Then we call "filter" to adapt that iterator into a new iterator that only contains elements for which the closure returns "true".

The closure captures the "shoe_size" parameter from the environment and compares the value with each shoe’s size, keeping only shoes of the size specified. Finally, calling "collect" gathers the values returned by the adapted iterator into a vector that’s returned by the function.

The test shows that when we call "shoes_in_size", we get back only shoes that have the same size as the value we specified.







---





## [Improving Our I/O Project](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#improving-our-io-project)

With this new knowledge about iterators, we can improve the I/O project in Chapter 12 by using iterators to make places in the code clearer and more concise. Let’s look at how iterators can improve our implementation of the "Config::build" function and the "search" function.

### [Removing a "clone" Using an Iterator](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#removing-a-clone-using-an-iterator)

In Listing 12-6, we added code that took a slice of "String" values and created an instance of the "Config" struct by indexing into the slice and cloning the values, allowing the "Config" struct to own those values. In Listing 13-17, we’ve reproduced the implementation of the "Config::build" function as it was in Listing 12-23:

Filename: src/lib.rs

```rust
impl Config {
    pub fn build(args: &[String]) -> Result<Config, &"static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

Listing 13-17: Reproduction of the "Config::build" function from Listing 12-23

At the time, we said not to worry about the inefficient "clone" calls because we would remove them in the future. Well, that time is now!

We needed "clone" here because we have a slice with "String" elements in the parameter "args", but the "build" function doesn’t own "args". To return ownership of a "Config" instance, we had to clone the values from the "query" and "file_path" fields of "Config" so the "Config" instance can own its values.

With our new knowledge about iterators, we can change the "build" function to take ownership of an iterator as its argument instead of borrowing a slice. We’ll use the iterator functionality instead of the code that checks the length of the slice and indexes into specific locations. This will clarify what the "Config::build" function is doing because the iterator will access the values.

Once "Config::build" takes ownership of the iterator and stops using indexing operations that borrow, we can move the "String" values from the iterator into "Config" rather than calling "clone" and making a new allocation.

#### [Using the Returned Iterator Directly](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#using-the-returned-iterator-directly)

Open your I/O project’s *src/main.rs* file, which should look like this:

Filename: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

We’ll first change the start of the "main" function that we had in Listing 12-24 to the code in Listing 13-18, which this time uses an iterator. This won’t compile until we update "Config::build" as well.

Filename: src/main.rs

```rust
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

Listing 13-18: Passing the return value of "env::args" to "Config::build"

The "env::args" function returns an iterator! Rather than collecting the iterator values into a vector and then passing a slice to "Config::build", now we’re passing ownership of the iterator returned from "env::args" to "Config::build" directly.

Next, we need to update the definition of "Config::build". In your I/O project’s *src/lib.rs* file, let’s change the signature of "Config::build" to look like Listing 13-19. This still won’t compile because we need to update the function body.

Filename: src/lib.rs

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &"static str> {
        // --snip--
```

Listing 13-19: Updating the signature of "Config::build" to expect an iterator

The standard library documentation for the "env::args" function shows that the type of the iterator it returns is "std::env::Args", and that type implements the "Iterator" trait and returns "String" values.

We’ve updated the signature of the "Config::build" function so the parameter "args" has a generic type with the trait bounds "impl Iterator<Item = String>" instead of "&[String]". This usage of the "impl Trait" syntax we discussed in the [“Traits as Parameters”](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters) section of Chapter 10 means that "args" can be any type that implements the "Iterator" type and returns "String" items.

Because we’re taking ownership of "args" and we’ll be mutating "args" by iterating over it, we can add the "mut" keyword into the specification of the "args" parameter to make it mutable.

#### [Using "Iterator" Trait Methods Instead of Indexing](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#using-iterator-trait-methods-instead-of-indexing)

Next, we’ll fix the body of "Config::build". Because "args" implements the "Iterator" trait, we know we can call the "next" method on it! Listing 13-20 updates the code from Listing 12-23 to use the "next" method:

Filename: src/lib.rs

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &"static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn"t get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn"t get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

Listing 13-20: Changing the body of "Config::build" to use iterator methods

Remember that the first value in the return value of "env::args" is the name of the program. We want to ignore that and get to the next value, so first we call "next" and do nothing with the return value. Second, we call "next" to get the value we want to put in the "query" field of "Config". If "next" returns a "Some", we use a "match" to extract the value. If it returns "None", it means not enough arguments were given and we return early with an "Err" value. We do the same thing for the "file_path" value.

### [Making Code Clearer with Iterator Adaptors](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#making-code-clearer-with-iterator-adaptors)

We can also take advantage of iterators in the "search" function in our I/O project, which is reproduced here in Listing 13-21 as it was in Listing 12-19:

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

Listing 13-21: The implementation of the "search" function from Listing 12-19

We can write this code in a more concise way using iterator adaptor methods. Doing so also lets us avoid having a mutable intermediate "results" vector. The functional programming style prefers to minimize the amount of mutable state to make code clearer. Removing the mutable state might enable a future enhancement to make searching happen in parallel, because we wouldn’t have to manage concurrent access to the "results" vector. Listing 13-22 shows this change:

Filename: src/lib.rs

```rust
pub fn search<"a>(query: &str, contents: &"a str) -> Vec<&"a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

Listing 13-22: Using iterator adaptor methods in the implementation of the "search" function

Recall that the purpose of the "search" function is to return all lines in "contents" that contain the "query". Similar to the "filter" example in Listing 13-16, this code uses the "filter" adaptor to keep only the lines that "line.contains(query)" returns "true" for. We then collect the matching lines into another vector with "collect". Much simpler! Feel free to make the same change to use iterator methods in the "search_case_insensitive" function as well.

### [Choosing Between Loops or Iterators](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#choosing-between-loops-or-iterators)

The next logical question is which style you should choose in your own code and why: the original implementation in Listing 13-21 or the version using iterators in Listing 13-22. Most Rust programmers prefer to use the iterator style. It’s a bit tougher to get the hang of at first, but once you get a feel for the various iterator adaptors and what they do, iterators can be easier to understand. Instead of fiddling with the various bits of looping and building new vectors, the code focuses on the high-level objective of the loop. This abstracts away some of the commonplace code so it’s easier to see the concepts that are unique to this code, such as the filtering condition each element in the iterator must pass.

But are the two implementations truly equivalent? The intuitive assumption might be that the more low-level loop will be faster. Let’s talk about performance.





---





## [Comparing Performance: Loops vs. Iterators](https://doc.rust-lang.org/book/ch13-04-performance.html#comparing-performance-loops-vs-iterators)

To determine whether to use loops or iterators, you need to know which implementation is faster: the version of the "search" function with an explicit "for" loop or the version with iterators.

We ran a benchmark by loading the entire contents of *The Adventures of Sherlock Holmes* by Sir Arthur Conan Doyle into a "String" and looking for the word *the* in the contents. Here are the results of the benchmark on the version of "search" using the "for" loop and the version using iterators:

```
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

The iterator version was slightly faster! We won’t explain the benchmark code here, because the point is not to prove that the two versions are equivalent but to get a general sense of how these two implementations compare performance-wise.

For a more comprehensive benchmark, you should check using various texts of various sizes as the "contents", different words and words of different lengths as the "query", and all kinds of other variations. The point is this: iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s *zero-cost abstractions*, by which we mean using the abstraction imposes no additional runtime overhead. This is analogous to how Bjarne Stroustrup, the original designer and implementor of C++, defines *zero-overhead* in “Foundations of C++” (2012):

> In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

As another example, the following code is taken from an audio decoder. The decoding algorithm uses the linear prediction mathematical operation to estimate future values based on a linear function of the previous samples. This code uses an iterator chain to do some math on three variables in scope: a "buffer" slice of data, an array of 12 "coefficients", and an amount by which to shift data in "qlp_shift". We’ve declared the variables within this example but not given them any values; although this code doesn’t have much meaning outside of its context, it’s still a concise, real-world example of how Rust translates high-level ideas to low-level code.

```rust
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

To calculate the value of "prediction", this code iterates through each of the 12 values in "coefficients" and uses the "zip" method to pair the coefficient values with the previous 12 values in "buffer". Then, for each pair, we multiply the values together, sum all the results, and shift the bits in the sum "qlp_shift" bits to the right.

Calculations in applications like audio decoders often prioritize performance most highly. Here, we’re creating an iterator, using two adaptors, and then consuming the value. What assembly code would this Rust code compile to? Well, as of this writing, it compiles down to the same assembly you’d write by hand. There’s no loop at all corresponding to the iteration over the values in "coefficients": Rust knows that there are 12 iterations, so it “unrolls” the loop. *Unrolling* is an optimization that removes the overhead of the loop controlling code and instead generates repetitive code for each iteration of the loop.

All of the coefficients get stored in registers, which means accessing the values is very fast. There are no bounds checks on the array access at runtime. All these optimizations that Rust is able to apply make the resulting code extremely efficient. Now that you know this, you can use iterators and closures without fear! They make code seem like it’s higher level but don’t impose a runtime performance penalty for doing so.

## [Summary](https://doc.rust-lang.org/book/ch13-04-performance.html#summary)

Closures and iterators are Rust features inspired by functional programming language ideas. They contribute to Rust’s capability to clearly express high-level ideas at low-level performance. The implementations of closures and iterators are such that runtime performance is not affected. This is part of Rust’s goal to strive to provide zero-cost abstractions.

Now that we’ve improved the expressiveness of our I/O project, let’s look at some more features of "cargo" that will help us share the project with the world.





---





# 14



# [More About Cargo and Crates.io](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html#more-about-cargo-and-cratesio)

So far we’ve used only the most basic features of Cargo to build, run, and test our code, but it can do a lot more. In this chapter, we’ll discuss some of its other, more advanced features to show you how to do the following:

- Customize your build through release profiles
- Publish libraries on [crates.io](https://crates.io/)
- Organize large projects with workspaces
- Install binaries from [crates.io](https://crates.io/)
- Extend Cargo using custom commands

Cargo can do even more than the functionality we cover in this chapter, so for a full explanation of all its features, see [its documentation](https://doc.rust-lang.org/cargo/).







---





## [Customizing Builds with Release Profiles](https://doc.rust-lang.org/book/ch14-01-release-profiles.html#customizing-builds-with-release-profiles)

In Rust, *release profiles* are predefined and customizable profiles with different configurations that allow a programmer to have more control over various options for compiling code. Each profile is configured independently of the others.

Cargo has two main profiles: the "dev" profile Cargo uses when you run "cargo build" and the "release" profile Cargo uses when you run "cargo build --release". The "dev" profile is defined with good defaults for development, and the "release" profile has good defaults for release builds.

These profile names might be familiar from the output of your builds:

```bash
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

The "dev" and "release" are these different profiles used by the compiler.

Cargo has default settings for each of the profiles that apply when you haven"t explicitly added any "[profile.*]" sections in the project’s *Cargo.toml* file. By adding "[profile.*]" sections for any profile you want to customize, you override any subset of the default settings. For example, here are the default values for the "opt-level" setting for the "dev" and "release" profiles:

Filename: Cargo.toml

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

The "opt-level" setting controls the number of optimizations Rust will apply to your code, with a range of 0 to 3. Applying more optimizations extends compiling time, so if you’re in development and compiling your code often, you’ll want fewer optimizations to compile faster even if the resulting code runs slower. The default "opt-level" for "dev" is therefore "0". When you’re ready to release your code, it’s best to spend more time compiling. You’ll only compile in release mode once, but you’ll run the compiled program many times, so release mode trades longer compile time for code that runs faster. That is why the default "opt-level" for the "release" profile is "3".

You can override a default setting by adding a different value for it in *Cargo.toml*. For example, if we want to use optimization level 1 in the development profile, we can add these two lines to our project’s *Cargo.toml* file:

Filename: Cargo.toml

```toml
[profile.dev]
opt-level = 1
```

This code overrides the default setting of "0". Now when we run "cargo build", Cargo will use the defaults for the "dev" profile plus our customization to "opt-level". Because we set "opt-level" to "1", Cargo will apply more optimizations than the default, but not as many as in a release build.

For the full list of configuration options and defaults for each profile, see [Cargo’s documentation](https://doc.rust-lang.org/cargo/reference/profiles.html).





---





## [Publishing a Crate to Crates.io](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-a-crate-to-cratesio)

We’ve used packages from [crates.io](https://crates.io/) as dependencies of our project, but you can also share your code with other people by publishing your own packages. The crate registry at [crates.io](https://crates.io/) distributes the source code of your packages, so it primarily hosts code that is open source.

Rust and Cargo have features that make your published package easier for people to find and use. We’ll talk about some of these features next and then explain how to publish a package.

### [Making Useful Documentation Comments](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#making-useful-documentation-comments)

Accurately documenting your packages will help other users know how and when to use them, so it’s worth investing the time to write documentation. In Chapter 3, we discussed how to comment Rust code using two slashes, "//". Rust also has a particular kind of comment for documentation, known conveniently as a *documentation comment*, that will generate HTML documentation. The HTML displays the contents of documentation comments for public API items intended for programmers interested in knowing how to *use* your crate as opposed to how your crate is *implemented*.

Documentation comments use three slashes, "///", instead of two and support Markdown notation for formatting the text. Place documentation comments just before the item they’re documenting. Listing 14-1 shows documentation comments for an "add_one" function in a crate named "my_crate".

Filename: src/lib.rs

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Listing 14-1: A documentation comment for a function

Here, we give a description of what the "add_one" function does, start a section with the heading "Examples", and then provide code that demonstrates how to use the "add_one" function. We can generate the HTML documentation from this documentation comment by running "cargo doc". This command runs the "rustdoc" tool distributed with Rust and puts the generated HTML documentation in the *target/doc* directory.

For convenience, running "cargo doc --open" will build the HTML for your current crate’s documentation (as well as the documentation for all of your crate’s dependencies) and open the result in a web browser. Navigate to the "add_one" function and you’ll see how the text in the documentation comments is rendered, as shown in Figure 14-1:

![Rendered HTML documentation for the "add_one" function of "my_crate"](https://doc.rust-lang.org/book/img/trpl14-01.png)

Figure 14-1: HTML documentation for the "add_one" function

#### [Commonly Used Sections](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commonly-used-sections)

We used the "# Examples" Markdown heading in Listing 14-1 to create a section in the HTML with the title “Examples.” Here are some other sections that crate authors commonly use in their documentation:

- **Panics**: The scenarios in which the function being documented could panic. Callers of the function who don’t want their programs to panic should make sure they don’t call the function in these situations.
- **Errors**: If the function returns a "Result", describing the kinds of errors that might occur and what conditions might cause those errors to be returned can be helpful to callers so they can write code to handle the different kinds of errors in different ways.
- **Safety**: If the function is "unsafe" to call (we discuss unsafety in Chapter 19), there should be a section explaining why the function is unsafe and covering the invariants that the function expects callers to uphold.

Most documentation comments don’t need all of these sections, but this is a good checklist to remind you of the aspects of your code users will be interested in knowing about.

#### [Documentation Comments as Tests](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests)

Adding example code blocks in your documentation comments can help demonstrate how to use your library, and doing so has an additional bonus: running "cargo test" will run the code examples in your documentation as tests! Nothing is better than documentation with examples. But nothing is worse than examples that don’t work because the code has changed since the documentation was written. If we run "cargo test" with the documentation for the "add_one" function from Listing 14-1, we will see a section in the test results like this:

```
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Now if we change either the function or the example so the "assert_eq!" in the example panics and run "cargo test" again, we’ll see that the doc tests catch that the example and the code are out of sync with each other!

#### [Commenting Contained Items](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commenting-contained-items)

The style of doc comment "//!" adds documentation to the item that contains the comments rather than to the items following the comments. We typically use these doc comments inside the crate root file (*src/lib.rs* by convention) or inside a module to document the crate or the module as a whole.

For example, to add documentation that describes the purpose of the "my_crate" crate that contains the "add_one" function, we add documentation comments that start with "//!" to the beginning of the *src/lib.rs* file, as shown in Listing 14-2:

Filename: src/lib.rs

```rust
//! # My Crate
//!
//! "my_crate" is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

Listing 14-2: Documentation for the "my_crate" crate as a whole

Notice there isn’t any code after the last line that begins with "//!". Because we started the comments with "//!" instead of "///", we’re documenting the item that contains this comment rather than an item that follows this comment. In this case, that item is the *src/lib.rs* file, which is the crate root. These comments describe the entire crate.

When we run "cargo doc --open", these comments will display on the front page of the documentation for "my_crate" above the list of public items in the crate, as shown in Figure 14-2:

![Rendered HTML documentation with a comment for the crate as a whole](https://doc.rust-lang.org/book/img/trpl14-02.png)

Figure 14-2: Rendered documentation for "my_crate", including the comment describing the crate as a whole

Documentation comments within items are useful for describing crates and modules especially. Use them to explain the overall purpose of the container to help your users understand the crate’s organization.

### [Exporting a Convenient Public API with "pub use"](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use)

The structure of your public API is a major consideration when publishing a crate. People who use your crate are less familiar with the structure than you are and might have difficulty finding the pieces they want to use if your crate has a large module hierarchy.

In Chapter 7, we covered how to make items public using the "pub" keyword, and bring items into a scope with the "use" keyword. However, the structure that makes sense to you while you’re developing a crate might not be very convenient for your users. You might want to organize your structs in a hierarchy containing multiple levels, but then people who want to use a type you’ve defined deep in the hierarchy might have trouble finding out that type exists. They might also be annoyed at having to enter "use" "my_crate::some_module::another_module::UsefulType;" rather than "use" "my_crate::UsefulType;".

The good news is that if the structure *isn’t* convenient for others to use from another library, you don’t have to rearrange your internal organization: instead, you can re-export items to make a public structure that’s different from your private structure by using "pub use". Re-exporting takes a public item in one location and makes it public in another location, as if it were defined in the other location instead.

For example, say we made a library named "art" for modeling artistic concepts. Within this library are two modules: a "kinds" module containing two enums named "PrimaryColor" and "SecondaryColor" and a "utils" module containing a function named "mix", as shown in Listing 14-3:

Filename: src/lib.rs

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

Listing 14-3: An "art" library with items organized into "kinds" and "utils" modules

Figure 14-3 shows what the front page of the documentation for this crate generated by "cargo doc" would look like:

![Rendered documentation for the "art" crate that lists the "kinds" and "utils" modules](https://doc.rust-lang.org/book/img/trpl14-03.png)

Figure 14-3: Front page of the documentation for "art" that lists the "kinds" and "utils" modules

Note that the "PrimaryColor" and "SecondaryColor" types aren’t listed on the front page, nor is the "mix" function. We have to click "kinds" and "utils" to see them.

Another crate that depends on this library would need "use" statements that bring the items from "art" into scope, specifying the module structure that’s currently defined. Listing 14-4 shows an example of a crate that uses the "PrimaryColor" and "mix" items from the "art" crate:

Filename: src/main.rs

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

Listing 14-4: A crate using the "art" crate’s items with its internal structure exported

The author of the code in Listing 14-4, which uses the "art" crate, had to figure out that "PrimaryColor" is in the "kinds" module and "mix" is in the "utils" module. The module structure of the "art" crate is more relevant to developers working on the "art" crate than to those using it. The internal structure doesn’t contain any useful information for someone trying to understand how to use the "art" crate, but rather causes confusion because developers who use it have to figure out where to look, and must specify the module names in the "use" statements.

To remove the internal organization from the public API, we can modify the "art" crate code in Listing 14-3 to add "pub use" statements to re-export the items at the top level, as shown in Listing 14-5:

Filename: src/lib.rs

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

Listing 14-5: Adding "pub use" statements to re-export items

The API documentation that "cargo doc" generates for this crate will now list and link re-exports on the front page, as shown in Figure 14-4, making the "PrimaryColor" and "SecondaryColor" types and the "mix" function easier to find.

![Rendered documentation for the "art" crate with the re-exports on the front page](https://doc.rust-lang.org/book/img/trpl14-04.png)

Figure 14-4: The front page of the documentation for "art" that lists the re-exports

The "art" crate users can still see and use the internal structure from Listing 14-3 as demonstrated in Listing 14-4, or they can use the more convenient structure in Listing 14-5, as shown in Listing 14-6:

Filename: src/main.rs

```rust
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
```

Listing 14-6: A program using the re-exported items from the "art" crate

In cases where there are many nested modules, re-exporting the types at the top level with "pub use" can make a significant difference in the experience of people who use the crate. Another common use of "pub use" is to re-export definitions of a dependency in the current crate to make that crate"s definitions part of your crate’s public API.

Creating a useful public API structure is more of an art than a science, and you can iterate to find the API that works best for your users. Choosing "pub use" gives you flexibility in how you structure your crate internally and decouples that internal structure from what you present to your users. Look at some of the code of crates you’ve installed to see if their internal structure differs from their public API.

### [Setting Up a Crates.io Account](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#setting-up-a-cratesio-account)

Before you can publish any crates, you need to create an account on [crates.io](https://crates.io/) and get an API token. To do so, visit the home page at [crates.io](https://crates.io/) and log in via a GitHub account. (The GitHub account is currently a requirement, but the site might support other ways of creating an account in the future.) Once you’re logged in, visit your account settings at https://crates.io/me/ and retrieve your API key. Then run the "cargo login" command with your API key, like this:

```bash
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

This command will inform Cargo of your API token and store it locally in *~/.cargo/credentials*. Note that this token is a *secret*: do not share it with anyone else. If you do share it with anyone for any reason, you should revoke it and generate a new token on [crates.io](https://crates.io/).

### [Adding Metadata to a New Crate](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#adding-metadata-to-a-new-crate)

Let’s say you have a crate you want to publish. Before publishing, you’ll need to add some metadata in the "[package]" section of the crate’s *Cargo.toml* file.

Your crate will need a unique name. While you’re working on a crate locally, you can name a crate whatever you’d like. However, crate names on [crates.io](https://crates.io/) are allocated on a first-come, first-served basis. Once a crate name is taken, no one else can publish a crate with that name. Before attempting to publish a crate, search for the name you want to use. If the name has been used, you will need to find another name and edit the "name" field in the *Cargo.toml* file under the "[package]" section to use the new name for publishing, like so:

Filename: Cargo.toml

```toml
[package]
name = "guessing_game"
```

Even if you’ve chosen a unique name, when you run "cargo publish" to publish the crate at this point, you’ll get a warning and then an error:

```bash
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
```

This errors because you’re missing some crucial information: a description and license are required so people will know what your crate does and under what terms they can use it. In *Cargo.toml*, add a description that"s just a sentence or two, because it will appear with your crate in search results. For the "license" field, you need to give a *license identifier value*. The [Linux Foundation’s Software Package Data Exchange (SPDX)](http://spdx.org/licenses/) lists the identifiers you can use for this value. For example, to specify that you’ve licensed your crate using the MIT License, add the "MIT" identifier:

Filename: Cargo.toml

```toml
[package]
name = "guessing_game"
license = "MIT"
```

If you want to use a license that doesn’t appear in the SPDX, you need to place the text of that license in a file, include the file in your project, and then use "license-file" to specify the name of that file instead of using the "license" key.

Guidance on which license is appropriate for your project is beyond the scope of this book. Many people in the Rust community license their projects in the same way as Rust by using a dual license of "MIT OR Apache-2.0". This practice demonstrates that you can also specify multiple license identifiers separated by "OR" to have multiple licenses for your project.

With a unique name, the version, your description, and a license added, the *Cargo.toml* file for a project that is ready to publish might look like this:

Filename: Cargo.toml

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) describes other metadata you can specify to ensure others can discover and use your crate more easily.

### [Publishing to Crates.io](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-to-cratesio)

Now that you’ve created an account, saved your API token, chosen a name for your crate, and specified the required metadata, you’re ready to publish! Publishing a crate uploads a specific version to [crates.io](https://crates.io/) for others to use.

Be careful, because a publish is *permanent*. The version can never be overwritten, and the code cannot be deleted. One major goal of [crates.io](https://crates.io/) is to act as a permanent archive of code so that builds of all projects that depend on crates from [crates.io](https://crates.io/) will continue to work. Allowing version deletions would make fulfilling that goal impossible. However, there is no limit to the number of crate versions you can publish.

Run the "cargo publish" command again. It should succeed now:

```bash
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Congratulations! You’ve now shared your code with the Rust community, and anyone can easily add your crate as a dependency of their project.

### [Publishing a New Version of an Existing Crate](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-a-new-version-of-an-existing-crate)

When you’ve made changes to your crate and are ready to release a new version, you change the "version" value specified in your *Cargo.toml* file and republish. Use the [Semantic Versioning rules](http://semver.org/) to decide what an appropriate next version number is based on the kinds of changes you’ve made. Then run "cargo publish" to upload the new version.



### [Deprecating Versions from Crates.io with "cargo yank"](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#deprecating-versions-from-cratesio-with-cargo-yank)

Although you can’t remove previous versions of a crate, you can prevent any future projects from adding them as a new dependency. This is useful when a crate version is broken for one reason or another. In such situations, Cargo supports *yanking* a crate version.

Yanking a version prevents new projects from depending on that version while allowing all existing projects that depend on it to continue. Essentially, a yank means that all projects with a *Cargo.lock* will not break, and any future *Cargo.lock* files generated will not use the yanked version.

To yank a version of a crate, in the directory of the crate that you’ve previously published, run "cargo yank" and specify which version you want to yank. For example, if we"ve published a crate named "guessing_game" version 1.0.1 and we want to yank it, in the project directory for "guessing_game" we"d run:

```bash
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

By adding "--undo" to the command, you can also undo a yank and allow projects to start depending on a version again:

```bash
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

A yank *does not* delete any code. It cannot, for example, delete accidentally uploaded secrets. If that happens, you must reset those secrets immediately.





---





## [Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#cargo-workspaces)

In Chapter 12, we built a package that included a binary crate and a library crate. As your project develops, you might find that the library crate continues to get bigger and you want to split your package further into multiple library crates. Cargo offers a feature called *workspaces* that can help manage multiple related packages that are developed in tandem.

### [Creating a Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#creating-a-workspace)

A *workspace* is a set of packages that share the same *Cargo.lock* and output directory. Let’s make a project using a workspace—we’ll use trivial code so we can concentrate on the structure of the workspace. There are multiple ways to structure a workspace, so we"ll just show one common way. We’ll have a workspace containing a binary and two libraries. The binary, which will provide the main functionality, will depend on the two libraries. One library will provide an "add_one" function, and a second library an "add_two" function. These three crates will be part of the same workspace. We’ll start by creating a new directory for the workspace:

```bash
$ mkdir add
$ cd add
```

Next, in the *add* directory, we create the *Cargo.toml* file that will configure the entire workspace. This file won’t have a "[package]" section. Instead, it will start with a "[workspace]" section that will allow us to add members to the workspace by specifying the path to the package with our binary crate; in this case, that path is *adder*:

Filename: Cargo.toml

```toml
[workspace]

members = [
    "adder",
]
```

Next, we’ll create the "adder" binary crate by running "cargo new" within the *add* directory:

```bash
$ cargo new adder
     Created binary (application) "adder" package
```

At this point, we can build the workspace by running "cargo build". The files in your *add* directory should look like this:

```
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

The workspace has one *target* directory at the top level that the compiled artifacts will be placed into; the "adder" package doesn’t have its own *target* directory. Even if we were to run "cargo build" from inside the *adder* directory, the compiled artifacts would still end up in *add/target* rather than *add/adder/target*. Cargo structures the *target* directory in a workspace like this because the crates in a workspace are meant to depend on each other. If each crate had its own *target* directory, each crate would have to recompile each of the other crates in the workspace to place the artifacts in its own *target* directory. By sharing one *target* directory, the crates can avoid unnecessary rebuilding.

### [Creating the Second Package in the Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#creating-the-second-package-in-the-workspace)

Next, let’s create another member package in the workspace and call it "add_one". Change the top-level *Cargo.toml* to specify the *add_one* path in the "members" list:

Filename: Cargo.toml

```toml
[workspace]

members = [
    "adder",
    "add_one",
]
```

Then generate a new library crate named "add_one":

```bash
$ cargo new add_one --lib
     Created library "add_one" package
```

Your *add* directory should now have these directories and files:

```
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

In the *add_one/src/lib.rs* file, let’s add an "add_one" function:

Filename: add_one/src/lib.rs

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Now we can have the "adder" package with our binary depend on the "add_one" package that has our library. First, we’ll need to add a path dependency on "add_one" to *adder/Cargo.toml*.

Filename: adder/Cargo.toml

```toml
[dependencies]
add_one = { path = "../add_one" }
```

Cargo doesn’t assume that crates in a workspace will depend on each other, so we need to be explicit about the dependency relationships.

Next, let’s use the "add_one" function (from the "add_one" crate) in the "adder" crate. Open the *adder/src/main.rs* file and add a "use" line at the top to bring the new "add_one" library crate into scope. Then change the "main" function to call the "add_one" function, as in Listing 14-7.

Filename: adder/src/main.rs

```rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
}
```

Listing 14-7: Using the "add_one" library crate from the "adder" crate

Let’s build the workspace by running "cargo build" in the top-level *add* directory!

```bash
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

To run the binary crate from the *add* directory, we can specify which package in the workspace we want to run by using the "-p" argument and the package name with "cargo run":

```bash
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running "target/debug/adder"
Hello, world! 10 plus one is 11!
```

This runs the code in *adder/src/main.rs*, which depends on the "add_one" crate.

#### [Depending on an External Package in a Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#depending-on-an-external-package-in-a-workspace)

Notice that the workspace has only one *Cargo.lock* file at the top level, rather than having a *Cargo.lock* in each crate’s directory. This ensures that all crates are using the same version of all dependencies. If we add the "rand" package to the *adder/Cargo.toml* and *add_one/Cargo.toml* files, Cargo will resolve both of those to one version of "rand" and record that in the one *Cargo.lock*. Making all crates in the workspace use the same dependencies means the crates will always be compatible with each other. Let’s add the "rand" crate to the "[dependencies]" section in the *add_one/Cargo.toml* file so we can use the "rand" crate in the "add_one" crate:

Filename: add_one/Cargo.toml

```toml
[dependencies]
rand = "0.8.5"
```

We can now add "use rand;" to the *add_one/src/lib.rs* file, and building the whole workspace by running "cargo build" in the *add* directory will bring in and compile the "rand" crate. We will get one warning because we aren’t referring to the "rand" we brought into scope:

```bash
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: "rand"
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: "#[warn(unused_imports)]" on by default

warning: "add_one" (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

The top-level *Cargo.lock* now contains information about the dependency of "add_one" on "rand". However, even though "rand" is used somewhere in the workspace, we can’t use it in other crates in the workspace unless we add "rand" to their *Cargo.toml* files as well. For example, if we add "use rand;" to the *adder/src/main.rs* file for the "adder" package, we’ll get an error:

```bash
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import "rand"
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate "rand"
```

To fix this, edit the *Cargo.toml* file for the "adder" package and indicate that "rand" is a dependency for it as well. Building the "adder" package will add "rand" to the list of dependencies for "adder" in *Cargo.lock*, but no additional copies of "rand" will be downloaded. Cargo has ensured that every crate in every package in the workspace using the "rand" package will be using the same version, saving us space and ensuring that the crates in the workspace will be compatible with each other.

#### [Adding a Test to a Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#adding-a-test-to-a-workspace)

For another enhancement, let’s add a test of the "add_one::add_one" function within the "add_one" crate:

Filename: add_one/src/lib.rs

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

Now run "cargo test" in the top-level *add* directory. Running "cargo test" in a workspace structured like this one will run the tests for all the crates in the workspace:

```bash
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The first section of the output shows that the "it_works" test in the "add_one" crate passed. The next section shows that zero tests were found in the "adder" crate, and then the last section shows zero documentation tests were found in the "add_one" crate.

We can also run tests for one particular crate in a workspace from the top-level directory by using the "-p" flag and specifying the name of the crate we want to test:

```bash
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

This output shows "cargo test" only ran the tests for the "add_one" crate and didn’t run the "adder" crate tests.

If you publish the crates in the workspace to [crates.io](https://crates.io/), each crate in the workspace will need to be published separately. Like "cargo test", we can publish a particular crate in our workspace by using the "-p" flag and specifying the name of the crate we want to publish.

For additional practice, add an "add_two" crate to this workspace in a similar way as the "add_one" crate!

As your project grows, consider using a workspace: it’s easier to understand smaller, individual components than one big blob of code. Furthermore, keeping the crates in a workspace can make coordination between crates easier if they are often changed at the same time.





---







## [Installing Binaries with "cargo install"](https://doc.rust-lang.org/book/ch14-04-installing-binaries.html#installing-binaries-with-cargo-install)

The "cargo install" command allows you to install and use binary crates locally. This isn’t intended to replace system packages; it’s meant to be a convenient way for Rust developers to install tools that others have shared on [crates.io](https://crates.io/). Note that you can only install packages that have binary targets. A *binary target* is the runnable program that is created if the crate has a *src/main.rs* file or another file specified as a binary, as opposed to a library target that isn’t runnable on its own but is suitable for including within other programs. Usually, crates have information in the *README* file about whether a crate is a library, has a binary target, or both.

All binaries installed with "cargo install" are stored in the installation root’s *bin* folder. If you installed Rust using *rustup.rs* and don’t have any custom configurations, this directory will be *$HOME/.cargo/bin*. Ensure that directory is in your "$PATH" to be able to run programs you’ve installed with "cargo install".

For example, in Chapter 12 we mentioned that there’s a Rust implementation of the "grep" tool called "ripgrep" for searching files. To install "ripgrep", we can run the following:

```bash
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package "ripgrep v13.0.0" (executable "rg")
```

The second-to-last line of the output shows the location and the name of the installed binary, which in the case of "ripgrep" is "rg". As long as the installation directory is in your "$PATH", as mentioned previously, you can then run "rg --help" and start using a faster, rustier tool for searching files!





---





## [Extending Cargo with Custom Commands](https://doc.rust-lang.org/book/ch14-05-extending-cargo.html#extending-cargo-with-custom-commands)

Cargo is designed so you can extend it with new subcommands without having to modify Cargo. If a binary in your "$PATH" is named "cargo-something", you can run it as if it was a Cargo subcommand by running "cargo something". Custom commands like this are also listed when you run "cargo --list". Being able to use "cargo install" to install extensions and then run them just like the built-in Cargo tools is a super convenient benefit of Cargo’s design!

## [Summary](https://doc.rust-lang.org/book/ch14-05-extending-cargo.html#summary)

Sharing code with Cargo and [crates.io](https://crates.io/) is part of what makes the Rust ecosystem useful for many different tasks. Rust’s standard library is small and stable, but crates are easy to share, use, and improve on a timeline different from that of the language. Don’t be shy about sharing code that’s useful to you on [crates.io](https://crates.io/); it’s likely that it will be useful to someone else as well!





---





# 15



# [Smart Pointers](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html#smart-pointers)

A *pointer* is a general concept for a variable that contains an address in memory. This address refers to, or “points at,” some other data. The most common kind of pointer in Rust is a reference, which you learned about in Chapter 4. References are indicated by the "&" symbol and borrow the value they point to. They don’t have any special capabilities other than referring to data, and have no overhead.

*Smart pointers*, on the other hand, are data structures that act like a pointer but also have additional metadata and capabilities. The concept of smart pointers isn’t unique to Rust: smart pointers originated in C++ and exist in other languages as well. Rust has a variety of smart pointers defined in the standard library that provide functionality beyond that provided by references. To explore the general concept, we’ll look at a couple of different examples of smart pointers, including a *reference counting* smart pointer type. This pointer enables you to allow data to have multiple owners by keeping track of the number of owners and, when no owners remain, cleaning up the data.

Rust, with its concept of ownership and borrowing, has an additional difference between references and smart pointers: while references only borrow data, in many cases, smart pointers *own* the data they point to.

Though we didn’t call them as such at the time, we’ve already encountered a few smart pointers in this book, including "String" and "Vec<T>" in Chapter 8. Both these types count as smart pointers because they own some memory and allow you to manipulate it. They also have metadata and extra capabilities or guarantees. "String", for example, stores its capacity as metadata and has the extra ability to ensure its data will always be valid UTF-8.

Smart pointers are usually implemented using structs. Unlike an ordinary struct, smart pointers implement the "Deref" and "Drop" traits. The "Deref" trait allows an instance of the smart pointer struct to behave like a reference so you can write your code to work with either references or smart pointers. The "Drop" trait allows you to customize the code that’s run when an instance of the smart pointer goes out of scope. In this chapter, we’ll discuss both traits and demonstrate why they’re important to smart pointers.

Given that the smart pointer pattern is a general design pattern used frequently in Rust, this chapter won’t cover every existing smart pointer. Many libraries have their own smart pointers, and you can even write your own. We’ll cover the most common smart pointers in the standard library:

- "Box<T>" for allocating values on the heap
- "Rc<T>", a reference counting type that enables multiple ownership
- "Ref<T>" and "RefMut<T>", accessed through "RefCell<T>", a type that enforces the borrowing rules at runtime instead of compile time

In addition, we’ll cover the *interior mutability* pattern where an immutable type exposes an API for mutating an interior value. We’ll also discuss *reference cycles*: how they can leak memory and how to prevent them.

Let’s dive in!





---



## [Using "Box" to Point to Data on the Heap](https://doc.rust-lang.org/book/ch15-01-box.html#using-boxt-to-point-to-data-on-the-heap)

The most straightforward smart pointer is a *box*, whose type is written "Box<T>". Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data. Refer to Chapter 4 to review the difference between the stack and the heap.

Boxes don’t have performance overhead, other than storing their data on the heap instead of on the stack. But they don’t have many extra capabilities either. You’ll use them most often in these situations:

- When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
- When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
- When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type

We’ll demonstrate the first situation in the [“Enabling Recursive Types with Boxes”](https://doc.rust-lang.org/book/ch15-01-box.html#enabling-recursive-types-with-boxes) section. In the second case, transferring ownership of a large amount of data can take a long time because the data is copied around on the stack. To improve performance in this situation, we can store the large amount of data on the heap in a box. Then, only the small amount of pointer data is copied around on the stack, while the data it references stays in one place on the heap. The third case is known as a *trait object*, and Chapter 17 devotes an entire section, [“Using Trait Objects That Allow for Values of Different Types,”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) just to that topic. So what you learn here you’ll apply again in Chapter 17!

### [Using a "Box" to Store Data on the Heap](https://doc.rust-lang.org/book/ch15-01-box.html#using-a-boxt-to-store-data-on-the-heap)

Before we discuss the heap storage use case for "Box<T>", we’ll cover the syntax and how to interact with values stored within a "Box<T>".

Listing 15-1 shows how to use a box to store an "i32" value on the heap:

Filename: src/main.rs

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

Listing 15-1: Storing an "i32" value on the heap using a box

We define the variable "b" to have the value of a "Box" that points to the value "5", which is allocated on the heap. This program will print "b = 5"; in this case, we can access the data in the box similar to how we would if this data were on the stack. Just like any owned value, when a box goes out of scope, as "b" does at the end of "main", it will be deallocated. The deallocation happens both for the box (stored on the stack) and the data it points to (stored on the heap).

Putting a single value on the heap isn’t very useful, so you won’t use boxes by themselves in this way very often. Having values like a single "i32" on the stack, where they’re stored by default, is more appropriate in the majority of situations. Let’s look at a case where boxes allow us to define types that we wouldn’t be allowed to if we didn’t have boxes.

### [Enabling Recursive Types with Boxes](https://doc.rust-lang.org/book/ch15-01-box.html#enabling-recursive-types-with-boxes)

A value of *recursive type* can have another value of the same type as part of itself. Recursive types pose an issue because at compile time Rust needs to know how much space a type takes up. However, the nesting of values of recursive types could theoretically continue infinitely, so Rust can’t know how much space the value needs. Because boxes have a known size, we can enable recursive types by inserting a box in the recursive type definition.

As an example of a recursive type, let’s explore the *cons list*. This is a data type commonly found in functional programming languages. The cons list type we’ll define is straightforward except for the recursion; therefore, the concepts in the example we’ll work with will be useful any time you get into more complex situations involving recursive types.

#### [More Information About the Cons List](https://doc.rust-lang.org/book/ch15-01-box.html#more-information-about-the-cons-list)

A *cons list* is a data structure that comes from the Lisp programming language and its dialects and is made up of nested pairs, and is the Lisp version of a linked list. Its name comes from the "cons" function (short for “construct function”) in Lisp that constructs a new pair from its two arguments. By calling "cons" on a pair consisting of a value and another pair, we can construct cons lists made up of recursive pairs.

For example, here’s a pseudocode representation of a cons list containing the list 1, 2, 3 with each pair in parentheses:

```
(1, (2, (3, Nil)))
```

Each item in a cons list contains two elements: the value of the current item and the next item. The last item in the list contains only a value called "Nil" without a next item. A cons list is produced by recursively calling the "cons" function. The canonical name to denote the base case of the recursion is "Nil". Note that this is not the same as the “null” or “nil” concept in Chapter 6, which is an invalid or absent value.

The cons list isn’t a commonly used data structure in Rust. Most of the time when you have a list of items in Rust, "Vec<T>" is a better choice to use. Other, more complex recursive data types *are* useful in various situations, but by starting with the cons list in this chapter, we can explore how boxes let us define a recursive data type without much distraction.

Listing 15-2 contains an enum definition for a cons list. Note that this code won’t compile yet because the "List" type doesn’t have a known size, which we’ll demonstrate.

Filename: src/main.rs

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

Listing 15-2: The first attempt at defining an enum to represent a cons list data structure of "i32" values

> Note: We’re implementing a cons list that holds only "i32" values for the purposes of this example. We could have implemented it using generics, as we discussed in Chapter 10, to define a cons list type that could store values of any type.

Using the "List" type to store the list "1, 2, 3" would look like the code in Listing 15-3:

Filename: src/main.rs

```rust
use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

Listing 15-3: Using the "List" enum to store the list "1, 2, 3"

The first "Cons" value holds "1" and another "List" value. This "List" value is another "Cons" value that holds "2" and another "List" value. This "List" value is one more "Cons" value that holds "3" and a "List" value, which is finally "Nil", the non-recursive variant that signals the end of the list.

If we try to compile the code in Listing 15-3, we get the error shown in Listing 15-4:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type "List" has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a "Box", "Rc", or "&") to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

For more information about this error, try "rustc --explain E0072".
error: could not compile "cons-list" due to previous error
```

Listing 15-4: The error we get when attempting to define a recursive enum

The error shows this type “has infinite size.” The reason is that we’ve defined "List" with a variant that is recursive: it holds another value of itself directly. As a result, Rust can’t figure out how much space it needs to store a "List" value. Let’s break down why we get this error. First, we’ll look at how Rust decides how much space it needs to store a value of a non-recursive type.

#### [Computing the Size of a Non-Recursive Type](https://doc.rust-lang.org/book/ch15-01-box.html#computing-the-size-of-a-non-recursive-type)

Recall the "Message" enum we defined in Listing 6-2 when we discussed enum definitions in Chapter 6:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

To determine how much space to allocate for a "Message" value, Rust goes through each of the variants to see which variant needs the most space. Rust sees that "Message::Quit" doesn’t need any space, "Message::Move" needs enough space to store two "i32" values, and so forth. Because only one variant will be used, the most space a "Message" value will need is the space it would take to store the largest of its variants.

Contrast this with what happens when Rust tries to determine how much space a recursive type like the "List" enum in Listing 15-2 needs. The compiler starts by looking at the "Cons" variant, which holds a value of type "i32" and a value of type "List". Therefore, "Cons" needs an amount of space equal to the size of an "i32" plus the size of a "List". To figure out how much memory the "List" type needs, the compiler looks at the variants, starting with the "Cons" variant. The "Cons" variant holds a value of type "i32" and a value of type "List", and this process continues infinitely, as shown in Figure 15-1.

![An infinite Cons list](https://doc.rust-lang.org/book/img/trpl15-01.svg)

Figure 15-1: An infinite "List" consisting of infinite "Cons" variants

#### [Using "Box" to Get a Recursive Type with a Known Size](https://doc.rust-lang.org/book/ch15-01-box.html#using-boxt-to-get-a-recursive-type-with-a-known-size)

Because Rust can’t figure out how much space to allocate for recursively defined types, the compiler gives an error with this helpful suggestion:

```
help: insert some indirection (e.g., a "Box", "Rc", or "&") to make "List" representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

In this suggestion, “indirection” means that instead of storing a value directly, we should change the data structure to store the value indirectly by storing a pointer to the value instead.

Because a "Box<T>" is a pointer, Rust always knows how much space a "Box<T>" needs: a pointer’s size doesn’t change based on the amount of data it’s pointing to. This means we can put a "Box<T>" inside the "Cons" variant instead of another "List" value directly. The "Box<T>" will point to the next "List" value that will be on the heap rather than inside the "Cons" variant. Conceptually, we still have a list, created with lists holding other lists, but this implementation is now more like placing the items next to one another rather than inside one another.

We can change the definition of the "List" enum in Listing 15-2 and the usage of the "List" in Listing 15-3 to the code in Listing 15-5, which will compile:

Filename: src/main.rs

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

Listing 15-5: Definition of "List" that uses "Box<T>" in order to have a known size

The "Cons" variant needs the size of an "i32" plus the space to store the box’s pointer data. The "Nil" variant stores no values, so it needs less space than the "Cons" variant. We now know that any "List" value will take up the size of an "i32" plus the size of a box’s pointer data. By using a box, we’ve broken the infinite, recursive chain, so the compiler can figure out the size it needs to store a "List" value. Figure 15-2 shows what the "Cons" variant looks like now.

![A finite Cons list](https://doc.rust-lang.org/book/img/trpl15-02.svg)

Figure 15-2: A "List" that is not infinitely sized because "Cons" holds a "Box"

Boxes provide only the indirection and heap allocation; they don’t have any other special capabilities, like those we’ll see with the other smart pointer types. They also don’t have the performance overhead that these special capabilities incur, so they can be useful in cases like the cons list where the indirection is the only feature we need. We’ll look at more use cases for boxes in Chapter 17, too.

The "Box<T>" type is a smart pointer because it implements the "Deref" trait, which allows "Box<T>" values to be treated like references. When a "Box<T>" value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the "Drop" trait implementation. These two traits will be even more important to the functionality provided by the other smart pointer types we’ll discuss in the rest of this chapter. Let’s explore these two traits in more detail.





---





## [Treating Smart Pointers Like Regular References with the "Deref" Trait](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait)

Implementing the "Deref" trait allows you to customize the behavior of the *dereference operator* "*" (not to be confused with the multiplication or glob operator). By implementing "Deref" in such a way that a smart pointer can be treated like a regular reference, you can write code that operates on references and use that code with smart pointers too.

Let’s first look at how the dereference operator works with regular references. Then we’ll try to define a custom type that behaves like "Box<T>", and see why the dereference operator doesn’t work like a reference on our newly defined type. We’ll explore how implementing the "Deref" trait makes it possible for smart pointers to work in ways similar to references. Then we’ll look at Rust’s *deref coercion* feature and how it lets us work with either references or smart pointers.

> Note: there’s one big difference between the "MyBox<T>" type we’re about to build and the real "Box<T>": our version will not store its data on the heap. We are focusing this example on "Deref", so where the data is actually stored is less important than the pointer-like behavior.



### [Following the Pointer to the Value](https://doc.rust-lang.org/book/ch15-02-deref.html#following-the-pointer-to-the-value)

A regular reference is a type of pointer, and one way to think of a pointer is as an arrow to a value stored somewhere else. In Listing 15-6, we create a reference to an "i32" value and then use the dereference operator to follow the reference to the value:

Filename: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Listing 15-6: Using the dereference operator to follow a reference to an "i32" value

The variable "x" holds an "i32" value "5". We set "y" equal to a reference to "x". We can assert that "x" is equal to "5". However, if we want to make an assertion about the value in "y", we have to use "*y" to follow the reference to the value it’s pointing to (hence *dereference*) so the compiler can compare the actual value. Once we dereference "y", we have access to the integer value "y" is pointing to that we can compare with "5".

If we tried to write "assert_eq!(5, y);" instead, we would get this compilation error:

```bash
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0277]: can"t compare "{integer}" with "&{integer}"
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^ no implementation for "{integer} == &{integer}"
  |
  = help: the trait "PartialEq<&{integer}>" is not implemented for "{integer}"
  = help: the following other types implement trait "PartialEq<Rhs>":
            f32
            f64
            i128
            i16
            i32
            i64
            i8
            isize
          and 6 others
  = note: this error originates in the macro "assert_eq" (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try "rustc --explain E0277".
error: could not compile "deref-example" due to previous error
```

Comparing a number and a reference to a number isn’t allowed because they’re different types. We must use the dereference operator to follow the reference to the value it’s pointing to.

### [Using "Box" Like a Reference](https://doc.rust-lang.org/book/ch15-02-deref.html#using-boxt-like-a-reference)

We can rewrite the code in Listing 15-6 to use a "Box<T>" instead of a reference; the dereference operator used on the "Box<T>" in Listing 15-7 functions in the same way as the dereference operator used on the reference in Listing 15-6:

Filename: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Listing 15-7: Using the dereference operator on a "Box<i32>"

The main difference between Listing 15-7 and Listing 15-6 is that here we set "y" to be an instance of a "Box<T>" pointing to a copied value of "x" rather than a reference pointing to the value of "x". In the last assertion, we can use the dereference operator to follow the pointer of the "Box<T>" in the same way that we did when "y" was a reference. Next, we’ll explore what is special about "Box<T>" that enables us to use the dereference operator by defining our own type.

### [Defining Our Own Smart Pointer](https://doc.rust-lang.org/book/ch15-02-deref.html#defining-our-own-smart-pointer)

Let’s build a smart pointer similar to the "Box<T>" type provided by the standard library to experience how smart pointers behave differently from references by default. Then we’ll look at how to add the ability to use the dereference operator.

The "Box<T>" type is ultimately defined as a tuple struct with one element, so Listing 15-8 defines a "MyBox<T>" type in the same way. We’ll also define a "new" function to match the "new" function defined on "Box<T>".

Filename: src/main.rs

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

Listing 15-8: Defining a "MyBox<T>" type

We define a struct named "MyBox" and declare a generic parameter "T", because we want our type to hold values of any type. The "MyBox" type is a tuple struct with one element of type "T". The "MyBox::new" function takes one parameter of type "T" and returns a "MyBox" instance that holds the value passed in.

Let’s try adding the "main" function in Listing 15-7 to Listing 15-8 and changing it to use the "MyBox<T>" type we’ve defined instead of "Box<T>". The code in Listing 15-9 won’t compile because Rust doesn’t know how to dereference "MyBox".

Filename: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Listing 15-9: Attempting to use "MyBox<T>" in the same way we used references and "Box<T>"

Here’s the resulting compilation error:

```bash
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0614]: type "MyBox<{integer}>" cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^

For more information about this error, try "rustc --explain E0614".
error: could not compile "deref-example" due to previous error
```

Our "MyBox<T>" type can’t be dereferenced because we haven’t implemented that ability on our type. To enable dereferencing with the "*" operator, we implement the "Deref" trait.

### [Treating a Type Like a Reference by Implementing the "Deref" Trait](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-a-type-like-a-reference-by-implementing-the-deref-trait)

As discussed in the [“Implementing a Trait on a Type”](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type) section of Chapter 10, to implement a trait, we need to provide implementations for the trait’s required methods. The "Deref" trait, provided by the standard library, requires us to implement one method named "deref" that borrows "self" and returns a reference to the inner data. Listing 15-10 contains an implementation of "Deref" to add to the definition of "MyBox":

Filename: src/main.rs

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

Listing 15-10: Implementing "Deref" on "MyBox<T>"

The "type Target = T;" syntax defines an associated type for the "Deref" trait to use. Associated types are a slightly different way of declaring a generic parameter, but you don’t need to worry about them for now; we’ll cover them in more detail in Chapter 19.

We fill in the body of the "deref" method with "&self.0" so "deref" returns a reference to the value we want to access with the "*" operator; recall from the [“Using Tuple Structs without Named Fields to Create Different Types”](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types) section of Chapter 5 that ".0" accesses the first value in a tuple struct. The "main" function in Listing 15-9 that calls "*" on the "MyBox<T>" value now compiles, and the assertions pass!

Without the "Deref" trait, the compiler can only dereference "&" references. The "deref" method gives the compiler the ability to take a value of any type that implements "Deref" and call the "deref" method to get a "&" reference that it knows how to dereference.

When we entered "*y" in Listing 15-9, behind the scenes Rust actually ran this code:

```rust
*(y.deref())
```

Rust substitutes the "*" operator with a call to the "deref" method and then a plain dereference so we don’t have to think about whether or not we need to call the "deref" method. This Rust feature lets us write code that functions identically whether we have a regular reference or a type that implements "Deref".

The reason the "deref" method returns a reference to a value, and that the plain dereference outside the parentheses in "*(y.deref())" is still necessary, is to do with the ownership system. If the "deref" method returned the value directly instead of a reference to the value, the value would be moved out of "self". We don’t want to take ownership of the inner value inside "MyBox<T>" in this case or in most cases where we use the dereference operator.

Note that the "*" operator is replaced with a call to the "deref" method and then a call to the "*" operator just once, each time we use a "*" in our code. Because the substitution of the "*" operator does not recurse infinitely, we end up with data of type "i32", which matches the "5" in "assert_eq!" in Listing 15-9.

### [Implicit Deref Coercions with Functions and Methods](https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods)

*Deref coercion* converts a reference to a type that implements the "Deref" trait into a reference to another type. For example, deref coercion can convert "&String" to "&str" because "String" implements the "Deref" trait such that it returns "&str". Deref coercion is a convenience Rust performs on arguments to functions and methods, and works only on types that implement the "Deref" trait. It happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition. A sequence of calls to the "deref" method converts the type we provided into the type the parameter needs.

Deref coercion was added to Rust so that programmers writing function and method calls don’t need to add as many explicit references and dereferences with "&" and "*". The deref coercion feature also lets us write more code that can work for either references or smart pointers.

To see deref coercion in action, let’s use the "MyBox<T>" type we defined in Listing 15-8 as well as the implementation of "Deref" that we added in Listing 15-10. Listing 15-11 shows the definition of a function that has a string slice parameter:

Filename: src/main.rs

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}
```

Listing 15-11: A "hello" function that has the parameter "name" of type "&str"

We can call the "hello" function with a string slice as an argument, such as "hello("Rust");" for example. Deref coercion makes it possible to call "hello" with a reference to a value of type "MyBox<String>", as shown in Listing 15-12:

Filename: src/main.rs

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

Listing 15-12: Calling "hello" with a reference to a "MyBox<String>" value, which works because of deref coercion

Here we’re calling the "hello" function with the argument "&m", which is a reference to a "MyBox<String>" value. Because we implemented the "Deref" trait on "MyBox<T>" in Listing 15-10, Rust can turn "&MyBox<String>" into "&String" by calling "deref". The standard library provides an implementation of "Deref" on "String" that returns a string slice, and this is in the API documentation for "Deref". Rust calls "deref" again to turn the "&String" into "&str", which matches the "hello" function’s definition.

If Rust didn’t implement deref coercion, we would have to write the code in Listing 15-13 instead of the code in Listing 15-12 to call "hello" with a value of type "&MyBox<String>".

Filename: src/main.rs

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

Listing 15-13: The code we would have to write if Rust didn’t have deref coercion

The "(*m)" dereferences the "MyBox<String>" into a "String". Then the "&" and "[..]" take a string slice of the "String" that is equal to the whole string to match the signature of "hello". This code without deref coercions is harder to read, write, and understand with all of these symbols involved. Deref coercion allows Rust to handle these conversions for us automatically.

When the "Deref" trait is defined for the types involved, Rust will analyze the types and use "Deref::deref" as many times as necessary to get a reference to match the parameter’s type. The number of times that "Deref::deref" needs to be inserted is resolved at compile time, so there is no runtime penalty for taking advantage of deref coercion!

### [How Deref Coercion Interacts with Mutability](https://doc.rust-lang.org/book/ch15-02-deref.html#how-deref-coercion-interacts-with-mutability)

Similar to how you use the "Deref" trait to override the "*" operator on immutable references, you can use the "DerefMut" trait to override the "*" operator on mutable references.

Rust does deref coercion when it finds types and trait implementations in three cases:

- From "&T" to "&U" when "T: Deref<Target=U>"
- From "&mut T" to "&mut U" when "T: DerefMut<Target=U>"
- From "&mut T" to "&U" when "T: Deref<Target=U>"

The first two cases are the same as each other except that the second implements mutability. The first case states that if you have a "&T", and "T" implements "Deref" to some type "U", you can get a "&U" transparently. The second case states that the same deref coercion happens for mutable references.

The third case is trickier: Rust will also coerce a mutable reference to an immutable one. But the reverse is *not* possible: immutable references will never coerce to mutable references. Because of the borrowing rules, if you have a mutable reference, that mutable reference must be the only reference to that data (otherwise, the program wouldn’t compile). Converting one mutable reference to one immutable reference will never break the borrowing rules. Converting an immutable reference to a mutable reference would require that the initial immutable reference is the only immutable reference to that data, but the borrowing rules don’t guarantee that. Therefore, Rust can’t make the assumption that converting an immutable reference to a mutable reference is possible.





---





## [Running Code on Cleanup with the "Drop" Trait](https://doc.rust-lang.org/book/ch15-03-drop.html#running-code-on-cleanup-with-the-drop-trait)

The second trait important to the smart pointer pattern is "Drop", which lets you customize what happens when a value is about to go out of scope. You can provide an implementation for the "Drop" trait on any type, and that code can be used to release resources like files or network connections.

We’re introducing "Drop" in the context of smart pointers because the functionality of the "Drop" trait is almost always used when implementing a smart pointer. For example, when a "Box<T>" is dropped it will deallocate the space on the heap that the box points to.

In some languages, for some types, the programmer must call code to free memory or resources every time they finish using an instance of those types. Examples include file handles, sockets, or locks. If they forget, the system might become overloaded and crash. In Rust, you can specify that a particular bit of code be run whenever a value goes out of scope, and the compiler will insert this code automatically. As a result, you don’t need to be careful about placing cleanup code everywhere in a program that an instance of a particular type is finished with—you still won’t leak resources!

You specify the code to run when a value goes out of scope by implementing the "Drop" trait. The "Drop" trait requires you to implement one method named "drop" that takes a mutable reference to "self". To see when Rust calls "drop", let’s implement "drop" with "println!" statements for now.

Listing 15-14 shows a "CustomSmartPointer" struct whose only custom functionality is that it will print "Dropping CustomSmartPointer!" when the instance goes out of scope, to show when Rust runs the "drop" function.

Filename: src/main.rs

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data "{}"!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

Listing 15-14: A "CustomSmartPointer" struct that implements the "Drop" trait where we would put our cleanup code

The "Drop" trait is included in the prelude, so we don’t need to bring it into scope. We implement the "Drop" trait on "CustomSmartPointer" and provide an implementation for the "drop" method that calls "println!". The body of the "drop" function is where you would place any logic that you wanted to run when an instance of your type goes out of scope. We’re printing some text here to demonstrate visually when Rust will call "drop".

In "main", we create two instances of "CustomSmartPointer" and then print "CustomSmartPointers created". At the end of "main", our instances of "CustomSmartPointer" will go out of scope, and Rust will call the code we put in the "drop" method, printing our final message. Note that we didn’t need to call the "drop" method explicitly.

When we run this program, we’ll see the following output:

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running "target/debug/drop-example"
CustomSmartPointers created.
Dropping CustomSmartPointer with data "other stuff"!
Dropping CustomSmartPointer with data "my stuff"!
```

Rust automatically called "drop" for us when our instances went out of scope, calling the code we specified. Variables are dropped in the reverse order of their creation, so "d" was dropped before "c". This example’s purpose is to give you a visual guide to how the "drop" method works; usually you would specify the cleanup code that your type needs to run rather than a print message.

### [Dropping a Value Early with "std::mem::drop"](https://doc.rust-lang.org/book/ch15-03-drop.html#dropping-a-value-early-with-stdmemdrop)

Unfortunately, it’s not straightforward to disable the automatic "drop" functionality. Disabling "drop" isn’t usually necessary; the whole point of the "Drop" trait is that it’s taken care of automatically. Occasionally, however, you might want to clean up a value early. One example is when using smart pointers that manage locks: you might want to force the "drop" method that releases the lock so that other code in the same scope can acquire the lock. Rust doesn’t let you call the "Drop" trait’s "drop" method manually; instead you have to call the "std::mem::drop" function provided by the standard library if you want to force a value to be dropped before the end of its scope.

If we try to call the "Drop" trait’s "drop" method manually by modifying the "main" function from Listing 15-14, as shown in Listing 15-15, we’ll get a compiler error:

Filename: src/main.rs

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
```

Listing 15-15: Attempting to call the "drop" method from the "Drop" trait manually to clean up early

When we try to compile this code, we’ll get this error:

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
error[E0040]: explicit use of destructor method
  --> src/main.rs:16:7
   |
16 |     c.drop();
   |     --^^^^--
   |     | |
   |     | explicit destructor calls not allowed
   |     help: consider using "drop" function: "drop(c)"

For more information about this error, try "rustc --explain E0040".
error: could not compile "drop-example" due to previous error
```

This error message states that we’re not allowed to explicitly call "drop". The error message uses the term *destructor*, which is the general programming term for a function that cleans up an instance. A *destructor* is analogous to a *constructor*, which creates an instance. The "drop" function in Rust is one particular destructor.

Rust doesn’t let us call "drop" explicitly because Rust would still automatically call "drop" on the value at the end of "main". This would cause a *double free* error because Rust would be trying to clean up the same value twice.

We can’t disable the automatic insertion of "drop" when a value goes out of scope, and we can’t call the "drop" method explicitly. So, if we need to force a value to be cleaned up early, we use the "std::mem::drop" function.

The "std::mem::drop" function is different from the "drop" method in the "Drop" trait. We call it by passing as an argument the value we want to force drop. The function is in the prelude, so we can modify "main" in Listing 15-15 to call the "drop" function, as shown in Listing 15-16:

Filename: src/main.rs

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

Listing 15-16: Calling "std::mem::drop" to explicitly drop a value before it goes out of scope

Running this code will print the following:

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running "target/debug/drop-example"
CustomSmartPointer created.
Dropping CustomSmartPointer with data "some data"!
CustomSmartPointer dropped before the end of main.
```

The text "Dropping CustomSmartPointer with data "some data"!" is printed between the "CustomSmartPointer created." and "CustomSmartPointer dropped before the end of main." text, showing that the "drop" method code is called to drop "c" at that point.

You can use code specified in a "Drop" trait implementation in many ways to make cleanup convenient and safe: for instance, you could use it to create your own memory allocator! With the "Drop" trait and Rust’s ownership system, you don’t have to remember to clean up because Rust does it automatically.

You also don’t have to worry about problems resulting from accidentally cleaning up values still in use: the ownership system that makes sure references are always valid also ensures that "drop" gets called only once when the value is no longer being used.

Now that we’ve examined "Box<T>" and some of the characteristics of smart pointers, let’s look at a few other smart pointers defined in the standard library.





---





## ["Rc", the Reference Counted Smart Pointer](https://doc.rust-lang.org/book/ch15-04-rc.html#rct-the-reference-counted-smart-pointer)

In the majority of cases, ownership is clear: you know exactly which variable owns a given value. However, there are cases when a single value might have multiple owners. For example, in graph data structures, multiple edges might point to the same node, and that node is conceptually owned by all of the edges that point to it. A node shouldn’t be cleaned up unless it doesn’t have any edges pointing to it and so has no owners.

You have to enable multiple ownership explicitly by using the Rust type "Rc<T>", which is an abbreviation for *reference counting*. The "Rc<T>" type keeps track of the number of references to a value to determine whether or not the value is still in use. If there are zero references to a value, the value can be cleaned up without any references becoming invalid.

Imagine "Rc<T>" as a TV in a family room. When one person enters to watch TV, they turn it on. Others can come into the room and watch the TV. When the last person leaves the room, they turn off the TV because it’s no longer being used. If someone turns off the TV while others are still watching it, there would be uproar from the remaining TV watchers!

We use the "Rc<T>" type when we want to allocate some data on the heap for multiple parts of our program to read and we can’t determine at compile time which part will finish using the data last. If we knew which part would finish last, we could just make that part the data’s owner, and the normal ownership rules enforced at compile time would take effect.

Note that "Rc<T>" is only for use in single-threaded scenarios. When we discuss concurrency in Chapter 16, we’ll cover how to do reference counting in multithreaded programs.

### [Using "Rc" to Share Data](https://doc.rust-lang.org/book/ch15-04-rc.html#using-rct-to-share-data)

Let’s return to our cons list example in Listing 15-5. Recall that we defined it using "Box<T>". This time, we’ll create two lists that both share ownership of a third list. Conceptually, this looks similar to Figure 15-3:

![Two lists that share ownership of a third list](https://doc.rust-lang.org/book/img/trpl15-03.svg)

Figure 15-3: Two lists, "b" and "c", sharing ownership of a third list, "a"

We’ll create list "a" that contains 5 and then 10. Then we’ll make two more lists: "b" that starts with 3 and "c" that starts with 4. Both "b" and "c" lists will then continue on to the first "a" list containing 5 and 10. In other words, both lists will share the first list containing 5 and 10.

Trying to implement this scenario using our definition of "List" with "Box<T>" won’t work, as shown in Listing 15-17:

Filename: src/main.rs

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

Listing 15-17: Demonstrating we’re not allowed to have two lists using "Box<T>" that try to share ownership of a third list

When we compile this code, we get this error:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0382]: use of moved value: "a"
  --> src/main.rs:11:30
   |
9  |     let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
   |         - move occurs because "a" has type "List", which does not implement the "Copy" trait
10 |     let b = Cons(3, Box::new(a));
   |                              - value moved here
11 |     let c = Cons(4, Box::new(a));
   |                              ^ value used here after move

For more information about this error, try "rustc --explain E0382".
error: could not compile "cons-list" due to previous error
```

The "Cons" variants own the data they hold, so when we create the "b" list, "a" is moved into "b" and "b" owns "a". Then, when we try to use "a" again when creating "c", we’re not allowed to because "a" has been moved.

We could change the definition of "Cons" to hold references instead, but then we would have to specify lifetime parameters. By specifying lifetime parameters, we would be specifying that every element in the list will live at least as long as the entire list. This is the case for the elements and lists in Listing 15-17, but not in every scenario.

Instead, we’ll change our definition of "List" to use "Rc<T>" in place of "Box<T>", as shown in Listing 15-18. Each "Cons" variant will now hold a value and an "Rc<T>" pointing to a "List". When we create "b", instead of taking ownership of "a", we’ll clone the "Rc<List>" that "a" is holding, thereby increasing the number of references from one to two and letting "a" and "b" share ownership of the data in that "Rc<List>". We’ll also clone "a" when creating "c", increasing the number of references from two to three. Every time we call "Rc::clone", the reference count to the data within the "Rc<List>" will increase, and the data won’t be cleaned up unless there are zero references to it.

Filename: src/main.rs

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

Listing 15-18: A definition of "List" that uses "Rc<T>"

We need to add a "use" statement to bring "Rc<T>" into scope because it’s not in the prelude. In "main", we create the list holding 5 and 10 and store it in a new "Rc<List>" in "a". Then when we create "b" and "c", we call the "Rc::clone" function and pass a reference to the "Rc<List>" in "a" as an argument.

We could have called "a.clone()" rather than "Rc::clone(&a)", but Rust’s convention is to use "Rc::clone" in this case. The implementation of "Rc::clone" doesn’t make a deep copy of all the data like most types’ implementations of "clone" do. The call to "Rc::clone" only increments the reference count, which doesn’t take much time. Deep copies of data can take a lot of time. By using "Rc::clone" for reference counting, we can visually distinguish between the deep-copy kinds of clones and the kinds of clones that increase the reference count. When looking for performance problems in the code, we only need to consider the deep-copy clones and can disregard calls to "Rc::clone".

### [Cloning an "Rc" Increases the Reference Count](https://doc.rust-lang.org/book/ch15-04-rc.html#cloning-an-rct-increases-the-reference-count)

Let’s change our working example in Listing 15-18 so we can see the reference counts changing as we create and drop references to the "Rc<List>" in "a".

In Listing 15-19, we’ll change "main" so it has an inner scope around list "c"; then we can see how the reference count changes when "c" goes out of scope.

Filename: src/main.rs

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

Listing 15-19: Printing the reference count

At each point in the program where the reference count changes, we print the reference count, which we get by calling the "Rc::strong_count" function. This function is named "strong_count" rather than "count" because the "Rc<T>" type also has a "weak_count"; we’ll see what "weak_count" is used for in the [“Preventing Reference Cycles: Turning an "Rc" into a "Weak"”](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#preventing-reference-cycles-turning-an-rct-into-a-weakt) section.

This code prints the following:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.45s
     Running "target/debug/cons-list"
count after creating a = 1
count after creating b = 2
count after creating c = 3
count after c goes out of scope = 2
```

We can see that the "Rc<List>" in "a" has an initial reference count of 1; then each time we call "clone", the count goes up by 1. When "c" goes out of scope, the count goes down by 1. We don’t have to call a function to decrease the reference count like we have to call "Rc::clone" to increase the reference count: the implementation of the "Drop" trait decreases the reference count automatically when an "Rc<T>" value goes out of scope.

What we can’t see in this example is that when "b" and then "a" go out of scope at the end of "main", the count is then 0, and the "Rc<List>" is cleaned up completely. Using "Rc<T>" allows a single value to have multiple owners, and the count ensures that the value remains valid as long as any of the owners still exist.

Via immutable references, "Rc<T>" allows you to share data between multiple parts of your program for reading only. If "Rc<T>" allowed you to have multiple mutable references too, you might violate one of the borrowing rules discussed in Chapter 4: multiple mutable borrows to the same place can cause data races and inconsistencies. But being able to mutate data is very useful! In the next section, we’ll discuss the interior mutability pattern and the "RefCell<T>" type that you can use in conjunction with an "Rc<T>" to work with this immutability restriction.





---





## ["RefCell" and the Interior Mutability Pattern](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#refcellt-and-the-interior-mutability-pattern)

*Interior mutability* is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data; normally, this action is disallowed by the borrowing rules. To mutate data, the pattern uses "unsafe" code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing. Unsafe code indicates to the compiler that we’re checking the rules manually instead of relying on the compiler to check them for us; we will discuss unsafe code more in Chapter 19.

We can use types that use the interior mutability pattern only when we can ensure that the borrowing rules will be followed at runtime, even though the compiler can’t guarantee that. The "unsafe" code involved is then wrapped in a safe API, and the outer type is still immutable.

Let’s explore this concept by looking at the "RefCell<T>" type that follows the interior mutability pattern.

### [Enforcing Borrowing Rules at Runtime with "RefCell"](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#enforcing-borrowing-rules-at-runtime-with-refcellt)

Unlike "Rc<T>", the "RefCell<T>" type represents single ownership over the data it holds. So, what makes "RefCell<T>" different from a type like "Box<T>"? Recall the borrowing rules you learned in Chapter 4:

- At any given time, you can have *either* (but not both) one mutable reference or any number of immutable references.
- References must always be valid.

With references and "Box<T>", the borrowing rules’ invariants are enforced at compile time. With "RefCell<T>", these invariants are enforced *at runtime*. With references, if you break these rules, you’ll get a compiler error. With "RefCell<T>", if you break these rules, your program will panic and exit.

The advantages of checking the borrowing rules at compile time are that errors will be caught sooner in the development process, and there is no impact on runtime performance because all the analysis is completed beforehand. For those reasons, checking the borrowing rules at compile time is the best choice in the majority of cases, which is why this is Rust’s default.

The advantage of checking the borrowing rules at runtime instead is that certain memory-safe scenarios are then allowed, where they would’ve been disallowed by the compile-time checks. Static analysis, like the Rust compiler, is inherently conservative. Some properties of code are impossible to detect by analyzing the code: the most famous example is the Halting Problem, which is beyond the scope of this book but is an interesting topic to research.

Because some analysis is impossible, if the Rust compiler can’t be sure the code complies with the ownership rules, it might reject a correct program; in this way, it’s conservative. If Rust accepted an incorrect program, users wouldn’t be able to trust in the guarantees Rust makes. However, if Rust rejects a correct program, the programmer will be inconvenienced, but nothing catastrophic can occur. The "RefCell<T>" type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

Similar to "Rc<T>", "RefCell<T>" is only for use in single-threaded scenarios and will give you a compile-time error if you try using it in a multithreaded context. We’ll talk about how to get the functionality of "RefCell<T>" in a multithreaded program in Chapter 16.

Here is a recap of the reasons to choose "Box<T>", "Rc<T>", or "RefCell<T>":

- "Rc<T>" enables multiple owners of the same data; "Box<T>" and "RefCell<T>" have single owners.
- "Box<T>" allows immutable or mutable borrows checked at compile time; "Rc<T>" allows only immutable borrows checked at compile time; "RefCell<T>" allows immutable or mutable borrows checked at runtime.
- Because "RefCell<T>" allows mutable borrows checked at runtime, you can mutate the value inside the "RefCell<T>" even when the "RefCell<T>" is immutable.

Mutating the value inside an immutable value is the *interior mutability* pattern. Let’s look at a situation in which interior mutability is useful and examine how it’s possible.

### [Interior Mutability: A Mutable Borrow to an Immutable Value](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#interior-mutability-a-mutable-borrow-to-an-immutable-value)

A consequence of the borrowing rules is that when you have an immutable value, you can’t borrow it mutably. For example, this code won’t compile:

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

If you tried to compile this code, you’d get the following error:

```bash
$ cargo run
   Compiling borrowing v0.1.0 (file:///projects/borrowing)
error[E0596]: cannot borrow "x" as mutable, as it is not declared as mutable
 --> src/main.rs:3:13
  |
2 |     let x = 5;
  |         - help: consider changing this to be mutable: "mut x"
3 |     let y = &mut x;
  |             ^^^^^^ cannot borrow as mutable

For more information about this error, try "rustc --explain E0596".
error: could not compile "borrowing" due to previous error
```

However, there are situations in which it would be useful for a value to mutate itself in its methods but appear immutable to other code. Code outside the value’s methods would not be able to mutate the value. Using "RefCell<T>" is one way to get the ability to have interior mutability, but "RefCell<T>" doesn’t get around the borrowing rules completely: the borrow checker in the compiler allows this interior mutability, and the borrowing rules are checked at runtime instead. If you violate the rules, you’ll get a "panic!" instead of a compiler error.

Let’s work through a practical example where we can use "RefCell<T>" to mutate an immutable value and see why that is useful.

#### [A Use Case for Interior Mutability: Mock Objects](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#a-use-case-for-interior-mutability-mock-objects)

Sometimes during testing a programmer will use a type in place of another type, in order to observe particular behavior and assert it’s implemented correctly. This placeholder type is called a *test double*. Think of it in the sense of a “stunt double” in filmmaking, where a person steps in and substitutes for an actor to do a particular tricky scene. Test doubles stand in for other types when we’re running tests. *Mock objects* are specific types of test doubles that record what happens during a test so you can assert that the correct actions took place.

Rust doesn’t have objects in the same sense as other languages have objects, and Rust doesn’t have mock object functionality built into the standard library as some other languages do. However, you can definitely create a struct that will serve the same purposes as a mock object.

Here’s the scenario we’ll test: we’ll create a library that tracks a value against a maximum value and sends messages based on how close to the maximum value the current value is. This library could be used to keep track of a user’s quota for the number of API calls they’re allowed to make, for example.

Our library will only provide the functionality of tracking how close to the maximum a value is and what the messages should be at what times. Applications that use our library will be expected to provide the mechanism for sending the messages: the application could put a message in the application, send an email, send a text message, or something else. The library doesn’t need to know that detail. All it needs is something that implements a trait we’ll provide called "Messenger". Listing 15-20 shows the library code:

Filename: src/lib.rs

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<"a, T: Messenger> {
    messenger: &"a T,
    value: usize,
    max: usize,
}

impl<"a, T> LimitTracker<"a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &"a T, max: usize) -> LimitTracker<"a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You"ve used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You"ve used up over 75% of your quota!");
        }
    }
}
```

Listing 15-20: A library to keep track of how close a value is to a maximum value and warn when the value is at certain levels

One important part of this code is that the "Messenger" trait has one method called "send" that takes an immutable reference to "self" and the text of the message. This trait is the interface our mock object needs to implement so that the mock can be used in the same way a real object is. The other important part is that we want to test the behavior of the "set_value" method on the "LimitTracker". We can change what we pass in for the "value" parameter, but "set_value" doesn’t return anything for us to make assertions on. We want to be able to say that if we create a "LimitTracker" with something that implements the "Messenger" trait and a particular value for "max", when we pass different numbers for "value", the messenger is told to send the appropriate messages.

We need a mock object that, instead of sending an email or text message when we call "send", will only keep track of the messages it’s told to send. We can create a new instance of the mock object, create a "LimitTracker" that uses the mock object, call the "set_value" method on "LimitTracker", and then check that the mock object has the messages we expect. Listing 15-21 shows an attempt to implement a mock object to do just that, but the borrow checker won’t allow it:

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

Listing 15-21: An attempt to implement a "MockMessenger" that isn’t allowed by the borrow checker

This test code defines a "MockMessenger" struct that has a "sent_messages" field with a "Vec" of "String" values to keep track of the messages it’s told to send. We also define an associated function "new" to make it convenient to create new "MockMessenger" values that start with an empty list of messages. We then implement the "Messenger" trait for "MockMessenger" so we can give a "MockMessenger" to a "LimitTracker". In the definition of the "send" method, we take the message passed in as a parameter and store it in the "MockMessenger" list of "sent_messages".

In the test, we’re testing what happens when the "LimitTracker" is told to set "value" to something that is more than 75 percent of the "max" value. First, we create a new "MockMessenger", which will start with an empty list of messages. Then we create a new "LimitTracker" and give it a reference to the new "MockMessenger" and a "max" value of 100. We call the "set_value" method on the "LimitTracker" with a value of 80, which is more than 75 percent of 100. Then we assert that the list of messages that the "MockMessenger" is keeping track of should now have one message in it.

However, there’s one problem with this test, as shown here:

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow "self.sent_messages" as mutable, as it is behind a "&" reference
  --> src/lib.rs:58:13
   |
2  |     fn send(&self, msg: &str);
   |             ----- help: consider changing that to be a mutable reference: "&mut self"
...
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ "self" is a "&" reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try "rustc --explain E0596".
error: could not compile "limit-tracker" due to previous error
warning: build failed, waiting for other jobs to finish...
```

We can’t modify the "MockMessenger" to keep track of the messages, because the "send" method takes an immutable reference to "self". We also can’t take the suggestion from the error text to use "&mut self" instead, because then the signature of "send" wouldn’t match the signature in the "Messenger" trait definition (feel free to try and see what error message you get).

This is a situation in which interior mutability can help! We’ll store the "sent_messages" within a "RefCell<T>", and then the "send" method will be able to modify "sent_messages" to store the messages we’ve seen. Listing 15-22 shows what that looks like:

Filename: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

Listing 15-22: Using "RefCell<T>" to mutate an inner value while the outer value is considered immutable

The "sent_messages" field is now of type "RefCell<Vec<String>>" instead of "Vec<String>". In the "new" function, we create a new "RefCell<Vec<String>>" instance around the empty vector.

For the implementation of the "send" method, the first parameter is still an immutable borrow of "self", which matches the trait definition. We call "borrow_mut" on the "RefCell<Vec<String>>" in "self.sent_messages" to get a mutable reference to the value inside the "RefCell<Vec<String>>", which is the vector. Then we can call "push" on the mutable reference to the vector to keep track of the messages sent during the test.

The last change we have to make is in the assertion: to see how many items are in the inner vector, we call "borrow" on the "RefCell<Vec<String>>" to get an immutable reference to the vector.

Now that you’ve seen how to use "RefCell<T>", let’s dig into how it works!

#### [Keeping Track of Borrows at Runtime with "RefCell"](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#keeping-track-of-borrows-at-runtime-with-refcellt)

When creating immutable and mutable references, we use the "&" and "&mut" syntax, respectively. With "RefCell<T>", we use the "borrow" and "borrow_mut" methods, which are part of the safe API that belongs to "RefCell<T>". The "borrow" method returns the smart pointer type "Ref<T>", and "borrow_mut" returns the smart pointer type "RefMut<T>". Both types implement "Deref", so we can treat them like regular references.

The "RefCell<T>" keeps track of how many "Ref<T>" and "RefMut<T>" smart pointers are currently active. Every time we call "borrow", the "RefCell<T>" increases its count of how many immutable borrows are active. When a "Ref<T>" value goes out of scope, the count of immutable borrows goes down by one. Just like the compile-time borrowing rules, "RefCell<T>" lets us have many immutable borrows or one mutable borrow at any point in time.

If we try to violate these rules, rather than getting a compiler error as we would with references, the implementation of "RefCell<T>" will panic at runtime. Listing 15-23 shows a modification of the implementation of "send" in Listing 15-22. We’re deliberately trying to create two mutable borrows active for the same scope to illustrate that "RefCell<T>" prevents us from doing this at runtime.

Filename: src/lib.rs

```rust
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            let mut one_borrow = self.sent_messages.borrow_mut();
            let mut two_borrow = self.sent_messages.borrow_mut();

            one_borrow.push(String::from(message));
            two_borrow.push(String::from(message));
        }
    }
```

Listing 15-23: Creating two mutable references in the same scope to see that "RefCell<T>" will panic

We create a variable "one_borrow" for the "RefMut<T>" smart pointer returned from "borrow_mut". Then we create another mutable borrow in the same way in the variable "two_borrow". This makes two mutable references in the same scope, which isn’t allowed. When we run the tests for our library, the code in Listing 15-23 will compile without any errors, but the test will fail:

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/limit_tracker-e599811fa246dbde)

running 1 test
test tests::it_sends_an_over_75_percent_warning_message ... FAILED

failures:

---- tests::it_sends_an_over_75_percent_warning_message stdout ----
thread "tests::it_sends_an_over_75_percent_warning_message" panicked at "already borrowed: BorrowMutError", src/lib.rs:60:53
note: run with "RUST_BACKTRACE=1" environment variable to display a backtrace


failures:
    tests::it_sends_an_over_75_percent_warning_message

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass "--lib"
```

Notice that the code panicked with the message "already borrowed: BorrowMutError". This is how "RefCell<T>" handles violations of the borrowing rules at runtime.

Choosing to catch borrowing errors at runtime rather than compile time, as we’ve done here, means you’d potentially be finding mistakes in your code later in the development process: possibly not until your code was deployed to production. Also, your code would incur a small runtime performance penalty as a result of keeping track of the borrows at runtime rather than compile time. However, using "RefCell<T>" makes it possible to write a mock object that can modify itself to keep track of the messages it has seen while you’re using it in a context where only immutable values are allowed. You can use "RefCell<T>" despite its trade-offs to get more functionality than regular references provide.

### [Having Multiple Owners of Mutable Data by Combining "Rc" and "RefCell"](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt)

A common way to use "RefCell<T>" is in combination with "Rc<T>". Recall that "Rc<T>" lets you have multiple owners of some data, but it only gives immutable access to that data. If you have an "Rc<T>" that holds a "RefCell<T>", you can get a value that can have multiple owners *and* that you can mutate!

For example, recall the cons list example in Listing 15-18 where we used "Rc<T>" to allow multiple lists to share ownership of another list. Because "Rc<T>" holds only immutable values, we can’t change any of the values in the list once we’ve created them. Let’s add in "RefCell<T>" to gain the ability to change the values in the lists. Listing 15-24 shows that by using a "RefCell<T>" in the "Cons" definition, we can modify the value stored in all the lists:

Filename: src/main.rs

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

Listing 15-24: Using "Rc<RefCell<i32>>" to create a "List" that we can mutate

We create a value that is an instance of "Rc<RefCell<i32>>" and store it in a variable named "value" so we can access it directly later. Then we create a "List" in "a" with a "Cons" variant that holds "value". We need to clone "value" so both "a" and "value" have ownership of the inner "5" value rather than transferring ownership from "value" to "a" or having "a" borrow from "value".

We wrap the list "a" in an "Rc<T>" so when we create lists "b" and "c", they can both refer to "a", which is what we did in Listing 15-18.

After we’ve created the lists in "a", "b", and "c", we want to add 10 to the value in "value". We do this by calling "borrow_mut" on "value", which uses the automatic dereferencing feature we discussed in Chapter 5 (see the section [“Where’s the "->" Operator?”](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator)) to dereference the "Rc<T>" to the inner "RefCell<T>" value. The "borrow_mut" method returns a "RefMut<T>" smart pointer, and we use the dereference operator on it and change the inner value.

When we print "a", "b", and "c", we can see that they all have the modified value of 15 rather than 5:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.63s
     Running "target/debug/cons-list"
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
```

This technique is pretty neat! By using "RefCell<T>", we have an outwardly immutable "List" value. But we can use the methods on "RefCell<T>" that provide access to its interior mutability so we can modify our data when we need to. The runtime checks of the borrowing rules protect us from data races, and it’s sometimes worth trading a bit of speed for this flexibility in our data structures. Note that "RefCell<T>" does not work for multithreaded code! "Mutex<T>" is the thread-safe version of "RefCell<T>" and we’ll discuss "Mutex<T>" in Chapter 16.





---





## [Reference Cycles Can Leak Memory](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#reference-cycles-can-leak-memory)

Rust’s memory safety guarantees make it difficult, but not impossible, to accidentally create memory that is never cleaned up (known as a *memory leak*). Preventing memory leaks entirely is not one of Rust’s guarantees, meaning memory leaks are memory safe in Rust. We can see that Rust allows memory leaks by using "Rc<T>" and "RefCell<T>": it’s possible to create references where items refer to each other in a cycle. This creates memory leaks because the reference count of each item in the cycle will never reach 0, and the values will never be dropped.

### [Creating a Reference Cycle](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#creating-a-reference-cycle)

Let’s look at how a reference cycle might happen and how to prevent it, starting with the definition of the "List" enum and a "tail" method in Listing 15-25:

Filename: src/main.rs

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

Listing 15-25: A cons list definition that holds a "RefCell<T>" so we can modify what a "Cons" variant is referring to

We’re using another variation of the "List" definition from Listing 15-5. The second element in the "Cons" variant is now "RefCell<Rc<List>>", meaning that instead of having the ability to modify the "i32" value as we did in Listing 15-24, we want to modify the "List" value a "Cons" variant is pointing to. We’re also adding a "tail" method to make it convenient for us to access the second item if we have a "Cons" variant.

In Listing 15-26, we’re adding a "main" function that uses the definitions in Listing 15-25. This code creates a list in "a" and a list in "b" that points to the list in "a". Then it modifies the list in "a" to point to "b", creating a reference cycle. There are "println!" statements along the way to show what the reference counts are at various points in this process.

Filename: src/main.rs

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

Listing 15-26: Creating a reference cycle of two "List" values pointing to each other

We create an "Rc<List>" instance holding a "List" value in the variable "a" with an initial list of "5, Nil". We then create an "Rc<List>" instance holding another "List" value in the variable "b" that contains the value 10 and points to the list in "a".

We modify "a" so it points to "b" instead of "Nil", creating a cycle. We do that by using the "tail" method to get a reference to the "RefCell<Rc<List>>" in "a", which we put in the variable "link". Then we use the "borrow_mut" method on the "RefCell<Rc<List>>" to change the value inside from an "Rc<List>" that holds a "Nil" value to the "Rc<List>" in "b".

When we run this code, keeping the last "println!" commented out for the moment, we’ll get this output:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running "target/debug/cons-list"
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

The reference count of the "Rc<List>" instances in both "a" and "b" are 2 after we change the list in "a" to point to "b". At the end of "main", Rust drops the variable "b", which decreases the reference count of the "b" "Rc<List>" instance from 2 to 1. The memory that "Rc<List>" has on the heap won’t be dropped at this point, because its reference count is 1, not 0. Then Rust drops "a", which decreases the reference count of the "a" "Rc<List>" instance from 2 to 1 as well. This instance’s memory can’t be dropped either, because the other "Rc<List>" instance still refers to it. The memory allocated to the list will remain uncollected forever. To visualize this reference cycle, we’ve created a diagram in Figure 15-4.

![Reference cycle of lists](https://doc.rust-lang.org/book/img/trpl15-04.svg)

Figure 15-4: A reference cycle of lists "a" and "b" pointing to each other

If you uncomment the last "println!" and run the program, Rust will try to print this cycle with "a" pointing to "b" pointing to "a" and so forth until it overflows the stack.

Compared to a real-world program, the consequences creating a reference cycle in this example aren’t very dire: right after we create the reference cycle, the program ends. However, if a more complex program allocated lots of memory in a cycle and held onto it for a long time, the program would use more memory than it needed and might overwhelm the system, causing it to run out of available memory.

Creating reference cycles is not easily done, but it’s not impossible either. If you have "RefCell<T>" values that contain "Rc<T>" values or similar nested combinations of types with interior mutability and reference counting, you must ensure that you don’t create cycles; you can’t rely on Rust to catch them. Creating a reference cycle would be a logic bug in your program that you should use automated tests, code reviews, and other software development practices to minimize.

Another solution for avoiding reference cycles is reorganizing your data structures so that some references express ownership and some references don’t. As a result, you can have cycles made up of some ownership relationships and some non-ownership relationships, and only the ownership relationships affect whether or not a value can be dropped. In Listing 15-25, we always want "Cons" variants to own their list, so reorganizing the data structure isn’t possible. Let’s look at an example using graphs made up of parent nodes and child nodes to see when non-ownership relationships are an appropriate way to prevent reference cycles.

### [Preventing Reference Cycles: Turning an "Rc" into a "Weak"](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#preventing-reference-cycles-turning-an-rct-into-a-weakt)

So far, we’ve demonstrated that calling "Rc::clone" increases the "strong_count" of an "Rc<T>" instance, and an "Rc<T>" instance is only cleaned up if its "strong_count" is 0. You can also create a *weak reference* to the value within an "Rc<T>" instance by calling "Rc::downgrade" and passing a reference to the "Rc<T>". Strong references are how you can share ownership of an "Rc<T>" instance. Weak references don’t express an ownership relationship, and their count doesn’t affect when an "Rc<T>" instance is cleaned up. They won’t cause a reference cycle because any cycle involving some weak references will be broken once the strong reference count of values involved is 0.

When you call "Rc::downgrade", you get a smart pointer of type "Weak<T>". Instead of increasing the "strong_count" in the "Rc<T>" instance by 1, calling "Rc::downgrade" increases the "weak_count" by 1. The "Rc<T>" type uses "weak_count" to keep track of how many "Weak<T>" references exist, similar to "strong_count". The difference is the "weak_count" doesn’t need to be 0 for the "Rc<T>" instance to be cleaned up.

Because the value that "Weak<T>" references might have been dropped, to do anything with the value that a "Weak<T>" is pointing to, you must make sure the value still exists. Do this by calling the "upgrade" method on a "Weak<T>" instance, which will return an "Option<Rc<T>>". You’ll get a result of "Some" if the "Rc<T>" value has not been dropped yet and a result of "None" if the "Rc<T>" value has been dropped. Because "upgrade" returns an "Option<Rc<T>>", Rust will ensure that the "Some" case and the "None" case are handled, and there won’t be an invalid pointer.

As an example, rather than using a list whose items know only about the next item, we’ll create a tree whose items know about their children items *and* their parent items.

#### [Creating a Tree Data Structure: a "Node" with Child Nodes](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#creating-a-tree-data-structure-a-node-with-child-nodes)

To start, we’ll build a tree with nodes that know about their child nodes. We’ll create a struct named "Node" that holds its own "i32" value as well as references to its children "Node" values:

Filename: src/main.rs

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

We want a "Node" to own its children, and we want to share that ownership with variables so we can access each "Node" in the tree directly. To do this, we define the "Vec<T>" items to be values of type "Rc<Node>". We also want to modify which nodes are children of another node, so we have a "RefCell<T>" in "children" around the "Vec<Rc<Node>>".

Next, we’ll use our struct definition and create one "Node" instance named "leaf" with the value 3 and no children, and another instance named "branch" with the value 5 and "leaf" as one of its children, as shown in Listing 15-27:

Filename: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

Listing 15-27: Creating a "leaf" node with no children and a "branch" node with "leaf" as one of its children

We clone the "Rc<Node>" in "leaf" and store that in "branch", meaning the "Node" in "leaf" now has two owners: "leaf" and "branch". We can get from "branch" to "leaf" through "branch.children", but there’s no way to get from "leaf" to "branch". The reason is that "leaf" has no reference to "branch" and doesn’t know they’re related. We want "leaf" to know that "branch" is its parent. We’ll do that next.

#### [Adding a Reference from a Child to Its Parent](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#adding-a-reference-from-a-child-to-its-parent)

To make the child node aware of its parent, we need to add a "parent" field to our "Node" struct definition. The trouble is in deciding what the type of "parent" should be. We know it can’t contain an "Rc<T>", because that would create a reference cycle with "leaf.parent" pointing to "branch" and "branch.children" pointing to "leaf", which would cause their "strong_count" values to never be 0.

Thinking about the relationships another way, a parent node should own its children: if a parent node is dropped, its child nodes should be dropped as well. However, a child should not own its parent: if we drop a child node, the parent should still exist. This is a case for weak references!

So instead of "Rc<T>", we’ll make the type of "parent" use "Weak<T>", specifically a "RefCell<Weak<Node>>". Now our "Node" struct definition looks like this:

Filename: src/main.rs

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

A node will be able to refer to its parent node but doesn’t own its parent. In Listing 15-28, we update "main" to use this new definition so the "leaf" node will have a way to refer to its parent, "branch":

Filename: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

Listing 15-28: A "leaf" node with a weak reference to its parent node "branch"

Creating the "leaf" node looks similar to Listing 15-27 with the exception of the "parent" field: "leaf" starts out without a parent, so we create a new, empty "Weak<Node>" reference instance.

At this point, when we try to get a reference to the parent of "leaf" by using the "upgrade" method, we get a "None" value. We see this in the output from the first "println!" statement:

```
leaf parent = None
```

When we create the "branch" node, it will also have a new "Weak<Node>" reference in the "parent" field, because "branch" doesn’t have a parent node. We still have "leaf" as one of the children of "branch". Once we have the "Node" instance in "branch", we can modify "leaf" to give it a "Weak<Node>" reference to its parent. We use the "borrow_mut" method on the "RefCell<Weak<Node>>" in the "parent" field of "leaf", and then we use the "Rc::downgrade" function to create a "Weak<Node>" reference to "branch" from the "Rc<Node>" in "branch."

When we print the parent of "leaf" again, this time we’ll get a "Some" variant holding "branch": now "leaf" can access its parent! When we print "leaf", we also avoid the cycle that eventually ended in a stack overflow like we had in Listing 15-26; the "Weak<Node>" references are printed as "(Weak)":

```
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

The lack of infinite output indicates that this code didn’t create a reference cycle. We can also tell this by looking at the values we get from calling "Rc::strong_count" and "Rc::weak_count".

#### [Visualizing Changes to "strong_count" and "weak_count"](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#visualizing-changes-to-strong_count-and-weak_count)

Let’s look at how the "strong_count" and "weak_count" values of the "Rc<Node>" instances change by creating a new inner scope and moving the creation of "branch" into that scope. By doing so, we can see what happens when "branch" is created and then dropped when it goes out of scope. The modifications are shown in Listing 15-29:

Filename: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

Listing 15-29: Creating "branch" in an inner scope and examining strong and weak reference counts

After "leaf" is created, its "Rc<Node>" has a strong count of 1 and a weak count of 0. In the inner scope, we create "branch" and associate it with "leaf", at which point when we print the counts, the "Rc<Node>" in "branch" will have a strong count of 1 and a weak count of 1 (for "leaf.parent" pointing to "branch" with a "Weak<Node>"). When we print the counts in "leaf", we’ll see it will have a strong count of 2, because "branch" now has a clone of the "Rc<Node>" of "leaf" stored in "branch.children", but will still have a weak count of 0.

When the inner scope ends, "branch" goes out of scope and the strong count of the "Rc<Node>" decreases to 0, so its "Node" is dropped. The weak count of 1 from "leaf.parent" has no bearing on whether or not "Node" is dropped, so we don’t get any memory leaks!

If we try to access the parent of "leaf" after the end of the scope, we’ll get "None" again. At the end of the program, the "Rc<Node>" in "leaf" has a strong count of 1 and a weak count of 0, because the variable "leaf" is now the only reference to the "Rc<Node>" again.

All of the logic that manages the counts and value dropping is built into "Rc<T>" and "Weak<T>" and their implementations of the "Drop" trait. By specifying that the relationship from a child to its parent should be a "Weak<T>" reference in the definition of "Node", you’re able to have parent nodes point to child nodes and vice versa without creating a reference cycle and memory leaks.

## [Summary](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#summary)

This chapter covered how to use smart pointers to make different guarantees and trade-offs from those Rust makes by default with regular references. The "Box<T>" type has a known size and points to data allocated on the heap. The "Rc<T>" type keeps track of the number of references to data on the heap so that data can have multiple owners. The "RefCell<T>" type with its interior mutability gives us a type that we can use when we need an immutable type but need to change an inner value of that type; it also enforces the borrowing rules at runtime instead of at compile time.

Also discussed were the "Deref" and "Drop" traits, which enable a lot of the functionality of smart pointers. We explored reference cycles that can cause memory leaks and how to prevent them using "Weak<T>".

If this chapter has piqued your interest and you want to implement your own smart pointers, check out [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/index.html) for more useful information.

Next, we’ll talk about concurrency in Rust. You’ll even learn about a few new smart pointers.





---





# 16





# [Fearless Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html#fearless-concurrency)

Handling concurrent programming safely and efficiently is another of Rust’s major goals. *Concurrent programming*, where different parts of a program execute independently, and *parallel programming*, where different parts of a program execute at the same time, are becoming increasingly important as more computers take advantage of their multiple processors. Historically, programming in these contexts has been difficult and error prone: Rust hopes to change that.

Initially, the Rust team thought that ensuring memory safety and preventing concurrency problems were two separate challenges to be solved with different methods. Over time, the team discovered that the ownership and type systems are a powerful set of tools to help manage memory safety *and* concurrency problems! By leveraging ownership and type checking, many concurrency errors are compile-time errors in Rust rather than runtime errors. Therefore, rather than making you spend lots of time trying to reproduce the exact circumstances under which a runtime concurrency bug occurs, incorrect code will refuse to compile and present an error explaining the problem. As a result, you can fix your code while you’re working on it rather than potentially after it has been shipped to production. We’ve nicknamed this aspect of Rust *fearless* *concurrency*. Fearless concurrency allows you to write code that is free of subtle bugs and is easy to refactor without introducing new bugs.

> Note: For simplicity’s sake, we’ll refer to many of the problems as *concurrent* rather than being more precise by saying *concurrent and/or parallel*. If this book were about concurrency and/or parallelism, we’d be more specific. For this chapter, please mentally substitute *concurrent and/or parallel* whenever we use *concurrent*.

Many languages are dogmatic about the solutions they offer for handling concurrent problems. For example, Erlang has elegant functionality for message-passing concurrency but has only obscure ways to share state between threads. Supporting only a subset of possible solutions is a reasonable strategy for higher-level languages, because a higher-level language promises benefits from giving up some control to gain abstractions. However, lower-level languages are expected to provide the solution with the best performance in any given situation and have fewer abstractions over the hardware. Therefore, Rust offers a variety of tools for modeling problems in whatever way is appropriate for your situation and requirements.

Here are the topics we’ll cover in this chapter:

- How to create threads to run multiple pieces of code at the same time
- *Message-passing* concurrency, where channels send messages between threads
- *Shared-state* concurrency, where multiple threads have access to some piece of data
- The "Sync" and "Send" traits, which extend Rust’s concurrency guarantees to user-defined types as well as types provided by the standard library



---





## [Using Threads to Run Code Simultaneously](https://doc.rust-lang.org/book/ch16-01-threads.html#using-threads-to-run-code-simultaneously)

In most current operating systems, an executed program’s code is run in a *process*, and the operating system will manage multiple processes at once. Within a program, you can also have independent parts that run simultaneously. The features that run these independent parts are called *threads*. For example, a web server could have multiple threads so that it could respond to more than one request at the same time.

Splitting the computation in your program into multiple threads to run multiple tasks at the same time can improve performance, but it also adds complexity. Because threads can run simultaneously, there’s no inherent guarantee about the order in which parts of your code on different threads will run. This can lead to problems, such as:

- Race conditions, where threads are accessing data or resources in an inconsistent order
- Deadlocks, where two threads are waiting for each other, preventing both threads from continuing
- Bugs that happen only in certain situations and are hard to reproduce and fix reliably

Rust attempts to mitigate the negative effects of using threads, but programming in a multithreaded context still takes careful thought and requires a code structure that is different from that in programs running in a single thread.

Programming languages implement threads in a few different ways, and many operating systems provide an API the language can call for creating new threads. The Rust standard library uses a *1:1* model of thread implementation, whereby a program uses one operating system thread per one language thread. There are crates that implement other models of threading that make different tradeoffs to the 1:1 model.

### [Creating a New Thread with "spawn"](https://doc.rust-lang.org/book/ch16-01-threads.html#creating-a-new-thread-with-spawn)

To create a new thread, we call the "thread::spawn" function and pass it a closure (we talked about closures in Chapter 13) containing the code we want to run in the new thread. The example in Listing 16-1 prints some text from a main thread and other text from a new thread:

Filename: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

Listing 16-1: Creating a new thread to print one thing while the main thread prints something else

Note that when the main thread of a Rust program completes, all spawned threads are shut down, whether or not they have finished running. The output from this program might be a little different every time, but it will look similar to the following:

```
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

The calls to "thread::sleep" force a thread to stop its execution for a short duration, allowing a different thread to run. The threads will probably take turns, but that isn’t guaranteed: it depends on how your operating system schedules the threads. In this run, the main thread printed first, even though the print statement from the spawned thread appears first in the code. And even though we told the spawned thread to print until "i" is 9, it only got to 5 before the main thread shut down.

If you run this code and only see output from the main thread, or don’t see any overlap, try increasing the numbers in the ranges to create more opportunities for the operating system to switch between the threads.

### [Waiting for All Threads to Finish Using "join" Handles](https://doc.rust-lang.org/book/ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles)

The code in Listing 16-1 not only stops the spawned thread prematurely most of the time due to the main thread ending, but because there is no guarantee on the order in which threads run, we also can’t guarantee that the spawned thread will get to run at all!

We can fix the problem of the spawned thread not running or ending prematurely by saving the return value of "thread::spawn" in a variable. The return type of "thread::spawn" is "JoinHandle". A "JoinHandle" is an owned value that, when we call the "join" method on it, will wait for its thread to finish. Listing 16-2 shows how to use the "JoinHandle" of the thread we created in Listing 16-1 and call "join" to make sure the spawned thread finishes before "main" exits:

Filename: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

Listing 16-2: Saving a "JoinHandle" from "thread::spawn" to guarantee the thread is run to completion

Calling "join" on the handle blocks the thread currently running until the thread represented by the handle terminates. *Blocking* a thread means that thread is prevented from performing work or exiting. Because we’ve put the call to "join" after the main thread’s "for" loop, running Listing 16-2 should produce output similar to this:

```
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

The two threads continue alternating, but the main thread waits because of the call to "handle.join()" and does not end until the spawned thread is finished.

But let’s see what happens when we instead move "handle.join()" before the "for" loop in "main", like this:

Filename: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

The main thread will wait for the spawned thread to finish and then run its "for" loop, so the output won’t be interleaved anymore, as shown here:

```
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Small details, such as where "join" is called, can affect whether or not your threads run at the same time.

### [Using "move" Closures with Threads](https://doc.rust-lang.org/book/ch16-01-threads.html#using-move-closures-with-threads)

We"ll often use the "move" keyword with closures passed to "thread::spawn" because the closure will then take ownership of the values it uses from the environment, thus transferring ownership of those values from one thread to another. In the [“Capturing References or Moving Ownership”](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership) section of Chapter 13, we discussed "move" in the context of closures. Now, we’ll concentrate more on the interaction between "move" and "thread::spawn".

Notice in Listing 16-1 that the closure we pass to "thread::spawn" takes no arguments: we’re not using any data from the main thread in the spawned thread’s code. To use data from the main thread in the spawned thread, the spawned thread’s closure must capture the values it needs. Listing 16-3 shows an attempt to create a vector in the main thread and use it in the spawned thread. However, this won’t yet work, as you’ll see in a moment.

Filename: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here"s a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Listing 16-3: Attempting to use a vector created by the main thread in another thread

The closure uses "v", so it will capture "v" and make it part of the closure’s environment. Because "thread::spawn" runs this closure in a new thread, we should be able to access "v" inside that new thread. But when we compile this example, we get the following error:

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0373]: closure may outlive the current function, but it borrows "v", which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value "v"
7 |         println!("Here"s a vector: {:?}", v);
  |                                           - "v" is borrowed here
  |
note: function requires argument type to outlive ""static"
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here"s a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of "v" (and any other referenced variables), use the "move" keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try "rustc --explain E0373".
error: could not compile "threads" due to previous error
```

Rust *infers* how to capture "v", and because "println!" only needs a reference to "v", the closure tries to borrow "v". However, there’s a problem: Rust can’t tell how long the spawned thread will run, so it doesn’t know if the reference to "v" will always be valid.

Listing 16-4 provides a scenario that’s more likely to have a reference to "v" that won’t be valid:

Filename: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here"s a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

Listing 16-4: A thread with a closure that attempts to capture a reference to "v" from a main thread that drops "v"

If Rust allowed us to run this code, there’s a possibility the spawned thread would be immediately put in the background without running at all. The spawned thread has a reference to "v" inside, but the main thread immediately drops "v", using the "drop" function we discussed in Chapter 15. Then, when the spawned thread starts to execute, "v" is no longer valid, so a reference to it is also invalid. Oh no!

To fix the compiler error in Listing 16-3, we can use the error message’s advice:

```
help: to force the closure to take ownership of "v" (and any other referenced variables), use the "move" keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

By adding the "move" keyword before the closure, we force the closure to take ownership of the values it’s using rather than allowing Rust to infer that it should borrow the values. The modification to Listing 16-3 shown in Listing 16-5 will compile and run as we intend:

Filename: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here"s a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Listing 16-5: Using the "move" keyword to force a closure to take ownership of the values it uses

We might be tempted to try the same thing to fix the code in Listing 16-4 where the main thread called "drop" by using a "move" closure. However, this fix will not work because what Listing 16-4 is trying to do is disallowed for a different reason. If we added "move" to the closure, we would move "v" into the closure’s environment, and we could no longer call "drop" on it in the main thread. We would get this compiler error instead:

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: "v"
  --> src/main.rs:10:10
   |
4  |     let v = vec![1, 2, 3];
   |         - move occurs because "v" has type "Vec<i32>", which does not implement the "Copy" trait
5  |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
7  |         println!("Here"s a vector: {:?}", v);
   |                                           - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move

For more information about this error, try "rustc --explain E0382".
error: could not compile "threads" due to previous error
```

Rust’s ownership rules have saved us again! We got an error from the code in Listing 16-3 because Rust was being conservative and only borrowing "v" for the thread, which meant the main thread could theoretically invalidate the spawned thread’s reference. By telling Rust to move ownership of "v" to the spawned thread, we’re guaranteeing Rust that the main thread won’t use "v" anymore. If we change Listing 16-4 in the same way, we’re then violating the ownership rules when we try to use "v" in the main thread. The "move" keyword overrides Rust’s conservative default of borrowing; it doesn’t let us violate the ownership rules.

With a basic understanding of threads and the thread API, let’s look at what we can *do* with threads.





---





## [Using Message Passing to Transfer Data Between Threads](https://doc.rust-lang.org/book/ch16-02-message-passing.html#using-message-passing-to-transfer-data-between-threads)

One increasingly popular approach to ensuring safe concurrency is *message passing*, where threads or actors communicate by sending each other messages containing data. Here’s the idea in a slogan from [the Go language documentation](https://golang.org/doc/effective_go.html#concurrency): “Do not communicate by sharing memory; instead, share memory by communicating.”

To accomplish message-sending concurrency, Rust"s standard library provides an implementation of *channels*. A channel is a general programming concept by which data is sent from one thread to another.

You can imagine a channel in programming as being like a directional channel of water, such as a stream or a river. If you put something like a rubber duck into a river, it will travel downstream to the end of the waterway.

A channel has two halves: a transmitter and a receiver. The transmitter half is the upstream location where you put rubber ducks into the river, and the receiver half is where the rubber duck ends up downstream. One part of your code calls methods on the transmitter with the data you want to send, and another part checks the receiving end for arriving messages. A channel is said to be *closed* if either the transmitter or receiver half is dropped.

Here, we’ll work up to a program that has one thread to generate values and send them down a channel, and another thread that will receive the values and print them out. We’ll be sending simple values between threads using a channel to illustrate the feature. Once you’re familiar with the technique, you could use channels for any threads that need to communicate between each other, such as a chat system or a system where many threads perform parts of a calculation and send the parts to one thread that aggregates the results.

First, in Listing 16-6, we’ll create a channel but not do anything with it. Note that this won’t compile yet because Rust can’t tell what type of values we want to send over the channel.

Filename: src/main.rs

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

Listing 16-6: Creating a channel and assigning the two halves to "tx" and "rx"

We create a new channel using the "mpsc::channel" function; "mpsc" stands for *multiple producer, single consumer*. In short, the way Rust’s standard library implements channels means a channel can have multiple *sending* ends that produce values but only one *receiving* end that consumes those values. Imagine multiple streams flowing together into one big river: everything sent down any of the streams will end up in one river at the end. We’ll start with a single producer for now, but we’ll add multiple producers when we get this example working.

The "mpsc::channel" function returns a tuple, the first element of which is the sending end--the transmitter--and the second element is the receiving end--the receiver. The abbreviations "tx" and "rx" are traditionally used in many fields for *transmitter* and *receiver* respectively, so we name our variables as such to indicate each end. We’re using a "let" statement with a pattern that destructures the tuples; we’ll discuss the use of patterns in "let" statements and destructuring in Chapter 18. For now, know that using a "let" statement this way is a convenient approach to extract the pieces of the tuple returned by "mpsc::channel".

Let’s move the transmitting end into a spawned thread and have it send one string so the spawned thread is communicating with the main thread, as shown in Listing 16-7. This is like putting a rubber duck in the river upstream or sending a chat message from one thread to another.

Filename: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

Listing 16-7: Moving "tx" to a spawned thread and sending “hi”

Again, we’re using "thread::spawn" to create a new thread and then using "move" to move "tx" into the closure so the spawned thread owns "tx". The spawned thread needs to own the transmitter to be able to send messages through the channel. The transmitter has a "send" method that takes the value we want to send. The "send" method returns a "Result<T, E>" type, so if the receiver has already been dropped and there’s nowhere to send a value, the send operation will return an error. In this example, we’re calling "unwrap" to panic in case of an error. But in a real application, we would handle it properly: return to Chapter 9 to review strategies for proper error handling.

In Listing 16-8, we’ll get the value from the receiver in the main thread. This is like retrieving the rubber duck from the water at the end of the river or receiving a chat message.

Filename: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

Listing 16-8: Receiving the value “hi” in the main thread and printing it

The receiver has two useful methods: "recv" and "try_recv". We’re using "recv", short for *receive*, which will block the main thread’s execution and wait until a value is sent down the channel. Once a value is sent, "recv" will return it in a "Result<T, E>". When the transmitter closes, "recv" will return an error to signal that no more values will be coming.

The "try_recv" method doesn’t block, but will instead return a "Result<T, E>" immediately: an "Ok" value holding a message if one is available and an "Err" value if there aren’t any messages this time. Using "try_recv" is useful if this thread has other work to do while waiting for messages: we could write a loop that calls "try_recv" every so often, handles a message if one is available, and otherwise does other work for a little while until checking again.

We’ve used "recv" in this example for simplicity; we don’t have any other work for the main thread to do other than wait for messages, so blocking the main thread is appropriate.

When we run the code in Listing 16-8, we’ll see the value printed from the main thread:

```
Got: hi
```

Perfect!

### [Channels and Ownership Transference](https://doc.rust-lang.org/book/ch16-02-message-passing.html#channels-and-ownership-transference)

The ownership rules play a vital role in message sending because they help you write safe, concurrent code. Preventing errors in concurrent programming is the advantage of thinking about ownership throughout your Rust programs. Let’s do an experiment to show how channels and ownership work together to prevent problems: we’ll try to use a "val" value in the spawned thread *after* we’ve sent it down the channel. Try compiling the code in Listing 16-9 to see why this code isn’t allowed:

Filename: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

Listing 16-9: Attempting to use "val" after we’ve sent it down the channel

Here, we try to print "val" after we’ve sent it down the channel via "tx.send". Allowing this would be a bad idea: once the value has been sent to another thread, that thread could modify or drop it before we try to use the value again. Potentially, the other thread’s modifications could cause errors or unexpected results due to inconsistent or nonexistent data. However, Rust gives us an error if we try to compile the code in Listing 16-9:

```bash
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: "val"
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because "val" has type "String", which does not implement the "Copy" trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro "$crate::format_args_nl" which comes from the expansion of the macro "println" (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try "rustc --explain E0382".
error: could not compile "message-passing" due to previous error
```

Our concurrency mistake has caused a compile time error. The "send" function takes ownership of its parameter, and when the value is moved, the receiver takes ownership of it. This stops us from accidentally using the value again after sending it; the ownership system checks that everything is okay.

### [Sending Multiple Values and Seeing the Receiver Waiting](https://doc.rust-lang.org/book/ch16-02-message-passing.html#sending-multiple-values-and-seeing-the-receiver-waiting)

The code in Listing 16-8 compiled and ran, but it didn’t clearly show us that two separate threads were talking to each other over the channel. In Listing 16-10 we’ve made some modifications that will prove the code in Listing 16-8 is running concurrently: the spawned thread will now send multiple messages and pause for a second between each message.

Filename: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

Listing 16-10: Sending multiple messages and pausing between each

This time, the spawned thread has a vector of strings that we want to send to the main thread. We iterate over them, sending each individually, and pause between each by calling the "thread::sleep" function with a "Duration" value of 1 second.

In the main thread, we’re not calling the "recv" function explicitly anymore: instead, we’re treating "rx" as an iterator. For each value received, we’re printing it. When the channel is closed, iteration will end.

When running the code in Listing 16-10, you should see the following output with a 1-second pause in between each line:

```
Got: hi
Got: from
Got: the
Got: thread
```

Because we don’t have any code that pauses or delays in the "for" loop in the main thread, we can tell that the main thread is waiting to receive values from the spawned thread.

### [Creating Multiple Producers by Cloning the Transmitter](https://doc.rust-lang.org/book/ch16-02-message-passing.html#creating-multiple-producers-by-cloning-the-transmitter)

Earlier we mentioned that "mpsc" was an acronym for *multiple producer, single consumer*. Let’s put "mpsc" to use and expand the code in Listing 16-10 to create multiple threads that all send values to the same receiver. We can do so by cloning the transmitter, as shown in Listing 16-11:

Filename: src/main.rs

```rust
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
```

Listing 16-11: Sending multiple messages from multiple producers

This time, before we create the first spawned thread, we call "clone" on the transmitter. This will give us a new transmitter we can pass to the first spawned thread. We pass the original transmitter to a second spawned thread. This gives us two threads, each sending different messages to the one receiver.

When you run the code, your output should look something like this:

```
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

You might see the values in another order, depending on your system. This is what makes concurrency interesting as well as difficult. If you experiment with "thread::sleep", giving it various values in the different threads, each run will be more nondeterministic and create different output each time.

Now that we’ve looked at how channels work, let’s look at a different method of concurrency.





---





## [Shared-State Concurrency](https://doc.rust-lang.org/book/ch16-03-shared-state.html#shared-state-concurrency)

Message passing is a fine way of handling concurrency, but it’s not the only one. Another method would be for multiple threads to access the same shared data. Consider this part of the slogan from the Go language documentation again: “do not communicate by sharing memory.”

What would communicating by sharing memory look like? In addition, why would message-passing enthusiasts caution not to use memory sharing?

In a way, channels in any programming language are similar to single ownership, because once you transfer a value down a channel, you should no longer use that value. Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time. As you saw in Chapter 15, where smart pointers made multiple ownership possible, multiple ownership can add complexity because these different owners need managing. Rust’s type system and ownership rules greatly assist in getting this management correct. For an example, let’s look at mutexes, one of the more common concurrency primitives for shared memory.

### [Using Mutexes to Allow Access to Data from One Thread at a Time](https://doc.rust-lang.org/book/ch16-03-shared-state.html#using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time)

*Mutex* is an abbreviation for *mutual exclusion*, as in, a mutex allows only one thread to access some data at any given time. To access the data in a mutex, a thread must first signal that it wants access by asking to acquire the mutex’s *lock*. The lock is a data structure that is part of the mutex that keeps track of who currently has exclusive access to the data. Therefore, the mutex is described as *guarding* the data it holds via the locking system.

Mutexes have a reputation for being difficult to use because you have to remember two rules:

- You must attempt to acquire the lock before using the data.
- When you’re done with the data that the mutex guards, you must unlock the data so other threads can acquire the lock.

For a real-world metaphor for a mutex, imagine a panel discussion at a conference with only one microphone. Before a panelist can speak, they have to ask or signal that they want to use the microphone. When they get the microphone, they can talk for as long as they want to and then hand the microphone to the next panelist who requests to speak. If a panelist forgets to hand the microphone off when they’re finished with it, no one else is able to speak. If management of the shared microphone goes wrong, the panel won’t work as planned!

Management of mutexes can be incredibly tricky to get right, which is why so many people are enthusiastic about channels. However, thanks to Rust’s type system and ownership rules, you can’t get locking and unlocking wrong.

#### [The API of "Mutex"](https://doc.rust-lang.org/book/ch16-03-shared-state.html#the-api-of-mutext)

As an example of how to use a mutex, let’s start by using a mutex in a single-threaded context, as shown in Listing 16-12:

Filename: src/main.rs

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

Listing 16-12: Exploring the API of "Mutex<T>" in a single-threaded context for simplicity

As with many types, we create a "Mutex<T>" using the associated function "new". To access the data inside the mutex, we use the "lock" method to acquire the lock. This call will block the current thread so it can’t do any work until it’s our turn to have the lock.

The call to "lock" would fail if another thread holding the lock panicked. In that case, no one would ever be able to get the lock, so we’ve chosen to "unwrap" and have this thread panic if we’re in that situation.

After we’ve acquired the lock, we can treat the return value, named "num" in this case, as a mutable reference to the data inside. The type system ensures that we acquire a lock before using the value in "m". The type of "m" is "Mutex<i32>", not "i32", so we *must* call "lock" to be able to use the "i32" value. We can’t forget; the type system won’t let us access the inner "i32" otherwise.

As you might suspect, "Mutex<T>" is a smart pointer. More accurately, the call to "lock" *returns* a smart pointer called "MutexGuard", wrapped in a "LockResult" that we handled with the call to "unwrap". The "MutexGuard" smart pointer implements "Deref" to point at our inner data; the smart pointer also has a "Drop" implementation that releases the lock automatically when a "MutexGuard" goes out of scope, which happens at the end of the inner scope. As a result, we don’t risk forgetting to release the lock and blocking the mutex from being used by other threads, because the lock release happens automatically.

After dropping the lock, we can print the mutex value and see that we were able to change the inner "i32" to 6.

#### [Sharing a "Mutex" Between Multiple Threads](https://doc.rust-lang.org/book/ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads)

Now, let’s try to share a value between multiple threads using "Mutex<T>". We’ll spin up 10 threads and have them each increment a counter value by 1, so the counter goes from 0 to 10. The next example in Listing 16-13 will have a compiler error, and we’ll use that error to learn more about using "Mutex<T>" and how Rust helps us use it correctly.

Filename: src/main.rs

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Listing 16-13: Ten threads each increment a counter guarded by a "Mutex<T>"

We create a "counter" variable to hold an "i32" inside a "Mutex<T>", as we did in Listing 16-12. Next, we create 10 threads by iterating over a range of numbers. We use "thread::spawn" and give all the threads the same closure: one that moves the counter into the thread, acquires a lock on the "Mutex<T>" by calling the "lock" method, and then adds 1 to the value in the mutex. When a thread finishes running its closure, "num" will go out of scope and release the lock so another thread can acquire it.

In the main thread, we collect all the join handles. Then, as we did in Listing 16-2, we call "join" on each handle to make sure all the threads finish. At that point, the main thread will acquire the lock and print the result of this program.

We hinted that this example wouldn’t compile. Now let’s find out why!

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: "counter"
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because "counter" has type "Mutex<i32>", which does not implement the "Copy" trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try "rustc --explain E0382".
error: could not compile "shared-state" due to previous error
```

The error message states that the "counter" value was moved in the previous iteration of the loop. Rust is telling us that we can’t move the ownership of lock "counter" into multiple threads. Let’s fix the compiler error with a multiple-ownership method we discussed in Chapter 15.

#### [Multiple Ownership with Multiple Threads](https://doc.rust-lang.org/book/ch16-03-shared-state.html#multiple-ownership-with-multiple-threads)

In Chapter 15, we gave a value multiple owners by using the smart pointer "Rc<T>" to create a reference counted value. Let’s do the same here and see what happens. We’ll wrap the "Mutex<T>" in "Rc<T>" in Listing 16-14 and clone the "Rc<T>" before moving ownership to the thread.

Filename: src/main.rs

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Listing 16-14: Attempting to use "Rc<T>" to allow multiple threads to own the "Mutex<T>"

Once again, we compile and get... different errors! The compiler is teaching us a lot.

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: "Rc<Mutex<i32>>" cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this "[closure@src/main.rs:11:36: 11:43]"
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ "Rc<Mutex<i32>>" cannot be sent between threads safely
   |
   = help: within "[closure@src/main.rs:11:36: 11:43]", the trait "Send" is not implemented for "Rc<Mutex<i32>>"
note: required because it"s used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in "spawn"
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:704:8
   |
   = note: required by this bound in "spawn"

For more information about this error, try "rustc --explain E0277".
error: could not compile "shared-state" due to previous error
```

Wow, that error message is very wordy! Here’s the important part to focus on: ""Rc<Mutex<i32>>" cannot be sent between threads safely". The compiler is also telling us the reason why: "the trait "Send" is not implemented for "Rc<Mutex<i32>>" ". We’ll talk about "Send" in the next section: it’s one of the traits that ensures the types we use with threads are meant for use in concurrent situations.

Unfortunately, "Rc<T>" is not safe to share across threads. When "Rc<T>" manages the reference count, it adds to the count for each call to "clone" and subtracts from the count when each clone is dropped. But it doesn’t use any concurrency primitives to make sure that changes to the count can’t be interrupted by another thread. This could lead to wrong counts—subtle bugs that could in turn lead to memory leaks or a value being dropped before we’re done with it. What we need is a type exactly like "Rc<T>" but one that makes changes to the reference count in a thread-safe way.

#### [Atomic Reference Counting with "Arc"](https://doc.rust-lang.org/book/ch16-03-shared-state.html#atomic-reference-counting-with-arct)

Fortunately, "Arc<T>" *is* a type like "Rc<T>" that is safe to use in concurrent situations. The *a* stands for *atomic*, meaning it’s an *atomically reference counted* type. Atomics are an additional kind of concurrency primitive that we won’t cover in detail here: see the standard library documentation for ["std::sync::atomic"](https://doc.rust-lang.org/std/sync/atomic/index.html) for more details. At this point, you just need to know that atomics work like primitive types but are safe to share across threads.

You might then wonder why all primitive types aren’t atomic and why standard library types aren’t implemented to use "Arc<T>" by default. The reason is that thread safety comes with a performance penalty that you only want to pay when you really need to. If you’re just performing operations on values within a single thread, your code can run faster if it doesn’t have to enforce the guarantees atomics provide.

Let’s return to our example: "Arc<T>" and "Rc<T>" have the same API, so we fix our program by changing the "use" line, the call to "new", and the call to "clone". The code in Listing 16-15 will finally compile and run:

Filename: src/main.rs

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Listing 16-15: Using an "Arc<T>" to wrap the "Mutex<T>" to be able to share ownership across multiple threads

This code will print the following:

```
Result: 10
```

We did it! We counted from 0 to 10, which may not seem very impressive, but it did teach us a lot about "Mutex<T>" and thread safety. You could also use this program’s structure to do more complicated operations than just incrementing a counter. Using this strategy, you can divide a calculation into independent parts, split those parts across threads, and then use a "Mutex<T>" to have each thread update the final result with its part.

Note that if you are doing simple numerical operations, there are types simpler than "Mutex<T>" types provided by the ["std::sync::atomic" module of the standard library](https://doc.rust-lang.org/std/sync/atomic/index.html). These types provide safe, concurrent, atomic access to primitive types. We chose to use "Mutex<T>" with a primitive type for this example so we could concentrate on how "Mutex<T>" works.

### [Similarities Between "RefCell"/"Rc" and "Mutex"/"Arc"](https://doc.rust-lang.org/book/ch16-03-shared-state.html#similarities-between-refcelltrct-and-mutextarct)

You might have noticed that "counter" is immutable but we could get a mutable reference to the value inside it; this means "Mutex<T>" provides interior mutability, as the "Cell" family does. In the same way we used "RefCell<T>" in Chapter 15 to allow us to mutate contents inside an "Rc<T>", we use "Mutex<T>" to mutate contents inside an "Arc<T>".

Another detail to note is that Rust can’t protect you from all kinds of logic errors when you use "Mutex<T>". Recall in Chapter 15 that using "Rc<T>" came with the risk of creating reference cycles, where two "Rc<T>" values refer to each other, causing memory leaks. Similarly, "Mutex<T>" comes with the risk of creating *deadlocks*. These occur when an operation needs to lock two resources and two threads have each acquired one of the locks, causing them to wait for each other forever. If you’re interested in deadlocks, try creating a Rust program that has a deadlock; then research deadlock mitigation strategies for mutexes in any language and have a go at implementing them in Rust. The standard library API documentation for "Mutex<T>" and "MutexGuard" offers useful information.

We’ll round out this chapter by talking about the "Send" and "Sync" traits and how we can use them with custom types.





---





## [Extensible Concurrency with the "Sync" and "Send" Traits](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits)

Interestingly, the Rust language has *very* few concurrency features. Almost every concurrency feature we’ve talked about so far in this chapter has been part of the standard library, not the language. Your options for handling concurrency are not limited to the language or the standard library; you can write your own concurrency features or use those written by others.

However, two concurrency concepts are embedded in the language: the "std::marker" traits "Sync" and "Send".

### [Allowing Transference of Ownership Between Threads with "Send"](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#allowing-transference-of-ownership-between-threads-with-send)

The "Send" marker trait indicates that ownership of values of the type implementing "Send" can be transferred between threads. Almost every Rust type is "Send", but there are some exceptions, including "Rc<T>": this cannot be "Send" because if you cloned an "Rc<T>" value and tried to transfer ownership of the clone to another thread, both threads might update the reference count at the same time. For this reason, "Rc<T>" is implemented for use in single-threaded situations where you don’t want to pay the thread-safe performance penalty.

Therefore, Rust’s type system and trait bounds ensure that you can never accidentally send an "Rc<T>" value across threads unsafely. When we tried to do this in Listing 16-14, we got the error "the trait Send is not implemented for Rc<Mutex<i32>>". When we switched to "Arc<T>", which is "Send", the code compiled.

Any type composed entirely of "Send" types is automatically marked as "Send" as well. Almost all primitive types are "Send", aside from raw pointers, which we’ll discuss in Chapter 19.

### [Allowing Access from Multiple Threads with "Sync"](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#allowing-access-from-multiple-threads-with-sync)

The "Sync" marker trait indicates that it is safe for the type implementing "Sync" to be referenced from multiple threads. In other words, any type "T" is "Sync" if "&T" (an immutable reference to "T") is "Send", meaning the reference can be sent safely to another thread. Similar to "Send", primitive types are "Sync", and types composed entirely of types that are "Sync" are also "Sync".

The smart pointer "Rc<T>" is also not "Sync" for the same reasons that it’s not "Send". The "RefCell<T>" type (which we talked about in Chapter 15) and the family of related "Cell<T>" types are not "Sync". The implementation of borrow checking that "RefCell<T>" does at runtime is not thread-safe. The smart pointer "Mutex<T>" is "Sync" and can be used to share access with multiple threads as you saw in the [“Sharing a "Mutex" Between Multiple Threads”](https://doc.rust-lang.org/book/ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads) section.

### [Implementing "Send" and "Sync" Manually Is Unsafe](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#implementing-send-and-sync-manually-is-unsafe)

Because types that are made up of "Send" and "Sync" traits are automatically also "Send" and "Sync", we don’t have to implement those traits manually. As marker traits, they don’t even have any methods to implement. They’re just useful for enforcing invariants related to concurrency.

Manually implementing these traits involves implementing unsafe Rust code. We’ll talk about using unsafe Rust code in Chapter 19; for now, the important information is that building new concurrent types not made up of "Send" and "Sync" parts requires careful thought to uphold the safety guarantees. [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/index.html) has more information about these guarantees and how to uphold them.

## [Summary](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#summary)

This isn’t the last you’ll see of concurrency in this book: the project in Chapter 20 will use the concepts in this chapter in a more realistic situation than the smaller examples discussed here.

As mentioned earlier, because very little of how Rust handles concurrency is part of the language, many concurrency solutions are implemented as crates. These evolve more quickly than the standard library, so be sure to search online for the current, state-of-the-art crates to use in multithreaded situations.

The Rust standard library provides channels for message passing and smart pointer types, such as "Mutex<T>" and "Arc<T>", that are safe to use in concurrent contexts. The type system and the borrow checker ensure that the code using these solutions won’t end up with data races or invalid references. Once you get your code to compile, you can rest assured that it will happily run on multiple threads without the kinds of hard-to-track-down bugs common in other languages. Concurrent programming is no longer a concept to be afraid of: go forth and make your programs concurrent, fearlessly!

Next, we’ll talk about idiomatic ways to model problems and structure solutions as your Rust programs get bigger. In addition, we’ll discuss how Rust’s idioms relate to those you might be familiar with from object-oriented programming.

