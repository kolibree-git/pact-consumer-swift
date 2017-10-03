# Pact Consumer Swift

[![Build Status](https://travis-ci.org/DiUS/pact-consumer-swift.svg?branch=master)](https://travis-ci.org/DiUS/pact-consumer-swift)
[![codecov](https://codecov.io/gh/DiUS/pact-consumer-swift/branch/master/graph/badge.svg)](https://codecov.io/gh/DiUS/pact-consumer-swift)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Swift Package Manager Compatible](https://img.shields.io/badge/swift_package_manager-compatible-brightgreen.svg)]()
![Swift](https://img.shields.io/badge/Swift-4.0-orange.svg?style=flat)
[![Badge w/ Version](https://cocoapod-badges.herokuapp.com/v/PactConsumerSwift/badge.png)](https://cocoadocs.org/docsets/PactConsumerSwift)
[![Badge w/ Platform](https://cocoapod-badges.herokuapp.com/p/PactConsumerSwift/badge.svg)](https://cocoadocs.org/docsets/PactConsumerSwift) ![MIT](https://cocoapod-badges.herokuapp.com/l/PactConsumerSwift/badge.png)
[![Twitter](https://img.shields.io/badge/twitter-@pact__up-blue.svg?style=flat)](http://twitter.com/pact_up)

This library provides a Swift / Objective C DSL for creating Consumer [Pacts](http://pact.io).

Why? To test communication boundaries between your app and services.
You can view a presentation on how Pact can work in a mobile context here: [Yow! Connected 2016 Andrew Spinks - Increasing The Confidence In Your Service Integrations](https://www.youtube.com/watch?v=UQkMr4bKYp4).

Implements [Pact Specification v2](https://github.com/pact-foundation/pact-specification/tree/version-2),
including [flexible matching](http://docs.pact.io/documentation/matching.html).

This DSL relies on the Ruby [pact-mock_service][pact-mock-service] gem to provide the mock service for the tests.

## Installation
Note: see [Upgrading][upgrading] for notes on upgrading from 0.2 to 0.3

### Install the [pact-mock_service][pact-mock-service]

Run `sudo gem install pact-mock_service -v 2.1.0` in your terminal.

In Xcode, edit your scheme and add _pre-_ and _post-actions_ for your `Test` step to run the provided scripts in `./scripts/` folder. Make sure you select your target in _Provide build settings from_ the drop down menu.
```
# Examples:
# Pre-actions
PATH=/full/path/to/your/rubies/bin:$PATH
"$SRCROOT"/scripts/start_server.sh

# Post-actions
PATH=/full/path/to/your/rubies/bin:$PATH
"$SRCROOT"/scripts/stop_server.sh
```
![Xcode Scheme Test Pre-actions](scripts/images/xcode-scheme-test-pre-actions.png)

### Add the PactConsumerSwift library to your project

#### Using [Carthage](https://github.com/Carthage/Carthage) library manager
- See the [PactSwiftExample](https://github.com/andrewspinks/PactSwiftExample) [![Swift, Carthage Example - Build Status](https://travis-ci.org/andrewspinks/PactSwiftExample.svg?branch=master)](https://travis-ci.org/andrewspinks/PactSwiftExample) for an example project using `pact-consumer-swift` with Carthage for an iOS target.
- See the [PactMacOSExample](https://github.com/surpher/PactMacOSExample) [![Build Status](https://travis-ci.org/surpher/PactMacOSExample.svg?branch=master)](https://travis-ci.org/surpher/PactMacOSExample) for an example project using `pact-consumer-swift` through Carthage for a macOS target.

#### Using [CocoaPods](https://cocoapods.org/pods/PactConsumerSwift) (Git Submodules)
- See the [PactObjectiveCExample](https://github.com/andrewspinks/PactObjectiveCExample) [![Build Status](https://travis-ci.org/andrewspinks/PactObjectiveCExample.svg?branch=master)](https://travis-ci.org/andrewspinks/PactObjectiveCExample) for an example project using `pact-consumer-swift` with CocoaPods for an iOS target.

#### Using [Swift Package Manager](https://swift.org/package-manager/) dependencies manager
- See the [PactSwiftPMExample](http://github.com/surpher/PactSwiftPMExample) [![Build Status](https://travis-ci.org/surpher/PactSwiftPMExample.svg?branch=master)](https://travis-ci.org/surpher/PactSwiftPMExample) for an example project using `pact-consumer-swift` library through Swift Package Manager for an executable file that runs in terminal.

## Writing Pact Tests

### Testing with Swift
  Write a Unit test similar to the following (NB: this example is using the [Quick](https://github.com/Quick/Quick) test framework)

```swift
import PactConsumerSwift

...
  beforeEach {
    animalMockService = MockService(provider: "Animal Service", consumer: "Animal Consumer Swift")
    animalServiceClient = AnimalServiceClient(baseUrl: animalMockService!.baseUrl)
  }

  it("gets an alligator") {
    animalMockService!.given("an alligator exists")
                      .uponReceiving("a request for an alligator")
                      .withRequest(method:.GET, path: "/alligator")
                      .willRespondWith(status:200,
                                       headers: ["Content-Type": "application/json"],
                                       body: ["name": "Mary"])

    //Run the tests
    animalMockService!.run { (testComplete) -> Void in
      animalServiceClient!.getAlligator { (alligator) in
        expect(alligator.name).to(equal("Mary"))
        testComplete()
      }
    }
  }
```

  An optional `timeout` (seconds) parameter can be included on the run function. This defaults to 30 seconds.

```swift
...
    animalMockService!.run(timeout: 60) { (testComplete) -> Void in
      animalServiceClient!.getAlligator { (alligator) in
        expect(alligator.name).to(equal("Mary"))
        testComplete()
      }
    }
```

### Testing with Objective-C
  Write a Unit test similar to the following

```objc
@import PactConsumerSwift;
...
- (void)setUp {
  [super setUp];
  self.animalMockService = [[MockService alloc] initWithProvider:@"Animal Provider"
                                                        consumer:@"Animal Service Client Objective-C"];
  self.animalServiceClient = [[OCAnimalServiceClient alloc] initWithBaseUrl:self.animalMockService.baseUrl];
}

- (void)testGetAlligator {
  typedef void (^CompleteBlock)();

  [[[[self.animalMockService given:@"an alligator exists"]
                             uponReceiving:@"oc a request for an alligator"]
                             withRequestHTTPMethod:PactHTTPMethodGET
                                              path:@"/alligator"
                                             query:nil headers:nil body:nil]
                             willRespondWithHTTPStatus:200
                                               headers:@{@"Content-Type": @"application/json"}
                                                  body: @"{ \"name\": \"Mary\"}" ];

  [self.animalMockService run:^(CompleteBlock testComplete) {
      Animal *animal = [self.animalServiceClient getAlligator];
      XCTAssertEqualObjects(animal.name, @"Mary");
      testComplete();
  }];
}
```

  An optional `timeout` (seconds) parameter can be included on the run function. This defaults to 30 seconds.

```objc
...
  [self.animalMockService run:^(CompleteBlock testComplete) {
      Animal *animal = [self.animalServiceClient getAlligator];
      XCTAssertEqualObjects(animal.name, @"Mary");
      testComplete();
  } timeout:60];
}
```

### Testing with XCTest
Write a Unit Test similar to the following:
```swift
import PactConsumerSwift
...
  var animalMockService: MockService?
  var animalServiceClient: AnimalServiceClient?

  override func setUp() {
    super.setUp()

    animalMockService = MockService(provider: "Animal Provider", consumer: "Animal Service Client")
    animalServiceClient = AnimalServiceClient(baseUrl: animalMockService!.baseUrl)
  }

  func testItGetsAlligator() {
    // Prepare the expecated behaviour using pact's MockService
    animalMockService!
      .given("an alligator exists")
      .uponReceiving("a request for alligator")
      .withRequest(method: .GET, path: "/alligator")
      .willRespondWith(status: 200,
                       headers: ["Content-Type": "application/json"],
                       body: [ "name": "Mary" ])

    // Run the test
    animalMockService!.run(timeout: 60) { (testComplete) -> Void in
      self.animalServiceClient!.getAlligator { (response) -> in
        XCTAssertEqual(response.name, "Mary")
        testComplete()
      }
    }
  }
  ...
```

An optional `timeout` (seconds) parameter can be included on the run function. Defaults to 30 seconds.

```swift
...
    // Run the test
    animalMockService!.run(timeout: 60) { (testComplete) -> Void in
      self.animalServiceClient!.getAlligator { (response) -> in
        XCTAssertEqual(response.name, "Mary")
        testComplete()
      }
    }
```

### Matching

In addition to verbatim value matching, you have 3 useful matching functions
in the `Matcher` class that can increase expressiveness and reduce brittle test
cases.

* `Matcher.term(matcher, generate)` - tells Pact that the value should match using
a given regular expression, using `generate` in mock responses. `generate` must be
a string.
* `Matcher.somethingLike(content)` - tells Pact that the value itself is not important, as long
as the element _type_ (valid JSON number, string, object etc.) itself matches.
* `Matcher.eachLike(content, min)` - tells Pact that the value should be an array type,
consisting of elements like those passed in. `min` must be >= 1. `content` may
be a valid JSON value: e.g. strings, numbers and objects.

*NOTE*: One caveat to note, is that you will need to use valid Ruby
[regular expressions](http://ruby-doc.org/core-2.1.5/Regexp.html) and double
escape backslashes.

See the `PactSpecs.swift`, `PactObjectiveCTests.m` for examples on how to expect error responses, how to use query params, and the Matchers.

For more on request / response matching, see [Matching](http://docs.pact.io/documentation/matching.html).

### Verifying your iOS client against the service you are integrating with
If your setup is correct and your tests run against the pack mock server, then you should see a log file here:
`$YOUR_PROJECT/tmp/pact.log`
And the generated pacts, here:
`$YOUR_PROJECT/tmp/pacts/...`

Share the generated pact file(s) with your API provider using a Pact Broker, or using the pacts in their project to test their API responses.  
See [Verifying pacts](http://docs.pact.io/documentation/verifying_pacts.html) for more information.

- For an end to end example with a ruby back end service, have a look at the [KatKit example](https://github.com/andrewspinks/pact-mobile-preso).  
- An article on [using a dockerized Node.js service](https://medium.com/@rajatvig/ios-docker-and-consumer-driven-contract-testing-with-pact-d99b6bf4b09e#.ozcbbktzk) which uses provider states.

## More reading
* The Pact website [Pact](http://pact.io)
* The pact mock server that the Swift library uses under the hood [Pact mock service](https://github.com/bethesque/pact-mock_service)
* A pact broker for managing the generated pact files (so you don't have to manually copy them around!) [Pact broker](https://github.com/bethesque/pact_broker)

## Contributing

Please read [CONTRIBUTING.md](/CONTRIBUTING.md)

[upgrading]: https://github.com/DiUS/pact-consumer-swift/wiki/Upgrading
[pact-readme]: https://github.com/realestate-com-au/pact
[pact-mock-service]: https://github.com/bethesque/pact-mock_service
[pact-mock-service-without-ruby]: https://github.com/DiUS/pact-consumer-js-dsl/wiki/Using-the-Pact-Mock-Service-without-Ruby
