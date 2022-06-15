## 1 Definition of slice in go language

Slice is a structure type, which is defined in the source code as:

> src/runtime/slice.go

```go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```

As can be seen from the definition, slice is a value type with three [element](https://developpaper.com/tag/element/ "View all posts in element")s. Array is an array pointer, which points to the array allocated at the [bottom](https://developpaper.com/tag/bottom/ "View all posts in bottom"); len is the number of elements in the bottom array; cap is the [capacity](https://developpaper.com/tag/capacity/ "View all posts in capacity") of the bottom array, which will be expanded if it exceeds the capacity.

## 2 Initialization operation

There are three initialization operations for slice. See the following code:

```go
package main
import "fmt"
func main() {
    //1、make
    a: = make([] int32, 0, 5)
    //2、[]int32{}
    b: = [] int32 {
            1, 2, 3
        }
    //3、new([]int32)
    c: = * new([] int32)
    fmt.Println(a, b, c)
}
```

These initialization methods are different in the underlying implementation. A good way to understand the underlying implementation is to look at the disassembly call function. Run the following command to see the disassembly of a line of code:

`go tool compile -S plan9 Test.go | grep plan9 Test.go : line number`

### 1\. Make initialization

The make function initialization has three parameters. The first is the type, the second length, and the third capacity. The capacity should be greater than or equal to the length. The make initialization of slice calls the underlying`runtime.makeslice`Function.

```go
func makeslice(et * _type, len, cap int) slice {
    // NOTE: The len > maxElements check here is not strictly necessary,
    // but it produces a 'len out of range' error instead of a 'cap out of range' error
    // when someone does make([]T, bignumber). 'cap out of range' is true too,
    // but since the cap is only being supplied implicitly, saying len is clearer.
    // See issue 4085.
    maxElements: = maxSliceCap(et.size)
    if len < 0 || uintptr(len) > maxElements {
        panic(errorString("makeslice: len out of range"))
    }

    if cap < len || uintptr(cap) > maxElements {
        panic(errorString("makeslice: cap out of range"))
    }

    p: = mallocgc(et.size * uintptr(cap), et, true)
    return slice {
        p, len, cap
    }
}
```

It is mainly called`mallocgc`Allocate a block of memory of cap \* type size to the underlying array, and then return a slice. The array pointer of slice points to the allocated bottom array.

### 2\. \[\] int32 {} initialization

The underlying initialization is the call`runtime.newobject`Function directly allocates the corresponding number of underlying arrays.

```go
// implementation of new builtin
// compiler (both frontend and SSA backend) knows the signature
// of this function
func newobject(typ * _type) unsafe.Pointer {
    return mallocgc(typ.size, typ, true)
}
```

### 3\. New (\[\] int32) initialization

This initialization layer is also called`runtime.newobject`, new is the [address](https://developpaper.com/tag/address/ "View all posts in address") of the returned slice, so the content in the address is the real slice.

## 3 Resclice (slice operation)

The so-called resellice is to create a new slice object based on an existing slice, so that the**Within the allowable range of capacity cap**Adjust the properties.

```go
data := []int32{0,1,2,3,4,5,6}
slice := data[1:4:5]  // [low:high:max]
```

The slicing operation has three parameters, low, high and Max, and three parameters of the newly generated slice structure. The pointer array points to the position where the subscript of the underlying array element of the original slice is low, len = high – low, cap = max – low. As shown in the figure below:

![reslice](reslice.png)

**The main thing to pay attention to in slicing operation is that it is within the allowable range of the original slice capacity. If it exceeds the capacity range, a panic will be reported**。

## 4 Append operation

The append operation of slice is to add data to the end of the underlying array and return**New slice object**。

Take a look at the following code:

```go
package main
import (
    "fmt"
)
func main() {
    a: = make([] int32, 1, 2)
    b: = append(a, 1)
    c: = append(a, 1, 2)
    fmt.Printf("A's address is% P, first element address is% P, capacity is% v / N", & A, & A[0], cap(a))
    //The address of a: 0xc4200a060, the address of the first element: 0xc4201a090, capacity: 2
    fmt.Printf("address of B is% P, first element address is% P, capacity is% v / N", & B, & B[0], cap(b))
    //The address of B: 0xc4200a080, the address of the first element: 0xc4201a090, capacity: 2
    fmt.Printf("address of C is% P, first element address is% P, capacity is% v / N", & C, & C[0], cap(c))
    //The address of C: 0xc4200a0a0a0, the address of the first element: 0xc4201a0a0, capacity: 4
}
```

From the print result of the above code, we can see that a is a bottom array with an element and a slice with a capacity of 2; after appending one element, the capacity does not exceed the capacity, and a new slice B is generated. The first element of a and B bottom array have the same address, indicating that a and B have the same address, B shares the bottom array; append 2 elements, which exceeds the capacity, and generates a new slice C. the address of the bottom array of C is changed, and the capacity is doubled.  
Then it is concluded that the operation process of the append operation is as follows:  
1\. If the added data does not exceed the original capacity, the new slice object and the original slice share the underlying array, the len data will change, and the cap data will not change;  
2\. If the capacity is exceeded after adding data, the capacity will be expanded. A new bottom array will be reallocated, and then the data of the underlying array will be copied. Then the new slice object generated after append has no relationship with the original slice.

### Expansion mechanism

You can see from the assembly code that the expansion calls the underlying function`runtime.growslice`。  
This function is defined as follows:  
`func growslice(et *_type, old slice, cap int) slice {}`  
This function passes in three parameters: the original type of slice, the original slice, and the expected minimum capacity; it returns a new slice, which at least has the expected minimum capacity, and the element comes from the original slice copy.

The expansion rule is mainly the following code:

```go
newcap := old.cap
doublecap: = newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    if old.len < 1024 {
        newcap = doublecap
    } else {
        for newcap < cap {
            newcap += newcap / 4
        }
    }
}
```

The expansion rules are two points:

1.  If the expected minimum capacity is more than twice the original capacity, then the new capacity is equal to the expected minimum capacity;
2.  If the first case is not satisfied, judge whether the length of the underlying array element of the original slice is less than 1024. If it is less than 1024, the new capacity is twice the original capacity; if it is greater than or equal to 1024, the new capacity is 1.25 times of the original capacity.

The above is the basic rule judgment of capacity expansion. In fact, the memory alignment should be taken into account in the expansion

```go
var lenmem, newlenmem, capmem uintptr
const ptrSize = unsafe.Sizeof(( * byte)(nil))
switch et.size {
    case 1:
        lenmem = uintptr(old.len)
        newlenmem = uintptr(cap)
        capmem = roundupsize(uintptr(newcap))
        newcap = int(capmem)
    case ptrSize:
        lenmem = uintptr(old.len) * ptrSize
        newlenmem = uintptr(cap) * ptrSize
        capmem = roundupsize(uintptr(newcap) * ptrSize)
        newcap = int(capmem / ptrSize)
    default:
        lenmem = uintptr(old.len) * et.size
        newlenmem = uintptr(cap) * et.size
        capmem = roundupsize(uintptr(newcap) * et.size)
        newcap = int(capmem / et.size)
}
```

```
After the memory alignment, the expansion multiple will be > = 2 or 1.25.
```

### Why memory alignment?

```
1. Platform reason (migration reason): not all hardware platforms can access any data at any address; some hardware platforms can only get some specific types of data at certain addresses, otherwise a hardware exception will be thrown.
2. Performance reasons: data structures (especially stacks) should be aligned on natural boundaries as much as possible. The reason is that in order to access the misaligned memory, the processor needs to make two memory accesses, while the aligned memory access needs only one access.
```

## 5 The interaction between parameter and parameter in function call

### 1\. Pass the slice value, and directly operate the underlying array in the calling function

Take a look at the following code:

```go
package main
import (
    "fmt"
)

func OpSlice(b[] int32) {
    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(b), cap(b), b)
        //len: 5, cap: 5, data:[1 2 3 4 5]
    fmt.Printf("bfirst element address% p-n", & B[0])
        //B first element address: 0xc42016120

    b[0] = 100
    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(b), cap(b), b)
        //len: 5, cap: 5, data:[100 2 3 4 5]
    fmt.Printf("bfirst element address% p-n", & B[0])
        //B first element address: 0xc42016120
}

func main() {
    a: = [] int32 {
        1, 2, 3, 4, 5
    }
    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(a), cap(a), a)
    //len: 5, cap: 5, data:[1 2 3 4 5]
    fmt.Printf("a first element address% p-n", & A[0])
    //A first element address: 0xc42016120
    OpSlice(a)

    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(a), cap(a), a)
    //len: 5, cap: 5, data:[100 2 3 4 5]
    fmt.Printf("a first element address% p-n", & A[0])
    //A first element address: 0xc42016120
}
```

From the printing of this code, we can see that:  
Slice a in the main function is an actual parameter. When the value is passed to the calling function, a copy should be temporarily copied to B. therefore, the addresses of a and B are different. The three elements in slice B structure are copies of a, but the element array is a pointer, and the copy of the pointer is still a pointer. They point to the same underlying array. Therefore, the address of the first element in the bottom array of a and B is the same. a. B shares the same underlying array. In the call function, the content of the first element of B is changed directly. After the function returns, the first element of a also changes, which is equivalent to changing the argument.

### 2\. Slice pointer passing

There is nothing to say about slice pointer passing. In the called function, it is equivalent to operating the same slice in the argument, and all changes will be reflected in the argument.

### 3\. Slice slice transfer

#### If there is no capacity expansion, take a look at the following code:

```go
package main
import (
    "fmt"
)
func OpSlice(b[] int32) {
    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(b), cap(b), b)
        //len: 3, cap: 9, data:[1 2 3]
    fmt.Printf("bfirst element address% p-n", & B[0])
        //B first element address: 0xc42007a064

    b = append(b, 100)
    fmt.Printf("len: %d, cap: %d, data:%+v \n", len(b), cap(b), b)
        //len: 4, cap: 9, data:[1 2 3 100]
    fmt.Printf("bfirst element address% p-n", & B[0])
        //B first element address: 0xc42007a064

}

func main() {
        a: = [] int32 {
            0, 1, 2, 3, 4, 5, 6, 7, 8, 9
        }
        fmt.Printf("address of 2nd element of"
            Aaddress of 2 nd element % p - n ", & A [1])
            //A address of the second element: 0xc42007a064

            fmt.Printf("len: %d, cap: %d, data:%+v \n", len(a), cap(a), a)
            //len: 10, cap: 10, data:[0 1 2 3 4 5 6 7 8 9]
            fmt.Printf("a first element address% p-n", & A[0])
            //A first element address: 0xc42007a060
            OpSlice(a[1: 4])

            fmt.Printf("len: %d, cap: %d, data:%+v \n", len(a), cap(a), a)
            //len: 10, cap: 10, data:[0 1 2 3 100 5 6 7 8 9]
            fmt.Printf("a first element address% p-n", & A[0])
            //A first element address: 0xc42007a060

        }
```

As mentioned earlier, slices and original slices share the underlying array. In the case of no capacity expansion, for the new slice append operation generated by the slice, the newly added elements will be added to the end of the bottom array, which will cover the original value and reflect it to the original slice;

### summary

No matter what operation of slice: copy, append, reselice and so on, new slices will be generated, but they share the underlying array. Without capacity expansion, their addition, deletion and modification of elements will affect the original slice bottom array; in the case of expansion, a “true” new slice object will be generated, which is completely independent of the original one, and the underlying array will not be affected at all.

## reference material

1.  Deep decryption of go language slice
2.  Go language learning notes




#go #cs 