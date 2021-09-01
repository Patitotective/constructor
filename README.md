# Constructor
A collection of useful macros, mostly related to the construction of objects.


Simply use Nimble to install, then
## Construct
`construct` generates constructors so you can quickly write constructors without having to write extremely redundant code.
```nim
import constructor/construct
type
    Awbject = object
        awesome : float
        beautiful : string
        coolInt : int
    Bwbject = ref object
        a : int
        b : string

Awbject.construct(false): # false means it is not exported
    awesome = 1.5
    coolInt = 10 # Uses = for default values

Awbject.construct(true): # true means it is exported.
    beautiful = "This is indeed" 
    coolInt: 10 #Uses : to indicate it's optional.
    awesome: required # Uses required to indicate it's an required parameter.
    _: # Code called after the creation of the object.
      echo "Created a new Awbject"

Bwbject.construct(false):
    (a, b): required # Uses tuple semantics for multiple variables.

Bwbject.construct(false) # All fields are required!.

assert initAwbject() == Awbject(awesome : 1.5, coolInt : 10)
assert initAwbject(1.1) == Awbject(beautiful: "This is indeed", awesome: 1.1, coolInt: 10)
assert newBwbject(10, "This is a ref so uses new")[] == Bwbject(a: 10, b: "This is a ref so uses new")[]
```

## Constructor
`constructor` works similarly to `construct` but does it with your own procedures so you can match your preferred method.
You can pass other parameters to the object constructor by having a variable named the same in the main scope of the procedure.
Aside from that it's practically like writting your own init procedure.
```nim
import constructor/constructor
type
  User = object
    name: string
    age: int
    lastOnline: float

proc initUser*(name: string, age: int): User {.constr.} =
  let lastOnline = 30f

proc init(T: typedesc[User], name: string, age: int) {.constr.} =
  let lastOnline = 30f # can provide defaults/options in the main scope

assert initUser("hello", 10) == User(name: "hello", lastOnline: 30f, age: 10)
assert User.init("hello", 30) == User(name: "hello", lastOnline: 30f, age: 30)
```


## Typedef
`typeDef` macro which can generate objects with properties.
Below is the syntax.
```nim
import ../src/constructor

import constructor

typeDef(*Test): # Notice `*` used for exporting
  *(a, b) = int # Uses tuple semantics for multiple vars
  c = string
  d = seq[int]:
    *get: # `result` holds the returned value
      return result
    *set: # `value` holds the value before setting
      if value.len >= 2:
        value = value[0..2]

var a = Test()
a.d = @[100, 200, 300, 400]
assert a.d == @[100 ,200, 300] # Means the Setter did the job
```
## Events
`event` macro which generates an event, and coresponding procs to interact with it

```nim
event(TestEvent, int)

var testEvent = TestEvent()

proc countTo(a: int)= 
    for x in 0..a:
        echo x

testEvent.add(countTo)

testEvent.invoke(10)
```


## Defaults
`defaults` macro which allows you to easily generate a constructor with default values.

```nim
type Thingy{.defaults.} = object
  a: float = 10 # Can do it this way
  b = "Hmm" # Can also do it this way
  c = 10
implDefaults(Thingy) # Required to embed the procedure
assert initThingy() == Thingy(a: 10, b: "Hmm", c: 10)
```
