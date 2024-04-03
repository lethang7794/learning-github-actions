# Chapter 4. Working with Workflows

## Creating the First Workflow in a Repository

There are a lot of way to create a new workflow for a repository

- Manually add the workflow file to `<repository>/.github/workflows` directory.
- In GitHub repository's "Actions" tab
  - Use **set up a workflow yourself** to add a blank workflow file.
  - Use a **suggested workflow** for this repository.
  - Use a **starter workflows** provided by Github.

> [!NOTE]
> GitHub web interface allows you to edit files, then directly commit to the repository.

> [!TIP]
> Based on the code in the repository, Github web interface will suggest
>
> - workflows when you create a new workflow ("suggested workflows")
> - actions when you edit a workflow ("featured actions")

## Committing the Initial Workflow

```yml
# .github/workflows/simple-proj.yml

# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This job checks out code from the repo
  checkout:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3
  
  # Another jobs with 2 steps
  process:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    steps:
      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
```

- This workflows will be invoked when:
  - A `push`/`pull request` trigger events to the `main` branch happens.
  - A manually invocation with the `Run workflow` button on the GitHub web interface.

## Update the workflow

There are also a lot of way to update the workflow

- Clone the repository, edit the file locally, commit and push to back to the repository.
- Edit a file directly using Github web interface.
- Edit the whole repository using VS Code in the browser.

> [!TIP]
> To edit a file directly using Github web interface, you open the file, then click `Edit this file`.
>
> To edit the whole Github repository, open the repository:
>
> - Press `.`
> - Or change the url from `github.com` to `github.dev`

> [!TIP]
> To make sure the default branch (e.g. `main`) isn't accidentally modified, you can use "Require status checks to pass" of:
>
> - Branch protection rule[^branch-protection-rule]
> - Repository Rules[^repository-rule] (aka `ruleset`s[^ruleset] ): more flexible
>
> The check can be based on the status of:
>
> - a job, or
> - a workflow
>
> depends on the type of rule is used.

## Using the VS Code GitHub Actions Extension

VS Code has an extension for Github Actions (`GitHub Actions for VS Code`[^vscode-github-actions]), which supports linting & code completion.

## Conclusion

- GitHub web interface allows you interact with GitHub Actions directly in the browser:
  - creating & editing workflow
  - executing workflows
  - checking workflow logs

- GitHub Actions has a lot of starter workflows that are ready-to-use, and easily modified to fit your needs.

[^branch-protection-rule]: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule>
[^ruleset]: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets>
[^vscode-github-actions]: <https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-github-actions>
[^repository-rule]: <https://github.blog/2023-07-24-github-repository-rules-are-now-generally-available/>
