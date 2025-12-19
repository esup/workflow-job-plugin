# Workflow Job Plugin - Architecture and Implementation Guide

## Quick Links
- [中文完整技术文档](./项目技术文档.md) - Comprehensive Chinese documentation
- [README](./README.md) - Project overview

## Overview

This plugin provides the fundamental job and build types for Jenkins Pipeline, serving as a key component of the Pipeline plugin suite.

## Core Components

### 1. WorkflowJob
- **Extends**: `Job<WorkflowJob,WorkflowRun>`
- **Purpose**: Represents a Pipeline job definition
- **Key Responsibilities**:
  - Manages FlowDefinition (pipeline script/config)
  - Handles job properties (concurrency, triggers, etc.)
  - Manages build scheduling and queuing
  - Persists job configuration

### 2. WorkflowRun
- **Extends**: `Run<WorkflowJob,WorkflowRun>`
- **Implements**: `FlowExecutionOwner.Executable`
- **Purpose**: Represents a single Pipeline build execution
- **Key Responsibilities**:
  - Manages FlowExecution lifecycle
  - Handles build console output
  - Supports build interruption and restart
  - Manages SCM changelogs

### 3. WorkflowJobProperty
- **Purpose**: Base class for job properties
- **Implementations**:
  - `DisableConcurrentBuildsJobProperty` - Controls concurrent builds
  - `PipelineTriggersJobProperty` - Manages build triggers
  - `DurabilityHintJobProperty` - Controls execution durability
  - `DisableResumeJobProperty` - Controls resume behavior

### 4. AfterRestartTask
- **Purpose**: Handles Pipeline recovery after Jenkins restart
- **Mechanism**: Automatically queues unfinished builds for resumption

## Architecture Patterns

### MixIn Pattern
```java
LazyBuildMixIn<WorkflowJob,WorkflowRun> buildMixIn;
```
- Provides lazy loading of builds
- Separates concerns through composition

### Strategy Pattern
```java
FlowDefinition definition;
```
- Allows different Pipeline definition types (CPS, Declarative, etc.)
- Runtime selection of execution strategy

### Observer Pattern
```java
FlowExecutionListener, GraphListener
```
- Monitors Pipeline execution events
- Enables extension through listeners

## Build Lifecycle

```
Job Creation → Configuration → Save → Trigger Build → Execute → Complete
     ↓             ↓            ↓          ↓            ↓          ↓
WorkflowJob   setDefinition  save()  scheduleBuild2  WorkflowRun  onEndNode
```

## Restart Recovery Mechanism

1. Jenkins restarts, loads all jobs from disk
2. `WorkflowRun.onLoad()` detects incomplete builds
3. Creates `AfterRestartTask` and queues it
4. Task executes, resuming the Pipeline from saved state
5. `FlowExecution` continues from the last persisted checkpoint

## Directory Structure

```
src/main/java/org/jenkinsci/plugins/workflow/job/
├── WorkflowJob.java              # Job type
├── WorkflowRun.java              # Build execution
├── WorkflowJobProperty.java      # Property base class
├── AfterRestartTask.java         # Restart handling
├── properties/                   # Job properties
│   ├── DisableConcurrentBuildsJobProperty.java
│   ├── PipelineTriggersJobProperty.java
│   └── DurabilityHintJobProperty.java
├── console/                      # Console annotations
│   ├── NewNodeConsoleNote.java
│   └── WorkflowRunConsoleNote.java
└── views/                        # UI actions
    ├── FlowGraphAction.java
    └── FlowGraphTableAction.java
```

## Building and Testing

```bash
# Build
mvn clean install

# Run tests
mvn test

# Start test Jenkins
mvn hpi:run

# Debug mode
mvnDebug hpi:run
```

## Extension Points

1. **WorkflowJobProperty** - Add custom job properties
2. **FlowDefinition** - Define new Pipeline types
3. **FlowExecutionListener** - Listen to execution events
4. **TransientActionFactory** - Add dynamic actions to runs

## Key Dependencies

- Jenkins Core 2.504.3+
- workflow-step-api
- workflow-api
- workflow-support

## Documentation

For detailed documentation in Chinese covering:
- Complete architecture analysis
- Implementation principles
- Usage guide with examples
- Advanced customization techniques

Please see: [项目技术文档.md](./项目技术文档.md)

## Resources

- [Plugin Page](https://plugins.jenkins.io/workflow-job/)
- [Source Code](https://github.com/jenkinsci/workflow-job-plugin)
- [Issue Tracker](https://issues.jenkins.io/)
- [Changelog](https://github.com/jenkinsci/workflow-job-plugin/releases)
