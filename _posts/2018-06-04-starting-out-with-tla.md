---
layout: post
title:  "Starting Out with TLA+"
tags: tla+
---
I recently attended Never Graduate Week at [Recurse Center](https://www.recurse.com/). For my project, I chose to tackle learning TLA+. Prior to the week at RC, I worked through [Learn TLA](https://learntla.com) and the [TLA+ video course](http://lamport.azurewebsites.net/video/videos.html). With some familiarity of the mechanics, my goal was to create my own specs. I shared my experience of going through this exercise with a few people and some similar questions/concerns came up. Here are my thoughts on those concerns.

## What is TLA?
TLA is a suite of tools for formal verification. The main tools we will focus on are the TLA+ specification language and the TLC model checker. While these names might sound completely foreign, they boil down to a languages for describing programs and a tool for checking those descriptions. This is similar to the relationship between a programming language which implements a program and a unit testing framework which checks that implementation. 

However, there is a big difference between a programming language and a specification language. A programming language tells a computer how to do something. A specification language focuses on what the programmer wants to do.  TLA+ is not executable and is not meant for translation.

To give you a feel for TLA+, here's a simple spec:
{% highlight tla+ %}
VARIABLES toggle

On == /\ toggle = FALSE
      /\ toggle' = TRUE
      
Off == /\ toggle = TRUE
       /\ toggle' = FALSE
       
Init == toggle = FALSE

Next == Init \/ [Next]_toggle
{% endhighlight %}
This is a spec for toggle functionality. Here `On` is defined as the step where the toggle is true in the current state and false in the next. `Off` is defined as the step where toggle is false in the current state and true in the next. The programs starts with the toggle as false and the next step can be `On` or `Off`. In TLA+ `/\` means `AND` and `\/` means `OR`. The prime (shown with `your_var'`) declares the variable in the next state.

## What about *real* programs?
While the above example is a toy spec, this small, simple language is incredibly expressive. In fact, **you can model any program or system in TLA+**. This is possible because TLA+ focuses on one core mental model -- state machines.

Leslie Lamport, the creator of TLA+, explains this in *Computer Science and State Machines* [^1]. The gist is that while programming languages may differ from each other, they all describe computations and computations can be described as state machines. One major benefit of this is that state machines are a powerful tool for reasoning about programs. With TLA+, we can reason about any program in any language, even if they are complex.

TLA+ makes this apparent when modeling concurrency. Concurrency bugs are notoriously difficult to reason about but TLA+ handles them as easily as a single-process program. First, we break down our processes into sequences of state transformations. For example, assume we have two processes, `A` and `B`. We can break `A` into `A1 -> A2 -> A3`. We can break `B` into `B1 -> B2 -> B3`. Then modeling concurrency is as simple as the permutation of these six steps, i.e.,
```
A1 -> B1 -> A2 -> B2 -> A3 -> B3
A1 -> A2 -> B1 -> A3 -> B2 -> B3
B1 -> A1 -> B2 -> B3 -> A2 -> A3
... etc ...
```
Even in this small example, there are a lot of sequences. This is where the model checker comes in. The model checker will try all of the different permutations and check that our state is always valid. The checks come in the form of invariants which are validations at each step and properties which are validations across steps.

## It's not just for *serious* problems
If you read around about TLA+, what you'll probably find is that it is useful for hard problems. These hard problems typically involve concurrency. These kinds of problems make a great use case because they are the types of issues that can't be found by many other tools.

However, verifying concurrency is not TLA+'s only focus. It is a consequence of how powerful the tool is. That power really comes from forcing the programmer to think precisely. Nothing in TLA+ can be ambiguous and everything must be defined in an atomic step. The magic of TLA+ is that in doing this exercise you understand your problem and then your solution more clearly. The model checker completes this process by verifying your thinking. 

Thinking problems creep up even in the simplest programs. Working memory is estimated to hold 7 things [^2]. That means that even a program with just 4 booleans in it, is hard to grasp. This state space explosion is one of the core problems of programming[^3].

## It's not code
TLA+ is not a programming language and specs are not code. At first this may seem like a downside of the tool but it's a major strength. Code serves as an isomorphism between what is in the programmer's head and what the computer will run. While we put a lot of effort in to keeping these two separate, they are inevitably tied together. To deal with this, we come up with abstractions. However, these abstractions always leak because we have to deal with the friction between the mental model and the implementation.

TLA+ lives completely in the world of abstractions. Because TLA+ is not code, we don't have to worry about implementation. The quirks of your programming language, framework, etc do not exist in TLA+. This dramatically reduces the cost of creating specs because they don't depend on code that already exists. You're small feature doesn't need to integrate with the existing code base. With TLA+, you can model how you believe the other code works and then spec your new work around that.

## Conclusion
Even after a week of using TLA+, I was really blown away. I definitely want to integrate this in to my dev process as it seems like a powerful tool for augmenting my problem solving skills. The learning curve was a lot less steep than I anticipated, especially because of the newer resources I mentioned above. My main takeaway from my week was that the economics on TLA+ are very different from what I thought. The language is purposely designed to be simple and you get a lot of value out of writing and checking the spec. The scope of projects that can benefit from this is a lot bigger than just mission critical software.

----
[^1]: *Computer Science and State Machines* by Leslie Lamport. [PDF](http://lamport.azurewebsites.net/pubs/deroever-festschrift.pdf)  
[^2]: <https://en.wikipedia.org/wiki/Working_memory#Capacity>  
[^3]: *No Silver Bullet -- Essence and Accident in Software Engineering* by Frederick Brooks. [PDF](http://worrydream.com/refs/Brooks-NoSilverBullet.pdf)  
