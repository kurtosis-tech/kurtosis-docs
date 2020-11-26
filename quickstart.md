Quickstart
==========

Prerequisites
-------------
* A [Kurtosis user account](https://www.kurtosistech.com/sign-up)
* A [Docker engine](https://docs.docker.com/get-started/) with version >= 2.3.0.0

Bootstrap your testsuite
------------------------
1. Visit [the list of supported language clients](https://github.com/kurtosis-tech/kurtosis-docs/blob/master/supported-languages.md)
1. Pick your favorite, and clone the repo to your local machine
1. Run the `bootstrap/bootstrap.sh` script from inside the cloned repo
1. Follow the language-specific instructions in the new `README.md` file

Run your testsuite
------------------
From the root directory of your bootstrapped repo: 

1. `git init` to initialize your new repo
1. `git add .` to stage all the files you've changed
1. `git commit -m "Init commit"` to commit the files (don't skip this - it's needed to run the testsuite!)
1. Run `scripts/build_and_run.sh all` to run the testsuite

If everything worked, you should see an output that looks something like this:

```
INFO[2020-10-15T18:43:27Z] ==================================================================================================
INFO[2020-10-15T18:43:27Z]                                      TEST RESULTS
INFO[2020-10-15T18:43:27Z] ==================================================================================================
INFO[2020-10-15T18:43:27Z] - singleNodeExampleTest: PASSED
INFO[2020-10-15T18:43:27Z] - fixedSizeExampleTest: PASSED
```

(This output is from the Kurtosis Go client; other languages might be slightly different)

If you don't see success messages, check out [the guide for debugging failed tests](./debugging-failed-tests.md) which contains solutions to common issues.

Customize your testsuite
------------------------
Now that you have a running testsuite, you'll want to start customizing the testsuite to your needs. Here we'll walk through the components inside a testsuite, as well as how to customize them.

**NOTE:** Kurtosis testsuites can be written in any language. This tutorial will avoid language-specific idioms and use a Java-like pseudocode notation in this documentation, but will reference the example in [the Go implementation](https://github.com/kurtosis-tech/kurtosis-go) to illustrate. All object names and methods will be more or less the same in your language of choice, and [all language repos](./supported-languages.md) come with an example.

### Service Interface & Implementation
You're writing a Kurtosis testsuite because you want to write tests for a network, and networks are composed of services. To Kurtosis, a service is an implementation of the `Service` interface that contains the methods a test will call. To tell Kurtosis what your particular service can do, write an interface that extends the `Service` interface [like this](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_services/example_service.go#L12). This interface is yours to write, so you can put whatever methods on it that will be easiest for your tests.

This gives us an interface that a test could use to interact with services in our network, but we still need to provide Kurtosis with an implementation. The `ExampleService` interface yields the IP address and port, so the implementation of the interface needs to return the IP address of the Docker container and the port our service is listening on inside the container. Looking at [the implementation](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_services/example_service_impl.go#L11), we can see that it does exactly this.

### Service Initializer Core
Our tests now have a nice interface for interacting with a service we represent the reality that the service is backed by a Docker container with an IP address, listening on a port. Next, we need to give Kurtosis the details on how to actually start a Docker container running one of our services, which we do by implementing the `ServiceInitializerCore` interface. This interface has several functions which will be very well-documented on the interface itself in your language, so we can use the documentation to write a service initializer core for our service [like in this example](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_services/example_service_initializer_core.go#L17).

Note the service "dependencies" that show up in the example. Kurtosis knows that some services will depend on others, and gives you the option to modify a service's files and start command based on other services in the network. We'll see how to declare these dependencies later.

### Service Availability Checker Core
Since a Docker container being up doesn't necessarily mean that the service inside is running and since we don't want to run a test against a network of services that are still starting up, the last piece we need for our service is a way to tell Kurtosis when the service is actually available for use. We'll therefore implement the `ServiceAvailabilityCheckerCore` interface [like we do here](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_services/example_availability_checker_core.go#L16), so Kurtosis knows when it's okay to run a test. 

The `IsServiceUp` method will be used to inform Kurtosis about when a particular service instance is actually available to ensure tests aren't run before the whole test network is available. During network setup, Kurtosis will continually call the `IsServiceUp` method until either it returns true or the timeout defined by `GetTimeout` is reached. Note that the service's dependencies are given as arguments to `IsServiceUp` but aren't used here - Kurtosis passes in a service's dependencies as an argument in case our service's availability is contingent on the state of its dependencies, but this won't be used in most distributed systems where nodes simply won't report themselves available until they successfully connect to their dependencies.

### Test Network
We're all set up to use this service in a network now... but of course, we still need to define what that network looks like. Kurtosis has a `ServiceNetwork` object that represents the underlying state of the test network, but interacting with it is often too low-level for writing clean tests. Just like with services, Kurtosis lets the you define an arbitrary object (an implementation of the `Network` interface) to represent your network - whatever is easiest for your tests. This custom network representation will be a parameter to your test logic, and wraps the low-level `ServiceNetwork` from Kurtosis. We can see this in action in the Go implementation for a network of a fixed size [here](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_networks/fixed_size_example_network/fixed_size_example_network.go#L22).

### Test Network Loader
Just like with services, we need to tell Kurtosis how to actually initialize our custom network representation. With services, we implemented a `ServiceInitializerCore`; with networks, we'll need to implement the `NetworkLoader` interface. Before we write our implementation though, it's worth understanding how Kurtosis networks are configured. Each network has one or more **service configurations**, which serve as templates for the service instances that will comprise the network. These service configurations are defined by a configuration ID, a Docker image, a `ServiceInitializerCore` implementation, and a `ServiceAvailabilityChecker` implementation. If a network is composed of only one type of service then the network only needs one configuration; if a network is made up of many different types of services then it will need many configurations. By way of example, the Go implementation for our fixed-sized network from above can be found [here](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_networks/fixed_size_example_network/fixed_size_example_network.go#L44).

### Test Suite
The heavy lifting is finally done - we've declared a service and the way to initialize it, a network composed of that service, and a loader to wrap the low-level Kurtosis representation with a simpler, test-friendly version. Now we can write some tests!

A test suite is a package of named tests, and a test is just a function that takes in `Network` and `TestContext` objects as parameters, makes calls against the network, and uses the `TestContext` object to make assertions about the results. To write a test, all we need to do is implement the `Test.run` method [like this](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_testsuite/fixed_size_example_test_.go#L27). It's very common that your test will need some amount of parameterization (at minimum, the name of the Docker image being tested); you can define that now, and we'll see how those variables get passed in in a bit.

Once you have your test, it can be packaged into a `TestSuite` implementation like [this](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/example_testsuite/example_testsuite.go#L21). Make sure to thread any parameterization for your tests through your `TestSuite` implementation as well.

### Main Function
With your testsuite complete, your only remaining step is to make sure it's getting used. When your testsuite bootstrapped, you will have received an entrypoint main function that receives several flags, instantiates a testsuite, and passes that to the Kurtosis client [like this Go example](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/main.go#L17). You will also have received a Dockerfile, for packaging that main CLI into a Docker image ([Go example](https://github.com/kurtosis-tech/kurtosis-go/blob/a278ad34d94e9c7968e3108cee8ed513b3eb39fd/example_impl/Dockerfile#L14)). What `build_and_run.sh` actually does is during its "build" phase is compile the main entrypoint CLI and package it into an executable Docker image. In order for your testsuite to get run, make sure the entrypoint CLI is using your testsuite.

You now have a custom testsuite running using Kurtosis!

### Custom Parameterization
You'll notice that one of the flags that the Go entrypoint CLI above receives is `serviceImageArg`, which defines the name of the Docker image to use in the services in the network. This a **custom parameter** - a parameter not used by Kurtosis itself but by the testsuite. Your testsuite might also need additional parameters. To pipe these through, you'll need to:

1. Add another flag to the main CLI and pass it to your testsuite at instantion
1. Modify the Dockerfile to set the flag value using an environment variable

These environment variables in the Dockerfile might seem like magic, but they're just set by Kurtosis when it launches the Docker image containing your testsuite. You'll need to tell Kurtosis what values to use for your custom flags, which can be done at time of launch with the `CUSTOM_ENV_VARS_JSON` parameter to the Kurtosis initializer. For example, if your Dockerfile uses the `MY_CUSTOM_VAR` environment variable then you might call `build_and_run.sh` with `--env CUSTOM_ENV_VARS_JSON="{\"MY_CUSTOM_ENV_VAR\":5}"`.

Next steps
----------
Now that you have your own custom testsuite running, you can:

* Take a look at [the testsuite for the Avalanche (AVAX) token](https://github.com/ava-labs/avalanche-testing), a real-world Kurtosis use-case 
* Visit [the Kurtosis testsuite docs](./testsuite-details.md) to learn more about what's in a testsuite and how to customize it to your needs
* Check out [the CI documentation](./running-in-ci.md) to learn how to run Kurtosis in your CI environment
* Take a step back and visit [the Kurtosis architecture docs](./architecture.md) to get the big picture
