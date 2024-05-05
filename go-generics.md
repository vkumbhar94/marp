---
marp : true
theme: gaia
#theme: uncover
#_class: lead
class:
  - lead
paginate: true
backgroundColor: black
#color: #ff737d
#color: #174d51
#color: #e3ded7
color: #8ead7c
#backgroundImage: url('https://marp.app/assets/hero-background.svg')
transition: dissolve
style: |
  @keyframes marp-transition-dissolve {
      from {
        opacity: 1;
      }
      to {
        opacity: 0;
      }
  }
  |

#footer: Footer content
---

![bg left:40% 80%](https://go.dev/blog/go-brand/Go-Logo/SVG/Go-Logo_Blue.svg)

# **Golang Generics**

Golang is awesome, isn't it?

Presenter: [Vaibhav Kumbhar](https://github.com/vkumbhar94)
<span style="font-size: 50%;"><span style="color:;">Staff Software engineer</span> at [Logicmonitor](https://www.logicmonitor.com/)</span>

<!--
Cheers to the Golang community!
-->

---

# What is Generics?

- Generics is a way to write functions and data structures that are type-agnostic.
- It allows you to write functions and data structures that can work with any type or set of types.

---

# Why Generics?

- **Code Reusability**: Write code once and use it with any type or set of allowed types.
<!--
- writing the same code for different types is a waste of time and effort.
- library functions to work on different types.
- structs that can hold any type of data without having to use interface{}
-->


--- 
- **Performance**: Avoid type assertions and conversions.
```go
func sum(a, b int64) int {
    return a + b
}
func main(){
    sum(int64(10), int64(20)) // need to convert other to int64
}
```
<!--
- type assertion and conversion are expensive operations.
- avoid type assertion and conversion
-->


- **Type Safety**: Avoid runtime errors due to type mismatch.
```go
func sum(a, b int64) int {
    return a + b
}
func calc(a, b any){
    sum(a, b) // runtime error: hey this is not int64 
}
```
<!--
- type mismatch errors are caught at compile time.
- avoid runtime errors due to type mismatch.
-->


---
## Golang Generics is available since Go 1.18

Before 1.18, interface type defined as method set, now it can defined as type sets.
`error` is builtin interface in go having `Error() string` method.
<!-- 
- Go 1.18 is released on 16th Feb 2022.
- Go 1.18 is the first release to support generics.
-->

---

# Syntax for defining generic functions

defining type parameters in square brackets `[]` with type constraints.
```go
func Add[T any](a, b T) T {
    return a + b
}
```

<!----->
<!-- _class: extra-slide -->


<!--
Let's see demo examples [here]() - add link.
-->
<!--
- go [intro-generics](https://go.dev/blog/intro-generics)
-->

---

## Predefined types in Go 1.18

- `golang.org/x/exp/constraints` package for defining type constraints.
```
- constraints.Integer
- constraints.Float
- constraints.Complex
- constraints.Ordered
- constraints.Signed
- constraints.Unsigned
```

```
- `cmp.Ordered` - also gave few util function using this for ease - `Less`, `Compare`, `isNaN`
- func `cmp.Or` - util functin on `comparable` constraint - returns first non zero value from given var args list 
```
- `cmp` package for defining comparison constraints.

<!--
- a is Less < b
- compare -1,0,1 return value
- isNan - check if value is NaN
-->

---

# Let's learn to define own constraints

---

## Restrictive constraints
<!--
- defining constraints without tilde prefix `int32` (`~int32`) - tilde allows all descendants also - will see in next slide
-->

```go
type OnlyInt32Or64 interface {
	int32 | int64
}
func Add[T OnlyInt32Or64](a, b T) T {
	return a + b
}

type MyInt32 int32
### Usage
// fmt.Println(Add(10,20)) // fails to work as int is not in allowed types
fmt.Println(Add(int32(10), int32(20))) // works
// fmt.Println(Add(MyInt32(10), MyInt32(20))) // doesn't work because constraint is not allowing descendants

```

--- 

## Flexible constraints 
allow all who's underlying type is T

```go
type OnlyInt32Or64 interface {
	~int32 | ~int64
}
func Add[T OnlyInt32Or64](a, b T) T {
	return a + b
}
type MyInt32 int32
### Usage
// fmt.Println(Add(10,20)) // fails to work as int is not in allowed types
fmt.Println(Add(int32(10), int32(20))) // works
fmt.Println(Add(MyInt32(10), MyInt32(20))) // works because constraint allows custom types with underlying type int32

```
---

## Constraint with method sets
```go
type Thread interface {
	Run()
}
func Run[T Thread](t T) {
	t.Run()
}
// you can embed interface type constraints into another interface type constraints
type NamedThread interface{
    Thread
    Id() string
}
```
---

Some more...

```go
// EmptyTypeSet tilde allows no types
// no type can have both int32 as well as float64
// primitive type should be unioned as per your use case
type EmptyTypeSet interface {
	int32
	float64
}
```

--- 

Some more...
Both type set and method set together

```go
// intersection with method sets would be helpful
// all types whos underlying type is int32 and having Run method
type Thread interface {
    ~int32
    Run()
}

```

---

## Let's now learn to define generic functions


--- 

## Util functions

```go
// single type
func Add[T constraints.Integer](a, b T) T {
    return a + b
}
// multiple types
func Append[T string, E constraints.Integer](s T, e E) s {
    return fmt.Sprintf("%s:%d", s, e)
}
// Union types
func Add[T constraints.Integer | constraints.Float](a, b T) T {
    return a + b
}

// Scale returns a copy of s with each element multiplied by c.
func Scale[S ~[]E, E constraints.Integer](s S, c E) S {
    r := make(S, len(s))
    for i, v := range s {
        r[i] = v * c
    }
    return r
}


```

---

## Let's define struct with generics

```go
package main

import (
	"errors"
	"fmt"
)

type Stack[T any] struct {
	data []T
}

func (s *Stack[T]) Push(t T) {
	s.data = append(s.data, t)
}

var s Stack // error: You must instantiate generic type with a type.
var s Stack[int] // works


```

<!--
package main

import (
	"errors"
	"fmt"
)

type Stack[T any] struct {
	data []T
}

func (s *Stack[T]) Push(t T) {
	s.data = append(s.data, t)
}

func (s *Stack[T]) Pop() (T, error) {
	if len(s.data) == 0 {
		var zero T
		return zero, errors.New("empty stack")
	}
	t := s.data[len(s.data)-1]
	s.data = s.data[:len(s.data)-1]
	return t, nil
}

func (s *Stack[T]) Peek() (T, error) {
	if len(s.data) == 0 {
		var zero T
		return zero, errors.New("empty stack")
	}
	return s.data[len(s.data)-1], nil
}

func main() {
	/**
	You must instantiate generic type with a type.
	You cannot do like `var s Stack`
	**/
	var s Stack[int]
	s.Push(1)
	s.Push(2)
	s.Push(3)
	fmt.Println(s.Peek())
	fmt.Println(s.Pop())
	fmt.Println(s.Pop())
	fmt.Println(s.Pop())
	fmt.Println(s.Pop())

}

-->
---

#### Thank you for taking the time to listen to me!

- [examples]()

---
