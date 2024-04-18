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

e.g.

- A workflow create issue via GitHub CLI

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

### Using `github-script` action (Write script in JavaScript)

With `github-script` action, you can write a script (in JavaScript) in your workflow that uses the GitHub API and the workflow run's `context`.

| Scope                  | Entity                    | Description                                                                                                           | Notes                                                                                         |
| ---------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Workflow file          | `github` context          | Information about **workflow run** & the **event** that triggered the run.                                            |                                                                                               |
| `github-script` action | `github` object           | A pre-authenticated **`octokit/rest.js` client** with pagination plugins                                              |                                                                                               |
|                        | `context` object          | An object containing the context of the **workflow run** (and the webhook payload object that triggered the workflow) | `github-script` action's `context` is a reference to the `@actions/github`'s `context` object |
| GitHub ToolKit         | `@actions/github` package | Provides an Octokit client hydrated with the context that the current action is being run in                          | `@actions/github` package also exposes `context` object                                       |

To use the `github-script` action, you provide the `script` input with the body of the _script_ you want to write.

e.g.

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

### Invoking `GitHub REST APIs` (Write shell script)

You can directly invoke GitHub REST APIs and do a lot of things by

- using a CLI HTTP client
- with the `GITHUB_TOKEN` from workflow run's context `secrets`/`github`

e.g.

- As in the example for [Outputs | Chap 12](chap-12.md#outputs)

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

## Using a Matrix Strategy to Automatically Create Jobs

### One-Dimensional Matrices

### Multi-dimensional Matrices

### Including Extra Values

### Excluding Values

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
