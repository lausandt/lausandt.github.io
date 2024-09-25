Title:  Function composition a deeper look 
Date: 2024-08-07
Modified: 2024-08-07
Author: Lau Sandt
Summary: A look at how Category Theory underpins function composition.

What is computer programming but the decomposition of computable problems into smaller problems, if necessary, the decomposition of the smaller problems into even smaller problems, and so on. Writing code to solve the smallest of the decomposed problems and composing those smaller solutions to create a solution to our larger problem.

A function is the manner with which we calculate the solution to our smallest problem. We give our function some input, and we get some output. If we decompose our problem, we inevitably need to recompose our set of smaller solutions to larger solutions. The mathematical idea of function composition is a natural fit to achieve this.

Let me give a small example of function composition. Say you want to add a number to some number and then multiply the result; that is function composition. $(n + 6) \times 4$ We take two functions, addition and multiplication, and compose them. We have done this since primary school. 

We can do the same in programming languages. For example, in Python, you would write something like:

```
def n_plus_six_times_four(n: Number) -> Number:
    return (n+6)*4
```
or if we really wanted to express the composition, we would write:

```
def plus_six(n: Number) -> Number:
    return n + 6

def times_four(n: Number) -> Number:
    return n*4

def n_plus_six_times_four(n: Number) -> Number:
    return times_four(plus_six(n))
```

There are two ways to continue: just accept function composition as a fact and just use it; let's call it the practical way, which is perfectly fine even for a developer. Or do it the curious way and try to understand function composition a bit deeper. For me, I was only able to really appreciate function composition on a deeper level once I got introduced to Haskell at uni.

While learning Haskell, you will get introduced to concepts like monoids, functors, and monads. Trying to understand these concepts is not easy, for you can do very powerful things with these concepts, yet they are deceptively simple. This combination, for me, makes them strangely complicated to grasp. These concepts come from Category Theory, a mathematical theory "competing" with Set Theory as foundational to mathematics itself.

Category Theory is a very interesting rabbit hole; you can get lost in it, as I am, but it has wonderful explanatory powers. The process of function composition can get its theoretical foundations in Category Theory. A category (this not a rigorous definition, see [wikipedia](https://en.wikipedia.org/wiki/Category_theory) for one) is a simple beast. It is basically nothing more than a class of objects and a class of morphisms to allow you to go from an object to another object. 

These morpisms are also known as arrows or mappings. To me this sounds like a function. A function is after all nothing more than a mapping from some domain to a codomain, often written as $A \rightarrow B$.

Finally, a category has a binary operation called composition. Given that I have the mappings $A \rightarrow B$ & $B \rightarrow C$ When I compose these mappings $A \rightarrow B \circ B \rightarrow C \implies A \rightarrow C$ 

There are some rules to such a composition in Category Theory.

1. function composition is associative it adheres to the associative law $h \cdot (g \cdot f) = (h \cdot g) \cdot f \quad \forall f,g,h \in S$ For instance addition is a binary operation that is associative. $(3 + 4) + 5 = 3 + (4 + 5)$ 
2. for every object A there is an identity on A such that the composition of $f \cdot id_a = f \quad \land \quad id_a \cdot f = f$

And if we see our objects as sets and our morphisms as functions, suddenly we have our theoretical underpinnings of function composition!

These ideas are extremely easy to express in Haskell:

```
-- given the twop mappings f and g
f :: b -> c
g :: a -> b

composition :: (b -> c) -> (a -> b) -> a -> c
composition = \x -> f (g x)

-- In Haskell there is a composition operator (.)
(.) :: (b -> c) -> (a -> b) -> a -> c
(f . g) x = f (g x)

-- given mapping h 
h :: c -> d

associativity = h . f . g 
-- you put the brackets where you want or just leave them out

id :: a -> a 
id x = x 
```

In Python we can do the same quite easily:

```
from typing import Callable, Collection
from functools import reduce

def comp(f: Callable, g: Callable) -> Callable: 
    return lambda *a, **kw: g(f(*a, **kw))

def compose(*fs: Collection[Callable])-> Callable:
    return reduce(comp, fs)

def h(x): return x+1
def g(x): return x+2
def f(x): return x+3

cf = compose(h,g,f)
# will result in 9
cf(3)

def identity(x: Any) -> Any:
    return x
```








