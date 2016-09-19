# Redux in Swift Part 1: Introduction and Example within 70 lines!

## TODO

- Explain the example in sections
- Ask Swifty people for proof-reading

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



## 70 Lines EXAMPLE!

[GitHub Link](https://github.com/NicholasTD07/TTTTT/blob/master/swift-experiments/redux-with-generic-class.swift)

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
