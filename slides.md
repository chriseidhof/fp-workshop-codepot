slidenumbers:true
autoscale:true

# Functional Swift

### @chriseidhof
### Please make sure you have Xcode 7 beta 6 installed.

---

![fit](fpinswift-hero.jpg)

---

### Intro

- What are we going to do?
- How will we do it?
- What are we not going to do?

---

> What is functional programming?

---

# Optionals

---

Javascript:

```javascript
var person = {
    "name": "Chris", 
    "address": {
        "street":"Kotbusser Damm"
    }
};

if (person != null && person["address"] != null) {
    var street = person["address"]["street"];
    if (street != null) {
        console.log(street);
    }
    else {
        console.log("Street unknown");
    }
}    
```

---

Swift:

```swift
let person = [
    "name": "Chris",
    "address": [
        "street": "Kotbusser Damm"
    ]
]
```

---


Another problem:

```objectivec
- (NSAttributedString *)attributedCapital:(NSString *)country {
  NSString *capital = self.capitals[country];
  return [[NSAttributedString alloc] initWithString:capital
    attributes:attributes];
}
```

---

One more example:

```objectivec
NSString *someString = ...;
if ([someString rangeOfString:@"swift"].location != NSNotFound]) {
  NSLog(@"Someone mentioned swift!"); }
```

---

Java:

```java
int i = Integer.getInteger("123")
```

---

### Introducing Optionals

```swift
enum Optional<T> { 
  ...
}
```

Now we can write:

```swift
Optional<Int>
Optional<String>
```

---

```swift
var optionalNumber: Optional<Int> = 3
optionalNumber = nil
```

---

```swift
var optionalNumber: Int? = 3
optionalNumber = nil
```

---

```swift
var optionalString: String? = nil
optionalString = "Hello"
```

---

### Working with optionals (the bad way)

```swift
let string = optionalString! // Now string is not optional anymore
```


---

### Working with optionals

```swift
if let string = optionalString {
    print("The string is: \(string)")
} else {
    print("There was no string")
}
```

---

### Exercises

```swift
let cities = ["Paris": 2243, "Madrid": 3216, "Amsterdam": 881, "Berlin": 3397]
let capitals = ["France": "Paris", "Spain": "Madrid", 
               "The Netherlands": "Amsterdam", "Sweden": "Stockholm"]
```

    git clone https://github.com/chriseidhof/fp-workshop-codepot

- Write `populationOfCapital`
- *Bonus*: write it using a single return
- *Bonus 2*: write it using `flatMap`

---

### Solution

```swift
func populationOfCapital(countryName: String) -> Int? {
  if let capital = capitals[countryName] {
      return cities[capital]
  }
  return nil
}
```

---

# Map and filter

---

### Map

```swift
let numbers = Array(1..<10)

var result: [Int] = []
for number in numbers {
   result.append(number*number)
}
```

---

```swift
result = numbers.map({ x in x*x })
```

---

### Trailing closures

```swift
numbers.map { x in x*x }
```

---

### Filter

---

```swift
var result: [Int] = []

for number in numbers {
    if number % 2 == 0  { 
        result.append(number) 
    }
}
```

---

```swift
let result = numbers.filter { n in n % 2 == 0 }
```

---

```swift
let result = numbers.filter { $0 % 2 == 0 }
```

---

### Exercises

- square an array using `map`
- find all squares below 100 using `filter`
- write `flatten`
- define your own `map` and `filter`

---

# Reduce

---

```swift
func sum(arr: [Int]) -> Int {
    var result = 0
    for i in arr {
        result += i
    }
    return result
}

sum(Array(1..<10)) // 45
```

---

```swift
func product(arr: [Int]) -> Int {
    var result = 1
    for i in arr {
        result *= i
    }
    return result
}

product(Array(1..<10)) // 362880
```

---


```swift
func reduce(arr: [Int], initialValue: Int, 
            combine: (Int, Int) -> Int) -> Int {
    var result = initialValue
    for i in arr {
        result = combine(result, i)
    }
    return result
}

sum = { arr in reduce(arr, initialValue: 0, combine: +) }
product = { arr in reduce(arr, initialValue: 1, combine: *) }
```
---

### Exercises

- make reduce generic
- reduce a list of strings into a string separated by newlines
- write flatten with reduce
- *Bonus 1*: define map and filter in terms of reduce
- *Bonus 2*: redefine reduce recursively using `decompose`:
  http://www.objc.io/snippets/1.html

---

# Type-driven development

---

### CoreImage

```swift
let blurFilter = CIFilter(name: "CIGaussianBlur")!
blurFilter.setDefaults()
blurFilter.setValue(image, forKey: kCIInputImageKey)
blurFilter.setValue(5.0, forKey: kCIInputRadiusKey)
let blurredImage2 = blurFilter.outputImage!
```

---

### Exercises!

- Run the Xcode project
- Change the last line to show the blurred image, and the composited image
- Extract all the blur code into a top-level function
- Extract the compositioning into a top-level function

---

### Filter Composition

Can we write a function `compose` that composes two filters?

---

```swift
typealias Filter = CIImage -> CIImage

func blur(radius: Double) -> Filter

func colorOverlay(color: NSColor) -> Filter
```

---

```swift
func composeFilters(filter1: Filter, filter2: Filter) -> Filter {
    return { image in filter1(filter2(image)) }
}
```

---

### Exercise

* Modify `blur` and `colorOverlay` to have the types as described above
* Use the `composeFilters` to compose `blur` and `colorOverlay`
* BONUS: what's the most generic type you can come up with for `composeFilters`?
* BONUS 2: can you abstract parts of `blur` and `colorOverlay`?

---

### Compose

```swift
func compose<A,B,C>(f1: B -> C, f2: A -> B) -> A -> C {
    return { x in f1(f2(x)) }
}
```

---

### Compose as operator

```swift
infix operator >>> { associativity left }

func >>> (filter1: Filter, filter2: Filter) -> Filter {
    return {img in filter1(filter2(img))}
}
```


---

```swift
let myFilter = blur(blurRadius) >>> colorOverlay(overlayColor)
myFilter(image)
```

---

# Enums and Structs

---

### Optionals

```swift
enum Optional<T> {
    case None
    case Some(T)
}
```

---

```swift
switch x {
  case .None:
    // Do something
  case .Some(let value):
    // Use `value`
}
```

---

```swift
enum Result<T> {
    case Error(String)
    case Success(T)
}
```

---

```swift
enum GithubAPI {
    case Zen
    case UserProfile(String)
    case Repositories(username: String, sortAscending: Bool)
}
```

---

### Structs

```swift
enum Currency {
    case Eur
    case Usd
}

struct Money {
    let amount : Double
    let currency: Currency
}
```

---

### Exercises

- Define `Person` as a struct 
- Write a function `path`
- Write a `lookup` on dictionaries that returns a `Result<A>` instead of an optional.
- *Bonus*: write an operator that takes an optional and a String and turns it into a `Result`

---

### Mutability

```swift
func sum(numbers: [Int]) -> Int
func product(numbers: [Int]) -> Int
```

---

```swift
func parseJSON(dictionary: JSON) -> [Person]
```

---

### Recap

- Having enums and structs is very precise
- Being explicit about your types helps to find bugs
- Be pragmatic about what works for you

---

# FP and the real world

---

- Simplicity
- Multi-threading ready
- Not for everything

---

- We need objects
- Can we combine OO and FP?

---

- Model layer: functional, structs, enums
- View layer, interactions: OO

---

### Taking this back to OO

- Be explicit about your dependencies
- Use strong types

---

### Loose dependencies

```swift
var person = Person()
person.name = "Chris"
person.address = Address()
person.address.street = "Kotbusser Damm"
```

---

### Strong dependencies

```swift
let address = Address(street: "Kotbusser Damm", ...)
let person = Person(name: "Chris", address: address)
```

Much harder to make an accidental mistake!

---

### Real-world usages

- React.js
- [Facebook Components](http://allfacebook.com/videos-at-scale-2014-mobile-track_b134986)
- ReactiveCocoa
- Scenery

---

### Where to go from here

- Sprinkle in some maps, reduces and filters
- Slowly make your model layer value-based
- Be explicit about dependencies, also in OO
- Use the type-checker for great good
- Books
- FRP

---

### Resources

- http://www.objc.io/snippets/
- http://www.objc.io/books/
- http://learnyouahaskell.com
- http://book.realworldhaskell.org
- http://www.manning.com/petricek/ (F# and C#)
- https://github.com/funjs/book-source

---

### Ideas

- Functional auto-layout description
- Phantom Types
- Wrappers around existing APIs: [snippet 11](http://www.objc.io/snippets/11.html)
- Wrapper types: [snippet 8](http://www.objc.io/snippets/8.html)
- Functional View Controllers

---

### Discussion

- What are the insights?
- How can we apply this in current projects?
- Any questions?

