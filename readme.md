#  Go Pierbook

This that follows is not a cheatsheet, it is not a reference book and it is not a comprehensive recap of all feature of Golang. This is just a collection of my notes while learning the language. Everything here is written by me, part of it is my original work, part of it is something I read from some books or courses and I reformulated that to better suite my mind, and some of it are things collegues teached to me. 

Hoping this could be helpful also to you dear reader, have a nice read.   

## Hello World

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello!")
}
```


## Variable Declarations

```go
var something string //uninitialized and "zero" valued
var name string = "Pier"
myName := "Pier" //shorthand

var ( //multi line
	city string = "Milan"
	nation string = "Italy"
)
```

Shortand declarations work only inside a function scope, the compiler will report an error if used outside of it.

### Zero Value as a Concept

In Go when we allocate memory for a variable via declarations or by using new and the variable is not explicitly initialized, the value of such variables are automatically initialized with their **zero value**. The initialization of the zero value is done recursively: every element of an array or a structs has its fields *zeroed*.

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

## Integers

Go has `int` and `uint` which size depends on the platform (32bits systems have 32bits int, 64bit systems have 64 bits int)

If you need, and have a valid reason, you can use some other fine grained int (`int8 int16 int32 int64`) or uint (`uint8 uint16 uint32 uint64`) sizes.

Go has also a `byte` type which is an alias of `uint8`.


## Floating Points

Go has `float32` and `float64`.


## Complex

2 types:
- `complex128` with real and imaginary parts as `float64`
- `complex64` with real and imaginary parts as `float32`

```go
var someComplex complex128 = complex(245, 3)
var someComplex2 complex128 = 245 + 3i 		//literal
```


## Bool
As usual the type `bool` can have values of `true` or `false`.


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
- This is because array are **Value Types** instead of reference types like in java, c#, c++, etc... Which means that even when assigning an array to another variable, it will be copied.

```go
	var array = [5]string{"ciao", "hola", "welcome", "salut", "benennidos"}
	var secondArray = array

	secondArray[0] = "buongiorno"
	fmt.Println(array[0]) //ciao
	fmt.Println(secondArray[0]) //buongiorno
```

These two disadvantages are enough to make **arrays rarely used** in go.



## Slices
The only case in which we prefere using arrays instead of slices is when we know the max size of the array.

Slices **are passed by reference** to functions, so the changes we make to them inside the functions are not lost once we exit.

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

It returns the number of elements copied and it tries to copy as many values as it can from source to destination, **limited by whichever is smaller**. 
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

`fmt.Printf` has a `%q` verb to print a double-quoted string safely escaped, while `%+q` will make ASCII only output.


To see a good cheatsheet about *formatting verbs* for most types go [here](https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/).


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

The functions below work on single objects, while `Marshal()` and `Unmarshal()` work on multiple objects.


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
```



---

## Packages and import/export
In Go an application is organized in packages which is a collection of source files located in the same folder. All source files in a folder must have the same package name at the top of the file.

By convention packages are named to be the same as the folder they are located in.

Inside a package any variable or function can be **exported** if they are defined with a name starting by an upper case letter:
```go
package something

var myValue int = 1 // not exported, acts like "private" in oop languages
var MyValue int = 1 //exported
```

To import a variable/function in another package, we first import the package of interest, then call the functions/variables via dot notation:
```go
package main

import "mymodulename/something"

func main(){
	fmt.Println(something.MyValue)
}
```

We can also rename (alias) an import if we need it to avoid collision or just to shorten it:
```go
package main

import some "mymodulename/something"

func main(){
	fmt.Println(some.MyValue)
}
```


If we want to install an external package from another source (commonly a github page) we use the command `go install` 
```go
go install github.com/something/something
```

and then in the imports we explicity refer to the url

```go
package main

import (
	"github.com/something/something"
)
```


---

## Maps

Built-in data type for situations where you want to associate one value to another, defined like `var someMap map[string]int` (notice this is unitialized or a **nil map**, so trying to access it will cause a panic)

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

A struct, which is basically a collection of named fields, is defined using the keyword `type`, the name of the struct and the keyword `struct`:
```go
	type city struct {
		name string
		populationMillions int
		dimensionKMq int
	}
```

Note that if the fields have the same type we can shorthand the definition like:
```go
	type city struct {
		name string
		populationMillions, dimensionKMq int
	}
```


then we can instanciate variables of that type like:
```go
	var torino city		// var declaration (note each field will be zero valued)
	
	milan := city{}		// no difference, as above 
	florence := new(city) //as above, but florence is of type pointer of city
	
	naples := city{		//struct literal
		"Naples",
		2,
		35,
	}
	
	olbia := city{		//named struct literal
		// with this we can leave out keys or change order
		dimensionKMq: 20,
		name: "Olbia",
							// any field not specified is zero valued
	}
```

A field in a struct is accessed via dotted notation:
```go
	olbia.name = "Olbia Costasmeralda"
	fmt.Println(olbia.name)
``` 

NOTE: It’s **idiomatic to encapsulate new struct creation in constructor functions** typically named as `newStructname` eg `newCity`.

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


### Struct are Value Types

Being value types means that when we assign a structure to another one of the same type, a **new copy** is created and assigned:

```go
	p1 := person{"Peter", 23}
	p2 := p1

	p2.age = 33
	fmt.Println(p2.age) //33
	fmt.Println(p1.age) //23
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

If we want to compare two structs of the **same type** that are comparable as defined above, we can use the `==` operator which check the equality of all their fields:

```go
	type salary struct{
		value int
		position string
	}

	s1 := salary{100000, "sw dev"}
	s2 := salary{100000, "dev ops"}

	fmt.Println(s1 == s2) //true
```

For *type conversions* from one struct to another one the field of both structs **MUST** have the same names, the same orders and the same types.

Also it is not possible to convert to a struct that has additional fields!



### Embedding for code composition
When a struct contains a field of another type **without name**, that field is called an **embedded field** and each of its own fields are automatically *promoted* to the containing struct
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

If the embedded field has fields with the same name as the containing struct, we need to use the embedded field type to refer to them:

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

Of course remember that embedding is not inheritance but a form of composition!

Often it is suggested to **avoid it** and prefer instead straightforward composition:
```go
type Inner struct{
	X int
}

type Outer struct{
	I Inner
	X int
}

i := Inner{20}
o := Outer{i, 30}
```



### Struct tags
A `struct tag` is a feature of the go language which allows to attach some sort of metadata to the fields of a struct, which then can be used by the `reflect` package to implement some behaviours.

A common case is to instruct how to marshal/unmarshal json, xml, etc... or for interactions with ORMs.

```go
type sandwich struct{
	ingredient1 string	`json:"name"`
	ingredient2 string	`json:"age"`
}
```


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
Note that we must use `:=` to initialize variables (or `=` if we already initialized a variable with the same name previously in the same scope), `var` is not allowed here!


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
	break
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
	case 1, 2, 3, 4: //note multiple cases
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

NOTE: We can switch on **any type that can be compared with ==**, so any built-in type EXCEPT slices, maps, channels, functions... and structs that contain those.


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


#### Multiple return values

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

#### Naked Return

If named return values are used, a `return` statement without arguments will return those values.
This is known as a *naked* return.

```go
func SumAndMultiplyThenMinus(a, b, c int) (sum, mult int) {
    sum, mult = a+b, a*b
    sum -= c
    mult -= c
    return //this will actually return sum, mult
}
```

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

```go
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

```go
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


Said so, when possible avoid to use pointers because they make it harder to understand data flow and can create extra work for the garbage collector.




### Pointers - Map and Slices

Within go, a **map** is implemented as a pointer to a struct: passing a map to a function means that we are copying a pointer. That's why modifications made to a map inside a function are reflected in the original variable in which it was passed in.

Because of this **avoid using maps as input parameters or return values**, especially in public APIs.


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


### Alias types

An *alias* allows to define just an alternate name for a type. For the runtime the two types are exactly the same and so they are assignable one to the other.

**Note** that this is very different from a new type definition, in that case for the runtime the types are ever different, even if theoretically they are represented in the same way

```go
	type myString = string //type ALIAS, note the "=" 
	type myOtherString string //new type definition

	func main(){
		var somestring string = "ciao"
		var mine myString
		var mine2 myOtherString

		mine = somestring //ok!
		mine2 = somestring //ERROR
	}
```

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

But generally, if a type has even a single method which use a *pointer* receiver, every other method will use a pointer receiver as common practice.

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

Also any method defined for one type is not present also in the sub-defined one.

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
Go **doesn't** have an enumeration type, instead it uses a similar concept called **iota**.

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
**NOTE** since the introduction of the `any` type (which is an alias for `inteface{}`) you should prefer using it directly. In old code is still possible to find `interface{}`.

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
data := map[string]interface{}{} //first {} for interface{} type, second to instanciate the map
contents, err = ioutil.ReadFile("testdata/sample.json")
if err != nil{
	return err
}
defer contents.Close()
json.Unmarshal(contents, &data) //put contents in the data map
```

Another case where we use the empty interface is a way to store a value in a user-created data structure where we want to store a lot of different types, and we cannot (yet) use generics.

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

Go generates **panic** whenever there is a situation where the Go runtime is unable to figure out what should happen next (eg for a programming error like trying to read past the end of a slice, or an environment problem like running out of memory).

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


## Generics

A generic function is defined like this:

```go
	func functionName[T constaint](){
		//...
	}
```

`T` represent our type parameter and `constraint` is the **interface** that will allow us to use any type which implements the interface itself.

Example:

```go
	func saySalary[T any] (employees, salary T){
		fmt.Printf("There are %v employee with a median salary of %v", employees, salary)
	}


	//to call this function we can explicitly pass the type argument or let go infer it
	saySalary[int](5, 10000)
	saySalary("10", "15000") //infers string
```


if we make the function returns something based on an operation over the generic type defined, we get an error

```go
	func saySalary[T any] (employees, salary T) T{
		fmt.Printf("There are %v employee with a median salary of %v", employees, salary)
		return employees * salary
	}

	saySalary[int](5, 10000) //invalid operation
```

This is because a constraint of `any` **doesn't support operators**. But we can define a *type set* which can support the multiplication operator:

```go
	type mydata interface{
		int | float64
	}

	func saySalary[T mydata] (employees, salary T) T{
		fmt.Printf("There are %v employee with a median salary of %v", employees, salary)
		return employees * salary
	}

	saySalary[int](5, 10000) //50000
```

In the experimental packages we have a `constraints` [package](https://pkg.go.dev/golang.org/x/exp/constraints) which defines some useful constraints:

```go
	type Complex interface {
		~complex64 | ~complex128
	}

	type Float interface {
		~float32 | ~float64
	}

	type Integer interface {
		Signed | Unsigned
	}

	type Ordered interface {
		Integer | Float | ~string
	}

	type Signed interface {
		~int | ~int8 | ~int16 | ~int32 | ~int64
	}

	type Unsigned interface {
		~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
	}
```

**NOTE** the new go token `~` which when used near a type means *the set of all types whose underlying type is the one following ~*.


The rule of thumb for using generics is the same of all other languages: use them only when you end up writing very similar code at least 2-3 times.



--- 

## Concurrency in Go

### Concepts

- **data race** -> different processes want to access the same resource concurrently
- **race conditions** -> occurs when the orders of access of different processes affects the correctness of the data 
- **deadlock** -> occurs when all processes are blocked while waiting on eachother
- **livelock** -> all processes are running actively but the operations that are being executed do not move the state of the program forward, basically stalling its execution
- **starvation** -> a process cannot access the resources needed and cannot complete (can happen because of deadlocks, or livelocks due to resource assignment mismanagement)


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

To start a goroutine we put a `go` keyword before a *function invocation*. Note that **any value returned by a goroutine is ignored!** The result of the function is instead written back to a different `channel` (we see them later)

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

A channel is a communications pipe between goroutines. Things go in one end and come out another in the same order until the channel is closed.

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
	//...
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
- In the case of a UNBUFFERED channel, reading after close will return the zero value for the channel type.

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


Because as said before we must **avoid panics**, the standard pattern is to let the writing goroutine be responsible for closing the channel. 

If there is more than one go routine writing in the same channel it all becomes more complicated... to address this, as we will se below, we have to use something like `sync.WaitGroup`.


#### Select
The `select` statement is the control structure for concurrency in Go. It solves the problem "*if you can perform two concurrent operations, which one do you do first?*" (Remember that favoring one over the other is extremely problematic because you risk to never process some cases -> **starvation**)

The `select` keyword t blocks the code and waits for multiple channel operations simultaneously, allowing a goroutine to **read or write to one of a set of multiple channels**

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

If multiple cases have channels that can read or write, the `select` **picks randomly from any of its cases** and the **order is unimportant**. Note that this is very different from a normal `switch` which takes just the first `case` at true. This **solves also the starvation** problem because all cases are checked at the same time.

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

```go
	for x := 0; x < 10; x++ {
		select {
		case result := <-one:
			fmt.Println("Received:", result)
		case result := <-two:
			fmt.Println("Received:", result)
		default:
			fmt.Println("Default...")
			time.Sleep(200 * time.Millisecond)
		}
	}
```


### Concurrency Practices and Patterns

#### Atomic Package

In go stdlib there is an `atomic` [link](https://pkg.go.dev/sync/atomic) package which contains low-level primitives which are very useful to implement synchronization algos. They must be used rarely, synchronization is easier and less error prone when done usyng the `sync` package (see below) and channels.

Methods in the `atomic` package are divided by the type used, we have methods for `int32`, `int64`, `uint32`, `uint64`, `uintptr` and `bool`. 

The atomic methods are: `Swap`, `Add`, `CompareAndSwap`, `Load`, `Store`.

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

A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls `Add` to set the number of goroutines to wait for. Then each of the goroutines runs and calls `Done` when finished. At the same time, `Wait` can be used to block until all goroutines have finished.

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

We can also use `TryLock` which tries to lock and reports if it went successfull.

The second is `RWMutex` and allows to have both *reader locks* and *writer locks*: only one writer can be in the critical section at time, while reader locks are shared as in multiple readers can be in the critical section at once. The writer lock is managed by `Lock` and `Unlock` methods, the reader lock is managed by `RLock` and `RUnlock` methods.

Every time we acquire a lock we must make sure that we release it. **It is convention to use a `defer`** statement to call `Unlock` **immediatly** after `Lock` or `RLock`:

```go
type MutexScoreboardManager struct{
	l				sync.RWMutex
	scoreboard		map[string]int
}

func NewMutexScoreboardManager() *MutexScoreboardManager{
	return &MutexScoreboardManager{
		scoreboard: map[string]int{},
	}
}

func(msm *MutexScoreboardManager) Update(name string, val int, wg *sync.WaitGroup){
	msm.l.Lock()
	defer wg.Done()
	defer msm.l.Unlock()
	msm.scoreboard[name] = val
}

func(msm *MutexScoreboardManager) Read(name string){
	msm.l.RLock()
	defer msm.l.RUnlock()
	val, _ := msm.scoreboard[name]
	fmt.Println(val)
}

func main(){
	var wg sync.WaitGroup
	scoreboard := NewMutexScoreboardManager()

	wg.Add(3)

	go scoreboard.Update("Jack", 5, &wg)
	go scoreboard.Update("Jack", 6, &wg)
	go scoreboard.Update("Julie", 3, &wg)

	wg.Wait()
	go scoreboard.Read("Jack")
	time.Sleep(2 * time.Second)
}
```

To decide when to use a mutex:
- if you are coordinating goroutines or tracking a value being transformed by a series of goroutines: use channels
- if you are sharing access to a field in a struct: use mutexes
- when data is stored in external services like HTTP servers or dbs: don't use mutex

**NOTE** as with waitgroups, also **mutex must NOT be copied but passed as pointers when needed**.




### Cond

`sync.Cond` can be used to coordinate goroutines which want to share resources, signaling to those waiting when the state of the resource of interest changes.

Each `Cond` object has an associated mutex (`*Mutex` or `RWMutex`).

A simple but very common usecase is when we have a process which is receiving data from some external source and other processes must wait for this one so they can read. We could use a simple mutex or channel but in that case we could notify only ONE process, a `Cond` allow us to notify multiple processes at the same time.

It has the methods:
- `NewCond(l Locker)` returns a new Cond
- `Broadcast()` wakes all goroutines waiting on the condition
- `Signal()` wakes ONE goroutine waiting on the condition
- `Wait()` atomically unlocks the mutex lock

Note that `Locker` is an interface implemented both by `sync.Mutex` than `sync.RWMutex`:

```go
type Locker interface {
    Lock()
    Unlock()
}
```


Usage example for `Cond`:

```go
var  writingComplete = false

func readData(readerName string, cond *sync.Cond){
	cond.L.Lock()
	defer cond.L.Unlock()
	for !writingComplete {
		cond.Wait()
	}

	fmt.Println(readerName + " now reading")

}

func writeData( cond *sync.Cond){
	fmt.Println("simulating some writing")
	time.Sleep(1 * time.Second)

	cond.L.Lock()
	writingComplete = true;
	cond.L.Unlock()

	fmt.Println("now signaling to everyone they can read")
	cond.Broadcast()
}

func main(){
	var m sync.Mutex
	cond := sync.NewCond(&m)

	go readData("Reader 1", cond)
	go readData("Reader 2", cond)
	go readData("Reader 3", cond)
	writeData(cond)

	time.Sleep(4 * time.Second)
}
```



### Once

`sync.Once` guarantees that only one execution will be run, even if more than one goroutine is waiting. To do so it has a method `Do(f func())` which, of course, even if called multiple times will ensure that only the first call will invoke the function `f`.

```go

	func main(){
		var wins int
		var once sync.Once
		var matches sync.WaitGroup

		addWin := func(){
			wins ++
		}

		matches.Add(20)

		for i:= 0; i < 20; i++{
			go func(){
				defer matches.Done()
				once.Do(addWin)
			}
		}

		matches.Wait()
		fmt.Println(wins) //it will print 1
	}
```


### Pool

To avoid creating and garbage collecting lot of heavy objects time over time, we can use `sync.Pool` which is a scalable pool of temporary objects and it is also concurrency-safe. The object pool can shrink and expand based on the load.

**NOTE**: do not copy a pool after its first use.

It has two methods:
- `Get()` to obtain one (random) item from the Pool (it will be removed from the pool and returned to the caller)
- `Put(obj any)` adds the item to the Pool

When creating a `sync.Pool` we can pass an optional function (which we most often want indeed to pass) which will generate a value when we call `Get` instead of returning `nil`.

```go
type City struct {
	Name string
	Population int
}

var myPool = sync.Pool{
	New: func() any{ //pay attention to the function signature here
		fmt.Println("creating new city")
		return &City{}
	}
}

func main(){
	city := myPool.Get().(*City) //pay attention to the type assertion here!
	//putting it back
	myPool.Put(city)
	city.Name = "Turin"
	city.Population = 1000000

	//now using get will return the object we just created because we put it back in the pool
	city2 := myPool.Get().(*City)
	fmt.Printf("%s", city2.Name) // will print "Turin"
	city3 := myPool.Get().(*City) //this will get now a new object because there isn't any left inside the pool
	fmt.Printf("%s", city3.Name) // will print ""
}

```


### Map 

`sync.Map` is a special form of `map[any]any` which has the property of being safe for concurrent usage without using lock or coordinating goroutines. Note that in exchange we lose type safety and speed. In general it is almost always better to use a common `map`

`sync.Map` is optimized for:
- cache-like usages, like when a key is written only once but read many many times
- when we have lot of goroutines writing and overwriting entities inside a map, in which case the performance of `sync.Map` would be greatly better than using a `map` with `Mutex`

**NOTE**: `sync.Map` must NOT be copied after use

It has the methods:
- `Store(key, value any)` sets the value for the key
- `Load(key any)` returns the value stored related to the key, or `nil` 
- `Delete(key any)` deletes the value related to the key
- `LoadAndDelete(key any)` deletes the value for the related key and returns it if present
- `LoadOrStore(key, value any)` returns the existing value related to the key IF present. If not it stores the given value.
- `Range(f func(key, value any) bool)` calls `f` sequentially for each key and value present in the map. If f returns false, range stops the iteration.




## Concurrency Common Patterns


### Generator

We have a *generator* function which returns a sequence of values, or to be more precise, in go, it will return a channel from which we will be capable of getting the produced values. (If you are used to javascript, it is like the `yield` keyword)

```
	_________________________
	|						|	--------> Value1
	|						|	--------> Value2
	|   GENERATOR			|	--------> Value3
	|						|	...
	|						|	--------> ValueN
	|_______________________|

```

```go
func myGenerator() <-chan int{
	ch := make(chan int)

	go func(){
		for  i := 0; i < 5 ; i++{
			ch <- i
		}
	}()

	return ch
}

func main(){
	ch := myGenerator()

	for  i := 0; i < 5 ; i++{
		fmt.Println( <- ch)
	}
}
```



### Fan-in

We combine multiple inputs into a single *output channel*. Usually we don't care about input order
```
	_________________
	|				|		
	|   input1		|	---------|
	|_______________|			 |
								 |
	_________________            |
	|				|			 |
	|   input2		|	---------| 
	|_______________|			 |
								 |				_________________
	_________________            |				|				|
	|				|		     ----------->   |   output		|
	|   input3		|	---------|				|_______________|
	|_______________|			 |
								 |
			...					 |
								 |
	_________________			 |
	|				|			 |
	|   inputN		|	---------|
	|_______________|

```

```go
func produce(values []int) <- chan int{
	ch := make(chan int)

	go func(){
		defer close(ch)

		for _, v := range values{
			ch <- v
		}
	}()

	return ch
}


func fanIn(inputs ... <- chan int) <- chan int{
	var wg sync.WaitGroup
	wg.Add(len(inputs))
	out := make(chan int)

	for _, input := range inputs { //for each input channel

		go func(ch <- chan int){ //this goroutine will wait on the input channels
			for {
				value, ok := <- ch
				if !ok{
					wg.Done()
					break
				}
				out <- value
			}
		}(input)
	}

	go func(){ //this gourite will close the output channel once all input channels are done
		wg.Wait()
		close(out)
	}()

	return out
}


func main(){
	input1 := produce([]int{1,2,3,4})
	input2 := produce([]int{5,6,7,8})
	input3 := produce([]int{9,10,11,12})

	out := fanIn(input1, input2, input3)

	for v := range out{
		fmt.Println(v)
	}
}

```


### Fan-out

We split a single input channel into multiple output channels, very useful when we want to redistribute computations into multiple actors/routines.

```
								_________________
								|				|		
					|---------->|   output1		|	
					|			|_______________|	
					|								
					|			_________________   
					|			|				|	
					|---------->|   output2		|	
					|			|_______________|	
_________________	|
|				|	|
|   input		|---|
|_______________|	|									
					|			_________________   
					|			|				|	
					|---------->|   output3		|	
					|			|_______________|	
					|								
					|					...			
					|								
					|			_________________	
					|			|				|	
					|---------->|   outputN		|	
								|_______________|

```

```go
func produce(values []int) <- chan int{
	ch := make(chan int)

	go func(){
		defer close(ch)

		for _, v := range values{
			ch <- v
		}
	}()

	return ch
}

func fanOut(input <- chan int) <- chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for data := range input {
			out <- data
		}
	}()

	return out
}

func main(){
	data := []int{1,2,3,4,5,6,7,8,9,10}
	input := produce(data)

	out1 := fanOut(input)
	out2 := fanOut(input)
	out3 := fanOut(input)
	out4 := fanOut(input)

	for range data {
		select{
		case value := <- out1:
			fmt.Println("Output 1 got:", value)
		case value := <- out2:
			fmt.Println("Output 2 got:", value)
		case value := <- out3:
			fmt.Println("Output 3 got:", value)
		case value := <- out4:
			fmt.Println("Output 4 got:", value)
		}
	}

}
```


### Pipeline

Very simple: we imagine groups of goroutines and each gruop is running the same function. Each of these groups are connected by channels, receiving data from **upstream**  and sending values **downstream**.

The first group is tipically called the **producer** while the last one is called the **consumer**

```
	_________________					_________________					_________________					_________________
	|				|					|				|					|				|					|				|		
	|   producer	|	--------->		|   stage1		|	--- ... --->	|   stageN		|	--------->		|   consumer	|	
	|_______________|					|_______________|					|_______________|					|_______________|		


```

The benefits of separating in groups in this way are that we make the function performed by each group strongly independent and as a consequence we make it easier to combine those groups in a different way when needed.

```go
func generate(values []int) <- chan int{
	ch := make(chan int)

	go func(){
		defer close(ch)

		for _, v := range values{
			ch <- v
		}
	}()

	return ch
}

func multiply(input <- chan int, multiplier int) <-chan int{
	out := make(chan int)

	go func(){
		defer close(out)

		for v := range input{
			out <- v * multiplier
		}
	}()

	return out
}

func filter(input <-chan int, filterValue int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for v := range input {
			if v > filterValue {
				out <- v
			}
		}
	}()

	return out
}

func main() {
	in := generate([]int{0, 1, 2, 3, 4, 5, 6, 7, 8,9,10})

	out := multiply(in, 2) 
	out = filter(out, 15) 


	for value := range out {
		fmt.Println(value)
	}
}
```



### Worker Pool

A very simple pattern in which we distribute the actions we want to complete over multiple goroutines ( **workers** ) concurrently.

We use a channel to send our actions to the workers, and another one to collect the result from the workers once they are done.

```
								_____________
								|			|		
					|---------->|  worker1	|-----------|
					|			|___________|			|
					|									|
					|			_____________  			|
					|			|			|			|
					|---------->|  worker2	|-----------|
					|			|___________|			|
_________________	|									|	 __________
|				|	|									|	 |		  |	
|   input		|---|									|--->| output |
|_______________|	|									|	 |________|
					|			_____________  			|
					|			|			|			|
					|---------->|  worker3	|-----------|
					|			|___________|			|
					|									|
					|				...					|
					|									|
					|			_____________			|
					|			|			|			|
					|---------->|  workerN	|-----------|
								|___________|

```

```go
func worker(id int, actions <-chan int, results chan<- int) {
	var wg sync.WaitGroup

	for action := range actions {
		wg.Add(1)

		go func(action int) {
			defer wg.Done()

			fmt.Printf("Worker %d started action %d\n", id, action)
			result := action * 10 //just simulating something
			results <- result
			fmt.Printf("Worker %d finished action %d\n", id, action)
		}(action)
	}

	wg.Wait()
}

const totalActions = 8
const totalWorkers = 4

func main() {
	//note here channel are buffered
	actions := make(chan int, totalActions)
	results := make(chan int, totalActions)

	for w := 1; w <= totalWorkers; w++ { //getting workers ready
		go worker(w, actions, results)
	}

	// Send action to the channel so workers will take it
	for action := 1; action <= totalActions; action++ {
		actions <- action
	}

	close(actions)

	// Receive results
	for action := 1; action <= totalActions; action++ {
		<-results
	}

	close(results)
}
```