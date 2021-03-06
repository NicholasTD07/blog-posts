# Swifty Bits / Q&A

**All bits/snippets in this blog compiles with Swift 3 (v3.0.1) compiler.**

## Howto use Iterator?

```swift
let array = Array(1...5)

var iterator = array.makeIterator()

print(iterator.next()) // Optional(1)
print(iterator.next()) // Optional(2)
print(iterator.next()) // Optional(3)
print(iterator.next()) // Optional(4)
print(iterator.next()) // Optional(5)
print(iterator.next()) // nil
```

https://github.com/NicholasTD07/TTTTT/blob/master/swift-3-experiments/iterator.swift

## enums are implicitly Hashable!

WAT?!

```swift
enum T {
    case t1
    case t2
}

func takeHashable<T: Hashable>(_ t: T) { }

takeHashable(T.t1)
```

https://github.com/NicholasTD07/TTTTT/blob/master/swift-3-experiments/implicit-protocol-requirement-resulting-weird-complier-error.swift

## Can I use protocol as the type of a var in another protocol?

YES!

You just need a bit help from generics.

```swift
protocol A {

}

protocol B {

}

protocol C {
    associatedtype A
    associatedtype B
    var a: A { get }
    var b: B { get }
}

func processC<T: C>(c: T) where T.A: A, T.B: B {
}
```

https://github.com/NicholasTD07/TTTTT/blob/master/swift-3-experiments/member-as-protocol.swift

## Can I modify the tuples in an array?

YES!

```swift
typealias T = (name: String, age: Int)

var ts = [T]()

ts.append((name: "Nick", age: 25)) // always 25
ts[0].age = 0

print(ts[0]) // `0`, reborn!
```

https://github.com/NicholasTD07/TTTTT/blob/master/swift-experiments/change-tuple-value-in-an-array.swift

## Non-optional is the same as itself and also the same as `.some(T)`

HOW?!

```swift
enum T {
    case t1
    case t2
}

let t: T? = .t1

assert(t == .t1)
assert(t == .some(.t1))
```

https://github.com/NicholasTD07/TTTTT/blob/master/swift-2-experiments/assert-optional-enum.swift
