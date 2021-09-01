https://stackoverflow.com/questions/41239913/why-additionalproperties-is-the-way-to-represent-dictionary-map-in-swagger-ope


First thing, I found a [better explanation for `additionalProperties`][1]:

> For an object, if this is given, in addition to the properties defined in `properties` all other property names are allowed. Their values must each match the schema object given here. If this is not given, no other properties than those defined in `properties` are allowed.

So here is how I finally understood this:

Using `properties`, we can define a known set of properties similar to [Python's namedtuple][2], however if we wish to have something more like [Python's dict][3], or any other hash/map where we can't specify how many keys there are nor what they are in advance, we should use `additionalProperties`.

`additionalProperties` will match any property name (that will act as the `dict`'s key, and the `$ref` or `type` will be the schema of the `dict`'s value, and since there should not be more than one properties with the same name for every given object, we will get the enforcement of unique keys.

Note that unlike Python's `dict` that accepts any immutable value as a key, since the keys here are in essence property names, they must be strings. (Thanks [Ted Epstein][4] for that clarification). This limitation can be tracked down to `pair := string : value` in the [json specification][5].

[1]: https://github.com/OAI/OpenAPI-Specification/issues/668#issuecomment-218829120
[2]: https://docs.python.org/3/library/collections.html#collections.namedtuple
[3]: https://docs.python.org/3/library/stdtypes.html?highlight=dict#dict
[4]: https://stackoverflow.com/users/776186/
[5]: http://www.json.org/

---

Chen, I think [your answer][1] is correct.

Some further background that might be helpful:

In JavaScript, which was the original context for JSON, an object is like a hash map of strings to values, where some values are data, others are functions. You can think of each name-value pair as a property. But JavaScript doesn't have classes, so the property names are not predefined, and each object can have its own independent set of properties.

JSON Schema uses the `properties` keyword to validate name-value pairs that are known in advance; and uses `additionalProperties` (or `patternProperties`, not supported in OpenAPI 2.0) to validate properties that are not known.

For clarity:

* The property names, or "keys" in the map, must be strings. They cannot be numbers, or any other value.
* As you said, the property names _should_ be unique. Unfortunately the JSON spec doesn't strictly require uniqueness, but uniqueness is recommended, and expected by most JSON implementations.  More background [here][2].
* `properties` and `additionalProperties` can be used alone or in combination.  When additionalProperties is used alone, without properties, the object essentially functions as a `map<string, T>` where T is the type described in the additionalProperties sub-schema. Maybe that helps to answer your original question.
* When evaluating an object against a single schema, if a property name matches one of those specified in `properties`, its value only needs to be valid against the sub-schema provided for that property. The `additionalProperties` sub-schema, if provided, will only be used to validate properties that _are not_ included in the `properties` map.
* There are some limitations of `additionalProperties` as implemented in Swagger's core Java libraries. I've documented these limitations [here][3].


[1]: https://stackoverflow.com/a/41240118/776186
[2]: https://stackoverflow.com/questions/21832701/does-json-syntax-allow-duplicate-keys-in-an-object?lq=1
[3]: https://support.reprezen.com/support/solutions/articles/6000162892-support-for-additionalproperties-in-swagger-2-0-schemas