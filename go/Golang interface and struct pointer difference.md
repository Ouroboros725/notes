https://stackoverflow.com/questions/44370277/type-is-pointer-to-interface-not-interface-confusion

So you're confusing two concepts here.  A pointer to a struct and a pointer to an interface are not the same.  An interface can store either a struct directly _or_ a pointer to a struct.  In the latter case, you still just use the interface directly, _not_ a pointer to the interface.  For example:

    type Fooer interface {
    	Dummy()
    }
    
    type Foo struct{}
    
    func (f Foo) Dummy() {}
    
    func main() {
    	var f1 Foo
    	var f2 *Foo = &Foo{}
    
    	DoFoo(f1)
    	DoFoo(f2)
    }
    
    func DoFoo(f Fooer) {
    	fmt.Printf("[%T] %+v\n", f, f)
    }

Output:

    [main.Foo] {}
    [*main.Foo] &{}

https://play.golang.org/p/I7H_pv5H3Xl

In both cases, the `f` variable in `DoFoo` is just an interface, _not_ a pointer to an interface.  However, when storing `f2`, the interface _holds_ a pointer to a `Foo` structure.

Pointers to interfaces are almost _never_ useful.  In fact, the Go runtime was specifically changed a few versions back to no longer automatically dereference interface pointers (like it does for structure pointers), to discourage their use.  In the overwhelming majority of cases, a pointer to an interface reflects a misunderstanding of how interfaces are supposed to work.

However, there is a limitation on interfaces.  If you pass a structure directly into an interface, only _value_ methods of that type (ie. `func (f Foo) Dummy()`, not `func (f *Foo) Dummy()`) can be used to fulfill the interface.  This is because you're storing a copy of the original structure in the interface, so pointer methods would have unexpected effects (ie. unable to alter the original structure).  Thus the default rule of thumb is to **store pointers to structures in interfaces**, unless there's a compelling reason not to.

Specifically with your code, if you change the AddFilter function signature to:

    func (fp *FilterMap) AddFilter(f FilterInterface) uuid.UUID

And the GetFilterByID signature to:

    func (fp *FilterMap) GetFilterByID(i uuid.UUID) FilterInterface

Your code will work as expected.  `fieldfilter` is of type `*FieldFilter`, which fullfills the `FilterInterface` interface type, and thus `AddFilter` will accept it.

Here's a couple of good references for understanding how methods, types, and interfaces work and integrate with each other in Go:

* https://medium.com/@agileseeker/go-interfaces-pointers-4d1d98d5c9c6
* https://www.goinggo.net/2014/05/methods-interfaces-and-embedded-types.html
* https://blog.golang.org/laws-of-reflection