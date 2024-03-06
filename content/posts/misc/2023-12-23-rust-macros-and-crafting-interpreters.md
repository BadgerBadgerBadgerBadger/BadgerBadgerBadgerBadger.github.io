---
layout: post
title: Rust Macros (while Crafting Interpreters)
excerpt: Learning about (some) rust macros work while trying to build an
  interpreter from Robert Nystrom's excellent book Crafting Interpreters.
modified: 2023-12-31T00:00:00+01:00
categories: misc
tags:
  - programming-languages
  - rust
  - macros
  - crafting-interpreters
comments: true
share: true
---

I have never used [Rust](https://www.rust-lang.org/) extensively. The few times I have tried have ended up in me giving up due to unwinnable fights against the [borrow-checker](https://doc.rust-lang.org/1.8.0/book/references-and-borrowing.html). By the recommendation of a friend I am giving this another shot. 

I'm making my way through [Robert Nystrom's](https://journal.stuffwithstuff.com/) [Crafting Interpreters](https://craftinginterpreters.com/) as well as doing [2023's Advent of Code](https://adventofcode.com/2023). I am late to the latter but I am enjoying myself regardless. I am using Rust as my language of choice for both endeavours.

I am on the [Representing Code](https://craftinginterpreters.com/representing-code.html) chapter of Crafting Interpreters where Robert mentions using a separate tool for generating the classes for various Expression types instead of hand-writing each since they share a tedious amount of boilerplate. Had I been writing this in Go, this is the path I would have followed. However, with Rust, we have the power of [macros](https://doc.rust-lang.org/book/ch19-06-macros.html) which presents me with the perfect opportunity to learn how they work.

In Java (which is the language of choice for the first of the two interpreters we write with Robert, in the book), Robert recommends structuring Expressions as subclasses of an [abstract](https://www.digitalocean.com/community/tutorials/abstract-class-in-java) Expression class. The following code shows this, along with the `Binary` class that would extend the abstract Expression class:

```java
package com.craftinginterpreters.lox;

abstract class Expr { 
  static class Binary extends Expr {
    Binary(Expr left, Token operator, Expr right) {
      this.left = left;
      this.operator = operator;
      this.right = right;
    }

    final Expr left;
    final Token operator;
    final Expr right;
  }

  // Other expressions...
}
```

Since Rust does not have inheritance, we will need to look to something else. With the help of Github Copilot, I decided to go with using enums representing expressions like so:

```rust
use crate::token::Token;

pub enum Expr {
    Binary(Binary),
}

pub struct Binary {
    pub left: Box<Expr>,
    pub operator: Token,
    pub right: Box<Expr>,
}

impl Binary {
    pub fn new(left: Expr, operator: Token, right: Expr) -> Self {
        Binary {
            left: Box::new(left),
            operator,
            right: Box::new(right),
        }
    }
}
```

But similar to Robert's approach, instead of writing these out by hand, we shall automate. I'm thinking of a macro which might look like this: 
```rust
create_expr!(Binary, left: Expr, operator: Token, right: Expr);
```
Which will, hopefully, result in the definition of the `Binary` struct as shown above as well as a constructor function. We will define the `Expr` enum by hand and `Token` is already available as a type I've previously defined.

> **Note**: Our macro accepts two arguments, the first being the name of the expression type and the second being a variadic list of its fields. In the `Binary` case, `Binary` is the name of the expression and `left`, `operator` and `right` would be its fields. For a different Expression type, say, `Unary` this would be `create_expr!(Unary, operator: Token, right: Expr);`.

Now let's read up on how macros work in rust. My first stab at it involved [this page](https://doc.rust-lang.org/rust-by-example/macros.html) on the [_rust-by_example_](https://doc.rust-lang.org/stable/rust-by-example/) site. While it reads like a perfectly passable statement of facts, I find the language difficult to parse.

When I decided to give learning Rust another go I looked for articles which could explain the _magic_ behind Rust in an intuitive way. One of my first stumbling blocks was the module system and it kept baffling me until I came across [this article](https://www.sheshbabu.com/posts/rust-module-system/) and the secrets were finally laid bare. Something in it _clicked_ in my head. I recommend reading it if you have a few minutes to spare. It's not long but very informative.

The tutorial from the [LogRocket blog](https://blog.logrocket.com/macros-in-rust-a-tutorial-with-examples/) told me that I could use repeating arguments for macros and have repeating blocks of code generated for those arguments. It also reminded me that I can't write the `enum Expr` by hand, nor can I do multiple invocations of my `create_expr` macro with for each of my Expression types. The base `enum Expr` needs to know all its variants right at the beginning. So taking that into account, I need to reach for something like this:
```rust
macro_rules!(
  Binary, left: Expr, operator: Token, right: Expr;
  Grouping, expression Expr;
  Literal, value: Literal;
  Unary, operator: Token, right Expr;
)
```

This is the analog of this block in Robert's Java code:
```java
defineAst(outputDir, "Expr", Arrays.asList(
  "Binary   : Expr left, Token operator, Expr right",
  "Grouping : Expr expression",
  "Literal  : Object value",
  "Unary    : Token operator, Expr right"
));
```

After reading through [another article](https://earthly.dev/blog/rust-macros/) that did not help me much (I find the examples too simplistic), I finally came across [A Practical Intro to Macros in Rust 1.0](https://danielkeep.github.io/practical-intro-to-macros.html) by [Daniel Keep](https://github.com/DanielKeep) which has just the kind of real-world use case with which I like to learn things, not only because it provides the necessary complexity to dive deeper into a system but also for the personal insights the author has gleamed from having attempted a task.

> **Side Note**: Daniel only has one other article on his blog, [Rust Iterator Cheat Sheet](https://danielkeep.github.io/itercheat_baked.html) which is also a fun read. He seems to be one of those amazing writers who only have the one or two pieces out but from whom you'd wish to read a lot more. I've let him know via a [Github issue](https://github.com/DanielKeep/DanielKeep.github.io/issues/18) since that's the only way I could see to reach him. I hope he writes more.

This was the first stab at defining the `create_expr` macro:
```rust
macro_rules! create_expr {
    ($($name:ident, $($field:ident : $typ: ty),+);+) => {

        pub enum Expr {
            $(
                $name($name),
            )*
        }

        $(
            pub struct $name {
            }

            impl $name {
                pub fn new() -> Self {
                    $name {
                        $($field: $typ),+
                    }
                }
            }
        )*
    };
}
```
Let's break this down, more for my own understanding than yours.

We are capturing a repeating list of items, separated by a `;`. That looks like `$(<something>);+` where `<something>` can be replaced by one of Rust's [designators](https://doc.rust-lang.org/rust-by-example/macros/designators.html). Whatever `<something>` turns out to be, we want to capture _one or more of it_, that's what the `+` is for ([just like in regex!](https://chortle.ccsu.edu/finiteautomata/Section07/sect07_19.html)).

Our `<something>` happens to be `$name:ident, $($field:ident : $typ: ty),+`. There are two pieces here:
1. `$name:ident`
2. `$($field:ident : $typ: ty),+`

The first is for capturing our syntax for defining the name of the expression type, while the second is for capturing a list of one or more fields. You'll notice the second piece also has the form `$(<something>),+` but this time with a `,` separator. And our `<something>` in this case happens to be `$field:ident : $typ: ty` which matches syntax of the form `left: Expr` (for an explanation of what `ty` and `ident` mean, I recommend reading through some of the blog posts I've already linked).

Trying to compile it, it works!

But trying to use it with an invocation
```rust
create_expr!(Binary, left: Expr, operator: Token, right: Expr;
             Grouping, expression: Expr;
             Literal, value: Token;
             Unary, operator: Token, right: Expr
            );
```
does not work!
```sh
error: expected expression, found `Expr`
  --> src/expr/mod.rs:19:35
   |
18 |                       $name {
   |                       ----- while parsing this struct
19 |                           $($field: $typ),+
   |                                     ^^^^ expected expression
...
27 | / create_expr!(Binary, left: Expr, operator: Token, right: Expr;
28 | |              Grouping, expression: Expr;
29 | |              Literal, value: Token;
30 | |              Unary, operator: Token, right: Expr
31 | |             );
   | |_____________- in this macro invocation
   |
   = note: this error originates in the macro `create_expr` (in Nightly builds, run with -Z macro-backtrace for more info)
```

I have no idea what I did wrong here. So I will turn to Github Copilot and see if it can help me. [Copilot Chat](https://docs.github.com/en/copilot/github-copilot-chat/about-github-copilot-chat) to be specific; it seems to be a version of ChatGPT restricted to answering programming-related questions.

It's answers pointed out that I had made some glaring initial mistakes. I blame my girlfriend's vacuuming (that was a joke, don't @ me, I love her very much).

This is the offending piece of code:
```rust
$(
    pub struct $name {
    }

    impl $name {
        pub fn new() -> Self {
            $name {
                $($field: $typ),+
            }
        }
    }
)*
```

I forgot to use `$($field: $typ),+` in the struct definition block and then accept arguments for the `new` method that are then assigned to the struct declaration, like so:
```rust
$(
    pub struct $name {
        $($field: $typ),+
    }

    impl $name {
        pub fn new($($field: $typ),+) -> Self {
            Self {
                $($field),+
            }
        }
    }
)*
```
Oh and I need to call the macro with [Boxed](https://dhghomon.github.io/easy_rust/Chapter_53.html) `Expr` types to avoid having to allocate infinite heap 👼.
```rust
create_expr!(Binary, left: Box<Expr>, operator: Token, right: Box<Expr>;
             Grouping, expression: Box<Expr>;
             Literal, value: Token;
             Unary, operator: Token, right: Box<Expr>
            );
```

And this finally compiles!

> **Random Aside**: I've always struggled with understanding [Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html) in Rust. However, I recall an evening when a former colleague and I were walking to a guy's place to play some DnD. We had taken a tram to the nearest tram stop to said guy's place and were walking from the tram stop to his house. We were both eager and beginner rustaceans and were talking to each other about our own intuitions regarding various Rust systems. Somewhere along that walk I had an epiphany and I felt in my _bones_ that I finally understood Rust's lifetime system. Too bad I did not immediately make a note because, for the life of me, I cannot recall what the epiphany was and here I am, back to not really understanding Lifetimes.

Now let's try to run this.
```rust
let exp = expr::Unary::new(
    token::Token::new(
        token::TokenType::Minus,
        "-".to_string(),
        1,
        None,
        0,
    ),
    Box::new(
        expr::Expr::Literal(
            expr::Literal::new(
                token::Literal::Number(123.0),
            )
        )
    ),
);

println!("Welcome to rlox! {:?}", exp);
```

And after a generous sprinkling of `#[derive(Debug)]`, it works!
```sh
Welcome to rlox! Unary { operator: Token { token_type: Minus, lexeme: "-", line: 1, literal: None, column: 0 }, right: Literal(Literal { value: Number(123.0) }) }
```

At least for this very contrived, manual invocation. As I continue writing the interpreter, I will no doubt come across some bugs and rough edges. But I am glad I was able to figure this out in Rust's own style of doing things.

> Note: `rlox` is the name of the first interpreter I'm building, from the book. Robert implements the first in Java and calls it _jlox_, and the second in C and calls it _clox_. I will be writing both in Rust, so once I'm done with `rlox`, I'm not quite sure what to name the second.

> **Random Aside**: I avoided asking Githup Copilot to do all my work for me. I think I could have fed it the macro syntax I wanted to use and the result I was looking for and it would have spat out exactly the macro that I needed. But I feel that would have robbed me of the opportunity to learn how macros in Rust actually work. For example, did you know that unlike, say, [C](https://www.programiz.com/c-programming/c-preprocessor-macros), macros in rust do not expand into text but rather into [directly into AST](https://rustc-dev-guide.rust-lang.org/macro-expansion.html). I feel these are the fascinating tidbits are what you miss unless you do your own research. However, for running through and making sense of error messages, I find it to be the perfect tool.
