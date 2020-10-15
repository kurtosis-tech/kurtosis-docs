Quickstart
==========

### Prerequisites
Ensure you have [Docker engine](https://docs.docker.com/get-started/) >= 2.3.0.0 (we've seen 2.0.0.0 behaving erratically)

### Bootstrapping A Test Suite
1. Visit [the list of supported language clients](https://github.com/kurtosis-tech/kurtosis-docs/blob/master/supported-languages.md)
1. Pick your favorite, and clone it to your local machine
1. Run the `scripts/bootstrap.sh` script from inside the cloned repo
1. Follow the language-specific instructions in the new `README.md` file

### Running The Test Suite
From the root directory of your bootstrapped repo: 

1. `git add .` to stage all the files you've changed
1. `git commit -m "Init commit"` to commit the files (don't skip this - it's needed to run the test suite!)
1. Run `scripts/build_and_run.sh all` to run the test suite

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

### Next Steps
Now that you have a running test suite, you can either visit [the Kurtosis test suite docs](TODO TODO TODO) to learn more about what's in a test suite and how to customize it, or take a step back and visit [the Kurtosis architecture docs](TODO TODO TODO) to get the big picture. Happy testing!
