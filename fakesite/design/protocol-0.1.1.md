# Overview

Fakesite Protocol Revision 0.1.1

## Changes from 0.1.0

* Design now assumes a *single* node
* The only network communication happens between the test node and the
  CFM servers
* Test cases are no longer declaratively specified. They are instead
  written as Python classes.

## Classes

### <a name="FSController"></a> FSController

**Purpose**:

**Fields**

**Methods**

### <a name="TestCase"></a> TestCase

**Purpose**:

**Fields**

**Methods**

#### <a name="AvailabilityTestCase"></a> AvailabilityTestCase

**Purpose**:

**Fields**

**Methods**

#### <a name="ComplianceTestCase"></a> ComplianceTestCase

**Purpose**:

**Fields**

**Methods**

#### <a name="IntegrityTestCase"></a> IntegrityTestCase

**Purpose**:

**Fields**

**Methods**

### <a name="Task"></a> Task

**Purpose**:

**Fields**

**Methods**

#### <a name="PushTask"></a> PushTask

**Purpose**:

**Fields**

**Methods**

#### <a name="PullTask"></a> PullTask

**Purpose**:

**Fields**

**Methods**

### <a name="TaskResult"></a> TaskResult

**Purpose**:

**Fields**

**Methods**

### <a name="TestCaseResult"></a> TestCaseResult

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
