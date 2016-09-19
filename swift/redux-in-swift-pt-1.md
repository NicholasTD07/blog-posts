# Redux in Swift Part 1: Introduction and Example within 70 lines!



## TODO

- Explain the example in sections
- Ask Swifty people for proof-reading


![](https://www.dropbox.com/s/51evjhxw35t59x2/Redux.png?raw=1)


## Redux 

> Redux architecture revolves around a **strict unidirectional data flow**.

[Redux Doc: Data Flow](http://redux.js.org/docs/basics/DataFlow.html)

### WAT?

In essense, it's an idea which trys to

- seperate/isolate where changes can happen to the "State" of an app/program
- limits the flow of the data in an application

### What composes Redux?

Five types (of things) composes Redux:

- *State*, which could be anything
- *Store*, which holds a *State*
- *Action*, which gets dispatched to change the *State* in a *Store*
- *Reducer*, which combines the current *State* and the dispatched action and generates a new *State*
- *Subscriber*, which listens to changes of the *State* in a *Store*

### Why go with Redux?

> All data in an application follows the same lifecycle pattern, making the logic of your app more predictable and easier to understand.

**More Reading:**

[Redux Doc: Motivation](http://redux.js.org/docs/introduction/Motivation.html)
[The Case for Flux](https://medium.com/swlh/the-case-for-flux-379b7d1982c6#.ddab5kuhi)

## Redux in Swift



## 70 Lines EXAMPLE! Bit by Bit!

[Full version here](https://github.com/NicholasTD07/TTTTT/blob/master/swift-experiments/redux-with-generic-class.swift)

As mentioned above, we need five types (of things) to compose/implement Redux, which are:

- State
- Store
- Action
- Reducer
- Subscriber

I will write the minimium implementation to get the snippet to compile while satisfying the requirements of what makes Redux.

Let's get the party started!

### Store and its State

A *Store* holds a *State* which could be anything, thus the simplest implementation would be:

```swift
class Store<State> {
    var state: State! // This `!` will be explained
}
```

### Reducer and Action

A *Store* need a *Reducer* to generate new *State* from current *State* and an Action, which means:

- A *Store* need to hold onto a *Reducer*
- A *Reducer*
    - takes a *State* and a *Action* and
    - returns a new *State*
- An *Action* is only needed to differentiate from each other

Essentially, *Reducer* in Swift can be defined as a type.


```swift
protocol ActionType { }

class Store<State> {
    // ... previous code ...
    typealias Reducer = (state: State?, action: ActionType) -> State
    final let reducer: Reducer
    
    init(with reducer: Reducer) {
        self.reducer = reducer
    }
}
```

#### Why `ActionType` cannot be an `Enum`?

Because you cannot add `case`s outside of `Enum` , otherwise, if you do `extend SomeEnum { case X }` you would get this compiler error: `enum 'case' is not allowed outside of an enum`.

#### Things to Note about Reducer Type

- It takes `nil` as `State` which implies the default initial state is defined in a *Reducer* rather than *Store*
    - This is partially why we need that `!` when defining the `state` in the `Store`
- It should return the current *State* if it is given an *Action* it cannot handle

### Dispatching Actions and Notifying Subscribers

An *Action* can be dispatched to a *Store* to change its *State*. After a *Store*'s *State* is changed, the *Store* should notify all of its *Subscribers*.

```swift
class Store<State> {
    // ... previous code ...
    typealias Subscriber = (store: Store) -> ()
    final var subscribers = [Subscriber]()

    final func dispatch(action: ActionType) {
        self.state = reducer(state: state, action: action)
        subscribers.forEach {
            $0(store: self)
        }
    }
}
```

Now the generic class `Store` is almost usable except a few pieces missing. Example usage:

```swift
struct Increase: ActionType { }

let counterStore = Store<Int>.init { (state: Int?, action: ActionType) -> Int in /* ... */ }

counterStore.dispatch(Increase())
```

If you want to run above snippet with minimum change, replace the `/* ... */` with `return 0`.

#### How does a Store Get its Initial State?

We need to update the `init` method in `Store` to the following:

```swift
class Store<State> {
    // ... previous code ...
    init(with reducer: Reducer) {
        self.reducer = reducer

        dispatch(InitialAction())
    }
}
```

and also add the definition of the `InitialAction` (outside the definition of the `Store`),

```swift
struct InitialAction: ActionType { }
```

So when a *Store* initializes, the following would happen:

- the `Store` will call its `dispatch` with the `InitialAction`
- `dispatch` would update the `Store`'s `state` by calling its `reducer` with `nil` as the `state` param and `InitialAction` 
- the `state` in the `Store` would be set to whatever the `Reducer` returns as the initial state
    - because a `Reducer` returns the actual initial state when it receives a `nil` as its `state` param

#### The BANG`!`

As you can see from the above and actually running the snippet, the forced unwrapped `state` in the `Store` will always be set to non nil values before it's used as a non nil value. It is not only safe but also convenient to use (no need for `guard let`s everywhere).

### Subscribing to a Store

*Subscriber*s are functions which takes a *Store* as its only param and they can subscribe to *Store*s by calling the following method.

```swift
class Store<State> {
    // ... previous code ...
    final func subscribe(with subscriber: Subscriber) {
        subscribers.append(subscriber)
        subscriber(store: self)
    }
}
```

With the `Store.subscribe` defined and the previous example snippet, you can do things like this: 

```swift
var counter: Int = -1000
counterStore.subscribe { (store: Store<Int>) in
    counter = store.state
}
```

#### 

### Full file template

TODO: REMOVE ME!

```swift
protocol ActionType { }
struct InitialAction: ActionType { }

class Store<State> {
    typealias Action = ActionType
    typealias Reducer = (state: State?, action: Action) -> State
    typealias Subscriber = (store: Store) -> ()

    final let reducer: Reducer
    final var state: State!
    final var subscribers = [Subscriber]()

    final func dispatch(action: Action) {
        self.state = reducer(state: state, action: action)
        subscribers.forEach {
            $0(store: self)
        }
    }

    final func subscribe(with subscriber: Subscriber) {
        subscribers.append(subscriber)
        subscriber(store: self)
    }

    init(with reducer: Reducer) {
        self.reducer = reducer

        dispatch(InitialAction())
    }
}

struct Increase: ActionType { }
struct Decrease: ActionType { }

let counterStore = Store<Int>.init { (state: Int?, action: ActionType) -> Int in
    let state = state ?? 0

    switch action {
    case _ as Increase:
        return state + 1
    case _ as Decrease:
        return state - 1
    default:
        return state
    }
}

// initial state is 0
assert(counterStore.state == 0)

// dispatch unknown actions does not affect the state
counterStore.dispatch(InitialAction())
assert(counterStore.state == 0)

// dispatch Increase will increase the state
counterStore.dispatch(Increase())
assert(counterStore.state == 1)

// dispatch Decrease will increase the state
counterStore.dispatch(Decrease())
assert(counterStore.state == 0)

var counter: Int = -1000
counterStore.subscribe { (store: Store<Int>) in
    counter = store.state
}
assert(counter == counterStore.state)
```

## Ref

[Getting Started with Redux (30 free videos, 2 minutes each)](https://egghead.io/series/getting-started-with-redux)

[Introduction to Redux (JS)](http://redux.js.org)
