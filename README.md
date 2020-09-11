Kurtosis Docs
=============
The docs for the Kurtosis testing framework

What's Kurtosis?
----------------
Kurtosis is a framework for writing tests for any networked system - be it a blockchain, a distributed datastore, or a microservice architecture. It handles all the gruntwork of setup, test execution, and teardown so you don't have to. These tests could be as simple as, "create a single Elasticsearch node and make a request against it", or as complex as "spin up a network containing a database, a Kafka queue, and your custom services and run end-to-end tests on it". Each test declares the network it wants and the test logic to run, and Kurtosis handles launching the network and executing the test. The nodes of the network are Docker containers, so anything you have a Docker image for can be used in Kurtosis!

How does it work?
-----------------
You'll implement a few interfaces in the language of your choice to create a test suite, package your suite into a Docker image, and run [the Kurtosis initializer Docker image](https://hub.docker.com/repository/docker/kurtosistech/kurtosis-core_initializer) with the name of your image. Kurtosis will load your test suite, coordinate the execution of the tests, and feed you back the results. It's that easy!

How do I create a test suite?
-----------------------------
### Overview
At a high level, you'll need to give Kurtosis details about:
1. The types of services your tests will use - what params to use when running the Docker container, how to verify the service is available, etc.
1. Your networks - what types and quantities of services they're composed of
1. Your tests - their names and logic, what networks they want, their timeouts, etc.

You'll tie these together with:
1. A thin CLI that will take in some Kurtosis-specific parameters and call the `RunTests` entrypoint with your test suite
1. A Dockerfile that receives several special Kurtosis environment variables and passes them to your CLI

The output product of a build on your test suite repo will be a Docker image containing your testsuite image, which you'll pass to Kurtosis to run.

### Details
At a lower level, the interfaces you'll need to implement are:
* `Service` to define the programmatic interface for your test to interact with the service(s) in your network
* `ServiceInitializerCore` to tell Kurtosis how to launch an instance of your service
* `ServiceAvailabilityCheckerCore` to tell Kurtosis how to verify your service is available
* `Network` to define the programmatic interface for your test to interact with the network as a whole
* `NetworkLoader` to tell Kurtosis how to construct an instance of your `Network` implementation
* `Test` to define a test and the `NetworkLoader` the test will use
* `TestSuite` to package your tests together

The client libraries we currently have built are:
* [Go](https://github.com/kurtosis-tech/kurtosis-go)

_If your language isn't on the list, let us know and we can discuss!_

Once you've defined your testsuite 


