# Introduction

Building continuous delivery pipelines and similarly complex tasks in Jenkins using freestyle projects and existing plugins is very awkward.
You need to mix Parameterized Trigger, Copy Artifact, Promoted Builds, Conditional Build Step, and more just to express what should be a simple script.
This project attempts to make it possible to directly write that script, what people often call a _workflow_ (sometimes abbreviated _flow_), while integrating with Jenkins features like slaves and publishers.

[JUC Boston slides](http://www.cloudbees.com/sites/default/files/juc/juc2014/boston/2014-0618-Boston-Jesse_Glick-Workflow.pdf)

[Early slides](https://docs.google.com/a/cloudbees.com/presentation/d/1ysu71kGpEjvsikKAXdPXTJULadHh9cRbpd0gJaIkVtA)

# Core features

Not all implemented yet, of course; see below for status.

## Scripted control flow

Your whole workflow is a single Groovy script using an embedded DSL, possibly quite short and legible; there is no need to jump between multiple job configuration screens to see what is going on.
Conditions, loops, variables, parallel tasks, and so on are defined using regular language constructs.
At any point you can insert a shell script to do “real work” (compilation, etc.).

## Jenkins model object integration

Standard DSL functions let you run external processes, grab slave nodes, allocate workspaces, build “legacy” (freestyle) jobs, and so on.

## Pause and resume execution

If Jenkins is restarted (intentionally, or because of a crash) while your workflow is running, when it comes back up, execution is resumed where it left off.
This applies to external processes (shell scripts) so long as the slave can be reattached, and losing the slave connection temporarily is not fatal either.

Flows can pause in the middle and wait for a human to approve something, or enter some information.
Executors are not consumed while the flow is waiting.

## Example script

```
with.node('linux') { // grab a slave and allocate a workspace
  sh('git clone …');
  sh('mvn verify');
}
```

# Installation

If you do not want to build from sources, some early releases (as of this writing `0.1-beta-1`) are available on the Jenkins experimental update center.
You need to be running a recent Jenkins weekly release, currently 1.568 or newer.
For OSS Jenkins users, follow [these instructions](http://jenkins-ci.org/content/experimental-plugins-update-center) and install _Workflow: Aggregator_ (its dependencies will be pulled in automatically).
Jenkins Enterprise by CloudBees users can click _Enable access_ under _Access to experimental plugin releases_ in the main Jenkins configuration screen, and then install both _Workflow: Aggregator_ and the proprietary add-on _Workflow: Groovy CPS Execution Checkpoint_.

There is a [demo](demo/README.md) using Docker available if you want to try a complete setup quickly.

# Development

* [Source repository](https://github.com/jenkinsci/workflow-plugin)
* [JIRA](https://issues.jenkins-ci.org/secure/IssueNavigator.jspa?reset=true&jqlQuery=project+%3D+JENKINS+AND+resolution+%3D+Unresolved+AND+component+%3D+workflow+ORDER+BY+priority+DESC&mode=hide)
* [Trello board](https://trello.com/b/u2fJQnDX/workflow) tracking active and proposed tasks.
* [CI job](https://jenkins.ci.cloudbees.com/job/plugins/job/workflow-plugin/) with validated merge support.
  [![Build Status](https://jenkins.ci.cloudbees.com/buildStatus/icon?job=plugins/workflow-plugin)](https://jenkins.ci.cloudbees.com/job/plugins/job/workflow-plugin/)
* [Video tutorial on implementing a Step API](http://jenkins-ci.org/content/workflow-plugin-tutorial-writing-step-impl)

# Source organization

While the implementation is divided into a number of plugins, for ease of prototyping they are all kept in one repository using snapshot dependencies.

* `step-api` defines a generic build step interface (not specific to flows) that many plugins could in the future depend on.
* `basic-steps` add some generic step implementations.
* `api` defines the essential aspects of flows and their executions. In particular, the engine running a flow is extensible and so could in the future support visual orchestration languages.
* `support` adds general implementations of some internals needed by flows, such as storing state.
* `job` provides the actual job type and top-level UI for defining and running flows.
* `durable-task-step` uses the `durable-task` plugin to define a shell script step that can survive restarts.
* `scm-step` adds SCM-related steps.
* `cps` is the flow engine implementation based on the Groovy language, and supporting long-running flows using a _continuation passing style_ transformation of the script.
* `stm` is a simple engine implementation using a _state transition machine_, less intended for end users than as a reference for how engines can work.
* `aggregator` is a placeholder plugin allowing you to `mvn hpi:run` and see everything working together.
