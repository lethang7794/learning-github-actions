# Chapter 1: The basics

## Whats GitHub Actions?

GitHub Actions
: an end-to-end _GitHub-centric_ SDLC process.
: provides an **automation platform** and **framework** that has been missing from GitHub previously and has had to be
added on with other solutions such as Jenkins or Travis CI.

### Automation Platform

GitHub Actions is a way to _create & execute_ automated **workflows** tied to **GitHub events**.

e.g.

- You make a change via a pull request, and GitHub kicks off a continuous delivery pipeline.
  - Prior to Github Actions:
    - you will need an external tool/process (e.g. Jenkins, Travis CI) to
      - respond to the notification from Github that a pull request happened,
      - then process it.
    - the automation is implemented via that external tool.
  - With Github Actions:
    - you don't need an external tool You define the what, when, and how for automated responses to Github events, e.g.
      pull requests
    - Conveniently, the automation definitions (workflows) can be stored alongside your code in the GitHub repository.

### Framework

GitHub Actions provides a core set of **components**, that can be put together to execute simple or complex automation
in an understandable & predictable way.

- In response to an occurrence of a matching _event_, a _workflow_ definition stored within the repository is triggered,
  which in turn fires off _jobs_ on designated systems called _runners_.

- The jobs are made up of sequences of _steps_ that either invoke a predefined _action_ or run a command on the runner’s
  OS shell.

Previously, similar capabilities were available via mechanisms such as API calls. But to achieve the same desired goal,
a lot of effect needs to be invest to:

- Use the right API
- Integrate with external tools, e.g. Jenkins
- Write custom scripts, program.

Now, Github Actions provides a _seamless_, _flexible_ experience to do the same thing.

- Seamless: Github Actions is natively integrated with Github.
- Flexible:
  - Actions Marketplace has a lot of shared actions, that can be reused in your workflow.
  - If these shared actions are not enough, you can create your own _custom actions_ for more extensive logic.

> [!TIP]
> Action vs action?
>
> Actions
> : the platform/framework
>
> actions
> : the individual units of functionality

## What Are the Use Cases for GitHub Actions?

> [!IMPORTANT]
> Github _workflow_ vs CI/CD _pipeline_?
>
> GitHub Actions does not use the term _pipeline_ in its processes, the overall _workflow_ approach it uses is a similar concept.
>
> Workflows chain together smaller units of work called _jobs_. Jobs are what you often might see in other applications as _stages_, meaning parts of a larger process that perform a distinct and separate function.
>
> You can think of the overall GitHub Actions flow as being a _pipeline_, meaning some change or event causes a series of automated actions to happen automatically in response.

The main use case of Github Actions is to response to something happened in Github (with workflows) - CI/CD purpose.

> [!NOTE]
> Workflows can also be kick off:
>
> - At a particular schedule
> - Manually via Github Actions interface of the Github repository.
> - ...

But with workflows & actions, you can automate nearly any process.

For more information about what can be done with Github Actions, see Starter Workflows[^starer-workflows][^start-workflows-code] and Github Marketplaces's Actions[^github-marketplace-actions]

### Starter Workflows

Github provides ready-to-use workflows, organized by categories:

- **Automation**: examples for basic automation.
- **Continuous Integration (CI)**: building, testing, publishing.
  - **Security**: code-scanning.
- **Deployment (CD)**: creating deployable objects (e.g. containers), then deploying them to cloud platforms.
  - **Pages**: package/deploy static content to Github Pages

### Actions Marketplace

Instead of writing your own action, you can uses existing actions, available via Github Marketplaces's Actions, to save time & effort.

These shared actions are fully functionality units that you can select, and use in your own workflow, to:

- Setup environment for programming languages, e.g. Node, Go
- Install tools needed in your workflow, e.g. linters, `jq`, `yq`
- Interact with repository, e.g. checkout, generate repos' metrics
- Do some specific tasks, e.g. Build & push Docker image

## What Costs Are Involved?

### The Free Model

Github Actions is free if:

- The repository is _public_
- Or the runner is yours (instead of the one provided by Github)

### The Paid Model

#### What items cost money?

2 type of items cost money for Github Actions:

- _Storage_:
  Actions allow you to share data between jobs in a workflow and store data once that workflow has completed. These data are called _artifact_.
  Storing artifacts uses storage space on GitHub.
- _Minutes_:
  Actions require processing time on runner. You'll for the compute time to process workflows.

#### When does these items cost money?

For _private_ repositories & _Github-hosted_ runner:

- Each GitHub account receives a certain amount of free minutes and storage, depending on the account's plan.
- Any usage beyond the included amounts is controlled by **spending limits**:
  - The compute time is accumulated but reset to 0 each month.
  - The storage space won't be reset.

> [!TIP]
> The Github Free Plan account receives 500MB storage & 2000 minutes (per month)
>
> For more information, see:
>
> - [Github pricing plans](https://github.com/pricing)
> - [Billing for GitHub Actions](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)

#### How does the pricing calculated?

The storage usage is:

- calculated for each month based on _hourly_ usage during the month
- rounded to the nearest _megabytes_.

The minutes usage:

- is rounded to the nearest _minutes_.
- has a minute multiplier[^minute-multiplier] (applied for a runner with the same vCPUs, but different OS):

  - Linux: 1
  - Windows: 2
  - MacOS: 10

> [!TIP]
> Per-minute cost of the cheapest runner of each OS:
>
> - $0.008: Linux
> - $0.018: Windows
> - $0.080: MacOS
>
> For more information, see [Per-minute rates](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions#per-minute-rates)

## When Does Moving to GitHub Actions Make Sense?

### Investment in GitHub

GitHub Actions are tightly bound to the GitHub ecosystem, anyone working with Github Actions needs to be familiar with Github (as an interface & environment).

### Use of Public Actions

The responsibility for fit, purpose, and security when using a public action is yours.

### Creating Your Own Actions

You'll need to

- learn action structure & syntax
- re-implement your custom functionality as actions
- or have a workflow invoke your existing functionality.

### Artifact Management

By default, GitHub stores artifacts (& build logs) for 90 days, and this retention period[^artifact-retention-period] can be customized.

If you need to store the artifact more than the limitation (400 days for private repositories), you’ll need to establish another way to manage
artifacts and connect your workflows to it.

### Action Management

- Ensure any public actions used are kept up to date and use of them is reviewed as needed.
- If custom actions are created, and shared, some sort of code review and standards should be in place.

## Conclusion

- Github Actions provides a full framework to automate the content you manage in GitHub.
- With workflows & actions, you can implement CI/CD pipeline or almost any process without rely to an external tool.
- There are many ready-to-use actions & workflows that can save you a lot of time & effort.

[^github-marketplace-actions]: <https://github.com/marketplace?type=actions>
[^starer-workflows]: <https://docs.github.com/en/actions/learn-github-actions/using-starter-workflows>
[^start-workflows-code]: <https://github.com/actions/starter-workflows>
[^minute-multiplier]: <https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions#minute-multipliers>
[^artifact-retention-period]: <https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts>
