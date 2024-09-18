---
title: "XUnit Test Lifecycles"
date: "2017-01-08"
categories: 
  - "net"
  - "software-development"
  - "unit-testing"
---

You are unit testing right? I hope so. If you are you may have run into some scenarios where things are not working quite right. One issue I've personally run into is things which attempt to maintain state between tests (eww, I know). Unfortunately, while it is good practice to make sure all of the tests you are writing are completely independent from each other, sometimes shared state will creep in due to other factors (like using an in-memory database because Microsoft kills kittens[\*](#footer) and didn't make Entity Framework easy to mock out).

## [XUnit Test Context](https://xunit.github.io/docs/shared-context.html)

How many times does a constructor get called for a class in C#? If you answered one, you'd be completely wrong when it comes to XUnit (it kind of surprised me too when I first learned about it). Turns out that as part of the XUnit lifecycle the constructor is called before each test is run, likewise you can implement `IDisposable` and have `Dispose()` called after each test is run. While it runs kind of counter to expectations, no state set in any instance variables will be shared between any tests. If you need the ability to share state XUnit provides not one, but two separate options: 1) Class Fixtures and 2) Collection Fixtures. The choice of which to use is entirely dependent upon the scope of what needs to share state. As you'll see below by default a class is by default a collection although you can also build collections composed of the tests of multiple classes.

### Class Fixtures

In order to create a class fixture a small amount of setup needs to be done: a new class needs to be created which will maintain the shared state across all runs of all tests that are part of the class. The test in question needs to implement `IClassFixture<YourFixtureName>` and your fixture will be passed in as a constructor argument.

An example fixture might be something like:

```
public class MyFixture{
   private MySuperObject _myInstanceVar;
   public MyFixture(){
       _myInstanceVar = new MySuperObject();
   }
```

while consuming the fixture would look like this:

```
public class FixtureTests : IClassFixture<MyFixture>
{
    private MyFixture _fixture;
    public FixtureTests(MyFixture <span class="hiddenGrammarError" pre=""><span class="hiddenGrammarError" pre=""><span class="hiddenGrammarError" pre="">fixture)
    {
         _fixture</span></span></span> = fixture;
    }
}
```

From a test writers perspective the Fixture mechanism almost looks like a dependency injection engine (except with absolutely zero magic in it).

### Collection Fixtures

Collection fixtures operate pretty similarly to class fixtures, although they do require a slightly larger amount of setup. In order to setup a collection fixture, much like a class fixture, you must first define the actual fixture. The wiring logic is a bit different: First define a collection:

```
[CollectionDefinition("Awesome Collection")]
public class MyCollection : ICollectionFixture<MyFixture>
{
    // Intentionally left empty. 
}
```

The actual test class would look like the below:

```
[Collection("Awesome Collection")]
public class MemberOfMyCollectionTests
{
    private MyFixture _fixture;
    public FixtureTests(MyFixture fixture)
    {
         _fixture = fixture;
    }
}
```

_Note on best practice: it is probably almost always best to pull the collection name out to a constants file to prevent simple typos from dramatically changing the behavior of your tests. But I'm in favor of just using constants everywhere unless there is a compelling reason not to... Second note: the fixtures must all be in the same assembly. XUnit doesn't let us get fancy with collection definitions spanning multiple assemblies (not that there would be any big benefit from that)_

### Fixture Lifetimes

For either of the possible fixture types the lifetime logic is basically the same: the fixture is created immediately prior to the first invocation of the first test in the collection (be it a class-collection or a collection-collection). The fixture is destroyed immediately after the last test in the collection.

## XUnit Test Collections

The lifecycle and parallelization of tests is largely driven by the concept of Test Collections. The [official documentation](https://xunit.github.io/docs/running-tests-in-parallel.html) does a good job of giving a quick overview. To summarize: by default all tests in a given class are part of the same collection thus will run in a serial fashion. Classes in the same assembly will run their collections in parallel. This behavior is able to customized by specifying a couple of assembly level attributes:

1. Forces all tests in all classes to be in a single collection - i.e. force serial execution of all tests in the assembly. `[assembly: CollectionBehavior(CollectionBehavior.CollectionPerAssembly)]`
2. Determines the maximum parallelization of tests. Setting this to 1 only allows one test to execute at a time, although multiple tests may be executed in an interleaved fashion. By default this is equal to the number of virtual CPUs on the PC (which seems to be a good default unless you have some very specific use-cases). `[assembly: CollectionBehavior(MaxParallelThreads = n)]`
3. Disables test paralleization assembly-wide. Note that this is different than MaxParallelThreads since it actually turns off the parallelization infrastructure in xunit (for the assembly this attribute decorates) `[assembly: CollectionBehavior(DisableTestParallelization = true)]`

\*Just kidding. No kittens were harmed in the writing of this post.
