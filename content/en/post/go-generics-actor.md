---
title: "Actor Model with Go 1.18 Generics"
author: "Rafael"
cover: "/images/go-generics-actor/cover.png"
tags: ["golang", "actor", "generics"]
date: 2022-02-07T00:00:00-03:00
---

Go 1.18 is scheduled to release in February 2022, this release is a big one
because it introduces the highly anticipated (and controversial) Type Parameter 
feature aka generics; it's probably going to take some time until we actually 
migrate to 1.18 at work, so I've been looking for excuses to play with it.

A few weeks ago I worked on a feature that could have been implemented 
succinctly using the actor model, after thinking it through, I realized generics 
would make it possible to have actors without relying on `interface{}` too much 
so I decided to play with how the API could look like.

<!--more-->

{{<figure src="/images/go-generics-actor/cover.png" alt="Gopher actors" class="no-box-shadow">}}

#### Generics in Go

Instead of jumping straight into the code, I thought getting a refresher on Go's
flavor of generics and its limitations would be a good idea. I gave the 
[Type Parameter Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md) 
a quick read, or at least the parts that were relevant for my endeavors.

I'll assume the reader is familiar with generics in other languages, we'll go 
over the bits that are relevant to Go:

Functions and type definitions can have type parameters, using the following 
syntax:

{{<highlight go>}}


func SendAndReceive[M, R any](msg M) R {
  // do something with the M and R types
}

type Set[T any] struct {
  /* field(s) that refer to type T */
}

{{</highlight>}}

However, receiver functions/methods **cannot** have type parameters, you can 
read more about the reasoning 
[here](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md#No-parameterized-methods).

{{<highlight go>}}

type Actor[St any] struct {}

// This is NOT possible
func (a *Actor[St]) SendAndReceive[R any](msg S) R {
  // code here
}

{{</highlight>}}

Type parameters can have constraints, the "any" constraint allows any type to 
be passed; a type argument satisfies a constraint if it implements the 
constraint (this is an oversimplification, for more details see: 
[Type sets of constraints](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md#type-sets-of-constraints)).

{{<highlight go>}}

// Only values that implement the `Stringer` interface can be passed 
// to this function

func GenericToString[T Stringer](val T) string {
  return val.String()
}

{{</highlight>}}

That's basically all we need to know to write a simple actor abstraction in Go!

#### Reducer based actor

At a very high level, an actor is a component we can communicate with by sending 
messages. Let's define an interface that conveys this:

{{<highlight go>}}

type Actor[Msg any] interface {
  Send(msg Msg)
}

{{</highlight>}}

Actors usually hold inner state and consume one message at a time, the inner 
state is modified/queried by reacting to the messages it receives.

Our implementation will need an extra type argument for its state, we'll also 
include a channel to send and receive messages; now we need a way to mutate 
the inner state as we handle messages, to accomplish this we'll borrow some 
ideas from F#'s `MailboxProcessor` and receive a `reducer` function that will 
return a new state given the current one and the received message. This is how 
the first iteration of our actor would look like:

{{<highlight go>}}

package actor

type reducerActor[Msg, St any] struct {
	state   St
	queue   chan Msg
	reducer func(Msg, St) St
}

func NewFromReducer[Msg, St any](initial St, reducer func(Msg, St) St) Actor[Msg] {
	ga := &reducerActor[Msg, St]{
		state:   initial,
		queue:   make(chan Msg, 1),
		reducer: reducer,
	}
	ga.start()
	return ga
}

func (a *reducerActor[_, _]) start() {
	go a.receiveLoop()
}

func (a *reducerActor[_, _]) receiveLoop() {
	for msg := range a.queue {
		a.state = a.reducer(msg, a.state)
	}
}

func (a *reducerActor[Msg, _]) Send(msg Msg) {
	a.queue <- msg
}

{{</highlight>}}

This actor implementation has its shortcomings, they will become obvious when we
actually use it, even for something simple, like adding numbers:

{{<highlight go>}}

type radderAdd struct {
	value int
}

type radderGet struct {
	reply chan int
}

func TestReducerActor(t *testing.T) {

	adder := actor.NewFromReducer(0, func(raw interface{}, sum int) int {
		switch msg := raw.(type) {
		case radderAdd:
			return msg.value + sum
		case radderGet:
			msg.reply <- sum
			return sum
		default:
			panic(fmt.Errorf("unsupported message %T", raw))
		}
	})
	adder.Send(radderAdd{value: 1})
	adder.Send(radderAdd{value: 2})
	adder.Send(radderAdd{value: 3})
	get := radderGet{reply: make(chan int, 1)}
	adder.Send(get)

	actual := <-get.reply
	assert.Equal(t, 6, actual)
}

{{</highlight>}}

As you can see, there's one big, ironic problem with this implementation: Go 
doesn't have pattern matching and the closest alternative is a type switch, 
which forces us to receive `interface{}` messages; exactly the kind of problem 
generics (and interfaces) should help us solve.

#### Second iteration, typed actor

There are plenty of reasons why using `interface{}` in our API is not ideal, 
especially if we can do better. We'll take advantage of Go's interfaces, 
instead of relying on the message's types to mutate the state, let's allow the 
messages to determine how the state should change!

Since the flow is pretty much the same, we can define our new actor abstraction 
using the previous one.

{{<highlight go>}}

// Message needs to be implemented by actor messages
// Apply should make use of the message's fields and the current state to
//  produce a new one
type Message[St any] interface {
	Apply(St) St
}

func NewTyped[St any](initial St) Actor[Message[St]] {
	return NewFromReducer(initial, func(msg Message[St], state St) St {
		return msg.Apply(state)
	})
}

{{</highlight>}}

I don't like how the message's type needs to _know_ about the actor's inner 
state type, but we'll have to live with it for now. 

One advantage this abstraction provides over the first one is that it allows us 
to create and share `Message`s for common actor patterns, like querying (a 
projection of) the actor's state:

{{<highlight go>}}

type getProj[St, Ret any] struct {
	reply      chan Ret
	projection func(St) Ret
}

func (a *getProj[St, _]) Apply(state St) St {
	a.reply <- a.projection(state)
	return state
}

// GetAsync applies the projection func to the actor's state and returns the
// result asynchronously
func GetAsync[St, Ret any](a Actor[Message[St]], projection func(St) Ret) <-chan Ret {
	reply := make(chan Ret, 1)
	gp := &getProj[St, Ret]{reply: reply, projection: projection}
	a.Send(gp)
	return reply
}

// Get applies the projection func to the actor's state and returns the result
func Get[St, Ret any](a Actor[Message[St]], projection func(St) Ret) Ret {
	reply := GetAsync(a, projection)
	return <-reply
}

{{</highlight>}}

This is how we would model the previous _adder_ actor using the new API and 
abstractions:

{{<highlight go>}}

type adderAdd struct {
	value int
}

func (msg *adderAdd) Apply(st int) int {
	return st + msg.value
}

func add(value int) actor.Message[int] {
	return &adderAdd{value: value}
}

func identity[T any](val T) T { return val }

func TestTypedActor(t *testing.T) {

	adder := actor.NewTyped(0)
	adder.Send(add(1))
	adder.Send(add(2))
	adder.Send(add(3))

	result := actor.Get(adder, identity[int])
	require.Equal(t, 6, result)
}

func TestAddMessage(t *testing.T) {
	require.Equal(t, 0, add(0).Apply(0))
	require.Equal(t, 11, add(1).Apply(10))
}

{{</highlight>}}

Using the new API feels cleaner and allows us to unit test our individual 
messages easily. 

{{<alert title="State mutation" class="warning">}}
Go is not a language that favors immutability, both implementations make it 
trivial to modify or reference an actor's inner state, which introduces a whole 
class of errors that actors solve in other languages.
{{</alert>}}

### Conclusion

The lack of features found commonly in functional languages make it specially
challenging to design an ergonomic actor API in Go. The absence of immutability 
like we have in most functional languages (Elixir, Erlang, F#, Scala, etc ...) 
forces us to think about (not) _leaking_ a reference (pointers, maps, slices)
that would allow the calling code to modify the actor's inner state.

This exercise gave me a good idea of how we can take advantage of Go generics in 
the future, they will be a valuable tool while designing APIs and libraries, 
however, like everything else, they should be used in moderation.

You can find the full source code [here](https://github.com/rviscarra/go-actor).

#### [Bonus] Go generics support in VS Code

VS Code's Go extension doesn't support generic code out-of-the box, but worry 
not, setting it up takes just a few commands and one config change.

In order to make this work, we need to compile the Go language server (`gopls`) 
with Go 1.18 (beta1 or beta2). You can find the instructions to install the Go 
1.18 beta in the 
[Getting started with generics tutorial](https://go.dev/doc/tutorial/generics#installing_beta). 
I'm using Go 1.18beta2

If everything went well, `go version` should output something like 
`go version go1.18beta2 linux/amd64`. 
Now we can install the latest `gopls` with 
`go install golang.org/x/tools/gopls@latest`; please note this will be installed 
under `$GOPATH/bin`, if you haven't explicitly set `GOPATH` it will live in 
`$HOME/go`.

Now we need the full path to the `gopls` binary, this command should give it to 
us: `echo $(go env GOPATH)/bin/gopls`; we need to make note of it, since we'll 
need it next.

Open VS Code's `settings.json` (Ctrl+Shift+P, select "Preferences: Open Settings 
(JSON)") and modify it so it uses our new `gopls` binary, by adding this 
fragment:

{{<highlight json>}}
{
  "go.alternateTools": {
    "gopls": "$NEW_GOPLS_PATH"
  },
}
{{</highlight>}}

Restart/reload VS Code so the new config. change takes effect, now it should 
recognize generic code and shouldn't report syntax errors anymore.

{{<alert title="Almost there">}}
Unfortunately, at time of writing, staticcheck is not supported on generic 
code, so you may still get some warnings.
{{</alert>}}

For more info you can refer to the "Working with generic code" section of 
[this page](https://github.com/golang/tools/blob/master/gopls/doc/advanced.md#working-with-generic-code)