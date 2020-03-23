[![Test](https://github.com/Nylle/KotlinFixture/workflows/Test/badge.svg)](https://github.com/Nylle/KotlinFixture/actions?query=workflow%3ATest)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.nylle/kotlinfixture.svg?label=maven-central)](http://search.maven.org/#artifactdetails|com.github.nylle|kotlinfixture|1.0.0|)
[![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat)](http://opensource.org/licenses/MIT)

# KotlinFixture
KotlinFixture is a wrapper for [JavaFixture](https://github.com/Nylle/JavaFixture), an attempt to bring the incredibly easy usage of [Mark Seemann's AutoFixture for .NET](https://github.com/AutoFixture/AutoFixture) to Java.

It strives to provide a kotlin-specific API for better integration. The purpose of this project is to generate full object graphs for use in test suites.


## Contents
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Generics](#generics)
- [Random Constructor](#constructor)
- [Configuration](#configuration)
- [JUnit5 Support](#junit5-support)
- [Parameterized Tests](#parameterized-tests)


## Getting Started
**Heads up!** Due to a [bug in dokka](https://github.com/Kotlin/dokka/issues/294) it is currently impossible to release to maven-central.
```xml
<dependency>
    <groupId>com.github.nylle</groupId>
    <artifactId>kotlinfixture</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```
## Usage

### Create a Fixture
```kotlin
val fixture = Fixture()
```

### Autogenerated String
```kotlin
val result = fixture.create<String>()
```
#### Sample Result
String: "c3932f6f-59ae-43df-8ee9-8788474a3f87"

### Autogenerated Number
```kotlin
val result = fixture.create<Int>()
```
#### Sample Result
int: -1612385443

### Complex Type
```kotlin
val result = fixture.create<ParentDto>()
```
#### Sample Result
- ParentDto:
    - id: String: "4ed0f3c4-5ea3-4dbb-b31c-f92c036af463"
    - child: ChildDto:
        - id: String: "c3932f6f-59ae-43df-8ee9-8788474a3f87"
        - names: ArrayList:
            - String: "9452541b-c6f9-4316-b254-28d00b327d0d"
            - String: "52ac46e4-1b21-40c8-9213-31fc839fbdf7"
            - String: "333af3f6-4ed1-4580-9cae-aaee271d7ba7"

### Collection of Strings
```kotlin
val result: Sequence<String> = fixture.createMany<String>()
```
#### Sample Result
ArrayList: 
- String: "333af3f6-4ed1-4580-9cae-aaee271d7ba7"
- String: "9452541b-c6f9-4316-b254-28d00b327d0d"
- String: "4ed0f3c4-5ea3-4dbb-b31c-f92c036af463"

### Add to Collection
```kotlin
val collection = mutableListOf("HELLO!")
fixture.addManyTo(collection)
```
#### Sample Result
ArrayList: 
- String: "HELLO!"
- String: "333af3f6-4ed1-4580-9cae-aaee271d7ba7"
- String: "9452541b-c6f9-4316-b254-28d00b327d0d"
- String: "4ed0f3c4-5ea3-4dbb-b31c-f92c036af463"

### Set Public Property
```kotlin
val result = fixture.build<TestDto>()
                    .with { it.myPublicField = 123 }
                    .create()
```
#### Sample Result
TestDto:
- myPrivateField: String: "349a1f87-9d00-4623-89cb-3031bb84ddb3"
- myPublicField: int: 123

### Set Private Field
```kotlin
val result = fixture.build<TestDto>()
                    .with("myPrivateField", "HELLO!")
                    .create()
```
#### Sample Result
TestDto:
- myPrivateField: String: "HELLO!"
- myPublicField: int: 26123854

### Omit Field
```kotlin
val result = fixture.build<TestDto>()
                    .without("myPrivateField")
                    .create()
```
#### Sample Result
TestDto:
- myPrivateField: String: null
- myPublicField: int: -128564

### Omit Primitive Field
```kotlin
val result = fixture.build<TestDto.class>()
                    .without("myPublicField")
                    .create()
```
#### Sample Result
TestDto:
- myPrivateField: String: "349a1f87-9d00-4623-89cb-3031bb84ddb3"
- myPublicField: int: 0


### Perform Multiple Operations
```kotlin
val child = fixture.create<String>()
val parent = fixture.build<ParentDto>()
                    .with { it.addChild(child) }
                    .with { it.youngestChild = child }
                    .create()
```
#### Sample Result
ParentDto:
- children: ArrayList:
    - String: "710ba467-01a7-4bcc-b880-84eda5458989"
    - String: "9452541b-c6f9-4316-b254-28d00b327d0d"
    - String: "4ed0f3c4-5ea3-4dbb-b31c-f92c036af463"
    - String: "349a1f87-9d00-4623-89cb-3031bb84ddb3"
- youngestChild: String: "349a1f87-9d00-4623-89cb-3031bb84ddb3"

## Generics
 ```kotlin
 val result = fixture.create<Optional<String>>();
 ```
 
## Constructor
There might be some cases when you want to create an object not by instantiating it and setting all fields, but by calling one of its constructors and feeding it random values.
```kotlin
val result = fixture.construct<String>()
```
_Keep in mind that only public constructors are allowed._

## Configuration
The values below are the default values, used when no configuration is provided.
```kotlin
val config = Configuration.configure()
                    .collectionSizeRange(2, 10)
                    .streamSize(3)
                    .usePositiveNumbersOnly(false)
                    .clock(Clock.fixed(Instant.now(), ZoneOffset.UTC));

val fixture = Fixture(config);
```
- `collectionSizeRange` determines the range from which a random collection size will be picked when creating any collection, map or array
- `streamSize` determines the number of objects to be returned when using `Sequence<T> Fixture.createMany<T>`
- `usePositiveNumbersOnly` defines whether to generate only positive numbers including 0 for `short`, `int`, `long`, `float`, and `double`
- `clock` sets the clock to use when creating time-based values

## JUnit5 Support

In order to use JUnit5 support you need to add the following dependencies if not already present.

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.5.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.5.1</version>
    <scope>test</scope>
</dependency>
```

_Remember to also include the `vintage` dependency if you still have JUnit4-tests, otherwise they won't be run._

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.5.1</version>
    <scope>test</scope>
</dependency>
```

### Inject Random Values Into Single Test
All arguments of the test-method below will be provided as a random object generated by JavaFixture.
```kotlin
@TestWithFixture
fun `inject parameter via method extension`(testObject: TestDto, intValue: int, genericObject: Optional<String>) {
    assertThat(testObject).isInstanceOf(TestDto::class.java)
    
    assertThat(intValue).isBetween(Int.MIN_VALUE, Int.MAX_VALUE)
    
    assertThat(genericObject).isInstanceOf(Optional::class.java)
    assertThat(genericObject).isPresent
    assertThat(genericObject.get()).isInstanceOf(String::class.java)
}
```
