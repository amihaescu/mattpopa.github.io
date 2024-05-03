---
title: "Go-ing away from Java"
date: 2024-02-10
author: "Andrei M"
---

When I started learning Java 10+ years ago in college it wasn't really because of the portability of
the language or the JVM. It was because the bar of entry seemed the lowest and the community support
was pretty amazing. There was an abundance of resources available to learn from and the language
made sense to me. And for a while, it was great. A few years into it, I obviously started working
with other technologies, but Java remained my core tool.

Fast-forward to today, I have been working with Go for a few years now and while the switch from one
language to another has not been that drastic, due to the fact that they both are statically typed
there are a few key points that I find quite interesting.

First off, there are a bunch of things that the two languages have in common. They are both
statically typed, they both have memory management, hence garbage collection and they both are multi
paradigm supporting both OOP and functional programming, to various levels of degree. There are
others as well but my purpose here is to actually go through the differences and what I find
interesting about Go coming from Java.

### Building

Unlike Java, go has a built-in build tool that manages dependencies and builds the project. This is
really convenient since you don't have to worry about setting up a build tool like Maven or Gradle.
The `go` tool is a command line tool that is used to build, test and run Go code. It also has a
built-in dependency management tool called `go mod` that is used to manage dependencies in a Go
project.

Building a Go project with the command `go build` results in a single native binary that can run
only on the platform it was built on or that was specified via the `GOOS` and `GOARCH` environment.
This differs from Java where you have to have the JVM installed on the target machine to run the
code. This means that go is less portable than Java. However, this is not a big issue since the
application will most probably run in Docker container with an `alpine-linux` base image. This is
similar to how now we can build native Java applications with GraalVM.

Dependencies are specified in a `go.mod` file that is located in the root of the project. This file
contains the URL to the git repository of the dependency and the version of the dependency that is
used. Based on that information and on what part of the code is used, the `go` tool will only
package up part of the library into your binary. This results in a smaller binary size compared to a
Java application. This also has some drawbacks. Caching the dependencies is not as easy as in Java
where you can use a company-wide repository like Nexus or Artifactory. In Go, you need to set up a
proxy to cache the repositories or alternatively download them and provide them to the build tool.
This can be easily achieved with Docker container that has the dependencies downloaded and mounted
as a volume, and it works really well in a CI/CD pipeline, however for local development it can be a
bit of a hassle.

### OOP and Interfaces

The first difference in terms of OOP is that Go does not have classes. Instead, it has structs that
can have methods. This is similar to how you would define a class in Java, however in Go you can
define methods on any struct, not just the one that you defined the methods on.

```go
type Person struct {
Name string
Age int
}

func (p *Person) SayHello()
fmt.Printf("Hello, my name is %s and I am %d years old\n", p.Name, p.Age)
}
```

What is different from Java is that you can define these methods anywhere in your codebase, not just
in the struct definition - as it is in Java. Coming from Java, this can be a bit confusing at first,
but you get used to it pretty quickly. Something you might notice is that the method definition has
a `p *Person` receiver. This simply tells the Go compiler that this method is defined on the Person
struct and that the person is passed as a reference rather than as a copy of the object.

As I mentioned earlier, both languages support OOP. However, the way they do it is quite different.
Go OOP is based on interfaces and composition, while Java is based on inheritance. This means that
in Go you can define an interface and then implement it on a struct. This is similar to how you
would do it in Java, however in Go you can implement the interface on any struct, not just the one
that you defined the interface on. This is really powerful and allows for a lot of flexibility in
the code. It also allows for a lot of code reuse, since you can define an interface and then
implement it on multiple structs.

```go
type Car interface {
GetModel() string
}

type SportsCar struct {
Model string
}

func (s *SportsCar) GetModel() string {
return s.Model
}
```

In the previous example `SportsCar` implements the `Car` interface because it implements all the
methods defined by the interface. Notice there is no `implements` keyword. This opens up a lot of
opportunities such as in the case of mocking. Assuming you are using a `http-client` library that
defines a bunch of methods, but you are only using ``get(string url) interface{}`` method. You can
define an interfaces that only has the `get` method. The `http-client` struct will implement this
interface which means that now you have decoupled it from your code. This is really powerful and
cannot be done in Java. The drawback is that give simple interface definitions many structs might
match it thus making the type system not as robust.

In the previous example I defined a `Person` struct with two fields, `Name` and `Age`. Notice the
capitalization. There are no access modifiers in Go, so if you want to export a field or a method
you need to capitalize the first letter of the name. This is similar to how you would define a
public field or method in Java. Fields and methods that start with a lowercase letter are private to
that structure. There is no protected, default or package-private access modifier as in Java.

Like Java, go supports polymorphism. Meaning we can define a variable of type `Car` and assign a new
`SportsCar` to it.

```go
var c Car
c = &SportsCar{Model: "Ferrari"}
```

Here I declare a variable `c` of type `Car` and assign a new `SportsCar` to it. Notice the `&` which
means that I assign the address of the `SportsCar` struct instance to the variable `c`, similar to
how you would assign a reference in Java.

Go also allows for embedding of structs. This is similar to inheritance in Java, however it is not
the same. In Go, you can embed a struct into another struct and then access the fields and methods
of
the embedded struct as if they were defined on the struct that embeds them. This is similar to how
you would extend a class in Java, however in Go you can embed multiple structs into another struct.

```go
type Engine struct {
HorsePower int
}

type Car struct {
Engine // embeds the Engine struct
}

func main() {
c := Car{Engine{HorsePower: 200}
fmt.Println(c.HorsePower) // prints 200
}
```

}

### Data structures

Coming from Java, you are used to having a lot of data structures available to you. In Go, you don't
really have this luxury. The language has a few built-in data structures like slices, maps and
arrays, but that's pretty much it. There are no sets, queues, stacks or linked lists. This can be a
bit of a drawback, but you can easily implement these data structures yourself. Out of the three
data structures mentioned the slice is the one that doesn't immediately resonate with Java
developers. It is similar to arrays but unlike them, slices are dynamic. They are a flexible view
into the elements of an array.

At this point it might be useful to mention that you cannot use the `new` keyword to create new map
or array. Instead, you can use `make` to create a new instance of a slice, map or struct.

```go
s := make([]int, 0) // creates a new slice of integers with a length of 0

m := make(map[string]int) // creates a new map with string keys and integer values
```

### Concurrency

This part requires a separate article, but I will briefly mention it here. Go has built-in support
for concurrency. This is done through goroutines and channels. Goroutines are lightweight threads
that are managed by the Go runtime. Goroutines are started with the `go` keyword and run
concurrently
with the main program. Any function can be run as goroutine.

```go
func longRunningFunction() {
// do some work
}

func main() {
    go longRunningFunction()
    // do some other work
}
```

Channels are used to communicate between goroutines. They are a way to pass data between goroutines.
Channels can be used to send and receive data between goroutines. They are similar to Unix pipes and
are used to connect goroutines together. Channels are created with the `make` keyword and can be of
two types: bounded and un-bounded. Bounded channels have a fixed size and can only hold a certain
number of elements. Un-bounded channels have no size limit and can hold an unlimited number of
elements.

```go

func main() {
    c := make(chan int) // creates an unbounded channel for int printingFunction(c)
    go printingFunction(c) // starts the printingFunction goroutine
    fmt.Println("Sending data to the channel...")
    for i := 0; i < 10; i++ {
        c <- i // sends the value of i to the channel
    }
}

func printingFunction(c chan int) {
    fmt.Println("Awaiting for data to be sent to the channel...")
    for i := range c { // reads from the channel
        fmt.Println(i) // prints the value of i
    }
}
```

Comparing it to how Java handles concurrency, Go is much simpler and easier to work with. However, 
things can get complicated when you have multiple goroutines running concurrently and you need to 
sychronize them. 

### Conclusion

Overall, I find go to be an interesting language. The strong points of this language are related to performance, 
low memory footprint and the built-in support for concurrency. The language is simple and easy to learn, but hard 
to master. My main complaint around Go is the rather immature ecosystem. There are a lot of libraries available,
however the quality of these libraries can vary. Some may not be supported anymore, some may not be well documented
and some may not be well tested. If you are working in a corporate environment, where stability and support are 
important, I think Java would be a better opportunity. Go would be more appropriate in an environment that can 
tolarate a slight lack of maturity in the ecosystem where most things are built in-house with little reliance on 
external libraries and frameworks.






