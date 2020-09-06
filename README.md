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
- Follow camelCaps to avoid exporting via variable naming.
- Typed constants are the regular constants with type specified, can be made of all the primitives and can't be evaluated during runtime. Eg:
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
