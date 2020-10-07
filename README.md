# Go-Codeline

| Noting down stuff as I learn Go. | 06/09/20 |
|---|---|

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
- Strings (utf-8) in Go are immutable (can't be changed), can be concatenated with `+` and are aliases for bytes. Eg:
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
- Note that Go follows short-circuiting, such as for multiple `or` (logical OR, stitched together by `||`) conditions in an `if` test, if one of the conditions returns a true, then the rest of the chained `or` conditions won't be evaluated (similar to lazy evaluation in R). 
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
However note that `fallthrough` triggers the next case as true even if the condition is false, i.e. even a condition like `i <= 10` would make it execute the second case.  
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
### Looping
- Go has only for loops :)
- Cut the brackets (like the `if`) and the `for` loops are the same, storing the same 3 aspects (initialization, condition, increment/decrement) seperated by `;`.
- Multiple variables have to be initialized and incremented/decremented in conjunction with each other, i.e. by seperating with `,` for first the variables, then the initialization/inc./dec. part: (The ordinary `for i:=0, j:=0, i < 2; i++, j++ {...}` is invalid)
```go
for i, j := 0, 0; i < 10; i, j = i+1, j+1 { 
	fmt.Println(i, j) 
}
```
- Note that we can't use `i++` and `j++` above since they won't be treated as a statement. (could use them for one variable tho, such as `i++`/`j++` for `i`/`j` only)
- Pre-default initialization of loop variable and going without the initializer works in go as well: (implied initialization)
```go
i := 0
for ; i < 10 ; i++ {...}
```
- More syntactic sugar, to comply with a while:
```go
i := 0
for i < 10 {
	...
	i++
}
```
- Do-while equivalent:
```go
i := 0
for {
	...
	i++
	if(i == 10) { 
		break
	}
}
```
- Can use labels to break out from a precise point or loop stage:
```go
Loop:
	for i := 1; i <= 10; i++ {
		for l := 1; j <= 10; j++ {
			if i * j >= 50 {
				break Loop
			}
		}
	}
```
- For-ranged ftw! (always with key-value syntax, `for key, value := range collectionName {...}`)
```go
slice := []int{10, 12}
for key, value := range slice {
	fmt.Println(key, value)
} // key -> index here, value -> element at index of slice
// 1, 10
// 2, 12
```
- Using it for traversing our map above, we can go with only keys: (can't have `for k, v`, just the `k` is required)
```go
for k := range contactList {
	fmt.Println(k)
}
```
- or only the values as well, instead of the using keys & values: (can't have just the `v`)
```go
for _, v := range contactList{
	fmt.Println(v)
}
```

---
### Defer
- `Defer` moves the execution of the deferred function (note that it doesn't take actually take a function, but rather a function call instead) to the end of the main function, precisely after the main function code is executed, but before it is returned:
```go
func main() {
	fmt.Println("a")
	defer fmt.Println("b")
        fmt.Println("c")
} // a-c-b order result
```
- Deferred functions are executed in LIFO order: (makes sense to close resources in reverse order)
```go
func main() {
	defer fmt.Println("a")
	defer fmt.Println("b")
        defer fmt.Println("c")
} // c-b-a order result
```
- It allows the opening and closing of a resource right next to each other.
- Arguments of a function call are evaluated at the time defer is executed, but not at the time of called function's execution:
```go
func main() {
	x := "a"
	defer fmt.Println(x)
        x = "b"
} // a
```

---
### Pointers
- Basics:
```go
var a int = 10
var b *int = &a // declare a pointer 'b', pointing to memory address of a
fmt.Println(a, *b) // *b -> dereference b (by prefixing with asterisk) and return the value at the memory address pointed by b
// 42 42
// For printing the address, go with b or &a, like the regular syntax.
```
- Pointer arithmetic is not supported (adheering to its 'simplicity' principle) by default, but can be achieved via the `unsafe` package.
- Zero value of a pointer is `nil`:
```go
func main() {
	var es *exampleStruct
	fmt.Println(es) // <nil>
	es = new(exampleStruct)
	fmt.Println(es) // &{0}
}
type exampleStruct struct {
	a int
}
```
- For dereferencing the pointer above, we would require to usually emplace brackets to it since the precedence of `*` is lower to `.`, and hence we would have to go with `(*es).a` to access the value of the struct member `a`:
```go
func main() {
	var es *exampleStruct
	es = new(exampleStruct)
	(*es).a = 10 // n
	fmt.Println((*es).a) // 10
}
type exampleStruct struct {
	a int
}
```
- However, Go's compiler automatically dereferences it for us while using complex types such as structs above, and so even `es.a` would work!
- Note that all assignment operations in Go are copy operations, but slices and maps contain internal pointers implying that the copies point to same underlying data. 

---
### Functions
- Starts with the 'func' keyword, with a typical syntax of `func functionName(variableName type, ...) returnType {...}` (no need to specify the return type for functions that don't return anything)
- Concering arguments of same type, a comma delimited list of variables with the type at the end works as well, such as for ex: `func Test(firstName, lastName string)`.
- Passing by reference (w/pointers) works as expected: (again, directly applicable to slices & maps)
```go
func main() {
	firstName := "Raven"
	lastName := "West"
	Greet(&firstName, &lastName)
	fmt.Println(*lastName) 
}
func Greet(firstName, lastName *string) {
	*lastName := "East"
	fmt.Println(*lastName)
} // East
 //  East 
```
- Variadics have to be the end argument: (similar to C++)
```go
func main() {
	product("Product of first 10 primes: ", 2, 3, 5, 7, 11, 13, 17, 19, 23, 29)
}
func product(message string, values ...int) {
	result := 0
	for _, v := range values {
		result *= v
	}	
	fmt.Println(message, result)
} // Product of first 10 primes: 6,46,96,93,230
```
- Named result parameters can be used with `return`: 
```go
func main() {
	p := product(, 2, 3, 5, 7, 11, 13, 17, 19, 23, 29)
	fmt.Println("Product of first 10 primes: ", p)
}
func product(values ...int) (result int) {
	for _, v := range values {
		result *= v
	}	
	return
} // Product of first 10 primes: 6,46,96,93,230
```
- Addresses of local variables can be returned and are automatically promoted from the local memory (stack) to the shared memory (heap).
- Can return multiple variables by specifying their types together: (error type as well)
```go
func main() {
	d, err := divide(10.0, 0.0)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Division result: ", d)
}
func divide(x, y float64) (float64, error) {
	if y == 0.0 {
		return 0.0, fmt.Errorf("Division by zero error (Inf)")
	}
	return x / y, nil
} // Division result: 10.0
```
- Temporary/Anonymous functions (like lambdas in C++) can be created to take advantage of an isolated scope whenever required:
```go
func main() {
	// Immediately invoking it:
	func() {
		fmt.Println(1)
	} ()
	// Working with the function as a variable:
	f := func() { // signature of var f func() 
		fmt.Println(2)
	} 
	f()
}
```
- Can apply that to our function above: 
```go
func main() {
	var divide func(float64, float64) (float64, error) 
	divide = func(x,y float64) (float64, error) {
		if y == 0.0 {
			return 0.0, fmt.Errorf("Division by zero error (Inf)")
		} else {
			return x / y, nil
		}
	} 
	d, err := divide(10.0, 0.0)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Division result: ", d) // Division result: 10.0
}
```
- Methods in Go are functions executing in a known context: (like for the types of a struct)
```go
func main() {
	g := greeter{
		greeting: "Hello",
		name: "Raven",
	}
	g.greet() // Hello Raven
}
type greeter struct {
	greeting string
	name string
}
func (g greeter) greet() {
	fmt.Println(g.greeting, g.name)
}
```
Note that we are operating on a copy of the `greeter` object above, and hence values changed inside the scope of `greet()` won't have any effect on the values at the calling end. Can use a pointer reciever to change the value:
```go
func main() {
	g := greeter{
		greeting: "Hello",
		name: "Raven",
	}
	g.greet() // Hello!
}
type greeter struct {
	greeting string
	name string
}
func (g *greeter) greet() {
	fmt.Println(g.greeting, g.name)
	g.name = "!"
}
```

---
### Interfaces
- Implements methods, instead of data, as expected. (like Java or C#)
- Implicitly implemented. (doesn't need keywords like `implements`, sorry Java)
- Implementing an interface with a struct:
```go
func main() {
	var w Writer = ConsoleWriter{}
	w.Write([]byte("Heya!")) // Heya!
}
// Defining method signatures inside an interface:
type Writer interface {
	Write([]byte) (int, error)
}
type ConsoleWriter struct {}
func(cw, ConsoleWriter) Write (data []byte) (int, error) {
	n, err := fmt.Println(string(data))
	return n, err
}
```
- Single method interfaces are some of the most powerful and flexible ones. Examples from the standard library inlcude `io.Writer`, `io.Reader` (one method each) and `interface{}` (the empty interface with no methods, which every type in Go implements)
- Avoid exporting interfaces for types that will be consumed, as a best practice. (export for types that will be used by the package)
- Type conversion example:
```go
var wc WriterCloser := NewBufferedWriterCloser()
bwc := wc.(*BufferedWriterCloser)
```

---
### Goroutines
- Basically functions or methods that run concurrently with other functions or methods.
- Go uses green threads, similar to Erlang. (not OS threads, such as in Java/C#)
- `go` is the keyword to prefix with, and here's a basic example:  
```go
import (
	"fmt"
	"time"
)
func main() {
	var message = "hey" // The concept of closures apply, the anonymous function has access to this variable or ones available in this outer scope (rc invoked)
	// Creating a temporary/anonymous function (Go style lambdas lol)
	go func() {
		fmt.Println(message)
	}()
	message = "hi"
	time.Sleep(100 * time.Millisecond)
} // hi
```
Here the Go scheduler is not gonna interrupt the main thread until it hits the `Sleep()` call. The value of `message` gets re-assigned before it gets printed in the goroutine, which becomes a rather good example of a race condition. (which we desire to avoid) <br>
A simple way to avoid this would be to use an argument and pass it by value: (a copy of the variable `message` when it gets passed onto the goroutine)
```go
import (
	"fmt"
	"time"
)
func main() {
	var message = "hey" // The concept of closures apply, the anonymous function has access to this variable or ones available in this outer scope (rc invoked)
	// Creating a temporary/anonymous function (Go style lambdas lol)
	go func(message string) {
		fmt.Println(message)
	}(message)
	message = "hi"
	time.Sleep(100 * time.Millisecond)
} // hey
```
Note that this is a bad practice since we are binding it to the real-world clock with the `Sleep()` call, making the program unreliable. Wait groups are a better alternative.
- Wait group (from `sync`) objects can be created to avoid skipping of output and manage goroutines via a counter:
```go
import (
	"fmt"
	"sync"
	"time"
)
func main() {
	var wg sync.WaitGroup 
	wg.Add(1) // +1 for 1 goroutine
	go func() { // anonymous func.
		count("Count value")
		wg.Done() - 1 from waitgroup counter
	}()
	wg.Wait() // counter 0, done
}
func count(thing string) {
	for i := 1; i <= 3; i++ {
		fmt.Println(i, thing)
		time.Sleep(time.Millisecond * 300)
	}
}
```
Replicating this, we can solve the above problem with wait groups:
```go
import (
	"fmt"
	"sync"
)
var wg = sync.WaitGroup{}
func main() {
	var message = "hey" // The concept of closures apply, the anonymous function has access to this variable or ones available in this outer scope (rc invoked)
	wg.Add(1)
	// Creating a temporary/anonymous function (Go style lambdas lol)
	go func(message string) {
		fmt.Println(message)
		wg.Done()
	}(message)
	message = "hi"
	wg.Wait()
} // hey
```
- Other modes of synchronization include `sync.Mutex` and `sync.RWMutex` to protect data access. (better to lock together and unlock together in some cases)
- Goroutines start with very small stack spaces, but can be re-allocated very quickly. (cheap to create and destroy)
- `GOMAXPROCS` from the `runtime` package gives the number of CPU threads allocated for go to use, which are equal to the number of cores in the machine: (if running on a VM, number of cores exposed apply. Eg: Go playground has one core)
```go
import(
       "fmt"
       "runtime"
      )
func main() {
	fmt.Printf("Threads: %v\n", runtime(GOMAXPROCS(-1)) // -1 by default takes all cores available in the machine.
}
```
And so yep, `GOMAXPROCS(1)` would mean parallelism doesn't exist because only one core is being alotted/available.
Also note that 1 OS thread per core is a mimimum, but the number can be increased and we can have values like `GOMAXPROCS(100)` or higher. But it might lead to the scheduler being overloaded with more threads to look after, and in turn can make the go application a lot slower. 
