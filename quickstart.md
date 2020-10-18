Quickstart
==========

### Prerequisites
Ensure you have [Docker engine](https://docs.docker.com/get-started/) >= 2.3.0.0 (we've seen 2.0.0.0 behaving erratically)

### Bootstrap your testsuite
1. Visit [the list of supported language clients](https://github.com/kurtosis-tech/kurtosis-docs/blob/master/supported-languages.md)
1. Pick your favorite, and clone the repo to your local machine
1. Run the `bootstrap/bootstrap.sh` script from inside the cloned repo
1. Follow the language-specific instructions in the new `README.md` file

### Run your testsuite
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

### Next steps
Now that you have a running testsuite, you can either visit [the Kurtosis testsuite docs](./testsuite-details.md) to learn more about what's in a testsuite and how to customize it to your needs, or take a step back and visit [the Kurtosis architecture docs](./architecture.md) to get the big picture. Happy testing!
