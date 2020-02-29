# C\# unit test

## Visual Studio Unit Test Docs

### [Get started with unit testing](https://docs.microsoft.com/en-us/visualstudio/test/getting-started-with-unit-testing?view=vs-2019)

### [Walkthrough: Create and run unit tests for managed code](https://docs.microsoft.com/en-us/visualstudio/test/walkthrough-creating-and-running-unit-tests-for-managed-code?view=vs-2019)

### [Walkthrough: Test-driven development using Test Explorer](https://docs.microsoft.com/en-us/visualstudio/test/quick-start-test-driven-development-with-test-explorer?view=vs-2019)

### [Isolate code under test with Microsoft Fakes](https://docs.microsoft.com/en-us/visualstudio/test/isolating-code-under-test-with-microsoft-fakes?view=vs-2019)

Fakes come in two flavors:

* A stub replaces a class with a small substitute that implements the same interface. To use stubs, you have to design your application so that **each component depends only on interfaces**, and not on other components. (By "component" we mean a class or group of classes that are designed and updated together and typically contained in an assembly.)

* A shim **modifies the compiled code** of your application at run time so that instead of making a specified method call, it runs the shim code that your test provides. Shims can be used to replace calls to assemblies that you cannot modify, such as .NET assemblies.

### [Use stubs to isolate parts of your application from each other for unit testing](https://docs.microsoft.com/en-us/visualstudio/test/using-stubs-to-isolate-parts-of-your-application-from-each-other-for-unit-testing?view=vs-2019)

To use stubs, you have to write your component so that it **uses only interfaces**, not classes, **to refer to other parts of the application**. This is a good design practice because it makes changes in one part less likely to require changes in another. For testing, it allows you to substitute a stub for a real component.

To use stubs, your application has to be designed so that the different components are not dependent on each other, but only dependent on interface definitions. Instead of being coupled at compile time, components are **connected at run time**. This pattern helps to make software that is robust and easy to update, because changes tend not to propagate across component boundaries. We recommend following it even if you don't use stubs. If you are writing new code, it's easy to follow the dependency injection pattern. If you are writing tests for existing software, you might have to refactor it. If that would be impractical, you could consider using shims instead.

The code of any component of your application should **never** explicitly refer to a class in another component, either in a **declaration** or in a **new statement**. Instead, **variables** and **parameters** should be **declared with interfaces**. Component instances should be created only by the component's container.

* By "component", we mean a class, or a group of classes that you develop and update together. Typically, a component is the code in one Visual Studio project. It's less important to decouple classes within one component, because they are updated at the same time.

* It is also not so important to decouple your components from the classes of a relatively stable platform such as System.dll. Writing interfaces for all these classes would clutter your code.

You could simply write the stubs as classes in the usual way. But Microsoft Fakes provides you with a more dynamic way to create the most appropriate stub for every test.

To use stubs, you must first generate stub types from the interface definitions.

### [Use shims to isolate your app for unit testing](https://docs.microsoft.com/en-us/visualstudio/test/using-shims-to-isolate-your-application-from-other-assemblies-for-unit-testing?view=vs-2019)

Use shims to isolate your code from assemblies that are not part of your solution. To isolate components of your solution from each other, use stubs.

...  
This problem is symptomatic of the isolation issue in unit testing: programs that directly call into database APIs, communicate with web services, and so on, are hard to unit test because their logic depends on the environment.

## NUnit

[Unit testing C# with NUnit and .NET Core](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-nunit)

[NUnit Documentation](https://github.com/nunit/docs/wiki/NUnit-Documentation)

[nunit-csharp-samples](https://github.com/nunit/nunit-csharp-samples/blob/master/syntax/AssertSyntaxTests.cs)

## mock

[moq](https://github.com/moq/moq4)

[moq Quickstart](https://github.com/Moq/moq4/wiki/Quickstart)

[How to mock ConfigurationManager.AppSettings with moq](https://stackoverflow.com/questions/9486087/how-to-mock-configurationmanager-appsettings-with-moq)

[A Guide to Moq for Rhino.Mocks Users](https://www.wrightfully.com/guide-to-moq-for-rhino-mocks-users)

[Intro to Mocking with Moq](https://spin.atomicobject.com/2017/08/07/intro-mocking-moq/)

[Mock a method for test](https://stackoverflow.com/questions/36345282/mock-a-method-for-test)

[Any alternative for Microsoft Fakes in .NET Core](https://stackoverflow.com/questions/52497439/any-alternative-for-microsoft-fakes-in-net-core)

[Pose document](https://www.nuget.org/packages/Pose)

[Pose - github](https://github.com/tonerdo/pose)

## Other resources

[Developer Testing - Why should developers write tests?](http://www.bradoncode.com/blog/2015/05/10/developer-testing/)

[Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)

[Mock framework vs MS Fakes frameworks](https://stackoverflow.com/questions/9677445/mock-framework-vs-ms-fakes-frameworks)

[Should I test private methods or only public ones?](https://stackoverflow.com/questions/105007/should-i-test-private-methods-or-only-public-ones)

[Best way to unit test methods that call other methods inside same class](https://softwareengineering.stackexchange.com/questions/188609/best-way-to-unit-test-methods-that-call-other-methods-inside-same-class)

[What would be an alternate to \[TearDown\] and \[SetUp\] in MSTest?](https://stackoverflow.com/questions/6193744/what-would-be-an-alternate-to-teardown-and-setup-in-mstest)
