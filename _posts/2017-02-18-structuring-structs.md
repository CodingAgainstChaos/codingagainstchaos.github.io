---
layout: post
title:  "Structuring Structs"
date:   2017-02-18 20:46:00 -0800
categories: swift
---
What’s the difference between a class and struct? The goto answer I often hear is about the memory model of reference vs value types. While technically correct, this misses the bigger picture. Structs produce data, classes produce objects. At first glance the difference is subtle but I will go through some of the design considerations while setting up a struct that will highlight the differences.

## Data + functions
Many functional programming languages, organize their namespaces in a similar way. A name space will consist of a data definition and some functions that operate on that data. For example in Clojure you might see:
{% highlight clojure %}
(defrecord Person [first-name last-name dob])

(defn full-name [person]
    (str (:first-name person) " " (:last-name person)))

(defn change-name [person first last]
    (assoc person :first-name first :last-name last))
{% endhighlight %}
This code will be in one file under one name space. Every function here deals with either deriving some information from the exisiting data or providing new data. If you squint your eyes, you can see a Swift struct in this Clojure code. To make it easier, here is the translation:
{% highlight swift %}
struct Person{
    var firstName: String
    var lastName: String
    var dob:Date

    var fullName: String {
        return "\(firstName) \(lastName)"
    }

    func changeName(first: String, last: String) -> Person {
       return Person(firstName: first, lastName: last, dob: dob)
    }
}
{% endhighlight %}
People with functional programming experience may lean towards only having static methods in a struct, making it impossible for those methods to contain any state. I would caution against that. If this was supposed to be the Swift way, we wouldn’t need the extra static keyword. It adds verbosity and Swift already provides syntax for that.

## Pure functions
Avoid creating methods with side effects. Methods in classes produce behavior, methods in structs produce data. Calling a method on an object is about telling the object to do something. Calling a method on a struct is always about asking for data. The easy rule of thumb here is that every struct method should return something unless it has the mutating keyword in which case it’s implictly returning a new instance of the struct. Your data shouldn’t know how to make network calls, save to the database, etc. Data is simple and declarative.

One more point to add is that structs have no deinit. This removes the opportunity to do some clean up for anything that’s stateful.

## Var properties, not let
Define your properties as vars not lets. While immutability does make everything easier, the truth is we often need to change our data. Most languages that use immutable data make this easy by providing some functions that help you change the value of one or more keys. Swift does not provide anything like this. This means that to update any value we have to use our constructor. This becomes more and more cumbersome as the number of properties grow. The Swiftier thing to do is default to making your properties vars. This allows the user of your struct to decide how they want to treat your data. The use of let on their side will enforce immutability. This gives us the flexibility within the struct to temporarily treat it as mutating.

For example:
{% highlight swift %}
struct Person {
    var firstName: String
    var lastName: String
    var dob: Date

    func changeName(first: String, last: String) -> Person {
       var temp = self
       temp.firstName = first
       temp.lastName = last
       return temp
    }
}
{% endhighlight %}
To the outside world, changeName is a pure function.

Of course, this doesn’t mean never use lets in a struct. It means that a let should be used for constants. A let in a struct should be for something that truly will never change. Things like pi will never change and make perfect sense as a let. However, if you’re unsure about whether something will eternally hold true then go with a var.

## Create initializers
Given the benefits of immutable data, which I won’t go into here, we should make it easy for users to instantiate our struct as a let constant. In order to do this, we need to provide good initializers.

Swift gives us a number of tools to create good initializers.

By default, we get an initializer that allows us to set all of our properties. The problem with this is that as soon as we declare our own init then we lose the default one. Although it may be verbose, it’s easy enough to rewrite this default init.

Swift allows us to have default parameters in our methods. This gives us the option of not passing that parameter in our method call. We can use this to create succint initializers.

For example:
{% highlight swift %}
struct Passenger {
    var seat: String
    var bags: [Bag]

    init(seat: String, bags: [Bag] = []){
        self.seat = seat
        self.bags = bags
    }
}

let passenger1 = Passenger(seat: "25A")
let passenger2 = Passenger(seat: "25B", bags: [Bag(id: "123")])
{% endhighlight %}
Also, reduce the number of states that can’t be reached through an initializer. While it may seem more intuitive to restrict data changes with methods, it’s more user friendly if that state can also be created directly with values. This is especially helpful when testing since the data won’t require any complex set up.

## Equality
One of the most basic operations we need to perform in a program is comparison. We often make decisions within our programs based on two values being equal or not. This idea of equality is independent of place and time when dealing with values. If we have:
{% highlight swift %}
let x = 1
let y = 1
{% endhighlight %}
then x always equals y regardless of where and when x and y are assigned. With reference types this isn’t the case. Two instances of a reference type are not equal by default since we compare their place in memory not the values they reference.

Implement the Equatable protocol for your structs. You may never check equality between your structs but it’s an important exercise for understanding the domain that you are modeling. It forces you to think about what the struct is representing. For example, if we have a Person struct and want to implement Equatable. Our default may be to just compare all of the properties. If a person changed their name, should they be considered a different person now? While at first this may seem too philosophical, it will help you model your domain.