# Overview

Fakesite Protocol Revision 0.1.1

## Changes from 0.1.0

* Design now assumes a *single* node
* The only network communication happens between the test node and the
  CFM servers and the logging server
* Test cases are no longer declaratively specified. They are instead
  written as Python classes.

## 0.1.1 Sample Execution

Pseudocode for a sample protocol execution:

```
FSController:
  create logger
  create cache
  run availability tests
  log results
  filter remaining tests to remove tests that use a site that failed an availability test
  run surviving tests
  log results
  
Every TestCase:
  check constraints
  run push task
  if push task fails, format and return a failed TestCaseResult
  else run all pull tasks
  aggregate results and return a TestCaseResult
  
Every Task:
  Execute when started (return failure if we timeout)
  Map server response to TaskResult using FSIO.equals() and return
  
  PullTask caching ops:
    First try cache.get()
    If that fails, pull FSIO from timestamp
    call cache.put() with resulting FSIO(s) (if successful)
```

## Classes

### <a name="FSController"></a> FSController

**Purpose**: There is exactly one FSController instance. The FSController has the following responsibilities:

* instantiate and run [TestCases](#TestCase)
* instantiate an [FSLogger](#FSLogger)
* instantiate an [FSIOCache](#FSIOCache)

**Fields**

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| runTests | Execute all known tests | - | defer.DeferredList([TestCaseResult](#TestCaseResult)) |
| runTest | Execute the specified test | [TestCase](#TestCase) | defer.Deferred([TestCaseResult](#TestCaseResult)) |

### <a name="TestCase"></a> TestCase

**Purpose**: A TestCase is a collection of atomic [Tasks](#Task) that has the following responsibilities:

* enforce [test case task constraints](#testcase_contraints)
* instantiate and run [Tasks](#Task) in the correct order
* aggregate the [TaskResults](#TaskResult) into a [TestCaseResult](#TestCaseResult)

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| tasks | list | a list of the tasks that make up this test case |

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| start | Execute the test case | - | defer.Deferred([TestCaseResult](#TestCaseResult)) |
| stop | Stop the test case execution | - | - |


### <a name="Task"></a> Task

**Purpose**: Describe a particular Task, the base component of test cases. A Task knows how to execute itself and how to make a [TaskResult](#TaskResult) describing the result of execution.

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| site | [Site](#Site) | The site this task executes as |
| input | [FSIO](#FSIO) | Input from task -> CFM server |
| expected_output | [FSIO](#FSIO) | Expected output from CFM server |

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| start | Execute the task | - | defer.Deferred([TaskResult](#TaskResult)) |
| stop | Stop the currently executing task | - | - |


#### <a name="PushTask"></a> PushTask

**Inherits From**: [Task](#Task)

**Purpose**: Execute a given push task

#### <a name="PullTask"></a> PullTask

**Inherits From**: [Task](#Task)

**Purpose**: Execute a given pull task

### <a name="TaskResult"></a> TaskResult

**Purpose**: Describe the result of a [Task](#Task)

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| started | int | timestamp when this task began |
| finished | int | timestamp when this task finished |
| payload_size | int | size, in bytes, of the payload for this task |
| site | [Site](#Site) | site that executed this task |
| code | int | HTTP code returned |
| desc | str | human-readable string describing result |
| UUID | int | UUID for task this result describes |
| result | enum | Describe actual result (e.g pass/fail/timeout/err) |

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| toSplunkFormat | return this task result formatted for a splunk log | - | str |

### <a name="TestCaseResult"></a> TestCaseResult

**Purpose**: Describe the result of a [TestCase](#TestCase)

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| started | int | timestamp when this {task|test} began |
| finished | int | timestamp when this {task|test} finished |
| UUID | int | UUID of the test case this result describes |
| result | enum | Describe the result of this test case (e.g. pass/fail/timeout/err/partial) |

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| toSplunkFormat | return this test case result formatted for a splunk log | - | str |

### <a name="Site"></a>Site

**Purpose**: Describe a particular site

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| name | str | this site's name |
| gpg_keypair | ? | keypair for this site |
| uname | str | username this site uses |
| passwd | str | passwd this site uses |

### <a name="FSIO"></a> FSIO

**Purpose**: Describe a discrete block of FakeSite IO

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| raw | str | the raw input used to create this FSIO |
| UUID | int | UUID for this FSIO |
| code | int | HTTP status code |

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| make | Construct an FSIO from raw data | raw | [FSIO](#FSIO) |
| equals | Test for equality with another FSIO | other, semantic_eq=False | bool |

### <a name="FSIOCache"> FSIOCache

**Purpose**: Cache FSIO objects

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| max_size | int | Upper bound on cache size|

**Public Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| get | Get a cached FSIO | UUID | [FSIO](#FSIO) |
| put | Cache an FSIO | - | - |

### <a name="FSLogger"></a> FSLogger

**Purpose**: Log a [TestCaseResult](#TestCaseResult) to a specific logging endpoint

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| endpoint | namedtuple | location of the remote logging endpoint |

**Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| log | Log a formatted test case result to endpoint | [TestCaseResult](#TestCaseResult) | - |

## <a name="testcase_constraints"></a> TestCase Constraints

The following constraints are imposed on test cases:

1. There must be exactly 1 push task
2. A particular site can execute at most 1 pull task
