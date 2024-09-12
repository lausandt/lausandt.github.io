Title:  Functors in Python 
Date: 2024-08-07
Modified: 2024-08-07
Author: Lau Sandt
Summary: A look at how to implement Functors in Python

---
If you are like me and love to program both in Python and in Haskell then you might stumble on some attempts to implement a Functor in Haskell. Take as example the Functor class below, adopted slightly from [Arjan Codes](https://www.arjancodes.com/blog/python-functors-and-monads/), I added some code to keep it simple and make the example work, so you can follow along. 



```python
from typing import Generic, TypeVar, Callable
from operator import pow, add 
from functools import partial

T = TypeVar('T')
U = TypeVar('U')

class Functor(Generic[T]):
    def __init__(self, value: T):
        self.value = value

    def map(self, f: Callable[[T], U]) -> "Functor[U]":
        return Functor(f(self.value))

Functor(1).map(partial(add, 1)).map(partial(pow,2)).value 

```
I have four problems with this example:

1. The first problem I have is with the stated goal of the Functor; Functors allow for clean, concise pipelines. No that is not the goal of a Functor. Functors allow for mappings between categories, that is also precisely the goal of the Functor, to map between to categories. Okay that doesn't explain a lot but we will get back to this shortly.
2. You cannot really map on an integer as it is a primitive type and mapping is done on wrapped types.
3. A functor should not be implemented as a class but as an interface.
4. What they say is a Monad simply is not.

Before I will explain you why these are problems, I first want to say that this article in no manner whatsoever suggest the [Arjan Codes](https://www.arjancodes.com/blog/python-functors-and-monads/) is bad, it is probably excellent and worth your attention, I have no opinion on the content of this site other than, on this article. 

To understand what a Functor does we should start with the `map` function, that represents a mapping in Python. 


```python
def succ(n: int) -> int: 
    return n + 1

list(map(succ, [1,2,3,4,5])) ==   [2, 3, 4, 5, 6]
``` 
What does the above map function do? It maps the successor function from the domain {1,2,3,4,5} to   
the codomain {2,3,4,5,6}. The application of such a function to all elements in a domain to elements in a codomain is called a mapping. Pictorial it is this familiar drawing:

![domain / codomain](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Codomain2.SVG/500px-Codomain2.SVG.png)

Which we probably all know from middle school math classes. Hence that we can not map succ to 1, we do not map from an element but from a collection of elements, usually in maths a set, in Python an iterable. The map function allows you to operate on elements of an underlying data structure without changing that data structure. This is exactly how a functor should be understood, as allowing a map function to operate without changing the underlying structure.


```python
tuple(map(succ, (1,)) ==  (2,))
```

The map function in Python is limited to iterables. What is an iterable in Python? Anything that has a next function and a StopIteration error, which is Python's manner to answer the question hasNext? 

If we consider an iterable to wrap its elements in a list, set, or whatever iterable you can imagine, then we could ask ourselves the question how can we use map on a with a structure wrapped element, where that structure is not an iterable?

Consider the following example of map not working:  


```python
d = dict(Arjan=1,Laurens=2)
dict(map(succ,d))
```

    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    /tmp/ipykernel_15923/1178047909.py in <module>
          1 d = dict(Arjan=1,Laurens=2)
    ----> 2 dict(map(succ,d))
    

    /tmp/ipykernel_15923/924292918.py in succ(n)
          1 def succ(n: int) -> int:
    ----> 2     return n + 1
          3 
          4 list(map(succ, [1,2,3,4,5]))


    TypeError: can only concatenate str (not "int") to str



```python
{k:succ(v) for k,v in d.items()} ==  {'Arjan': 2, 'Laurens': 3}
```

As you can see to be able to add to the value in the dictionary requires in Python to unpack the items in the dictionary and then apply the function on the value. We cannot map directly on a dict. To be able to do so requires a more generic map function, and that is what a functor is. A functor for all intents and purposes is a generalisation of map, it generalises on the structure that holds the element. 

The most common example to illustrate what a functor can do is with a Maybe type, [Arjan Codes](https://www.arjancodes.com/blog/python-functors-and-monads/). also uses the maybe as an example but it is more instructive use a bit of Haskell, which knows the Maybe type. 

```
Maybe a = Nothing | Just a
```
This you should interpret pretty must as you read it. We have Maybe value which wraps an unspecified type, a. That Maybe value consists of one of two things:

1. Nothing
2. Just a value of that type

For instance we can have Maybe Int. 

```
Nothing
Just 42
```

Now coming back to our functor how can we generalise our map function to also be able to use map our map function on it. Remember we want to apply a function to the element and not the structure. Haskell's answer to this problem is to use a Java like interface (akin to an abstract base class in Python), in Haskell an interface is called a typeclass. We declare the Maybe type to be an instance of the Functor typeclass. 

```
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

fmap here is what in Python we call an abstract method, meaning there is no adjoining implementation. For the Maybe type to be an instance (implementing / subclassing) the typeclass as in Python or Java we would have to give a meaning to fmap for the Maybe type. In Haskell:

```
instance Functor Maybe where
  fmap f Nothing = Nothing
  fmap f (Just a)  = Just (f a)
```
Again Haskell so just a bit of clarification. It says for the Maybe type if fmap a function (f) on a Nothing value, we get a Nothing back. If we fmap function f on the (Just a) value we get a Just value back and we apply f on the wrapped value a. If you would run the following in GHCI (the Haskell interpreter, also available online [Try Haskell](https://www.tryhaskell.org/) you will get the following.

```
fmap (+1) (Just 41) = Just 42
```

A Haskell list implements the Functor typeclass. So I want to use fmap to map over a list I can. 

```
fmap (+1) [1..5] = [2,3,4,5,6]
```

For those curious the list implementation of the Functor type class is:

```
instance Functor [] where
  fmap f [] = []
  fmap f (a:as) = f a : fmap f as
```

Haskell uses a lot of pattern matching (Python is starting to) but it basically says if I fmap f on an empty list I get that list back, if I fmap f on a list represenred as `(a:as)` then I apply the function to a and cons that on a recursive call to fmap. If you do not understand that sentence, do not worry we are programming in Python, not Haskell.

Let's see if we can implement this in Python:


```python
from abc import ABC, abstractmethod


class Maybe(ABC, Generic[T]):
    '''I am ducktyping the functor, see the explanation below'''

    @abstractmethod
    def fmap(self):
        ...

class Just(Maybe[T]):

    def __init__(self, value: T): 
        self.value = value

    def fmap(self, func: Callable[[T], U]) -> Maybe[U]:
        try:
            new_value = func(self.value)
            return Just(new_value)
        except Exception as e:
            print(e)
            return Nothing()

    def unwrap(self) -> U:
        return self.value


class Nothing(Maybe[T]): 

    def fmap(self, func: Callable[[T], U]) -> Maybe[U]: 
        return self
```


```python
Just(41).fmap(partial(add, 1)).unwrap() == 42
```

```python
Just(42).fmap(str).unwrap() == "42"
```
Now we have created a Maybe type that is a Functor, both the Just and Nothing classes subclass the Maybe. 

You might wonder why it is enough to implement only fmap for something to be a functor. This is due to the mathetical definition of a Functor, which I now am going to give but feel free skip it. 

Let *C* and *D* be categories. A functor $F: A \rightarrow B$ associates:

- to every object $x \in A$ an object $F x \in B$
- to every morphism $f: X \rightarrow Y \in A$ a morphism $ F(f): F(X) \rightarrow F(Y) \in B$ 

Such that the following conditions hold:

1. $F(ID_x) = ID_F(x)$ for every object $x \in A$ 
2. $F(g \circ f) = F(g) \circ F(f)$ for all morphisms $ f: X \rightarrow Y$ and $g: Y \rightarrow Z \in B$

Pictures speak more than words so ![Functor](https://nikgrozev.com/images/blog/Functional%20Programming%20and%20Category%20Theory%20Part%201%20-%20Categories%20and%20Functors/functor.jpg)

A and B are the categories, F is the functor. We only need to be able to map from A to B to be a functor all other we can derive from the fact that A and B are categories.  

But now of course you want a use case. For that I will introduce another Functor we can use to propagate an error message through a pipeline, for instance in an API. 


```python
class Either(ABC, Generic[T]):

    @abstractmethod
    def fmap(self):
        ...

class Right(Either[T]): 

    def __init__(self, value: T): 
        self.value: T = value 

    def fmap(self, func: Callable[[T], U]) -> Either[U]:
        try:
            new_val = func(self.value)
            return Right(new_val)
        except Exception as e:
            return Left(e.args[0]) 

    def unwrap(self) -> T:
        return self.value

class Left(Either[T]): 

    def __init__(self, exc: str): 
        self.exc:str = exc 

    def fmap(self, func: Callable[[T], U]) -> Either[str]:
        return Left(self.exc)

    def unwrap(self) -> str:
        return self.exc

```

Now we can propagate the error through our pipeline and not have it fail. Furthermore at the end of our pipeline we can unwrap our value for useful inspection. These types of continuation even after failure is very useful in amongst other things web api's.     


```python
Right(42).fmap(int).fmap(lambda x : x / 0).fmap(str).fmap(int).unwrap() ==  'division by zero'
```
To answer the last problem with the code why their Maybe class is not a monad is because for anything to be a Monad we need at least to implement the bind function. No bind function no monad. 

```python
def bind(self, func: Callable[[T], Maybe[U]]) -> Maybe[U]:
    try:
        new_val = func(self.val)
        return new_val
    except Exception as e:
        return Nothing()

```
