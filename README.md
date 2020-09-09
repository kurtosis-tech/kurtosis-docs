Kurtosis Docs
=============
The docs for the Kurtosis testing framework

What's Kurtosis?
----------------
The Kurtosis testing framework is a system that allows you to write tests against arbitrary distributed systems. These tests could be as simple as, "create a single Elasticsearch node and make a request against it", or as complex as "spin up a network containing a database, a Kafka queue, and your custom services and run end-to-end tests on it". Each test declares the network it wants and the test logic to run, and Kurtosis handles launching the network and running the test logic. The nodes of the network are Docker containers, so anything you have a Docker image for can be used in Kurtosis.

How does it work?
-----------------
You'll define a test suite in the language of your choice by importing one of our Kurtosis client libraries, implement a few interfaces to tell Kurtosis how to run the networks and tests you want, package your suite into a Docker image, and then instruct Kurtosis to run your image. It's that easy! 

At a high level, the interfaces you'll need to implement are:
* `Service` to define the programmatic interface for your test to interact with the service(s) in your network
* `ServiceInitializerCore` to tell Kurtosis how to launch an instance of your service
* `ServiceAvailabilityCheckerCore` to tell Kurtosis how to verify your service is available
* `Network` to define the programmatic interface for your test to interact with the network as a whole
* `NetworkLoader` to tell Kurtosis how to construct an instance of your `Network` implementation
* `Test` to define a test and the `NetworkLoader` the test will use
* `TestSuite` to package your tests together

The client libraries we currently have built are:
* [Go](https://github.com/kurtosis-tech/kurtosis-go)

Let us know if your language isn't on the list and we can discuss!
