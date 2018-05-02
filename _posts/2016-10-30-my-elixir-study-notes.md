---
layout: post
title: My Elixir Study Notes
---

![Article illustration](https://s20.postimage.org/mtahg5gwd/Untitled.jpg)

While studying the Elixir [Getting Started Guide](http://elixir-lang.org/getting-started/introduction.html) I decided to begin resuming it out as a way to get it fixed. I ended up with a very useful document for those who already read the guide and want to quickly review any topic. It’s a one page document resuming the whole guide, which is just great to use with the *Find* command.

I basically copied, slightly altered and omitted great part of the content to focus on things I thought more important (to me). But provides a quick explanation of everything. Also includes some notes I made.

*As this post still serve as a study resource, it’s constantly updated.*

* * *

## Important Links
* * *

+ [Writing Assertive Code With Elixir](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/)

## Notes
* * *
+ Get some help:

```elixir
iex> i 'hello'
Term
  'hello'
Data type
  List
Description
  ...
Raw representation
  [104, 101, 108, 108, 111]
Reference modules
  List
```

+ When “counting” the number of elements in a data structure, Elixir also abides by a simple rule: the function is named size if the operation is in constant time (i.e. the value is pre-calculated) or length if the operation is linear (i.e. calculating the length gets slower as the input grows).
+ String concatenation is done with: `<>`.
+  Operators `or`, `and`, `not` can only accept boolean values. Besides these boolean operators, Elixir also provides `||`, `&&` and `!` which accept arguments of any type. For these operators, all values except `false` and `nil` will evaluate to `true`.
+ The variable `_` is special in that it can never be read from. Trying to read from it gives an unbound variable error.
+ Guard Clauses are neat:

```elixir
# Anonymous functions can have guard clauses:
# They also apply to the 'case' statement, 'when'.
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

## Basic Types
* * *

### Booleans
All good, just `true` and `false`. Nothing special.

### Atoms
Pretty much like Lisp's Atoms, A.K.A. Symbols in Ruby.

### Anonymous Functions
Anonymous Functions (function literal, lambda abstraction) is delimited between `fn` and `end`.

```elixir
# first class citizens (can be passed as arguments)
iex> add = fn a, b -> a + b end
iex> add.(3, 2)
```

Anonymous functions are closures and as such they can access variables that are in scope when the function is defined.

### Lists
Describes itself.

```elixir
# Add or subtract using ++ or --
iex> [2, 23, 42, 11, true]
iex> list = [1, 2, 3]
# Get head and tail.
iex> hd(list)
1
iex>tl(list)
[2, 3]
```

+ When Elixir sees a list of printable ASCII numbers, Elixir will print that as a char list (literally a list of characters).
+ Single-quotes are char lists, double-quotes are strings.

### Tuples
Similar to lists, but stored in memory, all data is availible with no recursion needed.

```elixir
iex> {:ok, "hello"}
{:ok, "hello"}
iex> tuple_size {:ok, "hello"}
2
```

### List or Tuples?
Accessing the length of a list is a linear operation: we need to traverse the whole list in order to figure out its size. Updating a list is fast as long as we are prepending elements.  

Tuples, on the other hand, are stored contiguously in memory. This means getting the tuple size or accessing an element by index is fast. However, updating or adding elements to tuples is expensive because it requires copying the whole tuple in memory.  

When “counting” the number of elements in a data structure, Elixir also abides by a simple rule: the function is named size if the operation is in constant time (i.e. the value is pre-calculated) or length if the operation is linear (i.e. calculating the length gets slower as the input grows).  

## Pattern Matching
* * *
The match operator is not only used to match against simple values, but it is also useful for destructuring more complex data types. For example, we can pattern match on tuples:

```elixir
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

A list also supports matching on its own head and tail:

```elixir
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

**The pin operator `^` should be used when you want to pattern match against an existing variable’s value rather than rebinding the variable.**

```elixir
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {y, ^x} = {2, 1}
{2, 1}
iex> y
2
iex> {y, ^x} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

## `case`, `cond` and `if`
* * *

### `case`
Behavious pretty much as the classic `case` statement.

```elixir
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
```

If you want to pattern match against an existing variable, you need to use the `^` operator:

```elixir
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _  -> "Will match"
...> end
```

Another cool example, now with clauses conditions:

```elixir
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Would match, if guard condition were not satisfied"
...> end
```

+ If none of the clauses match, an error is raised.

### `cond`
`case` is useful when you need to match against different values. However, in many circumstances, we want to check different conditions and find the first one that evaluates to true. In such cases, one may use `cond`.

```elixir
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
```

+ This is equivalent to `else` and `if` clauses in many imperative languages.  
+ If none of the conditions return `true`, an error is raised. For this reason, it may be necessary to add a final condition, equal to `true`, which will always match.

### `if` and `unless`
Are useful when you need to check for just one condition, also pro provides a `else` statement.

```elixir
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

### `do` / `end` blocks
Equivalent to `{` / `}`, it's also possible things like:

```elixir
iex> if false, do: :this, else: :that
```

```elixir
# Expressions like:
iex> is_number if true do
...>  1 + 2
...> end
** (CompileError) undefined function: is_number/2
# Should be:
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

## Binaries, strings and char lists
* * *

### Binaries and bitstrings
You can define a binary using `<<>>`. It's just a sequence of bytes. The string concatenation operation is actually a binary concatenation operator `<>`.

A common trick in Elixir is to concatenate the null byte `<<0>>` to a string to see its inner binary representation:

```elixir
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

**A binary is a bitstring where the number of bits is divisible by 8. Smaller bit are just bitstrings!**

A string is a UTF-8 encoded binary, and a binary is a bitstring where the number of bits is divisible by 8.

### Char lists
A char list is nothing more than a list of characters.  

Char list contains the code points of the characters between single-quotes (note that IEx will only output code points if any of the chars is outside the ASCII range). So while double-quotes represent a string (i.e. a binary), single-quotes represents a char list (i.e. a list).

## Keywords and maps
* * *

### Keyword list
It's a associative data structure. In Elixir, when we have a list of tuples and the first item of the tuple (i.e. the key) is an atom, we call it a keyword list:

```elixir
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
iex> list[:a]
1
```

+ It's the default mechanism for passing options to functions in Elixir.
+ Only allows Atoms as keys.
+ Ordered as specified by the developer.
+ Remember, though, keyword lists are simply lists, and as such they provide the same linear performance characteristics as lists. The longer the list, the longer it takes to read from. For bigger data use maps instead.

### Maps
Whenever you need a key-value store, maps are the “go to” data structure in Elixir:

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

+ Allows any value as key.
+ Maps’ keys do not follow any ordering.

Interacts great with pattern matching:

```elixir
iex> %{} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

Better syntax when all keys are atoms:

```elixir
iex> map = %{a: 1, b: 2}
```

Another interesting property of maps is that they provide their own syntax for updating and accessing atom keys:

```elixir
iex> map = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
iex> %{map | :a => 2}
%{:a => 2, 2 => :b}
iex> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```


### Nested data structures (`put_in/2` and `update_in/2`)

```elixir
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
```

We have a keyword list of users where each value is a map containing the name, age and a list of programming languages each user likes. If we wanted to access the age for john, we could write:

```elixir
iex> users[:john].age
27
```

It happens we can also use this same syntax for updating the value:

```elixir
iex> users = put_in users[:john].age, 31
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}]
```

The `update_in/2` macro is similar but allows us to pass a function that controls how the value changes. For example, let’s remove “Clojure” from Mary’s list of languages:

```elixir
iex> users = update_in users[:mary].languages, &List.delete(&1, "Clojure")
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
mary: %{age: 29, languages: ["Elixir", "F#"], name: "Mary"}]
```

There is more to learn about `put_in/2` and `update_in/2`, including the `get_and_update_in/2` that allows us to extract a value and update the data structure at once. There are also `put_in/3`, `update_in/3` and `get_and_update_in/3` which allow dynamic access into the data structure. [Check their respective documentation in the Kernel module](http://elixir-lang.org/docs/stable/elixir/Kernel.html) for more information.

## Modules
* * *

In Elixir we group several functions into modules.

```elixir
iex> defmodule Math do
...>   def sum(a, b) do
...>     a + b
...>   end
...> end

iex> Math.sum(1, 2)
3
```

### Compilation
Given a file `math.ex`:

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

This file can be compiled using `elixirc`:

```elixir
$ elixirc math.ex
```

This will generate a file named Elixir.Math.beam containing the bytecode for the defined module. If we start iex again, our module definition will be available (provided that iex is started in the same directory the bytecode file is in).  

Elixir projects are usually organized into three directories:

+ `ebin` - contains the compiled bytecode
+ `lib` - contains elixir code (usually .ex files)
+ `test` - contains tests (usually .exs files)

When working on actual projects, the build tool called mix will be responsible for compiling and setting up the proper paths for you.

### Scripted mode

+ `.ex` - files to be compiled
+ `.exs` - files to run in scripted mode (Learning purposes)

Executing:

```elixir
$ elixir math.exs
```

### Named functions

+ `def/2` - defines a function
+ `defp/2` - defines a private function

Function declarations also support guards and multiple clauses. If a function has several clauses, Elixir will try each clause until it finds one that matches.

```elixir
defmodule Math do
  def zero?(0) do
    true
  end

  def zero?(x) when is_integer(x) do
    false
  end
end

IO.puts Math.zero?(0)         #=> true
IO.puts Math.zero?(1)         #=> false
IO.puts Math.zero?([1, 2, 3]) #=> ** (FunctionClauseError)
IO.puts Math.zero?(0.0)       #=> ** (FunctionClauseError)
```

Similar to constructs like `if`, named functions support both `do:` and `do/end` block syntax, as we learned `do/end` is just a convenient syntax for the keyword list format. For example, we can edit `math.exs` to look like this:

```elixir
defmodule Math do
  def zero?(0), do: true
  def zero?(x) when is_integer(x), do: false
end
```

### Function capturing
Can actually be used to retrieve a named function as a function type. (Given the file)

```elixir
iex> Math.zero?(0)
true
iex> fun = &Math.zero?/1
&Math.zero?/1
iex> is_function(fun)
true
iex> fun.(0)
true
```

Local or imported functions, like `is_function/1`, can be captured without the module:

```elixir
iex> &is_function/1
&:erlang.is_function/1
iex> (&is_function/1).(fun)
true
```

Note the capture syntax can also be used as a shortcut for creating functions:

```elixir
iex> fun = &(&1 + 1)
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> fun.(1)
2
```

The `&1` represents the first argument passed into the function.  
The above is the same as `fn x -> x + 1 end`. It's useful for short function definitions.

If you want to capture a function from a module, you can do `&Module.function()`:

```elixir
iex> fun = &List.flatten(&1, &2)
&List.flatten/2
iex> fun.([1, [[2], 3]], [4, 5])
[1, 2, 3, 4, 5]
```

`&List.flatten(&1, &2)` is the same as writing `fn(list, tail) -> List.flatten(list, tail) end` which in this case is equivalent to `&List.flatten/2`. You can read more about the capture operator `&` in [the `Kernel.SpecialForms` documentation](http://elixir-lang.org/docs/stable/elixir/Kernel.SpecialForms.html#&/1).

### Default arguments
Named functions default arguments:

```elixir
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

If a function with default values has multiple clauses, it is required to create a function head (without an actual body) for declaring defaults:

```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
IO.puts Concat.join("Hello")               #=> Hello
```

+ Default values won’t be evaluated during the function definition
+ When using default values, one must be careful to avoid overlapping function definitions

## Recursion
* * *

### Loops through recursion
Beautifully without mutating:

```elixir
defmodule Recursion do
  def print_multiple_times(msg, n) when n <= 1 do
    IO.puts msg
  end

  def print_multiple_times(msg, n) do
    IO.puts msg
    print_multiple_times(msg, n - 1)
  end
end

Recursion.print_multiple_times("Hello!", 3)
# Hello!
# Hello!
# Hello!
```

### Reduce and map algorithms
Let’s now see how we can use the power of recursion to sum a list of numbers:

```elixir
defmodule Math do
  def sum_list([head | tail], accumulator) do
    sum_list(tail, head + accumulator)
  end

  def sum_list([], accumulator) do
    accumulator
  end
end

IO.puts Math.sum_list([1, 2, 3], 0) #=> 6
```

**The process of taking a list and *reducing* it down to one value is known as a *reduce algorithm* and is central to functional programming.**  

What if we instead want to double all of the values in our list?

```elixir
defmodule Math do
  def double_each([head | tail]) do
    [head * 2 | double_each(tail)]
  end

  def double_each([]) do
    []
  end
end
```

```elixir
$ iex math.exs
iex> Math.double_each([1, 2, 3]) #=> [2, 4, 6]
```

Here we have used recursion to traverse a list, doubling each element and returning a new list. The process of taking a list and *mapping* over it is known as a *map algorithm*.

Recursion and [tail call](https://en.wikipedia.org/wiki/Tail_call) optimization are an important part of Elixir. **However, when programming in Elixir you will rarely use recursion as above.** The `Enum` [module](http://elixir-lang.org/docs/stable/elixir/Enum.html), (next chapter), already provides many conveniences for working with lists. For instance, the examples above could be written as:

```elixir
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
6
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
[2, 4, 6]
```

Or, using the capture syntax:

```elixir
iex> Enum.reduce([1, 2, 3], 0, &+/2)
6
iex> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]
```

## Enumerables and Streams
* * *

[The `Enum` module](http://elixir-lang.org/docs/stable/elixir/Enum.html) provides a huge range of functions to transform, sort, group, filter and retrieve items from enumerables. It is one of the modules developers use frequently in their Elixir code.

 For specific operations, like inserting and updating particular elements, you may need to reach for modules specific to the data type. For example, if you want to insert an element at a given position in a list, you should use the `List.insert_at/3` function from the List module.

 We say the functions in the Enum module are polymorphic because they can work with diverse data types. In particular, the functions in the Enum module can work with any data type that implements the [Enumerable protocol](http://elixir-lang.org/docs/stable/elixir/Enumerable.html).

 BTW, Elixir also provides ranges:

```elixir
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

### Eager vs Lazy
All the functions in the `Enum` module are eager. Many functions expect an enumerable and return a list back. This means that when performing multiple operations with `Enum`, each operation is going to generate an intermediate list until we reach the result:

```elixir
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

### The pipe operator
The `|>` symbol used in the snippet above is the [pipe operator](http://elixir-lang.org/docs/stable/elixir/Kernel.html#%7C%3E/2): it simply takes the output from the expression on its left side and passes it as the first argument to the function call on its right side. It’s similar to the Unix `|` operator.

### Streams
As an alternative to `Enum`, Elixir provides [the `Stream` module](http://elixir-lang.org/docs/stable/elixir/Stream.html) which supports lazy operations:

```elixir
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

In the example above, `1..100_000 |> Stream.map(&(&1 * 3))` returns a data type, an actual stream, that represents the `map` computation over the range `1..100_000`. Furthermore, they are composable because we can pipe many stream operations:

```elixir
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<[enum: 1..100000, funs: [...]]>
```

Instead of generating intermediate lists, streams build a series of computations that are invoked only when we pass the underlying stream to the `Enum` module. Streams are useful when working with large, *possibly infinite*, collections.

It also provides functions for creating streams. For example, `Stream.cycle/1` can be used to create a stream that cycles a given enumerable infinitely:

```elixir
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

Another interesting function is `Stream.resource/3` which can be used to wrap around resources, guaranteeing they are opened right before enumeration and closed afterwards, even in the case of failures. For example, we can use it to stream a file:

```elixir
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

The example above will fetch the first 10 lines of the file you have selected. This means streams can be very useful for handling large files or even slow resources like network resources.

## Processes
* * *

In Elixir, all code runs inside processes. Processes are isolated from each other, run concurrent to one another and communicate via message passing.

### `spawn`
The basic mechanism for spawning new processes is with the auto-imported `spawn/1` function:

```elixir
iex> pid = spawn fn -> 1 + 2 end
#PID<0.43.0>
iex> Process.alive?(pid)
false
```

We can retrieve the PID of the current process by calling `self/0`.

### `send` and `receive`
We can send messages to a process with `send/2` and receive them with `receive/1`:

```elixir
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

When a message is sent to a process, the message is stored in the process mailbox. The `receive/1` block goes through the current process mailbox searching for a message that matches any of the given patterns. `receive/1` supports guards and many clauses, such as `case/2`.

If there is no message in the mailbox matching any of the patterns, the current process will wait until a matching message arrives. A timeout can also be specified (A timeout of `0` can be given when you already expect the message to be in the mailbox):

```elixir
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

Let’s put it all together and send messages between processes:

```elixir
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>
```

While in the shell, you may find the helper `flush/0` quite useful. It flushes and prints all the messages in the mailbox.

```elixir
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

### Links
The most common form of spawning in Elixir is actually via `spawn_link/1`. Before we show an example with `spawn_link/1`, let’s try to see what happens when a process fails:

```elixir
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Process #PID<0.58.00> raised an exception
** (RuntimeError) oops
    :erlang.apply/2
```

It merely logged an error but the spawning process is still running. That’s because processes are isolated. If we want the failure in one process to propagate to another one, we should link them. This can be done with `spawn_link/1`:

```elixir
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

In Elixir applications, we often link our processes to supervisors which will detect when a process dies and start a new process in its place. This is only possible because processes are isolated and don’t share anything by default. And since processes are isolated, there is no way a failure in a process will crash or corrupt the state of another.

While other languages would require us to catch/handle exceptions, in Elixir we are actually fine with letting processes fail because we expect supervisors to properly restart our systems. “Failing fast” is a common philosophy when writing Elixir software!

`spawn/1 and spawn_link/1` are the basic primitives for creating processes in Elixir. Although we have used them exclusively so far, most of the time we are going to use abstractions that build on top of them. Let’s see the most common one, called tasks.

### Tasks
Tasks build on top of the spawn functions to provide better error reports and introspection:

```elixir
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
** (RuntimeError) oops
    (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
    (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []

```

Instead of `spawn/1` and `spawn_link/1`, we use `Task.start/1` and `Task.start_link/1` to return `{:ok, pid}` rather than just the PID. This is what enables Tasks to be used in supervision trees. Furthermore, Task provides convenience functions, like `Task.async/1` and `Task.await/1`, and functionality to ease distribution.

### State
We haven’t talked about state so far in this guide. If you are building an application that requires state, for example, to keep your application configuration, or you need to parse a file and keep it in memory, where would you store it?

Processes are the most common answer to this question. We can write processes that loop infinitely, maintain state, and send and receive messages. As an example, let’s write a module that starts new processes that work as a key-value store in a file named `kv.exs`:

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

Let’s give it a try by running `$ iex kv.exs`:

```elixir
iex> {:ok, pid} = KV.start_link
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
nil
:ok
```

At first, the process map has no keys, so sending a `:get` message and then flushing the current process inbox returns `nil`. Let’s send a `:put` message and try it again:

```elixir
iex> send pid, {:put, :hello, :world}
{:put, :hello, :world}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
:ok
```

Notice how the process is keeping a state and we can get and update this state by sending the process messages. In fact, any process that knows the `pid` above will be able to send it messages and manipulate the state.

It is also possible to register the `pid`, giving it a name, and allowing everyone that knows the name to send it messages:

```elixir
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
:ok
```

Using processes around state and name registering are very common patterns in Elixir applications. However, most of the time, we won’t implement those patterns manually as above, but by using one of the many abstractions that ship with Elixir. For example, Elixir provides [agents](http://elixir-lang.org/docs/stable/elixir/Agent.html), which are simple abstractions around state:

```elixir
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

A `:name` option could also be given to `Agent.start_link/2` and it would be automatically registered. Besides agents, Elixir provides an API for building generic servers (called GenServer), tasks and more, all powered by processes underneath. Those, along with supervision trees, will be explored with more detail in the **Mix and OTP guide** which will build a complete Elixir application from start to finish.

## IO and the file system
* * *

The IO module is the main mechanism in Elixir for reading and writing to standard input/output (`:stdio`), standard error (`:stderr`), files and other IO devices. Usage of the module is pretty straightforward:

```elixir
iex> IO.puts "hello world"
hello world
:ok
iex> IO.gets "yes or no? "
yes or no? yes
"yes\n"
```

By default, functions in the `IO` module read from the standard input and write to the standard output. We can change that by passing, for example, `:stderr` as an argument (in order to write to the standard error device):

```elixir
iex> IO.puts :stderr, "hello world"
hello world
:ok
```

### The `File` module
The `File` module contains functions that allow us to open files as IO devices. By default, files are opened in binary mode, which requires developers to use the specific `IO.binread/2` and `IO.binwrite/2` functions from the `IO` module:

```elixir
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
iex> IO.binwrite file, "world"
:ok
iex> File.close file
:ok
iex> File.read "hello"
{:ok, "world"}
```

A file can also be opened with `:utf8` encoding, which tells the `File` module to interpret the bytes read from the file as UTF-8-encoded bytes.

It also provides Unix like functions: `File.rm/1`, `File.mkdir/1`, `File.mkdir_p/1`, etc. (Checkout the [module documentation](http://elixir-lang.org/docs/stable/elixir/File.html)) Notice the variations with a trailing bang `!`.

```elixir
iex> File.read "hello"
{:ok, "world"}
iex> File.read! "hello"
"world"
iex> File.read "unknown"
{:error, :enoent}
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
```

Notice that when the file does not exist, the version with `!` raises an error. The version without `!` is preferred when you want to handle different outcomes using pattern matching:

```elixir
case File.read(file) do
  {:ok, body}      -> # do something with the `body`
  {:error, reason} -> # handle the error caused by `reason`
end
```

However, if you expect the file to be there, the bang variation is more useful as it raises a meaningful error message. Avoid writing: `{:ok, body} = File.read(file)` as, in case of an error, `File.read/1` will return `{:error, reason}` and the pattern matching will fail. You will still get the desired result (a raised error), but the message will be about the pattern which doesn’t match (thus being cryptic in respect to what the error actually is about).

Therefore, if you don’t want to handle the error outcomes, prefer using `File.read!/1`.

### The `Path` module
The majority of the functions in the `File` module expect paths as arguments. Most commonly, those paths will be regular binaries. The [Path](http://elixir-lang.org/docs/stable/elixir/Path.html) module provides facilities for working with such paths:

```elixir
iex> Path.join("foo", "bar")
"foo/bar"
iex> Path.expand("~/hello")
"/Users/jose/hello"
```

### Processes and group leaders
By modelling IO devices with processes, the Erlang VM allows different nodes in the same network to exchange file processes in order to read/write files in between nodes. Of all IO devices, there is one that is special to each process: the **group leader**.

When you write to `:stdio`, you are actually sending a message to the group leader, which writes to the standard-output file descriptor:

```elixir
iex> IO.puts :stdio, "hello"
hello
:ok
iex> IO.puts Process.group_leader, "hello"
hello
:ok
```

The group leader can be configured per process and is used in different situations. For example, when executing code in a remote terminal, it guarantees messages in a remote node are redirected and printed in the terminal that triggered the request.

## `alias`, `require` and `import`
* * *

```elixir
# Alias the module so it can be called as Bar instead of Foo.Bar
alias Foo.Bar, as: Bar

# Ensure the module is compiled and available (usually for macros)
require Foo

# Import functions from Foo so they can be called without the `Foo.` prefix
import Foo

# Invokes the custom code defined in Foo as an extension point
use Foo
```

We are going to explore them in detail now. Keep in mind the first three are called directives because they have **lexical scope** , while `use` is a common extension point.

### `alias`

```elixir
defmodule Math do
  alias Math.List, as: List
end
```

From now on, any reference to `List` will automatically expand to `Math.List`. In case one wants to access the original `List`, it can be done by prefixing the module name with `Elixir.`:

```elixir
List.flatten             #=> uses Math.List.flatten
Elixir.List.flatten      #=> uses List.flatten
Elixir.Math.List.flatten #=> uses Math.List.flatten
```

**Note:** *All modules defined in Elixir are defined inside a main Elixir namespace. However, for convenience, you can omit “Elixir.” when referencing them.*  

Note that alias is lexically scoped, which allows you to set aliases inside specific functions:

```elixir
defmodule Math do
  def plus(a, b) do
    alias Math.List
    # ...
  end

  def minus(a, b) do
    # ...
  end
end
```

### `require`
In order to use a macro, we need to guarantee its module and implementation are available during compilation. This is done with the require directive:

```elixir
iex> Integer.is_odd(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.is_odd/1
iex> require Integer
Integer
iex> Integer.is_odd(3)
true
```

Note that like the `alias` directive, `require` is also **lexically scoped**.

### `import`
We use `import` whenever we want to easily access functions or macros from other modules without using the fully-qualified name. For instance, if we want to use the `duplicate/2` function from the `List` module several times, we can import it:

```elixir
iex> import List, only: [duplicate: 2]
List
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

*`:except` could also be given as an option.*
`import` also supports `:macros` and `:functions` to be given to `:only`. For example, to import all macros, one could write: `import Integer, only: :macros`.

Note that `import` is **lexically scoped** too. This means that we can import specific macros or functions inside function definitions:

```elixir
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    duplicate(:ok, 10)
  end
end
```

Note that `import`ing a module automatically `require`s it.

### `use`
Although not a directive, `use` is a macro tightly related to `require` that allows you to use a module in the current context. The `use` macro is frequently used by developers to bring external functionality into the current lexical scope, often modules.

For example, in order to write tests using the ExUnit framework, a developer should use the `ExUnit.Case module`:

```elixir
defmodule AssertionTest do
  use ExUnit.Case, async: true

  test "always pass" do
    assert true
  end
end
```

## Module attributes
* * *

+ They serve to annotate the module, often with information to be used by the user or the VM.
+ They work as constants.
+ They work as a temporary module storage to be used during compilation.

### As annotations
Elixir has a handful of reserved attributes. Here are a few of them, the most commonly used ones:

+ `@moduledoc` - provides documentation for the current module.
+ `@doc` - provides documentation for the function or macro that follows the attribute.
+ `@behaviour` - (notice the British spelling) used for specifying an OTP or user-defined behaviour.
+ `@before_compile` - provides a hook that will be invoked before the module is compiled. This makes it possible to inject functions inside the module exactly before compilation.

### As constants

```elixir
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

**Note:** *Unlike Erlang, user defined attributes are not stored in the module by default. The value exists only during compilation time. A developer can configure an attribute to behave closer to Erlang by calling `Module.register_attribute/3`.*

Attributes can also be read inside functions:

```elixir
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

**Every time an attribute is read inside a function, a snapshot of its current value is taken. In other words, the value is read at compilation time and not at runtime.**

### As temporary storage
Attributes can be used to do so. The [ExUnit framework](http://elixir-lang.org/docs/stable/ex_unit/) which uses module attributes as annotation and storage:

```elixir
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

## Structs
* * *

Structs are extensions built on top of maps that provide compile-time checks and default values.

```elixir
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

Structs take the name of the module they’re defined in. In the example above, we defined a struct named `User`. Let's create one so:

```elixir
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

To access and update:

```elixir
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

When using the update syntax (`|`), the VM is aware that no new keys will be added to the struct, allowing the maps underneath to share their structure in memory.

Structs can also be used in pattern matching, both for matching on the value of specific keys as well as for ensuring that the matching value is a struct of the same type as the matched value.

```elixir
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

**Note:** *Structs are bare maps underneath. As maps, structs store a “special” field named `__struct__` that holds the name of the struct. We referred to structs as bare maps because none of the protocols implemented for maps are available for structs. For example, you can neither enumerate nor access a struct. However, since structs are just maps, they work with the functions from the `Map` module.*

**Structs alongside protocols provide one of the most important features for Elixir developers: data polymorphism.**

## Protocols
* * *

Protocols are a mechanism to achieve polymorphism in Elixir. Dispatching on a protocol is available to any data type as long as it implements the protocol.

Let's implement that to specify a `blank?` protocol that returns a boolean for other data types that should be considered blank.

```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

The protocol expects a function called `blank?` that receives one argument to be implemented. We can implement this protocol for different Elixir data types as follows:

```elixir
# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# Just empty map is blank
defimpl Blank, for: Map do
  # Keep in mind we could not pattern match on %{} because
  # it matches on all maps. We can however check if the size
  # is zero (and size is a fast operation).
  def blank?(map), do: map_size(map) == 0
end

# Just the atoms false and nil are blank
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

```elixir
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

And we would do so for all native data types. The types available are:

+ `Atom`
+ `BitString`
+ `Float`
+ `Function`
+ `Integer`
+ `List`
+ `Map`
+ `PID`
+ `Port`
+ `Reference`
+ `Tuple`

Manually implementing protocols for all types can quickly become repetitive and tedious. In such cases, Elixir provides two options: we can explicitly derive the protocol implementation for our types or automatically implement the protocol for all types. In both cases, we need to implement the protocol for `Any`.

### Deriving

```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

```elixir
defmodule DeriveUser do
  @derive Blank
  defstruct name: "john", age: 27
end
```

### Fallback to `Any`
Another alternative to `@derive` is to explicitly tell the protocol to fallback to `Any` when an implementation cannot be found. This can be achieved by setting `@fallback_to_any` to `true` in the protocol definition:

```elixir
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```

## Comprehensions
* * *

Declared by `for`:

```elixir
# Map a list of integers into their squared values.
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

**A comprehension is made of three parts: generators, filters and collectables.**

### Generators and filters
In the expression above, `n <- [1, 2, 3, 4]` is the **generator**. It is literally generating values to be used in the comprehension. Any enumerable can be passed in the right-hand side of the generator expression:

```elixir
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

It also supports pattern matching:

```elixir
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

Alternatively to pattern matching, filters can be used to select some particular elements:

```elixir
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..5, multiple_of_3?.(n), do: n * n
[0, 9]
```

Comprehensions also allow multiple generators and filters to be given. Here is an example that receives a list of directories and gets the size of each file in those directories:

```elixir
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.stat!(path).size
end
```

Calculating the cartesian product of two lists:

```elixir
iex> for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
[a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

A more advanced example of multiple generators and filters is Pythagorean triples. A Pythagorean triple is a set of positive integers such that `a*a + b*b = c*c`, let’s write a comprehension in a file named `triple.exs`:

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

Outputs:

```elixir
$ iex triple.exs
iex> Triple.pythagorean(5)
[]
iex> Triple.pythagorean(12)
[{3, 4, 5}, {4, 3, 5}]
iex> Triple.pythagorean(48)
[{3, 4, 5}, {4, 3, 5}, {5, 12, 13}, {6, 8, 10}, {8, 6, 10}, {8, 15, 17},
 {9, 12, 15}, {12, 5, 13}, {12, 9, 15}, {12, 16, 20}, {15, 8, 17}, {16, 12, 20}]
```

 Take a closer look on how it performs without filters:

```elixir
 defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        do: IO.puts "#{a} #{b} #{c}"
  end
end

Triple.pythagorean(10)
```

Outputs:

```elixir
$ elixir triple.exs
1 1 1
1 1 2
1 1 3
1 1 4
1 1 5
1 1 6
1 1 7
1 1 8
1 1 9
1 1 10
1 2 1
1 2 2
[...]
```

## Sigils
* * *

Is one of the mechanisms provided by the language for working with textual representations (also allowing extensibility). Sigils start with the tilde (`~`) character which is followed by a letter (which identifies the sigil) and then a delimiter; optionally, modifiers can be added after the final delimiter.

The most common sigil in Elixir is `~r`, which is used to create [regular expressions](https://en.wikipedia.org/wiki/Regular_Expressions):

```elixir
# A regular expression that matches strings which contain "foo" or "bar":
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

So far, all examples have used `/` to delimit a regular expression. However sigils support 8 different delimiters:

```elixir
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

### Strings
The `~s` sigil is used to generate strings, like double quotes are. The `~s` sigil is useful, for example, when a string contains both double and single quotes:

```elixir
iex> ~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"
```

### Char lists

```elixir
iex> ~c(this is a char list containing 'single quotes')
'this is a char list containing \'single quotes\''
```

### Word lists (words are just regular strings)

```elixir
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

The `~w` sigil also accepts the `c`, `s` and `a` modifiers (for char lists, strings and atoms, respectively), which specify the data type of the elements of the resulting list:

```elixir
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

### Interpolation and escaping in sigils
Besides lowercase sigils, Elixir supports uppercase sigils to deal with escaping characters and interpolation.

```elixir
iex> ~s(String with escape codes \x26 #{"inter" <> "polation"})
"String with escape codes & interpolation"
iex> ~S(String without escape codes \x26 without #{interpolation})
"String without escape codes \\x26 without \#{interpolation}"
```

The following escape codes can be used in strings and char lists:

+ `\"` – double quote
+ `\'` – single quote
+ `\\` – single backslash
+ `\a`– bell/alert
+ `\b` – backspace
+ `\d` - delete
+ `\e` - escape
+ `\f` - form feed
+ `\n` – newline
+ `\r` – carriage return
+ `\s` – space
+ `\t` – tab
+ `\v` – vertical tab
+ `\0` - null byte
+ `\xDD` - represents a single byte in hexadecimal (such as \x13)
+ `\uDDDD` and `\u{D...}` - represents a Unicode codepoint in hexadecimal (such as `\u{1F600}`)

Also supports herecods:

```elixir
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

Writing escape characters in documentation would soon become error prone because of the need to double-escape some characters. By using `~S`, this problem can be avoided altogether:

```elixir
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

### Custom sigils
We can also provide our own sigils by implementing functions that follow the `sigil_{identifier}` pattern. For example, let’s implement the `~i` sigil that returns an integer (with the optional `n` modifier to make it negative):

```elixir
iex> defmodule MySigils do
...>   def sigil_i(string, []), do: String.to_integer(string)
...>   def sigil_i(string, [?n]), do: -String.to_integer(string)
...> end
iex> import MySigils
iex> ~i(13)
13
iex> ~i(42)n
-42
```

Sigils can also be used to do compile-time work with the help of macros. For example, regular expressions in Elixir are compiled into an efficient representation during compilation of the source code, therefore skipping this step at runtime. If you’re interested in the subject, we recommend you learn more about macros and check out how sigils are implemented in the `Kernel` module (where the `sigil_*` functions are defined).

## `try`, `catch` and `rescue`
* * *

### Errors
Errors (or *exceptions*) are used when exceptional things happen in the code. A sample error can be retrieved by trying to add a number into an atom:

```elixir
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

A runtime error can be raised any time by using `raise/1`:

```elixir
iex> raise "oops"
** (RuntimeError) oops
```

Other errors can be raised with `raise/2` passing the error name and a list of keyword arguments:

```elixir
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument fo
```

You can also define your own errors by creating a module and using the `defexception` construct inside it; this way, you’ll create an error with the same name as the module it’s defined in. The most common case is to define a custom exception with a message field:

```elixir
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

Errors can be **rescued** using the `try/rescue` construct:

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

If you don’t have any use for the error, you don’t have to provide it:

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   RuntimeError -> "Error!"
...> end
"Error!"
```

In Elixir, we avoid using `try/rescue` because **we don’t use errors for control flow**. We take errors literally: they are reserved for unexpected and/or exceptional situations. In case you actually need flow control constructs, *throws* should be used. That’s what we are going to see next.

### Throws
In Elixir, a value can be thrown and later be caught. `throw` and `catch` are reserved for situations where it is not possible to retrieve a value unless by using `throw` and `catch`.

Those situations are quite uncommon in practice except when interfacing with libraries that do not provide a proper API. For example, let’s imagine the `Enum` module did not provide any API for finding a value and that we needed to find the first multiple of 13 in a list of numbers:

```elixir
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

Since `Enum` **does** provide a proper API, in practice `Enum.find/2` is the way to go:

```elixir
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

### Exits
All Elixir code runs inside processes that communicate with each other. When a process dies of “natural causes” (e.g., unhandled exceptions), it sends an exit signal. A process can also die by explicitly sending an `exit` signal:

```elixir
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

`exit` can also be “caught” using `try/catch`:

```elixir
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

**Using `try/catch` is already uncommon and using it to catch exits is even more rare.**

### After
Sometimes it’s necessary to ensure that a resource is cleaned up after some action that could potentially raise an error. The `try/after` construct allows you to do that. For example, we can open a file and use an `after` clause to close it–even if something goes wrong:

```elixir
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

Sometimes you may want to wrap the entire body of a function in a `try` construct, often to guarantee some code will be executed afterwards. In such cases, Elixir allows you to omit the `try` line:

```elixir
iex> defmodule RunAfter do
...>   def without_even_trying do
...>     raise "oops"
...>   after
...>     IO.puts "cleaning up!"
...>   end
...> end
iex> RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

## Typespecs and behaviours
* * *

### Types and specs
It's used for:

+ declaring custom data types;
+ declaring typed function signatures (specifications).

#### Function specifications

```elixir
@spec round(number) :: integer
def round(number), do: # implementation...
```

#### Defining custom types

```elixir
defmodule LousyCalculator do
  @typedoc """
  Just a number followed by a string.
  """
  @type number_with_remark :: {number, String.t}

  @spec add(number, number) :: number_with_remark
  def add(x, y), do: {x + y, "You need a calculator to do that?"}

  @spec multiply(number, number) :: number_with_remark
  def multiply(x, y), do: {x * y, "It is like addition on steroids."}
end
```

Custom types defined through `@type` are exported and available outside the module they’re defined in:

```elixir
defmodule QuietCalculator do
  @spec add(number, number) :: number
  def add(x, y), do: make_quiet(LousyCalculator.add(x, y))

  @spec make_quiet(LousyCalculator.number_with_remark) :: number
  defp make_quiet({num, _remark}), do: num
end
```

If you want to keep a custom type private, you can use the `@typep` directive instead of `@type`.

#### Static code analysis
Typespecs are not only useful to developers and as additional documentation. The Erlang tool [Dialyzer](http://www.erlang.org/doc/man/dialyzer.html), for example, uses typespecs in order to perform static analysis of code. That’s why, in the `QuietCalculator` example, we wrote a spec for the `make_quiet/1` function even if it was defined as a private function.

### Behaviours
Behaviours provide a way to:

+ Define a set of functions that have to be implemented by a module;
+ Ensure that a module implements all the functions in that set.

#### Defining behaviours

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```

Modules adopting the `Parser` behaviour will have to implement all the functions defined with the `@callback` directive. As you can see, `@callback` expects a function name but also a function specification like the ones used with the `@spec` directive we saw above.

Adopting a behaviour is straightforward:

```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

```elixir
defmodule YAMLParser do
  @behaviour Parser

  def parse(str), do: # ... parse YAML
  def extensions, do: ["yml"]
end
```

## Erlang libraries
* * *

As you grow more proficient in Elixir, you may want to explore the Erlang [STDLIB Reference Manual](http://erlang.org/doc/apps/stdlib/index.html) in more detail. Check below some of the most widely used libraries.

### The binary module
The built-in Elixir String module handles binaries that are UTF-8 encoded. [The binary module](http://erlang.org/doc/man/binary.html) is useful when you are dealing with binary data that is not necessarily UTF-8 encoded.

### Formatted text output
Elixir does not contain a function similar to `printf` found in C and other languages. Luckily, the Erlang standard library functions `:io.format/2` and `:io_lib.format/2` may be used. The first formats to terminal output, while the second formats to an iolist. The format specifiers differ from `printf`, [refer to the Erlang documentation for details](http://erlang.org/doc/man/io.html#format-1).

### The crypto module
[The crypto module](http://erlang.org/doc/man/crypto.html) contains hashing functions, digital signatures, encryption and more:

```elixir
iex> Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

The `:crypto` module is not part of the Erlang standard library, but is included with the Erlang distribution. This means you must list `:crypto` in your project’s applications list whenever you use it. To do this, edit your `mix.exs` file to include.

### The digraph module
[The digraph module](http://erlang.org/doc/man/digraph.html) (as well as [digraph_utils](http://erlang.org/doc/man/digraph_utils.html)) contains functions for dealing with directed graphs built of vertices and edges. After constructing the graph, the algorithms in there will help finding for instance the shortest path between two vertices, or loops in the graph.

### Erlang Term Storage
The modules [`ets`](http://erlang.org/doc/man/ets.html) and [`dets`](http://erlang.org/doc/man/dets.html) handle storage of large data structures in memory or on disk respectively.

ETS lets you create a table containing tuples. By default, ETS tables are protected, which means only the owner process may write to the table but any other process can read. ETS has some functionality to be used as a simple database, a key-value store or as a cache mechanism.

The functions in the `ets` module will modify the state of the table as a side-effect.

```elixir
iex> table = :ets.new(:ets_test, [])
# Store as tuples with {name, population}
iex> :ets.insert(table, {"China", 1_374_000_000})
iex> :ets.insert(table, {"India", 1_284_000_000})
iex> :ets.insert(table, {"USA", 322_000_000})
iex> :ets.i(table)
<1   > {"USA", 322000000}
<2   > {"China", 1_374_000_000}
<3   > {"India", 1_284_000_000}
```

### The math module
[The `math` module](http://erlang.org/doc/man/math.html) contains common mathematical operations covering trigonometry, exponential and logarithmic functions.

### The queue module
[The `queue`](http://erlang.org/doc/man/queue.html) is a data structure that implements (double-ended) FIFO (first-in first-out) queues efficiently:

```elixir
iex> q = :queue.new
iex> q = :queue.in("A", q)
iex> q = :queue.in("B", q)
iex> {value, q} = :queue.out(q)
iex> value
{:value, "A"}
iex> {value, q} = :queue.out(q)
iex> value
{:value, "B"}
iex> {value, q} = :queue.out(q)
iex> value
:empty
```

### The rand module
[`rand` has functions](http://erlang.org/doc/man/rand.html) for returning random values and setting the random seed.

```elixir
iex> :rand.uniform()
0.8175669086010815
iex> _ = :rand.seed(:exs1024, {123, 123534, 345345})
iex> :rand.uniform()
0.5820506340260994
iex> :rand.uniform(6)
6
```

### The zip and zlib modules
[The `zip` module](http://erlang.org/doc/man/zip.html) lets you read and write zip files to and from disk or memory, as well as extracting file information.

## References
* * *

+ [http://elixir-lang.org/getting-started/introduction.html/](http://elixir-lang.org/getting-started/introduction.html)
+ Cover picture: *The Banquet in the Pine Forest (1482/3) is the third painting in Sandro Botticelli's series The Story of Nastagio degli Onesti, which illustrates events from the Eighth Story of the Fifth Day.* (Public Domain)
