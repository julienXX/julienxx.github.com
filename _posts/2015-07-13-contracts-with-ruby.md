---
title:  "How I upgraded my Ruby with Contracts"
date:   2015-07-13 17:18:00
description: "and made my code safer"
---

Toying with many languages made me discover new approaches and techniques. For example, Haskell taught me about [Types](http://learnyouahaskell.com/making-our-own-types-and-typeclasses) and Erlang/Elixir enlightened me on [Pattern-matching](http://learnyousomeerlang.com/syntax-in-functions).

Professionally I mainly code with Ruby and I dreamed of having an advanced type system and some pattern-matching. I discovered this brilliant gem [Contracts.ruby](https://github.com/egonSchiele/contracts.ruby) by [Aditya Bhargava](http://adit.io) and in this article I will try to present [Design by Contracts](https : //en.wikipedia.org/wiki/Design_by_contract) through the use of this gem.

## What is a contract?

A contract ensures what kind of input a method expects (pre-condition), what it outputs (post-condition). It will define how our method behaves but also check its behavior.

The gem `Contracts.ruby` allows us to decorate our methods with code that will check that the inputs and outputs correspond to what the contract specifies. Of course, one is not obliged to annotate each method but I think that specifying a contract for your public API can only be beneficial.

## A first example

```ruby
Contract Num, Num => Num
def add(a, b)
  a + b
end
```

The contract of my method is **Contract Num Num => Num**, meaning that the *add* method takes two numbers as input and returns a number. Simple, right?

You will object that ok, it's documentation, I could just add a comment. But since this is a contract, the Contracts.ruby gem will help ensure that it is respected.

```ruby
require 'contracts'

class Foo
  include Contracts

  Contract Num, Num => Num
  def self.add(a, b)
    a + b
  end
end
```

*Foo.add(1, 2)* obviously returns 3 but *Foo.add(1, '2')* will return:

```ruby
ParamContractError: Contract violation for argument 2 of 2:
        Expected: Num,
        Actual: "2"
        Value guarded in: Foo::add
        With Contract: Num, Num => Num
```

The error highlights that the contract of the method *add* wasn't respected because the second parameter we sent, '2', isn't of the type *Num*.

Note that you must always specify the type of the value returned even if the method does not return anything:

```ruby
Contract String => nil
def hello(name)
  puts "hello, #{name}!"
end
```

If our method returns many values, its signature will be

```ruby
Contract Num => [Num, Num]
```

## Types at our disposal

Besides the classics **Num**, **String**, **Bool**, we can use more interesting types like:

- **Any** when we have no type constraint
- **None** when you need no argument
- **Or** if our argument can be of different types, for example **Or[Fixnum, Float]**
- **Not** if our argument can't be of a certain type, like **Not[nil]**
- **Maybe** if our argument is optionnal, example **Maybe[String]**

And many others that you will discover in the documentation.

## Advanced Types Contracts

We can use contracts with advanced types like lists:

```ruby
Contract ArrayOf[Num] => Num
def multiply(vals)
  vals.reduce(:*)
end
```

The contract of *multiply* method indicates that it wants a list of values of the type Num. Therefore *multiply([2, 4, 16])* is valid but *multiply([2, 4, 'foo'])* is not.

Hashes:

```ruby
Contract ({ nom: String, age: Num, ville: String }) => nil
```

Methods:

```ruby
Contract ArrayOf[Any], Proc => ArrayOf[Any]
```

If you use Ruby 2.x keyword arguments, the contract will look like:

```ruby
Contract KeywordArgs[foo: String, bar: Num] => String
```

We can also define our own contracts with *synonyms*:

```ruby
Token = String
Client = Or[Hash, nil]

Contract Token => Client
def authenticate(token)
```

Our *authenticate* method is thus more clear as to what it expects and what it does. A *Token* of type *String* is desired as input and it returns a *Client* which can be a *Hash* or nothing (nil).

## Pattern-matching

Pattern-matching will, for a given value, test if it matches a pattern or not. If this is the case an action is triggered. It's a bit like Java method overloading. One could imagine it as a giant switch case but much more elegant.

A simple example with calculation (not effective at all) of the Fibonacci sequence:

```ruby
Contract 0 => 0
def fib(n)
  0
end

Contract 1 => 1
def fib(n)
  1
end

Contract Num => Num
def fib(n)
  fib(n-1) + fib(n-2)
end
```

For a given argument, each method will be tried in order. The first method that does not generate an error will be used.

A little more real-world™ example, the management of an HTTP response based on its status code:

```ruby
Contract 200, JsonString => JsonString
def handle_response(status, response)
  transform_response(response)
end

Contract Num, JsonString => JsonString
def handle_response(status, response)
  response
end
```

If the HTTP response code is 200 it will transform the answer, otherwise we will simply return the response.

## Conclusion

There are many benefits. Contracts allow us to have greater consistency in our inputs and outputs. The flow of data in our system is clearer. And most the type errors of our system can be detected and fixed quickly. Additionally it's easier to understand what a method does, needs and returns. It also provides some kind of documentation that would always be up to date :p.

I think we can thus save a lot of unit tests on the type of the argument(s) received by a method and focus on what it produces with this contract system. Refactoring also becomes a lot easier with this kind of safety.

I hope this article has convinced you of the value of contracts and pattern-matching in your daily Ruby and also gave you the urge to explore other languages ​​with other paradigms.
