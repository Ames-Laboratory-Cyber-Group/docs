# Overview

Fakesite Protocol Revision 0.1.1

## Changes from 0.1.0

* Design now assumes a *single* node
* The only network communication happens between the test node and the
  CFM servers and the logging server
* Test cases are no longer declaratively specified. They are instead
  written as Python classes.

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
| runTests | Execute all known tests | - | defer.Deferred({[TestCase](#TestCase): [TestCaseResult](#TestCaseResult)}) |
| runTest | Execute the specified test | [TestCase](#TestCase) | defer.Deferred([TestCaseResult](#TestCaseResult)) |
| logResults | Log test results | {[TestCase](#TestCase): [TestCaseResult](#TestCaseResult)} | - |

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

**Purpose**:

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| site | [Site](#Site) | The site this task executes as |
| input | [FSIO](#FSIO) | Input from task -> CFM server |
| expected_output | [FSIO](#FSIO) | Expected output from CFM server |

**Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| start | Execute the task | - | defer.Deferred([TaskResult](#TaskResult)) |
| stop | Stop the currently executing task | - | - |


#### <a name="PushTask"></a> PushTask

**Inherits From**: [Task](#Task)

**Purpose**:

**Fields**

**Public Methods**


#### <a name="PullTask"></a> PullTask

**Inherits From**: [Task](#Task)

**Purpose**:

**Fields**

**Methods**

### <a name="TaskResult"></a> TaskResult

**Purpose**:

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| started | int | timestamp when this {task|test} began |
| finished | int | timestamp when this {task|test} finished |

**Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| toSplunkFormat | return this task result formatted for a splunk log | - | str |

### <a name="TestCaseResult"></a> TestCaseResult

**Purpose**:

**Fields**

|Name|Type|Description|
|:---|:---|:----------|
| started | int | timestamp when this {task|test} began |
| finished | int | timestamp when this {task|test} finished |

**Methods**

|Name|Purpose|Arguments|Return Value|
|:---|:------|:-------:|:-----------|
| toSplunkFormat | return this test case result formatted for a splunk log | - | str |

### <a name="Site"></a>Site

**Purpose**:

**Fields**

**Methods**

### <a name="FSIO"></a> FSIO

**Purpose**:

**Fields**

**Methods**

### <a name="FSIOCache"> FSIOCache

**Purpose**:

**Fields**

**Methods**

### <a name="FSLogger"></a> FSLogger

**Purpose**:

**Fields**

**Methods**
