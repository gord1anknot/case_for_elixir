## this talk's goals

- describes the [Erlang](http://erlang.org) runtime system through [Elixir](http://elixir-lang.org)
- describes the unique benefits of the Erlang runtime system as a platform
- not really about Erlang, the language, except to compare it to Elixir

Note:
- not the phoenix tutorial expected
    - that wouldn't have been as audience appropriate
- failures to meet these goals implies presenter error

- - -

## about me

- graduated 2011
- started work @ pearson education 2012
- working with Erlang in production for 1 year

Note:

total dork since `tempus immemorial`

case in point: the man just said `tempus immemorial`

- - -

## history + architectural goals

- industry: telecom in the ‘80s
- planning the construction of a very large software system, 1 million+ loc

Note:
- "I believe that this company should commit itself to achieving the goal, before this decade is out, of putting some sort of portable telephone in Zack Morris' hands"

- - -

## problem description

- multiple teams will work on the codebase. integration tests must occur later.
    - corollary: build and debugging tooling must be robust, and reach target scale
    - one example: compile times shouldn’t increase exponentially over the size of the code

- -

## problem description pt. 2

- the telephone switching code will be deployed into physical hardware
- hardware will be physically deployed in many remote locations


- -

## problem description pt. 3

- both deployment and later on site maintenance is very costly
- it is expected that software cannot be proven correct in advance of deployment
- the software should be able to continue operating despite defects inside itself

- - -

## solution part 1: let it crash

- Erlang's architecture is based on the observation that most software defects are resolved with a reset
- Create software that has the capability to reset it’s own modules when errors are detected in those modules

Note: in order to do this, concurrent tasks must be addressable using a name. sorry go-lang, goroutines can’t do this

- -

<img src="img/failed-me.jpg" alt="darth vader does erlang"/>

- it's more fun than it sounds

Note:
- Erlang software libraries and documentation encourages developers to be skeptical and cynical of 3rd party modules they interact with, to not let errors in their dependencies cause their own modules to behave incorrectly
- this is the opposite of typical waterfall inspired all or nothing approach to quality

- -

## solution part 2: aggressively abstract away platform

- Erlang's designers don’t know what hardware architecture will be used
- Erlang's designers know the performance characteristics of the platform
    - number or size of cpus, memory, disk, network interfaces


Note:
- operators must have the capabilities needed to get the software working again to minimize mttr. these features built into the software
- for more, see Armstrong’s problem statement from his 2003 thesis


- - -


### two assumptions this argument makes about your problem

- the profitability of your enterprise is limited primarily by the architectural approach of the technology in use <!-- .element: class="fragment" data-fragment-index="1" -->

- you have experienced one or more of these problems in production: <!-- .element: class="fragment" data-fragment-index="2" -->
    - your software is not performant <!-- .element: class="fragment" data-fragment-index="3" -->
    - it's difficult to reason about whether or not the code you write will be performant <!-- .element: class="fragment" data-fragment-index="4" -->
    - your apps do not scale across cores and servers like you think they need to <!-- .element: class="fragment" data-fragment-index="5" -->

Note:
- not technologically related limitations: recruiting, organizational politics, etc.
    - obviously the benefit of moving to erlang needs to outweigh the costs
    - the first type of stuff won’t necessarily stop Elixir from being a good idea, but I’m too green to give much useful advice on that
    - Spruce Goose metaphor
- performant is a loose definition here, could be cpu, memory, power consumption, latency, etc.
    - your software is performant (at the moment), but uses so many gimmicks to get there that it is unmaintainable given a sufficient period of time
    - example: given a Java app, you spend more time tuning your double-checked locking implementation than you do coding features
    - you have an opportunity to re-write the software in another language, but don’t want to do it that way again
- you need it to scale in a platform neutral way
    - your code needs to outlive the hardware it’s running on without changes, even if that means it doesn’t use that hardware to 100% of all currently possible efficiency
    - 3 ubuntu precise servers with 8 cores should be the same as 24 coreos servers with 24 cores. servers are cheaper than code rewrites. Amdahl’s law is good enough, and more than you expected using existing solutions.
    - it’s OK if your solution is 80% (number totally made up) as fast as a platform specific, hand tuned solution, as long as you eventually realize Amdahl’s law.
    - example: a Java language solution using LMAX disruptor, although performant, is platform specific. you need particular system architectures in order to yield any benefit. Present generation intel chips with a known number of cores above a big L3 cache, and certain consistency properties in the cache, are needed for that to work

- -

### Regardless of your problem, I'll make the argument that:

- platforms that you will eventually use will have adopted many of these concepts <!-- .element: class="fragment" data-fragment-index="1" -->
- Erlang's history is a good study in how to effectively evolve an architecture fit for purpose <!-- .element: class="fragment" data-fragment-index="2" -->


- - -

# the runtime

- -

## glossary

*let's drink a tall glass of the kool aid*

- process === fine grained task
- concurrency vs parallelism
- soft real time

Note:
- process
    - the word process is used in the sense of the the original definition of the word in computer science literature, as seen in the [October 1974 SIGOPS Review ACM](http://dx.doi.org/10.1145/775280.775282)
- concurrent v parallel
    - wordsmithing
    - philosophy goes that parallel is for hardware people, concurrency is for software people
    - it's saying that we want to defer making decisions about actual parallel behavior to runtime, even if it’s just vm startup, so that our code doesn’t make those decisions
    - anything possibly runtime configurable, e.g.; maximum process count, memory usage, number of nodes in a cluster, number of CPUs used in a node, etc., will be at the discretion of the program’s operator and not the programmer
    - very much the mentality that was needed for systems where downtime or unscheduled maintenance visits by a field operator were very very expensive relative to the rest of the project’s costs. the operator should have the power to do what she has to in order to get the system running well
- soft real time
    - used to describe a problem domain composed of some number of common tasks, where the value of handling a task correctly decreases exponentially over time. in other words, a task is assumed to complete very quickly, and unless it does, an error response is preferable to silence
    - contrast to hard real time, and “firm real time” (source: Brian Troutwine Erlang factory youtube), where the value of a request - response pair is a windowed function ; it must be totally done by X time or else there’s no value at all

- -

## glossary, pt. 2

- beam = *Bogdan/Björn's Erlang Abstract Machine*
    - the defacto Erlang virtual machine implementation
- erlang = a programming language with a compiler to bytecode run by the beam <!-- .element: class="fragment" data-fragment-index="1" -->
- elixir = a programming language with a compiler to bytecode run by the beam <!-- .element: class="fragment" data-fragment-index="2" -->
- erts (erlang runtime system) = the erlang distribution. erlang + beam + standard libraries + otp <!-- .element: class="fragment" data-fragment-index="3" -->
- OTP = the portion of the libraries included with distributions specifically intended for creating concurrent, highly available, soft real time systems <!-- .element: class="fragment" data-fragment-index="4" -->

Note:

- the beam
    - Scandinavian programmers are apparently required to name all stuff after themselves; they must all be gits
    - register based
        - I'm lying if I imply I understand what that means
        - makes it more difficult from the perspective of virtual machine engineers and compiler authors, but has a better potential optimization long term (source: Ulf Wiger on Erlang factory youtube)
    - bytecode instructions for tail recursion

- -

## features

- functional, but not purely functional
- [actor concurrency model](https://en.wikipedia.org/wiki/Actor_model) enforced
    - anti-feature: you can’t not use it
- sophisticated pre-emption of concurrent tasks
- shared nothing memory

Note:
- side effects can and should include message passing
- *pause to discuss actor model if needed*
    - single threaded entities can perform calculations, and send messages to other entities
    - actor model differentiated by it's asynchronicity, and lack of "reliability"
    - build your own request-reply paradigm if you need to
    - messages are not guaranteed to arrive
    - multiple messages from a given process to another given process will arrive in the order they were sent
- actor concurrency model enforced. inside beam, message passing isn’t just a good idea, it’s the law (tm)
    - not true for go-lang, akka, ruby + celluloid, enforced actor computation is pretty much the strongest possible stance
    - it’s possible to share state between actors, or have side effects not in the form of message passing, but programmer laziness will not cause that to happen; it’s awkward to set up
- most actions, including function calls, increment a number called the reduction count
    - a process will yield to the scheduler when the reduction count is up
- copy ALL the things between actors
    - efficency? generally speaking, not a big deal, because immutability of data and lots of fine grained processes makes garbage collecting a bajillion times more efficient 
    - “bajillion” is a technical term

- - -

# elixir

- -

## elixir's compiler

- elixir code will become beam bytecode
- relaxed, ruby-like syntax (parentheses optional on function calls)

- -

## elixir's compiler: example

```elixir
defmodule Hello.Demo do
  IO.puts "Hello, World!"
end
Hello, World!
{:module, Hello.Demo,
<<70, 79, 82, 49, 0, 0, 3, 152, 66, 69, 65, 77, 69, 120, 68, 99, 0, 0, 0, 60, 131, 104, 2, 100, 0, 14, 101, 108, 105, 120, 105, 114, 95, 100, 111, 99, 115, 95, 118, 49, 108, 0, 0, 0, 2, 104, 2, ...>>,
:ok}
```

- define an Erlang module
- compile it to bytecode
- return that bytecode back to my interactive shell
- load that bytecode into my beam vm
- run code inside the block
- in doing so send a message "Hello World!" to the default IO sink, which is another Erlang process

Note:

That's an awful lot to have all on one slide, but I do it to make the point that Elixir is hiding a lot of work from the user. Even though the result follows the "Matz-ian" principal of least surprise, users can and will care about details when code hits production

- -

## elixir's tooling

- mix, for projects
    - uses a single project definition file
    - fetch dependencies using git, build 
      documentation, create release packages, generate OTP compilant application 
      boilerplate, and more
    - transitive dependency management
- [hex](https://hex.pm)
    - [rubygems](https://rubygems.org/) / [cheese shop](https://pypi.python.org/pypi) for elixir
- documention: python inspired features are now standard issue
    - docstrings, doctests

Note:
- mix was originally created by Phil Hagelberg, creator of leiningen for Clojure
- mix allows for projects to have a standard for builds, erlang has Emakefiles, erlang.mk, rebar, and others, such as a custom build script (example: RabbitMQ)
- mix: inconsistency in tooling has strong implications when it comes to transitive dependency management (rebar != maven, even though it theoretically could work like that)
- hex directly addresses problems with erlang usability around discovering 3rd party modules
- hex: erlang has… google.
- docs: doctests: executable documentation isn’t just for pythonistas
- docs: documentation can be embedded into the byte code, and seen inside the repl. makes exploring APIs easier, like with ipython
- docs: ex_doc: takes docstrings, makes webpages

- -

## elixir's features

- hygienic macros + hygienic macros + more hygienic macros
- erlang typespecs still work, for dialyzer’s bytecode mode
- right association inside data structures are represented using Elixir's `Stream`
    - takes you down the path toward lazy evaluation

Note:
- dialyzer: the linter you wish you had
- lazy evaluation is something Erlang avoided explicitly as an anti-feature
- lazy evaluation was seen as a barrier to reasoning about realtime properties of production code

- -

## elixir's features, pt. 2

- "protocols" provide polymorphism
    - they're not generics - those are a runtime thing, instead it's templates + metaprogramming
    - if you refer to a protocol by name, the elixir compiler will rewrite your code

- -

## elixir's features, pt. 3

- "sigils" provide an extensible way to do string interpolation

```elixir
# A regular expression that returns true if the text has foo or bar
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Note:
- Scala's [StringContext](http://www.scala-lang.org/api/current/scala/StringContext.html)

- -

*what? isn't that a little too much information???*

## the features elixir brings are for software library authors. <!-- .element: class="fragment" data-fragment-index="1" -->

## after a sufficient number of libraries are published, this is good for everybody <!-- .element: class="fragment" data-fragment-index="2" -->

Note:
- if you haven't adopted Erlang but have been wanting too, most of the major learning curve issues are directly addressed by elixir
    - by elixir's technology
    - and by the stewardship of the project

- - -

## learn us some elixir for great good

#### perilous live demo

- - -

## questions?

