# Unit Testing

Unit Tests: readability on the same level as business code
Unit Tests: more than one assert is still ok if makes sense

Truth.assertThat(flattened).containsExactlyElementsIn(expected)

## Test Driven Development (TDD)

### LAWS

1. **First Law:** You may not write production code until you have written a failing unit test.
1. **Second Law:** You may not write more of a unit test than is sufficient to fail, and not compiling is failing.
1. **Third Law:** You may not write more production code than is sufficient to pass the currently failing test.

### UI tests

1. Add test rule @Rule for activity, it will run:
   a) run before all @before methods
   b) created for each @test method
   c) will be destroyed after @after methods
   you can specify how it should start when creating it with parameter giving false for launchActivity cause that you
   have implement in @before method creating intent that will starts the activity

Espresso:
ViewMatchers - allows for selecting view to interact with with usage of: withId(),withText(), withState() and so on
ViewAction - what you can perform() on view from viewMatcher,
ViewAssertion - used inside check(viewAssertion) to verify state

### Instrumented Unit Test

to make it work you need add:
manifest:
androidTestImplementation "androidx.arch.core:core-testing:2.0.0"
testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
androidTestImplementation 'androidx.test:runner:1.1.0'
androidTestImplementation 'androidx.test:rules:1.1.0'
class:     @get:Rule
var instantTaskExecutorRule = InstantTaskExecutorRule()

var value: T? = null
val latch = CountDownLatch(1)

val observer = Observer<T> { t ->
value = t
latch.countDown()
}

observeForever(observer)

latch.await(2, TimeUnit.SECONDS)
return value
}

### Parameterized Unit Tests

To keep the testing class clean from all the sources for the testing I extracted all testing data to separate file.
When extracting the source/arguments to different file I noticed below behaviour:

1. Using `@JvmStatic` on source/argument method is causing lint to stop working - I am not sure what is the reason, I couldn't find anything interesting while searching. 
2. To fix this, I started using `@TestInstance(TestInstance.Lifecycle.PER_CLASS)` on the test class. This leads to:
```
object is not an instance of declaring class java.lang.IllegalArgumentException: object is not an instance of declaring class...
Suppressed: org.junit.platform.commons.PreconditionViolationException: Configuration error: You must configure at least one set of arguments for this @ParameterizedTest
```
I couldn't find explanation for this either. 

Though the working solution is implementing `ArgumentsProvider` for the source/arguments 
```
internal class MyTestData : ArgumentsProvider {
    override fun provideArguments(context: ExtensionContext?): Stream<out Arguments> {
        return Stream.of(
          Arguments.of(...), 
          Arguments.of(...), 
          Arguments.of(...) 
        );
    }
}
```
and instead of using `@MethodSource(...)` using `@ArgumentsSource(MyTestData::class)`

This way arguments/sources are in separate file, and you have them nicely linked to the tested class (as you specify class, instead of the string).
In case of using different file && `@MethodSource(you.have.to.specify.whole.path.to.MyTestData#function)` and it won't auto update itself when changing the package.
