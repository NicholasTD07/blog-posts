---
title: Use Protocol in Swift as Function Parameter - TL;DR
created_at: 2015-06-07 21:15:01
kind: article

tags:
  - Swift
  - Swift 1.2

permalink: "/tldr/2015/06/08/use-protocol-in-swift-as-function-parameter.html"
---

Trying to achieve something easy in dynamic language like Python and Ruby in Swift turned out to be not that easy, though code written in Swift really has a "dynamic" look.

<!-- more -->

Something easy in Python:

```python
class Person:
    def __init__(self, name):
        self.name = name

def changeName(named, newName):
    named.name = newName

nick = Person('Nicholas')
nick.name # => 'Nicolas'

changeName(nick, 'Nicole')
nick.name # => 'Nicole'
```

and Ruby:

```ruby
class Person
  attr_accessor :name
  def initialize(name)
    @name = name
  end
end

def changeName(named, newName)
  named.name = newName
end

nick = Person.new('Nicholas')
nick.name # => "Nicholas"

changeName(nick, 'Nicole')
nick.name # => "Nicole"
```

Not that easy in Swift:

For class only protocols:

* Declare protocol as class only by inheriting the `class` protocol. YES, protocol in Swift have inheritance.
* Use protocol as the type for parameter in the function.

```swift
protocol Named: class {
    var name: String { get set }
}

class Person: Named {
    var name: String

    init(name: String) {
        self.name = name
    }
}

func changeName(named: Named, newName: String) {
    named.name = newName
}

let nick = Person(name: "nick")
changeName(nick, "NICK")
println(nick.name) // "NICK"
```

Even more complicated if you want `changeName` function to work with both class and sturct:

* Declare function as a generic function, where type `T` conforms to `Named` protocol

* The parameter for `T` must be an `inout` parameter for modification to the object itself to work for both class and struct.

    Because struct is passed by **value**.

* All parameter themselves need to be a **var**. NO **let**.


<!-- TODO -->
{% gist NicholasTD07/1ac49c97a8704cc44dba %}
