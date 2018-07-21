---
layout: post
title:  "Testing the Unthinkable"
date:   2018-07-21 09:00:00 -0700
tags: tla+, formal methods
---
Here's a quick exercise. Below are 4 different kinds of bugs. Rank them from most difficult to least difficult to debug, analyze and fix.

Bugs:
- String typo (e.g. typing a URL incorrectly)
- Race condition
- Logic error (e.g. calculating the wrong threshold amount, using > instead of <)
- Bad state after 10 step execution

Here's my ranking:

![Ranking]({{ "assets/img/error_ranking.jpg" | absolute_url }}){:width="400px"}

I ranked the string typo as the least difficult because you can probably detect that by taking a second look at the code. The logic error is a little harder in that you might have to do some debugging and step through to find the exact place where the program's next state is calculated incorrectly. However, one the place is found, it's often easy to analyze and fix. The 10 step execution jumps up significantly in difficulty because it might be difficult to find the exact 10 steps. Once found, the bug can be replicated and fixed. The most difficult is the race condition because it might not be possible to consistently replicate which makes analyzing and fixing close to impossible.

## Thinking vs. implementation
These bugs can be split in two categories: thinking and implementation errors.

![Categories]({{ "assets/img/error_categories.jpg" | absolute_url }}){:width="400px"}

Implementation errors are a mismatch between what we wanted to code and what we actually did. With an implementation error, the problem and solution space are completely understood but there was a copy error when transcribing from your brain to your text editor.

Thinking errors occur when the behavior of a program is underspecified. They aren't the result of a mistake but rather the result of an unknown or unforeseen scenario. These are all bugs that you couldn't even try to guard against because you didn't imagine them in the first place.

This distinction matters because each category has a different impact on the software lifecycle. Implementation errors are typically easier to debug, analyze and fix as we saw above and because of this, estimates for addressing the bugs tend to be more accurate. Thinking errors, on the other hand, introduce a lot of risk into the development process. Estimates can be highly inaccurate because they are "unknown unknowns". Additionally, fixes may require significant changes to the design which increases the probability of regressions in other areas of the code.

## Testing against errors
The most common method for reducing the risk of errors is software testing.

Testing these bugs also falls into different categories: 

![Testing]({{ "assets/img/error_testing.jpg" | absolute_url }}){:width="400px"}

High-level tests = end to end, integration, UI, etc.

Here we see a relationship: high-level tests are to thinking errors as unit tests are to implementation errors.

This creates a problem for us because as we move higher from unit tests, the cost of writing and maintaining the tests increases. To deal with this, it's common to use the strategy of a testing pyramid where most of the testing suite consists of unit tests. This means that the testing suite is focused on catching implementation errors. We end up with a mass of cheap tests that cover the less risky parts of our programs. 

![Testing]({{ "assets/img/error_testing_pyramid.jpg" | absolute_url }}){:width="400px"}

## The economics of lightweight formal methods
Lightweight formal methods completely change the ROI on verifying the thinking parts of building software. They lower the cost and increase the return. 

The term lightweight means that we do not specify our entire program, rather we focus on the level of abstraction we care about. Once the abstraction is specified, we exhaustively test the state space with a model checker.

### Costs
Tools like TLA+ do not have any of the costs that high-level tests have. [Martin Fowler writes](https://martinfowler.com/bliki/TestPyramid.html) that end to end tests are **brittle, expensive to write and time consuming to run**. Let's analyze each of these.

#### Brittle
Brittle means that the tests break easily when they shouldn't. High-level tests are brittle because they consist of several moving parts that have to be setup correctly and all working as expected. Also, a lot of the tooling for these tests depend on implementation details that can easily change such as a specific element in the UI having a specific name.

Formal specifications do not suffer from moving parts because they are just descriptions. All of the pieces needed for checking the model are contained within the specification. There is no dependency on outside systems and there is no execution of other programs. In addition, these tools do not rely on any implementation details so are not at risk when those details change.

#### Expensive to write
The main expense when writing integration tests is the setup. Because there are many moving parts, each part needs to be configured, initialized and potentially mocked. This is all additional code that needs to be maintained and increases coupling to the implementation. Often checking a few properties of a system requires a order of magnitude more setup code.

Formal methods do not require any setup code because they are not tied to the actual components under test. The specifications only need to change when the abstractions change. There is the cost of having to completely describe your system in order to check it properly. Upfront this seems like duplication. However, this should still result in less lines of specification than code because in formal methods the programmer doesn't have to define all of the permutations possible to test them -- the model checker does that for you.

#### Time consuming to run
High-level tests can take a long time to run because they rely on the computational complexity of each component and the latency between those components. This introduces accidental complexity into the test suite. Rarely is the computational complexity and latency relevant to the test.

Model checking can also take a long time to run. What makes model checking slow is the state space of the program. Because model checkers exhaustively iterate through the state space, a lot of state means a lot of checking. However, this is essential complexity. While this certainly is a cost, it is directly proportional to the return.

### Returns
All tests are dependent on the inputs they use. A test can only show that an error doesn't occur for that specific example. While this is often used a knock against tests, in practice it's a cost trade off. We pick a range of inputs which reduces the probability that an error will occur. This moves us from proof to statistics. Unit tests are often a good trade off where they are low cost but also provide significant risk reduction. Integration tests on the other hand are high cost and quickly reach a number of permutations that are impossible to test. Thus the probability of errors exisiting remains high.

In all formal methods, the returns are easier to measure -- no thinking errors. Of course, this is limited by the quality of the checks and constraints embedded in the spec. This amounts to the absence of thinking errors being limited by the ability to think clearly. However, this isn't a one-way street. While formal methods are able to check the thinking, they do also help clarify your thinking. Note that there is a distinction here between being wrong in your thinking and thinking the wrong thing. [Software verification covers the first where as validation covers the latter](https://en.wikipedia.org/wiki/Software_verification_and_validation#Definitions). This post only addresses verification.

## Working together
The majority of this post contrasts lightweight formal methods with high-level tests (end to end and integration). However, the point is not to suggest formal methods replace testing. Overall, our goal is to provide our customers with quality software and reduce the cost of doing so. Rather than treating all bugs as defects in the code, we can gain a lot by focusing on implementation vs. thinking errors and using the appropriate tools to tackle each. Formal methods are typically thought of as being prohibitively expensive except for in the most safety-critical software. Lightweight formal methods have much more favorable economics, not just when compared to traditional formal methods but even when compared to traditional testing techniques. All but the most trivial programs can benefit from adding these tools to their verification strategy.
