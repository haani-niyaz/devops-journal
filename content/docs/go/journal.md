---
title: Journal
---

# Journal
---

- Interfaces are **abstract types** that define a set of functions that need to be implemented so that a type can be considered an instance of the interface. When this is done, the type is said to satisfy the interface. So, an interface is two things: a set of methods and a type, and it is used to define the behavior of other types.
- The biggest advantage you get from having and using an interface is that you can pass a variable of a type that implements that particular interface to any function that expects a parameter of that specific interface.


## Strings vs. Runes

### String

1. A string value is a read-only byte slice.
2. each string literal is encoded in utf-8. *A byte is an alias of uint8*. For example:

{{< highlight go >}}
s = "hello" # read-only byte slice

fmt.Printf("type: %T\n", s[0]) # uint8
{{< / highlight >}}

3. Each character in a string takes **1-3** bytes.

4. When you look up the `len()` of a string it returns the number of bytes used.

### Rune

1. A rune is an alias for int32.
2. Each rune in a []rune takes up **4** bytes.
3. When you look up the `len()` of a string it returns number of runes used.

More details can be found [here](https://stackoverflow.com/a/51611567/2180697)

### Why should I care?

When you need to shuffle a string for instance, you cannot shuffle it based on string literal indexes if special characters are present. The reason for this is because the in utf-8, ASCII characters are allocated a 1 byte to correspond to the first 128 unicode characters. However, special characters can take up to **1-3** bytes i.e:  `s := "hello你好"` where the length of the string is `11`due to the 3 bytes allocated to each special character. This poses a problem when you want to index a characters in a string and swap them. To get around this what you do is convert the string to a `[]rune` because each rune is allocated is **4** bytes allowing you to index character in the slice of runes.

See an example [here](https://golangbyexample.com/generate-random-password-golang/#:~:text='math%2Frand'%20package%20of,password%20from%20a%20character%20set.)


## Context

### Reading

- https://stackoverflow.com/questions/40379960/golang-context-withvalue-how-to-add-several-key-value-pairs 




