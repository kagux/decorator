# Elixir function decorators

A decorator is a macro which is executed while the function is
defined. It can be used to add extra functionality to Elixir
functions. The runtime overhead of a function decorator is zero, as it
is executed on compile time.

Examples of function decorators include: loggers, instrumentation
(timing), precondition checks, et cetera.


## Installation

Add `decorator` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:decorator, "~> 0.0"}]
end
```

You can now define your function decorators.

## Usage

Function decorators are macros which you put just before defining a
function. It looks like this:

```elixir
defmodule MyModule do
  use PrintDecorator

  print()
  def square(a) do
    a * a
  end
end
```

Now whenever you call `MyModule.square()`, you'll see the message: `Function called: square` in the console.

Defining the decorator is pretty easy. Create a module in which you
*use* the `Decorator.Define` module, passing in the decorator name and
arity, or more than one if you want.

The following defines a print() decorator which prints a message every time the function is called:

```elixir
defmodule PrintDecorator do
  use Decorator.Define, [print: 0]

  def print(body, context) do
    quote do
      IO.puts("Function called: " <> Atom.to_string(unquote(context.name)))
      unquote(body)
    end
  end

end
```

Note that `print()` here is a function, not a macro! The actual macro
has arity 0, and will be imported in the caller module. The decorator
module's `print()` function gets called when the actual function is
being defined.

The arguments to the decorator function are the function's body (the
AST), as well as a `context` argument which holds information like the
function's name, defining module, arity and the arguments AST.


### Compile-time arguments

Decorators can have compile-time arguments passed into the decorator
macros.

For instance, you could let the print function only print when a certain logging level has been set:

```elixir
print(:debug)
def foo() do
 ...
```

In this case, you specify the arity 1 for the decorator:

```elixir
defmodule PrintDecorator do
  use Decorator.Define, [print: 1]
```

And then your `print()` decorator function gets the level passed in as
the first argument:

```elixir
  def print(level, body, context) do
    # ...
  end
```
