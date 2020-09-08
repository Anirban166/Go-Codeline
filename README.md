# Go-Codeline

Noting down stuff as I learn Go. | 06/09/20 

### Variables

Go Follows a type system which is strong (fixed types) and statically typed (follows compile-time determination of variables) <br>
- Auto type detection can't select float32: 
```go
a := 32.
fmt.Printf("%v, %T", a, a) 
// Outputs: 32, float64
// Will need to explicitly specify, eg: var a float32 = 32. or 32, or typecast, i.e. float32(a)
```
- Can't re-declare variables, but can shadow them: (similar to scope)
```go
package main
import ("fmt")
var i int = 10
func main() {
  fmt.Println(i)
	var i int = 20
	fmt.Println(i)
} 
// 10
// 20
```
- Local variable(s) declared but left unused throw compile time error(s). (prefer the instant detection without compilation from IDEs such as in Eclipse, but okay)
- Go variable cases:
  - Capital cased variables at the package level are global & exported to other packages.<sup>1</sup>
  - Lower cased versions of the above are local/available/restricted to the package.<sup>2</sup>
  - declarations inside main are local to main, as expected.<sup>3</sup>
```go
var I int // 1
var i int // 2
func main() { var i int // 3 }
```
- String conversions accessible via `strconv` package
```go
import("fmt", "strconv")
func main() { 
  var int a = 10
  var string b
  j = strconv.Itoa(i) // j : "10"
}
```

---
### Primitives
- Boolean via `bool` (Eg: `a := 1 == 2` -> `false`, `a := 1 != 2` -> `true`)
- Default is `false` for `bool`, as per its 0-value
- Integers range from 8 bit to 64 bit (exceeding would require the big pacakge, similar to bit integer package implementations in C++/Java)
- and yep, type conversions are required while mixing them. E.g.:
```go
func main() {
var(a int = 10
    b int8 = 12)
fmt.Println(a+int(b))
}  // O/P: 22
```
- Unsigned variants follow same prefix-appended syntax eg: `uint32` (`8<->32` unsigned, `8<->64` signed)
- Bit operations are same as well. Eg: 
    - and/or:`a&b`, `a|b` 
    - only 11/00 sets: `a^b`, `a&^b`
    - bitshifts L/R: `2^10 << 10` (2<sup>10</sup> * 2<sup>10</sup> = 2048), `a >> b` (2<sup>10</sup> / 2<sup>10</sup> = 1)
- Floating point (follow the IEE 754 standard) & exp. shorthand notations apply (eg: `1.2E14` = `1.2e+14`)
- Complex type is introduced with 64/128 variants. eg: `var a complex64 = 1+2i`. 
    - Operations supported include `/, *, +, -`
    - `real()` & `imag()` give desired components with `size/2` (64 for 128, 32 for 64) 
    - can use `complex` function as an alternative. (eg: `var n complex128 = complex(10, 12)`) //10+12i
- Strings (utf-8) in go are immutable (can't be changed), can be concatenated with `+` and are aliases for bytes. Eg:
```go
a := "Fit"
fmt.Printf("%v, %T", a[1], a[1]) // 105, uint8 
```
Use `string()`:
```go
fmt.Printf("%v, %T", string(a[1]), string(a[1]))// i, string 
```
- Can work with byte slices: (hey not bad!)
```go
func main() {
s := "String"
b := []byte(s) // ASCII values under array []uint8 
}
```
- Runes are type aliases for int32:
```go
var r rune = 'a' // or just r := 'a'
// type int32, value 97
```

---
### Constants
- Follow camelCaps to avoid exporting via variable naming. (and PascalCase for exporting)
- Typed constants are the regular constants with type specified, interpolating with same type. (untyped variant can with similar types)
- They can be made of all the primitives and can't be evaluated during runtime. Eg:
```go
// Import "math"
const x float64 = math.Sin(1.2) // compiler error: not a constant
```
- Shadowing works:
```go
const a int16 = 10
func main() { const a int = 12 // local
              fmt.Printf("%v, %T\n", a, a) // 12, int 
}
```
- Enumerated constant `iota` increments subsequent values by one when used inside a constant block: (and is scoped to it, i.e. another block would reset the value)
```go
const(a = iota
      b
      c
)
const(d = iota)
func main() {
	fmt.Printf("%v\n", a) // 0
	fmt.Printf("%v\n", b) // 1
	fmt.Printf("%v\n", c) // 2
	fmt.Printf("%v\n", d) // 0, from second const block
}
```
- Assigning it to a blank identifier (underscore) ignores the first value:
```go
const( _ = iota + 5
       a
       b
)
func main() {
	fmt.Printf("%v\n", a) // 6, from (0+5)+1
}
```
- A good example (using the above) is evaluating file size using bitshift:
```go
const( _ = iota
       KB = 1 << (10*iota) 
       MB 
       GB
)
```

---
### Arrays & Slices
- Default syntax: `var arrayname [size]type` (functions apply directly, such as `len(arrayname) = size`)
- For the other option, gotta use an ellpsis with the initializer for known set of values in an *array*, ex: `arrayname := [...]int{"go","golang"}`
- Implicitly, copies are created by array assignment:
```go
a := [...]int{10, 12}
b := a      
b[1] = 20  
// a = [10, 12], b = [10, 20]
```
- To avoid copies, a reference can be considered: (similar to C++)
```go
b := &a
b[1] := 20
// a = b = [10, 20]
```
- Remove the ellipsis and we get *slices*, where the values are determined at runtime, unlike compile-time determination for arrays: (here the copy assignment for them works like references, i.e. the address is taken) 
```go
a := []int{10, 12}
b := a
b[1] = 20
// a = b = [10, 20]
```
- Slicing operations (similar to NumPy) will work both on arrays and slices: 
```go
a := []int{10, 20, 30} // Slice
a := [...]int{10, 20, 30} // Array; equivalent to the slice a[:]
b := a[n:] // slice from (n+1)th element to end
c := a[:n] // slice first n elements
d := a[m:n] // slice all the elements between (m,n] (m excluded)
```
- Can use make operator: (args include type, length, capacity)
```go
a := make([]int, 3) // 0-value initialization takes place: (like in C++)
fmt.Println(a) // [0, 0, 0]
b := make([]int, 3, 100)
fmt.Printf("%v, %v", len(a), cap(a)) // 3 100
```
- Can concatenate slices via append, but for dissimilar type it throws error. For types such as integers and an array of integers, the later can be turned into the former by spreading out the values: (like in Javascript by the ellipsis)
```go
a := []int{}
a = append(a, []int{3, 5, 6}...) // 
```
- Removing elements from slices:
```go
a := []int{1, 2, 3}
b := a[:len(a)-1] // [1, 2]
b := append(a[:1], a[2:]...) // [1, 3]
// references to b will change a! (since copies of the array refer to same underlying array data)
// i.e. a becomes [1, 3, 3], which is not expected. (b = [1, 3], expecting the same for a)
```

---
### Maps 
- Syntax: `mapname := map[keytype]valuetype{ k-v pairs }`. Ex: `contactList := map[string]int{"John": 5400323467, "Doe": 5674322119}` 
	- Access is similar to normal map syntax, by the key e.g. `contactList["John"]` gives `5400323467`
	- Adding is simple as well `contactList["Alexandra"] = 9235707791`
	- Removing elements/pairs can be done via the `delete` function: `delete(contactList, "Doe")`
- Can initialize without values using `make`: `mapname := make(map[keytype]valuetype)` 
- While accessing via `mapname[keyname]`, even keys/entries not present (deleted entries are taken into account as well) are returned as 0 when printed. Such as for the above example, random keys not present would give 0:
```go
fmt.Println(contactList["Newperson"]) // 0
```
- The way to go about this (which would create visible confusion in my example, such as whether the "Newperson" key has a contact number of 0, or whether the entry even exists) is to use a variable such as `ok` (similar to HTTP-200. jk) which factors out a boolean indicating if that entry is present in the map or not: (true or false respectively)
```go
pop, ok := contactList["Newperson"]
fmt.Println(pop, ok) // 0, false -> invalid/non-existent entry
pop, ok := contactList["John"]
fmt.Println(pop, ok) // 5400323467, true -> invalid/non-existent entry
```
```go
_, ok := contactList["Alexandra"]
fmt.Println(ok) // true 
```
- Can use with if-statements: (as discussed below)
```go
if pop, ok := contactList["John"]; ok { // If entry with key as "John" exists in map
	fmt.Println(pop) // John's number
}
```

---
### Control Flow
- Going as per the `if` block syntax, no `()` brackets required (similar to python), but note that they need `{ }`, even for the commands within a single line.
- Use of logical and relational operators stay the same, example: `if n < 25 || n > 50 {...}` is equivalent to `if n >= 25 && n <= 50 {...}`.
- Note that go follows short-circuiting, such as for multiple `or` (logical OR, stitched together by `||`) conditions in an `if` test, if one of the conditions returns a true, then the rest of the chained `or` conditions won't be evaluated (similar to lazy evaluation in R). 
- else and nested if-else blocks are the same as expected.
- Due to rounding-off of decimal places, such a computation won't be true: (whilst it is expected to be true, which absolutely holds for any solid integer or even unary decimals)
```go
num := 0.143
if num == math.Pow(math.Sqrt(num, 2)) { 
fmt.Println("same")
} else {
fmt.Println("different")
}
// different 
```
- A better way to check for the above would be to check for the difference between the expected and computed values within a threshold (typical based on the extent of decimal points):
```go
num := 0.143
if maths.Abs(num / math.Pow(math.Sqrt(num, 2)) - 1 < 0.001 { 
fmt.Println("same")
} else {
fmt.Println("different")
}
// same
```
- Switches in go relatively the same as well, with a few exceptions/advantages:
	- No `break` statements are required! But for exclusively neglecting logic from a point within a case, a `break` can be used explicitly.
	- Delimiters of a case are the next cases, which implies no use of curly braces or `{ }` is required to enclose multiple statements for a case.
	- Multiple cases can be considered together in one `case` segment, thereby discarding the use of subsequent cases that fall under the same category. (which were the fall-through cases in other languages, with multiple cases adorning the same code with the ones which fall under the same being left blank or carry-forwarded to the next case `case 1: case 2: case 3: some code for all the 3 cases`)
```go
switch i:=5*3;i {
case 15:
        fmt.Println("15? Hell yeah")
case 1, 3, 7, 9, 11, 13: // multiple cases chained together
	fmt.Println("Odds uptil 15, but not 15")
case 2, 4, 6, 8, 10, 12, 14:
	fmt.Println("Evens uptil 15 (not 15, of course)")	
default:	
	fmt.Println("0 or a number > 15")	
} // 15? Hell yeah
```
- Note that here the cases cannot overlap, hence we had to make a distinct case for each of the numbers seperately. If we were to use the tag sytax, they can overlap:
```go
i := 15
switch i {
case i <= 15:
	fmt.Println("True")
case i <= 20:
	fmt.Println("True as well")
default:
	fmt.Println("False")
} // True
// Here both i <= 15, <= 20 apply and the first result shadows the second.
```
- In order to print both the results for the above, a `fallthrough` can be added to the first case:
```go
...
case i <= 15:
	fmt.Println("True")
	fallthrough
case i <= 20:
	fmt.Println("True as well")
	...
	// True
	// True as well
```
However note that `fallthrough` triggers the next case as true even if the condition is false, i.e. even a condition like `i <= 10` would make it false.  
- One example with interfaces:
```go
var i interface{} = [3]int{} 
switch i.(type) {
case int:
	fmt.Println("Integer") // e.g. i = 1
case float64:
	fmt.Println("Float64") // e.g. i = 1.01
case string:
	fmt.Println("String") // e.g. i = "1"
case [2]int:
	fmt.Println("Integer array of size 2") // e.g. i = [2]int{}
default:
	fmt.Println("Other type")
} // Other type
// since for an array to be equal, sizes must be the same.
```

---
