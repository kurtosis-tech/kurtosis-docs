Kurtosis
========
Kurtosis is a platform for running whole-system tests against distributed systems with the frequency and repeatability of unit tests.

The world is moving to microservices and systems are becoming increasingly complex. The more components a system has, the more [emergent phenomena](https://en.wikipedia.org/wiki/Emergence) and [unexpected outlier events](https://en.wikipedia.org/wiki/Black_swan_theory) that occur. More components equal more difficulty running a representative system, and more corners cut when testing. If nothing is done, testing will continue to decline and unpredictability will continue to rise. Engineers need new tools to tame the complexity of the distributed age, that marry the ease and safety of unit tests with the representativeness of testing in production. Kurtosis is one such tool.

Diving In
=========
From here, you have two paths:

* For those who like to jump in and see things running,     head over to [the quickstart instructions](./quickstart.md)
* For those who prefer to start at a high level, check out [the Kurtosis architecture docs](./architecture.md)

How does it work?
=================
You'll implement a few interfaces in the language of your choice to create a test suite, package your suite into a Docker image, and run [the Kurtosis initializer Docker image](https://hub.docker.com/repository/docker/kurtosistech/kurtosis-core_initializer) with the name of your image like so. Kurtosis will load your test suite, coordinate the execution of the tests, and feed you back the results - it's that easy!

How do I create a test suite?
=============================
Overview
--------
At a high level, you'll need to give Kurtosis details about:

* The types of services your tests will use - what params to use when running the Docker container, how to verify the service is available, etc.
* Your networks - what types and quantities of services they're composed of
* Your tests - their names and logic, what networks they want, their timeouts, etc.

This is done by implementing the interfaces in the "Kurtosis client" library in [the language of your choice](./supported-languages.md). To use the interfaces, you'll then write:

* A thin CLI that receives some Kurtosis-specific parameters, instantiates your test suite, and calls the `RunTests` entrypoint from your Kurtosis client library
* A Dockerfile that receives several special Kurtosis environment variables and passes them as arguments to your CLI

The output of a build on your test suite repo will be a Docker image containing your test suite image, which you'll use to run your test suite on Kurtosis.

Details
-------
At a lower level, the Kurtosis client interfaces you'll need to implement are:

* `Service` to define the programmatic interface for the service(s) in your network so your test can use them
* `ServiceInitializerCore` to tell Kurtosis how to launch an instance of your service
* `ServiceAvailabilityCheckerCore` to tell Kurtosis how to verify your service is available
* `Network` to define the programmatic interface for your test to interact with the network as a whole
* `NetworkLoader` to tell Kurtosis how to construct an instance of your `Network` implementation
* `Test` to define a test and the `NetworkLoader` the test will use
* `TestSuite` to package your tests together

This will yield your own custom instance of `TestSuite`, which is half of what you need to call the `RunTests` method. The other half is the parameters that Kurtosis needs to run, which Kurtosis will be set as Docker environment variables when your test suite image is run. These are the parameters your Dockerfile should receive:

* `METADATA_FILEPATH`
* `TEST`
* `KURTOSIS_API_IP`
* `SERVICES_RELATIVE_DIRPATH`
* `LOG_FILEPATH`
* `LOG_LEVEL`

With the exception of `LOG_LEVEL`, your Dockerfile should pass the values of these environment variables as-is to your CLI, which should in turn pass the values to the `RunTests` entrypoint. 

`LOG_LEVEL` is a special, optional variable indicating the log level that your test suite image should output at. However, Kurtosis can't know what logging framework or log levels your test suite image uses, so it will be up to your test suite image to interpret this string as you please. The source of this value will come from you, when you call the Kurtosis initializer image with your test suite (we'll see this in detail later).

By this point, you should have:
* Implementations of all the interfaces in your Kurtosis client library of choice
* A CLI that wraps your Kurtosis client's `RunTests` method
* A Dockerfile that builds an image which receives the special Kurtosis environment variables and passes them as arguments to your CLI

Examples
--------
* [Go example implementation](https://github.com/kurtosis-tech/kurtosis-go/tree/develop/example_impl)

How do I run my test suite?
===========================
Overview
--------
The code contained in the Kurtosis initializer Docker image serves as the "CLI" for running Kurtosis. You'll run the initializer with your test suite like so:

```bash
# The name of the release channel of Kurtosis that you'll be using, which indicates which Kurtosis Docker images you'll be using
KURTOSIS_RELEASE_CHANNEL="master"

# The name of your test suite image that you built earlier
TEST_SUITE_IMAGE="my-image-name"

# Kurtosis needs a Docker volume to store its data in
# To learn more about volumes, see: https://docs.docker.com/storage/volumes/
VOLUME_NAME="my-test-suite_$(date +%s)"
docker volume create "${VOLUME_NAME}"

# The directory where Kurtosis will store files it uses in between executions, e.g. access tokens
KURTOSIS_DIRPATH="${HOME}/.kurtosis"
mkdir -p "${KURTOSIS_DIRPATH}"

docker run \
    `# The Kurtosis initializer runs inside a Docker container, but needs to access to the Docker engine; this is how to do it` \
    `# For more info, see the bottom of: http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/` \
    --mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \

    `# Because the Kurtosis initializer runs inside Docker but needs to persist & read files on the host filesystem between execution,`
    `#  the container expects the Kurtosis directory to be bind-mounted at the special "/kurtosis" path`
    --mount "type=bind,source=${HOME}/.kurtosis,target=/kurtosis" \

    `# The Kurtosis initializer image requires the volume to be mounted at the special "/suite-execution" path` \
    --mount "type=volume,source=${VOLUME_NAME},target=/suite-execution" \

    `# Tell the initializer which test suite image to use` \
    --env "TEST_SUITE_IMAGE=${TEST_SUITE_IMAGE}" \

    `# Tell the initializer the name of the volume to store data in, so it can mount it on new Docker containers it creates` \
    --env "SUITE_EXECUTION_VOLUME=${VOLUME_NAME}" \

    `# The initializer needs a special Kurtosis API image to operate` \
    `# The release channel here should match the release channel of the initializer itself` \
    --env "KURTOSIS_API_IMAGE=kurtosistech/kurtosis-core_api:${KURTOSIS_RELEASE_CHANNEL}" \

    kurtosistech/kurtosis-core_initializer:${KURTOSIS_RELEASE_CHANNEL}
```

Details
-------
Because Kurtosis runs entirely in Docker, we use the `--env` flag to the `docker run` command to set the `TEST_SUITE_IMAGE`, `SUITE_EXECUTION_VOLUME`, and `KURTOSIS_API_IMAGE` "parameters" to control Kurtosis behaviour. The full list of Docker environment variable "parameters" that can be set (in the form `--env NAME=value`) is as follows:

| Parameter     | Required/Optional | Description |
| ------------- | ----------------- | ----------- |
| `CLIENT_ID` | Optional | An Oauth client ID, which should be provided only when running the testsuite on a CI machine |
| `CLIENT_SECRET` | Optional | An Oauth client secret, which should be provided only when running the testsuite on a CI machine |
| `CUSTOM_ENV_VARS` | Optional | A key-value mapping in JSON object form of additional Docker environment variables that should be set when running your test suite image. For example, if your test suite Dockerfile wants an extra `SOME_CUSTOM_ENV_VAR` variable that it then passes to your CLI, you'd add a `--env 'CUSTOM_ENV_VARS={"SOME_CUSTOM_ENV_VAR": "some value"}'` flag when calling the Kurtosis initializer. |
| `DO_LIST` | Optional; default `false` | A boolean variable indicating if Kurtosis should list the names of the tests in the test suite rather than executing any tests. |
| `KURTOSIS_API_IMAGE` | Required | A Docker image from [the Kurtosis API image repo](https://hub.docker.com/repository/docker/kurtosistech/kurtosis-core_api) that the initializer should use during operation. The tag of the API image should match the tag of the initializer image. |
| `KURTOSIS_LOG_LEVEL` | Optional; default `debug` | The log level that all output generated by the Kurtosis framework itself should log at. Acceptable values are `trace`, `debug`, `info`, `warn`, `error`, and `fatal`. |
| `PARALLELISM` | Optional; default `4` | A positive integer variable telling Kurtosis how many tests it should run in parallel. **This should be set no higher than the number of cores on your machine, else you'll slow down your tests and potentially hit test timeouts!** |
| `SUITE_EXECUTION_VOLUME` | Required | The name of the Docker volume in which this execution of Kurtosis should store its data. **Use a new volume every time!** |
| `TEST_NAMES` | Optional | A comma-separated list of specific test names to run. If this option is not specified, all tests in the test suite are run. |
| `TEST_SUITE_IMAGE` | Required | The name of the Docker image containing your test suite. |
| `TEST_SUITE_LOG_LEVEL` | Optional; default `debug` | A string that will be passed as-is to the test suite container to indicate what log level the test suite container should output at. Kurtosis won't know what logging framework the test suite container uses, so this string should be meaningful to the test suite image. |

Debugging Failed Tests
======================
See [the "Debugging Failed Tests" tutorial](./debugging-failed-tests.md) for information on how to approach some common failure scenarios.

Developer Notes
===============

### Abnormal Exit
While running, Kurtosis will create several Docker containers and networks during test execution. In most situations, Kurtosis will stop these containers and delete the networks on exit (including Ctrl-C). **In some rare situations when Kurtosis is killed abnormally (e.g. SIGKILL aka `kill -9`), you'll need to remove the Docker network and stop the running containers!** This can be done like so:

Find & remove Kurtosis Docker networks:
```
docker network ls  # See which Docker networks are left around - will be in the format of UUID-TESTNAME
docker network rm some_id_1 some_id_2 ...
```

**If the network isn't removed, you'll get IP conflict errors from Docker on next Kurtosis run!**

Stop running containers (replace `YOUR-IMAGE-NAME` with the name of the image of the containers you want to remove):
```
docker container ls    # See which Docker containers are left around - these will depend on the containers spun up
docker stop $(docker ps -a --quiet --filter ancestor="YOUR-IMAGE-NAME" --format="{{.ID}}")
```
