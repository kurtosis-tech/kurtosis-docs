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

**NOTE:** Kurtosis testsuites can be written in any language. This tutorial will avoid language-specific idioms and use a pseudocode `Objection.function` notation.

### Networks & services in a test
You're writing a Kurtosis testsuite because you want to write tests for a network. This means that your test needs to have a way of interacting with the network. To this end, every Kurtosis test is just a function that takes in `Network` and `TestContext` objects as parameters, makes calls against the network, and uses the `TestContext` object to make assertions about the results (e.g. `TestContext.assertTrue`).

Kurtosis needs to be told what your network looks like, so `Network` is an interface that you can define with whatever methods are easiest for your testsuite. For example, if you were running tests against a simple network composed of two Elasticsearch nodes, the `Network` interface that gets passed to your test might have methods like `getElasticsearchServiceOne` and `getElasticsearchServiceTwo` which return Elasticsearch client instances. To interact with your network, your test would call these methods, get a client, make the necessary calls, and use the `TestContext` to make assertions about the result.

Networks are composed of services, and Kurtosis likewise needs to be told what your services look like. 

Next steps
----------
Now that you have a running testsuite, you can:

* Visit [the Kurtosis testsuite docs](./testsuite-details.md) to learn more about what's in a testsuite and how to customize it to your needs
* Check out [the CI documentation](./running-in-ci.md) to learn how to run Kurtosis in your CI environment
* Take a step back and visit [the Kurtosis architecture docs](./architecture.md) to get the big picture
