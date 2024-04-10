# Chap 9. Actions and Security

GitHub Actions is tightly integrated with GitHub - a platform for _collaboration_

- to achieve automation tasks that is highly integrated with your repositories and execution environment

- with the cost of carrying a high risk allowing security vulnerabilities.

As a user of GitHub Actions, you have the responsibility for the security of your workflows:

- _Before_ workflows run - **Security by Configuration**: Configure your repository & execution environment to control:

  - which actions & reusable workflow can be used (in your workflows)?
  - who can run workflows (on their forks)
  - which permissions the workflows have (read/write)
  - who gives approvals & prevents unapproved modifications to your repository (& workflow file).
  - which rules to enforce when collaborating (repositories/branches/tags rules)

- _While_ workflows run - **Security by Design**: Apply security best practices:

  - Not expose secrets.
  - Use token to limit access of users/processes.
  - Prevent common attacks: script injection, dependencies confusion...

- _After_ workflows run - **Security by Monitoring**:

  - Scanning for exposed secret, your code, supply chain.
  - Automatically handling pull requests

## Security by Configuration (Before workflows run)

### Actions Permissions

Which actions & reusable workflow can be used (in your workflows)?

- Disable the whole Actions feature of your repository
- Allow actions, reusable workflows from
  - Owner of the repository
  - Owner of the repository, and from
    - GitHub itself[^github-actions]
    - Marketplace _verified creators_[^verified-creators]
    - Your specified criteria
  - **Anyone** - _Default_

### Managing Execution of Workflows on Pull Requests (from a fork)

Anyone can forks a public GitHub repository[^fork-a-repository] and do whatever they want with their forked repository:

- Making changes to the code
- Making changes to the workflow files

You don't want to allow everyone to execute the workflows you defined, you need to know who are gonna run workflows (in your runners)

- They are new to GitHub (and don't know how workflow works): `Require approval for first-time contributors who are new to GitHub`
- They are familiar with GitHub (and know how workflow works): **`Require approval for first-time contributors`** (Default)

- They are some strangers (outside collaborators) and you want to review all cases: `Require approval for all outside collaborators`

### Workflow Permissions

You can configure the permission set grated to `GITHUB_TOKEN` when running workflows in this repository:

- Read & write (for all scopes)
- **Only read contents & packages** - Default

> [!NOTE]
> More granular permissions can be granted in the workflow files with `permissions` key.
>
> For more information, see [Managing permissions in your workflow | Chap 6](chap-06.md#managing-permissions-for-your-workflow)

> [!IMPORTANT]
> You can also `Allow Github Actions to create & approve pull requests` on your behalf.

### The `CODEOWNERS` File

The `CODEOWNERS` file is used to define individuals or teams that are responsible for code in a repository (_code owners_).

Code owners are automatically requested for review when someone opens a pull request that modifies code that they own

For more information, see [About code owners | GitHub Docs][about-code-owners]

> [!TIP]
> Specify an entry for `.github/workflows` directory in `CODEOWNERS` file so any proposed changes to workflow files can be reviewed from a designated reviewer.

### Rules for Collaboration

#### Protecting Branches with `Branches Protection Rules`

`Branch` is considered "killer feature" of `Git`[^Git-Branching-Branches-in-a-Nutshell].

- Git `branch` is lightweight, with allow switching between branches nearly instantaneous.
- `Git` encourages workflows that branch and merge often (usually to a branch named `main`/`master` - called `default branch` in GitHub).

To collaborate effective, a _branching strategy_[^branching-strategy] needs to be agree on how & when to create & merge branches.

With `Branches Protection Rules`, you can enforce branching strategy for your repository.

Each `Branches Protection Rule` has settings for:

- `Branch name pattern`: which branches to be applied.

- `Protect matching branches`: the rules to enforce
  - 💱 Require a **pull request** before merging
  - ✅ Require **status checks** to pass before merging
    - 🆕 Require branches to be **up to date** before merging
  - 🫂 Require **conversation resolution** before merging
  - 🎟️ Require **signed commits**
  - 📏 Require **linear history**
  - 🚀 Require **deployments to succeed** before merging
  - 🔒 **Lock** branch
  - 🚑 Do **not allow bypassing** the above settings
- Rules **applied to everyone** including administrators
  - 💪 Allow **force pushes**
  - 🚮 Allow **deletions**

#### Protecting Tags with `Protected Tag Rules`

Usually, a tag is used to mark a release point[^gitlab-tags] in your source.

e.g. [2 pipelines that creating a release]

- **Tag first, release second**: When a `tag` is created, it triggers a pipeline that create a `release`
- **Release first, tag second**: When the `default branch` changed, it triggers a pipeline that create a `release`, then a `tag` is created

Those release tag (e.g. `rel/*`, `v*`) shouldn't be create/delete without any restriction.

`Protected Tag Rules` let you protect these tags, which are a part of your version history and should be immutable.

Only users with `admin`/`maintain` permissions or have a custom role with `edit repository rules` permission can create/delete a tag.

#### Protecting Repository with `Rulesets`

ruleset
: a named list of rules that applies to a repository
: `branch ruleset` / `tag ruleset`
: has the same set of rules `a branches protection rule`

A ruleset can be used as a replacement to `branches protection rule` or `protected tag rule`, with some [advantages][ruleset-advantages]:

- A ruleset has an _enforcement status_ that can set to `Enabled`/`Disabled`.

- A ruleset has more control over

  - Who it apply to: with `Bypass list`
  - Which target it apply to: with `Targets`

- Multiple rulesets can apply at the same time through `rule layering`[^rule-layering]

- Users with read access can view the active rulesets, so they know which rule they are violating.

> [!NOTE]
> If a contributor violates a protection rule, they need to rewrite their commit history, e.g. through a rebase

## Security by Design (While workflows run)

Workflows are code that executed in runners.

You should apply software best practice to secure access & prevent misuse when run.

### Secrets

`secret`
: (software) a privileged credential stored securely as an object in the system
: used to protect sensitive information,

e.g. `access token`: allow workflow to have permissions to do designated operations

In GitHub Actions, secrets can be at many levels (in order of precedence if there are secrets with same name):

- Deployment environment
- Repository
- Organization

### Securing Secrets

"Secrets should stay secret":

- 🤏 Limit the privileges of credentials secured by secrets.

- ⌛ Limit the lifetime of a secret rotating 💱 secrets regularly.

- 🕴️ Limit who can access secrets:

  - 🖊️ Limit who has `write` access to your repository

    Any user with write access can read all secrets (in your repository).

  - ✉️ Don't expose secrets to user with `read` access

    GitHub Actions redacts secrets when writing out logs, but it only works for:

    - Data that's registered as a secret.
    - Plain-text logs (not structured data such as YAML, JSON, XML...)

  - 🎫 Require reviewers for any attempt to access secrets.

### Tokens

token
: an cryptographically key, used to access resources
: used in-place of password

Advantages of token:

- Can be stored & referenced programmatically.
- Have scopes for permissions.
- Have a limited lifetime.
- Can be revoked.

In GitHub Actions, there are 2 types of tokens:

- Personal access token (PAT)
- GitHub Token

#### Personal access token (PAT)

There are 2 types of PAT:

- A classic PAT
- (New - Beta) A fine-grained, repository-scoped token

Create PAT:

- For a classic PAT:

  - Go to `Account Settings` / `Developer Settings` / `Personal access tokens` / `Tokens (classic)` / `Generate new token (classic)`
  - Specify:
    - Token name
    - Expiration
    - Scopes
  - `Generate token`
  - Copy the token and store it securely (You only have one chance to view the token)

- For a fine-grained, repository-scoped token
  - Go to `Account Settings` / `Developer Settings` / `Personal access tokens` / `Fine-grained token` / `Generate new token`
  - Specify:
    - Token name
    - Expiration
    - Repository access:
      - `Public Repositories (read-only)`
      - `All repositories`
      - `Only select repositories`
    - Permissions

Use PAT:

- Store the PAT as a secret.
- In workflow files, access it via `secrets` context

#### GitHub Token

As in [Managing Permissions for Your Workflow | Chap 6](chap-06.md#managing-permissions-for-your-workflow):

- when GitHub Actions are enabled in a repository, GitHub installs a GitHub App in the repository
- this app has an access token (referred to as the `GITHUB_TOKEN`) with permissions to your repository.

> [!TIP] > [GitHub Apps][github-apps] are tools that extend GitHub's functionality (by interacting directly with the GitHub APIs):
>
> - Do things on GitHub,
>
>   e.g. open issues, comment on pull requests.
>
> - Do things outside of GitHub (based on events happen on GitHub)
>
>   e.g. Post on Slack when an issue is opened on GitHub

Prior to executing a job, GitHub gets this token and uses it to execute the job.

> [!NOTE]
> Token Lifetime: The token for a job _expires_:
>
> - when the job is **completed**.
> - or after **24 hours**, which ever comes first.

#### Using GitHub Token

As in [Managing Permissions for Your Workflow | Chap 6](chap-06.md#managing-permissions-for-your-workflow):

- The `GITHUB_TOKEN` can be accessed via `secrets` context or `github` context.

- The permissions of `GITHUB_TOKEN` takes effect in order: (the latter override the former):

  - Permissions as set by default for enterprise, organization, or repository
  - Configuration globally in a workflow
  - Configuration in a job
  - Adjusted to read-only if:
    — Workflow triggered by a pull request from a forked repository

> [!TIP]
> The `GITHUB_TOKEN` permissions for a job can be seen in the job log.
>
> - It's in `##[group]GITHUB_TOKEN Permissions`
> - In the web interface, it's in log of `Setup up job`

> [!WARNING]
> With the `GITHUB_TOKEN`, a workflow can run recursively.
>
> To avoid this, only `workflow_dispatch` & `repository_dispatch` events can create new workflows.

### Dealing with Untrusted Input

When an event triggers your workflow:

- That trigger brings with it a set of information related to the event.
- These information about the event (along with the workflow run) is available via the `github` context[^github-context]:

|            | Property name         | Description                                                                                                                 |
| ---------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Event      | `github.event_name`   | The name of the event that triggered the workflow run.                                                                      |
|            | `github.event`        | The webhook payload of the event that triggered the workflow run (different for each event[^events-that-trigger-workflows]) |
|            | `github.event_path`   | The path to the file on the runner that contains the full event webhook payload.                                            |
| ...        | ...                   |                                                                                                                             |
| Commit     | `github.ref`          | The fully-formed ref of the branch or tag that triggered the workflow run,                                                  |
|            | `github.sha`          | The commit SHA that triggered the workflow.                                                                                 |
| ...        | ...                   |                                                                                                                             |
| Workflow   | `github.workflow_sha` | The commit SHA for the workflow file.                                                                                       |
| ...        | ...                   |                                                                                                                             |
| Actor      | ...                   |                                                                                                                             |
| Repository | ...                   |                                                                                                                             |
| ...        | ...                   |                                                                                                                             |

These data can be classified in 2 lists:

- _generally trusted_: permanent data is less likely to be exploited
- _generally untrusted_: data tied to the current event and/or has user-configuration information, e.g.
  - Commit messages.
  - Pull requests titles/bodies, references/labels.
  - Issue titles/bodies.
  - Author emails/names.
  - Review bodies/comments.

> [!TIP]
> These data can include ``!#$%&'*+-/=?^_`{|}~``, which could be use in a shell program with commands like `echo` to gather data.

#### Script injection

script injection
: a security vulnerability whereby an attacker can inject malicious code into user input, such as a text field on a website

> [!CAUTION]
> If these _generally untrusted_ data get passed to your workflows to the shell on the runner, the code could be interpreted and executed there.

e.g.

- Script injection via commit message

  - Your workflow

    ```yml
    on:
      push:
        branches: ["main"]
    jobs:
      process:
        runs-on: ubuntu-latest
        steps:
          - run: echo ${{ github.event.head_commit.message }}
    ```

  - The commit message

    ```bash
    `echo my content > demo.txt; ls -la; printenv;`
    ```

  - The shell script in the commit message will be executed in the runner.

- Script injection via secret

  - Your workflow

    ```yml
    on:
      push:
        branches: ["main"]
    jobs:
      process:
        runs-on: ubuntu-latest
        steps:
          - run: echo ${{ secrets.DEMO_SECRET }
    ```

  - The secret

    ```text
    printf "Starting custom code\n"
    mkdir foo
    echo data.txt > foo/foo1.txt
    echo more > foo/foo2.txt
    ls -la foo
    rm -rf foo
    ls -la
    printf "Executed!\n"
    ```

  - The shell script in the secret will be executed in the runner.

#### Prevent script injection vulnerabilities

> [!IMPORTANT]
> Why script injection works in your workflows?
>
> Because of the way strings are interpreted on the runner:
>
> - The run command executes within a temporary shell script on the runner.
> - Before this temporary shell script is run, the expressions inside `${{ }}` are
>   evaluated.
> - Then, substitution happens with the resulting **values** from the evaluation.

To prevent script injection:

- Avoid using inline scripts, instead call an action (if available) to do the same operation.

  > [!TIP]
  > How it mitigates script injection?
  > Context values are passed to the actions as an argument instead of being
  > directly evaluated.

- If you `run` a shell command, capture any values passes to the run command in an intermediate variable.

  e.g.

  - For the commit message

    ```yml
    steps:
      - env:
          DATA_VALUE: ${{ github.event.head_commit.message }}
        run: echo $DATA_VALUE
    ```

  - For the secret

    ```yml
    steps:
      - env:
          DATA_VALUE: ${{ secrets.DEMO_SECRET }}
        run: echo $DATA_VALUE
    ```

### Securing Your Dependencies

You have the responsibility to secure your dependencies (3rd-party code).

To secure your dependencies:

- Apply principle of least privilege.
- Use trusted code (someone's reviewed it):
  - Only use verified actions.
  - (Review the code yourself)
- Use the code that you've trusted[^referencing-actions].
  - Instead of referencing a dependency with
    - branch names, e.g. `main`
    - tag/release, e.g. `v1`, `v2`
  - Referencing the full _commit hash_ (the safest way, protect you against _dependency substitution attacks_)

> [!TIP]
> Is it a _changeset_ hash or a _commit_ hash?
>
> - Git doesn’t store data as a series of changesets or differences, but instead as a series of **snapshots**[^Git-Branching-Branches-in-a-Nutshell].
> - You reference an action with its commit hash[^jobs-job_id-steps-uses] (not the changeset hash).
>
> For more information, see:
>
> - [Git References | Pro Git][Git-Internals-Git-References]
> - [Git Objects | Pro Git][Git-Internals-Git-Objects]

> [!TIP]
> What is dependency substitution attack?
>
> dependency substitution
> : substituting a module dependency with another one
>
> dependency substitution attack[^dependency-substitution-attacks]
> : aka dependency confusion attack
> : a type of _supply chain attack_[^what-is-a-supply-chain-attack]
> : attack that causes a "confusion" or "substitution" between the desired package and the malicious package, leading to the code being compromised

## Security by Monitoring (After workflows run)

### Scanning

### Processing Pull Requests Securely

### Vulnerabilities with Workflows in Pull Requests

### Vulnerabilities with Source Code in Pull Requests

### Adding a Pull Request Validation Script

### Safely Handling Pull Requests

## Conclusion

[about-code-owners]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
[2 pipelines that creating a release]: https://docs.gitlab.com/ee/user/project/releases/release_cicd_examples.html#skip-multiple-pipelines-when-creating-a-release
[ruleset-advantages]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets#about-rulesets-protected-branches-and-protected-tags
[github-apps]: https://docs.github.com/en/apps/overview
[Git-Internals-Git-References]: https://git-scm.com/book/en/v2/Git-Internals-Git-References
[Git-Internals-Git-Objects]: https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

[^github-actions]: <http://github.com/actions>
[^verified-creators]: <https://docs.github.com/en/actions/learn-github-actions/finding-and-customizing-actions#browsing-marketplace-actions-in-the-workflow-editor>
[^fork-a-repository]: <https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo#find-another-repository-to-fork>
[^Git-Branching-Branches-in-a-Nutshell]: <https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>
[^branching-strategy]: <https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/branching-strategy>
[^gitlab-tags]: <https://docs.gitlab.com/ee/user/project/repository/tags/>
[^rule-layering]: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets#about-rule-layering>
[^events-that-trigger-workflows]: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows>
[^github-context]: <https://docs.github.com/en/actions/learn-github-actions/contexts#github-context>
[^dependency-substitution-attacks]: <https://docs.aws.amazon.com/codeartifact/latest/ug/dependency-substitution-attacks.html>
[^what-is-a-supply-chain-attack]: <https://www.cloudflare.com/learning/security/what-is-a-supply-chain-attack/>
[^referencing-actions]: <https://securitylab.github.com/research/github-actions-building-blocks#referencing-actions>
[^jobs-job_id-steps-uses]: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsuses>
