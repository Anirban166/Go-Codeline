# Go-Codeline

Noting down stuff as I learn Go. | 06/09/20 

Go Follows a type system which is strong (fixed types) and statically typed (follows compile-time determination of variables) <br>
- Auto type detection can't select float32: 
```go
a := 32.
fmt.Printf("%v, %T", a, a) 
// Outputs: 32, float64
// Will need to explicitly specify, eg: var a float32 = 32. or 32, or typecast, i.e. float32(a)
```
- Can't re-declare variables, but can shadow them, working similar to scope:
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
- Local variable declared but unused throws compile time error. (prefer the instant detection without compilation from IDEs such as in Eclipse, but okay)
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

