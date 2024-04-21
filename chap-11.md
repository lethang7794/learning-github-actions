# Chap 11. Creating Custom actions

In [chap 3](/chap-03.md), you've already known:

- What is an action? Interface of an action
- How to use an action?

Why creating custom actions?

- Action is a module of code, you create an action to organize your code.

- Sometimes the shell language is not enough, you need a more powerful language. Currently, GitHub only "natively" supports JavaScript.

  > [!NOTE]
  > JavaScript actions run directly on the runner and use binaries that already exist in the runner image[^creating-a-javascript-action][^javascript-actions].

  > [!TIP]
  > Some guys have found a way to write action in Go and make their action run directly on the runner[^how-we-write-github-actions-in-go.html].

- Sometimes use need to do tasks that works on all runner OSes, you can write a Docker action.

You can create a custom action:

- use it as a `local action`
- share it:
  - with your organization[^sharing-actions-and-workflows-with-your-organization]
  - with everyone via the [Actions Marketplace]

## Anatomy of an action

All actions require a metadata file (`action.yml` or `action.yaml`)

The data in the metadata file defines:

- basic information: **`name`**, **`description`**, `author`
- `inputs`,
- `outputs`

- **`runs`** configuration:
  - `using`: `composite` | `node20` | `docker`

for your action.

e.g.

- Metadata (`action.yml`[^cache-action-metadata]) of [`cache` action][cache-action]

  ```yml
  name: "Cache"
  description: "Cache artifacts like dependencies and build outputs to improve workflow execution time"
  author: "GitHub"

  inputs:
    path:
      description: "A list of files, directories, and wildcard patterns to cache and restore"
      required: true
    key:
      description: "An explicit key for restoring and saving the cache"
      required: true
    restore-keys:
      description: "An ordered list of keys to use for restoring stale cache if no cache hit occurred for key. Note `cache-hit` returns false in this case."
      required: false
    # ...
    # ... A lot of other inputs

  outputs:
    cache-hit:
      description: "A boolean value to indicate an exact match was found for the primary key"

  runs:
    using: "node20"
    main: "dist/restore/index.js"
    post: "dist/save/index.js"
    post-if: "success() || github.event.inputs.save-always"

  branding:
    icon: "archive"
    color: "gray-dark"
  ```

### `inputs` section - input parameters

input parameters (`inputs`) specify data that the action expects to use during runtime

e.g.

- `cache` action has a 2 required input parameters `path` & `key`:

  ```yml
  inputs:
    path:
      description: "A list of files, directories, and wildcard patterns to cache and restore"
      required: true
    key:
      description: "An explicit key for restoring and saving the cache"
      required: true
  ```

- When you use the `cache` action, you must provides the values for these 2 input parameters (as in [Creating cache | Chap 7](/chap-07.md#creating-cache))

  ```yml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/cache@v3
          with:
            path: |
              ~/.cache/go-build
              ~/go/pkg/mod
            key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
  ```

- When the action is run on a runner, an environment variable for each input parameter is created. This is the way that the values get transferred to the running process (for that step)[^step-process].

> [!NOTE]
> The environment variables will have a name like `INPUT_<PARAMETER_NAME>` with letters converted to uppercase and spaces replaced with underscores.

### `outputs` section - Output parameters

Output parameters (`outputs`) allow you to declare data that an action sets.

Actions that run later in a workflow (other steps) can use the output data set in previously run actions.

e.g.

- A action that sum 2 inputs, should have an output for the sum (so other actions can use as their input)

  ```yml
  outputs:
    sum: # id of the output
      description: "The sum of the inputs"
  ```

- `cache` action has one output parameter

  ```yml
  outputs:
    cache-hit: # id of the output
      description: "A boolean value to indicate an exact match was found for the primary key"
  ```

  Another step can use the output of the `cache` action to make decision:

  ```yml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - name: Cache Primes
          id: cache-primes
          uses: actions/cache@v3
          with:
            path: prime-numbers
            key: ${{ runner.os }}-primes
        - name: Generate Prime Numbers
          if: steps.cache-primes.outputs.cache-hit != 'true'
          run: /generate-primes.sh -d prime-numbers
  ```

> [!CAUTION]
> If you don't declare an output in your action metadata file, you can still set outputs and use them in a workflow.
>
> The output declaration only acts as a documentation.

## 3 Types of Actions

- `Composite action`: an action implemented with steps, scripting.
- `JavaScript action`: an action implemented with JavaScript (and still run directly on a runner machine).
- `Docker container action`: an action runs within a Docker container.

### Composite Action

Although having a scary name, composite action is the simplest one.

A composite action allows you to combine multiple [steps][action-runs-steps] within one action.

> [!NOTE]
> Each step of a composite action acts just like a step of job. It can
>
> - `run`[action-runs-steps-run] a shell command
> - `uses`[action-runs-steps-uses] an action

e.g.

- The script

  ```bash
  # count-args.sh
  #!/bin/bash

  args=($@)
  echo ${#args[@]}
  ```

- The `action.yml`

  ```yml
  name: "Argument Counter"
  description: "Count # of arguments passed in"

  inputs:
    arguments-to-count: # input id
      description: "arguments to count"
      required: true
      default: ""

  outputs:
    arg-count:
      description: "Count of arguments passed in"
      value: ${{ steps.return-result.outputs.num-args }}

  runs:
    using: "composite"
    steps:
      - name: Print arguments if any
        run: |
          echo Arguments: ${{ inputs.arguments-to-count }}.
        shell: bash

      - id: return-result
        run: |
          echo "num-args=`${{ github.action_path }}/count-args.sh ${{ inputs.arguments-to-count }}`" >> $GITHUB_OUTPUT
        shell: bash
  ```

- Use the action an a workflow

  ```yml
  workflow_dispatch:
    inputs:
      myValues:
      description: "Input Values"

  jobs:
    count-args:
      runs-on: ubuntu-latest
      steps:
        - id: report-count
          uses: skillrepos/arg-count-action@main
          with:
            arguments-to-count: ${{ github.event.inputs.myValues }}
        - run: echo
        - shell: bash
          run: |
            echo argument count is ${{ steps.report-count.outputs.arg-count }}
  ```

### JavaScript Action

javascript action
: an action written in JavaScript, run directly on runner machine

JavaScript Action:

- let you escape the limit of scripting languages (with JavaScript 🤯).
- runs directly on runner machine.
- is official supported with [_GitHub Actions Toolkit_][actions/toolkit], which provides a set of packages to make creating actions easier:
  - [`@actions/core`][@actions/core]: provides interfaces to read action inputs, set outputs, export variables, set secrets...
  - [`@actions/github`][@actions/github]: provides a HTTP client (`octokit`) to interact with GitHub APIs[^octokit-rest]

e.g. A JavaScript action to count the number of arguments passed in

- The `action.yml` metadata:

  ```yml
  name: "Argument Counter"
  description: "Count # of arguments passed in"

  inputs:
    arguments-to-count: # input id
      description: "arguments to count"
      required: true
      default: ""

  outputs:
    argcount:
      description: "Count of arguments passed in"

  runs:
    using: "node16"
    main: "index.js"
  ```

- The JavaScript code in `index.js` file

  ```js
  // simple demo file for javascript github action
  const core = require("@actions/core");
  const github = require("@actions/github");

  try {
    // `arguments-to-count` input defined in action metadata file
    const inputArgs = core.getInput("arguments-to-count");

    console.log(`Arguments = ${inputArgs}!`);

    const argCount = inputArgs.split(/\s+/).length;

    core.setOutput("argcount", argCount);

    // Get the JSON webhook payload for the event that triggered the workflow
    const payload = JSON.stringify(github.context.payload, undefined, 2);
    console.log(`The event payload: ${payload}`);
  } catch (error) {
    core.setFailed(error.message);
  }
  ```

- `README.md`

  ````md
  # Count arguments javascript action

  This action prints out the number of arguments passed in

  ## Inputs

  ## `arguments-to-count`

  **Required** The arguments to count.

  ## Outputs

  ## `argcount`

  The count of the arguments.

  ## Example usage

  ```yaml
  uses: <repo>/arg-count-javascript-action@v1
  with:
  arguments: <arguments>
  ```
  ````

> [!IMPORTANT]
> GitHub Actions provides a [template][actions/javascript-action] to bootstrap the creation of JavaScript action.
>
> This template includes compilation support, tests, a validation workflow, publishing, and versioning guidance.

> [!TIP]
> There is also a [TypeScript template][actions/typescript-action] for JavaScript action.

### Docker Container Action

docker container action
: an action that packaged with the environment (via Docker container)

The image for the container can be provided via:

- a Dockerfile
- pre-build Docker image

> [!WARNING]
> Currently, only _public_ Docker images can be used with Docker container action.

> [!IMPORTANT]
> Dockerfile support for GitHub Actions
>
> - Some Docker instructions interact with GitHub Actions
> - An action's **metadata file** can override some Docker instructions.

e.g. A Docker container action to count the number of arguments passed in

- The `Dockerfile`

  ```Dockerfile
  # Base image to execute code
  FROM alpine:3.3

  # Add in bash for our shell script
  RUN apk add --no-cache bash

  # Copy in entrypoint startup script
  COPY entrypoint.sh /entrypoint.sh

  # Script to run when container starts
  ENTRYPOINT ["/entrypoint.sh"]
  ```

  > [!TIP]
  > The Docker `ENTRYPOINT` instruction has 2 forms:
  >
  > - `exec` form, e.g. `ENTRYPOINT ["echo $MY_VAR"]`
  > - `shell` form, e.g. `ENTRYPOINT ["sh", "-c", "echo $MY_VAR"]`
  >
  > The Docker ENTRYPOINT documentation recommends using the `exec` form.
  >
  > But if you need to substitute variable, you need to use `shell` form.

- The `entrypoint.sh`

  ```bash
  #!/bin/bash

  args=($@)
  argcount="${#args[@]}"
  echo "argcount=$argcount" >> $GITHUB_OUTPUT
  ```

- The `README.md`

  ````markdown
  # Count arguments docker action

  This action prints out the number of arguments passed in

  ## Inputs

  ## `arguments to count`

  **Required** The arguments to count.

  ## Outputs

  ## `argcount`

  The count of the arguments.

  ## Example usage

  ```yaml
  uses: <repo>/arg-count-docker-action@v1
  with:
    arguments: <arguments>
  ```
  ````

- The `action.yml`

  ```yml
  name: "Argument Counter"
  description: "Count # of arguments passed in"

  inputs:
    arguments-to-count: # input id
      description: "arguments to count"
      required: true
      default: ""

  outputs:
    argcount:
      description: "Count of arguments passed in"
  ```

  1. Provide the `Dockerfile`

     ```yml
     runs:
       using: "docker"
       image: "Dockerfile"
       args:
         - ${{ inputs.arguments-to-count }}
     ```

  2. Provide the pre-build image

     ```yml
     runs:
       using: "docker"
       image: "docker://quay.io/techupskills2/arg-count-action:1.0.1"
       args:
         - ${{ inputs.arguments-to-count }}
     ```

## Completing Your Action Creation

In [Using actions | Chap 3](./chap-03.md#using-actions), as a user of an action, you can refer to a version of an action by its:

1. commit hash SHA.
1. tag (Git tag, Docker tag).
1. branch, e.g. `main` - You want to use the most recent code.

The recommenced way to use refer a version of an action is second option - using a Git tag.

As the author of an action, you have the responsibility to naming the tag of the released version with semantic versioning (`major.minor.patch`).

> [!NOTE]
> It's a convention to point the tags with only the `major` portion (v1, v2...) to the most recent version (with that `major` version) as the new ones created.
>
> e.g.
>
> - Currently, `v1` points to `v1.2.3`
> - You released version `v1.2.4`:
>   - `v1`should point to the `v1.2.4` now.

## Publishing Actions on the GitHub Marketplace

> [!NOTE]
> Actions are published to GitHub Marketplace immediately and aren't reviewed by GitHub[^https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace]

> [!CAUTION]
> Because anyone can publish any code as an action, be caution when using an unknown action in the GitHub Marketplace for Actions.

To be publish to the GitHub Marketplace, an action:

- Must be in a _public_ repository.
  - That repository only contains a single action.
  - The metadata of the action (`action.yml` / `action.yaml`) must be in the root directory of the repository.
    - The `name` in the `action.yaml must be unique (across the GitHub Marketplace).

> [!TIP]
> An action in your _private_ repository
>
> - cannot be published to the GitHub Marketplace
> - but can be shared to other repositories
>   - of you.
>   - of your organization.
>
> For more information, see [Allowing access to components in a private repository]

If your action's repository has all these conditions, there will be a `Draft a release` button.

To release your repository:

- Click the `Draft a release` button
- Fill out `Release Action` form:
  - (Read the agreement)
  - Tick the `accept the GitHub Marketplace Developer Agreement` checkbox.
    - Fill out information for the action
      - Name
      - Description
      - Icon
      - Color
      - Categories
  - From the `Choose a tag` dropdown
    - Choose a tag
    - or create a new tags
  - Choose the target branch
  - Enter the title (for the release)
  - Enter the description (for the release)
  - Click `Publish release` button

### Updating Actions on the Marketplace

The updating process for an action follow the same process as a normal code changes:

- Anyone can forks the repository, makes a pull request to your repository.
- You, as the owner of the repo, can review the pull request, and decide whether to accept it.

### Removing an Action from the Marketplace

If you're the owner of an action, you can remove that action from the Marketplace by using the `Delist` button on the marketplace page of the action.

## The Actions Toolkit

In additional to `@actions/core` & `@actions/github` as in [JavaScript Action](#javascript-action):

| Package           | Purpose                                                                                  |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `@actions/core`   | Provides interfaces to read action inputs, set outputs, export variables, set secrets... |
| `@actions/github` | Provides a HTTP client (`octokit`) to interact with **GitHub APIs**[^octokit-rest]       |

the Actions Toolkit also also has:

| Package                | Purpose                                                                                         |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| `@actions/exec`        | Provides functions to **exec cli tools** and process output                                     |
| `@actions/glob`        | Provides functions to **search for files** matching glob patterns                               |
| `@actions/http-client` | A lightweight HTTP client optimized for building actions                                        |
| `@actions/io`          | Provides disk i/o functions like `cp`, `mv`, `rmRF`, `which`, etc.                              |
| `@actions/tool-cache`  | Provides functions for **downloading & caching tools**, e.g., `setup-*` actions                 |
| `@actions/artifact`    | Provides functions to **interact with artifacts**                                               |
| `@actions/cache`       | Provides functions to **cache dependencies & build outputs** to improve workflow execution time |

To use the packages of the Actions Toolkit, you

- Install the npm package

  ```bash
  npm install @actions/core
  ```

- Import the package in your JavaScript code

  ```javascript
  const core = require("@actions/core");
  ```

  > [!TIP]
  > You can use TypeScript to have additional information about the types
  >
  > ```typescript
  > import * as core from "@actions/core";
  > ```

- Invoke the functions provided by that package

  ```javascript
  const myInput = core.getInput("inputName", { required: true });

  core.setOutput("outputKey", "outputVal");

  core.exportVariable("envVar", "envVal");

  core.setSecret("myPassword");
  ```

### Using Workflow Commands from the Toolkit

The `actions/toolkit` includes a number of functions that can be executed as workflow commands[^using-workflow-commands-to-access-toolkit-functions].

| Toolkit function      | Equivalent workflow command | Accessible using <br/> environment _variable_ | Accessible using <br/> environment _file_ |
| --------------------- | --------------------------- | --------------------------------------------- | ----------------------------------------- |
| `core.addPath`        |                             |                                               | `GITHUB_PATH`                             |
| `core.debug`          | `debug`                     |                                               |                                           |
| `core.notice`         | `notice`                    |                                               |                                           |
| `core.error`          | `error`                     |                                               |                                           |
| `core.endGroup`       | `endgroup`                  |                                               |                                           |
| `core.exportVariable` |                             |                                               | `GITHUB_ENV`                              |
| `core.getInput`       |                             | `INPUT_{NAME}`                                |                                           |
| `core.getState`       |                             | `STATE_{NAME}`                                |                                           |
| `core.isDebug`        |                             | `RUNNER_DEBUG`                                |                                           |
| `core.summary`        |                             |                                               | `GITHUB_STEP_SUMMARY`                     |
| `core.saveState`      |                             |                                               | `GITHUB_STATE`                            |
| `core.setCommandEcho` | `echo`                      |                                               |                                           |
| `core.setFailed`      | `error` and `exit 1`        |                                               |                                           |
| `core.setOutput`      |                             |                                               | `GITHUB_OUTPUT`                           |
| `core.setSecret`      | `add-mask`                  |                                               |                                           |
| `core.startGroup`     | `group`                     |                                               |                                           |
| `core.warning`        | `warning`                   |                                               |                                           |

e.g. To create an error annotation, you can:

- Use the `@actions/core`'s `error` function (in your action code)

  ```javascript
  core.error("Missing semicolon", { file: "app.js", startLine: 1 });
  ```

- Or use the `error` workflow command (in your workflow file)

  - Syntax: `::error file={name},line={line},endLine={endLine},title={title}::{message}`

  - e.g.

    ```yaml
    steps:
      - name: Create annotation for build error
        run: echo "::error file=app.js,line=1::Missing semicolon"
    ```

> [!IMPORTANT]
> The question is:
>
> - The workflow commands is a way to invoke the toolkit function?,
> - or the inverse - the toolkit function is a way to invoke the workflow command?

## Local actions

local action
: an action that has the code in the same repository as the workflow that reference it

e.g.

- `.github/actions/hello-world-action/dist/index.js`

  ```javascript
  const core = require("@actions/core");
  const github = require("@actions/github");

  /**
   * The main function for the action.
   * @returns {Promise<void>} Resolves when the action is complete.
   */
  async function run() {
    try {
      // The `who-to-greet` input is defined in action metadata file
      const whoToGreet = core.getInput("who-to-greet", { required: true });
      core.info(`Hello, ${whoToGreet}!`);

      // Get the current time and set as an output
      const time = new Date().toTimeString();
      core.setOutput("time", time);

      // Output the payload for debugging
      core.info(
        `The event payload: ${JSON.stringify(github.context.payload, null, 2)}`
      );
    } catch (error) {
      // Fail the workflow step if an error occurs
      core.setFailed(error.message);
    }
  }

  run();
  ```

- `.github/actions/hello-world-action/action.yml`

  ```yml
  name: Hello, World!
  description: Greet someone and record the time
  author: GitHub Actions

  # Define your inputs here.
  inputs:
    who-to-greet:
      description: Who to greet
      required: true
      default: World

  # Define your outputs here.
  outputs:
    time:
      description: The time we greeted you

  runs:
    using: node20
    main: dist/index.js
  ```

- `.github/workflows/my-workflow.yaml`

  ```yml
  name: My Workflow

  on:
    workflow_dispatch:
      inputs:
        who-to-greet:
          description: Who to greet in the log
          required: true
          default: "World"
          type: string

  jobs:
    say-hello:
      name: Say Hello
      runs-on: ubuntu-latest

      steps:
        - name: Print to Log
          id: print-to-log
          uses: ./.github/actions/hello-world-action
          with:
            who-to-greet: ${{ inputs.who-to-greet }}
  ```

> [!NOTE]
> The path, to the directory that contains the action in your workflow's repository, is relative (`./`) to the default working directory (`github.workspace`)

> [!NOTE]
> For `github.workspace`:
>
> - The default working directory on the runner for steps.
> - When using the `checkout` action: the default location of your repository

For more information, see [Example: Using an action in the same repository as the workflow] at [`jobs.<job_id>.steps[*].uses` | Workflow syntax][jobs-job_id-steps-uses]

> [!NOTE]
> Besides using a _local action_, you can also use a "local" workflow, which is also a _reusable workflow_.

## Conclusion

You can create you own custom action with:

- composite action: consists of running steps (similar to a job)
- JavaScript action: use a more powerful programming language (JavaScript) with the official Actions Toolkit.
- Docker container action: bundle the environment (or use any programming language)

A custom action needs to have:

- Have the implement, i.e. the code
- The action metadata (`action.yaml`/`action.yml`): defines
  - inputs
  - outputs
  - run configurations

[Actions Marketplace]: https://github.com/marketplace?type=actions
[cache-action]: https://github.com/actions/cache
[actions/toolkit]: https://github.com/actions/toolkit
[@actions/core]: https://www.npmjs.com/package/@actions/core
[@actions/github]: https://www.npmjs.com/package/@actions/github
[actions/javascript-action]: https://github.com/actions/javascript-action
[actions/typescript-action]: https://github.com/actions/typescript-action
[action-runs-steps]: https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runssteps
[action-runs-steps-run]: https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runsstepsrun
[action-runs-steps-uses]: https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runsstepsuses
[Allowing access to components in a private repository]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-a-private-repository
[Example: Using an action in the same repository as the workflow]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-an-action-in-the-same-repository-as-the-workflow
[jobs-job_id-steps-uses]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsuses

[^sharing-actions-and-workflows-with-your-organization]: <https://docs.github.com/en/actions/creating-actions/sharing-actions-and-workflows-with-your-organization>
[^creating-a-javascript-action]: <https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action>
[^javascript-actions]: <https://docs.github.com/en/actions/creating-actions/about-custom-actions#javascript-actions>
[^how-we-write-github-actions-in-go.html]: <https://full-stack.blend.com/how-we-write-github-actions-in-go.html>
[^cache-action-metadata]: <https://github.com/actions/cache/blob/main/action.yml>
[^step-process]: Each step run in their own individual process, whether it's `run` (a shell command) or `use` (an action).
[^octokit-rest]: <https://octokit.github.io/rest.js>
[^https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace]: <https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace>
[^using-workflow-commands-to-access-toolkit-functions]: <https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions?tool=bash#using-workflow-commands-to-access-toolkit-functions>
