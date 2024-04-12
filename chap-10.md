# Chap 10. Monitoring, Logging, and Debugging

## Gaining More Observability

> [!TIP]
> General goal of observability?
> Quickly/easily _identify/find_ the information about the **state** of a **process/system**.

### Understanding Status at a High Level

By default, the `Actions` tab shows run from all workflows:

The list of workflow runs can be filtered by:

- Workflow / Event / Status / Branch / Actor
- or with `Filter workflow runs` search box

### Creating Status Badges for Workflows

status badge
: a badge shows whether a workflow is currently failing or passing[^adding-a-workflow-status-badge].
: usually embedded in the `README.md` file in a repository

By default, badges display the status of your `default branch`.

- You can also display the status of a workflow run for a specific `branch` or `event` (using the branch and event query parameters in the URL).

To create a status badge:

- Open UI for `Create status badge`:
  - `Actions` tab (Workflow runs list) / Filter workflow runs by a workflow / `...` button (`Show workflow options`) / `Create status badge`.
  - Detail page of a workflow run / `...` button (`Workflow run options`) / `Create status badge`.
- Choose branch, event
- Copy status badge Markdown

> [!TIP]
> A status badge is a set of Markdown code that can be placed on any page that supports Markdown.

## Working with Past States of the Code Base

### Mapping Workflow Files to Runs

Each of the workflow run is linked back to

- the commit that triggered that workflow:
  - the workflow file
  - the changeset of that commit
- the pull request involved.

In the detail page of the workflow run, you can

- See the summary of the workflow.
- The list of jobs and its statuses, output data.
- Re-run all jobs or a specific job.

### Re-running Jobs in a Workflow

Within a 30-day window from the initial run, GitHub Actions allows you re-run:

- all jobs
- all failed jobs
- a specific job

> [!NOTE]
> When you re-run a job, you can `Enable debug logging` which will enable:
>
> - `step debug logging`
> - `runner diagnostic logging`
>
> For more information, see
>
> - [Enabling debug logging]
> - [Re-running workflows and jobs]

## Debugging Workflows

By default, a job log output has just enough information to know what's happening in the happy case.

If something went wrong, you can turn on `debug logging` to know exactly what's happened.

> [!WARNING]
> When you enable `debug logging`, there will be a lot of _noise_.

### Step Debug Logging

When `Step Debug Logging` is enabled, additional log events with the prefix `::debug::` will now also appear in the job's logs:

- These log events are provided by the Action's author and the runner process.

> [!NOTE]
> The job raw logs is available to:
>
> - View online (via `View raw log`)
> - Download (via `Download log archive`)
>
>   The logs are organized:
>
>   - A whole file for the job.
>   - Split to files for each step of the job (and store in a directory)

### Debugging the Runner Environment

`Runner diagnostic logging`[^runner-diagnostic-logging] provides additional log files that contain information about how a runner is executing a job.

In the log archive's `runner-diagnostic-logs` folder, for each job, there are 2 extra log files:

- `Runner` process log: how the runner interact with GitHub to prepare for the job.
- `Worker` process log: logs of the execution of the job.

### Activating Debugging

- For re-run jobs: Tick the `Enable debug logging` checkbox.

- For all jobs: Use secret/variable
  - To enable `step debug logging`: set `ACTIONS_STEP_DEBUG` to `true`.
  - To enable `runner diagnostic logging`: set `ACTIONS_RUNNER_DEBUG` `true`.

> [!NOTE]
> If both the secret & variable are set, the value of the secret takes precedence over the variable.

### Debugging Self-Hosted Runners

GitHub Actions has a good [write up][troubleshooting-self-hosted-runners] for debugging self-hosted runners.

## Augmenting and Customizing Logging

### Adding Your Own Messages in Logs

You can add your own messages to the log of a job, by using _workflow commands_ (as a part of a step's `run` clause) to:

- Set a _debug_ message
- Set a _notice_ message
- Set a _warning_ message
- Set an _error_ message

> [!IMPORTANT]
> How to view your own messages?
>
> - For debug messages: `Step Debug Logging` needs to be enabled before the workflow run for the debug messages to show up (with other debug messages from GitHub Actions ) in the job log.
> - For notice/warning/error messages: they will produce annotations in the workflow run summary.

> [!NOTE]
> Workflow command is how you tell the runner machine to do tasks, e.g.
>
> - Add debug messages to the output logs
> - Set environment variables
> - Output values used by other actions

> [!NOTE]
> To invoke a workflow command, in a step `run` clause you use the shell command `echo` :
>
> - In the format of `echo "::<workflow-command>::"`
>
>   e.g.
>
>   - Setting a debug message `::debug::{message}`
>
>     Invoke: `run: echo "::debug::This is a debug message"`
>
>   - Masking a value in a log: `::add-mask::{value}`
>
>     Invoke: `run: echo "::add-mask::THIS MESSAGE SHOULD BE MASKED"`
>
> - To write to a file.
>
>   e.g.
>
>   - Setting an environment for subsequent steps
>
>     `echo "{environment_variable_name}={value}" >> "$GITHUB_ENV"`

### Additional Log Customizations

#### Grouping lines in a log

To create a group:

- Use the `group` command and specify a `title`.
- When you finish, use the `endgroup` command

Anything you print to the log between the `group` and `endgroup` commands is nested inside an expandable entry in the log.

Syntax:

- ```yml
  ::group::{title}
  ```

- ```yml
  ::endgroup::
  ```

e.g.

- ```yml
  steps:
    - name: Group lines in log
      run: |
        echo "::group::Extended info"
        echo "Info line 1"
        echo "Info line 2"
        echo "Info line 3"
        echo "::endgroup::"
  ```

#### Masking values in logs

`add-mask` command masks the value of a string/variable

Syntax: `::add-mask::{value}`

e.g.

```yml
jobs:
  log_formatting:
    runs-on: ubuntu-latest
    env:
      USER_ID: "User 1234"
    steps:
      - run: echo "::add-mask::$USER_ID"
      - run: echo "USER_ID is $USER_ID"
```

> [!NOTE]
> The repository's secret are automatically masked by GitHub Actions.

### Add a Job Summary

job summary
: the output displayed on the summary page of a job
: a grouping of any summaries for the individual steps in the job

To add a summary for a step to a job’s summary, you write the summary to "`GITHUB_STEP_SUMMARY` environment file".

> [!NOTE]
> During the execution of a workflow, the runner generates **temporary files** that can be used to perform certain actions.
>
> The **path** to these files are exposed via environment variables, e.g.
>
> - `GITHUB_STEP_SUMMARY`
> - `GITHUB_ENV`
> - `GITHUB_OUTPUT`

When a job finishes, the summaries for all steps in a job are

- grouped together into a single job summary
- (shown on the workflow run summary page)

> [!TIP]
> The `GITHUB_STEP_SUMMARY` is unique for each step in a job.

## Conclusion

- The execution information of your workflow runs can be access in a lot of ways:
  - The status badges: Is a workflow `passing` or `failing`?
  - `Actions` Tab: The list of workflow runs:
    - The detail page of workflow run:
      The run progress is visualized with a graph for all the jobs
      - Each job log output
- If a workflow doesn't work, let it runs again. Via the web interface, you can:
  - Re-run a specific job
  - Re-run failed jobs
  - Re-run all jobs
- But that workflow still doesn't work, let's fix it:
  - But, wait, maybe re-run it once more time (and turn on debug logging) and it's fixed.
  - It failed again:
    - Check the `debugging log` that's you've just turned on.
    - Add your own logs.
- OK, everything works now, but you want to quickly know whether the workflow do as you want. You can add job summaries to add much more detail about the outcome of the job.

[Enabling debug logging]: https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging
[Re-running workflows and jobs]: https://docs.github.com/en/actions/managing-workflow-runs/re-running-workflows-and-jobs
[troubleshooting-self-hosted-runners]: https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/monitoring-and-troubleshooting-self-hosted-runners

[^adding-a-workflow-status-badge]: <https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge>
[^runner-diagnostic-logging]: <https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging#enabling-^runner-diagnostic-logging>
