---
layout: post
title: Love for Expression Languages
modified: 2023-12-27T00:00:00+01:00
categories: misc
tags: [programming-languages]
comments: true
share: true
---

I love expression languages. In Rust I can do this:

```rust
let last_integer = {
    if last_number_which_list == 0 {
        word_to_integer(&last_number_in_words)
    } else {
        last_number_in_words.parse::<i32>()
    }
};
```

While in Go, to achieve the same result I would have to do something like this:

```go
last_integer := 0

if last_number_which_list == 0 {
  last_integer = word_to_integer(last_number_in_words)
} else {
  last_integer = strconv.Atoi(last_number_in_words)
}
```

Or what feels clunkier but comes closer to an expression:

```go
last_integer = func() int {
  if last_number_which_list == 0 {
    return word_to_integer(last_number_in_words)
  } else {
    return strconv.Atoi(last_number_in_words)
  }
}()
```

> Note: Error handling has been elided in all code samples for the sake of brevity. The idiomatic naming conventions and casing styles have also been ignored in order to provide as similar code samples as possible.

I wonder what the costs are associated with having expression blocks in your programming language.
