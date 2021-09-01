https://www.baeldung.com/jackson-deserialization

## **1. Overview**[](https://www.baeldung.com/jackson-deserialization#overview)

This quick tutorial will illustrate how to use Jackson 2 to deserialize JSON using a  **custom Deserializer**.

If you want to dig deeper and learn  **other cool things you can do with the Jackson 2**  – head on over to  [the main Jackson tutorial](https://www.baeldung.com/jackson "The main Jackson Tutorial").

## Further reading:

## [Intro to the Jackson ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)

The article discusses Jackson's central ObjectMapper class, basic serialization and deserialization as well as configuring the two processes.

[Read more](https://www.baeldung.com/jackson-object-mapper-tutorial)  →

## [Jackson – Decide What Fields Get Serialized/Deserialized](https://www.baeldung.com/jackson-field-serializable-deserializable-or-not)

How to control which fields get serialized/deserialized by Jackson and which fields get ignored.

[Read more](https://www.baeldung.com/jackson-field-serializable-deserializable-or-not)  →

## [Jackson – Custom Serializer](https://www.baeldung.com/jackson-custom-serialization)

Control your JSON output with Jackson 2 by using a Custom Serializer.

[Read more](https://www.baeldung.com/jackson-custom-serialization)  →

## **2. Standard Deserialization**[](https://www.baeldung.com/jackson-deserialization#standard)

Let's start by defining 2 entities and see how Jackson will deserialize a JSON representation to these entities without any customization:

```java
public class User {
    public int id;
    public String name;
}
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
```

Now, let's define the JSON representation we want to deserialize:

```javascript
{
    "id": 1,
    "itemName": "theItem",
    "owner": {
        "id": 2,
        "name": "theUser"
    }
}
```

And finally, let's unmarshall this JSON to Java Entities:

```java
Item itemWithOwner = new ObjectMapper().readValue(json, Item.class);
```

## **3. Custom Deserializer on  _ObjectMapper_**[](https://www.baeldung.com/jackson-deserialization#on-mapper)

In the previous example, the JSON representation matched the java entities perfectly – next, we will simplify the JSON:

```javascript
{
    "id": 1,
    "itemName": "theItem",
    "createdBy": 2
}
```

When unmarshalling this to the exact same entities – by default, this will of course fail:

```bash
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: 
Unrecognized field "createdBy" (class org.baeldung.jackson.dtos.Item), 
not marked as ignorable (3 known properties: "id", "owner", "itemName"])
 at [Source: java.io.StringReader@53c7a917; line: 1, column: 43] 
 (through reference chain: org.baeldung.jackson.dtos.Item["createdBy"])
```

We'll solve this by doing  **our own deserialization with a custom Deserializer**:

```java
public class ItemDeserializer extends StdDeserializer<Item> { 

    public ItemDeserializer() { 
        this(null); 
    } 

    public ItemDeserializer(Class<?> vc) { 
        super(vc); 
    }

    @Override
    public Item deserialize(JsonParser jp, DeserializationContext ctxt) 
      throws IOException, JsonProcessingException {
        JsonNode node = jp.getCodec().readTree(jp);
        int id = (Integer) ((IntNode) node.get("id")).numberValue();
        String itemName = node.get("itemName").asText();
        int userId = (Integer) ((IntNode) node.get("createdBy")).numberValue();

        return new Item(id, itemName, new User(userId, null));
    }
}
```

As you can see, the deserializer is working with the standard Jackson representation of JSON – the  _JsonNode_. Once the input JSON is represented as a  _JsonNode_, we can now  **extract the relevant information from it**  and construct our own  _Item_  entity.

Simply put, we need to  **register this custom deserializer**  and simply deserialize the JSON normally:

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addDeserializer(Item.class, new ItemDeserializer());
mapper.registerModule(module);

Item readValue = mapper.readValue(json, Item.class);
```

## **4. Custom Deserializer on the Class**[](https://www.baeldung.com/jackson-deserialization#on-class)

Alternatively, we can also  **register the deserializer directly on the class**:

```java
@JsonDeserialize(using = ItemDeserializer.class)
public class Item {
    ...
}
```

With the deserializer defined at the class level, there is no need to register it on the  _ObjectMapper_  – a default mapper will work fine:

```java
Item itemWithOwner = new ObjectMapper().readValue(json, Item.class);
```

This type of per-class configuration is very useful in situations in which we may not have direct access to the raw  _ObjectMapper_  to configure.

## **5. Conclusion**[](https://www.baeldung.com/jackson-deserialization#conclusion)

This article shows how to leverage Jackson 2 to  **read non-standard JSON input**  – and how to map that input to any java entity graph with full control over the mapping.

The implementation of all these examples and code snippets  **can be found in  [over on GitHub](https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-custom-conversions "Github Project using custom Deserializers")**  – it's a Maven-based project, so it should be easy to import and run as it is.