# Chap 14. Migrating to GitHub Actions

GitHub Actions can be implemented to replace almost any current automation framework.

You can use _GitHub Actions Importer_ to plan, test and **_automate_ migration** to GitHub Actions from the following platforms:

- Azure DevOps
- Bamboo<sup>\*</sup>
- Bitbucket Pipelines<sup>\*</sup>
- **CircleCI**
- GitLab
- **Jenkins**
- **Travis CI**

and can achieve an **80% conversion** rate for every workflow.

\*: Limited support

## Prep

### Source Code

GitHub Actions workflows are tied to a GitHub repository, which is built upon Git[^1], just like almost everything in **Git**Hub.

If your source code doesn't use Git, you need to migrate to git. For more information about migrating to Git, see

- GitHub guides to import source code from:

  - a Subversion repository
  - a Mercurial repository
  - a Team Foundation Version Control repository

  [Source code imports | Migrations | GitHub Docs](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/about-source-code-imports-using-the-command-line)

- [Git and Other Systems - Migrating to Git | Pro Git](https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git)

Now your source code is in Git repository, you need to consider the [migration type](https://docs.github.com/en/migrations/overview/planning-your-migration-to-github#about-migration-types):

- Source snapshot
- Source and history
- Source, history, and metadata

### Automation

You need to answer:

- Move all projects or only a subset?
- Are there any parts of the existing automation
  - not essential?
  - not longer used?
  - use custom/ad-hoc scripting, kludges?
  - use outdated plug-ins, modules? Are there any equivalent actions to replace them?
  - broken? need to be fixed?

### Infrastructure

- Which runner? GitHub-hosted or self-hosted?
- Are there any:
  - Custom setups/configuration?
  - Custom applications?
  - Specific application versions? OSes?
- For Github-hosted runner:
  - Can each job run on an ephemeral VM?
  - Are there any Mac/Windows systems? Can they switch to Linux?

### Users

Consider:

- The permissions for team member to each repository?

  - Who are the administrators?
  - Who are the contributors?
  - Is it time to cleanup those permissions?

- Do they know? What they say about the timing? Do they all have GitHub accounts?

- Do they have appropriate training about GitHub, Github Actions?

You should try to ensure that the migration to GitHub Actions is as smooth & painless as possible for your users:

- Before the migration,

  - remove any unnecessary things: source code, automation, infrastructure, users, user access.
  - standardize as much as possible

- Ensure you are compliant with security mandates.

  Explain to your users about these compliance, and tell them why?

- Allow sufficient time to do the migration.

  There will be a lots of questions & problems.

- Track the process formally

  Make it's clear what stage of migration each repository is in.

- Require appropriate training for all team members that will be working with the migrated content.

- Set up a test repository with all pieces associated ASAP.

  Give all people involved access and have them run through a full "workflow":

  - Making source change
  - Doing a pull request
  - Looking at the automation runs in GitHub Actions

## Compare other platform concepts with GitHub Actions

The similarities & key differences between GitHub Actions and other platform is available in [Manually Migrating | GitHub Docs](https://docs.github.com/en/actions/migrating-to-github-actions/manually-migrating-to-github-actions) for:

- Azure Pipelines
- CircleCI
- GitLab CI/CD
- Jenkins
- Travis CI

See this [book](https://learning.oreilly.com/library/view/learning-github-actions/9781098131067/ch14.html#id153) for a more in-depth comparison.

## GitHub Actions Importer

Migrate a few pipelines is usually not too difficult. However, what if you have hundreds, thousands pipelines to migrate?

GitHub Actions Importer is a tool that help you import a large number of pipelines from other platforms to GitHub Actions.

The recommended way to use GitHub Actions Importer (available via CLI `gh actions-importer`):

1. `configure`: Configure credentials used to authenticate with your CI server(s)
2. `audit`: Plan your CI/CD migration by analyzing your current CI/CD footprint
3. `forecast`: Forecast GitHub Actions usage from historical pipeline utilization
4. `dry-run`: Convert a pipeline to a GitHub Actions workflow and output its YAML file
5. `migrate`: Convert a pipeline to a GitHub Actions workflow and open a pull request with the changes

### Configure Authentication

The importer tool has to access both platforms:

- The platform you are converting from
- The target repos in GitHub

For each platform, the importer tool need to know:

- Personal access token
- Base url of the instance
- ...

GitHub Actions Importer get these information from environment variables in the `.env.local` file.

You can

- manually configure the environment variables for each platform.

- use the interactive prompt provided by `gh actions-importer configure`

  <details>
  <summary>Example</summary>

  ```shell
  gh actions-importer configure
  ✔ Which CI providers are you configuring?: Jenkins
  Enter the following values (leave empty to omit):
  ✔ Personal access token for GitHub: *******************************
  ✔ Base url of the GitHub instance: https://github.com
  ✔ Personal access token for Jenkins: *******************************
  ✔ Username of Jenkins user: admin
  ✔ Base url of the Jenkins instance: http://localhost:8080/
  Environment variables successfully updated.
  ```

  ```shell
  cat .env.local
  GITHUB_ACCESS_TOKEN=ghp_P73jshbAcUmCQvaOyAIuxNUE---------
  GITHUB_INSTANCE_URL=https://github.com
  JENKINS_ACCESS_TOKEN=117e5929321809d5eeb9a91684--------
  JENKINS_INSTANCE_URL=http://localhost:8080/
  JENKINS_USERNAME=admin
  ```

  </details>

### Planning - Audit summary

`gh actions-importer audit` help:

- Analyzing your current CI/CD footprint

- Plan your CI/CD migration:
  - Try and run a conversion
  - Produce an `audit summary`

The `audit summary` contains the following sections:

- Pipelines
- Build steps
- Manual tasks
- Files

#### Pipelines: High level statistics regarding the conversion rate

- Pipelines

  - Total
    - Successful
    - Partially successful
    - Unsupported
    - Failed

- Job types: The job types used by the original platform

  - Supported
  - Unsupported: e.g. scripted

#### Build Steps: Overview of the individual build steps that are used across all pipelines

- Total
  - Known
  - Unknown
  - Unsupported
  - Actions

#### Manual Tasks: Tasks that you will need to manually perform

- Total
  - Secrets
  - Self hosted runners

#### Files Manifest: A manifest of all of the files that are written to disk during the audit

Each pipeline will have a variety of files written that include:

- The original pipeline as it was defined in Jenkins.
- Any network responses used to convert a pipeline.
- The converted workflow.
- Stack traces that can used to troubleshoot a failed pipeline conversion

<details>
<summary>
Example
</summary>

```md
### Successful

#### demo_pipeline

- [demo_pipeline/.github/workflows/demo_pipeline.yml](demo_pipeline/.github/workflows/demo_pipeline.yml)
- [demo_pipeline/config.json](demo_pipeline/config.json)
- [demo_pipeline/jenkinsfile](demo_pipeline/jenkinsfile)

### Partially successful

#### monas_dev_work/monas_pipeline

- [monas_dev_work/monas_pipeline/.github/workflows/monas_pipeline.yml](monas_dev_work/monas_pipeline/.github/workflows/monas_pipeline.yml)
- [monas_dev_work/monas_pipeline/config.json](monas_dev_work/monas_pipeline/config.json)
- [monas_dev_work/monas_pipeline/jenkinsfile](monas_dev_work/monas_pipeline/jenkinsfile)
</details>

### Failed

#### groovy_script

- [groovy_script/error.txt](groovy_script/error.txt)
- [groovy_script/config.json](groovy_script/config.json)
```

</details>

#### Workflow usage: What are used by each successfully converted pipelines

`workflow_usage.csv` contains a comma-separated list of all actions, secrets, and runners that are used by each successfully converted pipeline.

<details>
<summary>
Example
</summary>

```csv
Pipeline,Runner,File path
demo_pipeline,TeamARunner,tmp/audit/demo_pipeline/.github/workflows/demo_pipeline.yml
test_freestyle_project,DemoRunner,tmp/audit/test_freestyle_project/.github/workflows/test_freestyle_project.yml
test_pipeline,TeamARunner,tmp/audit/test_pipeline/.github/workflows/test_pipeline.yml
```

</details>

For more information about planning with `gh actions-importer audit`, see

- [Perform an audit of Jenkins | GitHub Docs](https://docs.github.com/en/actions/migrating-to-github-actions/automated-migrations/migrating-from-jenkins-with-github-actions-importer#perform-an-audit-of-jenkins)
- [Audit | Importer Lab for Jenkins](https://github.com/actions/importer-labs/blob/main/jenkins/2-audit.md)

### Forecasting

`gh actions-importer forecast`

- looks at the _historical_ pipeline utilization (in other platform) with metrics for:

  - completed **job count**<sup>\*\*</sup>
  - unique **pipeline count**
  - **execution time**<sup>\*\*</sup>
  - **queue time**: wait time before a runner execute the job
  - **concurrent jobs**

  \*\*: Metrics used to estimate the cost of GitHub-hosted runners.

- forecasts _potential_ GitHub Actions build runner usage

> [!TIP]
> By default, GitHub Actions Importer includes the previous **7 days** in the forecast report.

> [!IMPORTANT]
> With _job count_ and _execution time_, you can use [GitHub Actions pricing calculator] to estimate the cost of GitHub-hosted runners.

For more information, see [Forecast potential build runner usage | Jenkins migration | GitHub Docs](https://docs.github.com/en/actions/migrating-to-github-actions/automated-migrations/migrating-from-jenkins-with-github-actions-importer#forecast-potential-build-runner-usage)

### Doing a Dry Run

dry run
: việc tập bắn không có đạn ([dry run](https://www.informatik.uni-leipzig.de/~duc/TD/td/index.php?word=dry+run) - [The Free Vietnamese Dictionary Project - Hồ Ngọc Đức](http://www.informatik.uni-leipzig.de/~duc/Dict/))
: an occasion in which you practice a particular activity or performance in preparation for the real event ([dryrun - Cambridge Dictionary](https://dictionary.cambridge.org/dictionary/english/dry-run))
: (_testing_) a software testing process used to make sure that a system works correctly and will not result in severe failure - [Wikipedia](<https://en.wikipedia.org/wiki/Dry_run_(testing)>)

`--dry-run` mode
: a mode for

- new user who is playing around with your tool
- experienced user who is trying to explore some edge case
- or who is unsure if their environment is right for the tool to do what they expect
  ([In praise of --dry-run | G-Research](https://www.gresearch.com/news/in-praise-of-dry-run/))

`gh actions-importer dry-run`

- converts a pipeline to a GitHub Actions workflow
- output its YAML file (without open a pull request)

<details>
<summary>
Example
</summary>

- Run actions-import dry-run

  ```bash
  gh actions-importer dry-run jenkins --source-url http://localhost:8080/job/test_pipeline --output-dir tmp/dry-run
  ```

- Each converted job will be created as a result workflow file at `tmp/dry-run/<job-name>/.github/workflows/<job-name>.yml`

  - The importer process may not be able to automatically convert every section/construct of the origin pipeline.
  - Those _unconvertible_ parts will be **commented out** in the resulting workflow.

    ```yaml
    jobs:
      build:
        runs-on:
          - self-hosted
          - TeamARunner
        steps:
          - name: checkout
            uses: actions/checkout@v3.3.0
          - name: echo message
            run: echo "Database engine is ${{ env.DB_ENGINE }}"
          # # This item has no matching transformer
          # - sleep:
          #   - key: time
          #     value:
          #       isLiteral: true
          #       value: 80
    ```

  - You can manually resolve these part or implement a _custom transformer_

</details>

#### Creating Custom Transformers for the Importer

GitHub Actions Importer offers the ability to extend its built-in mapping by creating custom transformers.

A custom transformer:

- converts some part of your pipeline that the importer doesn’t handle automatically.
- contains mapping logic that GitHub Actions Importer can use to transform your plugins, tasks, runner labels, or environment variables to work with GitHub Actions.

Custom transformers are written with a domain-specific language (DSL) built on top of _Ruby_, and are defined within a file with the `.rb` file extension.

<details>
<summary>
Example
</summary>

- Jenkin's item to sleep that's can be automatically converted

  ```yaml
  # # This item has no matching transformer
  # - sleep:
  #   - key: time
  #     value:
  #       isLiteral: true
  #       value: 80
  ```

- It can be replace with a simple shell command implemented as a job's step

  ```yml
  - name: Sleep for 80 seconds
    run: sleep 80s
    shell: bash
  ```

- The custom transformer looks like this

  ```ruby
  # sleep-transformer.rb
  transform "sleep" do |item|
    wait_time = item["arguments"][0]["value"]["value"]

    {
      "name": "Sleep for #{wait_time} seconds",
      "run": "sleep #{wait_time}s",
      "shell": "bash"
    }
  ```

- Provide the transformer to the importer

  ```bash
  gh actions-importer dry-run jenkins \
    --source-url http://localhost:8080/job/test_pipeline \
    --output-dir tmp/dry-run --custom-transformers sleep-transformer.rb
  ```

  </details>

### Doing the Actual Migration

After completed auditing, done dry-run, the final step is doing the actual migration.

> [!TIP]
> Don't forget to ensure the target repository has been created/

`gh actions-importer migrate` will do everything the `dry-run` do and open a pull request with the changes.

- You can review the code in the pull request. If there is something you want to change, you can start over with the `migrate` step.
- If there is any manual steps needed, it will be included in the pull request for you.
- Once the manual steps and any code reviews of the pull request are completed, the pull request can be merged, and the workflow will have been successfully migrated.

## Conclusion

- For a long time, GitHub has been integrated with different automation platforms.

  - With GitHub Actions, you can choose to tightly integrate with automation built into GitHub.
  - GitHub Actions Importer can help you to plan, test, automate migration to GitHub Actions from other popular platforms.

- GitHub Actions Importer includes the functionality to:

  - Analyze yor current CI/CD footprint.
  - Forecast GitHub Actions usage from historical pipeline utilization.
  - Dry-run pipelines from other platforms and output in GitHub Actions equivalent in workflow file.
  - Migrate pipelines from other platforms and open a pull request with the changes.

- For the small parts that can't be default automate migrated, GitHub Actions Importer offers the ability to extend its built-in mapping.

  - Custom transformer can be written and pulled in when running the importer.

[^1]: Git is a version control system that intelligently tracks changes in files. [About Git | GitHub Docs](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git#about-git)

[GitHub Actions pricing calculator]: https://github.com/pricing/calculator
