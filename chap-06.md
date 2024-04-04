# Chapter 6. Managing Your Workflow Environments

## Naming Your Workflow & Workflow Runs

`name`
: The name of the _workflow_
: Displayed in the list of workflows (on repository's 'Actions' tab)
: Fallback to workflow file **path** (relative to the root of the repository)

`run-name`
: The name for _workflow runs_ generated from the workflow.
: Displayed in the list of workflow runs (on your repository's 'Actions' tab)
: Fallback to **event-specific information** (for the workflow run). e.g. The pull request's name
: Can include expressions & reference the `github` & `inputs` contexts.

e.g.

```yml
run-name: Run by @${{ github.actor }}
```

## Contexts

Contexts
: Contexts are a way to access information about workflow runs, jobs, steps, runner environments, variables...

Each context is an object that contains properties, which can be strings or other objects.

To access contexts & its properties, you use

- the expression syntax `${{` & `}}`
- property dereference syntax, e.g. `github.actor`, or
- index syntax, e.g. `github['sha']`

e.g. `${{ github.actor }}`

There are a lot of contexts:

| Context Name | Description                                                                    | Example                                                |
| ------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------ |
| `github`     | Information about the **workflow run** & the **event** that triggered the run  | `github.ref`, `github.event_name`, `github.repository` |
| `job`        | Information about the currently **running job**                                | `job.status`                                           |
| `steps`      | Information about the **steps** that have been run in the current job          | `steps.<step_id>.outcome`                              |
| `runner`     | Information about the **runner** that is running the current job               | `runner.os`, `runner.arch`                             |
| `env`        | (Environment) Variables set at **workflow/job/step** level                     | `env.<env_name>`                                       |
| `vars`       | (Configuration) Variables set at **organization/repository/environment** level | `vars.<var_name>`                                      |
| `secrets`    | Secrets that are available to a workflow run.                                  | `secrets.GITHUB_TOKEN`                                 |

The contexts can be used as part of conditional expression

e.g. `${{ github.ref == 'ref/heads/main' }}`

## Environment Variables

Environment variables allow you to manage reusable configuration data.

Within a workflow, you can define environment variables to be used at the level of

- a workflow,
- an individual job, or
- even an individual step.

To set these up, you use an `env` section - a mapping of variables to values - at the level use want.

If the same variable exists at multiple levels:

- variables defined at the step level
  - override variables defined at the job level, which in turn
    - override variables defined at the workflow level.

e.g.

```yml
# workflow level
env:
  PIPE: ci

# job level
jobs:
  build:
    env:
      STAGE: dev

  steps:
    # step level
    - name: create item with token
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Environment variables you define within a workflow are called **_custom_ environment variables**.

### Default Environment Variables

GitHub provides a default set of environment variables available for workflows to use. These environment variables

- are called **_default_ environment variables**[^default-environment-variables]
- are named starting with `GITHUB_` or `RUNNER_`

e.g.

- `GITHUB_WORKFLOW`
- `RUNNER_OS`

These variables can be used together in workflows to get information at runtime.

e.g. A job to report the URL of the running workflow

```yml
jobs:
  report-url:
    runs-on: ubuntu-latest
    steps:
      - run: echo "$GITHUB_SERVER_URL/$GITHUB_REPOSITORY/actions/runs/$GITHUB_RUN_ID"
        # Output: https://github.com/gwstudent2/greetings-ci/actions/runs/4744932978
```

Most of the default environment variables have corresponding properties that can be used from the `github` or `runner` contexts.

e.g.

```yaml
jobs:
  report-url:
    runs-on: ubuntu-latest
    steps:
      - run: echo ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

```yml
jobs:
  report-os:
    runs-on: ubuntu-latest
    steps:
      - name: check-os
        if: runner.os != 'Windows'
        run: echo "The runner's operating system is $RUNNER_OS."
        # Output: The runner's operating system is Linux.The runner's operating system is Linux.
```

## Secrets & Configuration Variables

Environment variables has 2 types:

- `Secrets` are encrypted and are used for sensitive data.
- `Configuration Variables` (aka `variables`) are shown as plain text and are used for non-sensitive data.

To create a secret/variable:

- Go to repository/organization _Settings_ tab / _Security_ section / _Secrets & variables_ / _Actions_
- Click the appropriate tab for _Secrets_/_Variables_
- Click _New repository/organization secret/variable_ button
- Fill in the Name & Value.

> [!IMPORTANT]
> Environment variables vs Configuration variables
>
> Environment variables
> : Intended for use only in the scope of a workflow
>
> Configuration variables
> : Intended for use in many workflows (at repository/organization/enterprise level)
>
> Both type of variables can be used at the same time, e.g.
>
> ```yml
> env:
>   INFO_LEVEL: ${{ vars.INFO_LEVEL }}
> ```

## Managing Permissions for Your Workflow

You can use the `GITHUB_TOKEN` secret to authenticate in the workflow job & make calls to the GitHub API.

- At the start of each workflow job, GitHub automatically creates a unique `GITHUB_TOKEN` secret to use in your workflow.[^automatic-token-authentication]
- Before each job begins, GitHub fetches that `GITHUB_TOKEN` secret for the job.

By default, `GITHUB_TOKEN` has a `read-only` permission[^default-github_token-permissions-to-read-only] apply for the entire workflow.

You can modify the [default permissions][defaul-permissions] granted to the `GITHUB_TOKEN` by using `permissions` keyword.

- For each of the available scopes[^available-scopes],

  - you can assign one of the permissions: `read`, `write`, or `none`
  - those scopes that are not specified are set to `none`.

    e.g.

    - Default permissions (restricted)

      ```yml
      permissions:
        contents: read
        metadata: read
        packages: read
        # All other scopes are not specified, Actions sets these scopes to `none`
      ```

    - Default permissions (permissive)

      ```yml
      permissions:
        id-token: none
        metadata: read
        contents: write
        # All other scopes are set to `write`
      ```

  - You can define permissions for all available scopes with `write-all`, `read-all`, `{}`

    - Grant write permission to all scopes

      ```yml
      permissions: write-all
      ```

    - Grant read permission to all scopes

      ```yml
      permissions: read-all
      ```

    - Disable permissions for all scopes

      ```yml
      permissions: {}
      ```

You can use `permissions`

- as a top-level key, to apply to all jobs in the workflow

  e.g.

  ```yml
  permissions: write-all
  ```

- within specific jobs.

  e.g.

  ```yml
  jobs:
    stale:
      runs-on: ubuntu-latest

      permissions:
        issues: write
        pull-requests: write

      steps:
        - uses: actions/stale@v5
  ```

> [!TIP]
> When you enable GitHub Actions, GitHub installs a GitHub App on your repository.
>
> The `GITHUB_TOKEN` secret is a `installation access token` of that GitHub App.

> [!NOTE]
> The `GITHUB_TOKEN` secret can be access via
>
> - `secrets` context[^secrets-context]
>
> e.g. `${{ secrets.GITHUB_TOKEN }}`
>
> - `github` context[^github-context] (only available within the execution steps of a job)
>
> e.g. `${{ github.token }}`

> [!NOTE]
> Use case of the `GITHUB_TOKEN` secret:
>
> - Passing to an action, e.g.
>
> ```yml
> steps:
>   - uses: actions/labeler@v5
>     with:
>       repo-token: ${{ secrets.GITHUB_TOKEN }}
> ```
>
> - Make GitHub REST API calls
>
> ```yml
> steps:
>   - run: |
>       curl --request GET \
>       --url "https://api.github.com/octocat" \
>       --header "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}"
> ```

> [!NOTE]
> If you need more permissions than the `GITHUB_TOKEN` can provide, you can create your personal access token, and store it as a secret in the repository.

## Deployment Environments

## Conclusion

[^default-environment-variables]: <https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables>
[^automatic-token-authentication]: <https://docs.github.com/en/actions/security-guides/automatic-token-authentication>
[^default-github_token-permissions-to-read-only]: <https://github.blog/changelog/2023-02-02-github-actions-updating-the-default-github_token-permissions-to-read-only/>
[^available-scopes]: <https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs>
[^secrets-context]: <https://docs.github.com/en/actions/learn-github-actions/contexts#secrets-context>
[^github-context]: <https://docs.github.com/en/actions/learn-github-actions/contexts#github-context>

[defaul-permissions]: https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token
