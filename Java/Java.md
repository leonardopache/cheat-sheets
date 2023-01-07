
## Contents
1. [Lambda](#lambda)
2. [Streams](#stream)
3. [Deserializer - Jackson](#jackson)
	1. [@JsonRootName](#JsonRootName)
	2. [@JsonUnwrapProperty (generic annotation)](#JsonUnwrapProperty)
	3. [@JsonDeserialize(using = CustomDeserializer.class)](#JsonDeserialize)
4. [Junit Jupter](#junitjupter)
	1. [@ParameterizedTest unit test](#parameterizedTest)
	2. 

----
_My notes for the future_

----

###### #lambda  

```java
(parameter1, parameter2) -> { code block }
```

```java
Arrays.asList(1,2,3,4).stream()
.forEach( n -> { 
System.out.println(n);
});
```

###### #stream

###### #jackson
###### #JsonRootName
```JSON
{
	"wrapper_element": {
	"boolean_attribute": false,
	"result_type_car": {
		"id": 1234,
		"subject": "Super Car",
		"model": "Opel",
		"status": "NOT_IN_PRODUCTION",
		"create_date": "2022-03-29T15:57:25.717+02:00",
		"update_date": "2022-03-29T16:35:03.420+02:00",
		"close_date": "2022-03-29T16:35:03.414+02:00"
		}
	}
}
```
``` Java
@JsonRootName("wrapper_element")  
record SomeDTO(
	@JsonProperty("boolean_attribute") boolean booleanAttribute,  
	@JsonProperty("result_type_car")  
	@JsonFormat(with = JsonFormat.Feature.ACCEPT_SINGLE_VALUE_AS_ARRAY) 
	List<Car> cars) {}
```

*For JsonRootName works is needed to set enable in the mapper the **DeserializationFeature.UNWRAP_ROOT_VALUE***

###### #JsonUnwrapProperty

```Json
{
	"devices": {
		"device": [
			{
			"id": 1,
			"name": "device-name",
			"ip": "127.0.0.1",
			"vendor": "Check Point",
			"device_type": "localhost"
			}
		]
	}
} 
```
```Java
// Generic annotation for deserialization JsonUnwrapProperty.java
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonDeserialize(using = JsonUnwrapPropertyDeserializer.class)
public @interface JsonUnwrapProperty {
}

// Generic Deserialize used in the annotation JsonUnwrapPropertyDeserializer.java
public class JsonUnwrapPropertyDeserializer extends JsonDeserializer<Object> implements ContextualDeserializer {
	private JavaType unwrappedJavaType;
	private String unwrappedProperty;
	
	@Override
	public JsonDeserializer<?> createContextual(DeserializationContext deserializationContext, BeanProperty beanProperty)
		throws JsonMappingException {
		unwrappedProperty = beanProperty.getMember().getName();
		unwrappedJavaType = beanProperty.getType();
		return this;
	}
	
	@Override
	public Object deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
		ObjectNode treeNode = jsonParser.readValueAsTree();
		final TreeNode targetObjectNode = treeNode.findPath(unwrappedProperty);
		return !targetObjectNode.isMissingNode() ?
				jsonParser.getCodec().readValue(targetObjectNode.traverse(), unwrappedJavaType) : null;
	}
}
```
```Java
// Usage of JsonUnwrapProperty annotation DeviceResponse.java
class DeviceResponse{
	@Getter
	@JsonUnwrapProperty()
	@JsonProperty("devices")
	private List<OneDevice> device;
	}
```

###### #JsonDeserialize 
```Json
{
	"devices": {
		"device": [
			{
				"id": 1,
				"name": "device-name",
				"ip": "127.0.0.1",
				"vendor": "Check Point",
				"device_type": "localhost"
			}
		]
	}
} 
```
```Java
// Place holder class to use @JsonDeserialize(using = DeviceCustomDeserializer.class) with also @JsonRootName("devices")  DevicesResponse.java

@JsonRootName("devices")
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonDeserialize(using = DeviceCustomDeserializer.class)
public class DevicesResponse {
}

// Custom Device Deserializer DeviceCustomDeserializer.java
public class DeviceCustomDeserializer extends StdDeserializer<Set<OneDevice>> {
	public DeviceCustomDeserializer() {
		this(null); 
	}
public DeviceCustomDeserializer(Class<?> vc) { super(vc);}

@Override
public Set<OneDevice> deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException {
	ObjectMapper mapper = new ObjectMapper();
	JsonNode deviceNode =context.readTree(jsonparser).get("device");
	return mapper.convertValue(deviceNode, new TypeReference<Set<OneDevice>>() {});
	}
}
```
```java
// At the end the in the usage will be cast to what is defined in the custom deserialization
final Set<OneDevice> devices = (Set<OneDevice>) mapper.readValue(json, DevicesResponse.class);
```

###### #junitjupter 
###### #ParameterizedTest 
* Method source:
```Java
private static Stream<Arguments> provideArgsForTest() {  
    return Stream.of(Arguments.of(new Object(), "",""));  
}

@ParameterizedTest  
@MethodSource("provideArgsForTest")  
void getNetworkObjects(Object expectedResponse, String key, String value) {
...
}
```
* B
```Java
```
* C
```Java
```
* D
```Java
```


----
### Webflux 
- Mono 
```java
// Convet Mono<List<Object>> to List<Object>
List<Object> block = myService.MethReturMonoListObject().block();
```

