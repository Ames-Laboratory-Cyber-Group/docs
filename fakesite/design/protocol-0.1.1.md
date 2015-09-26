## Overview

Fakesite Protocol Revision 0.1.1

### Changes from 0.1.0

* Design now assumes a *single* node
* The only network communication happens between the test node and the
  CFM servers
* Test cases are no longer declaratively specified. They are instead
  written as Python classes.

### Classes

#### FSController

#### PushQueue

#### PullQueue

#### TestCase

##### AvailabilityTestCase

##### ComplianceTestCase

##### IntegrityTestCase

#### Task

##### PushTask

##### PullTask

#### TaskResult

#### TestCaseResult

#### Site
