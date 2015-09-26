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

**Inherits From**: Object

**Purpose**: There is exactly one FSController instance. The FSController has the following responsibilities:

* daemonize
* instantiate and run [TestCases](#TestCase)
* instantiate an [FSLogger](#FSLogger)
* instantiate an [FSIOCache](#FSIOCache)

**Fields**

**Methods**

### <a name="TestCase"></a> TestCase

**Purpose**: A TestCase is a collection of atomic [Tasks](#Task) that has the following responsibilities:

* enforce [test case task constraints](#testcase_contraints)
* instantiate and run [Tasks](#Task) in the correct order
* aggregate the [TaskResults](#TaskResult) into a [TestCaseResult](#TestCaseResult)

**Fields**

**Methods**

#### <a name="AvailabilityTestCase"></a> AvailabilityTestCase

**Inherits From**: [TestCase](#TestCase)

**Purpose**: 

**Fields**

**Methods**

#### <a name="ComplianceTestCase"></a> ComplianceTestCase

**Inherits From**: [TestCase](#TestCase)

**Purpose**:

**Fields**

**Methods**

#### <a name="IntegrityTestCase"></a> IntegrityTestCase

**Inherits From**: [TestCase](#TestCase)

**Purpose**:

**Fields**

**Methods**

### <a name="Task"></a> Task

**Purpose**:

**Fields**

**Methods**

#### <a name="PushTask"></a> PushTask

**Inherits From**: [Task](#Task)

**Purpose**:

**Fields**

**Methods**

#### <a name="PullTask"></a> PullTask

**Inherits From**: [Task](#Task)

**Purpose**:

**Fields**

**Methods**

### <a name="Result"></a> Result

**Purpose**:

**Fields**

**Methods**

### <a name="TaskResult"></a> TaskResult

**Inherits From**: [Result](#Result)

**Purpose**:

**Fields**

**Methods**

### <a name="TestCaseResult"></a> TestCaseResult

**Inherits From**: [Result](#Result)

**Purpose**:

**Fields**

**Methods**

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
