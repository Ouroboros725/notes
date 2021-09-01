https://www.baeldung.com/jackson-annotations

## **1. Overview**[](https://www.baeldung.com/jackson-annotations#overview)

In this tutorial, we'll do a deep dive into  **Jackson Annotations**.

We'll see how to use the existing annotations, how to create custom ones, and finally, how to disable them.

## Further reading:

## [More Jackson Annotations](https://www.baeldung.com/jackson-advanced-annotations)

This article covers some lesser-known JSON processing annotations provided by Jackson.

[Read more](https://www.baeldung.com/jackson-advanced-annotations)  →

## [Jackson – Bidirectional Relationships](https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion)

How to use Jackson to break the infinite recursion problem on bidirectional relationships.

[Read more](https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion)  →

## [Getting Started with Custom Deserialization in Jackson](https://www.baeldung.com/jackson-deserialization)

Use Jackson to map custom JSON to any java entity graph with full control over the deserialization process.

[Read more](https://www.baeldung.com/jackson-deserialization)  →

## **2. Jackson Serialization Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-serialization-annotations)

First, we'll take a look at the serialization annotations.

### **2.1.  _@JsonAnyGetter_**[](https://www.baeldung.com/jackson-annotations#1-jsonanygetter)

The _@JsonAnyGetter_  annotation allows for the flexibility of using a  _Map_  field as standard properties.

For example, the  _ExtendableBean_  entity has the  _name_  property and a set of extendable attributes in the form of key/value pairs:

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
}
```

When we serialize an instance of this entity, we get all the key-values in the  _Map_  as standard, plain properties:

```bash
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

Here's how the serialization of this entity looks in practice:

```java
@Test
public void whenSerializingUsingJsonAnyGetter_thenCorrect()
  throws JsonProcessingException {
 
    ExtendableBean bean = new ExtendableBean("My bean");
    bean.add("attr1", "val1");
    bean.add("attr2", "val2");

    String result = new ObjectMapper().writeValueAsString(bean);
 
    assertThat(result, containsString("attr1"));
    assertThat(result, containsString("val1"));
}
```

We can also use the optional argument  _enabled_  as  _false_  to disable  _@JsonAnyGetter()._  In this case, the  _Map_  will be converted as JSON and will appear under the  _properties_  variable after serialization.

### **2.2.  _@JsonGetter_**[](https://www.baeldung.com/jackson-annotations#2-jsongetter)

**The _@JsonGetter_  annotation is an alternative to the  _@JsonProperty_  annotation, which marks a method as a getter method.**

In the following example, we specify the method  _getTheName()_  as the getter method of the  _name_  property of a  _MyBean_  entity:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

Here's how this works in practice:

```java
@Test
public void whenSerializingUsingJsonGetter_thenCorrect()
  throws JsonProcessingException {
 
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
 
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
}
```

### **2.3.  _@JsonPropertyOrder_**[](https://www.baeldung.com/jackson-annotations#3-jsonpropertyorder)

We can use the  _@JsonPropertyOrder_  annotation to specify  **the order of properties on serialization**.

Let's set a custom order for the properties of a  _MyBean_  entity:

```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

Here's the output of serialization:

```bash
{
    "name":"My bean",
    "id":1
}
```

Then we can do a simple test:

```java
@Test
public void whenSerializingUsingJsonPropertyOrder_thenCorrect()
  throws JsonProcessingException {
 
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
}
```

We can also use  _@JsonPropertyOrder(alphabetic=true)_  to order the properties alphabetically. In that case, the output of serialization will be:

```bash
{
    "id":1,
    "name":"My bean"
}
```

### **2.4.  _@JsonRawValue_**[](https://www.baeldung.com/jackson-annotations#4-jsonrawvalue)

The _@JsonRawValue_  annotation can  **instruct Jackson to serialize a property exactly as is**.

In the following example, we use  _@JsonRawValue_  to embed some custom JSON as a value of an entity:

```java
public class RawBean {
    public String name;

    @JsonRawValue
    public String json;
}
```

The output of serializing the entity is:

```java
{
    "name":"My bean",
    "json":{
        "attr":false
    }
}
```

Next here's a simple test:

```java
@Test
public void whenSerializingUsingJsonRawValue_thenCorrect()
  throws JsonProcessingException {
 
    RawBean bean = new RawBean("My bean", "{\"attr\":false}");

    String result = new ObjectMapper().writeValueAsString(bean);
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("{\"attr\":false}"));
}
```

We can also use the optional boolean argument  _value_  that defines whether this annotation is active or not.

### **2.5.  _@JsonValue_**[](https://www.baeldung.com/jackson-annotations#5-jsonvalue)

_@JsonValue_  indicates a single method that the library will use to serialize the entire instance.

For example, in an enum, we annotate the  _getName_  with  _@JsonValue_ so that any such entity is serialized via its name:

```java
public enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type 2");

    private Integer id;
    private String name;

    // standard constructors

    @JsonValue
    public String getName() {
        return name;
    }
}
```

Now here's our test:

```java
@Test
public void whenSerializingUsingJsonValue_thenCorrect()
  throws JsonParseException, IOException {
 
    String enumAsString = new ObjectMapper()
      .writeValueAsString(TypeEnumWithValue.TYPE1);

    assertThat(enumAsString, is(""Type A""));
}
```

### **2.6.  _@JsonRootName_**[](https://www.baeldung.com/jackson-annotations#6-jsonrootname)

The  _@JsonRootName_  annotation is used, if wrapping is enabled, to specify the name of the root wrapper to be used.

Wrapping means that instead of serializing a  _User_  to something like:

```javascript
{
    "id": 1,
    "name": "John"
}
```

It's going to be wrapped like this:

```javascript
{
    "User": {
        "id": 1,
        "name": "John"
    }
}
```

So let's look at an example.  **W****e're going to use the  _@JsonRootName_ annotation to indicate the name of this potential wrapper entity**:

```java
@JsonRootName(value = "user")
public class UserWithRoot {
    public int id;
    public String name;
}
```

By default, the name of the wrapper would be the name of the class –  _UserWithRoot_. By using the annotation, we get the cleaner looking  _user:_

```java
@Test
public void whenSerializingUsingJsonRootName_thenCorrect()
  throws JsonProcessingException {
 
    UserWithRoot user = new User(1, "John");

    ObjectMapper mapper = new ObjectMapper();
    mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
    String result = mapper.writeValueAsString(user);

    assertThat(result, containsString("John"));
    assertThat(result, containsString("user"));
}
```

Here's the output of serialization:

```java
{
    "user":{
        "id":1,
        "name":"John"
    }
}
```

Since Jackson 2.4, a new optional argument  _namespace_  is available to use with data formats such as XML. If we add it, it will become part of the fully qualified name:

```java
@JsonRootName(value = "user", namespace="users")
public class UserWithRootNamespace {
    public int id;
    public String name;

    // ...
}
```

If we serialize it with  _XmlMapper,_  the output will be:

```xml
<user xmlns="users">
    <id xmlns="">1</id>
    <name xmlns="">John</name>
    <items xmlns=""/>
</user>
```

### **2.7.  _@JsonSerialize_**[](https://www.baeldung.com/jackson-annotations#7-jsonserialize)

**_@JsonSerialize_ indicates a custom serializer to use when** marshalling **the entity.**

Let's look at a quick example. We're going to use  _@JsonSerialize_  to serialize the  _eventDate_  property with a  _CustomDateSerializer_:

```java
public class EventWithSerializer {
    public String name;

    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```

Here's the simple custom Jackson serializer:

```java
public class CustomDateSerializer extends StdSerializer<Date> {

    private static SimpleDateFormat formatter 
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateSerializer() { 
        this(null); 
    } 

    public CustomDateSerializer(Class<Date> t) {
        super(t); 
    }

    @Override
    public void serialize(
      Date value, JsonGenerator gen, SerializerProvider arg2) 
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

Now let's use these in a test:

```java
@Test
public void whenSerializingUsingJsonSerialize_thenCorrect()
  throws JsonProcessingException, ParseException {
 
    SimpleDateFormat df
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithSerializer event = new EventWithSerializer("party", date);

    String result = new ObjectMapper().writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

## **3. Jackson Deserialization Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-deserialization-annotations)

Next let's explore the Jackson deserialization annotations.

### **3.1.  _@JsonCreator_**[](https://www.baeldung.com/jackson-annotations#1-jsoncreator)

**We can use the _@JsonCreator_  annotation to tune the constructor/factory used in deserialization.**

It's very useful when we need to deserialize some JSON that doesn't exactly match the target entity we need to get.

Let's look at an example. Say we need to deserialize the following JSON:

```bash
{
    "id":1,
    "theName":"My bean"
}
```

However, there is no  _theName_  field in our target entity, there is only a  _name_  field. Now we don't want to change the entity itself, we just need a little more control over the unmarshalling process by annotating the constructor with  _@JsonCreator,_  and using the  _@JsonProperty_  annotation as well:

```java
public class BeanWithCreator {
    public int id;
    public String name;

    @JsonCreator
    public BeanWithCreator(
      @JsonProperty("id") int id, 
      @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}
```

Let's see this in action:

```java
@Test
public void whenDeserializingUsingJsonCreator_thenCorrect()
  throws IOException {
 
    String json = "{\"id\":1,\"theName\":\"My bean\"}";

    BeanWithCreator bean = new ObjectMapper()
      .readerFor(BeanWithCreator.class)
      .readValue(json);
    assertEquals("My bean", bean.name);
}
```

### **3.2.  _@JacksonInject_**[](https://www.baeldung.com/jackson-annotations#2-jacksoninject)

**_@JacksonInject_  indicates that a property will get its value from the injection and not from the JSON data.**

In the following example, we use  _@JacksonInject_ to inject the property  _id_:

```java
public class BeanWithInject {
    @JacksonInject
    public int id;
    
    public String name;
}
```

Here's how it works:

```java
@Test
public void whenDeserializingUsingJsonInject_thenCorrect()
  throws IOException {
 
    String json = "{\"name\":\"My bean\"}";
    
    InjectableValues inject = new InjectableValues.Std()
      .addValue(int.class, 1);
    BeanWithInject bean = new ObjectMapper().reader(inject)
      .forType(BeanWithInject.class)
      .readValue(json);
    
    assertEquals("My bean", bean.name);
    assertEquals(1, bean.id);
}
```

### **3.3.  _@JsonAnySetter_**[](https://www.baeldung.com/jackson-annotations#3-jsonanysetter)

_@JsonAnySetter_  allows us the flexibility of using a  _Map_  as standard properties. On deserialization, the properties from JSON will simply be added to the map.

First, we'll use  _@JsonAnySetter_ to deserialize the entity  _ExtendableBean_:

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnySetter
    public void add(String key, String value) {
        properties.put(key, value);
    }
}
```

This is the JSON we need to deserialize:

```bash
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

Then here's how it all ties in together:

```java
@Test
public void whenDeserializingUsingJsonAnySetter_thenCorrect()
  throws IOException {
    String json
      = "{\"name\":\"My bean\",\"attr2\":\"val2\",\"attr1\":\"val1\"}";

    ExtendableBean bean = new ObjectMapper()
      .readerFor(ExtendableBean.class)
      .readValue(json);
    
    assertEquals("My bean", bean.name);
    assertEquals("val2", bean.getProperties().get("attr2"));
}
```

### **3.4.  _@JsonSetter_**[](https://www.baeldung.com/jackson-annotations#4-jsonsetter)

_@JsonSetter_  is an alternative to  _@JsonProperty_  that marks the method as a setter method.

This is incredibly useful when we need to read some JSON data, but  **the target entity class doesn't exactly match that data**, and so we need to tune the process to make it fit.

In the following example, we'll specify the method s_etTheName()_  as the setter of the  _name_  property in our  _MyBean_  entity:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonSetter("name")
    public void setTheName(String name) {
        this.name = name;
    }
}
```

Now when we need to unmarshall some JSON data, this works perfectly well:

```java
@Test
public void whenDeserializingUsingJsonSetter_thenCorrect()
  throws IOException {
 
    String json = "{\"id\":1,\"name\":\"My bean\"}";

    MyBean bean = new ObjectMapper()
      .readerFor(MyBean.class)
      .readValue(json);
    assertEquals("My bean", bean.getTheName());
}
```

### **3.5.  _@JsonDeserialize_**[](https://www.baeldung.com/jackson-annotations#5-jsondeserialize)

**_@JsonDeserialize_  indicates the use of a custom deserializer.**

First, we'll use  _@JsonDeserialize_  to deserialize the  _eventDate_  property with the  _CustomDateDeserializer_:

```java
public class EventWithSerializer {
    public String name;

    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
```

Here's the custom deserializer:

```java
public class CustomDateDeserializer
  extends StdDeserializer<Date> {

    private static SimpleDateFormat formatter
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateDeserializer() { 
        this(null); 
    } 

    public CustomDateDeserializer(Class<?> vc) { 
        super(vc); 
    }

    @Override
    public Date deserialize(
      JsonParser jsonparser, DeserializationContext context) 
      throws IOException {
        
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Next here's the back-to-back test:

```java
@Test
public void whenDeserializingUsingJsonDeserialize_thenCorrect()
  throws IOException {
 
    String json
      = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";

    SimpleDateFormat df
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    EventWithSerializer event = new ObjectMapper()
      .readerFor(EventWithSerializer.class)
      .readValue(json);
    
    assertEquals(
      "20-12-2014 02:30:00", df.format(event.eventDate));
}
```

### 3.6.  _@JsonAlias_[](https://www.baeldung.com/jackson-annotations#6-jsonalias)

The _@JsonAlias_ defines  **one or more alternative names for a property during deserialization**.

Let's see how this annotation works with a quick example:

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

Here we have a POJO, and we want to deserialize JSON with values such as  _fName_,  _f_name_, and  _firstName_  into the  _firstName_  variable of the POJO.

Below is a test making sure this annotation works as expected:

```java
@Test
public void whenDeserializingUsingJsonAlias_thenCorrect() throws IOException {
    String json = "{\"fName\": \"John\", \"lastName\": \"Green\"}";
    AliasBean aliasBean = new ObjectMapper().readerFor(AliasBean.class).readValue(json);
    assertEquals("John", aliasBean.getFirstName());
}
```

## **4. Jackson Property Inclusion Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-property-inclusion-annotations)

### **4.1.  _@JsonIgnoreProperties_**[](https://www.baeldung.com/jackson-annotations#1-jsonignoreproperties)

**_@JsonIgnoreProperties_  is a class-level annotation that marks a property or a list of properties that Jackson will ignore.**

Let's look at a quick example ignoring the property  _id_  from serialization:

```java
@JsonIgnoreProperties({ "id" })
public class BeanWithIgnore {
    public int id;
    public String name;
}
```

Now here's the test making sure the ignore happens:

```java
@Test
public void whenSerializingUsingJsonIgnoreProperties_thenCorrect()
  throws JsonProcessingException {
 
    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

To ignore any unknown properties in JSON input without exception, we can set  _ignoreUnknown=true_  of  _@JsonIgnoreProperties_ annotation.

### **4.2.  _@JsonIgnore_**[](https://www.baeldung.com/jackson-annotations#2-jsonignore)

**In contrast, the _@JsonIgnore_  annotation is used to mark a property to be ignored at the field level.**

Let's use  _@JsonIgnore_  to ignore the property  _id_  from serialization:

```java
public class BeanWithIgnore {
    @JsonIgnore
    public int id;

    public String name;
}
```

Then we'll test to make sure that  _id_  was successfully ignored:

```java
@Test
public void whenSerializingUsingJsonIgnore_thenCorrect()
  throws JsonProcessingException {
 
    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

### **4.3.  _@JsonIgnoreType_**[](https://www.baeldung.com/jackson-annotations#3-jsonignoretype)

**_@JsonIgnoreType_  marks all properties of an annotated type to be ignored.**

We can use the annotation to mark all properties of type  _Name_ to be ignored:

```java
public class User {
    public int id;
    public Name name;

    @JsonIgnoreType
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

We can also test to ensure the ignore works correctly:

```java
@Test
public void whenSerializingUsingJsonIgnoreType_thenCorrect()
  throws JsonProcessingException, ParseException {
 
    User.Name name = new User.Name("John", "Doe");
    User user = new User(1, name);

    String result = new ObjectMapper()
      .writeValueAsString(user);

    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
    assertThat(result, not(containsString("John")));
}
```

### **4.4.  _@JsonInclude_**[](https://www.baeldung.com/jackson-annotations#4-jsoninclude)

**We can use  _@JsonInclude_ to exclude properties with empty/null/default values.**

Let's look at an example excluding nulls from serialization:

```java
@JsonInclude(Include.NON_NULL)
public class MyBean {
    public int id;
    public String name;
}
```

Here's the full test:

```java
public void whenSerializingUsingJsonInclude_thenCorrect()
  throws JsonProcessingException {
 
    MyBean bean = new MyBean(1, null);

    String result = new ObjectMapper()
      .writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
}
```

### **4.5.  _@JsonAutoDetect_**[](https://www.baeldung.com/jackson-annotations#5-jsonautodetect)

_@JsonAutoDetect_  can override the default semantics of  **which properties are visible and which are not**.

First, let's take a look at how the annotation can be very helpful with a simple example; let's enable serializing private properties:

```java
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class PrivateBean {
    private int id;
    private String name;
}
```

Then the test:

```java
@Test
public void whenSerializingUsingJsonAutoDetect_thenCorrect()
  throws JsonProcessingException {
 
    PrivateBean bean = new PrivateBean(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, containsString("My bean"));
}
```

## **5. Jackson Polymorphic Type Handling Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-polymorphic-type-handling-annotations)

Next let's take a look at Jackson polymorphic type handling annotations:

-   _@JsonTypeInfo_  – indicates details of what type information to include in serialization
-   _@JsonSubTypes_  – indicates sub-types of the annotated type
-   _@JsonTypeName_  – defines a logical type name to use for annotated class

Let's examine a more complex example, and use all three –  _@JsonTypeInfo_,  _@JsonSubTypes,_ and _@JsonTypeName –_ to serialize/deserialize the entity  _Zoo_:

```java
public class Zoo {
    public Animal animal;

    @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME, 
      include = As.PROPERTY, 
      property = "type")
    @JsonSubTypes({
        @JsonSubTypes.Type(value = Dog.class, name = "dog"),
        @JsonSubTypes.Type(value = Cat.class, name = "cat")
    })
    public static class Animal {
        public String name;
    }

    @JsonTypeName("dog")
    public static class Dog extends Animal {
        public double barkVolume;
    }

    @JsonTypeName("cat")
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;
    }
}
```

When we do serialization:

```java
@Test
public void whenSerializingPolymorphic_thenCorrect()
  throws JsonProcessingException {
    Zoo.Dog dog = new Zoo.Dog("lacy");
    Zoo zoo = new Zoo(dog);

    String result = new ObjectMapper()
      .writeValueAsString(zoo);

    assertThat(result, containsString("type"));
    assertThat(result, containsString("dog"));
}
```

Here's what serializing the  _Zoo_  instance with the  _Dog_  will result in:

```javascript
{
    "animal": {
        "type": "dog",
        "name": "lacy",
        "barkVolume": 0
    }
}
```

Now for de-serialization. Let's start with the following JSON input:

```bash
{
    "animal":{
        "name":"lacy",
        "type":"cat"
    }
}
```

Then let's see how that gets unmarshalled to a  _Zoo_  instance:

```java
@Test
public void whenDeserializingPolymorphic_thenCorrect()
throws IOException {
    String json = "{\"animal\":{\"name\":\"lacy\",\"type\":\"cat\"}}";

    Zoo zoo = new ObjectMapper()
      .readerFor(Zoo.class)
      .readValue(json);

    assertEquals("lacy", zoo.animal.name);
    assertEquals(Zoo.Cat.class, zoo.animal.getClass());
}
```

## **6. Jackson General Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-general-annotations)

Next let's discuss some of Jackson's more general annotations.

### **6.1.  _@JsonProperty_**[](https://www.baeldung.com/jackson-annotations#1-jsonproperty)

We can add  **the _@JsonProperty_  annotation to indicate the property name in JSON**.

Let's use  _@JsonProperty_  to serialize/deserialize the property  _name_  when we're dealing with non-standard getters and setters:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonProperty("name")
    public void setTheName(String name) {
        this.name = name;
    }

    @JsonProperty("name")
    public String getTheName() {
        return name;
    }
}
```

Next is our test:

```java
@Test
public void whenUsingJsonProperty_thenCorrect()
  throws IOException {
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));

    MyBean resultBean = new ObjectMapper()
      .readerFor(MyBean.class)
      .readValue(result);
    assertEquals("My bean", resultBean.getTheName());
}
```

### **6.2.  _@JsonFormat_**[](https://www.baeldung.com/jackson-annotations#2-jsonformat)

**The _@JsonFormat_  annotation specifies a format when serializing Date/Time values.**

In the following example, we use  _@JsonFormat_  to control the format of the property  _eventDate_:

```java
public class EventWithFormat {
    public String name;

    @JsonFormat(
      shape = JsonFormat.Shape.STRING,
      pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}
```

Then here's the test:

```java
@Test
public void whenSerializingUsingJsonFormat_thenCorrect()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithFormat event = new EventWithFormat("party", date);
    
    String result = new ObjectMapper().writeValueAsString(event);
    
    assertThat(result, containsString(toParse));
}
```

### **6.3.  _@JsonUnwrapped_**[](https://www.baeldung.com/jackson-annotations#3-jsonunwrapped)

**_@JsonUnwrapped_  defines values that should be unwrapped/flattened when serialized/deserialized.**

Let's see exactly how this works; we'll use the annotation to unwrap the property  _name_:

```java
public class UnwrappedUser {
    public int id;

    @JsonUnwrapped
    public Name name;

    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

Now let's serialize an instance of this class:

```java
@Test
public void whenSerializingUsingJsonUnwrapped_thenCorrect()
  throws JsonProcessingException, ParseException {
    UnwrappedUser.Name name = new UnwrappedUser.Name("John", "Doe");
    UnwrappedUser user = new UnwrappedUser(1, name);

    String result = new ObjectMapper().writeValueAsString(user);
    
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("name")));
}
```

Finally, here's what the output looks like – the fields of the static nested class unwrapped along with the other field:

```java
{
    "id":1,
    "firstName":"John",
    "lastName":"Doe"
}
```

### **6.4.  _@JsonView_**[](https://www.baeldung.com/jackson-annotations#4-jsonview)

**_@JsonView_  indicates the View in which the property will be included for serialization/deserialization.**

For example, we'll use  _@JsonView_  to serialize an instance of  _Item_  entity.

First, let's start with the views:

```java
public class Views {
    public static class Public {}
    public static class Internal extends Public {}
}
```

Next here's the  _Item_  entity using the views:

```java
public class Item {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Internal.class)
    public String ownerName;
}
```

Finally, the full test:

```java
@Test
public void whenSerializingUsingJsonView_thenCorrect()
  throws JsonProcessingException {
    Item item = new Item(2, "book", "John");

    String result = new ObjectMapper()
      .writerWithView(Views.Public.class)
      .writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("2"));
    assertThat(result, not(containsString("John")));
}
```

### **6.5.  _@JsonManagedReference, @JsonBackReference_**[](https://www.baeldung.com/jackson-annotations#5-jsonmanagedreference-jsonbackreference)

**The _@JsonManagedReference_  and  _@JsonBackReference_  annotations can handle parent/child relationships**  and work around loops.

In the following example, we use  _@JsonManagedReference_  and  _@JsonBackReference_  to serialize our  _ItemWithRef_  entity:

```java
public class ItemWithRef {
    public int id;
    public String itemName;

    @JsonManagedReference
    public UserWithRef owner;
}
```

Our  _UserWithRef_  entity:

```java
public class UserWithRef {
    public int id;
    public String name;

    @JsonBackReference
    public List<ItemWithRef> userItems;
}
```

Then the test:

```java
@Test
public void whenSerializingUsingJacksonReferenceAnnotation_thenCorrect()
  throws JsonProcessingException {
    UserWithRef user = new UserWithRef(1, "John");
    ItemWithRef item = new ItemWithRef(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("userItems")));
}
```

### **6.6.  _@JsonIdentityInfo_**[](https://www.baeldung.com/jackson-annotations#6-jsonidentityinfo)

_@JsonIdentityInfo_ indicates that Object Identity should be used when serializing/deserializing values, like when dealing with infinite recursion types of problems, for instance.

In the following example, we have an  _ItemWithIdentity_  entity with a bidirectional relationship with the  _UserWithIdentity_  entity:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class ItemWithIdentity {
    public int id;
    public String itemName;
    public UserWithIdentity owner;
}
```

The  _UserWithIdentity_ entity:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class UserWithIdentity {
    public int id;
    public String name;
    public List<ItemWithIdentity> userItems;
}
```

Now  **let's see how the infinite recursion problem is handled**:

```java
@Test
public void whenSerializingUsingJsonIdentityInfo_thenCorrect()
  throws JsonProcessingException {
    UserWithIdentity user = new UserWithIdentity(1, "John");
    ItemWithIdentity item = new ItemWithIdentity(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, containsString("userItems"));
}
```

Here's the full output of the serialized item and user:

```javascript
{
    "id": 2,
    "itemName": "book",
    "owner": {
        "id": 1,
        "name": "John",
        "userItems": [
            2
        ]
    }
}
```

### **6.7.  _@JsonFilter_**[](https://www.baeldung.com/jackson-annotations#7-jsonfilter)

**The _@JsonFilter_  annotation specifies a filter to use during serialization.**

First, we define the entity and we point to the filter:

```java
@JsonFilter("myFilter")
public class BeanWithFilter {
    public int id;
    public String name;
}
```

Now in the full test, we define the filter, which excludes all other properties except  _name_  from serialization:

```java
@Test
public void whenSerializingUsingJsonFilter_thenCorrect()
  throws JsonProcessingException {
    BeanWithFilter bean = new BeanWithFilter(1, "My bean");

    FilterProvider filters 
      = new SimpleFilterProvider().addFilter(
        "myFilter", 
        SimpleBeanPropertyFilter.filterOutAllExcept("name"));

    String result = new ObjectMapper()
      .writer(filters)
      .writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

## **7. Custom Jackson Annotation**[](https://www.baeldung.com/jackson-annotations#custom-jackson-annotation)

Next let's see how to create a custom Jackson annotation.  **We can make use of the  _@JacksonAnnotationsInside_  annotation:**

```java
@Retention(RetentionPolicy.RUNTIME)
    @JacksonAnnotationsInside
    @JsonInclude(Include.NON_NULL)
    @JsonPropertyOrder({ "name", "id", "dateCreated" })
    public @interface CustomAnnotation {}
```

Now if we use the new annotation on an entity:

```java
@CustomAnnotation
public class BeanWithCustomAnnotation {
    public int id;
    public String name;
    public Date dateCreated;
}
```

We can see how it combines the existing annotations into a simple custom one that we can use as a shorthand:

```java
@Test
public void whenSerializingUsingCustomAnnotation_thenCorrect()
  throws JsonProcessingException {
    BeanWithCustomAnnotation bean 
      = new BeanWithCustomAnnotation(1, "My bean", null);

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("dateCreated")));
}
```

The output of the serialization process:

```bash
{
    "name":"My bean",
    "id":1
}
```

## **8. Jackson MixIn Annotations**[](https://www.baeldung.com/jackson-annotations#jackson-mixin-annotations)

Next let's see how to use Jackson MixIn annotations.

For example, let's use the MixIn annotations to ignore properties of type  _User_:

```java
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
```

```java
@JsonIgnoreType
public class MyMixInForIgnoreType {}
```

Then let's see this in action:

```java
@Test
public void whenSerializingUsingMixInAnnotation_thenCorrect() 
  throws JsonProcessingException {
    Item item = new Item(1, "book", null);

    String result = new ObjectMapper().writeValueAsString(item);
    assertThat(result, containsString("owner"));

    ObjectMapper mapper = new ObjectMapper();
    mapper.addMixIn(User.class, MyMixInForIgnoreType.class);

    result = mapper.writeValueAsString(item);
    assertThat(result, not(containsString("owner")));
}
```

## **9. Disable Jackson Annotation**[](https://www.baeldung.com/jackson-annotations#disable-jackson-annotation)

Finally, let's see how we can  **disable all Jackson annotations**. We can do this by disabling the  _MapperFeature._USE_ANNOTATIONS as in the following example:

```java
@JsonInclude(Include.NON_NULL)
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

Now, after disabling annotations, these should have no effect and the defaults of the library should apply:

```java
@Test
public void whenDisablingAllAnnotations_thenAllDisabled()
  throws IOException {
    MyBean bean = new MyBean(1, null);

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(MapperFeature.USE_ANNOTATIONS);
    String result = mapper.writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, containsString("name"));
}
```

The result of serialization before disabling annotations:

```bash
{"id":1}
```

The result of serialization after disabling annotations:

```bash
{
    "id":1,
    "name":null
}
```

## **10. Conclusion**[](https://www.baeldung.com/jackson-annotations#10-conclusion)

In this article, we examined Jackson annotations, just scratching the surface of the kind of flexibility we can get by using them correctly.

The implementation of all of these examples and code snippets can be found  [over on GitHub](https://github.com/eugenp/tutorials/tree/master/jackson-simple).