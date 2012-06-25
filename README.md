# asd

[dg](https://github.com/pyos/dg) to CPython byte-compiler.

### What a good name for a compiler.

As you may have already noticed, I have some problems thinking of project names.
Got any suggestions?

## Requirements

 * [dg](https://github.com/pyos/dg)
 * [interactive](https://github.com/pyos/interactive)

## Unsupported features

1. `try/except/else/finally` clauses produce very complex bytecode sequences I was unable to decode so far. Maybe later.
2. `if/elif/else` is not something I've started working on.
3. Methods lack `__class__` cell; `super` needs both arguments to work correctly.
4. Functions only support positional arguments with no default values. Support for defaults, keyword-only arguments, var(kw)args and annotations will be implemented later.
5. Only absolute imports.

## A Small Reference

First of all, parentheses are only used to explicitly set the evaluation order.
Function call is implemented as in Haskell: it is an empty operator that has
the highest priority of all.

A dot (`.`) is an exception, though.

```coffeescript
# Without parentheses: print('Hello', 'World!', sep) = ', '
print 'Hello' 'World!' sep = ', '
# With parentheses: print('Hello', 'World!', sep=', ')
print 'Hello' 'World!' (sep = ', ')

# Without parentheses: f(a) + b
f a + b
# With parentheses: f(a + b)
f (a + b)

# Without parentheses: f(a.b)
f a.b
# With parentheses: f(a).b
(f a).b
```

Prefix the name of a function with a colon to call it without any arguments.
Don't forget to place parentheses accordingly.

```coffeescript
:f  # f()

g :h  # Similar to `g : h` and means g(h), since `:` is defined as a call operator.
g (:h)  # g(h())
```

### Functions

Syntax for functions is similar to that of CoffeeScript...

```coffeescript
square = (x) -> x * x
cube   = (x) -> square x * x
```

...but with some changes. In particular, you can't omit either side of a definition:

```coffeescript
# WRONG
noop = ->
defined_function (-> print 'callback')

# CORRECT
noop = () -> ()
defined_function (() -> print 'callback')
```

(The following syntax is not supported yet, move along.)

You may omit parentheses in the argument list if you only have one argument
with no annotation or default value, though.

```coffeescript
double    =  x  -> x * 2
doubleAll = *xs -> map double xs
```

Variable-length argument lists and argument annotation have syntax similar to
Python, not CoffeeScript:

```coffeescript
f = (*args)    -> print args
g = (**kwargs) -> print kwargs
h = (x: 'annotation' = 'default value') -> print x
```

### Scoping

The compiler has support for closures and other stuff. Two caveats here:

1. You can't use variables from outer scopes before they are assigned to.
2. Forget about changing non-local scopes. Neither `global` nor `nonlocal` keyword is availale.

```coffeescript
outer = 1

changeNumbers = () ->
  stuff = outer + 5  # OK
  inner = -1  # Perfectly fine, creates a new local variable.
  outer = 10  # Still fine, does not change the global variable though.

  f = () ->
    inner = 100500  # Now that is a compile-time error.

inner = :changeNumbers
```

### Objects

There is no built-in syntax for lists, dicts and other stuff for now, so you'll
have to use Python types instead.

```coffeescript
song = list ('do', 're', 'mi', 'fa', 'so')

kids = set (('Max', 11), ('Ida', 9))
```

In contrast with Python, classes are always anonymous, so they are
defined with a function called `inherit`, not with the `class` keyword.

```coffeescript
# `object` is optional here, as it is a default base class.
Animal = inherit object:
  eat = (self) -> print 'NOM NOM'

Mammal = inherit Animal:
  breathe = (self) -> print 'phhh'

WingedAnimal = inherit Animal:
  fly = (self) -> print 'flap flap'

# Multiple inheritance is supported, too.
#
# Remember that `:` operator is just a function call and indentation is used
# to create blocks? Well, parentheses create blocks, too. () is an empty one.
Bat = inherit Mammal WingedAnimal ()

pet = :Bat
:pet.eat
:pet.fly
```

### Modules

1. Insert a `=` between `import` and a module name.
2. Reverse the order of operands.
3. Enjoy the might of Python modules.

```coffeescript
os.path = import

# By the way, `$` is a low-priority function call operator, too.
print $ os.path.exists '/'
```

### Everything is an Expression

*Every* statement returns a value. No exceptions. Assignments return the assigned
value, blocks return the result of the last expression, etc.

### Generators

They are pretty useless if you note that neither `for` nor `while` is available yet.

```coffeescript
countdown () ->
  yield 3
  yield 2
  yield 1

print $ list $ :countdown
```

Returning a value from a generator is the same as yielding it.

```coffeescript
countdown2 () ->
  yield 3
  yield 2
  1
```

Note that since `yield` is also an expression, sending a value to the last
`yield` will cause that value to be yielded, too. If that's not what you want,
simply return `None` from the generator.

```coffeescript
countdown3 () ->
  yield 3
  yield 2
  yield 1
  some_stuff_that_shouldnt_be_yielded
  None
```

### Redefining Operators

One more neat feature borrowed from Haskell is the ability to create new operators.
Sadly, they all have whatever priority is hardcoded as default in the parser,
but it's still pretty cool.

```coffeescript
($%^&) = (a, b) -> max (a, b) + a + b

print $ 1234 $%^& 5678
```

