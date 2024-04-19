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

Workflows can be triggered without a change in your repository:

- As in [Defining & Referencing Workflow Inputs](chap-07.md#defining--referencing-workflow-inputs)

  - `on.workflow_dispatch`[^on-workflow_dispatch][^workflow_dispatch]: This workflow can be invoked via:

    - Actions tab,
    - GitHub CLI,
    - a REST API call.

    > [!NOTE]
    > The Run workflow button on the actions is show only when:
    >
    > - The workflow has the `workflow_dispatch` trigger.
    > - That workflow file is in the `default branch`.

  - `on.workflow_call`[^on-workflow_call][^workflow_cal]: This workflow can be used as a _reusable workflow_ - one that can be called by another workflow.

    > [!TIP]
    > Reusable workflows can be used in place of the main code of a job via the `job.<job-id>.uses` statement.

- or

  - `on.repository_dispatch`[^repository_dispatch]: Multiple workflows of this repository can be invoked by some activity that occurs outside of GitHub.

    > [!NOTE]
    > The trigger `repository_dispatch` is generally in response to some custom or external event

  - `on.workflow_run`[^on-workflow_run][^workflow_run]: This workflow can be invoked based on a separate workflow executing.

    e.g.

    - Workflow `Release` runs when workflow `Run Tests` `completed`

      ```yml
      name: Release
      on:
        workflow_run:
          workflows: [Run Tests]
          types:
            - completed
      ```

## Dealing with Concurrency

The default behavior of GitHub Actions is to allow _multiple_ jobs or workflow runs to run concurrently[^concurrency-default-behavior].

The `concurrency` keyword allows you to ensure that only a _single_ job or workflow using the same _concurrency group_ will run at a time.

e.g.

- Only 1 `release` workflow should be running in the `release-build` concurrency group

  ```yml
  concurrency: release-build
  ```

- A different syntax for concurrency group

  ```yml
  concurrency:
    group: release-build
  ```

  > [!TIP]
  > If you want a more precise concurrency group, you can leverage properties of the `github` context, e.g.
  >
  > ```
  > concurrency: {{ $github.ref }}
  > ```

If there is already a running instance (the first instance), the new instance gets trigger will:

- be "queued":

  - The second instance will be in `pending`.
  - The third instance will be in `pending`
    - The second instance will be cancelled.
    - The third instance will replace the second instance.
  - etc

  > [!IMPORTANT]
  > There can be at most
  >
  > - one running
  > - one pending job
  >
  > in a concurrency group at any time.

- replace the "in-progress instance" (by specifying `cancel-in-progress: true`), e.g.

  ```yml
  concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true
  ```

The `concurrency` keyword also works at the job level

```yml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: example-group
      cancel-in-progress: true
```

## Running a Workflow with a Matrix

`matrix` (aka `matrix strategy`)
: Github Actions way to define variants for each job
: ~ "for loop"

A matrix strategy lets you

- use variables in a single job definition
- to automatically create multiple job runs (that are based on the combinations of the variables)

### Example: A single-dimension matrix (matrix with one variable)

```yml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

### Example: A multi-dimension matrix (matrix with multiple variables)

```yml
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-20.04]
        version: [10, 12, 14]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

> [!TIP]
> With matrix strategy, you can create:
>
> - variants of jobs
> - or variants of workflows (when using with _reusable workflow_)

## Workflow Functions

GitHub Actions' workflow is declarative (in YAML format).

To give you a little more control, workflow syntax supports some [functions][functions] that you can use in [expressions][expressions]

|           | Function                  | Function parameters                            | Purpose                                                                                 | Notes                                                                                                                |
| --------- | ------------------------- | ---------------------------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Inspect   | `contains`[^contains]     | `( search, item )`                             | Returns `true` if `search` contains `item`                                              |                                                                                                                      |
|           | `startsWith`[^startsWith] | `( searchString, searchValue )`                | Returns `true` when `searchString` starts with `searchValue`                            |                                                                                                                      |
|           | `endsWith`[^endsWith]     | `( searchString, searchValue )`                | Returns `true` if `searchString` ends with `searchValue`                                |                                                                                                                      |
| Format    | `format`[^format]         | `( string, replaceValue0, ..., replaceValueN)` | Replaces values (`{0}`, ..., `{N}`) in the `string`, with the variable `replaceValueN`. |                                                                                                                      |
| Transform | `join`[^join]             | `( array, optionalSeparator )`                 | Concatenates values in the `array` together into a string.                              | The default separator is `,`                                                                                         |
|           | `toJSON`[^toJSON]         | `( value )`                                    | Returns a pretty-print JSON representation of `value`                                   | Useful to debug the information provided in contexts.                                                                |
|           | `fromJSON`[^fromJSON]     | `( value )`                                    | Returns a JSON object or JSON data type for `value`                                     | Useful to convert env variables from a string to another data type (such as boolean or integer) if needed            |
| Hash      | `hashFiles`[^hashFiles]   | `( path )`                                     | Returns a single hash for the set of files that matches the `path` pattern              | The path is relative to the `GITHUB_WORKSPACE` directory and can only include files inside of the `GITHUB_WORKSPACE` |

### Conditionals & Status Functions

#### Conditionals

You can use an `if` clause[^using-conditions-to-control-job-execution] at the start of a `job`[^jobs-job_id-if] or a `step`[^jobs-job_id-steps-if] to check a condition (with an [expression][expressions]) and determine if execution should occur or not.

When an `if` conditional is true, the `job`/`step` will run.[^about-expressions]

e.g.

- Use `if` conditional with an `expression`

  ```yml
  jobs:
    job-1:
      if: ${{  startsWith(github.ref, 'refs/tags/') }}
  ```

  `job-1` will run if the `github.ref` starts with `refs/tags/`.

- Use `if` conditional with an `expression` and `!` (NOT operator[^operators])

  ```yml
  jobs:
    job-2:
      if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
  ```

  `job-1` will run if the `github.ref` does _NOT_ start with `refs/tags/`

#### Status Check Functions

In additional to functions such as `startsWith`, `hashFiles` in [Workflow Functions](#workflow-functions), GitHub Actions also has a set of [status check functions][status-check-functions]

| Function                  | Purpose                                                                     | Notes                                                 |
| ------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------- |
| `success()`[^success]     | Returns `true` when **_all_ previous steps** have succeeded.                |                                                       |
| `always()`[^always]       | Causes the step to always execute, and returns `true`, even when cancelled. | `always` can cause the workflow to hang until timeout |
| `cancelled()`[^cancelled] | Returns `true` if the **workflow** was cancelled.                           |                                                       |
| `failure()`[^failure]     | Returns `true` when **_any_ previous step** of a job fails.                 |                                                       |

When using both `if` conditional and `status check functions`, you can control the execution of a job/step based on the status of previous steps, workflow.

> [!TIP]
> When you are using `expressions` in an `if` clause, you can omit `${{` and `}}`
>
> e.g.
>
> - An expression that is a function
>
>   ```yml
>     if: ${{ startsWith(github.ref, 'refs/tags/') }} # Expression with ${{ and }}
>     if: startsWith(github.ref, 'refs/tags/')        # Expression without ${{ and }}
>   ```
>
> - An expression that is a status check function
>
>   ```yml
>     if: ${{ always() }}
>     if: always()
>   ```

## Conclusion

A workflow can be started by:

- `Event Trigger` (some changes in your GitHub repository) with a lot of fine-gained control over:

  - _When_ to trigger workflows that involve certain types of GitHub objects - with `Activity types`
  - _What_ patterns of changes trigger the workflow - with `Filtering`

- `Non-event Trigger` (not a change in your GitHub repository)

A workflow/job can have multiple instance running at the same time:

- To ensure there is only 1 running instance of a workflow/job, you use `concurrency` control[^workflow-concurrency][^job-concurrency].
- To have multiple instances of your job running at the same time, you use a [`matrix` strategy][job-stratery].

The execution flow of a workflow (jobs/steps) can be control with [status check function][status-check-functions].

[job-stratery]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategy
[status-check-functions]: https://docs.github.com/en/actions/learn-github-actions/expressions#status-check-functions
[issues-event]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issues
[expressions]: https://docs.github.com/en/actions/learn-github-actions/expressions
[functions]: https://docs.github.com/en/actions/learn-github-actions/expressions#functions

[^workflow-concurrency]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#concurrency>
[^job-concurrency]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idconcurrency>
[^on-workflow_call]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call>
[^on-workflow_dispatch]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatch>
[^on-workflow_run]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_runbranchesbranches-ignore>
[^repository_dispatch]: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch>
[^workflow_cal]: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_call>
[^workflow_dispatch]: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch>
[^workflow_run]: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_run>
[^concurrency-default-behavior]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-concurrency-and-the-default-behavior>
[^contains]: <https://docs.github.com/en/actions/learn-github-actions/expressions#contains>
[^startsWith]: <https://docs.github.com/en/actions/learn-github-actions/expressions#startsWith>
[^endsWith]: <https://docs.github.com/en/actions/learn-github-actions/expressions#endsWith>
[^format]: <https://docs.github.com/en/actions/learn-github-actions/expressions#format>
[^join]: <https://docs.github.com/en/actions/learn-github-actions/expressions#join>
[^toJSON]: <https://docs.github.com/en/actions/learn-github-actions/expressions#toJSON>
[^fromJSON]: <https://docs.github.com/en/actions/learn-github-actions/expressions#fromJSON>
[^hashFiles]: <https://docs.github.com/en/actions/learn-github-actions/expressions#hashFiles>
[^jobs-job_id-if]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif>
[^jobs-job_id-steps-if]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif>
[^using-conditions-to-control-job-execution]: <https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution>
[^about-expressions]: <https://docs.github.com/en/actions/learn-github-actions/expressions#about-expressions>
[^operators]: <https://docs.github.com/en/actions/learn-github-actions/expressions#operators>
[^success]: <https://docs.github.com/en/actions/learn-github-actions/expressions#success>
[^always]: <https://docs.github.com/en/actions/learn-github-actions/expressions#always>
[^cancelled]: <https://docs.github.com/en/actions/learn-github-actions/expressions#cancelled>
[^failure]: <https://docs.github.com/en/actions/learn-github-actions/expressions#failure>
