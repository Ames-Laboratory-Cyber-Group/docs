# Overview

## View from 10,000 feet

On every execution:
   * For every type of test "Availability":
      * Execute test
      * Collect results:
         * If any node fails an availability test, mark
           it as down, note the failure, and do not run any
           further tests on that node in this execution.
   * For all remaining tests:
      * For all sites not marked as down in the availability step:
         * Execute all remaining tests for this node concurrently
         * Collect results
   * Aggregate all results and pass to logger

## Node Types

There are two kinds of nodes:

1. **Controller**. There is only one controller, and it instructs
   test runners when to run tests, which tests to run, and is reported
   back to by all test runners with results.
2. **Test Runner**. The test runners are dumb: they do whatever the
   controller tells them and report back results.

Communication is only done between the controller and the test runners.
No test runners talk to each other or know of each other's existence.

Note: this protocol may be extended in the future to support multiple
controllers.

## Test Cases

Test cases are declaratively specified and contain both required test inputs,
expected outputs, and associated test metadata.

### (Tentative) Test Case Format

Note: this format was made with little to no knowledge of what CFM protocol
messages look like and may need to be revised, possibly heavily.

A test case is composed of a number of tasks. There is exactly one
write task that begins a test case (a "push"), followed by a variable
number of read operations ("pulls"). A particular site can perform at most 1
read operation per test case.

A test case file is a list of the tasks that make up that test.

The format of a test case file is:

* At start, exactly once:
   * Type => Availability, Policy Compliance, Parsing
   * TaskNum => integer describing the number of tasks that make up this test
                case
* Tasks, TaskNum times
   * TestID => UUID
   * Action => push/pull (exactly one push allowed per test case)
   * Site => site that should run this test (exactly 1 pull allowed per site)
   * Input => task input
   * Expected response code => e.g. 200 OK
   * Expected output => expected task output (getting this output
                        means a "pass" for the test case)

## Message Format

Messages are used to communicate between nodes. A message consists of
a header and a payload.

### Header Format

A message header has the following fields:
   * Type: 1 byte => identifies the type of message this is
   * Payload Length: 2 bytes => Length, in bytes, of the message payload

The following types of messages are currently used:

### RunTask Message

Sent by: Controller
Purpose: Signal a test runner to run a particular task
Format:
   * TaskID: 32 bytes => UUID of the task to run
   * Timestamp: 8 bytes => the "new since" time to request data from server
                           (zeroed and ignored in push tasks)
   * Delay: 2 bytes => how long, in seconds, to wait before running test
                       (can be zero)

### RunningTask Message

Sent by: TestRunner
Purpose: Tell controller that a particular task is being run
Format:
   * TaskID: 32 bytes => UUID of the task being run

#### SendTask Message

Sent by: TestRunner
Purpose: Tell controller that test runner does not know about the
         requested task and the controller should send the task data.
Format:
   * TaskID: 32 bytes => UUID of the test being run

### TaskData

Sent by: Controller
Purpose: Tell a test runner that doesn't know about this task yet what
         the task actually is
Format:
   * TaskID: 32 bytes => UUID of the task being run
   * TaskData: N bytes => Serialized Task

### Result

Sent by: TestRunner
Purpose: Tell controller the results of a task
Format:
   * TaskID: 32 bytes => UUID of the task being run
   * StartTime: 8 bytes => timestamp when task run began
   * EndTime: 8 bytes => timestamp when task run finished
   * ResultData: M bytes => Serialized Result

### Error

Sent by: TestRunner
Purpose: Tell controller that a test runner had some sort of internal
         error that was *not* related to running the task (e.g. couldn't
         read a file due to bad permissions, received a malformed/unparseable
         message, etc.)
Format:
   * TaskID: 32 bytes => Task the test runner was trying to execute when
                         the error occured
   * Error code: 1 byte => Protocol error code
   * Message: Y bytes => human-readable string containing additional
                         info about the error (e.g. string from an exception)


## Detailed Protocol Description

Definitions:
   * TIMEOUT: the number of seconds the controller will wait before deciding
              a test runner is down
   * RESPONSEMAX: the number of seconds the controller will wait before
                  deciding that a task has failed
   * Available site: a site not marked as down during the Availability
                     testing phase
   * fn RunTask (task t) => executed by controller:
      * Send a RunTask message to the specified site
         * If the response is a RunningTask message:
            * wait for a Result message
         * If the response is a SendTask message:
            * serialize the task and send to the test runner
               * If the response is an Error message:
                  * mark the task as failed due to an internal testing error
               * If the response is a RunningTask message:
                  * wait for a Result message
               * If there is no response in TIMEOUT seconds, mark the node
                 as down
         * If the response is an Error message:
            * mark the task as failed due to an internal testing error
         * If there is no response in TIMEOUT seconds, mark the node
           as down
   * fn RunnerTask (task t) => executed by the test runner:
      * Store the current timestamp
      * Send a RunningTask message to the controller
      * Execute the task
      * Store the timestamp when the task finishes
      * Send a Result message to the controller
      * Discard the task

**Controller**:
   * Read all test case files
   * For every test case of type Availability:
      * For each push task *t* in each availability test:
         * execute RunTask(t)
      * **Wait** for all availability push tasks to complete:
         * If RESPONSE_MAX seconds pass without a received Result message,
           mark the task as a failure and mark the node as down
         * When a Result message is received:
            * Record the result
            * If the result is a failure:
               * mark the node as down
      * For all pull tasks t:
         * For all available sites:
            * execute RunTask(t)
               * If an Error is received:
                  * mark the task as failed due to an internal testing error
               * If the task fails:
                  * mark the node as down and record the failure
               * If there is no response in RESPONSEMAX seconds, mark the
                 node as down and record the failure
               * If success is received, record result
   * For all available sites:
      * For all remaining test cases:
         * For all push tasks t:
            * execute RunTask(t)
         * When **any** push task result is received:
            * If the result is a success:
               * Immediately execute RunTask(t) for *all* pull tasks remaining
                 in the test case
            * If the result is a failure:
               * Mark as a failure and discard pull tasks
   * Aggregate results and push to logging/alerting server

**Test Runner**

Test runners are stateless. They don't know when a protocol execution is
taking place, and they don't make any decisions. They simply listen for
instructions from the controller.

When a TestRunner receives a RunTask message for a task *t*:
   * Store Delay seconds
   * If it has a cached copy of the task:
      * Wait for Delay seconds
      * execute RunnerTask(t)
   * If it does not have a cached copy
      * Send a SendTask message to the controller
      * When a TestData test is received:
         * If it cannot parse the task
            * Sends an Error message to the controller and discard
              the pending task
         * If it parses correctly:
            * Cache t
            * Wait for Delay seconds
            * execute RunnerTask(t)
      * If a response is not received within RESPONSEMAX seconds, discard
        the pending task

## Security

### Transport

All controller--testrunner communication must be done over TLS.

#### Version

The controller and test runners must exclusively use TLS 1.2.

#### Certificate Authorities

A certificate authority should be created to issue and sign
certificates used by test runners and the controller.

#### Authentication

The controller must present a valid client certificate when connecting to
a test runner.

Test runners **must** drop all connections not authenticated by a client
certificate.

The controller and the test runners **must** only trust certificates
signed by the in-house certificate authority and **must** ignore certificates
from any other CA, including the operating system's trust store.

#### Ciphersuites

Currently, the only acceptable cipher suite should be:
`TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`

When implemented in OpenSSL, this should be upgraded to:
`TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256`

#### DH Params

Each site should use a randomly generated 4096 bit dh param value.
