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

## Security by Configuration

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
> Specify an entry for `.github/worflows` directory in `CODEOWNERS` file so any proposed changes to workflow files can be reviewed from a designated reviewer.

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

- Multile rulesets can apply at the same time through `rule layering`[^rule-layering]

- Users with read access can view the active rulesets, so they know which rule they are violating.

> [!NOTE]
> If a contributor violates a protection rule, they need to rewrite their commit history, e.g. through a rebase

## Security by Design

### Secrets

### Securing Secrets

### Tokens

### Dealing with Untrusted Input

### Securing Your Dependencies

## Security by Monitoring

### Scanning

### Processing Pull Requests Securely

### Vulnerabilities with Workflows in Pull Requests

### Vulnerabilities with Source Code in Pull Requests

### Adding a Pull Request Validation Script

### Safely Handling Pull Requests

## Conclusion

[^github-actions]: <http://github.com/actions>

[^verified-creators]: <https://docs.github.com/en/actions/learn-github-actions/finding-and-customizing-actions#browsing-marketplace-actions-in-the-workflow-editor>

[^fork-a-repository]: <https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo#find-another-repository-to-fork>

[about-code-owners]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
[^Git-Branching-Branches-in-a-Nutshell]: <https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>

[^branching-strategy]: <https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/branching-strategy>

[^gitlab-tags]: <https://docs.gitlab.com/ee/user/project/repository/tags/>

[2 pipelines that creating a release]: https://docs.gitlab.com/ee/user/project/releases/release_cicd_examples.html#skip-multiple-pipelines-when-creating-a-release
[ruleset-advantages]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets#about-rulesets-protected-branches-and-protected-tags
[^rule-layering]: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets#about-rule-layering>
