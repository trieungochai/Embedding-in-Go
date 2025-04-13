## Embedding structs in structs
We'll start with a simple example demonstrating the embedding of a struct in another struct:
```go
type Base struct {
    b int
}

type Container struct { // Container is the embedding struct
    Base                // Base is the embedded struct
    c string
}
```

Instances of `Container` will now have the field `b` as well.
In the [spec](https://tip.golang.org/ref/spec) it's called a `promoted` field. We can access it just as we'd do for `c`:

```go
co := Container{}
co.b = 1
co.c = "string"
fmt.Printf("co -> {b: %v, c: %v}\n", co.b, co.c)
```

When using a struct literal, however, we have to initialize the embedded struct as a whole, not its fields. Promoted fields cannot be used as field names in composite literals of the struct:

```go
co := Container{Base: Base{b: 10}, c: "foo"}
fmt.Printf("co -> {b: %v, c: %v}\n", co.b, co.c)
```

Note that the access `co.b` is a syntactic convenience; we can also do it more explicitly with `co.Base.b`.

---
## Methods
Embedding structs also works well with methods. Suppose we have this method available for Base:

```go
func (base Base) Describe() string {
  return fmt.Sprintf("base %d belongs to us", base.b)
}
```

We can now invoke it on instances of Container, as if it had this method too:

```go
fmt.Println(cc.Describe())
```

To understand the mechanics of this call better, it helps to visualize `Container` having an explicit field of type `Base` and an explicit `Describe` method that forwards the call:

```go
type Container struct {
  base Base
  c string
}

func (cont Container) Describe() string {
  return cont.base.Describe()
}
```

The effect of calling `Describe` on this alternative `Container` is similar to our original one which uses an embedding.

This example also demonstrates an important subtlety in how methods on embedded fields behave; when Base's Describe is called, it's passed a Base receiver (the leftmost (...) in the method definition), regardless of which embedding struct it's called through. This is different from inheritance in other languages like Python and C++, where inherited methods get a reference to the subclass they are invoked through. This is a key way in which embedding in Go is different from classical inheritance.

---
## Shadowing of embedded fields
What happens if the embedding struct has a field `x` and embeds a struct which also has a field `x`? In this case, when accessing `x` through the embedding struct, we get the embedding struct's field; the embedded struct's `x` is shadowed.

Here's an example demonstrating this:
```go
type Base struct {
  b   int
  tag string
}

func (base Base) DescribeTag() string {
  return fmt.Sprintf("Base tag is %s", base.tag)
}

type Container struct {
  Base
  c   string
  tag string
}

func (co Container) DescribeTag() string {
  return fmt.Sprintf("Container tag is %s", co.tag)
}
```

When used like this:

```go
b := Base{b: 10, tag: "b's tag"}
co := Container{Base: b, c: "foo", tag: "co's tag"}

fmt.Println(b.DescribeTag())
fmt.Println(co.DescribeTag())
```

This prints:
```
Base tag is b's tag
Container tag is co's tag
```

Note that when accessing co.tag, we get the tag field of `Container`, not the one coming in through the shadowing of `Base`. We could access the other one explicitly, though, with `co.Base.tag`.
