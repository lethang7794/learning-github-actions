# Chap 13. Advanced Workflow Techniques

## Interacting Directly with GitHub Resources from Your Workflow

### Using the `GitHub CLI` (Write shell script)

[GitHub CLI](https://cli.github.com/) (`gh`) brings your entire GitHub "workflow" to the CLI

You can use GitHub CLI to:

- Manage repositories, pull requests, labels, issues, releases
- Manage your GitHub account, organization
- Interact with GitHub Actions
- Make GitHub API requests
- Connect to GitHub Codespaces
- ...

If you execute your workflows on a GitHub-hosted runner, the GitHub CLI is already installed.

- You can simple call `gh` in a `jobs.<job_id>.steps[*].run` command

- The only prerequisite is you need to give `gh` permissions via the `GITHUB_TOKEN` environment variable, e.g.

  ```yaml
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```

<details>
<summary><b>Example: A workflow create issue via GitHub CLI</b></summary>

```yaml
# .github/workflows/create-issue-via-gh.yml
name: create issue via gh

on:
  workflow_call:
    inputs:
      title:
        description: "Issue title"
        required: true
        type: string
      body:
        description: "Issue body"
        required: true
        type: string
    outputs:
      issue-number:
        description: "The issue number"
        value: ${{ jobs.create-issue.outputs.issue-num }}

jobs:
  create-issue:
    runs-on: ubuntu-latest
    # Map job outputs to step outputs
    outputs:
      issue-num: ${{ steps.new-issue.outputs.issue-num }}

    permissions:
      issues: write

    steps:
      - name: Create issue using CLI
        id: new-issue
        run: |
          response=`gh issue create \
          --title "${{ inputs.title }}" \
          --body "${{ inputs.body }}" \
          --repo ${{ github.event.repository.url }}`
          echo "issue-num=$response | rev | cut -d'/' -f 1" >> $GITHUB_OUTPUT
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

</details>

### Using `github-script` action (Write script in JavaScript)

With `github-script` action, you can write a script (in JavaScript) in your workflow that uses the GitHub API and the workflow run's `context`.

| Scope                  | Entity                    | Description                                                                                                           | Notes                                                                                         |
| ---------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Workflow file          | `github` context          | Information about **workflow run** & the **event** that triggered the run.                                            |                                                                                               |
| `github-script` action | `github` object           | A pre-authenticated **`octokit/rest.js` client** with pagination plugins                                              |                                                                                               |
|                        | `context` object          | An object containing the context of the **workflow run** (and the webhook payload object that triggered the workflow) | `github-script` action's `context` is a reference to the `@actions/github`'s `context` object |
| GitHub ToolKit         | `@actions/github` package | Provides an Octokit client hydrated with the context that the current action is being run in                          | `@actions/github` package also exposes `context` object                                       |

To use the `github-script` action, you provide the `script` input with the body of the _script_ you want to write.

<details>
<summary>
<b>Example: A github script that add labels to issue</b>
</summary>

```yaml
steps:
  - uses: actions/github-script@v6
    with:
    script: |
      github.rest.issues.addLabels({        # Interact with GitHub APIs via the Octokit client `github`
        issue_number: context.issue.number, # Access the workflow run information via `context`
        owner: context.repo.owner,          # ...
        repo: context.repo.repo,            # ...
        labels: ['Triage']
      })
```

</details>

### Invoking `GitHub REST APIs` (Write shell script)

You can directly invoke GitHub REST APIs and do a lot of things by

- using a CLI HTTP client
- with the `GITHUB_TOKEN` from workflow run's context `secrets`/`github`

<details>
<summary><b>Example: Create a issue from workflow inputs (As in the example for <a href="./chap-12.md#outputs">Outputs | Chap 12</a>)</b></summary>

```yaml
# .github/workflows/create-repo-issue-v2
name: create-repo-issue-v2

on:
# ...

jobs:
  create-issue:
    runs-on: ubuntu-latest
    # ...
    steps:
      - name: Create issue using REST API
        run: |
          response=$(curl --request POST \                                       # Use curl to invoke a HTTP request
          --url https://api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.PAT }}' \                  # ... provide the token
          --header 'content-type: application/json' \
          --data '{
            "title": "${{ inputs.title }}",                                      # ... and the inputs
            "body": "${{ inputs.body }}"
            }' \
          --fail | jq '.number')
          echo "issue-num=$response" >> $GITHUB_OUTPUT
```

</details>
<details>
<summary><b>Example: Create an issue on failure</b></summary>

```yaml
jobs:
  create-issue-on-failure:
    runs-on: ubuntu-latest
    needs: test-run
    if: always() && failure()
    steps:
      - name: invoke workflow to create issue
        run: >
          curl -X POST
            -H "authorization: Bearer ${{ secrets.PIPELINE_USE }}"
            -H "Accept: application/vnd.github.v3+json"
            "https://api.github.com/repos/${{ github.repository }}/actions/workflows/create-failure-issue.yml/dispatches"
            -d '{
                  "ref": "main",
                  "inputs": {
                    "title": "Automated workflow failure issue for commit ${{ github.sha }}",
                    "body": "This issue was automatically created by the GitHub Action workflow ** ${{ github.workflow }} **"
                  }
                }'
```

</details>

## Using a Matrix Strategy to Automatically Create Jobs

As in [chap 8](chap-08.md), a workflow can be triggered:

- By a **change** (in the repository)
- Without a ~~change~~ (in the repository)
- By a combination of **values** you want

e.g. Run tests:

- across multiple environments: `dev`, `test`, `release`
  - across multiple OSes: `Linux`, `Mac`, `Windows`

### Using a matrix strategy

Use `jobs.<job_id>.strategy.matrix` to define a matrix of different job configurations:

- Within your matrix, define one or more variables followed by an array of values.

  <details>
  <summary>
  e.g.
  </summary>

  ```yml
  jobs:
    example_matrix:
      strategy:
        matrix:
          version: [10, 12, 14] # version variable has 3 values
  ```

  </details>

- A job will run for each possible combination of the variables.

  <details>
  <summary>
    e.g.
  </summary>

  - The above example creates 3 jobs in the following order:

    - `{ version: 10 }`
    - `{ version: 12 }`
    - `{ version: 14 }`

  </details>

- The variables that you define become properties in the [`matrix` context], and you can reference the property in other areas of your workflow file.

  <details>
  <summary>e.g.</summary>

  ```yaml
  jobs:
    example_matrix:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          version: [10, 12, 14]
      steps:
        - uses: actions/setup-node@v4
          with:
            node-version: ${{ matrix.version }} # Reference the value for version variable via matrix context's version property
  ```

  </details>

#### Single-Dimension Matrix

The above example is the example for an single-dimension matrix at [Chap 8](chap-08.md#example-a-single-dimension-matrix-matrix-with-one-variable), it's also the example on the [docs page](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-single-dimension-matrix)

#### Multi-Dimension Matrix

You can define multiple variables in the matrix and make it a multi-dimension matrix.

e.g. This is also the [example for multi-dimension matrix](chap-08.md#example-a-multi-dimension-matrix-matrix-with-multiple-variables) in chap 8

<details>
<summary></summary>

```yaml
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

</details>

#### Using context to create matrix

A matrix variable's values can be:

- hard-code (as the above examples)

- or not.

  You can pass values from a `github` context (which including the event payload) to a matrix variable.

e.g. For a workflow triggered by `repository_dispatch` event

- The payload of [`repository_dispatch` event]

  ```json
  {
    "event_type": "node-version-updated",
    "client_payload": {
      "versions": [12, 14, 16]
    }
  }
  ```

- The workflow

  ```yaml
  on:
    repository_dispatch:
      types:
        - node-version-updated

  jobs:
    example_matrix:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          version: ${{ github.event.client_payload.versions }}
      steps:
        - uses: actions/setup-node@v4
          with:
            node-version: ${{ matrix.version }}
  ```

- Trigger the workflow in a step of another workflow's job

  ```shell
  curl -X POST \
    -H "Authorization: Bearer ${{ secrets.PAT }}" \
    -H "Accept: application/vnd.github.v3+json" \
    "https://api.github.com/repos/${{ github.repository }}/dispatches" \
    -d '{"event_type":"node-version-updated",
    "client_payload":{"versions":["12","14","16"]}}'
  ```

### Including Extra Values

Use [`jobs.<job_id>.strategy.matrix.include`] to

- expand existing matrix configurations
- or to add new configurations.

The value of `include` is a list of objects.

For each object in the include list:

- If none of the `key:value` pairs overwrite any of the _original_ matrix values, the `key:value` pairs in the object will be added to each of the matrix combinations
- If the object cannot be added to any of the matrix combinations, a new matrix combination will be created instead.
- Note that the original matrix values will not be overwritten, but added matrix values can be overwritten.

Example 1: Expanding configuration

- The matrix

  ```yaml
  jobs:
    example_matrix:
      strategy:
        matrix:
          os: [windows-latest, ubuntu-latest]
          node: [14, 16]
          include:
            - os: windows-latest
              node: 16
              npm: 6
  ```

- The configurations:

  <details>
  <summary>
  <b>Before</b>
  </summary>

  - `{ "os": "windows-latest", "node": "14" }`
  - `{ "os": "windows-latest", "node": "16" }` (This will be expanded)
  - `{ "os": "ubuntu-latest", "node": "14" }`
  - `{ "os": "ubuntu-latest", "node": "16" }`

  </details>

  <details>
  <summary>
  <b>After</b>
  </summary>

  - `{ "os": "windows-latest", "node": "14" }`
  - `{ "os": "windows-latest", "node": "16", "npm": "6" }`
  - `{ "os": "ubuntu-latest", "node": "14" }`
  - `{ "os": "ubuntu-latest", "node": "16" }`

  </details>

Example 2: Add configuration

- The matrix

  ```yaml
  jobs:
    example_matrix:
      strategy:
        matrix:
          os: [windows-latest, ubuntu-latest]
          node: [14, 16]
          include:
            - os: ubuntu-latest
              version: 18
  ```

- The configurations:

  <details>
  <summary>
  <b>Before</b>
  </summary>

  - `{ "os": "windows-latest", "node": "14" }`
  - `{ "os": "windows-latest", "node": "16" }`
  - `{ "os": "ubuntu-latest", "node": "14" }`
  - `{ "os": "ubuntu-latest", "node": "16" }`

  </details>

  <details>
  <summary>
  <b>After</b>
  </summary>

  - `{ "os": "windows-latest", "node": "14" }`
  - `{ "os": "windows-latest", "node": "16" }`
  - `{ "os": "ubuntu-latest", "node": "14" }`
  - `{ "os": "ubuntu-latest", "node": "16" }`
  - `{ "os": "ubuntu-latest", "node": "18" }` (The added configuration)

  </details>

### Excluding Values

To remove specific configurations defined in the matrix, use [`jobs.<job_id>.strategy.matrix.exclude`].

An excluded configuration only has to be a partial match for it to be excluded.

Example: Exclude configurations

- The matrix

  ```yaml
  jobs:
    my-job:
      strategy:
        matrix:
          os: [ubuntu-latest, windows-latest]
          version: [12, 14]
          environment: [staging, production]
          exclude:
            - os: ubuntu-latest
              version: 12
              environment: production
            - os: windows-latest
              version: 14
      runs-on: ${{ matrix.os }}
  ```

- The configurations:

  <details>
  <summary>
  <b>Before</b>
  </summary>

  - `{ "os": "ubuntu-latest", "node": "12", "environment": "staging" }`
  - `{ "os": "ubuntu-latest", "node": "12", "environment": "production" }`
  - `{ "os": "ubuntu-latest", "node": "14", "environment": "staging" }`
  - `{ "os": "ubuntu-latest", "node": "14", "environment": "production" }`
  - `{ "os": "windows-latest", "node": "12", "environment": "staging" }`
  - `{ "os": "windows-latest", "node": "12", "environment": "production" }`
  - `{ "os": "windows-latest", "node": "14", "environment": "staging" }`
  - `{ "os": "windows-latest", "node": "14", "environment": "production" }`

  </details>

  <details>
  <summary>
  <b>After</b>
  </summary>

  - `{ "os": "ubuntu-latest", "node": "12", "environment": "staging" }`
  - ~~`{ "os": "ubuntu-latest", "node": "12", "environment": "production" }`~~ (This configuration will be excluded)
  - `{ "os": "ubuntu-latest", "node": "14", "environment": "staging" }`
  - `{ "os": "ubuntu-latest", "node": "14", "environment": "production" }`
  - `{ "os": "windows-latest", "node": "12", "environment": "staging" }`
  - `{ "os": "windows-latest", "node": "12", "environment": "production" }`
  - ~~`{ "os": "windows-latest", "node": "14", "environment": "staging" }`~~ (This configuration will be excluded)
  - ~~`{ "os": "windows-latest", "node": "14", "environment": "production" }`~~ (This configuration will be excluded)

  </details>

### Handling Failure Cases

### Defining Max Concurrent Jobs

## Using Containers in Your Workflow

### Using a Container as the Environment for a Job

### Using a Container with a Step

### Running Containers as Services in a Job

## Conclusion

- If you need to _directly interact_ with **GitHub** resources from your workflow (without creating an action, a workflow), you can:

  - Use [GitHub **CLI**][github-cli]
  - Use [`github-script` action][github-script]
  - Invoke [GitHub REST **APIs**][github-rest-api], e.g. with `curl`

- To automatically _generate **jobs**_, you can use **matrix strategy**.

- To have more _control_ about the **environment**/tooling of your workflow, you can use **container**:

  - as the environment for a step
  - as the environment for a job
  - to provide services for a job

[github-script]: https://github.com/marketplace/actions/github-script
[github-cli]: https://github.com/cli/cli
[github-rest-api]: https://docs.github.com/en/rest
[`matrix` context]: https://docs.github.com/en/actions/learn-github-actions/contexts#matrix-context
[`repository_dispatch` event]: https://docs.github.com/en/webhooks/webhook-events-and-payloads#repository_dispatch
[`jobs.<job_id>.strategy.matrix.include`]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategymatrixinclude
[`jobs.<job_id>.strategy.matrix.exclude`]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategymatrixexclude
