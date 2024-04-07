# Chapter 7. Managing Data Within Workflows

Usually, you will need to pass data & files between jobs of within a workflow.

e.g.

- For a typical CI/CD pipeline, there are following jobs:
  - a job that does the building,
  - multiple jobs for testing,
  - a job for packaging
- The latter jobs need to use outputs from the former jobs.

These files are called _artifacts_.

Remember,

- By default, jobs run in parallel.
- Each job should run in its isolated environment.
  - For GitHub-hosted runner, this is the default behavior.
  - For self-hosted runner, a runner should be an ephemeral, just-in-time runner.

so by default, a job cannot access another job's artifacts.

To share artifacts between jobs (of a workflow), you need to _persists_ artifacts during a workflow run:

- Before each job finished, it _upload_ the artifacts to (...)
- If a job needs artifacts of another job, it download these artifacts.

## Working with Inputs & Outputs in Workflows

### Defining & Referencing Workflow Inputs

inputs
: explicitly values supplied to the workflow (by a user/process)

You can use `inputs` keyword to supply inputs to 2 kind of triggers:

- `workflow_dispatch`[^on-workflow_dispatch-inputs]: This trigger allows this workflow to be invoked manually from the Actions tab

  e.g.

  ```yml
  on:
    # Allows this workflow to be invoked manually from the Actions tab
    workflow_dispatch:
      inputs:
        title:
          description: "Issue title"
          required: true
        body:
          description: "Issue body"
          required: true
  ```

- `workflow_call`[^on-workflow_call-inputs]: This trigger allows this workflow to be called from another workflow

  e.g.

  ```yml
  on:
    # Allows this workflow to be called from another workflow
    workflow_call:
      inputs:
        title:
          required: true
          type: string
        body:
          required: true
          type: string
  ```

Regardless of the kind of trigger events, these `inputs` values can be

- references with an expression syntax `${{` & `}}`
- via the `inputs` context

e.g.

- `${{ inputs.title }}`
- `${{ inputs.body }}`

### Capturing Output

#### From a Step

A step output can be:

- Captured from a step by [setting step's output parameter][setting-an-output-parameter]

  - defining (the output) as an environment variable
  - writing to `$GITHUB_OUTPUT`

  e.g.

  ```yml
  jobs:
    setup:
      runs-on: ubuntu-latest
      steps:
        - name: Set debug
          id: set-debug-stage
          run: echo "BUILD_STAGE=debug" >> $GITHUB_OUTPUT
  ```

- Referenced in another step (in the same job):

  - via `steps` context
  - at `steps.<step-id>.outputs.<env-var-name>`

  e.g.

  ```yml
  jobs:
    setup:
      runs-on: ubuntu-latest
      steps:
        - name: Get stage
          run: echo "The build stage is ${{ steps.set-debug-stage.outputs.BUILD_STAGE }}"
  ```

#### From an Action Used in a Step

When

- a step uses an action
- that action has outputs defined (via its `action.yml` metadata),

you can capture that output in the same way.

e.g.

- An action: `Conventional Changelog Action`

  ```yml
  jobs:
    pre-release:
      steps:
        - name: Conventional Changelog Action
          id: changelog
          uses: TriPSs/conventional-changelog-action@v3.14.0
  ```

- Its `action.yml`

  ```yml
  name: "Conventional Changelog Action"
  description: "Bump version, tag commit and generates changelog with conventional commits."

  outputs:
    changelog:
      description: "The generated changelog for the new version"
    version:
      description: "The new version"
    tag:
      description: "The name of the generated tag"
  ```

- Capture the version from action's `outputs`

  ```yml
  jobs:
    build:
      # Map a step output to a job output
      outputs:
        artifact-tag: ${{ steps.changelog.outputs.version }}
  ```

#### From a Job

A job output can be:

- Captured from a job

  - with the `jobs.<job-id>.outputs`[^jobs-job_id-outputs] section (through a step output)

  e.g.

  ```yml
  jobs:
    setup:
      runs-on: ubuntu-latest

      outputs:
        build-stage: ${{ steps.set-debug-stage.outputs.BUILD_STAGE }}

      steps:
        - name: Set debug
          id: set-debug-stage
          run: echo "BUILD_STAGE=debug" >> $GITHUB_OUTPUT
  ```

- Referenced in another job:

  - via `needs` context
  - at `needs.<job>.outputs.<output value name>`

  e.g.

  ```yml
  jobs:
    setup:
    # ...
    report:
      runs-on: ubuntu-latest
      needs: setup
      steps:
        - name: Get stage
          run: echo "The build stage is ${{ needs.setup.outputs.build-stage }}"
  ```

  > [!NOTE]
  > Don't forget to ensure the first job has finished before trying to access its outputs.
  >
  > ```yml
  > jobs:
  >   setup:
  >   # ...
  >   report:
  >     needs: setup
  > ```

  > [!TIP]
  > Use an environment variable to store the value of a job outputs.
  >
  > ```yml
  > jobs:
  >   setup:
  >   # ...
  >   report:
  >     steps:
  >       - env:
  >           BUILD_STAGE: ${{ needs.setup.outputs.build-stage }}
  >         run: echo "The build stage is $BUILD_STAGE"
  > ```

## Working with Artifacts

### Defining Artifacts

artifact
: **files** or collections of files, created as the **result** of a _job or workflow run_
: _persisted_ (in GitHub Actions)
: can be _shared_ with other jobs (in the same workflow), or accessed via GitHub web interface/REST API

e.g.

- modules produced in a build job, that need to be packaged/tested in other jobs
- test files, log outputs (that needs to be look later)

> [!IMPORTANT]
> By default, GitHub will keep your artifacts (& build logs) around for 90 days for a public repository.
>
> For private repository, the maximum period is 400 days.

> [!IMPORTANT]
> GitHub Packages: a different GitHub product, repository for software packages, e.g. containers, npm packages, RubyGems...

### Uploading & Downloading Artifacts

To be persistent, artifacts needs to be uploaded, so it can be downloaded when needed.

GitHub provides two actions that you can use to upload and download build artifacts. For more information, see the [upload-artifact][upload-artifact] and [download-artifact][download-artifact] actions.

> [!TIP]
> But, [where does the upload go?][where-does-the-upload-go]

To share data between jobs:

- **Uploading files**: Give the uploaded file a name and upload the data before the job ends.
- **Downloading files**: You can only download artifacts that were uploaded during the same workflow run. When you download a file, you can reference it by name.

#### Adding Parameters (via action `inputs`)

Some actions, e.g. _checkout_, don't require additional parameters (although they may take optional ones).

However, many actions require one or more parameters to be useful.

To add parameters to an action, you use `with` keyword, as we've already done.

e.g.

- Upload an individual file

  ```yml
  jobs:
    upload:
      steps:
        - run: mkdir -p path/to/artifact
        - run: echo hello > path/to/artifact/world.txt
        - uses: actions/upload-artifact@v4
          with:
            name: my-artifact
            path: path/to/artifact/world.txt
  ```

- Upload an entire directory

  ```yml
  - uses: actions/upload-artifact@v4
    with:
      name: my-artifact
      path: path/to/artifact/ # or path/to/artifact
  ```

After an artifact has been uploaded, it is now available to

- download manually, delete (via the Github web interface)
- use with another job in the same workflow.

To download an artifact, you use `download-artifact` action with:

- `name`: Name of the artifact to download. If unspecified, all artifacts for the run are downloaded.
- `path`: Destination path. Default is $GITHUB_WORKSPACE

e.g.

```yml
jobs:
  download:
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: my-artifact
      - run: world.txt # Output: hello
```

> [!NOTE]
> For more information about artifacts, see [Storing workflow data as artifacts][storing-workflow-data-as-artifacts]

## Working with Caches

Caches

: Reusable files that don't change often between jobs or workflow runs

Usually, caches are local copy of dependencies or build outputs.

e.g.

- For Golang, caches are:

  - Downloaded module files - Go module cache[^go-module-cache]
  - Information for reuse in future builds - Go build cache[^go-build-cache]

- For pnpm, caches is the `store-dir`[^pnpm-store-dir]

> [!IMPORTANT]
> Why use caches?
>
> - The second time something is done, it should be faster.
> - Caches could save you money, because you're paying by your workflow runtime.

### Using Caches in GitHub Actions

#### Using the `cache` action

The [`cache` action][cache-action]

- creates and restores a cache identified by a unique key.
- supports a large number of programming languages
  and frameworks[^cache-implementations]

##### How `cache` action works?

The `cache` action will attempt to restore a cache based on the `key` you provide.

Every time the action run, it finds a cache that _exactly_ matches the `key`:

- If no cache _exactly_ matches the provided `key`, this is considered a `cache miss`. e.g. The first time the action run

  On a `cache miss`:

  - the action automatically creates a new cache if the job completes successfully.
  - the new cache will use the `key` you provided and contains the files you specify in `path`.

- If there is an _exact_ match to the provided `key`, this is considered a `cache hit`.

  On a `cache hit`, the action restores the cached files to the `path` you configure.

##### Creating cache

To create cache, you use the `cache` action with:

- `key` to identify the cache
- `path` to specify which files to be cached

e.g. Using `cache` action for a Go build workflow

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/cache@v3
        with:
          # path: A list of files, directories, and wildcard patterns to cache and restore
          path: |
            ~/.cache/go-build
            ~/go/pkg/mod
          # key - An explicit key for a cache entry
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
```

With this `key` - `${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}` , our cache is only restored if

- the workflow is run on the same runner
- our `go.sum` doesn't change.

##### Matching keys

By default, `cache` action finds a cache that _exactly_ matches the `key`.

You can optionally provide a list of `restore-keys` to use in case the `key` doesn't _exactly_ match an existing cache.

- `cache` action will proceed down that list (from top to bottom) looking for partial matches to the `restore-key`
- If a partial match is found, the most recent cache that matches that partial key is restored to the `path` locations.

e.g.

- Using `cache` action with `restore-keys` for a Go build workflow

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
            # restore-keys - An ordered list of prefix-matched keys to use for restoring stale cache if no cache hit occurred for key.
            restore-keys: |
              ${{ runner.os }}-go-
              ${{ runner.os }}-
  ```

#### Using a setup action's cache feature

The `setup-*` action[^setup-action], e.g. `setup-node`, `setup-go`

- uses `cache` action under the hood but requires less configuration settings.
- (manages the creation and restoration of the cache automatically).

Depend on the language, framework, each `setup-*` action cache can cache:

- the language runtime
- package dependencies
- build outputs

e.g.

- `setup-go` action can cache:
  - Dependency files & build outputs[^caching-dependency-files-and-build-outputs]
  - How about Golang distribution? You need to [populate runner's tool cache][tool-cache-on-self-hosted-runners] or add them to Runner Images[^runner-images]'s Cached Tools[^cached-tools][^runner-images-support-policy]
- `setup-node` action can cache:
  - Package manager's global packages data [^caching-global-packages-data] (not `node_modules`)
  - How about Node.js distribution? The same as for `setup-go`

### Cache scope

Caches are associated with a specific branch. A cache created for a branch in a workflow run can be accessed & restored from another workflow run for the same repository & branch.

Workflow runs can restore caches created in:

- the current branch or
- the default branch (usually `main`).

Caches can be shared between:

- jobs in a workflow
- runs of a workflow
- branches of a repository

### Cache lifecycle

GitHub Actions will delete caches that:

- have not been accessed in 7 days
- over the storage limit of 10 GB (least recently used cache will be deleted first)

### Monitoring Caches

The information about caches can be accessed via:

- Github web interface / `Actions` tab / `Management` / `Caches`
  The cache list can be:
  - sorted by create time/used time/size
  - filter by branch
- Via APIs:
  e.g.

  - Endpoint `https://api.github.com/repos/<owner>/<repository>/actions/cache/usage`
  - Output:

    ```yml
    {
      "full_name": "<owner>/<repository>",
      "active_caches_size_in_bytes": 312329569,
      "active_caches_count": 2,
    }
    ```

## Conclusion

- By default, workflows/jobs/steps don't share data:

  - Each job runs in a _isolated_ runner environment specified by `runs-on`[^using-jobs-in-a-workflow].
  - The steps of a job share the same environment on the runner machine, but run in their own individual processes[^storing-workflow-data-as-artifacts].

- To share data (that's your step/job/workflow "outputs") between:

  - Steps (in a job): you can use step's output parameter (via setting environment variable and writing to `$GITHUB_OUTPUT` file)
  - Jobs (in a workflow):
    - For simple data, you can use `inputs` & `outputs` (via a step `outputs`)
    - For files, you needs to upload/download them as `artifact`.
  - Workflows: you needs to upload/download them as `artifact`.

  | Where to share?                   | What to share?              | How to share?                                                                                                         | How to access?                                                                 | Notes                                                                                                                             | For more information                                                         |
  | --------------------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
  | Steps                             | Simple data: string, number | Step's `output parameter`s                                                                                            | Via [`steps` context][step-context] at `steps.<step-id>.outputs.<env-name>`    |                                                                                                                                   |                                                                              |
  | Jobs                              | Simple data: string, number | Job's `outputs`                                                                                                       | Via [`needs` context][needs-context] at `needs.<job-id>.outputs.<output-name>` | Use with `jobs.<job_id>.needs`[^jobs-job_id-needs] to identify any jobs that must complete successfully before this job will run. |                                                                              |
  |                                   | Complex data: files         | Upload as _artifacts_                                                                                                 | Download uploaded _artifacts_                                                  |                                                                                                                                   |                                                                              |
  | Steps/jobs in a reusable workflow | Simple data                 | Via `on.workflow_dispatch.inputs`[^on-workflow_dispatch-inputs] / `on.workflow_call.inputs`[^on-workflow_call-inputs] | Via [`inputs` context][inputs-context]                                         |                                                                                                                                   |                                                                              |
  | Workflows                         | Complex data: files         | Upload as _artifacts_ (using `cache` action or `upload-artifact` action)                                              | Download uploaded _artifacts_ (`cache` action or `download-artifact` action)   |                                                                                                                                   | See [Storing workflow data as artifacts][storing-workflow-data-as-artifacts] |

- To share data (that's your step/job/workflow needs to do their work, e.g. downloaded packages, previous build outputs, called `dependencies`), you upload/download them (just as for artifact) by using:

  - `cache` action
  - `setup-*` action's cache feature

  For more information, see [Caching dependencies to speed up workflows][caching-dependencies-to-speed-up-workflows]

[^on-workflow_call-inputs]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callinputs>
[^on-workflow_dispatch-inputs]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatchinputs>
[^jobs-job_id-outputs]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs>
[^go-module-cache]: <https://go.dev/ref/mod#module-cache>
[^go-build-cache]: <https://pkg.go.dev/cmd/go#hdr-Build_and_test_caching>
[^pnpm-store-dir]: <https://pnpm.io/npmrc#store-dir>
[^cache-implementations]: <https://github.com/actions/cache?tab=readme-ov-file#implementation-examples>
[^caching-global-packages-data]: <https://github.com/actions/setup-node?tab=readme-ov-file#caching-global-packages-data>
[^caching-dependency-files-and-build-outputs]: <https://github.com/actions/setup-go?tab=readme-ov-file#caching-dependency-files-and-build-outputs>
[^cached-tools]: <https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2204-Readme.md#cached-tools>
[^runner-images]: <https://github.com/actions/runner-images/>
[^runner-images-support-policy]: <https://github.com/actions/runner-images/tree/main?tab=readme-ov-file#software-and-image-support>
[^setup-action]: <https://github.com/actions?q=setup>
[^using-jobs-in-a-workflow]: <https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#overview>
[^storing-workflow-data-as-artifacts]: <https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts#about-workflow-artifacts>
[^jobs-job_id-needs]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds>

[where-does-the-upload-go]: https://github.com/actions/upload-artifact?tab=readme-ov-file#where-does-the-upload-go
[upload-artifact]: https://github.com/actions/upload-artifact
[download-artifact]: https://github.com/actions/download-artifact
[storing-workflow-data-as-artifacts]: https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
[cache-action]: https://github.com/actions/cache
[tool-cache-on-self-hosted-runners]: https://docs.github.com/en/enterprise-server@3.12/admin/github-actions/managing-access-to-actions-from-githubcom/setting-up-the-tool-cache-on-self-hosted-runners-without-internet-access
[setting-an-output-parameter]: https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-output-parameter
[step-context]: https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context
[needs-context]: https://docs.github.com/en/actions/learn-github-actions/contexts#needs-context
[inputs-context]: https://docs.github.com/en/actions/learn-github-actions/contexts#inputs-context
[caching-dependencies-to-speed-up-workflows]: https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows
