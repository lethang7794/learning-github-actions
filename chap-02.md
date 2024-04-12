# How Does Actions Work?

## An Overview of a workflow

At a high level, a GitHub Actions workflow works like this:

1. Some triggering _event_ happened in a Github repository.

   > [!TIP]
   > The triggering event is often associated with a Git reference (e.g. a branch) that resolve to a commit's SHA1 value.
   >
   > Or it may be an event in Github.

1. _Workflow_ files are searched (to look for the one respond to that _event type_).

   > [!TIP]
   > In addition to the type, many events can also have other qualifiers.
   >
   > e.g. Qualifier for `push`/`pull` operation

1. Corresponding workflows are identified, new _runs_ of matching workflows are triggered.

A Github Actions workflow is:

- aka workflow object, a set of code that defines a sequence of _jobs_, which are made up of _steps_ to be executed.
- in YAML format
- stored in `<repository>/.github/workflows` directory.

> [!NOTE]
> By default, jobs are run in parallel.

> [!NOTE]
> A step either
>
> - runs a shell command
> - or invokes a predefined Github action

> [!NOTE]
> All steps (in a job of a workflow) are executed on a _runner_.

> [!NOTE]
> The runner is
>
> - a server (virtual or physical)
> - a container
>
> that has been setup to understand how to interact with Github Actions.

![Relationship of GitHub Actions components](assets/github-actions-components.png)
Relationship of GitHub Actions components

> [!NOTE]
> GitHub Actions use the CI pattern:
>
> - Some change is made that is automatically detected.
> - When a change is detected, it triggers a set of automated processes to run & respond to that change.

## Triggering Workflows

In GitHub Actions, the changes that signal that work needs to be kicked off are triggering _events_ (aka _triggers_).

When an event happens, and there are workflows that has start conditions matches that event, runs of matching workflows are kicked off.

Workflow triggers are events that cause a workflow to run. These events can be:

- Events that occur in your workflow's repository
- Events that occur outside of GitHub and trigger a `repository_dispatch` event on GitHub.
- Scheduled times
- Manual

To define which events can cause the workflow to run, you use `on`.

The workflow can:

- Respond to single event

  ```yml
  on: push
  ```

- Respond to multiple event

  ```yml
  on: [push, pull_request]
  ```

- Respond to event's types, qualifiers (e.g. branches, tags, file paths)

  ```yml
  on:
    push:
      branches:
        - main
  on:
    push:
      branches:
        - main
        - 'rel/v*'
      tags:
        - v1.*
        - beta
      paths:
        - '**.ts'
  ```

- Execute on a schedule (using standard cron syntax)

  ```yml
  on:
    scheduled:
      - cron: "30 5,15 * * *"
  ```

- Respond to specific manual events

  ```yml
  on: workflow-dispatch
  ```

- Respond to external event that trigger `repository-dispatch`

  ```yml
  on: repository-dispatch
  ```

- Be called by other workflows

  ```yml
  on: workflow_call
  ```

- Respond to activities on Github items, e.g. a comment is added to a Github issue

  ```yml
  on: issue_comment
  ```

- Respond to _webhook events_

> [!CAUTION]
> A subset of less-common events will only trigger a workflow run if the workflow file (the YAML file in `.github/workflows`) is on the default branch (usually `main`)

For more about trigging workflows, see:

- [Triggering a workflow][triggering-a-workflow]
- [Events that trigger workflows][events-that-trigger-workflows]

## Components

### Steps

_Steps_:

- are the basic unit of execution of GitHub Actions.
- consist of invocations of
  - a predefined action (using `uses` clause)
  - or a shell command (using `run` clause)

e.g.

- A step that invokes a predefined action (by using `uses` clause)

  ```yml
  steps:
    - uses: actions/checkout@v4
  ```

- A step can have

  - a `name` associated with it, which will display on Github interface
  - `with` clause to passed parameters to action

    ```yml
    steps:
      - uses: actions/setup-go@v5
        name: setup Go version
        with:
          go-version: "1.14.0"
    ```

- A step that runs a shell command

  ```yml
  steps:
    - run: go run hello_world.go
  ```

> [!TIP]
> Each `run` keyword represents a new process and shell in the runner environment[^jobs-job_id-steps-run].

> [!NOTE]
> You can use an action defined in
>
> - the same repository as the workflow,
> - a public repository,
> - in a published Docker container image.
>
> e.g.
>
> - A public action (in a public GitHub repository) is specified with `{owner}/{repo}@{ref}`, e.g. `actions/checkout@v4`
>
> For more information, see [`jobs.<job_id>.steps[*].uses`][uses]

### Runners

Runners

- are where the code for a workflow is executed, which can be

  - the physical/virtual computers
  - containers

- can be hosted

  - by Github
  - by you (self-hosted)

- are configured to interact with Github Action framework to:

  - access workflows, actions
  - execute steps
  - report outcomes

Runners are defined for each job via `jobs.<job_id>.runs-on`.

e.g.

```yml
jobs:
  my-job:
    runs-on: ubuntu-latest
  steps:
    - run: lsb_release --release # Should show "Release: 23.04"
```

```yml
jobs:
  my-job:
    runs-on: ubuntu-22.04
  steps:
    - run: lsb_release --release # Should show "Release: 22.04"
```

### Jobs

A job is a set of `steps` (in a workflow) that are executed

- in order
- on the same runner.

A job is usually targeted towards accomplished one particular goal (of the overall workflow).

e.g.

- A step that builds your application.
- Followed by a step that tests the application that was built.

> [!TIP]
> As you've already known, each step is either
>
> - a shell script that will be executed, or
> - an action that will be run.

Steps (of a job) are

- executed in _order_
- _dependent_ on each other (and can share data from one step to another)

The outcome of a job is surfaced in the Github Actions interface. Success or failure is display at job-level, not the individual steps.

- If you need more granular reports, put fewer steps in a job.
- If you need less granular reports, put more steps in a job.

e.g.

- A simple job that has 3 steps: checkout, setup, run the code

  ```yml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: setup Go version
          uses: actions/setup-go@v5
          with:
            go-version: "1.22.0"
        - run: go run hello_world.go
  ```

### Workflow

A `workflow` is like a pipeline, it defines:

- first, the type of **events** & the **conditions** that it will respond to.
- then, the **response**:
  - the series of jobs that will be executed
    - for each job, the series of steps to be executed

e.g.

- A workflow that processing Go programming

```yml
# .github/workflows/simple-go-build.yml
name: Simple Go Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Go version
        uses: actions/setup-go@v5
        with:
          go-version: "1.20"
      - run: go run hello_world.go
```

## Workflow Execution

When you push the above workflow file, with a corresponding `hello_world.go` file in the repository, a `run` of the workflow will be executed.

The execution of the workflow can be seen in `Actions` tab of the GitHub repository.

## Conclusion

- `Actions` refers to the entire system for executing automated workflows in response to events.
- `actions` refer to the units of code that perform a repeated tasks.
- `workflow`s are like software pipeline:
  - they can be initiated when a `triggering event` occurs
  - a `workflow` aggregate one or more `jobs` to accomplish their overall outcome.
    - each `job` aggregate one or more `steps` to do smaller units of work.
      - each `step` invoke predefined actions or run a shell command
      - the execution of the `steps` in a single `job` results in success/failure outcome for the `job`, which feeds back into success/failure for the overall workflow.
    - each `job` declare what kind of `runner` (OS & version) it will run in.

[triggering-a-workflow]: https://docs.github.com/en/actions/using-workflows/triggering-a-workflow
[events-that-trigger-workflows]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows
[uses]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsuses

[^jobs-job_id-steps-run]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun>
