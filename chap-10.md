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

### Additional Log Customizations

### Creating a Customized Job Summary

## Conclusion

[Enabling debug logging]: https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging
[Re-running workflows and jobs]: https://docs.github.com/en/actions/managing-workflow-runs/re-running-workflows-and-jobs
[troubleshooting-self-hosted-runners]: https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/monitoring-and-troubleshooting-self-hosted-runners

[^adding-a-workflow-status-badge]: <https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge>
[^runner-diagnostic-logging]: <https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging#enabling-^runner-diagnostic-logging>
