# Chapter 8. Managing Workflow Execution

GitHub Actions workflow is declarative:

- You define triggers, jobs, steps (its action, commands).
- The workflow file is in YAML format.

GitHub Actions provides a number of constructs/approach to make workflow a little imperative so you can control their flow of execution:

- Manage how workflow are **started**
- Manage how workflow **progress** once started

## Advanced Triggering from Changes

> Workflow triggers are events that cause a workflow to run
>
> ([Chapter 2 - Triggering workflows](chap-02.md#triggering-workflows))

Some events have `activity types`/`filters` that give you more control over when your workflow should run

### Triggering Based on Activity Types

The `activity types` values allow you to specify what kinds of operations on the object will cause your workflow to run.

Use `on.<event_name>.types` to define the type of event activity that will trigger a workflow run.

e.g.

- Trigger this workflow on an `issues` event with any `activity types`

  ```yml
  on:
    issues:
  ```

- Trigger this workflow on an `issues`[issues-event] event with `activity type` of `opened`, or `labeled`

  ```yml
  on:
    issues:
      types:
        - opened
        - labeled
  ```

When you specify multiple `activity types`,

- Only one of those event `activity types` needs to occur to trigger your workflow.
- If multiple triggering event `activity types` for your workflow occur at the same time, multiple workflow runs will be triggered.

  e.g. For the above example, if an issue with 2 labels is opened, 3 workflow runs will start:

  - 1 for the issue `opened` event
  - 2 for the 2 issue `labeled` events.

### Using Filters to Refine Triggers

Some triggering events allow using `filters` to further define when a workflow will run in response to the event:

- `on.push.<branches|tags>`
- `on.pull_request.branches`
- `on.<push|pull_request>.paths`

A filter is specified using

- a keyword that defines the type of entity to filter, e.g. `branches`, `tags`, `paths`
- one or more strings that are specific names or patterns.
  The strings can use standard glob syntax (`*`, `**`, `?`, `!`, `+`, etc.) to match multiples.

e.g.

- Specify which branches & tags cause a workflow to run when a push event occurs

  ```yml
  on:
    push:
      branches:
        - main
        - "rel/v*"
      tags:
        - v1.*
        - beta
  ```

- Specify which paths causes a workflow to run when a push event occurs

  ```yml
  on:
    push:
      paths:
        - "**.go"
  ```

For each of the entity to filter, there is a corresponding one to filter out: `branches-ignore`, `tags-ignore`, `paths-ignore`

> [!IMPORTANT]
> For an entity to filter, you can only use both pair of filter. e.g.
>
> ```yml
> on:
>   push:
>     paths:
>       - "**.go"
>     paths-ignore: # This doesn't work
>       - "**.js"
> ```
>
> To use both include & exclude paths, use the `!` pattern, e.g.
>
> ```yml
> on:
>   push:
>     paths:
>       - "**.go"
>       - "!**.js" # The workflow only runs when there is a change to that not match this pattern
> ```

## Triggering Workflows Without a Change

## Dealing with Concurrency

## Running a Workflow with a Matrix

## Workflow Functions

### Conditionals and Status Functions

## Conclusion

A workflow can be started by:

- `Event Trigger` with a lot of fine-gained control over:

  - _When_ to trigger workflows that involve certain types of GitHub objects - with `Activity types`
  - _What_ patterns of changes trigger the workflow - with `Filtering`

- `Non-event Trigger`

A workflow/job can have multiple instance running at the same time:

- To ensure there is only 1 running instance of a workflow/job, you use `concurrency` control[^workflow-concurrency][^job-concurrency].
- To have multiple instances of your workflow running at the same time, you use a [`matrix` strategy][job-stratery].

The execution flow of a workflow (the steps) can be control with [status check function][status-check-functions].

[job-stratery]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategy
[status-check-functions]: https://docs.github.com/en/actions/learn-github-actions/expressions#status-check-functions

[^workflow-concurrency]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#concurrency>
[^job-concurrency]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idconcurrency>

[issues-event]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issues
