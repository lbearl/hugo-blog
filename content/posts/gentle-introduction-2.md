---
title: "A Gentle Introduction to Onion Architecture in ASP.NET MVC - Part 2"
date: "2016-10-19"
categories: 
  - "net"
  - "architecture"
  - "software-development"
---

In [part 1](http://lukebearl.com/2016/09/gentle-introduction-1/) of this series we discussed what an onion architecture application would look like and discussed the technologies that we can leverage in .Net 4 in order to make that work. In this section we'll go over how the project is structured, including spending a bit of time looking at how the IoC container is configured. This being a simple application the configuration is significantly easier to understand than it can be in more complex applications.

### Project Structure

The application consists of 4 projects: Core, Infrastructure, Infrastructure.Tests, and Web. Each one of these projects has a unique purpose and it behooves all developers to ensure that they don't mix concerns between projects.

#### Core Project

The core project is responsible for defining all interfaces for all services which will be implemented in the infrastructure layer, and it also is responsible for having all domain models. In Entity Framework Code First projects, the EF Entity Models can exist in Core. The models that exist in the sample application are not "true" domain models, instead they are just POCO representations

#### Infrastructure and Test Project

The infrastructure project is responsible for the implementation of all of the services defined in the Core Project. One of the critical distinctions between onion architecture and traditional layered applications is that the data access code (if there is any) will live in an infrastructure style project instead of living in the base/core layer. In the sample application the Infrastructure project only calls out to third-party services. The test project only tests the behavior of the composite service as I have not written this application in a sufficiently decoupled fashion to pass in the `RestClient` and ideally an abstraction would also be built around the `ConfigurationManager`

#### Web Project

As the name probably implies, this is where the "web" parts of the application go, including controllers, views, front-end assets, etc. For this application front end assets are just managed by downloading and saving them into `/Scripts`, and then everything is manually wired up using the `BundleConfig.cs` in `App_Start`. The interactivity within the application is achieved by using a little bit of jQuery.

### Dependency Resolution

This application exposes 4 services in total, but only has 2 interfaces. This is due to the fact that the `CompositeBounceCheckerService` is composed of both `MailgunService` and `SendGridService`, hence all three of them share the same interface. The final service, the `SuppressionListCheckService` just consumes the `CompositeBounceCheckerService`. This final layer of indirection isn't, strictly speaking, necessary, however it does afford the ability easily pass one of the service-specific `IThirdPartyBounceService` services as its dependency if we only wanted to check for suppression in a single ESP. The `DefaultRegistry` below shows how to get that all setup.

```
            For<IThirdPartyBounceService>().Use<CompositeBounceCheckerService>()
                .EnumerableOf<IThirdPartyBounceService>().Contains(x =>
                {
                    x.Type<SendGridService>();
                    x.Type<MailgunService>();
                });
```

This code basically tells StructureMap to scan all registered assemblies (all assemblies listed in the `Scan` call above this) and register the `SendGridService` and `MailgunService` as services within the composite service.

StructureMap is capable of doing a lot more, such as having custom life-cycles for certain services or handling weird object hierarchies (you can do a lot more than just have interfaces and services).

### Testing

Testing is fairly direct as we just mock out the services that feed into the service we want to test (generally known as the "SUT" or the System Under Test). One easy example of a test is this:

```
        [Fact]
        public void Get_Bounces_Returns_A_List_Of_View_Models()
        {
            // Arrange
            var returnList = new List<SuppressedEmailViewModel>
            {
                new SuppressedEmailViewModel
                {
                    AddedOn = DateTime.Now,
                    EmailAddress = "test@example.org",
                    EmailServiceProvider = EspEnum.UNKNOWN,
                    ErrorCode = string.Empty,
                    ErrorText = string.Empty
                }
            };

            var mockService1 = new Mock<IThirdPartyBounceService>();
            mockService1.Setup(x => x.GetBounces()).Returns(returnList);
            // Initialize the composite service with an array of one third party service.
            var compositeService = new CompositeBounceCheckerService(new[] { mockService1.Object });

            //Act and Assert
            var result = compositeService.GetBounces();

            Assert.Equal(returnList, result);
            mockService1.Verify(x => x.GetBounces(), Times.Once);
        }
```

The code above will first: create data for the Mock to return, then: create and setup the mock, then: inject it into the service, then: call the service, and finally: make sure that the behavior of the service was correct. Having a robust suite of tests allows us to change the implementation of any of the services and still verify that the output is correct.

When writing tests, I generally follow the AAA pattern (Arrange, Act, Assert) and leave comments in the code where those things are happening as an easy way to make sure that my tests are structured in a consistent fashion.

That's all for now folks. I hope that this two part series on the onion architecture made the benefits of using it a bit more clear.

Please, check out the code on [Github](https://github.com/lbearl/BlockListChecker), and [drop me a line](//twitter.com/lukebearl) if you have any questions!
