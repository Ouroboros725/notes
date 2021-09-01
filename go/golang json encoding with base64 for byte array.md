If the value of the map must be base64 encoded, why is this []byte rather than string? Why the base64 rule at all ?

http://golang.org/pkg/encoding/json/

```Array and slice values encode as JSON arrays, except that `[]byte` encodes as a base64-encoded string, and a nil slice encodes as the null JSON object.```

https://github.com/kubernetes/kubernetes/pull/4514#discussion_r25120217