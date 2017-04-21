# Redux in Swift Part 2

In the last blog, [Part 1](https://github.com/NicholasTD07/blog-posts/blob/master/swift/redux-in-swift-pt-1.md),
we walked through the basics of what is Redux and how to build one ourselves in Swift. 

The basic implementation of that Redux is really simple and it's only 30 lines of code.
However, it was written in Swift 2, shown as in
[this gist](https://gist.github.com/NicholasTD07/f7c6f2fb7863ca4ad85c9b9a6dfa1dd3/3aaf134e57329b336dbbaa531a4a117bce3ac616).

In this blog, we will go through the followings:

0. Make it compatible with Swift 3 [diff here](https://github.com/NicholasTD07/TTTTT/commit/d826f7f61afbed13d4c6da301f63e886d40163a8?diff=unified)
0. Helper methods to avoid type-casting in *Reducers*
0. Helper methods to combine *Reducers*
0. Remove the force-unwrap on `Store.state`
0. Make a project for this micro Redux framework and split things properly, so that
  - We can have proper tests! Hooray!
  - etc etc
 
## Swift 3

NOTE: This commit https://github.com/NicholasTD07/TTTTT/commit/d826f7f61afbed13d4c6da301f63e886d40163a8

## Helper Methods for Reducers

https://github.com/NicholasTD07/TDRedux.swift/blob/1.0.0/TDRedux/ReducerHelpers.swift

### Specific Action Type Typed `Reducer`

```swift
func Reducer<State, SpecificActionType>(initalState: State, reducer: (State, SpecificActionType
```

### Combining *Reducers*

```swift
combine(reducers: [Reducer])
```

## Something something

Finally we will have the `Store.swift` looking like
[this](https://github.com/NicholasTD07/TDRedux.swift/blob/1.0.0/TDRedux/Store.swift)
and the helper methods in the [ReducerHelpers.swift], like
[this](https://github.com/NicholasTD07/TDRedux.swift/blob/1.0.0/TDRedux/ReducerHelpers.swift).
