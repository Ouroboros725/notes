https://stackoverflow.com/questions/30716354/how-do-i-do-a-literal-int64-in-go

The Go Language Specification ([Address operators][1]) does not allow to take the address of a numeric constant (not of an _untyped_ nor of a _typed_ constant).

> The operand must be _addressable_, that is, either a variable, pointer indirection, or slice indexing operation; or a field selector of an addressable struct operand; or an array indexing operation of an addressable array. As an exception to the addressability requirement, `x` [in the expression of `&x`] may also be a (possibly parenthesized) [composite literal][2].

For reasoning why this isn't allowed, see related question: https://stackoverflow.com/questions/35146286/find-address-of-constant-in-go/35146856#35146856. A similar question (similarly not allowed to take its address): https://stackoverflow.com/questions/34197248/how-can-i-store-reference-to-the-result-of-an-operation-in-go/34197367#34197367

Your options (try all on the [Go Playground][3]):

### 1) With `new()`

You can simply use the builtin [`new()`][4] function to allocate a new zero-valued `int64` and get its address:

	instance := SomeType{
		SomeField: new(int64),
	}

But note that this can only be used to allocate and obtain a pointer to the zero value of any type.

### 2) With helper variable

Simplest and recommended for non-zero elements is to use a helper variable whose address can be taken:

	helper := int64(2)
	instance2 := SomeType{
		SomeField: &helper,
	}

### 3) With helper function

Or if you need this many times, you can create a helper function which allocates and returns an `*int64`:

    func create(x int64) *int64 {
    	return &x
    }

And using it:

	instance3 := SomeType{
		SomeField: create(3),
	}

Note that we actually didn't allocate anything, the Go compiler did that when we returned the address of the function argument. The Go compiler performs escape analysis, and allocate local variables on the heap (instead of the stack) if they may escape the function. For details, see https://stackoverflow.com/questions/42530219/is-returning-a-slice-of-a-local-array-in-a-go-function-safe/42530418#42530418

### 4) With a one-liner anonymous function

	instance4 := SomeType{
		SomeField: func() *int64 { i := int64(4); return &i }(),
	}

Or as a (shorter) alternative:

	instance4 := SomeType{
		SomeField: func(i int64) *int64 { return &i }(4),
	}

### 5) With slice literal, indexing and taking address

If you would want `*SomeField` to be other than `0`, then you need something addressable.

You can still do that, but that's ugly:

	instance5 := SomeType{
		SomeField: &[]int64{5}[0],
	}
	fmt.Println(*instance2.SomeField) // Prints 5

What happens here is an `[]int64` slice is created with a literal, having one element (`5`). And it is indexed (0th element) and the address of the 0th element is taken. In the background an array of `[1]int64` will also be allocated and used as the backing array for the slice. So there is a lot of boilerplate here.

### 6) With a helper struct literal

Let's examine the exception to the addressability requirements:

> As an exception to the addressability requirement, `x` [in the expression of `&x`] may also be a (possibly parenthesized) [composite literal][2].

This means that taking the address of a composite literal, e.g. a struct literal is ok. If we do so, we will have the struct value allocated and a pointer obtained to it. But if so, another requirement will become available to us: **"field selector of an addressable struct operand"**. So if the struct literal contains a field of type `int64`, we can also take the address of that field!

Let's see this option in action. We will use this wrapper struct type:

    type intwrapper struct {
    	x int64
    }

And now we can do:

	instance6 := SomeType{
		SomeField: &(&intwrapper{6}).x,
	}

Note that this

    &(&intwrapper{6}).x

means the following:

    & ( (&intwrapper{6}).x )

But we can omit the "outer" parenthesis as the address operator `&` is applied to the result of the [selector expression][5].

Also note that in the background the following will happen (this is also a valid syntax):

    &(*(&intwrapper{6})).x

### 7) With helper anonymous struct literal

The principle is the same as with case #6, but we can also use an anonymous struct literal, so no helper/wrapper struct type definition needed:

	instance7 := SomeType{
		SomeField: &(&struct{ x int64 }{7}).x,
	}

[1]: http://golang.org/ref/spec#Address_operators
[2]: http://golang.org/ref/spec#Composite_literals
[3]: https://play.golang.org/p/d2Hks6mYcr5
[4]: http://golang.org/pkg/builtin/#new
[5]: https://golang.org/ref/spec#Selectors