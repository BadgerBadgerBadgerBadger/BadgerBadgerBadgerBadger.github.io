---
layout: post
title: "My ES Lint and Why"
excerpt: "The linting rules I use with ESLint and why they make sense to me."
modified: 2018-09-21T16:17:25-04:00
categories: code-craftsmanship
tags: [nodejs, coding-style, eslint]
comments: true
share: true
---

I recently updated my company's main NodeJS repository to include formatting via [prettier-eslint](https://github.com/prettier/prettier-eslint) and linting via [eslint](https://eslint.org/). My colleague asked me to explain my choice of rules since it's tedious to go through each and every rule and figure out why it was chosen and I thought it was worth getting a  blog post out of it, so here we go.

This is the eslint config I plan to use:
```
{
    "env": {
        "node": true,
        "es6": true
    },
    "extends": "eslint:recommended",
    "rules": {
        "arrow-spacing": "error",
        "curly": "error",
        "dot-notation": "error",
        "eol-last": "error",
        "generator-star-spacing": [
            "error",
            {
                "before": false,
                "after": true
            }
        ],
        "indent": ["error", 4],
        "no-buffer-constructor": "error",
        "no-confusing-arrow": "error",
        "no-magic-numbers": [
            "error",
            {
                "enforceConst": true
            }
        ],
        "no-multi-spaces": "error",
        "no-path-concat": "error",
        "no-var": "error",
        "no-self-compare": "error",
        "no-throw-literal": "error",
        "no-trailing-spaces": "error",
        "no-unused-expressions": "error",
        "no-useless-concat": "error",
        "padding-line-between-statements": [
            "error",
            { "blankLine": "always", "prev": "directive", "next": "*" }
        ],
        "prefer-const": "error",
        "prefer-promise-reject-errors": "error",
        "prefer-template": "error",
        "quotes": ["error", "double"],
        "object-shorthand": "error",
        "semi": ["error", "never"],
        "strict": ["error", "global"],
        "yoda": "error"
    }
}
```

I've set the env as `node` and `es6`, I hope those choices are fairly obvious. I'm also extending the [eslint:recommended](https://eslint.org/docs/rules/)(look for the ones with the ✔️ before them), most of whose rules I've gone over and satisfied myself that I agree with.

On top of this I have added extra rules, all of them as errorables (they'll throw an error when they fail, not a warning). Below I'll be going through each and giving a short description of what the rule does (very tl;dr, go to the rule page for comprehensive details), and why I personally chose it.

**arrow-spacing**

This rule normalizes style of spacing before/after an arrow function’s arrow(=>) The default is to have one space before and one space after.

I find it easier to read when we have `(arg => expression)` as opposed to `(arg=> expression)` or `(arg =>expression)`

**curly**

JavaScript allows the omission of curly braces when a block contains only one statement. However, it is considered by many to be best practice to never omit curly braces around blocks, even when they are optional, because it can lead to bugs and reduces code clarity. By default this rule requires curly braces for all block statements.

I like my block statements to be clearly defined. Blocks imply scope and when I'm reading code it's much better that a curly easily tells me that this is a different scope.

**dot-notation**

In JavaScript, one can access properties using the dot notation (foo.bar) or square-bracket notation (foo["bar"]). However, the dot notation is often preferred because it is easier to read, less verbose, and works better with aggressive JavaScript minimizers.

The dot notation syntax is terser. `object.field` is not harder to read than `object["field"]` or `object['field']` but uses less characters. In some cases, of course, you need square-bracket syntax (`object["field-name"]`) and the rule respects that.

**eol-last**

Trailing newlines in non-empty files are a common UNIX idiom. Benefits of trailing newlines include the ability to concatenate or append to files as well as output files to the terminal without interfering with shell prompts.

Enough said.

**generator-star-spacing**

Generators are a new type of function in ECMAScript 6 that can return multiple values over time. These special functions are indicated by placing an * after the function keyword. To keep a sense of consistency when using generators this rule enforces a single position for the *.

I prefer `function* genName` over `function * genName` or `function *genName` for a very simple reason. A generator function is not the same as a normal function. It has different calling semantics. Syntactically the `*` is the only thing that differentiates a generator function from a regular function. It makes sense to me that this syntax is attached to the `function` keyword rather the name (which can be arbitrary).

**indent**

I hardly need to explain this one.

**no-buffer-constructor**

In Node.js, the behavior of the Buffer constructor is different depending on the type of its argument. Passing an argument from user input to Buffer() without validating its type can lead to security vulnerabilities such as remote memory disclosure and denial of service. As a result, the Buffer constructor has been deprecated and should not be used. Use the producer methods `Buffer.from`, `Buffer.alloc`, and `Buffer.allocUnsafe` instead.

**no-confusing-arrow**

Arrow functions (=>) are similar in syntax to some comparison operators (>, <, <=, and >=). This rule warns against using the arrow function syntax in places where it could be confused with a comparison operator. Even if the arguments of the arrow function are wrapped with parens, this rule still warns about it unless allowParens is set to true.

I love being able to read code fast with my brain doing minimal work in trying to parse text.

**no-magic-numbers**

‘Magic numbers’ are numbers that occur multiple time in code without an explicit meaning. They should preferably be replaced by named constants. The no-magic-numbers rule aims to make code more readable and refactoring easier by ensuring that special numbers are declared as constants to make their meaning explicit.

1. You think this is only important if someone else will read your code. Well, guess what? 6 months from the time of coding, you will be a very different person, and I guarantee your own code won't make sense to you if you use magic numbers.
2. Refactoring becomes _so_ much easier when the number is used in more than one place.

**no-multi-spaces**

Multiple spaces in a row that are not used for indentation are typically mistakes. This rule aims to disallow multiple whitespace around logical expressions, conditional expressions, declarations, array elements, object properties, sequences and function parameters.

Multiple spaces are often hard to read especially if you aren't vieweing your code in your personal, fully-configured IDE.

**no-path-concat**

In Node.js, the `__dirname` and `__filename` global variables contain the directory path and the file path of the currently executing script file, respectively. Sometimes, developers try to use these variables to create paths to other files, such as:
```
var fullPath = __dirname + "/foo.js";
```
However, there are a few problems with this. First, you can’t be sure what type of system the script is running on. Node.js can be run on any computer, including Windows, which uses a different path separator. It’s very easy, therefore, to create an invalid path using string concatenation and assuming Unix-style separators. There’s also the possibility of having double separators, or otherwise ending up with an invalid path.

In order to avoid any confusion as to how to create the correct path, Node.js provides the path module. This module uses system-specific information to always return the correct value. So you can rewrite the previous example as:
```
var fullPath = path.join(__dirname, "foo.js");
```
This example doesn’t need to include separators as `path.join()` will do it in the most appropriate manner. Alternately, you can use `path.resolve()` to retrieve the fully-qualified path:
```
var fullPath = path.resolve(__dirname, "foo.js");
```
Both `path.join()` and `path.resolve()` are suitable replacements for string concatenation wherever file or directory paths are being created.

**no-var**

ECMAScript 6 allows programmers to create variables with block scope instead of function scope using the let and const keywords. This rule is aimed at discouraging the use of var and encouraging the use of const or let instead.

`const` and `let` have been a godsend in terms of indicating intent.

**no-self-compare**

Comparing a variable against itself is usually an error, either a typo or refactoring error. It is confusing to the reader and may potentially introduce a runtime error.

The only time you would compare a variable against itself is when you are testing for NaN. However, it is far more appropriate to use typeof x === 'number' && isNaN(x) or the Number.isNaN ES2015 function for that use case rather than leaving the reader of the code to determine the intent of self comparison.

This error is raised to highlight a potentially confusing and potentially pointless piece of code. There are almost no situations in which you would need to compare something to itself.

**no-throw-literal**

It is considered good practice to only throw the Error object itself or an object using the Error object as base objects for user-defined exceptions. The fundamental benefit of Error objects is that they automatically keep track of where they were built and originated.

This rule restricts what can be thrown as an exception. When it was first created, it only prevented literals from being thrown (hence the name), but it has now been expanded to only allow expressions which have a possibility of being an Error object.

**no-trailing-spaces**

Sometimes in the course of editing files, you can end up with extra whitespace at the end of lines. These whitespace differences can be picked up by source control systems and flagged as diffs, causing frustration for developers. While this extra whitespace causes no functional issues, many code conventions require that trailing spaces be removed before check-in.

This rule disallows trailing whitespace (spaces, tabs, and other Unicode whitespace characters) at the end of lines.

**no-unused-expressions**

An unused expression which has no effect on the state of the program indicates a logic error.

For example, `n + 1;` is not a syntax error, but it might be a typing mistake where a programmer meant an assignment statement `n += 1;` instead.

**no-useless-concat**

It’s unnecessary to concatenate two strings together, such as: `var foo = "a" + "b";` This code is likely the result of refactoring where a variable was removed from the concatenation (such as "a" + b + "b"). In such a case, the concatenation isn’t important and the code can be rewritten as: `var foo = "ab";`

**padding-line-between-statements**

This rule requires or disallows blank lines between the given 2 kinds of statements. Properly blank lines help developers to understand the code.

For example, the following configuration requires a blank line between a variable declaration and a return statement.

```
/*eslint padding-line-between-statements: [
    "error",
    { blankLine: "always", prev: "var", next: "return" }
]*/

function foo() {
    var a = 1;

    return a;
}
```

A configuration is an object which has 3 properties; `blankLine`, `prev` and `next`. For example, `{ blankLine: "always", prev: "var", next: "return" }` means “it requires one or more blank lines between a variable declaration and a return statement.” You can supply any number of configurations. If a statement pair matches multiple configurations, the last matched configuration will be used.

I have just started fleshing out this rule. I'd like a newline after a directive, please. Makes it easier to read, for me.

**prefer-const**

If a variable is never reassigned, using the `const` declaration is better.

`const` declaration tells readers, “this variable is never reassigned,” reducing cognitive load and improving maintainability.

This rule is aimed at flagging variables that are declared using `let` keyword, but never reassigned after the initial assignment.

**prefer-promise-reject-errors**

It is considered good practice to only pass instances of the built-in Error object to the `reject()` function for user-defined errors in Promises. Error objects automatically store a stack trace, which can be used to debug an error by determining where it came from. If a Promise is rejected with a non-Error value, it can be difficult to determine where the rejection occurred.

This rule aims to ensure that Promises are only rejected with Error objects.

**prefer-template**

In ES2015 (ES6), we can use template literals instead of string concatenation. This rule is aimed to flag usage of `+` operators with strings.

Templated strings are easier to read than their `+` concatenated cousins since they follow a more natural flow.

**quotes**

JavaScript allows you to define strings in one of three ways: double quotes, single quotes, and backticks (as of ECMAScript 6). Each of these lines creates a string and, in some cases, can be used interchangeably. The choice of how to define strings in a codebase is a stylistic one outside of template literals (which allow embedded expressions to be interpreted).

Double quotes were chosen arbitralily.

**object-shorthand**

ECMAScript 6 provides a concise form for defining object literal methods and properties. This syntax can make defining complex object literals much cleaner.

This rule enforces the use of the shorthand syntax. This applies to all methods (including generators) defined in object literals and any properties defined where the key name matches name of the assigned variable.

Having `{name: name}` conveys no extra information.

**semi**

Typing mistakes and misunderstandings about where semicolons are required can lead to semicolons that are unnecessary. While not technically an error, extra semicolons can cause confusion when reading code.

Explained [here](https://scionofbytes.me/code-craftsmanship/semico-what/).

**strict**

A strict mode directive is a "use strict" literal at the beginning of a script or function body. It enables strict mode semantics.

I've seen issues with non-strict mode especially that it allows variable assignment without declaration. It's a buggy nightmare.

**yoda**

Yoda conditions are so named because the literal value of the condition comes first while the variable comes second. For example, the following is a Yoda condition:
```
if ("red" === color) {
    // ...
}
```
This is called a Yoda condition because it reads as, “if red equals the color”, similar to the way the Star Wars character Yoda speaks. Compare to the other way of arranging the operands:
```
if (color === "red") {
    // ...
}
```
This typically reads, “if the color equals red”, which is arguably a more natural way to describe the comparison.

Proponents of Yoda conditions highlight that it is impossible to mistakenly use = instead of == because you cannot assign to a literal value. Doing so will cause a syntax error and you will be informed of the mistake early on. This practice was therefore very common in early programming where tools were not yet available.

Opponents of Yoda conditions point out that tooling has made us better programmers because tools will catch the mistaken use of = instead of == (ESLint will catch this for you). Therefore, they argue, the utility of the pattern doesn’t outweigh the readability hit the code takes while using Yoda conditions.
