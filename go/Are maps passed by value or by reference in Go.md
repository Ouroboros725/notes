https://stackoverflow.com/questions/40680981/are-maps-passed-by-value-or-by-reference-in-go

In this thread you will find your answer :

https://stackoverflow.com/questions/28384343/golang-accessing-a-map-using-its-reference


> You don't need to use a pointer with a map.
>
> Map types are reference types, like pointers or slices[1]
>
> If you needed to change the Session you could use a pointer:
>
>     map[string]*Session
>
> https://blog.golang.org/go-maps-in-action
