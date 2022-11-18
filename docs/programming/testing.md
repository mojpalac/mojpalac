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
you can specify how it should start when creating it with parameter giving false for launchActivity cause that you have implement in @before method creating intent that will starts the activity

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