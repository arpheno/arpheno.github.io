---
layout: post
title:  "Syncying two waves and low pass filtering a wave file with python"
date:   2016-2-2 14:43:18
categories: python clean-architecture coroutines generators yield
---
Today I had an epiphany.

# The "yield" statement

You might think: "Oh come on this has been done a million times and generators are sort-of-cool-ish, but nothing revolutionary.
And you'd be right. Except: It's not all about generators in the traditional sense... Let's see.

## Generators

When you use the `yield` statement in a python function definition it becomes a generator:

    def fibonacci_numbers():
        """ A generator that yields fibonacci numbers and then stops"""
        yield 1
        yield 1
        yield 2
        yield 3
        yield 5
        yield 8
        yield 13
Generators implement the iterator protocol, and some of the most typical usage is to iterate over a generator:

    >>> for fib in fibonacci_numbers():
    >>>     print fib
    1
    1
    2
    3
    5
    8
    13

So far it's old news. I want to examine another interesting property of yield and how it changed the way I program business logic:

## `yield` is essentially an inverse subroutine call

That's one confusing sentence, I know. I will explain it the end, for now bear with me.

Let's consider the following task: Implement a gradient descend algorithm. Being the professional programmers we are, we start with a test:

tests.py:
 
    def test_gradient_descend():
        gradient_descend()
    
graddesc.py

    def gradient_descend():
        pass # Stay agile, stay alive

Since we don't really know how we would go about implementing the entire algorithm, we would like to start with something we know:
In order to generate the gradient at point P, we need to calculate the difference of function values at P+delta and P-delta, 
for some delta across all dimensions. The vector of differences in all dimensions will then give us delta. Let's start by generating 
the array of P+delta,P-delta across all dimensions:

tests.py:

    def test_adjacents():
        point = (1,0)
        delta = 0.1
        assert adjacents(point, delta) == [(0.9,0),(1.1,0),(1,-0.1),(1,0.1)]
graddesc.py

    def adjacents(point, delta):
        adjacents = []
        for i in range(len(point)):
            new = deepcopy(point)
            new[i] += delta
            adjacents.append(new)
            new = deepcopy(point)
            new[i] -= delta
            adjacents.append(new)
        return adjacents

It's beautiful. We are trying to calculate the adjacents and have written a low level business logic function. We don't depend on anything 
and our function doesn't rely on any outside state.

Let's take the next step and calculate the gradient from the adjacents:
### The traditional appraoch

tests.py

    def test_calc_grad_traditional():
        point = (1,0)
        assert calc_grad_traditional(point) == (5,1) # this takes 2 minutes
graddesc.py

    from helpers import cluster_evaluate
    def calc_grad_traditional(point):
        function_values =  cluster_evaluate(adjacents(point))  # retrieve the function values from a computing cluster
        gradient = [v1.fitness.values[0] - v2.fitness.values[0] for v1, v2 in
                    zip(adj[::2], adj[1::2])]
        norm = np.linalg.norm(gradient)
        gradient = [v / norm for v in gradient]
        return gradient
Uh, oh, something went wrong here. As it happens, the function value of a point that we would like to evaluate, is actually measured
on a 100 machines cluster and evaluates whether patterns. The evaluation of one point takes around two minutes. This is unacceptable for a unit test.

The "solution"? Passing the evaluation function as an argument and mocking it out in the test:
tests.py
    def mock_evaluate():
        return [0,5,0,1]
    def test_calc_grad_traditional():
        point = (1,0)
        assert calc_grad_traditional(point,mock_evaluate) == (5,1) # this takes 2 minutes
graddesc.py

    def calc_grad_traditional(point,evaluation_function):
        function_values =  evaluation_function(adjacents(point))  # retrieve the function values from a computing cluster
        gradient = [v1.fitness.values[0] - v2.fitness.values[0] for v1, v2 in
                    zip(adj[::2], adj[1::2])]
        norm = np.linalg.norm(gradient)
        gradient = [v / norm for v in gradient]
        return gradient

Phew, we've decoupled from "helpers.py" for our graddesc! Great success! But wait...

Let's say we'd like to run several evaluations on different clusters using different methods and then average over it:

tests.py

    def mock_evaluate(points):
        return [0,5,0,1]
    def test_calc_grad_traditional():
        point = (1,0)
        assert calc_grad_traditional(point,mock_evaluate,mock_evalute,mock_evaluate,mock_evaluate) == (5,1) # this fast and ugly
graddesc.py

    def calc_grad_traditional(point,evaluation_function1,evaluation_function2,evaluation_function3,evaluation_function4):
        function_values =  evaluation_function1(adjacents(point))  # retrieve the function values from a computing cluster
        function_values2=  evaluation_function2(adjacents(point))  # retrieve the function values from a computing cluster
        function_values3=  evaluation_function3(adjacents(point))  # retrieve the function values from a computing cluster
        function_values4=  evaluation_function4(adjacents(point))  # retrieve the function values from a computing cluster
        gradient = [v1.fitness.values[0] - v2.fitness.values[0] for v1, v2 in
                    zip(adj[::2], adj[1::2])]
        norm = np.linalg.norm(gradient)
        gradient = [v / norm for v in gradient]
        return gradient
This doesn't look good.... our real parameters that we would like to pass(really just point) are getting swamped by parameters
that are only being passed because of the way we implemented our business logic. Imagine a large high level function that needs many 
mid level functions in order to execute correctly. The amount of functions passed can grow huge.
## `yield` to the rescue

Generators have had the interesting `send()` functionality in python for a while now.
It can be used to "communicate" with a generator, but genrally it sounds scary and confusing.

A simple Example:

    def double_number():
        number=2
        while True:
            new_number = yield number*2
            if new_number:
                number = new_number
            else:
                number = number*2
    my_generator = double_number()
    print my_generator.next()
    print my_generator.next()
    print my_generator.send(5)

This will print `4 8 10`.

### Understanding yield in the sense of flow of control
Think of `yield` as " I don't want to deal with this problem right now, let someone else deal with it. ( Much like "implementing" a python function with `pass`)
Rather than  thinking about `yield` as an element defining a generator, think of it as a way of telling your code that you will solve this part of the problem later.
In order to use this aspect of `yield` we need to use the methods `.next()` and `.send()` in order to communicate with our logic.
Consider this example:

graddesc.py

    def calc_grad_yield(point):
        function_values =   yield adjacents(point)  # Let someone else deal with the problem
        gradient = [v1.fitness.values[0] - v2.fitness.values[0] for v1, v2 in
                    zip(adj[::2], adj[1::2])]
        norm = np.linalg.norm(gradient)
        gradient = [v / norm for v in gradient]
        return gradient
At this point we have eliminated all dependencies entirely. The function `calc_grad_traditional` doesn't know anything about the outside
world or how `function_values` might be generated. It could be from a function call or just a variable.

The test now looks like this:

tests.py

    def mock_evaluate(points):
        return [0,5,0,1]
    def test_calc_grad_yield():
        point = (1,0)
        grad_calculator = calc_grad_yield() # instantiate coroutine 
        gradient = grad_calculator.send(mock_evaluate(grad_calculator.next())
        assert gradient == (5,1) # this is fast

We could also get rid of mock_evaluate entirely:

    def test_calc_grad_yield():
        point = (1,0)
        grad_calculator = calc_grad_yield(point) # instantiate coroutine 
        grad_calculator.next()
        gradient = grad_calculator.send([0,5,0,1))
        assert gradient == (5,1) # this is fast
For the same scenario that we would like to average over different clusters as before:

    def test_calc_grad_yield():
        point = (1,0)
        grad_calculator = calc_grad_yield(point) # instantiate coroutine 
        grad_calculator.next()
        gradient = grad_calculator.send([0,5,0,1])
        grad_calculator.next()
        gradient = grad_calculator.send([0,5,0,1])
        grad_calculator.next()
        gradient = grad_calculator.send([0,5,0,1])
        grad_calculator.next()
        gradient = grad_calculator.send([0,5,0,1])
        assert gradient == (5,1) # this is fast
or simply:

    def test_calc_grad_yield():
        point = (1,0)
        grad_calculator = calc_grad_yield(point) # instantiate coroutine 
        for adj in grad_calculator:
            gradient = grad_calculator.send([0,5,0,1])
        assert gradient == (5,1) # this is fast
## Why is this useful?

We all know that there are no silver bullets, and no language construct can make complexity
"just go away". Obviously this use of `yield` can only put the complexity somewhere else: 

**The interactor**
Let's create a function that encapsulates and hides away the uglyness of the `next()` and `send()` from
high level control functions like `test_calc_grad_yield()` or `main()`:

    def mock_grad_interactor(point): # The testing interface
        grad_calculator = calc_grad_yield(point) # instantiate coroutine 
        for adj in grad_calculator:
            gradient = grad_calculator.send([0,5,0,1])
        return gradient
        
    def grad_interactor(point): # The real thing
        grad_calculator = calc_grad_yield(point) # instantiate coroutine 
        for adj in grad_calculator:
            values = cluster_evaluate(adj)
            gradient = grad_calculator.send(values)
        return gradient
      
The complexity gets shifted to this routine operating the high level business logic containing `yield`.
The whole business of using `next()` and `send()` is arguably not prettier to look at than 
passing a million function references to a low level function. 

But it can do more: It flips around the chain of dependency. It serves as an interface between the 
high-level imperative code of `main()` and similar functions and the high-level
 object oriented code of the business logic.
 
Traditional Downward dependency from high level to low level:

    main() -> gradient_descend() -> calc_gradient()  -> evaluate()

When using an interactor and `yield`:

                                        evaluate()
                                           / \
                                            I
    main() -> gradient_descend() -> gradient_interactor <-> calc_gradient()

As you can see the interactor encapsulates all dependencies of the business logic, allowing
the business rules to remain absolutely clean of pollution from the outside world.
 
## Conclusion

In this post I have shown how `yield` can be used to reverse the flow of control at the border
of high level glue-code functions like `main()` and high level business logic functions. `yield`
used in coroutines rather than traditional generators can be viewed as an "inverse subroutine call",
while a sequence of yields in a function defines a certain interface.

It can be used in a manner similar to subclassing interfaces in programming languages in java to decouple
concept and implementation of functionality.

In practice I have begun programming all my business logic using generators and co-routines, 
while sticking to the good ol' subroutine calls for `main()`

# Last words
What I really want to convey, more than the relevance of `yield` for software design and architecture, 
is its significance for staying agile while programming. 

When you start programming business logic using coroutines typing `yield` becomes much like
typing:

    class ComplexObject(object):
        pass # I'll implement this later
The most awesome change that `yield` has brought to me is that I stopped worrying about implementations of
*any* kind while writing abstract business logic. Once I don't know how I am going to get some particular data
transformed in the way I want, even for a second, I just throw in a `yield` and go on.

It puts me at peace mentally, that by defining a loose
interface with a couple of yields, I can finish writing beautiful self contained independent rules, and can
worry about the ugliness of I/O later, in one very specific place and probably one specific file.

I hope you found this post entertaining and worthwhile to read and would appreciate any 
comments and feedback at arpheno@gmail.com.

