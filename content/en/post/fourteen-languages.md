---
title: "Fourteen languages in one month"
author: "Rafael Viscarra"
cover: "/images/fourteen-languages/cover.png"
tags: ["seven languages in seven weeks", "clojure", "idris", "elixir", "lua", "factor", "julia"]
date: 2020-10-06T00:00:00-03:00
---

I remember the first time I read about the book ["Seven languages in seven weeks"](https://pragprog.com/titles/btlang/seven-languages-in-seven-weeks/), back then I was still a student working on my graduation project but even then I was able to see the value in exploring seven languages in such a short amount of time. For one reason or another I ended up not reading it at the time, I just added it to an ever growing list of "things I'll eventually get back to". Fast-forward ~ 8 years, I found myself in lockdown "thanks" to the COVID-19 pandemic and thus was looking for something fun yet challenging to do. I remembered the book and decided to give it a go, even better, after all this time a sequel ["Seven more languages in seven weeks"](https://pragprog.com/titles/7lang/seven-more-languages-in-seven-weeks/) had come out so I decided I would do both. After going through the first language (Ruby) I realized the book's format was 3 days per "week" instead of 7, as the title implies; I also noticed that I was familiar with a few languages from the first book so I challenged myself to go through both books in seven weeks or less.

## The first seven

The first book came out back in 2010; things were different back then, functional programming was a novelty (in the industry), Ruby on Rails was the hottest thing ever and Clojure had only been invented a couple of years back. At first I assumed this would be an issue, a lot of things can change in 10 years, even more so in the technology world; surprisingly, the 1st book wasn't as outdated as I thought it would be, probably because the languages it explored weren't exactly new.

The book starts by warning the reader about the lack of detailed instructions to install the tools required for each language, which makes a lot of sense. I enjoyed Bruce Tate's examples and analogies, his style of writing made me want to keep reading. Now, onto the languages:

### Ruby

I wouldn't call myself a Ruby expert but I've worked on a few projects that used it, so I know my way around it; the fact that the language reads almost like English, even more so than Python, also helps. The Ruby chapter was a breeze, the 1st and 2nd day's content was just an explanation of Ruby's syntax and its data structures. Sadly, I didn't learn anything new until the 3rd day where it briefly explains how metaprogramming works in Ruby.

### Io

This was the _weirdest_ language in this book, I had never heard of it before and I like to think that I'm a _language nerd_. It's a prototype oriented language, the syntax is simple and with good reason; I remember the book mentions at some point that basically everything but the commas that separate the arguments on a function call is a message, which didn't make a lot of sense until I did the exercises from the 2nd day. One way to describe it is that everything is _lazy_ in Io, a function's argument is not evaluated before the function is called, it's the receiver's responsibility to evaluate it (if it needs to). Other things that I found particularly interesting was being able to modify the operator table and the support for the actor model out of the box. Sadly, it looks like the language hasn't received a lot of attention since 2013; I started playing with the actors by pulling a remote URL but it turned out that TLS support was lacking so I left it there. Even so, the language was _alien_ enough to make me think things twice, it was also a nice refresher on the prototype paradigm.

{{<highlight io>}}
// Here's an interesting fragment of Io code. An HTML builder DSL
HtmlBuilder ul({"class": "language-list"},
  li({"class": "scala"}, "Scala"),
  li("Elixir"),
  li("Io"),
)
{{</highlight>}}


### Prolog

I took a logic programming class in college where I got to use SWI-Prolog so I still remembered the _core_ of the language and the idea behind it: facts, unification, pattern matching, recursion, etc ... the syntax also felt familiar, probably because I've written my fair share of Erlang. Since I understood how unification and pattern matching works this chapter didn't feel like a challenge, it was more like a mind teaser but it made me remember the _woah_ feeling I had when I realized I could _ask_ it to unify any of the variables in an `append` call.

{{<highlight prolog>}}
% append/concat in Prolog
% it's able to unify any of its parameters, meaning it can also work as a starts_with/ends_with
concat([], L, L).
concat([Head|Tail1], L, [Head|Tail2]) :- concat(Tail1, L, Tail2).
{{</highlight>}}

### Scala

Once again, this is a language I'm familiar with, I even have a Lightbend Scala certification (another by-product of the COVID-19 lockdown). I went through the 3 days fairly quickly, as I was already familiar with all the concepts the author was explaining. While working on the 3rd day's exercises I found an instance of the book showing its age; the book suggest an old Scala version and I used the one that I had installed already (2.13.x) which doesn't include the `scala-actors` library anymore, it was deprecated on 2.10. I still wanted to play with it so I implemented the exercise using Akka.

### Erlang

As I mentioned before, I've written and debugged a considerable amount of Erlang intermittently over the last 5 years. This chapter was an easy read, I could appreciate the explanation from the point of view of someone that already knows the language and I think it did well. I worked on the exercises just for completeness sake.

### Clojure

I don't get to write as much Clojure as I want to but I've read books and seen talks about it. I didn't learn a lot from this chapter as I'm already familiar with functional programming, macros, recursion, etc ... but it was a nice excuse to fire up a REPL and write some Clojure. I finished the chapter with the feeling that I want to become more fluent in Clojure.

### Haskell

I've played with Haskell before, I read the [Learn you a Haskell](http://learnyouahaskell.com/)  book a few years ago and played with it just to satisfy my functional programming curiosity. I know that one of the biggest _pain points_ of learning Haskell is grasping monads; the 7 languages book doesn’t do a great job explaining them, in my opinion; the example feels contrived. Also, I find that this [bind](https://fsharpforfunandprofit.com/posts/computation-expressions-bind/) explanation is easier to understand.

{{<highlight haskell>}}
-- Book example to illustrate monads and introduce do notation
data Position t = Position t deriving (Show)

stagger (Position d) = Position $ d + 2
crawl (Position d) = Position $ d + 1

rtn x = x
x >>== f = f x

treasureMap pos = pos >>==
                  stagger >>==
                  stagger >>==
                  crawl >>==
                  rtn
{{</highlight>}}

The first book made me feel like I didn't learn as much as I thought I would, luckily there's still one more book and seven more languages to explore.

## The last seven

The second book was published in November 2014, when I went through the list of languages it explored I felt excited, the only one I had used before was Elixir and I was eager to learn something new and experience those _aha_ moments when a new paradigm finally clicks. Now, onto the last seven languages:

### Lua

I'd been wanting to explore Lua after I found out it's possible to write Redis scripts in Lua, it's also being used as an extension scripting language in [other projects](https://github.com/meetecho/janus-gateway) that I use every now and then. Lua is pretty straight forward, a bit too verbose for my taste but also as explicit as it can be. It was the second prototype-based language that I got to explore during this journey but Lua gave me the impression of being more flexible in that regard. I also liked the toy application that's developed during the 3rd day, a DSL to write MIDI songs; it felt more like a fun toy project than just a mere exercise.

{{<highlight lua>}}
-- Lua sample/exercise from the 3rd day.
-- It's a DSL to write songs and play them using a MIDI synthesizer
song.set_tempo(50)

song.part{
  D3s,  Fs3s, A3s,  D4s,
  A2s,  Cs3s, E3s,  A3s,
  B2s,  D3s,  Fs3s, B3s,
  Fs2s, A2s,  Cs3s, Fs3s,

  G2s,  B2s,  D3s,  G3s,
  D2s,  Fs2s, A2s,  D3s,
  G2s,  B2s,  D3s,  G3s,
  A2s,  Cs3s, E3s,  A3s,
}

song.part{
  Fs4ed,           Fs5s,
  Fs5s, G5s, Fs5s, E5s,
  D5ed,            D5s,
  D5s,  E5s, D5s,  Cs5s,

  B4q,
  D5q,
  D5s,  C5s, B4s,  C5s,
  A4q
}
{{</highlight>}}

### Factor

Factor is probably the language that meant the biggest paradigm shift, I knew of stack based programming conceptually but I'd never worked with it; "concatenative programming" was completely new to me, but also fascinating. I spent a couple of hours debugging the hardest exercise of the 2nd day, even after I wrapped my head around it I couldn't understand how more complex pieces of code could be written in such a language, although the book doesn't mention it, Factor supports lexical variables.

One exercise of the 3rd day invites the reader to explore the toy applications that are included with the Factor package; reading some applications made me realise how capable the language can be.

{{<highlight factor>}}
USING: kernel sequences io ;
IN: examples.sequences

! This is my take on find-first
! It's a function that returns the first element in the sequence for which the predicate returns true
: find-first ( seq pred -- el )
  [ 2dup drop empty? [ f ] [ 2dup [ first ] dip call( x -- y ) not ] if ] [
    [ rest ] dip
  ] while
  drop
  dup empty? [ drop f ] [ first ] if ;
{{</highlight>}}

### Elm

This chapter was particularly interesting and challenging, not because of the Elm language but because everything that the books says, beyond syntax and basic concepts, is outdated; it turns out that Elm moved away from reactive programming a few years ago. This meant I had to do my _homework_ in order to understand what was going on; I really liked how the language _felt_ like Haskell with F#/OCaml semantics sprinkled around so I decided to do some reading in order to implement the examples and exercises from the book.

After struggling to get an example running on the lastest Elm (0.19.1) I came across a few articles that complained about the 0.19 release; apparently Elm changed significantly from 0.18 to 0.19. One of the most curious changes was that only the core Elm team is allowed to publish packages that contain JavaScript code; the restriction is actually [baked into the Elm](https://github.com/elm/compiler/blob/d07679322ef5d71de1bd2b987ddc660a85599b87/compiler/src/Parse/Parse.hs#L29) compiler. I thought this was pretty weird for an open-source project; this, combined with the volatility of the project makes it a poor choice for a new long-term project, IMO.

{{<highlight elm>}}
-- Code that shows a counter that increases by one every second
-- Imports omitted for brevity
main =
  Browser.element
    { init = init
    , subscriptions = subscriptions
    , view = view
    , update = update
    }

init: () -> (Int, Cmd ())
init () =
  (0, Cmd.none)

subscriptions _ = 
  Time.every 1000 <| always ()

view: Int -> Html ()
view count =
  text <| String.fromInt count

update: () -> Int -> (Int, Cmd ())
update msg count = (count + 1, Cmd.none)
{{</highlight>}}


### Elixir

I'm back into my comfort zone, the Elixir _week_ went smoothly; I've used the language professionally before so I didn't learn a lot in this one. The 3rd day introduces macros and I got to play with them a little bit, something that I don't usually get to do while writing application code.

{{<highlight elixir>}}
# One of the exercises/examples from the book
# StateMachine uses macros in order to enable simple DSL
#  to defined transitions in a state machine
defmodule VidStore do

  use StateMachine

  state :available,
    rent:   [ to: :rented,    calls: [ &VidStore.renting/1 ] ]

  state :rented,
    return: [ to: :available, calls: [ &VidStore.returning/1 ] ],
    lose:   [ to: :lost,      calls: [ &VidStore.losing/1 ] ]

  state :lost,
    find:   [ to: :rented,    calls: [ &VidStore.finding/1 ] ]

  # Callbacks omitted for brevity

end
{{</highlight>}}

### Julia

When I started the Julia chapter I was excited and afraid; I imagined Julia like a R or Matlab with syntactic sugar thanks to its scientific background. Thankfully I couldn't be more wrong, this language has it all: clean syntax, macros, union types, good performance, message passing for process synchronization, simple multithreading primitives and multiple dispatch. Needless to say, I really enjoyed working in Julia and would jump at the opportunity to work with it on a real project!

{{<highlight julia>}}
# These 15 lines of code:
# - Creates 4 processes
# - Defines the flip_coins function in all of them
# - Runs a parallel simulation and then aggregates the results
# Concise and powerful in my opinion
addprocs(4)

@everywhere function flip_coins(n :: Int64)
  ct = 0
  for i = 1:n
    ct += convert(Int64, rand(Bool))
  end
  ct
end

function produce_heads(trials :: Array{Int64, 1})
  heads = pmap((l :: Int64 -> flip_coins(l)), trials, distributed=true)
  for (t, h) in zip(trials, heads)
    println("On $t trails we got $h heads")
  end
end
{{</highlight>}}

### miniKanren

I didn't know what to expect from the miniKanren chapter since it started by saying that it's an implementation of a logic language embedded in Clojure. It was nice to get to work in Clojure again but I didn't feel like I learned a lot from miniKanren. The only advantages I found vs a modern Prolog were the improved (Clojure) syntax and the benefit of being embedded in Clojure.

### Idris

I knew since the beginning that Idris was going to be a tough one. The prologue mentions that the language was challenging for the authors, also, I had read a few articles on dependent types before and I can't say it was a concept that I grasped completely.

The 1st day covers the basic syntax which heavily resembles Haskell's so that was easier than expected. One of the exercises instructed the reader to look for other languages that used dependent types, I found the [F*](https://www.fstar-lang.org/) language which looks like F# with dependent types; this is now the latest addition to my _I'll get back to it some other time_ list.
The 2nd day explains dependent types, their benefits and how they work in Idris; one of the 2nd day's exercises is transposing a 2D Matrix which was defined as a vector of vectors; I thought of implementing it by using a helper [mapi](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-arraymodule.html#mapi) function `(Fin n -> a -> b) -> Vect n a -> Vect n b` that I also needed to code. After a couple of hours of the compiler refusing to take my code I decided to jump into the Idris standard library to see if I was missing something; this made me realize the extent of the Idris language.
The 3rd day explained how to translate Idris concepts into _industry_ languages (C++), upon reading this I felt a bit disappointed as I thought it was going to go deeper into the dependent type rabbit hole I had glanced at the day before.

I enjoyed learning about the Idris language in general, I'm even entertaining the idea of looking into it a [little bit more](https://www.manning.com/books/type-driven-development-with-idris).

{{<highlight idris>}}
-- This is a fragment from the Idris Vect module I found interesting.
-- The replicate call can infer its 1st param thanks to dependent types
transpose' : {n : Nat} -> (xss : Vect m (Vect n x)) -> Vect n (Vect m x)
transpose' []          = replicate _ []
transpose' (xs :: xss) = zipWith (::) xs (transpose' xss)
{{</highlight>}}

## 14 languages later

Even though only half of the languages (Io, Julia, Factor, Lua, Idris, miniKanren and Elm) were novel to me, I felt like I learned a lot more. Getting to work in languages I hadn’t used for a few years certainly put things into perspective. These are some of the concrete learnings/ideas I got from all of this:

* I want to write more Clojure, this might be the choice for my next side project.
* The next time I need something to crunch some numbers while exposing an API to do so, Julia will be the first on my list.
* Dependent types is an idea that's worth exploring; F* maybe a good candidate for this since I enjoy working in F# but Idris seems to be a popular option as well.
* My aversion towards JavaScript has pushed me to find more elegant languages that transpile to it, Elm seemed like a good candidate but the walled garden situation would make me think twice before choosing it.
* I'm really enjoying the "Seven X in seven weeks" series, I might do the databases one next time.

