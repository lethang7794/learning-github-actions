# Chap 12. Advanced Workflows

As in [Starter Workflows | Chapter 1](chap-01.md#starter-workflows), GitHub provides a lot of starter workflows in several categories:

- Automation
- Continuous Integration (CI)
- Security
- Deployment (CD)
- Pages

These starter workflows provide a starting point for your workflow (when you creating a new workflow):

- If they're all you need, you are lucky.
- If they're not enough, you can modify/extend them to better suit your needs.
  - When you find yourself doing this repeatedly (with the same kind of changes), it's time to create a **starter workflow** (for your organization).

> [!IMPORTANT]
> You can only share starter workflows within an [organization][organization][^creating-starter-workflows][^sharing-workflows].

> [!CAUTION]
> Starter workflows created by users can only be used to create workflows in public repositories.
>
> Organizations using [GitHub Enterprise Cloud] can also use starter workflows to create workflows in private repositories.
>
> GitHub Enterprise Cloud is not cheap (21USD per user/month[^github-pricing])

## Creating Your Own Starter Workflows

### Creating a Starter Workflow Area

All starter workflows shared within an organization is stored at the organization's `.github` repository, in the `workflow-templates` directory.

> [!TIP]
> The `.github` repository, a special repository in GitHub, is used for many purposes depend on the type of the GitHub.
>
> - For a personal account, `.github` repository stores default [community health file].
> - For an organization account, `.github` repository stores:
>   - Starter workflows in `workflow-templates` directory.
>   - Organization profile in `profile/README.md` file.

### Creating a Starter Workflow File

starter workflow
: _template_ to create workflow
: contains

- **template** (for workflow file) with placeholder, e.g. `$default-branch`
- **metadata file**: describes how the starter workflows will be presented to users when they are creating a new workflow.

e.g.

1. A simple starter workflow from the docs

   - The starter workflow structure:

     ```bash
     # At the .github repository root
     tree
     .
     ├── workflow-templates
         ├── octo-organization-ci.yml
         ├── octo-organization-ci.properties.json
     ```

   - The template (for the workflow file):

     `octo-organization-ci.yml`

     ```yaml
     name: Octo Organization CI

     on:
       push:
         branches: [$default-branch]
       pull_request:
         branches: [$default-branch]

     jobs:
       build:
         runs-on: ubuntu-latest

         steps:
           - uses: actions/checkout@v4

           - name: Run a one-line script
             run: echo Hello from Octo Organization
     ```

   - The metadata file (for the starter workflow):

     `octo-organization-ci.properties.json`

     ```json
     {
       "name": "Octo Organization Workflow",
       "description": "Octo Organization CI starter workflow.",
       "iconName": "example-icon",
       "categories": ["Node", "CI"],
       "filePatterns": ["package.json$", "^Dockerfile", ".*\\.md$"]
     }
     ```

1. A more complex starter workflow

   - The starter workflow structure:

     ```bash
     # At the .github repository root
     tree
     .
     ├── workflow-templates
         ├── rndrepos-info.yml
         ├── rndrepos-info.properties.json
     ```

   - The template (for the workflow file):

     `rndrepos-info.yml`

     ```yaml
     name: Repo Info with context

     on:
       push:
         branches: [$default-branch]
       pull_request:
         branches: [$default-branch]

     jobs:
       info:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v3
           - name: Show repo info
             env:
               GITHUB_CONTEXT: ${{ toJson(github) }}
             run: |
               echo Repository is ${{ github.repository }}
               echo Size of local repository area is `du -hs ${{ github.workspace }}`
               echo Context dump follows:
               echo "$GITHUB_CONTEXT"
     ```

   - The metadata file (for the starter workflow):

     `rndrepos-info.properties.json`

     ```json
     {
       "name": "RndRepos Info Workflow",
       "description": "RndRepos informational starter workflow.",
       "categories": ["Text"],
       "filePatterns": [".*\\.md$"]
     }
     ```

### Using the New Starter Workflow

After you create your starter workflow, it will show up when a user in your repository creating a new workflow (It's below the suggested workflows)

## Reusable Workflows

Rather than copying & pasting from one workflow to another, you can make workflows reusable.

You and anyone with access to the reusable workflow can then _**call** the reusable workflow_ **from another workflow**.

- A workflow that uses/call another workflow is referred to as a **"caller" workflow**:

  > [!NOTE]
  > One caller workflow can call multiple called workflows

- The reusable workflow is a **"called" workflow**.

  > [!NOTE]
  > Each called workflow is referenced in a single line.

> [!IMPORTANT]
> The caller workflow file may contain just a few lines of YAML, but may perform a large number of tasks when it's run.

### What is reusable workflow?

reusable workflow
: a workflow that can be triggered by the `workflow_call` event

> [!TIP]
> You've already known about `workflow_call` event:
>
> - [Triggering Workflow | Chap 2](./chap-02.md#triggering-workflows)
>   - [Triggering Workflows Without a Change | Chap 8](chap-08.md#triggering-workflows-without-a-change)
>     - [Defining & Referencing Workflow Inputs | Chap 3](chap-07.md#defining--referencing-workflow-inputs)

### Where is a reusable workflow?

At the time of writing, (reusable) workflow files can only be located

- at `.github/workflows` directory
  - within that repository
  - in another repository of the organization (when you share the reusable workflow with your organization)
- NOT at its sub-directories, e.g. `.github/workflows/service-a`

For more information, see [Feature request: allow workflow configuration in sub-folders](https://github.com/orgs/community/discussions/18055)

### How to use a reusable workflow?

To call a reusable workflow, you use `jobs.<job_id>.uses`[^jobs-job_id-uses] with 1 of 2 syntaxes:

1. `./.github/workflows/{filename}`: for reusable workflows in the same repository.

1. `{owner}/{repo}/.github/workflows/{filename}@{ref}` for reusable workflows in public and private repositories.

   `{ref}`: can be a SHA, a release tag, or a branch name

e.g.

- ```yml
  jobs:
    call-workflow-1-in-local-repo:
      uses: ./.github/workflows/workflow-1.yml
    call-workflow-2-in-local-repo:
      uses: octo-org/this-repo/.github/workflows/workflow-2.yml@v3
    call-workflow-in-another-repo:
      uses: octo-org/another-repo/.github/workflows/workflow.yml@v1
  ```

> [!IMPORTANT]
> Each reusable workflow, i.e. called workflow, is _run **as a job**_ of the caller workflow.

> [!WARNING]
> For calling reusable workflows to succeed, the access needs to be allow from 2 sides:
>
> - The caller workflow can called the called workflow. See [Actions Permissions | Chap 9](chap-09.md#actions-permissions)
> - The called workflow allows it to be called by the caller workflow (by allowing caller workflow access to called workflow's _private_ repository). See [Sharing actions and workflows with your organization]

### Inputs and Secrets

### Outputs

### Limitations

## Required Workflows

### Constraints

### Example

### Execution

## Conclusion

[organization]: https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations
[GitHub Enterprise Cloud]: https://docs.github.com/en/get-started/onboarding/getting-started-with-github-enterprise-cloud
[community health file]: https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file

[^sharing-workflows]: <https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization#sharing-workflows>
[^creating-starter-workflows]: <https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization>
[^github-pricing]: <https://github.com/pricing>
[^jobs-job_id-uses]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_iduses>

```

```

[Sharing actions and workflows with your organization]: https://docs.github.com/en/actions/creating-actions/sharing-actions-and-workflows-with-your-organization
