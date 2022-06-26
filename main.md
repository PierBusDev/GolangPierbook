#  Go Pierbook

## Hello World

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello!")
}
```

## Constants
Constants are considered global variables so don't use too many. 

Basically the value of a constant variable is defined at compile time, not a runtime.

```go
const HEIGHT = 200
//or for multiple constants
const (
	C1 = "C1C1C1"
	C2 = "C2C2C2"
)
```

## Arrays
Creating an array is quite simple
```go
anArray := [4]int{1,2,3,4}
twoD := [4][4]int{ {1,2,3,4}, {5,6,7,8}, {9,10,11,12}, {13,14,15,16}}
```

We can iterate over array with a normal for-loop or using a for-range loop (preferred) like:
```go
for _, subArr := range twoD{
	for _, elem := range subArr{
		fmt.Println(elem, " ")
	}
	fmt.Println()
}
```

### Arrays Shortcomings

- Once defined you **cannot change the size** (if you need to add elements you have to create a bigger array and copy the content)
- When you pass an array to a function as a parameter, you actually **pass a copy** of that array (and if the array is big, copying it will be slow)

These two advantages are enough to make **arrays rarely used** in go.



## Slices
The only case in which we prefere using arrays instead of slices is when we know the max size of the array.

Slices **are passed by reference** to functions, so the modifications we make to them inside the functions are not lost once we exit.

```go
x := [4]int{5,6,7,8}	//array if we specify length
y := []int{5,6,7,8} 	//slice otherwise

z := make([]int, 20)	//created via make
``` 

### SubSlicing
Slicing a slice **DO NOT create a copy of the data**, instead the two variables will share the memory => changes to one will also change the other

```go
func main() {
	x := []int{1, 2, 3, 4}
	y := x[:2]
	z := x[1:]
	x[1] = 20
	y[0] = 10
	z[1] = 30
	fmt.Println("x:", x)	// [10 20 30 4]
	x = append(x,6)
	fmt.Println("x:", x)	// [10 20 30 4 6]
	fmt.Println("y:", y)	// [10 20]
	fmt.Println("z:", z)	// [20 30 4]
}
```


### Make to create a slice
The standard method to create a slice doesn't allow us to create an empty one with predefined length or cap. To do so we have to use `make`:

```go
x := make([]int, 5, 10)
```
In this case we are creating a int slice, with 5 elements (initialized at 0) and a cap of 10! Careful, if we use `append` here we are actually adding at the end of a slice which already is `[0,0,0,0,0]`

```go
x := make([]int, 0, 6)
```
Here we are creating a int slice, with 0 elements (so empty), and with a cap of 6


### Append array to slice
We first need to make a slice of the array, else the compiler will return a type error
```go
s := []int{1,2,3}
a := [3]int{4,5,6}

s2 := a[:]
s := appens(s,s2)
```


### Never use `append` with subslices
We can access multiple continuous sice elements usign the `[x:y]` notation where x is the **included starting** index, and y is the **excluced ending** index
```go
arr := []int{1,2,3,4,5}
fmt.Println(arr[1:3]) 	// [2 3]
```

Whenever you take a slice from another slice, **the subslice’s capacity is set to the capacity of the original slice**, minus the offset of the subslice within the original slice.

```go
	x := []int{1, 2, 3, 4}
	y := x[:2]
	fmt.Println(cap(x), cap(y)) 	// 4 4 
	y = append(y, 30)
	fmt.Println("x:", x) 			// [1 2 30 4]
	fmt.Println("y:", y) 			// [1 2 30]
```
What's happening?
- when we create `y` its length is set to 2 and its capacity to 4 (Same as x)
- appending at the end of `y` puts the value in the third position of `x` because it sees that there are 2 empty position (cap-len = 2) for itself but collaterally it modifies also the third position in `x`!

This behavior can be extremely hard to manage as soon as we add other subslices or append. 

The alternative is using **full slice expressions**, it includes a third value which indicates *the last position in the parent slice's capacity that is available for the subslice*.
```go
y := x[:2:2]
z := x[2:4:4]
```
Both these new slices have a capacity of 2. Limiting their capacity we made sure that appending new elements to these two will not have effect with the other slices!

### From array to slice
It's simple, we just take a slice of it. But be careful -> **a slice of an array share its memory** as a slice of a slice does.


### Creating an independent slice with `copy`
`copy` expects 2 parameters:

1. the destination slice
2. the source slice

It returns the number of elements copies and it tries to copy as many values as it can from source to destination, **limited by whichever is smaller**. 
NOTE: the capacity doesn't matter, the **length** is what is important!

```go
	x := []int{1, 2, 3, 4}
	y := make([]int, 4)
	num := copy(y, x)
	fmt.Println(y, num) 	// [1 2 3 4 ] 4 
``` 

other examples:

Coping just 2 elements:
```go
	x := []int{1, 2, 3, 4}
	y := make([]int, 2) 	// done by reducing the dest slice length
	num = copy(y, x)
```

Copying from a different starting position:
```go
	x := []int{1, 2, 3, 4}
	y := make([]int, 2)
	copy(y, x[2:]) 			// done by subslicing the source slice
```

We can use `copy` also with arrays... just creating a slice from the array:
```go
	x := []int{1, 2, 3, 4}
	d := [4]int{5, 6, 7, 8}		//we will slice this via d[:]
	y := make([]int, 2)
	copy(y, d[:])				
	fmt.Println(y)				// [5,6]
	copy(d[:], x)				
	fmt.Println(d)				// [1,2,3,4]
```
Remember that this work because a slice created from an array **shares its memory** as a slice created from another slice!


### Slices and Strings
We can indeed use the slice notation to get part of strings:
```go
	var s string = "Hello there"
	var s2 string = s[4:7]
	var s3 string = s[:5]
	var s4 string = s[6:]
```

But remember that **strings are immutable** in go, so modifying is not possible and so we don't have the modification problems that slices of slices have.


### Sorting a slice
We use the function `sort.Slice()`.

Assuming to have a slices `mySlice` with elements defined by the type 
```go
type aStructure struct{
	person string
	height int
	weight int
}
```

we sort it like:
```go
sort.Slice(mySlice, func(i, j int) bool { //using an anonymous function
	return mySlice[i].height < mySlice[j].height
})
```

---
## Strings

Strings in go are value types, not pointers like in C. Go also supports UTF8 by default, without the need to import a specific package.

A go string is a **read-only byte slice** that can hold any type of byte and have arbitrary length. To find the length of a string we use the `len()` function. 

`fmt.Printf` has a `%q` verb to print a double-quoted string safely escaped, while `%+q` will garanteee ASCII only output.


### What is a rune
A `rune` is an `int32` value used to represent Unicode (UTF-8) code point (aka a numeric value used for representing single unicode chars). A *rune literal* or *rune constant* is a character in single quotes.

Note that in go **strings are made of runes** and not characters as they are usually in other languages.

[Detailed blogpost here](https://go.dev/blog/strings)

```go
func main() {
	const s = "สวัสดี"
	fmt.Println("Len: ", len(s)) //this will produce the lenght of the bytes stored inside ( == 8)
}
```

We can compare a *rune literal*  to a rune value directly:

```go
func examineRune(r rune) {
    if r == 't' {
        fmt.Println("found tee")
    } else if r == 'ส' {
        fmt.Println("found so sua")
    }
}
```


### JSON
The `encoding/json` package offers the `Encode()` and `Decode()` functions, allowing conversion of Go objects into JSON documents and viceversa.

The functions above work on single objects, while `Marshal()` and `Unmarshal()` work on multiple objects.


```go
func loadFromJSON(filename string, key interface{}) error{
	in, err := os.Open(filename)
	if err != nil{
		return err
	}

	decodeJSON := json.NewDecoder(in)
	err = decodeJson.Decode(key)
	if err != nil {
		return err
	}
	in.Close()
	return nil
}


func saveToJSON(filename *os.File, key interface{}){
	encodeJSON := json.NewEncoder(filename)
	err := encodeJSON.Encode(key)
	if err != nil {
		fmt.Println(err)
		return
	}
}
---

## Maps
Built-in data type for situation where you want to associate one value to another, defined like `var someMap map[string]int` (notice this is unitialized or a **nil map**, so trying to access it will cause a panic)

We can declare it like:
```go
	effects := map[string]int{}
```

In this case it is initialized but empty (we can read and write on it)

If we want to create a not empty map:
```go
	//note the value type is []string
	teams := map[string][]string { 				
		"Milano": []string{"inter", "milan"},
		"Torino": []string{"torino", "juventus"},
	}
```

Passing a map to the `len` function returns the number of pair elements inside the map.


Read and Write operation just use the classic array/slice notation, no fancy methods.

Notice: when we try to **read the value assigned to a map key that was NEVER set** the map **returns the zero value** for that map's value type.


**IMPORTANT**: for security reasons, the order of the elements in a map is not fixed, in the sense that if I have to maps and I try to insert the same elements in both, usually each map will put those in a different order (this is important when using for-range)
**BUT** printing via `fmt.Println` will put the map keys in a ascending sorted order anyway!!!

It is possible to iterate over a map via
```go
for key, value := range myMap{
	fmt.Println(key, value)
}
```

The *keys* of a map can be of **any type that is comparable**, which means any type on which we can use the `==` operator.

### Maps comma ok idiom
To know how to differentiate from a zero (or another default value) returned because the element it's not instantiated and a zero returned because that's the value for the requested key, we use the go **comma ok idiom**

```go
	m := map[string]int{
		"hello": 5,
		"world": 0,
	}
	
	v, ok := m["world"]
	fmt.Println(v, ok)	//0, true
	
	v, ok = m["bye"]
	fmt.Println(v, ok)	//0, false
```


### Deleting from maps
Key-value pairs are deleted using the built-in `delete` function:

```go
	delete(m, "hello")
```

Note: it doesn't return a value and nothing happens is the key is not present inside the map.


### Assigning to a nil map will panic
```go
func main() {
	aMap := map[string]int{}
	aMap = nil
	fmt.Println(aMap) // map[]
	aMap["test"] = 1 //panic: assignment to entry in nil map
}
```

### Set (data structure)
Go doesn't offer a Set datastructure
To simulate it we use a map with the key of the type of the value we would want to put into the set, and bool as the type of the value

```go
	intSet := map[int]bool{}
	vals := []int{1, 4, 5, 9, 4, 1}

	for _, v := range vals {
		intSet[v] = true
	}

	fmt.Println(len(vals), len(intSet)) 	// 6 4 
	fmt.Println(intSet[5])					// true
	fmt.Println(intSet[500])				// false
	if intSet[100] {
		fmt.Println("100 is in the set")
	}
```

For advanced functionalities related to set look for third party libraries.



---
## Structs

Go doesn't have classes because it doesn't have inheritance.

A struct is defined using the keyword `type`, the name of the struct and the keyword `struct`:
```go
	type city struct {
		name string
		populationMillions int
		dimensionKMq int
	}
```

then we can define variables of that type like:
```go
	var torino city		// var declaration (zero valued)
	
	milan := city{}		// no difference, as above 
	
	naples := city{
		"Naples",
		2,
		35
	}
	
	olbia := city{			// with this we can leave out keys or change order
		dimensionKMq: 20,
		name: "Olbia"
							// any field not specified is zero valued
	}
```

A field in a struct is accessed via dotted notation:
```go
	olbia.name = "Olbia Costasmeralda"
	fmt.Println(olbia.name)
``` 

NOTE: It’s **idiomatic to encapsulate new struct creation in constructor functions**

```go
type person struct {
	name string
	age  int
}

func newPerson(name string) *person {
	p := person{name: name}
	p.age = 42
	return &p
}
```

NOTE: we can access field of *pointers to struct* with the same notation! The **pointer is automatically dereferenced**: 

```go
pperson := newPerson("Jack")
fmt.Println(pperson.age) //42
pperson.age = 51
fmt.Println(pperson.age) //51
```


### Anonymus structs
We can declare variables that implement a struct type without giving it a name first.
 
Eg1: 
```go
	var person struct{
		name string
		age int
		pet string
	}

	person.name = "bob"
	person.age = 50
	person.pet = "dog"
```

Eg2:
```go
	pet := struct {
		name string
		kind string
	}{
		name: "Miù",
		kind: "cat",
	}
```

They are used:

- when we need to translate external data into a struct or struct into external data
- when writing tests (slice of anonymous structs when writing table driven tests)



### Struct Comparability and Type Conversion

Easy: structs that are composed of comparable types are comparable... struct with slices or map fields **are not**.


For *type conversions* from one struct to another one the field of both structs **MUST** have the same names, the same orders and the same types.

Also it is not possible to convert to a struct that has additional fields!



---

## Loops and Blocks

**if blocks** can scope variables both for all the blocks related:
```go
if n := rand.Intn(10); n == 0 {
	fmt.Println("That's too low")
} else if n > 5 {
	fmt.Println("That's too big:", n)
} else {
	fmt.Println("That's a good number:", n)
}
```
Once the last else ends, `n` is again undefined!


### for Loop
In go it is the only type of loop, but can be used to simulate all the loop constructs of other languages.

##### 1. complete for
Standard loop present in almost any language

```go
for i := 0; i < 10; i++ {
	fmt.Println(i)
}
```
Note that we must use `:=` to initialize variables, `var` is not allowed here!


##### 2. Condition only for
This basically simulates a `while` statement of other languages

```go
i := 1
for i < 100 {
	fmt.Println(i)
	i = i * 2
}
```


##### 3. Infinite for 

```go
for {
	fmt.Println("Hello")
}
```

Obviously to exit from these type of loops we can use `break` and `continue` keywords, that we can use in any type of for loop. 

Go has also the `labels` to which we can point continue to.


##### 4. for-range (MOST USED)

We can use `for-range` statement only over built-in compound types and user-defined types based on them.

```go
evenVals := []int{2, 4, 6, 8, 10, 12}
for i, v := range evenVals {
	fmt.Println(i, v)
}
``` 

Note that we are getting **two** variables: the first is the position in the data structure being iterated, the second is its value.

Of course if you don't need to use the key, just discard it using `_`.
If you want just the key but not the value (eg if you are iterating over a map and you want just its keys), this is a valid for-range loop:

```go
for k := range uniqueNames {
	fmt.Println(k)
}
```

`for-range` it is also **used for iterating over string** because the type of the value will be a `rune`! Instead if we use another for we get bytes, which will be problematic for handling strings


Also the **value** in a for-range loop **is always a COPY** and not a reference of the original: it means that modifying it will not modify the original value.




### Switch

```go
for _, word := range words {
	switch size := len(word); size {
	case 1, 2, 3, 4:
		fmt.Println(word, "is a short word!")
	case 5:
		wordLen := len(word)
		fmt.Println(word, "is exactly the right length:", wordLen)
	case 6, 7, 8, 9:
	default:
		fmt.Println(word, "is a long word!")
	}
}
```

Variables declared inside switch cases are block scoped (eg `wordLen`).
By default **cases in switch statements don't fall through**, no need to `break` like in other languages. 
So when we need more cases to do the same thing, we just use a comma to separate them in the same case block (there is also a `fallthrough` keyword but it is very rarely used).
For the line with `case 6, 7, 8, 9:` we have an empty block, so nothing happens.

NOTE: We can switch on **any type that can be compared with ==**, so any built-in type EXCEPT slices, maps, channels, functions and structs that contain those.


We can also create a **blank switch** that is a switch that does not specify which value we are comparing against. In a normal switch we can just compare for equality, in a blank switch we can use directly a boolean comparision for each case. EG:

```go
for _, word := range words {
	switch wordLen := len(word); {
	case wordLen < 5:
		fmt.Println(word, "is a short word!")
	case wordLen > 10:
		fmt.Println(word, "is a long word!")
	default:
		fmt.Println(word, "is exactly the right length.")
	}
}

//or just

switch {
	case a == 2:
		fmt.Println("a is 2")
	case a > 3:
		fmt.Println("a is bigger than 3")
	case a < 0:
		fmt.Println("a is negative")
	default:
		fmt.Println("a is ", a)
}

```




---

## Functions

Go **doesn't** have named and optional input parameters. If we need to simulate them we just use a struct as a parameter and pass it (not initialized values will have a default value).

Go **does** support **variadic parameters** indicated with three dots before the *type*... it will create a variable which will be a `slice` of the specified type.

```go
func addTo(base int, vals ...int) []int {
	out := make([]int, 0, len(vals))
	for _, v := range vals {
	out = append(out, base+v)
	}	
	return out
}

```

we can also supply directly a slice using the `...` after the slice itself:

```go
a := []int{4, 3}
fmt.Println(addTo(3, a...))
fmt.Println(addTo(3, []int{1, 2, 3, 4, 5}...))
```


#### multiple return values

```go
func div(numerator int, denominator int) (int, int, error) {
	if denominator == 0 {
		return 0, 0, errors.New("cannot divide by zero")
	}

	return numerator / denominator, numerator % denominator, nil
}
```
Careful to not put parentheses around the return values! They are only on the return type definition!

Also it is not like in python when you can assign multiple return values to a single variable, that here will be a compile time error!

If we want to ignore a value, as usual, just use `_`.


#### Named return values

```go
func divAndRemainder(numerator int, denominator int) (result int, remainder int, err error) {
	if denominator == 0 {
		err = errors.New("cannot divide by zero")
		return result, remainder, err
	}	
	result, remainder = numerator/denominator, numerator%denominator
	return result, remainder, err
}
```

What we are doing is just pre-declaring (and so initializing) variables that we use within the function. We can return them before any explicit use or assignment.
Obviously those name are enforced only INSIDE the function.


#### Functions are values

So they can be used in any place we usually put values, for example as the value part of a map or passed as parameters of another function!

We can also define new functions within a function and assign them to variables (*anonymus functions*).

Eg example of one written inline e immediatly called:

```golang
func main(){
	for i:=0; i < 5 ; i++ {
		func(j int){
			fmt.Println("printing", j)
		}(i) //calling it here
	}
}
```

NOTE that it is a compile time error to put a function name between func and the input parameters!



#### Returning a function
Being values, we can of course return a function from another one:

```golang
func increment(x int) func() int{
	return func() int{
		x++
		return x
	}
}
```



#### Closures

Functions declared inside other functions are special because they are **closures**, which means that they are **able to access and modify variables declared in the outer function!**

```go
func intSeq() func() int{ //note that the return type is "func() int", aka a func which will return an int
	i := 0
	return func() int{
		i++
		return i
	}
}

func main() {
	nextInt := intSeq()

	fmt.Println(nextInt()) //1
	fmt.Println(nextInt()) //2
	fmt.Println(nextInt()) //3
}
```


#### Function Type Declarations

We can use the `type` keyword to define a function type:   
```golang
type opFuncType func(int,int) int

var opMap = map[string]opFuncType{
	//...
}
```


#### DEFER

Programs often create *temporary resources* (files, network connections, etc) that must be cleaned up. This clean up in GO is attached to the `defer` keyword.

As an example look at this reimplementation of the `cat` command in bash:

```golang
func main() {
	if len(os.Args) < 2 {
		log.Fatal("no file specified")
	}

	f, err := os.Open(os.Args[1])
	if err != nil {
		log.Fatal(err)
	}

	defer f.Close() // <============ HERE
	data := make([]byte, 2048)
	for {
		count, err := f.Read(data)
		os.Stdout.Write((data[:count]))
		if err != nil {
			if err != io.EOF {
				log.Fatal(err)
			}
		}
		break
	}
}
```

Once we have a valid file handle we know we need to close it after we finish using it, no matter how the function will exit! To ensure this cleanup we use the `defer` keyword followed by a function ( the `file.Close()` method in this case). **Normally a function call runs immediately, with `defer` the invocation is delayed until the surrounding function exits!**

BE CAREFUL this also means that **any variables passed into a deferred closure is not evaluated until the closure runs!!**. 

Also there is no way to read a return value from a deferred function.


NOTE that **we can defer multiple closures** in go functions, and they run in **LIFO** order: the last `defer` we put in the body will be the first to run!

It is a very common pattern in GO for **any function that allocates a resource to also return a closure that cleans up that resource**. So for example if we want to have a function which opens a file:

```
func getFile(name string) (*os.File, func(), error) {
	file, err := os.Open(name)
	if err != nil {
		return nil, nil, err
	}

	return file, func() {
		file.Close()
	}, nil
}
```

and we call it in the main with:
```
func calling() {
	f, closer, err := getFile(os.Args[1])
	if err != nil {
		log.Fatal(err)
	}
	defer closer()
}
```

Thanks to the fact that Go does not allow unused variables, returning the function `closer` means that the program would not compile if that is not called! So that will remind us to use `defer`.




---

## Go is Call-by-value

It means that whenever we supply a variable for a parameter to a function, Go **always makes a copy of the value** of the variable.

This behavior is different for `maps` and `slices` (whatever they are passed directly or inside a struct):
- for the `map` any change we make is reflected in the variable passed
- for the `slice` we can modify any present element but **we cannot lengthen the slice itself**

This of course happens because maps and slices are implemented via pointers.



---

## Pointers

As usual a pointer is just a variable that holds the location in memory where a value is stored. While different types of variables can take different numbers of memory location, the pointer is always the same size.

The zero value for a pointer is `nil` (which is not equal to 0 like it is in C).

Pointer Arithmetic is not allowed in Go.


The `&` is the *address operator* and preceding a *value* type it returns its address.

The `*` is the *indirection operator* and preceding a *pointer* type it returns the pointed-to value. This is also called **dereferencing**.

```go
x := 10
pointerToX := &x

fmt.Println(pointerToX) // prints a memory address
fmt.Println(*pointerToX) // prints 10

z := 5 + *pointerToX
fmt.Println(*pointerToX) //prints 15
```

BE CAREFUL: **before dereferencing a pointer you must be sure it is not nil** else the program will PANIC.


A *pointer type* is a type that represents a pointer and it is written like `var pointerToX *int`.

The built-in function `new` (which is **rarely used**) creates a pointer variable and returns a pointer to a zero value (so not to `nil`) instance of the specified type:
```golang
var x = new(int)
fmt.Println(x == int) //true
fmt.Println(*x) //prints 0
```

In Go we **can't get the address of a constant** so something like this will fail to compile:
```go
type person struct {
	FirstName string
	MiddleName *string
	LastName string
}
p := person{
	FirstName: "Pat",
	MiddleName: "Perry", // This line won't compile
	LastName: "Peterson",
}
```

That because we are trying to assing a `string` to a `*string`. But even if we put the `&` before `"Perry"` we still get an error message saying that we cannot get the address of "Perry". We have two solution: or to introduce a variable to hold the constant value, or to write an helper function that take the type and returns a pointer to that type:

```go
func stringp(s string) *string {
	return &s
}
```

This basically works because when we pass a constant to a function, that constant is copied to a parameter which is a variable. It is the suggested solution to turn a constant into a pointer.



### Pointers indicate Mutable Parameters

Technically speaking in Go only constants provide names for literal expressions that can be calculated at compile time, there is **no other mechanism** to declare an immutable value.

But since Go is a call-by-value language, the values passed to functions are always copies: so **for non-pointer types** like primitives, structs and arrays, this means that **the called function cannot modify the original**. This makes sort of immutable the original values.

Instead if a **pointer** is passed to a function, the function gets a copy of the pointer: this means that **the original data can be modified by the callee**. 


A couple of derived strange implication:
- if we pass a `nil` pointer to a function, we **cannot** make the value non-nil. (it makes sense since we passed a pointer to that memory location)

```go
func main() {
	var f *int //nil pointer
	failedUpdate(f)
	fmt.Println(f) //prints <nil>
}

func failedUpdate(g *int) {
	x := 10
	g = &x //trying to change what g points to, without success
}
```

- if we want the value assigned to a pointer to still be there once out of the function, we must **dereference the pointer and set the value**: dereferencing puts the new value in the memory location pointed to by both the original and the copy.

```go
func testPointer2() {
	x := 10
	failedUpdate2(&x)
	fmt.Println(x) //no effect, x is still 10
	update(&x)
	fmt.Println(x) // x == 20
}

func failedUpdate2(px *int) {
	x2 := 20
	px = &x2 //trying to change what px points to
}

func update(px *int) {
	*px = 20 //changing directly the value pointed by px
}
```


Said so, when possible avoid to use pointers because they make harder to understand data flow and can create extra work for the garbage collector.




### Pointers - Map and Slices

Withing go, a **map** is implemented as a pointer to a struct: passing a map to a function means that we are copying a pointer. That's why modifications made to a map inside a function are reflected in the original variable in which it was passed in.

Because of this **avoid using maps for input parameters or return values**, especially in public APIs.


For **slices** it is more complicated:
- any modification to the content is reflected in the original variable
- but, using `append` to change the length **IS NOT** reflected in the original variable, even if the slice has capacity greater than the length

This last thing happens because the slice is implemented as a struct with three fields: a `int` for length, a `int` for capacity and a `pointer` to a block of memory.

So basically **a slice passed to a function can have its contents modified** but **can't be resized**.




---

## Types, Methods and Interfaces

Some examples on how to define types:
```go
type Person struct{
	FirstName string
	LastName string
	Age int
}

type Score int

type Converter func(string)Score

type TeamScores map[string]Score
```

In Go types are - theoretically defined - only *abstract* (in the sense they specify just WHAT a type should do, not HOW it is done) or *concrete* (specify WHAT and HOW). There are not partially abstract types like in other languages.


### Methods

Go supports methods on user-defined types

```go
type Person struct{
	FirstName string
	LastName string
	Age int
}

func (p Person) String() string{
	return fmt.Sprintf("%s %s, age %d", p.FirstName, p.LastName, p.Age)
}
```

then we can invoke those methods as in any other language

```go
p := Person {
	FirstName: "Fred",
	LastName: "Joe",
	Age: 42
}

output := p.String()
```

Note it looks like a normal function declaration except for `(p Person)` between the keyword `func` and the name of the method `String()`. That part is called **receiver specification**. It is not idiomatic to use `this` or `self` as a name for the receiver, typically like in this case we use directly the first letter of the type.

Obviously methods must be declared in the same package as their associated type.

Even the **receivers** can be *pointer receivers*  or *value receivers*. The rule to decide which to use is:
- If the method needs to modify the receiver or handle `nil` instances --> POINTER receiver
- Else --> VALUE receiver

But generally, if a type as even a single method which use a *pointer* receiver, every single method will use a pointer receiver as common practice.

```go

type Counter struct{
	total			int
	lastUpdated		time.Time
}

func(c *Counter) Increment(){
	c.total++
	c.lastUpdated = time.Now()
}

func(c Counter) String() string {
	return fmt.Sprintf("total: %d, last updated: %v", c.total, c.lastUpdated)
}

var c Counter
fmt.Println(c.String())
c.Increment()			// NOTE: here goes automatically convert this in (&c).Increment()
fmt.Println(c.String())
```


### Type declaration are not inheritance

Of course we can declare a user-defined type based on another user-defined one:
```go
type HighScore Score
type Employee Person
```

Even doing this we are NOT CREATING any hierarchy, so we cannot use a "child-type" whenever in the code we expect a "parent-type".

Also any methods defined for one type is not present also in the sub-defined one.

```go
var i int = 300
var s Score = 100
var hs HighScore = 500
hs = s		//compile error
s = i		//compile error
s = Score(i)	//ok
hs = HishScore(s)	//ok
```



### iota for Enumerations
Go **doesn't** have an enumeration type, instead uses a similar concept called **iota**.

First, as best practice, we create a type based on int that will represent all of the valid values `type MailCategory int`

Next, we use a const block to define a set of values for our type:
```go
const (
	Uncategorized MailCategory = iota,
	Personal
	Spam
	Social
	Advertisements
)
```

The first constant in the block has the type specified and its value set to `iota`. Every subsequent line has not type or value assigned to it. The compiler will automatically assign to the first variable the value of 0, then 1, etc... Of course **when a new const block is created, iota is set back to 0**



### Embedding for code composition
When a struct contains a field of another type **without name**, that field is called an **embedded field** and each of its own fields (and methods!) is automatically *promoted* to the containing struct
```go
type Employee struct {
	Name string
	ID	string
}

func (e Employee) Description() string{
	return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}

type Manager struct{
	Employee // <-- embedded field
	Reports []Employee
}

m := Manager{
	Employee: Employee{
		Name: "JAck",
		ID: "123"
	},
	Reports: []Employee{},
}
fmt.Println(m.ID) // note we are indeed not calling anything like m.Employee.ID
```

If the embedded field has methods or fields with the same name as the containing struct, we need to use the embedded field type to refer to them:

```go
type Inner struct{
	X int
}

type Outer struct{
	Inner
	X int
}

o := Outer{
	Inner: Inner{
		X: 10
	}
	X: 20
}

fmt.Println(o.X)
fmt.Println(o.Inner.X)
```

Of course remember that embedding is not inheritance!


### Interfaces

A simple definition from `fmt` package:
```go
type Stringer interface {
	String() string
}
```

An interface lists the methods to be implemented by a concrete type to meet the interface. By convention they are usually named with an -er ending.

Like in typescript, they are **implemented implicitly**: that means that a concrete type does not declare that it implements an interface... if it has all the methods requested by the definition of the interface (note that of course it can have also some more), it automatically implements it.

Just like we can embed a type in a struct, we can also **embed an interface in an interface**:
```go
type Reader interface {
	Read (p []byte) (n int, err error)
}

type Closer interface{
	Close() error
}

type ReadCloser interface{
	Reader
	Closer
}
```

We can also embed an interface in a struct, we'll see that later.


### Mantra: "Accept interfaces, return struct"

The business logic invoked by your functions should be invoked via interfaces, the output should be concrete types.

One reason to avoid (when possible) returning interfaces is versioning: if a concrete type is returned, new methods and fields can be added without breaking existing code. Instead if we return interfaces we have to change their definition and most probably break our whole codebase.

Returning `error` (which is an interface type) is a common exception of this rule.


### Empty interface represents any type

```go
var i interface{}

i = "hello"
i = 20
i = struct {
	FirstName string
	LastName string
}{"Luke", "Skywalker"}
```

One common use for the empty interface is as a placeholder for data of uncertain schema which is read from an external source (Eg JSON api).

```go
data := map[string]interface{}{} //first {} for interface{} type, secondo to instanciate the map
contents, err = ioutil.ReadFile("testdata/sample.json")
if err != nil{
	return err
}*
defer contents.Close()
json.Unmarshal(contents, &data) //put contents in the data map
```

Another case where we use the empty interface, is a way to store a value in a user-created data structure where we want to store a lot of different types, and we cannot (yet) use generics.

To read back a value stored in an `inteface{}` we need to understand **type assertions** and **type switches**.


### Type Assertions and Type Switches

A **type assertion** names the concrete type that implemented the interface OR names another interface that is also implemented by the concrete type underlying the interface

```go

type MyInt int

func main(){
	var i interface{}
	var mine MyInt = 20
	i = mine
	i2 := i.(MyInt) //i2 is now of type MyInt
	fmt.Println(i2 + 1)
}
```

What happens if a type assertion is **wrong**? Your code **panics**.

```go
i2 := i.(string) //panic: interface conversion: interface{} is main.MyInt, not string
fmt.Println(i2) 
```

Note that even if two types share an underlying type, a type assertion **must match**:

```go
i2 := i.(int)//panic: interface conversion: interface{} is main.MyInt, not int
fmt.Println(i2) 
```

Of course, to avoid a panic we use the comma-ok idiom (and it is convention to always use it, even when sure the assertion is valid):
```go
i2, ok := i.(int)
if !ok {
	return fmt.Errorf("unexpected type for %v", i)
}
fmt.Println(i2+1)
```

Remember that **type assertion is not type conversion**! The former can only be checked at runtime and only applied to interfaces.


When an interface could be of multiple possible values, we use a **type switch**:
```go
func doThings(i interface{}){
	switch j := i.(type){
		case nil:
			//i is nil, j is of type interface{}
		case int:
			//j is of type int
		case MyInt:
			//j is of type MyInt
		case io.Reader:
			//j is of type io.Reader
		case bool, rune: //listing more than one type keeps j as interface{}
			//i is either a bool or rune, j is of type interface{}
		default:
			//no idea what i is, j is of type interface{}
	}
}
```

Note that it is very similar to a normal switch, but instead of specifying a boolean op, we specify a variable of an interface type and follow with `.(type)`.

NOTE that by convention, even if above we did differently, the variable being switched is assigned to one with the same name, defacto shadowing it (so like `i:= i.(type)`)




---

## Errors

Go handles errors (by convention) by returning a value of type `error` (it's an interface) as the **last return value for a function**. When a function executes as expected `nil` is returned as the value for the error parameter.

A new error is created from a string calling the `New` function in the errors package like `errors.New("invalid user id")`. Go doesn't have special constructs to detect if an error was returned, we must use an `if` statement to check the error variable to see if it is non-nil.

Indeed, a `nil` value returned in the error position indicates that there was no error.

`error` is a built-in interface that defines a single method
```go
type error interface{
	Error() string
}
```

Because in go we must read all variables for compiler rules, we must always check the error or explicitely ignore it for example using the underscore.

Alternatively we can use `fmt.Errorf()` to create an error, passing inside a string which can contain formatting verbs like those accepted by `fmt.Printf`. 


### Errors are Value
Since `error` is an interface, we can define our own errors including additional information for logging or error handling

```go
//first defining enumerations for error codes

type Status int

const (
	InvalidLogin Status = iota + 1
	NotFound
)

//defining StatusErr to hold its value
type StatusErr struct {
	Status Status
	Message string
}

func (se StatusErr) Error() string{
	return se.Message
}


//now we can use StatusErr to provide more details about what went wrong
func LoginAndGetData(uid, pwd, file string) ([]byte, error){
	err := login(uid, pwd)
	if err != nil{
		return nil, StatusErr{
			Status: InvalidLogin,
			Message: fmt.Sprintf("invalid credentials for user %s", uid)
		}
	}
	data, err := getData(file)
	if err != nil{
		return nil, StatusErr{
			Status: NotFound,
			Message: fmt.Sprintf("file %s not found", file),
		}
	}

	return data, nil
}
```

Even when defining your custom error types, **always** use `error` as return type for the error result.

NOTE: DO NOT use *type assertions/switches* to access the field and methods of a custom error! As we see later, we are gonna use `error.As`.


### Wrapping Errors
To add additional context when an error is passed back, creating an *error chain*, we have a stdlib function exactly to wrap errors, and it is just `fmt.Errorf` with the use of the special verb `%w`: this create an error whose formatted string includes the formatted string of another error. The convention is to put the verb at the end of the string of the current-level error.

There is also a function called `errors.Unwrap` to which we can pass an error and it returns the wrapped error if there is one, `nil` if there isn't.

In those case in which we are wrapping errors with the **same** message, we can use a `defer` to simplify a bit:

So, from:
```go
func DoSomething(val1 int, val2 string) (string, error){
	val3, err := doThing1(val1)
	if err != nil{
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}

	val4, err := doThing1(val2)
	if err != nil{
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}

	result, err := doThing1(val1)
	if err != nil{
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}
	return result, nil
}
```

to:
```go
func DoSomething(val1 int, val2 string) (_ string, err error){ //note we have to name our returns here
	defer func() {
		if err != nil{
			err = fmt.Errorf("in DoSomeThings: %w", err)
		}
	}()
	val3, err := doThing1(val1)
	if err != nil{
		return "", err
	}

	val4, err := doThing1(val2)
	if err != nil{
		return "", err
	}

	return doThing1(val3, val4)
}
```


### Is and As
To check if the returned error or any errors that it wraps match a **specific sentinel error** ( = errors defined at package level in stdlib) instance, use `errors.Is`. It takes two params: the error that is being checked and the instance we are comparing against. It returns `true` if there is an error *in the error chain* that matches the provided sentinel error.

```go
func fileChecker(name string) error {
	f, err := os.Open(name)
	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err) //wrapping
	}
	f.Close()
	return nil
}

func main() {
	err := fileChecker("not_here.txt")
	if err != nil{
		if errors.Is(err, os.ErrNotExist) { //os.ErrNotExist is indeed a sentinel error
			fmt.Println("that file doesn't exist")
		}
	}
}
```

By default `errors.Is` uses `==` to compare each wrapped error to the passed as parameter one. If we are passing an user-created error, probably we have to implement the `Is` method on it.

```go

//...
func (me MyErr) Is(target error) bool {
	if me2, ok := target.(MyErr); ok {
		return reflect.DeepEqual(me, me2)
	}
	return false
}
```


The `errors.As` function allow us to check if a returned error (or any it wraps) matches a **specific type**. It takes two params: the first is the error being examined, the second is a *pointer*  to a variable of the type that we are looking for.

If a match is found the function returns `true` and the error is assigned to the second parameter, else it returns false.

```go
err :=  AFunctionThatReturnsAnError()
var myErr MyErr
if errors.As(err, &myErr){
	fmt.Println(myErr.code)
}
```


### Panic 

Go generates **panic** whenever there is a situation where the Go runtime is unable to figure out what should happen next (eg for a programming error like trying to read past the end of a slice, or environmental problem like running out of memory).

When a panic happens the current function exits immediatly and any defer attached to it is called. When this is complete, the defers attached to the calling function run, ans so on until main is reached. Then the program exits with a message + stack trace.

It is possible to create your own panic using the builtin function `panic` which takes one parameter which can be of any type.

There is also a built-in `recover` function to capture a panic, which is called from within a `defer` to check if a panic happened. If something was passed to the panic function, it is returned from the recover. Once `recover` happens, execution continues normally.

```go
func div60(i int){
	defer func(){
		//this below is the common convention to use recover
		if v := recover(); v != nil{
			fmt.Println(v)
		}
	}()

	fmt.Println(60/i)
}

func main(){
	for _, val := range []int{1,2,0,6}{
		div60(val) // prints 60, 30, runtime error: integer divide by zero, 10
	}
}
```



--- 

## Concurrency in Go

### Goroutines

Just to have clear definitions:
- **process**: instance of a program being run by the OS. The OS gives to it some resources such as memory and makes sure other processes can't access them.
- **thread**: a process is composed by one or more threads. A thread is a unit of execution that is given some time to run by the OS. Threads within the same process share access to the resources.

**Goroutines** are lightweight process managed directly by go runtime. When a program starts the Go runtime creates some threads and **launches a single goroutine** to run your program. The *Go runtime scheduler* assignes, during the lifetime of our program, each goroutine to a thread,


Advantages of goroutines:
- creation is faster than thread creation (no syscall)
- smaller stack size, but can grow as needed
- switching between goroutines is faster than between threads (no syscall)
- runtime scheduler can optimize decisions (faster to detect blocking I/O, integrated with garbage collector, etc)

All this allows to run thousands of goroutines when needed, without problems.

To start a goroutine we put a `go` keyword before a *function invocation*. Note that **any value returned be a goroutine is ignored!** The result of the function is instead written back to a different `channel` (we see them later)

```go
func process(val int) int {
	//do something with val
}

func runThingConcurrently( in <-chan int, out chan<- int){
	go func(){
		for val := range in {
			result := process(val)
			out <- result
		}
	}()
}
```


### Channels

Goroutines communicate using `channels`. Like `slices` and `maps`, `channels` are built-in (reference) types created using the `make` function:
```go
ch := make(chan int)
```

When passing a channel to a function, we are just passing a pointer to the channel. It's zero value is `nil`.

We use the `<-` operator to interact with a channel:
- to **read** we place it to the left of the channel
- to **write** we place it to the right of the channel

```go
a := <-ch //read from ch and assign to a
ch <- b		//write the value in b to ch
```

**Each value written to a channel can only be read once**. So if multiple goroutines are reading from the same channel, one value written in it will be read by ONLY one of the goroutine.

It's rare for a goruotine to read and write in the same channel.

When assigning a `channel` to a variable or a field or passing to a function, we use the arrow before the `chan` keyword to indicate that the goroutine only **reads**  from that channel: `ch <-chan int`.

We put the arrow after to indicate that the goroutine only **writes** to the channel: `ch chan<- int`.


Be default **every channel is UNBUFFERED**: it means that every write to an open unbuffered channel causes the *writing goroutine* to **pause** until another goroutine reads from the same channel. In the same way, a read from an open unbuffered channels causes the *reading goroutine* to **pause** until another writes in the same channel.

We can also have **BUFFERED** channels which buffer a limited number of writes without blocking until the buffer is filled.

A BUFFERED channel is created specifying the capacity of the buffer when creating the channel:
```go
ch := make(chan int, 10);
```

The functions `len` and `cap` return information about a buffered channel: `len` will tell how many elements are currently in the buffer, and `cap` returns the maximum buffer size. **BUFFER capacity cannot be changed**.

`len` and `cap` used on a unbuffered channel will both return 0.


It is possible to read from a channel using a `for range` loop:
```go
for v :=  range ch { //this loop will continue until the channel is closed (or a break/return is reached)
	fmt.Println(v)
}
```

NOTE: that in this particular `for range` usage we have only one variable declared.


#### Closing a Channel
When done using a `channel` we close it using the builtin `close` function 
```go
	close(ch)
```

Once closed, any attemp to **write** or **close** again the channel will `panic`. Instead a **read** will always succeed. 

- In the case of a BUFFERED channel, reading after close will return all the values not read yet, or the zero value for the channel type if empty.
- In the case of a UNBUFFERED channel, reading after close will return the zero value fort he channel type.

This is a problem because now we need to discriminate between a `nil` element returned because it is the zero value of the channel which is empty, and a `nil` returned because it was an element previously inserted in the channel! To solve this we use the `comma-ok` idiom so we can detect whether a channel has been closed or not:

```go
v, ok := <-ch
```

If `ok == true` the channel is still open, else is closed.

Of course is good practice to always use ok-comma when in doubt that the channel could be closed. Also **the responsability to close the channel** is tipically assigned to the goroutine that writes to the channel.

It is necessary to close the channel only if there is a goroutine waiting for it (eg one running a `for-range` on the channel)... else, anyway, the garbage collector will detect unused channels and close them.

#### Channels Directions
We can specify a direction on a channel type thus restricting it to either sending or receiving.
```go
func pinger(c chan<- string){// c can only be sent to
	//...
}

func printer(c <-chan string){// c can only receive
	//...
}
```
 
A bi-directional channel can be passed to a function that takes send-only or receive-only channels, but the reverse is not true.


#### Channels behavior

For a **Read**:
- UNBUFFERED OPEN channel: pause until something is written
- UNBUFFERED CLOSED: returns zero value (use comma-ok to check)
- BUFFERED OPEN: pause if buffer is empty
- BUFFERED CLOSED: returns a remaining value in the buffer, or zero value if empty (use comma-ok)
- `nil`: hangs forever

For a **Write**:
- UNBUFFERED OPEN channel: pause until something is read
- UNBUFFERED CLOSED: **panic**
- BUFFERED OPEN: pause if buffer is full
- BUFFERED CLOSED: **panic**
- `nil`: hangs forever

For a **Close**:
- UNBUFFERED OPEN channel: closes it
- UNBUFFERED CLOSED: **panic**
- BUFFERED OPEN: closes it, remaining values still there
- BUFFERED CLOSED: **panic**
- `nil`: **panic**


Because as said before we must **avoid panics**, the standard pattern is to let the writing goroutine to be responsible for closing the channel. 

If there is more than one go routine writing in the same channel it all becomes more complicated... to address this, as we will se below, we have to use something like `sync.WaitGroup`.


#### Select
The `select` statement is the control structure for concurrency in Go. It solves the problem "*if you can perform two concurrent operations, which one do you do first?*" (Remember that favoring one over the other is extremely problematic because you risk to never process some cases -> **starvation**)

The `select` keyword allows a goroutine to **read or write to one of a set of multiple channels**

```go
select {
	case v := <-ch:
		fmt.Println(v)
	case v : = <-ch2:
		fmt.Println(v)
	case ch3 <- x:
		fmt.Println("wrote", x)
	case <-ch4:
		fmt.Println("got value on ch4, but ignore it")
}
```

It is visually similar to a `switch`, but each `case` is a read or a write to a channel: if a read/write is possible for a case, it is executed with the body of the case itself.

If multiple cases have channels that can read or write the `select` **picks randomly from any of its cases** and the **order is unimportant**. Note that this is very different from a normal `switch` which takes just the first `case` at true. This **solves also the starvation** problem because all cases are checked at the same time.

It also prevents the most common case for **DEADLOCKS** which is acquiring locks in an inconsistent order: eg when we have 2 goroutines both accessing the same 2 channels, we must be sure to access them in the same order in both goroutines or they will deadlock and wait on each other.

If **every** goroutine in a go application is deadlocked, the go runtime will kill the whole program.

```go
//example of a deadlock which will kill the program
func main(){
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func(){
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()

	v := 2
	ch2 <- v
	v2 := <-ch1
	fmt.Println(v, v2)
}
```

The program above will return the error `fatal error: all goroutines are asleep - deadlock!`

Remembering that also our `main` is a goroutine launched at startup, the goroutine launched inside it cannot proceed until ch1 is read, while the `main` goroutine cannot proceed until ch2 is read! To solve this problem we use indeed a `select`:


```go
func main(){
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func(){
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()

	v := 2
	var v2 int
	select{
	case ch2 <- v:
	case v2 = <-ch1:
	}
	fmt.Println(v, v2)
}
```

Now running this program will return `2 1`. This is because the `select` will check if **any** of its cases can proceed, avoiding the deadlock.

Since `select` is responsible for communicating over a number of channels, it is often put inside a `for` (and commonly called a `for-select` loop):
```go
for {
	select {
		case <-done:
			return
		case v := <-ch:
			fmt.Println(v)
	}
}
```

When using a *for-select* it is important to **include a way to exit the loop** (more details below).

A `select` statement can also have a `default` clause which is selected when there are no cases with channels that can be read or written.
When we use a `for-select` it is considered **almost always wrong** to have a default case inside because it will make the loop run constantly (instead of pausing), causing a great consumption of cpu resources.



### Concurrency Practices and Patterns


#### Keep APIs Concurrency-Free
You should never expose channels and mutex in your APIs types, functions or methods, else you put the responsability of managing them in the users of the API

#### Goroutines, for loops and varying variables
Typically the closure used to launch a goroutine has no parameters, just captures values where it was declared: this **doesn't work** when trying to capture a index or value of a `for` loop.

```go
func main(){
	a := []int{2,4,56,8,10}
	ch := make(chan int, len(a))
	for _, v := range a{
		go funct(){
			ch <- v * 2
		}()
	}
	for i := 0; i < len(a); i++ {
		fmt.Println(<-ch)
	}
}
```

Even if it looks like we are passing everytime a different `v` value of `a`, running the code will print `20 20 20 20 20`. When the goroutines run the only one they see is the last one captured by the loop (10).

This problem is NOT unique for the `for` loops: **everytime a goroutine depends on a variable which value might change** we MUST pass the value INTO the goroutine.

Two ways to do this: the first one is **shadow the value within the loop**:
```go
for _, v := range a {
	v := v
	go func(){
		ch <- v * 2
	}()
}
```

the second - and preferred one - is to **pass the value as a parameter** to the goroutine:
```go
for _, v := range a {
	go func(val int){
		ch <- v * 2
	}(v) //here passing the parameter
}
```

#### Cleanup the goroutines
Whenever we start a goroutine, we must make sure it will eventually exit (the go runtime CANNOT detect when a goroutine will never be used again). If the goroutine does not exit, the scheduler will periodically allocate to it time to do nothing (*goroutine leak*).


#### "Done channel" pattern
Used to provide to a goroutine that it's time to stop processing / exit.

Example where we pass the sama data to multiple functions but only want to get the result from the fastest one:
```go
func searchData(s string, searchers []func(string) []string) []string{
	done := make(chan struct{})
	result := make(chan []string)
	for _, searcher := range searchers {
		go func( searcher func(string) []string){
			select{
				case result <- searcher(s):
				case <-done:
			}
		}(searcher)
	}
	r := <-result
	close(done)
	return r
}
```

Above we declare a channel called `done` which contains data of type `struct{}`. We use this type because the value is unimportant because **we never write to this channel, we just close it**. Then we lunch a goroutine for each searcher passed in. The `select` then waits either for a *write* on the `result` channel or a *read* on the `close` channel. (remember, a read on a open channel pauses until there is data, a read on a closed channel always return the zero value).

The case which waits on a read on the `done` channel will stay paused until `done` is closed. We close `done` only after we get a result, this allow the `select` to exit and the goroutine to end.


#### "Cancel function" to terminate goroutine
Used together with the `done` channel pattern, we return a cancellation function with the channel. For this to work, the function must be called AFTER the `for` loop:

```go
func countTo(max int) (<-chan int, func()) {
	ch := make(chan int)
	done := make(chan struct{})
	cancel := func(){
		close(done)
	}

	go func(){
		for i := 0; i < max; i++{
			select {
				case <- done:
					return;
				case ch<-i:
			}
		}
		close(ch)
	}()
	return ch, cancel
}

func main() {
	ch, cancel := countTo(10)
	for i := range ch {
		if i > 5{
			break
		}
		fmt.Prinln(i)
	}
	cancel()
}
```

The `countTo` creates two channels, one for returning the data and one for signaling done. Instead of returning the done channel directly (and so delegating to the client how to manage a channel) we return the `cancel` function that closes the channel via a closure. 


#### Buffered or Unbuffered?
Choosing to use a Buffered channel means that we must choose a size (it is not possible to have a unlimited one) and we must handle the case where the buffer is full.

So when to use Buffered instead of a simplier Unbuffered? Buffered channels are useful when:
- we know how many goroutines we have launched
- we want to limit the number of goroutines we are gonna launch
- we want to limit the amount of work queued up

Example: processing the first 10 results on a channel launching 10 goroutines each of which writes the result on a buffered channel:
```go
func processChannel(ch chan int) []int{
	const conc = 10
	result := make(chan int, conc) //buffered channel with 10 spaces
	for i := 0; i < conc; i++{
		go func(){
			v := <-ch
			results <- process(v)
		}()
	}

	var out []int
	for i := 0; i < conc; i++{
		out = append(out, <-results)
	}
	return out
}
```

We know exactly how many goroutines we have launched and we want each of them to exit as soon as it finishes the work. We know there will be no blocking on each write.



#### Backpressure
Another tecnique we can use with **buffered** channels is **backpressure**: we can use a buffered channel and a `select` to limit the number of simultaneous requests to a system:

```go

type PressureGauge struct{
	ch chan struct{}
}

func New(limit int) *PressureGauge{
	ch := make(chan struct{}, limit)
	for i := 0; i < limit; i++ {
		ch <- struct{}{}
	}
	return &PressureGauge{
		ch: ch,
	}
}

func (pg *PressureGauge) Process(f func()) error{
	select{
		case <- pg.ch:
			f()
			pg.ch <- struct{}{}
			return nil
		default:
			return errors.New("no more capacity")
	}
}
```

- First we create a struct containing a buffered channel with a number of tokens and a function to run.
- Every time a goroutine wants to use the function, it calls `Process`
- The `select` tries to read a token from the channel
- If it can the function runs and, after it, the token is returned to the buffered channel
- If it can't, the `default` case runs, and an error is returned instead

This pattern can be used, for example, to limit the use of an http server.



#### Turning off a case in a select
If one of the cases in a `select` is reading a closed channel, it will always be successfull and returning the zero value, so every time the case is selected you need to check to make sure that the value is valid, else the program is gonna waste a lot of time reading garbage values.

We rely on something that looks like an error: reading a `nil` channel. Doing so causes the code to hang forever, so we can use this bad effect to disable a case in a select: **when we detect that a channel has been closed, we set its variable to `nil`** and the associated case will no longer run!

```go
for{
	select{
		case v, ok := <-in:
			if !ok{
				in = nil //this case will never succeed again
				continue
			}
			// ... process v read from in
		case v, ok := <-in2:
			if !ok{
				in2 = nil //this case will never succeed again
				continue
			}
			// ... process v read from in2
		case <-done:
			return
	}
}
```

#### Timing out 
It is often useful to manage how much time a request (or part of it) has to run. Go has not a specific feature to do it, but we use some conventions:

```go
func timeLimit() (int, error){
	var result int
	var err error
	done := make(chan struct{})

	go func(){
		result, err = doSomeWork()
		close(done)
	}()

	select{
		case <- done:
			return result, err
		case <- time.After(2 * time.Second):
			return 0, errors.New("work timed out")
	}
}
```

This is a common pattern to limit how long ad operation takes in Go.

We have a `select` choosing over two cases: 
- the first case use the `done` channel pattern that we expleined above. The goroutine closure assigns values to `result` and `err`, and eventually closes the `done` channel. If the `done` channel closes first, the read from `done` succeed and the values are returned.
- the second case is returned by the `time.After` function and has a value written on it once the specified `time.Duration` has passed. When thi value is read BEFORE `doSomeWork` finishes, we will return an error. Note that in this last case the goroutine of `doSomeWork` will keep running, we just don't use the result: if we want to stop it we must use something like the cancellation seen before.


#### WaitGroups
Sometimes one goroutine needs to wait for multiple goroutines to complete their work. To wait for a single goroutine we keep using the `done` channel pattern as before, but for this new case we need to use `waitGroup` from the `sync` package of the stdlib

```go
func main(){
	var wg sync.WaitGroup
	wg.Add(3)

	go func(){
		defer wg.Done()
		doThing1()
	}()

	go func(){
		defer wg.Done()
		doThing2()
	}()

	go func(){
		defer wg.Done()
		doThing3()
	}()

	wg.Wait()
}
```

First, note that a `WaitGroup` doesn't need to be initialized, just declared. It has 3 methods:
- `Add`: increments the counter of goroutines to wait for. Usually it is called once with the number of goroutines which are goona be launched.
- `Done`: decrements the counter and is ccalled by a goroutine when it is finished. Called inside the goroutine and always using a `defer` so we are sure it is called even if the goroutine panics
- `Wait`: pauses the goroutine until the counter hits zero

Notice **we don't explicit pass the `wg` variable**, we do so because: 
- we must assure every goroutine is using the same instance: if we forgot to pass via a pointer the goroutine will get just a copy
- by design, we just let the closure manage the concurrency


Here an useful example where we use a `WaitGroup` to make sure the channel in which we are writing is being closed only once:
```go
func processAndGather( in <-chan int, processor func(int) int, num int) []int{
	out := make(chan int, num)
	var wg sync.WaitGroup
	wg.Add(num)

	for i := 0; i < num; i++{
		go func(){
			defer wg.Done()
			for v := range in{
				out <- processor(v)
			}
		}()
	}

	go func(){
		wg.Wait()
		close(out) // this will be executed only when wg counter == 0
	}()

	var result []int

	for v := range out{ //exits ONLY after out is closed
		result = append(result, v)
	}

	return result
}
```

`WaitGroups` should not be your first choice for coordinating goroutines, but must be **used mainly when you have something to clean up** like closing a channel in which things were being written.



### Mutexes instead of Channels
In other programming languages we usually use `mutex` (short for *mutual exclusion*). The job of a mutex is to **limit the concurrent execution of code** or **to limit the acces of shared data**. The protected parts are called *critical sections*.

The main problem of mutex is that they obscure the flow of data through a program, because there is nothing to indicate which goroutine currently has ownership of the value. 

But there are some situation in which a mutex is better than a channel, so the go stdlib includes some mutex implementations. The most common situation in which mutexes are used is when goroutines read or write a shared value, but don't process it.

The are two mutex implementation in the `sync` package:

The first is called `Mutex` and has two methods `Lock` and `Unlock`. Calling `Lock` causes the current goroutine to pause as long as another one is currently in the critical section. When the critical section is clear the lock is *acquired* by the current goroutine. A call to `Unlock`marks the end of the critical section.

The second is `RWMutex` and allows to have both *reader locks* and *writer locks*: only one writer can be in the critical section at time, while reader locks are shared as in multiple readers can be in the critical section at once. The writer lock is managed by `Lock` and `Unlock` methods, the reader lock is managed by `RLock` and `RUnlock` methods.

Every time we acquire a lock we must make sure that we release it. **It is convention to use a `defer`** statement to call `Unlock` **immediatly** after `Lock` or `RLock`:

```go
type MutexScoreboardManager struct{
	l				sync.RWMutex
	scoreboard		map[string]int
}

func NewMutexScoreboardManager() *MutexScoreboardManager{
	return &MutexScoreboardManager{
		scoreboard: map[string]int{}
	}
}

func(msm *MutexScoreboardManager) Update(name string, val int){
	msm.l.Lock()
	defer msm.l.Unlock()
	msm.scoreboard[name] = val
}

func(msm *MutexScoreboardManager) Read(name string) (int, bool){
	msm.l.RLock()
	defer msm.l.RUnlock()
	val, ok := msm.scoreboard[name]
	return val, ok
}
```

To decide when to use a mutex:
- if you are coordinating goroutines or tracking a value being transformed by a series of goroutines: use channels
- if you are sharing access to a field in a struct: use mutexes
- when data is stored in external services like HTTP servers or dbs: don't use mutex



--- 


## Standard Library