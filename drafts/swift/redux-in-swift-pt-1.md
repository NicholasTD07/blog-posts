# Redux in Swift Part 1: Introduction and Example within 70 Lines!

In this blog, I will introduce you to the concept of Redux architecture briefly, implement and explain it in Swift bit by bit with minimum changes each time, while satisfying Redux's requirements and making it compilable at the same time.

Although the snippet in this blog is written with Swift 2.2, I will update it to Swift 3 syntax in the next blog.

## Redux 

> Redux architecture revolves around a **strict unidirectional data flow**.

![](https://www.dropbox.com/s/51evjhxw35t59x2/Redux.png?raw=1)

[Redux Doc: Data Flow](http://redux.js.org/docs/basics/DataFlow.html)

### WAT?

In essence, it's an idea which tries to

- seperate/isolate the 'state' of an application to a single location
- limits the flow of the data in an application

### What is Redux?

Redux is composed of five types (of things):

- *State*, the entire internal state of an application, which could be anything
- *Store*, which holds a *State*
- *Action*, which gets dispatched to change the *State* in a *Store*
- *Reducer*, which combines the current *State* and the dispatched action and generates a new *State*
- *Subscriber*, which listens to changes of the *State* in a *Store*

### Why Go with Redux?

> All data in an application follows the same lifecycle pattern, making the logic of your app more predictable and easier to understand.

**More Reading:**

[Redux Doc: Motivation](http://redux.js.org/docs/introduction/Motivation.html)

[The Case for Flux](https://medium.com/swlh/the-case-for-flux-379b7d1982c6#.ddab5kuhi)

## Redux in Swift

I will write the minimium implementation of a Redux architecture bit by bit, to get the snippet to compile while satisfying the requirements, with detailed explanations.

## 70 Lines EXAMPLE!

[Full version here](https://github.com/NicholasTD07/TTTTT/blob/master/swift-experiments/redux-with-generic-class.swift)

As mentioned above, we need five types (of things) to compose/implement Redux, which are:

- State
- Store
- Action
- Reducer
- Subscriber


Let's get the party started!

### Store and its State

A *Store* holds a *State* which could be anything, thus the simplest implementation would be:

```swift
class Store<State> {
    private(set) var state: State! // This `!` will be explained later
}
```

The `state` is marked as a `var` because:

the *State* in a *Store* can be changed by dispatching *Actions* to a *Store*.


The `state` is marked as `private(set)` because:

- the *State* in a *Store* should be read-only and
- "The only way to change the state is to emit an action, an object describing what happened."

[Redux Doc: Three Principles](http://redux.js.org/docs/introduction/ThreePrinciples.html)

### Reducer and Action

A *Store* needs a *Reducer* to generate new *State*s from the current *State* and an Action, which means:

- A *Store* needs to hold onto a *Reducer*
- A *Reducer*
    - takes a *State* and a *Action* and
    - returns a new *State*
- An *Action* describes what happen**ed** while it may carry some data, e.g. `AddTodo(text: "buy milk")` or `ToggleTodo(index: 1)`
    - An *Action* can be just an empty Struct or an Enum as demonstrated in later sections

Essentially, a *Reducer* in Swift can be defined as a function type.


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

#### Things to Note about Reducers

- They should be pure: No side-effects. No API calls. etc.
- They can take an Optional `State` which implies
    - the *State* in a *Store* can be nil at the very beginning before
      anything's dispatched (Don't worry. Next section will explain dispatching
      *Action*s)
    - the default initial state is defined in a *Reducer* rather than a *Store*.
    - This is also partially why we need that `!` when defining the `state` in the `Store`.
- They should return the current *State* if they are given an *Action* they cannot handle.

### Dispatching Actions and Notifying Subscribers

An *Action* can be dispatched to a *Store* to change its *State*. After a *Store*'s *State* is changed, the *Store* should notify all of its *Subscribers*. Thus,

- A *Subscriber* is a function takes a *Store*
- *Subscribers* can subscribe to a *Store* (will be explained in the next section)
- A *Store* needs to hold onto an array of *Subscribers*
- After a *Store*'s *State* changes, the *Store* needs to notify the change to all of its *Subscribers*

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

Now the generic class `Store` is almost usable except a few missing pieces. Example usage:

```swift
struct Increase: ActionType { }

let counterStore = Store<Int>.init { (state: Int?, action: ActionType) -> Int in /* ... */ }

counterStore.dispatch(Increase())
```

If you want to run the snippet above at this point, you will need to replace the `/* ... */` with `return 0`.

#### How does a Store get its initial State?

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

and also add the definition of the `InitialAction` (before and outside the definition of the `Store`),

```swift
struct InitialAction: ActionType { }
```

So when a *Store* initializes, the following would happen regarding its `state`:

- the `Store` will call its `dispatch` with the `InitialAction`
- `dispatch` would update the `Store`'s `state` by calling its `reducer` with `nil` as the `state` param and `InitialAction` 
- the `state` in the `Store` would be set to whatever the `Reducer` returns as the initial state
    - because a `Reducer` returns the actual initial state when it receives a `nil` as its `state` param

#### The BANG`!`

As you can see from the above and actually running the snippet, the force-unwrapped `state` in the `Store` will always be set to non-nil values before it's used as a non nil value. It is not only safe but also convenient to use (no need for `guard let`s everywhere).

### Subscribing to a Store

*Subscriber*s are functions which takes a *Store* as their only param. They can subscribe to *Store*s by calling the `subscribe` method as illustrated below. When a *Subscriber* is subscribed to a *Store*, it will receive the *Store*'s current *State*.

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

#### TESTS!

If you remove all the example snippets, add the following and run the file in either:

- Xcode Playground or,
- command line by running `swift path/to/your-file` or,
- Vim by doing `:w | !swift %` (personal preference)

the test suite should run without any problem which indicates all the assert (tests) passes.

```swift
enum CounterActions: ActionType {
    case Increase
    case Decrease
}

let counterStore = Store<Int>.init { (state: Int?, action: ActionType) -> Int in
    let state = state ?? 0

    guard let counterAction = action as? CounterActions else {
        return state
    }

    switch counterAction {
    case .Increase:
        return state + 1
    case .Decrease:
        return state - 1
    }
}

// initial state is 0
assert(counterStore.state == 0)

// dispatch unknown actions does not affect the state
counterStore.dispatch(InitialAction())
assert(counterStore.state == 0)

// dispatch Increase will increase the state
counterStore.dispatch(CounterActions.Increase)
assert(counterStore.state == 1)

// dispatch Decrease will increase the state
counterStore.dispatch(CounterActions.Decrease)
assert(counterStore.state == 0)

var counter: Int = -1000
counterStore.subscribe { (store: Store<Int>) in
    counter = store.state
}
assert(counter == counterStore.state)

// when state is updated, subscirbers will get notified of the new state
counterStore.dispatch(CounterActions.Increase)
assert(counter == counterStore.state)
```

## What's Next?

In the future blogs, I will

- Update the snippet to Swift 3 syntax (current version is written with Swift 2.2)
- Make the `state` in the `Store` not force-unwrapped, i.e. `var state: State`. No more `!`.
- Create a helper function to remove the need for *Reducer*s to do the type casting
- Show how to split large *Reducer*s into smaller ones and combine them back into one.
- Build an iOS Todo app with Swift and the micro Redux framework we built in this blog post.

Stay tuned!

Also, let me know if you think the current implementation can be improved in anyway.

## Ref

[Getting Started with Redux (30 free videos, 2 minutes each)](https://egghead.io/series/getting-started-with-redux)

[Introduction to Redux (JS)](http://redux.js.org)
