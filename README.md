# py3streams - manipulate collections

Package contains classes to support operations on collections and container object to control data flow.
*Stream*, *IntStream*, *DictStream* manipulate lists, tuples, dicts, ranges and other streams with chain of generators.
*Optional* object is a container which can hold other object. With defined methods *Optional* helps manage the object and control flow if its exists or not, return default value in case of None or empty, or execute lambda function using stored object if stored object exists (if_present(lambda)).

## Table of contents
* [Description](#Description)
* [Installation](#Installation)
* [Usage](#Usage)
* [Build-in functions](#Build-in functions)

## Description
Classes from the package are: **Stream**, **IntStream** and **DictStream**. Classes may help with manipulate nested lists or dicts, filter values with build-in object functions or map functions on the values. There are few cases which is very important to understand:
* classes are not collections but they are iterable,
* methods which manipulate values (*filter*, *map*, *fmap*) are stored in chain of generators and wait for invoke,
* Streams cannot be reused and stored chain of generators can be evaluated only once,
* Streams are invoked during iteration with *for-loop* or with methods which build collection from them (*to_list()*, *to_dict()*), evaluate if any value match something (*any_match()* or *all_match()*) or check *sum()*, *count()*, *max()* etc.

**Optional** object is a container which may hold value (any other object) and helps with manipulate it. 
* *get()* method from *Optional* object will return stored value. *get_or_else(default)* will return if stored value is not None and exists, otherwise return default,
* *is_present()* allows check if stored data is not None and exists,
* *if_present(lambda)* executes lambda function on stored data if its exists. Method will return result from lambda, or None if nothing was executed.

For example the *Optional* object is returned during *Stream* operation with method: *get_first()*.

Lets define example list which will be used in few next examples:
```
from py3streams import Stream

example_list = ["1", "5", 2, "10", 11]
```
#### Example: sum values from collection.
```
sum = Stream(example_list).map(lambda x: int(x)).filter(lambda x: x < 10).sum()
# result: [1, 5, 2] -> 8
```
Alternative behaviour with build-in Stream's methods:
```
sum = Stream(example_list).map_to_int().lt(10).sum()
```

In the example class *Stream* has been used on the *example_list*. First there is registered generator for changing strings to ints, then registered generator with lambda expression for elements lower than *10* and sum them.

Streams can be used with *for-loop*:
```
for i in Stream(example_list).map_to_int().lt(10).sum():
    print(i)
# results are 1, 5 and 2.
```

## Installaton
To use the streams install the package.
```
pip3 install py3streams
```

## Usage
Streams help manipulate collections. For most actions its enough to use filter(), map() and fmap() methods.
Classes contain build-in functions which include lambdas and allow use short-name methods for similar result.

#### Example 1: filter elements.

Lets define a list:
```
elements = [0, 1, "2", 3, None, [5, 6, "7"], ["8"]]
```
Stream will find elements lower than 3. Sub-lists and None values should be ignored.
```
stream = Stream(elements).filter(lambda x: x is not None).filter(lambda x: not isinstance(x, list)).map(lambda x: int(x)).filter(lambda x: x < 3) # still we have a stream 
for e in stream:
    print(e)
# result 0, 1 and 2
```
and alternative with Stream's functions:
```
stream = Stream(elements).no_list().no_none().map_to_int().lt(3)
for e in stream:
    print(e)
# result 0, 1 and 2
```

#### Example 2: get_first() with *Optional* object.

Lets define a list:
```
elements = ["a", "b", 3, 4]
```
Stream will find first int.
```
opt = Stream(elements).filter(lambda x: isinstance(x, int)).get_first()
# opt is an Optional object with stored value=3.
print(opt.get())
# print-out: 3
```
*get_first(default_value=None)* returns None if Stream does not find element by default.
```
opt = Stream(elements).filter(lambda x: isinstance(x, list)).get_first()
# opt is an Optional object. Value does not exists in Stream after apply filters. Stored value in Optional Object is **None**.
print(opt.get_or_else("another-value"))
# print-out: another-value
```
for short example, lets apply lambda function with print() if value exists.
```
Stream(elements).filter(lambda x: isinstance(x, list)).get_first().if_present(lambda x: print(x))
# result: nothing will be printed, because stream does not have elements after filter apply
```

#### Example 3: in-line nested lists with *fmap()*

Lets define a list:
```
elements = [[1,2,3], [4], "val1", "val2", [5,6,7]]
```
Filter list elements and merge them.
```
nested_elements = Stream(elements).filter(lambda x: isinstance(x, list)).fmap(lambda x: Stream(x)).to_list()
# result: [1,2,3,4,5,6,7]
```

## Build-in functions
Lambda mechanism can be replaced with few build-in methods which contains lambdas inside and hide them from the user.
Alternatives work **only** with **Stream** class.

| classic filter/map | build-in alternative | Comments |
|-------------------:|:---------------------|----------|
| map(lambda x: str(x))| map_to_str() | change elements to str |
| map(lambda x: int(x))| map_to_int() | change elements to int |
| map(lambda x: float(x)) | map_to_float() | change elements to float |
| filter(lambda x: isinstance(x, dict)) | only_dict() | select only dicts |
| filter(lambda x: isinstance(x, (list, set, tuple))) | only_list() | select only lists |
| filter(lambda x: not isinstance(x, (list, set, tuple)))| no_list() | select all except lists |
| filter(lambda x: x is not None) | no_none() | select all (includes zeros, empty string:"" and empty lists:[]) except None |
| filter(lambda x: x) | exists() | elements which exists and are not empty |
| filter(lambda x: x%2 == 0) | even() | even numbers |
| filter(lambda x: (x%2 - 1) == 0) | odd() | odd numbers |
| filter(lambda x: x > value) | gt(value: int) | greater than |
| filter(lambda x: x < value) | lt(value: int) | less than |
| filter(lambda x: x >= value) | ge(value: int) | greater and equal |
| filter(lambda x: x <= value) | le(value: int) | less and equal |
| filter(lambda x: x == value) | eq(value: object) | equal to | 

