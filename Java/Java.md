
## Contents
1. [Lambda](#lambda)
2. [Streams](#stream)
3. [Deserializer - Jackson](#jackson)
	1. [@JsonRootName](#JsonRootName)
	2. [@JsonUnwrapProperty (generic annotation)](#JsonUnwrapProperty)
	3. [@JsonDeserialize(using = CustomDeserializer.class)](#JsonDeserialize)
4. [Junit Jupter](#junitjupter)
	1. [@ParameterizedTest unit test](#parameterizedTest)
	2. [@Assertions](#Assertions)
	3. [@Stepverifier](#Stepverifier)
5. [Spring Reactive Webflux](#Webflux)
6. [Spring Service factory](#serviceFactory)
7. [Design](#Design)
	1. [Abstract Class Injection](#abstractClassInjection)



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
```Java 
/**  
 * Code to transformation of List<A<B>> to List<B<A>>  
 *     [A1[B1], A2[B1, B2], A3[B1, B2, B3]] => [B1[A1, A2, A3], B2[A2, A3], B3[A3]]
 * 
 */
Map<Policy, List<Rule>> policiesAndRulesMap = allRuleFromAPI.stream()  
        .map(ruleAPI ->  
                ruleAPI.getListOfPolicyFromAPI.stream()  
                        .flatMap(policyFromAPI -> Stream.of(PolicyMapper.map(policyFromAPI)))
                        .collect(Collectors.toMap(
                        k -> k, 
                        v -> Arrays.asList(RuleMapper.map(ruleAPI)))))
        .flatMap(map -> map.entrySet().stream()) // Stream of map to Stream of entries
/** 
*	PART 1  
*     ([B1], [A1]),  
*     ([B2], [A1]),  
*     ([B2], [A2]),  
*     ([B3], [A1]),  
*     ([B3], [A2]),  
*     ([B3], [A3])
*/
        .collect(Collectors.toMap(
		        Entry::getKey, 
		        v -> new ArrayList<>(v.getValue()),  
                (left, right) -> {  
                    left.addAll(right);  
                    return left;  
                })
        ); // Group the entry::key and merge values with the same key 
/**
*  PART 2  
*     ([B1], [A1]),  
*     ([B2], [A1], [A2]),  
*     ([B3], [A1]), [A2]), [A3])
*/

List<Policy> policiesResponse = policiesAndRulesMap.entrySet()  
        .stream()  
        .flatMap(policyAPIListEntry -> {  
            policyListEntry.getKey().setRules(policyListEntry.getValue());  
            return Stream.of(policyListEntry);  
        })  
        .flatMap(entry -> Stream.of(entry.getKey()))  // get only key's
        .toList();
/** 
*  PART 3  
*      [B1[A1], B2[A1,A2], B3[A1,A2,A3]
*/
```

```Java
// SummaryStatistics to get Max and Min for a list
LongSummaryStatistics collect = bookings.stream()  
        .sorted(Comparator.comparing(Booking::getDate))  
        .map(Booking::getDate)  
        .map(LocalDate::toEpochDay)  
        .collect(
	        Collectors.summarizingLong(Long::longValue)
	    );
	    
LocalDate from = LocalDate.ofEpochDay(collect.getMin());  
LocalDate to = LocalDate.ofEpochDay(collect.getMax());
```

```Java
// PartitioningBy to collect
Map<Boolean, List<Boolean>> collect = dates.stream()  
        .flatMap(date -> Stream.of(Objects.nonNull(date)))  
        .collect(
	        Collectors.partitioningBy(booking -> Objects.nonNull(booking))
        );
```

```java
// Stream to reduce
BigDecimal sumEasy = Stream.iterate(BigDecimal.ZERO, z -> z.add(BigDecimal.ONE))  
        .limit(20)  
        .reduce(BigDecimal.ZERO, BigDecimal::add);  
  
BigDecimal sumComplex = Stream.iterate(BigDecimal.ZERO, z -> z.add(BigDecimal.ONE))  
        .limit(20)  
        .reduce(BigDecimal.ZERO, (a, b) -> a.add(b), BigDecimal::add);
```


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
* Annotation Value source
```Java
@ParameterizedTest 
@ValueSource(strings = {"", " ", null})
void isStringTest(String value) {
...
}
```
* CSV Source
```Java
@ParameterizedTest  
@CsvSource({  
        ",false",  
        "'',false",  
        "-1,false",  
        "33,false",  
        "23,true"})  
void isStringWhatExpectedTest(String value, boolean expected) {
...
}
```

###### #Assertions 
```Java
Throwable runtime = Assertions.assertThrows(  
        ApiBadRequestException.class,  
        () -> ruleDocService.searchNetworkObjects(filterKey, value));  
Assertions.assertEquals(ApiBadRequestException.class, runtime.getClass());
```

```Java
Assertions.assertAll(  
        () -> Assertions.assertNotNull("true"),
        () -> Assertions.assertNull(null)
);
```

###### #Stepverifier
```Java 
// On ERROR
Mockito.when(teamRepository.findByName(Mockito.anyString()))
	.thenReturn(Mono.empty());
           
StepVerifier
           .create(teamService.findByName("some name"))
           .expectErrorMatches(error -> error instanceof TeamServiceException)
           .verify();
    }

// ExpectNext
when(service.search(filterKey, value))
	.thenReturn(arrayListOfA);

StepVerifier.create(service.search(filterKey, value))  
        .expectNext(arrayListOfA)  
        .verifyComplete();
```


----
###### #Webflux 
- Mono 
```java
// Convet Mono<List<Object>> to List<Object>
List<Object> block = myService.MethReturMonoListObject().block();
```

```Java 
// Adapter Mono<List<ResponseData>> from Mono<List<T>> 
Mono<List<ResponseData>> listMono = service.returnMonoListSomething(name, value)  
        .log()  
        .flatMapIterable(listSomething -> listSomething.stream()  
                .map(ResponseDataMapper::map)  
                .toList())  
        .distinct()  
        .collectList();
```
```Java
// Mono.Zip 
Mono.zip(
		// First List to zip, List Of Two
		listOne.stream()  
        .map(itemListOne ->  
                service.getMonoTwoFromItemListOne(itemListOne)  
                    .doOnNext(two -> log.info("Retrieve Two Information :{}", two))  
                    .map(Two::nameFromTwo))  
        .toList(), // Return List<Mono<Two>> 
        // Second List to zip, For block the first list and cast to List<Two>
        // some magic to hold the execution until all items from first list is loaded
        data ->  
        Arrays.stream(data)  
                .map(Two.class::cast)  
                .collect(Collectors.toList()))
        ) // This return Tuple
```

```Java
public Mono<List<Response>> getDataFromIntegration(String Key, String value) {
        return service.returnMonoListA(key, value) // return Mono<List<A>>
                .log()
                .flatMapIterable(a -> a.stream()
                        .map(ResponseMapper::map)
                        .toList())
                .distinct()
                .collectList();
    } // Mapper A to Response
```

```Java
// Convert Mono object of List to a Flux 
Mono<List<String>> monoWithList = Mono.empty();  
Flux<String> stringFlux = monoWithList  
        .flatMapIterable(Function.identity());
```

###### #serviceFactory
```Java
@Service  
public class ServiceFactory {  
    private final Map<ServiceType, IService> servicesMap = new HashMap<>();  
  
    @Autowired  
    public ServiceFactory(List<IService> services) {  
        if (Objects.nonNull(services))  
            services.forEach(service -> servicesMap.put(service.getType(), service));  
        else            
	        throw new RuntimeException("List of IServices not found: ");  
    }  
  
    public IService getInstance(ServiceType serviceType) {  
        IService serviceInstance = servicesMap.get(serviceType);  
        if (Objects.isNull(serviceInstance))  
            throw new RuntimeException("Service not found: " + serviceType);  
        return serviceInstance;  
    }  
}
```


###### #Design  
###### #abstractClassInjection
**Setter Injection**
```Java 
public abstract class BallService {

    private LogRepository logRepository;

	// Should use FINAL keyword to avoid override from subclass
    @Autowired
    public final void setLogRepository(LogRepository logRepository) {
        this.logRepository = logRepository;
    }
}
```
**Constructor Injection**
```Java
// Abstract class 
public abstract class BallService {

    private RuleRepository ruleRepository;

    public BallService(RuleRepository ruleRepository) {
        this.ruleRepository = ruleRepository;
    }
}
 // Subclass implementation
@Component
public class BasketballService extends BallService {

    @Autowired
    public BasketballService(RuleRepository ruleRepository) {
        super(ruleRepository);
    }
}
```