Go doesn't support inheritance in the classical sense; instead, in encourages `composition` as a way to extend the functionality of types. This is not a notion peculiar to Go. [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance) is a known principle of OOP and is featured in the very first chapter of the <i>Design Patterns</i> book.

`Embedding` is an important Go feature making composition more convenient and useful. While Go strives to be simple, embedding is one place where the essential complexity of the problem leaks somewhat.

There are three kinds of embedding in Go:
1. Structs in structs
2. Interfaces in interfaces
3. Interfaces in structs
