Testsuite
=========
### Overview
Every testsuite is simply a Docker image that runs a CLI that executes a test. The test execution logic for the CLI is encapsulated in [a Kurtosis client library in your language of choice](https://github.com/kurtosis-tech/kurtosis-docs/blob/master/supported-languages.md), so your main CLI function will only be a thin layer of arg-parsing, an instantiation of your suite object that contains test and testnet info, and a call to the Kurtosis client's `Client.Run` function. The components required to instantiate a testsuite are as follows:

![](./images/testsuite-architecture.md)

We recommend you follow along in code as you read this document. Each client library contains an example implementation of the components as well as the ability to bootstrap a new testsuite from the example implementation. The rest of the guide will assume you've bootstrapped a new testsuite, so if you haven't done so already we recommend following the [the quickstart instructions](./quickstart.md) to bootstrap now.

**NOTES:**
* Kurtosis provides clients for writing testsuites in multiple languages. While the languages differ, the objects and function calls are named the same. For consistency, this guide will avoid language-specific idioms and will use pseudocode with a generic notation `Object.function` notation.
* Each Kurtosis client provides comments on all objects and functions, so this guide will focus on the higher-level interaction between the components rather than specific documentation of each function and argument. For detailed docs, see the in-code comments in your client of choice.

### CLI Main Function
Your Kurtosis testsuite CLI in your repo will be a `main` function that does the following in order:

1. **Arg-Parsing:** Your CLI will receive several Kurtosis-specific arguments, which (with the exception of log level) will in turn be passed as-is to the `Client.run` function. You can also receive custom arguments specific to your testsuite (more on this later).
2. **Set Log Level:** Using the log level arg, set the logging level for the test's execution.
3. **TestSuite Instantiation:** The `Client.run` function needs details about the test logic to run as well as instructions on setting up the network required. These are contained in a `TestSuite` object, whose behaviour can be modified with custom arguments specific to your testsuite.
4. **Test Execution:** Calling `Client.run` with the Kurtosis arguments and `TestSuite` object.

### Dockerfile
To package the CLI into a Docker image, your repo will have a Dockerfile that defines how to build the image. (If Dockerfiles are alien to you, we recommend [the official Docker docs](https://docs.docker.com/get-started/) as a great place to start.) Kurtosis testsuite Dockerfiles are very simple, and simply compile and run the CLI. 

The only bit of complexity is that the Kurtosis-specific parameters will be passed in as environment variables, which will then be passed to your testsuite CLI in the form of flag args:

* `KURTOSIS_API_IP`
* `LOG_LEVEL`
* `METADATA_FILEPATH`
* `SERVICES_RELATIVE_DIRPATH`
* `TEST`

With the exception of the log level, all of these should be passed as-is to the `Client.run` call. If you modify the Dockerfile, you will need to make sure that you continue to receive these variables and pass them as-is to your CLI.

### TestSuite
Every Kurtosis client's `Client.Run` function requires a `TestSuite` object that contains details about:

* Your services - what Docker images to use, what params to give when running the container, how to verify the service is available, etc.
* Your networks - their topology and what types and quantities of services they're composed of
* Your tests - their names and logic, what networks they want, their timeouts, etc.

All this information is packaged inside the `Test` object, so a `TestSuite` is really a wrapper class for a set of `Test`s.

### Test
A `Test` object contains 
