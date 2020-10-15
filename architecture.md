Kurtosis Architecture
=====================
### Problem Statement
Every test in existence goes through four phases:

1. Setup
2. Perturbation
3. Validation
4. Teardown

In unit testing, this usually looks like:

1. Instantiate the object to test and mock all surrounding objects
2. Execute the functions being tested
3. Assert the expected result
4. Garbage collect all objects

Each phase is easy to write, so developers write lots of unit tests. 

When the component being tested is an entire distributed system, however, things become seriously difficult: where do we run the test nodes? How do we initialize their config & data files? Where do we run the test logic, and how does it talk to the system? How do we ensure the network is cleaned up after the test is done?

These are challenging questions, and in our experience teams tend to go one of two ways - write their own in-house automation framework for launching test networks from scratch, or skip sandboxed networks altogether and have a single test network that everyone tests against. The former is bespoke and costly to write, and the latter is prone to error and costly to maintain.

### Solution
Kurtosis rejects both these approaches, and provides a platform for defining 


When we do testing, we want to spin up 
At the core of testing is the test itself. In Kurtosis, a test is a function 


To give engineers the ability to run whole-system tests on distributed systems with the ease of unit tests, Kurtosis needs several pieces of machinery:

* An entrypoint for running a test suite
* 
