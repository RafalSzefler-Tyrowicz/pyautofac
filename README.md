pyautofac
=========

Lightweight thread safe container builder inspired by the excelent C#
library Autofac. The aim of the lib is to simplify dependencies between
classes.

Requirements
============

Python3.5+ (or whichever version that supports annotations)

Usage
=====

Assume that we have few dependent classes:

```
class Foo:
    def __init__(self, value):
        self.value = value


class Bar:
    def __init__(self, foo: Foo):
        self.foo = foo

    def square(self):
        return self.foo.value ** 2


class Zoo:
    def __init__(self, bar: Bar):
        self.bar = bar

    def increment(self):
        return self.bar.square() + 1
```

usually we would do something like this in order to work with them:

```
foo = Foo(2)
bar = Bar(foo)
zoo = Zoo(bar)
print(zoo.increment())
# 5
```

with pyautofac you create a container which tracks these dependencies
automatically:

```
from pyautofac import ContainerBuilder
builder = ContainerBuilder()
builder.register_instance(Foo(2))
builder.register_class(Bar)
builder.register_class(Zoo)
container = builder.build()
zoo = container.resolve(Zoo)
print(zoo.increment())
# 5
```

While it looks like more code it saves lots of time when dealing with
complicated dependencies.

More info
=========

By default `pyautofac` instantiates each class passed to `register_class`
as a singleton.


When you use `register_class` then `pyautofac` requires each
parameter (except `self`) in the constructor to be annotated. Class like this:

```
class Test:
    def __init__(self, foo):
        pass
```

will fail during `resolve` because `pyautofac` won't know what to
do with the `foo` argument. 

However if you want to register instance of class `Test` it is doable
via `register_instance`. In that case you have to create the instance
manually though.

It is also possible to register a class as an implementation of an interface.
For example assume that we have an interface

```
class SomeInterface:
    def call_me(self):
        raise NotImplementedError()


class ConcreteImplementation(SomeInterface):
    def call_me(self):
        print(1)
```

you can the register it:

```
builder.register_class(ConcreteImplementation).as_interface(SomeInterface)
```

and then resolving will work correctly:

```
impl = container.resolve(SomeInterface)
impl.call_me()
# 1
```