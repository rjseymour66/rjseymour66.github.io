---
title: "Python"
linkTitle: "Python"
# weight: 4
description: >
  Notes about basic Python.
---

# Toolset

## venv

### Create virtual env

```bash
$ python3 -m venv ./<venv-name>
# Ex: python3 -m venv ./venv
```

### start venv
```bash
$ source ./<venv-name>/bin/activate # you don't need the './' before the command, but the tutorial used it
```

### exit venv
```bash
$ deactivate
```

## pip

1. Run pip show on every package that you want to uninstall so you know that it doesn't depend on any package that you plan to uninstall
2. pip uninstall <package-name> -y

### Commands
```bash
$ pip list
$ pip show <package-name>
$ pip uninstall <package-name>
```

## requirements.txt

Requirements file lists all of the program's dependencies. You can pass this program to pip.

1. To get all of the program's dependencies, enter pip freeze > requirements.txt
2. Verify with cat requirements.txt
3. Go to your new venv enter pip install -r requirements.txt to install every package listed in tthe file. 

Manually change requirements.txt to update package version requirements.

```
== means the exact version
>= means version greater than or equal to 
, != 1.0.1 means there is a restriction. Put those after a comma in the version
Ex: pyvim>=3.0.2, <= 4.0.0 or , != 3.0.6
```

To upgrade new requirements, enter pip install --upgrade -r requirements.txt

### Dev requirements files

requirements_dev.txt

```bash
$ -r requirements.txt # copies regular requirements.txt files into this 
prod-packageName>=5.3.5
```

requirements_locked.txt is a copy of requirements.txt
	- special version that has locked down, known good dependencies

### Types of req files
	requirements.txt	base level file
	requirements_dev.txt	dev env
	requirements_locked.txt	prod env

# Project setup

# Basics

## Operators

```python
=       # assignment
==      # equality (not memory)

# Identity
is      # same object with same memory location
is not  # different objects with same memory location

# Membership
in      # True if x in y
not in  # True if x not in y

# Logical
and
or
not

# Boolean
True
False
```

To test equality, use `==`.

## if, elif, else

```python
name = 'Smith'

if name == 'Nelson':
    print('This is wrong')
elif name == 'Douglas':
    print('This is wrong')
elif name == 'Smith':
    print('This is right')
else:
    print('No answer')
```

# Strings

```python
# Use three single or double quotes for multi-line strings
song = '''Happy birthday to you,
Happy birthday to you,
Happy birthday dear person,
Happy birthday to you!'''

# create a string out of another data type
str(9.87)
song.startswith('Happy')    # True
song.endswith('everyone!')  # False

# Find the offset of the first occurrence (returns -1 if not found)
word = 'to'
song.find(word)
15

# last occurrence (returns -1 if not found)
song.rfind(word)
89

# number of occurrences
song.count(word)
3

# all chars letters or numbers?
song.isalnum()
False

# string case funcs
phrase.strip(' .')
'you win some'
phrase = 'you win some ...'
phrase.capitalize()
'You win some ...'
phrase.title()
'You Win Some ...'
phrase.upper()
'YOU WIN SOME ...'
phrase.lower()
'you win some ...'
phrase.swapcase()
'YOU WIN SOME ...'

# multiply strings
name = 'Jack'
name * 4

# 'a', get char from string
name[1]
```
## Substring with a slice

```python
offsets = '0123456789'

# [:] extracts the entire sequence from start to end
offsets[:]  # 0123456789

# [start:] from the start offset to the end, inclusive
offsets[5:]     # 56789

# [:end] from the beginning to the end offset -1 (non-inclusive)
offsets[:5]     # 01234

# [start:end] from the start offset to the end offset - 1 (end is non-inclusive)
offsets[2:7]    # 23456

# [start:end:step] from the start offset to the end offset - 1 (end is non-inclusive), skipping characters by step
offsets[2:7:2]  # 246
```
## Built-in functions

```python
offsets = '0123456789'

len(offsets)        # 10
```
### split()

```python
tasks = 'one,two,three,four'
# if you don't provide a separator, split() uses anything that makes sense
tasks.split(',')    # ['one', 'two', 'three', 'four']
tasks.split()       # ['one,two,three,four']
```

### join()
join() collapses a list of strings into a single string. Use the 
following syntax:
<separator>.join(list_of_strings)

```python
str_list = ['one', 'two', 'three', 'four']
'\n'.join(str_list)
'one\ntwo\nthree\nfour'

' '.join(str_list)
'one two three four'
```

### replace() 

replace() does simple substring substitution

```python
sub = 'This is the dumbest string ever'
sub.replace('dumb', 'cool')
'This is the coolest string ever'
```
### Formatting strings

```python

animal = 'dog'
place = 'house'

# {} and format()
'The {} is in the {}.'.format(animal, place)
'The dog is in the house.'
'The {a} is in the {b}.'.format(a='cat', b='litter box')
'The cat is in the litter box.'

# f-strings
f'The {animal} is in the {place}'
'The dog is in the house'
f'The {animal.capitalize()} is in the {place.rjust(20)}'
'The Dog is in the                house'
```

# Loops

## while loop

```python
count = 0
while count < 5:
     print(count)
     count += 1 
0
1
2
3
4
```

## for and in with iterators

Iterators allow you to traverse a data structure without knowing how loarge they are or how they are implemented. They include:
- strings
- lists
- tuples
- dictionaries

```python
# for element in interable:
#   do something

sentence = 'A string is an iterable'
for letter in sentence:
     print(letter)
 
A
 
s
t
r
i
n
g
 
i
...
```

## range() to generate numbers

`range()` returns a stream of numbers within a specified range without having to use memory on a data structure. `range()` returns in iterable object that you can step through with a `for ... in` loop or convert to a sequence, such as a list.

`range(start: stop: step)`

```python

# standard usage
for x in range(0, 5):
     print(x) 
0
1
2
3
4

# create a list
list(range(0, 5))
[0, 1, 2, 3, 4]

# count down with a step
for x in range(5, -1, -1):
     print(x)
 
5
4
3
2
1
0

# create a list
list((5, -1, -1))
[5, -1, -1]
```

# Tuples

Tuples and lists can both contain zero or more Python objects of any type. 

## Creating a tuple 

Use parentheses or a ',':

```python
# parens
> empty_tuple = ()
> empty_tuple
()

# string followed by ','
> one_stooge = 'Larry',
> one_stooge
('Larry',)

# for multiple items, do not add a ',' after the last one
> all_stooges = 'Larry', 'Curly', 'Moe'
> all_stooges
('Larry', 'Curly', 'Moe')

# parens make it easier to read
> parens_stooges = ('Larry', 'Curly', 'Moe')
> parens_stooges
('Larry', 'Curly', 'Moe')
```


## Tuple actions

When you assign more than one tuple value at the same time:

```python
> parens_stooges = ('Larry', 'Curly', 'Moe')
> parens_stooges
('Larry', 'Curly', 'Moe')
> a, b, c = all_stooges
> a
'Larry'
> b
'Curly'
> c
'Moe'

# swap vals without creating a temp var
> one = 'one'
> two = 'two'
> one, two = two, one
> one
'two'
> two
'one'

# Tuple from a list
> stooge_list = ['Larry', 'Curly', 'Moe']
> tuple(stooge_list)
('Larry', 'Curly', 'Moe')

# Combine tuples with '+'
# Combine with a '+'. Note the ',' after the item in the single-value tuple:

> ('Larry',) + ('Curly', 'Moe')
('Larry', 'Curly', 'Moe')

# Duplicate items in tuple 
> ('howdy',) * 5
('howdy', 'howdy', 'howdy', 'howdy', 'howdy')

# Compare tuples
> x = (1, 2, 3)
> y = (2, 3, 4)
> x == y
False
> x < y
True
> x <= y
True

# Modify a tuple
# When you concatenate a tuple with another tuple, it creates a new Python object
> first = ('one', 'two', 'three')
> second = ('four',)
> third = first + second
> third
('one', 'two', 'three', 'four')
> id(first)
2049613520640
> id(second)
2049618746960
> id(third)
2049614519888
```
## Iterate through a tuple

Use for and in to iterate through a tuple:

```python 
> nums
('one', 'two', 'three', 'four')
> for n in nums:
...     print(n)
...
one
two
three
four
```

# Lists

Use lists to keep track of ordered items, especially when the order might change. Lists are mutable.

## Creating a list

```python

# Create an empty list with brackets
> empty_list = []
> empty_list
[]

# Empty list with list() func
> empty_too = list()
> empty_too
[]

# List with values
> beatles = ['John', 'Paul', 'George', 'Ringo']
> beatles
['John', 'Paul', 'George', 'Ringo']


# Create list out of items in an iterable
> list('individual')
['i', 'n', 'd', 'i', 'v', 'i', 'd', 'u', 'a', 'l']

# List from a tuple with list()
> tup = ('one', 'two', 'three')
> list(tup)
['one', 'two', 'three']

# list using string.split() func
> adj = 'once-in-a-lifetime'
> adj.split('-')
['once', 'in', 'a', 'lifetime']
```

## Getting list items

```python
> turtles = ['Leonardo', 'Donatello', 'Michaelangelo', 'Raphael']

# Get with an offset
> turtles[2]
'Michaelangelo'

# Get with a slice
> turtles[0:2]
['Leonardo', 'Donatello']
```

## List functions

```python
> nums = ['one', 'two', 'three', 'four']
> nums
['one', 'two', 'three', 'four']

# append to the end
> nums.append('five')
> nums
['one', 'two', 'three', 'four', 'five']

# insert at an index
> nums.insert(0, 'zero')
> nums
['zero', 'one', 'two', 'three', 'four', 'five']

# multiply a list
> ['blah'] * 3
['blah', 'blah', 'blah']

# combine lists
> numeros = ['uno', 'dos', 'tres']
> nums.extend(numeros)
> nums
['zero', 'one', 'two', 'three', 'four', 'five', 'uno', 'dos', 'tres']

# edit by index
> nums[2] = 'too'
> nums
['zero', 'one', 'too', 'three', 'four', 'five', 'uno', 'dos', 'tres']

# edit by slice
> nums[6:] = ['six', 'seven', 'eight']
> nums
['zero', 'one', 'too', 'three', 'four', 'five', 'six', 'seven', 'eight']

# delete by index
> del nums[0]
> nums
['one', 'too', 'three', 'four', 'five', 'six', 'seven', 'eight']

# delete by value
> nums.remove('seven')
> nums
['one', 'too', 'three', 'four', 'five', 'six', 'eight']

# remove from end of list
> nums.pop()
'eight'
> nums
['one', 'too', 'three', 'four', 'five', 'six']

# remove by index
> nums.pop(1)
'too'
> nums
['one', 'three', 'four', 'five', 'six']

# delete every item in the list
> nums.clear()
> nums
[]

# get index by value
> nums.index('two')
1

# use 'in' to check if in list
> 'three' in nums
True
> 'four' in nums
True
> 'nine' in nums
False


# get number of occurences by value
> nums
['one', 'two', 'three', 'four', 'two']
> nums.count('two')
2

# convert to string with a delimiter
> ', '.join(nums)
'one, two, three, four, two'

# get the length
> len(nums)
5

# sort with sorted(list)
> alpha_nums = sorted(nums)
> alpha_nums
['four', 'one', 'three', 'two', 'two']

# sort with .sort()
> nums
['one', 'two', 'three', 'four', 'two']
> nums.sort()
> nums
['four', 'one', 'three', 'two', 'two']

# copy and compare lists
> a = nums
> a
['four', 'one', 'three', 'two', 'two']

# not a copy, assigned same object to two different variables
> b = a
> b
['four', 'one', 'three', 'two', 'two']
> id(b)
2660463301888
> id(a)
2660463301888
> c = a.copy()
> id(c)
2660462605824
> d = list(c)
> id(d)
2660463342336
> e = d[:]
> id(d)
2660463342336
> a = b
> a == b
True
> a == c
True
> a == d
True
> a == e
True
```

## Iterating through lists 

```python

# for and in to iterate through a list
> ls = ['one', 'two', 'three', 'four', 'five']
> for i in ls:
...     print(i)
...
one
two
three
four
five
> es = ['uno', 'dos', 'tres', 'quatro', 'cinco']
> es
['uno', 'dos', 'tres', 'quatro', 'cinco']

# zip to iterate through multiple lists simultaneously
> for english, spanish in zip(ls, es):
...     print(english, ":\t", spanish)
...
one :    uno
two :    dos
three :  tres
four :   quatro
five :   cinco
```

## List comprehensions

List comprehensions are a way to build a list. It uses the following format:

[`expression` for `item` in `iterable`]

It moves the loop into the brackets to create a list out of what the expression returns for each item in the list. This would otherwise be written as follows:

for `item` in `iterable`:
    `expression`

```python
> ls = ['one', 'two', 'three']

# basic list comprehensions
> upper = [item.capitalize() for item in ls]
> upper
['One', 'Two', 'Three']

> num_ls = [n for n in range(0, 10)]
> num_ls
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# expression
> index_adjusted = [n + 1 for n in num_ls]
> index_adjusted
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

Conditional list comprehensions add a condition after the iterable:

[`expression` for `item` in `iterable` if `condition`]

This is equal to the following:

for `item` in `iterable`:
    if (`condition`):
        `expression`

```python
> animals = ['cat', 'dog', 'mouse', 'rat', 'bird']
> three = [a for a in animals if len(a) == 3]
> three
['cat', 'dog', 'rat']
> Caps = [a.capitalize() for a in animals if len(a) != 3]
> Caps
['Mouse', 'Bird']
```

Create nested list comprehensions by listing the conditions one after the other:

```python

# standard nested loop
> nums = range(1, 4)
> nums
range(1, 4)
> alpha = ['x', 'y']
> for num in nums:
...     for a in alpha:
...             print(num, a)
...
1 x
1 y
2 x
2 y
3 x
3 y

# same loop as a list comprehension
> ls_comp = [(num, a) for num in nums for a in alpha]
> for x in ls_comp:
...     print(x)
...
(1, 'x')
(1, 'y')
(2, 'x')
(2, 'y')
(3, 'x')
(3, 'y')

# tuple unpacking
> for (num, a) in ls_comp:
...     print(num, a)
...
1 x
1 y
2 x
2 y
3 x
3 y
```

## Lists of lists 

Exactly what it sounds like:

```python
> evens = [2, 4, 6, 8, 10]
> odds = [1, 3, 5, 7, 9]
> prime = [1, 7, 13, 19, 23]
> nums = [evens, odds, prime]
> nums
[[2, 4, 6, 8, 10], [1, 3, 5, 7, 9], [1, 7, 13, 19, 23]]

```

# Dictionaries

Dictionaries are key/value pairs. Keys are often strings, but they can be any value.

## Creating dictionaries

```python
# with {}
> empty_dict = {}
> empty_dict
{}
> beatles = {
...     "John": "Lennon",
...     "Paul": "McCartney",
...     "George": "Harrison",
...     "Ringo": "Starr",
...     }
> beatles
{'John': 'Lennon', 'Paul': 'McCartney', 'George': 'Harrison', 'Ringo': 'Starr'}

# Create with dict()
> poet = dict(first='Edgar', middle='Alan', last='Poe')
> poet
{'first': 'Edgar', 'middle': 'Alan', 'last': 'Poe'}

```
You can convert two-value sequences into a dictionary:

```python
# two-item tuples
> tups = [('a', 'z'), ('b', 'y'), ('c', 'x')]
> dict(tups)
{'a': 'z', 'b': 'y', 'c': 'x'}

# tuple of two-item lists
> tup_of_lis = (['a', 'z'], ['b', 'y'], ['c', 'x'])
> dict(tup_of_lis)
{'a': 'z', 'b': 'y', 'c': 'x'}
> two_char_str = ['ab', 'cd', 'ef']

# two-char strings
> dict(two_char_str)
{'a': 'b', 'c': 'd', 'e': 'f'}

```

## Adding or changing an item by [key]

If there is no item with the key, then it is added. If there is, then it is replaced:

```python
> beatles = {
...     'Lennon': 'John',
...     'McCartney': 'Paul',
...     'Harrison': 'George',
...     'Starr': 'Ringo',
...     }
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo'}

# Add an item
> beatles['Preston'] = 'Billy'
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy'}

# Add, then replace an item
> beatles['Clapton'] = 'Erik'
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy', 'Clapton': 'Erik'}
> beatles['Clapton'] = 'Eric'
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy', 'Clapton': 'Eric'}
```

## Getting items 

Use .get() or .keys() to return items from the dictionary:

```python
# get by key
> beatles.get('Martin')
> beatles.get('Martin', 'Not in the Beatles')
'Not in the Beatles'
> beatles.get('Harrison')
'George'

# get all keys in the dict
> beatles.keys()
dict_keys(['Lennon', 'McCartney', 'Harrison', 'Starr', 'Preston', 'Clapton'])

# get all values in the dict
> beatles.values()
dict_values(['John', 'Paul', 'George', 'Ringo', 'Billy', 'Eric'])

# store them in a list
> list(beatles.values())
['John', 'Paul', 'George', 'Ringo', 'Billy', 'Eric']

# get the items as a tuple
> list(beatles.items())
[('Lennon', 'John'), ('McCartney', 'Paul'), ('Harrison', 'George'), ('Starr', 'Ringo'), ('Preston', 'Billy'), ('Clapton', 'Eric')]

# length
> len(beatles)
6
> stones = {
...     'Jagger': 'Mick',
...     'Richards': 'Keith',
...     'Watts': 'Charlie',
...     'Wyman': 'Bill'
... ,}
> stones
{'Jagger': 'Mick', 'Richards': 'Keith', 'Watts': 'Charlie', 'Wyman': 'Bill'}

# combine dicts with **
> supergroup = {**beatles, **stones}
> supergroup
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy', 'Clapton': 'Eric', 'Jagger': 'Mick', 'Richards': 'Keith', 'Watts': 'Charlie', 'Wyman': 'Bill'}
> ruttles = {}

# combine dicts with update()
> ruttles.update(beatles)
> ruttles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy', 'Clapton': 'Eric'}

# delete an item by key
> del beatles['Preston']
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Clapton': 'Eric'}
> del beatles['Clapton']
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo'}

# remove an item with .pop()
> beatles.pop('Martin')
'George'
> beatles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo'}

# delete everything in the dict with .clear()
> ruttles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo', 'Preston': 'Billy', 'Clapton': 'Eric'}
> ruttles.clear()
> ruttles
{}

# check for a value with 'in'
> 'Lennon' in beatles
True

# copy()
> ruttles = beatles.copy()
> ruttles
{'Lennon': 'John', 'McCartney': 'Paul', 'Harrison': 'George', 'Starr': 'Ringo'}

# check equality
> beatles == ruttles
True

# for in loops print key by default
> for guy in beatles:
...     print(guy)
... 
Lennon
McCartney
Harrison
Starr

# use .values() to get values
> for guy in beatles.values():
...     print(guy)
... 
John
Paul
George
Ringo

# use items() to get a tuple for each dict entry
> for guy in beatles.items():
...     print(guy)
... 
('Lennon', 'John')
('McCartney', 'Paul')
('Harrison', 'George')
('Starr', 'Ringo')

# assign var names to k/v pairs
> for last, first in beatles.items():
...     print(first, '\'s last name is ', last)
... 
John 's last name is  Lennon
Paul 's last name is  McCartney
George 's last name is  Harrison
Ringo 's last name is  Starr
```
## Dictionary comprehensions

Like lists, dictionaries have comprehension that use the following format:

{`key_expression` : `value_expression` for `expression` in `iterable`}

The following example creates a dictionary where each letter in `word` is a key, and its value is the number of occurrences of each key:
```python
> word = 'better'
# for every letter in word, create a k/v pair 
> better_count = {letter: word.count(letter) for letter in word}
> better_count
{'b': 1, 'e': 2, 't': 2, 'r': 1}

```

Dictionary comprehensions with conditions using the following format:

{`key_expression` : `value_expression` for `expression` in `iterable` if `condition`}

```python
> vowels = 'aeiou'
> word = 'superpower'
> vowel_counts = {letter: word.count(letter) for letter in set(word) if letter in vowels}
> vowel_counts
{'o': 1, 'u': 1, 'e': 2}
```

# Sets

A set is like a dictionary with only keys. Each key must be unique.

## Creating sets

Sets are unordered:

```python
> empty_set = set()
> empty_set
set()
> evens = {2, 4, 6, 8}
> evens
{8, 2, 4, 6}
> odds = {1, 3, 5, 7, 9}
> odds
{1, 3, 5, 7, 9}

# Convert data structures to sets

> set('letters')
{'e', 's', 't', 'r', 'l'}
> set(['Leonardo', 'Donatello', 'Raphael', 'Michaelangeo'])
{'Leonardo', 'Michaelangeo', 'Donatello', 'Raphael'}
> set( ('dog', 'cat', 'fish') )
{'cat', 'fish', 'dog'}

# Only uses keys from dictionaries
> set( {'John': 'Lennon', 'Paul': 'McCartney', 'George': 'Harrison', 'Ringo': 'Starr'} )
{'John', 'George', 'Ringo', 'Paul'}

```

## Set functions

```python
> evens
{8, 2, 4, 6}
> len(evens)
4
> evens.add(10)
> evens
{2, 4, 6, 8, 10}
> evens.remove(10)
> evens
{2, 4, 6, 8}
```
## Iterate and test with for and in

Sets are often nested, like the following example that is an example of a pizza menu:

```python
> menu = {
    'classic': {'pepperoni', 'cheese'},
    'italian': {'sausage', 'peppers', 'onions'},
    'veggie': {'peppers', 'onions', 'mushrooms', 'olives'},
    'supreme': {'pepperoni', 'ham', 'beef', 'sausage', 'peppers', 'onions', 'mushrooms', 'olives'}
}

> for pizza, topping in menu.items():
...     if 'onions' in topping:
...             print(pizza)
... 
italian
veggie
supreme

> for pizza, topping in menu.items():
...     if 'onions' in topping and not ('sausage' in topping):
...             print(pizza)
... 
veggie

```

## Combinations


```python

# Set intersection (&) checks for combinations
# of set values
> for pizza, topping in menu.items():
...     if topping & {'peppers', 'onions'}:
...             print(pizza)
... 
italian
veggie
supreme

> for pizza, topping in menu.items():
...     if 'onions' in topping and not topping & {'mushrooms', 'olives'}:
...             print(pizza)
... 
italian


```

## Operators

```python

> a = {1, 2}
> b = {2, 3}
> a
{1, 2}
> b
{2, 3}

# get the intersection
> a & b
{2}
> a.intersection(b)
{2}


> fave = menu['italian']
> fave
{'peppers', 'onions', 'sausage'}
> worst = menu['veggie']
> worst
{'peppers', 'mushrooms', 'onions', 'olives'}
> fave & worst
{'peppers', 'onions'}

# get the union (combine both sets)
> fave | worst
{'peppers', 'olives', 'onions', 'sausage', 'mushrooms'}
> fave.union(worst)
{'peppers', 'olives', 'onions', 'sausage', 'mushrooms'}
> a | b
{1, 2, 3}

# difference
> a - b
{1}
> fave - worst
{'sausage'}

# exclusive or (item in one set but not both)
> a ^ b
{1, 3}
> fave ^ worst
{'olives', 'sausage', 'mushrooms'}

# subsets
> a <= b
False
> a.issubset(b)
False
> fave <= worst
False
> fave.issubset(worst)
False
> best = menu['supreme']
> fave <= best
True
> fave < best
True

# supersets
> fave >= best
False
> fave.issuperset(worst)
False
> fave >= fave
True
> fave > fave
False

```

## Set comprehensions

Set comprehensions use the following format:

[`expression` for `expression` in `iterable`]

You can add a condition afer the iterable:

[`expression` for `expression` in `iterable` if `condition`]

```python
> even_set = {number for number in range(1,20) if number % 2 == 0}
> even_set
{2, 4, 6, 8, 10, 12, 14, 16, 18}

```
## Immutable sets

Use frozenset() to create a set that you cannot change:

```python

# create a frozenset()
> frozenset([1, 2, 3])
frozenset({1, 2, 3})
> frozenset(set([2, 4, 6]))
frozenset({2, 4, 6})
> frozenset({1, 2, 3})
frozenset({1, 2, 3})
> frozenset( (2, 3, 1) )
frozenset({1, 2, 3})

# cannot change a frozenset()

> test = frozenset({1, 2, 3})
> test
frozenset({1, 2, 3})
> test.remove(3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'remove'
> test.add(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'add'
```

# Functions

When you call a function with arguments, the values of the arguments are copied to the corresponding parameters inside the function body. 

Python requires the `pass` statement to show that the function does nothing:

```python
> def nothing():
...     pass
... 
> nothing()
>
```

Positional argument values are copied to their corresponding parameters in order:

```python
> def menu(wine, entree, dessert):
...     return {'wine': wine, 'entree': entree, 'dessert': dessert}
... 
> menu('Pino noir', 'Ribeye', 'Chocolate cake')
{'wine': 'Pino noir', 'entree': 'Ribeye', 'dessert': 'Chocolate cake'}
```
## Default Parameter Values

The default value is used if the caller does not provide a corresponding argument.

```python

> def menu(wine, entree, dessert='ice cream'):
...     return {'wine': wine, 'entree': entree, 'dessert': dessert}
... 
> menu('Chardonnay', 'Tilapia')
{'wine': 'Chardonnay', 'entree': 'Tilapia', 'dessert': 'ice cream'}

```
Do not use data structures, such as lists, as default values. Mutable values are instantiated with the function is defined, not when it is run:

```python

# The first sample creates the list when the function is
# defined, and it retains the values throughout calls
> def todo(arg, tasks=[]):
...     tasks.append(arg)
...     print(tasks)
... 
> todo('shop')
['shop']
> todo('clean')
['shop', 'clean']

# Using None as the default allows you instantiate the list each time
> def todo(arg, result=None):
...     if result is None:
...             result = []
...     result.append(arg)
...     print(result)
... 
> todo('shop')
['shop']
> todo('clean')
['clean']
```

## None vs False

`None` is a special value that is not the same as `False`, although it might seem that way. Use `None` to distinguish a missing value from an empty value. Zero-valued objects like empty strings are all `False`, but not `None`.

Use the `is` operator to test 

```python
> test = None
> test
> if test is None:
...     print('Use the is operator')
... 
Use the is operator

```

## * parameter

The `*<arg-name>` groups a variable number of positional arguments into a single tuple of parameter values:

```python

# All args are stored in a tuple
> def print_args(*args):
...     print('positional tuple:', args)
... 
> print_args('one', 1, ['two', 'three'], 2, 'four')
positional tuple: ('one', 1, ['two', 'three'], 2, 'four')


# After the positional args are satisfied, all args are stored in a tuple
> def multi_args(a1, a2, *args):
...     print('First:\t', a1)
...     print('Second:\t', a2)
...     print('Rest:\t', args)
... 
> multi_args('one', 'two', 3, 4, 5, 6, 7, 8, 9)
First:	 one
Second:	 two
Rest:	 (3, 4, 5, 6, 7, 8, 9)
```
## ** keyword argument parameter

The `**<arg-name>` groups keywords into a dictionary, where the first word is the key and the second the value. It is common to name this argument `kwargs`:

```python

# No args
> print_kwargs()
Keyword arguments: {}

# adding args
> print_kwargs(first=1, second=2, third=3)
Keyword arguments: {'first': 1, 'second': 2, 'third': 3}
```

## Docstrings

Add docstrings to explain your functions. Use `'` for a single line. For longer quotes, place `'''` at the beginning and end of the quote, where the quotes are on their own line:

```python
> def printer(arg):
...     print(arg)
... 
> def printer(arg):
...     'printer prints any argument to the console'
...     print(arg)

# Multi-line docstrings
> def print_if_true(arg, check):
...     '''
...     Prints the first argument if a second argument is true.
...     The operation is:
...             1. Check whether the *second* argument is true.
...             2. If it is, print the *first* argument.
...     '''
...     if check:
...             print(thing)
... 

# Pass the function name to the help() function to print the docstring
> help(print_if_true)
```

The `__doc__` dunder method is the internal name of the docstring as a variable within the function. Use it to access the docstring for a function:

```python
> print(print_if_true.__doc__)

	Prints the first argument if a second argument is true.
	The operation is:
		1. Check whether the *second* argument is true.
		2. If it is, print the *first* argument.
```

## First-class citizens

Each function is an object. This means that you can assign them to variables, use them as arguments to other functions, and return them from functions.

Remember not to use the `()` after a first-class function becuase that calls the function. To assign the function, just use the name:

```python
# define a function
> def name():
...     print('My name is Bob.')
... 

# function that takes a function as an argument
> def do_func(func):
...     func()
... 
> do_func(name)
My name is Bob.

# assign the function to a variable
> test = name

# pass the variable as an argument
> do_func(test)
My name is Bob.

# functions are just another object
> type(name)
<class 'function'>
> type(test)
<class 'function'>
```

# Inner functions and closures

Define a function within another function to take advantage of scoped.

A closure is a function that is dynamically generated by another function and can both change and remember the values of variables that were created outside the function.

```python

# function that contains another function, and returns that other function
> def cue_card(phrase):
...     def announcer():
...             return f'The first guest tonight is {phrase}'
...     return announcer
... 

# cannot call the func with an argument
> cue_card('Elvis')
<function cue_card.<locals>.announcer at 0x7f223d6754c0>

# assign the funciton to variables
> a = cue_card('Elvis')
> b = cue_card('Ozzy Osbourne')

# types
> type(a)
<class 'function'>
> type(b)
<class 'function'>

# call the functions
> a()
'The first guest tonight is Elvis'
> b()
'The first guest tonight is Ozzy Osbourne'
```

## Lambdas

A lambda function is an anonymous function expressed as a single statement. It has zero or more comma-separated arguments followed by a colon (:), and then the definition of the function. You don't have to use parentheses.

Lambdas use the following format:

`lambda arg1[,arg2,...]: expression`

```python

# normal functions
> def alter_list(alist, func):
...     for item in alist:
...             print(func(item))
... 
> def upper(word):
...     return word.capitalize()
... 
> alter_list(names, upper)
John
Paul
George
Ringo

# with a lambda
> alter_list(names, lambda word: word.capitalize())
John
Paul
George
Ringo
```

## Generators

A generator is a sequence creation object that iterates through huge sequences without creating and storing the entire sequence in memory. `range()` is an example.

The generator keeps track of where it was the last time it was called and returns the next value.

To write a generator, use a `yield` statement instead of `return`:

```python
> def new_range(first=0, last=10, step=1):
...     number = first
...     while number < last:
...             yield number
...             number += step
... 
> test = new_range(1, 5)
> type(test)
<class 'generator'>
> test
<generator object new_range at 0x7f223d687dd0>

```

## Decorators

A decorator is a function that takes one function as input and returns another function.

```python
> def document_func(func):
     def new_function(*args, **kwargs):
             print('Running function:', func.__name__)
             print('Positional args:', args)
             print('Keyword args:', kwargs)
             result = func(*args, **kwargs)
             print('Result:', result)
             return result
     return new_function
 
> def add_ints(a, b):
     return a + b
 
> add_ints(4, 5)
9
> doc_add_ints = document_func(add_ints)
> doc_add_ints(3, 5)
Running function: add_ints
Positional args: (3, 5)
Keyword args: {}
Result: 8
8
```
Use the `@decorator-name` before the function that you want to decorate:

```python
> @document_func
 def add_ints(a, b):
     return a + b
 
> add_ints(4, 5)
Running function: add_ints
Positional args: (4, 5)
Keyword args: {}
Result: 9
9

# Multiple decorators per function
@document_func
@times_three_func
def add_ints(4, 5)
    ...
```
# Exceptions

Exceptions are code that is executed when an error occurs. Use `try` to wrap code that might fail for whatever reason, and use `except` to handle the error:

```python
> alist = [0, 1, 2, 3, 4]
> bad_index = 6
> alist[bad_index]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range

# With error handling
> try:
    alist[bad_index]
  except:
    print('Requires an index between 0 and ', len(alist)-1, ' but got', bad_index)
 
Requires an index between 0 and  4  but got 6
```
Provide an `except` for each possible exception that might occur. You can capture the full exception object in a variable as follows:

`except` *exceptiontype* `as` *name*:

```python
> alist = [1, 2, 3]
> bad_index = 4

# Function that catches exceptions
> def exception_func(the_list, index):
     try:
             the_list[index]
     except IndexError as err:
             print('Requires index between 0 and ', len(the_list)-1,' but got', index)
     except Exception as other:
             print('There was a different error:', other)

# Throw IndexError 
> exception_func(alist, 5)
Requires index between 0 and  2  but got 5

# Throw Exception
> exception_func(alist, 'dog')
There was a different error: list indices must be integers or slices, not str
```

## Defining your own Exceptions

Python's standard library provides predefined exceptions, but you can also define your own by inheriting the `Exception` class:

```python
> class UppercaseException(Exception):
     print('You threw the UpperclassException!')
 
> words = ['one', 'two', 'three', 'FOUR']
> words
['one', 'two', 'three', 'FOUR']

# Throw the exception that you made
> for word in words:
     if word.isupper():
             raise UppercaseException(word)
 
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
__main__.UppercaseException: FOUR

```

# Objects

An object is a data structure that contains data, stored in attributes, and behavior, called methods (or functions).

## Simple objects

```python

# Create a dog class
> class Dog():
     pass

# Instantiate a few dog objects
> fido = Dog()
> fido
<__main__.Dog object at 0x7f3c175ecfa0>
> pluto = Dog()
> pluto
<__main__.Dog object at 0x7f3c18c230d0>

```

## Adding attributes

Assign object attributes at creation time with the `__init__()` method.

```python
> class Teacher:
     def __init__(self):
             pass
```
In the previous example:

- `__init__()` is the special name for a method that initializes an object from its class definition. It is a constructor.
- `self` argument specifies that the `__init__()` method refers to the individual object itself. You also have to pass `self` as the first argument to an instance method.

## Initializers, not constructors

`__init__()` is an initializer, not a constructor. After Python finds the class code, it creates an object. `__init()__` initializes that object with the attributes and methods that make it unique. You do not need to have an `__init()__` method for each class definition. `__init()__` is used to distinguish each object of the class from one another.

Its first parameter must be `self` so that it can assign any attributes to the object. For example:

```python
> class Teacher:
     def __init__(self, name):
             self.name = name

> mr_smith = Teacher('Smith')
> mr_smith.name
'Smith'
```
In the previous example:
1. Python looks up the Teacher class.
2. Instantiates a new object in memory
3. Calls the object's `__init()__` method and passes the new object to `__init()__` as `self`, and the argument as the `name` attribute.
4. Stores the value of `name` in the object
5. Returns the new object.
6. Assigns the variable `mr_smith` to the object.

## Inheritance

Inheritance is when you create a new class from an existing one, with some modifications. In a child class, you define only what you need to add or change in the new class, and that overrides the behavior of the old class.

To inherit from a parent class, define a subclass and pass the name of the parent in the parentheses:

```python
> class Guitar():
     def strum(self):
             print('raaaaannnngggg')
 
> class Fender(Guitar):
     pass
 
> issubclass(Fender, Guitar)
True

# The subclass accesses methods from the parent
> blackie = Fender()
> blackie.strum()
raaaaannnngggg

# Override the strum method
> class Fender(Guitar):
     def strum(self):
             print('brrroooooonnnnngggggg')
 
> blackie = Fender()

# Python looks up the class of the object 'blackie'
# Passes the object 'blackie' as the self param to the strum() method of the class
> blackie.strum()
brrroooooonnnnngggggg

# Overriding __init__()
> class Person():
     def __init__(self, name):
             self.name = name
 
> class MDPerson(Person):
     def __init__(self, name):
             self.name = "Doctor " + name
 
> class JDPerson(Person):
     def __init__(self, name):
             self.name = name + ", Esquire"
 
> person = Person('Jack')
> doctor = MDPerson('Jack')
> lawyer = JDPerson('Jack')
> print(person.name)
Jack
> print(doctor.name)
Doctor Jack
> print(lawyer.name)
Jack, Esquire
```
### super()

`super()` calls a parent method. The `__init__()` method for a child class replaces the __init__() method for the parent class:

```python
> class Person():
     def __init__(self, name):
             self.name = name

> class EmailPerson(Person):
     def __init__(self, name, email):
             super().__init__(name)
             self.email = email
```

In the previous example:
- `super()` gets the definition of the parent class, Person
- `__init__()` calls the Person.__init__() method. It passes the self arg to the superclass, so it needs any optional arguments that the superclass __init__() method needs
- `self.email = email` assigns a value to email, which is the attribute that makes the child class unique

Use `super()` to make sure that you always inherit from the superclass, especially if the superclass changes in the future.

## Getters and Setters

Python doesn't have a way to make attributes private. The best way to handle this is with 'secret' attribute names and **properties**. Use the following decorators:
- `@property` for the getter method.
- `@<name>`.setter for the setter method. If you don't specify a setter property for an attribute, you can't set it from outside code.

Prepend hidden attribute names with double underscores (`__`). For example, `__hidden_attribute`.

In the following example, the object has an attribute `__name` that cannot be accessed from outside the class. However, you can set it with the getters and setters:

```python
> class Person():
     def __init__(self, name):
             self.__name = name
     @property
     def name(self):
             print('getter method')
             return self.__name
     @name.setter
     def name(self, name):
             print('setter method')
             self.__name = name
 
> musician = Person('Paul')
> musician.name
getter method
'Paul'
> musician.name = 'John'
setter method
> musician.name
getter method
'John'
```

Python changes the hidden attributes to look like this:

```python
> musician._Person__name
'John'
```

## Class and object attributes

You can assign attributes with default values to classes and change them in any objects:

```python
> class Boots:
     color = 'brown'
 
> red_wing = Boots()

> red_wing.color
'brown'

> red_wing.color = 'tan'
> red_wing.color
'tan'

# default class value is preserved
> Boots.color
'brown'

```

## Method types

There are three main method types:

- *instance*: First argument is `self`. `self` refers to the individual object.  
- *class*: First argument is `cls` (or anything you want, just be consistent). Naming it `cls` is Pythonic and refers to the class itself. The first parameter refers to the class itself.  
  Precede class methods with the `@classmethod` decorator.  
  Any change you make to the class affects all of the objects instantiated from the class.
- *static*: No first argument. Precede static methods with the `@staticmethod` decorator.  
  Static mthods do not affect the class or its objects, it's a convenience method.

To demonstrate a class method, we create a class that tracks the number of objects created using it:

```python
> class ChildCounter():
     count = 0
     def __init__(self):
             # set the class attribute with the class name 
             ChildCounter.count += 1
     def exclaim(self):
             print("I'm in the ChildCounter() class!")
     @classmethod
     def children(cls):
             # access the class with cls    
             print('ChildCounter has ', cls.count, 'child objects.')
 
> first = ChildCounter()
> second = ChildCounter()
> third = ChildCounter()
> ChildCounter.count
```

To demonstrate a static method, we create a simple class with a method that does not require an object:

```python
> class Nike():
     @staticmethod
     def slogan():
             print('Just Do it')
 
> Nike.slogan()
Just Do it
```

## Magic methods

Magic methods are methods that Python recongnizes that you can define on your objects. Here is a good list of [magic methods](https://docs.python.org/3/reference/datamodel.html#special-method-names).

Common ones are:

| Method                | Implementation | Description                                                |
|-----------------------|----------------|------------------------------------------------------------|
| `__eq__(self, other)` | self == other  | checks if `self` is equal to `other`                       |
| `__str__(self)`       | str(self)      | returns a string of your object. How you print your object |
| `__len__(self)`       | len(self)      | returns the length of `self`                               |

## Aggregation and Composition

Build objects with objects, like this boat:

```python
> class Sail():
     def __init__(self, description):
             self.description = description
 
> class Tiller():
     def __init__(self, length):
             self.length = length
 
> class Boat():
     def __init__(self, sail, tiller):
             self.sail = sail
             self.tiller = tiller
     def about(self):
             print('This boat has a', self.sail.description, 'sail and a', self.tiller.length, 'long tiller')
 
> sail = Sail('blue and white')
> tiller = Tiller('5 ft')
> boat = Boat(sail, tiller)
> boat.about()
This boat has a blue and white sail and a 5 ft long tiller

```

## Named tuples

Named tuples are a type of tuples that you can access values by `.name` and position with [offset]. You have to load the `namedtuple` module, and they use the following format:

`namedtuple('Type-name', 'attr1 attr2[ ...]')`

```python
This boat has a blue and white sail and a 5 ft long tiller
> from collections import namedtuple
> Boat = namedtuple('Boat', 'sail tiller')
> boat = Boat('blue', 'rudder')
> boat
Boat(sail='blue', tiller='rudder')
> boat.sail
'blue'
> boat.tiller
'rudder'
```

## Dataclasses

Data classes let you create objects that store only data--they don't have behavior. To define a dataclass, you have to do the following things:

1. Import dataclass
2. Use the `@dataclass` decorator
3. Define attributes with the variable annotations form of `name: type` or `name: type = val`

Here is how you define a dataclass:

```python
> from dataclasses import dataclass
> @dataclass
 class GuitarClass:
     brand: str
     color: str
     price: int = 0  # 0 is the default value
 
> strat = GuitarClass('Fender', 'Black', 1000)
> strat
GuitarClass(brand='Fender', color='Black', price=1000)

# Define attributes in any order with named arguments
> les_paul = GuitarClass(price=2500, brand='gibson', color='honeyburst')
> les_paul
GuitarClass(brand='gibson', color='honeyburst', price=2500)
```

# Functional programming
## map

## reduce

## filter

# Data and encoding