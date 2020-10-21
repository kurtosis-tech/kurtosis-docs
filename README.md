Kurtosis
========
Kurtosis is a platform for running whole-system tests against distributed systems with the frequency and repeatability of unit tests.

The world is moving to microservices and systems are becoming increasingly complex. The more components a system has, the more [emergent phenomena](https://en.wikipedia.org/wiki/Emergence) and [unexpected outlier events](https://en.wikipedia.org/wiki/Black_swan_theory) that occur. More components equal more difficulty running a representative system, and more corners cut when testing. If nothing is done, testing will continue to decline and unpredictability will continue to rise. Engineers need a new tool to tame the complexity of the distributed age, that marries the ease and safety of unit tests with the representativeness of testing in production. Kurtosis is that tool.

Diving In
---------
You have two paths you can start with:

* For those who like to jump in and see things running, head over to [the quickstart instructions](./quickstart.md)
* For those who prefer to start at a high level, check out [the Kurtosis architecture docs](./architecture.md)


Debugging Failed Tests
----------------------
See [the "Debugging Failed Tests" tutorial](./debugging-failed-tests.md) for information on how to approach some common failure scenarios.

Developer Notes
---------------
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
