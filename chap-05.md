# Chapter 5. Runners

A brief introduction about runners is available at [Chap 2 - GitHub Actions' Components - Runners][runners]

## GitHub-Hosted Runners

GitHub-Hosted Runners are the easiest way to execute jobs of a workflow.

The only configuration needed for a Github-Hosted Runners is the `runs-on` declaration (with the correct label[^github-hosted-runner]) in each job. e.g. `runs-on: ubuntu-latest`

- When executed, each job runs in a fresh instance of a runner image specified by `runs-on`.
- GitHub takes care of upgrade & maintenance for the VMs.

### What's in the Runner Images?

- For each images, the OS information and installed software is available in the `<OS>-<VERSION>-Readme.md` file, which are in the `images/<OS>` directory.

The Runner Image information for each job is available in the job log's `Set up job` section.

### Adding Additional Software on Runners

The source code to create the VM images for GitHub-hosted runners is at [github.com/actions/runner-images][actions-runner-images]
With the information about the package manager[^runner-image-package-manager] for each image, you can easily add any additional packages by creating a `job` that has a `step` invoking the package manager.

e.g.

- Linux

    ```yml
    jobs:
      update-env:
        runs-on: ubuntu-latest
        steps:
          - name: Install Package
            run: |
              sudo apt-get update
              sudo apt-get install <package-name>
    ```

- MacOS

    ```yml
    jobs:
      update-env:
        runs-on: macos-latest
        steps:
          - name: Install Package
            run: |
              brew update
              brew install <package-name>
    ```

## Self-Hosted Runners

A self-hosted runner is a system that you deploy and manage to execute jobs from GitHub Actions on GitHub.com

Self-hosted runners offer more control of hardware, operating system, and software tools than GitHub-hosted runners provide

You can add self-hosted runners at various levels in the management hierarchy:

- Repository-level runners are dedicated to a single repository.
- Organization-level runners can process jobs for multiple repositories in an organization.
- Enterprise-level runners can be assigned to multiple organizations in an enterprise account.

To connect to Github, a `self-hosted runner` run the `GitHub Actions runner application`[^actions-runner]

### Requirements for Self-Hosted Runners

Any machine can be a self-hosted runner if:

- It can run the `GitHub Actions runner application`.
- It can communicate with GitHub Actions.
- It has enough hardware resource to run the workflows.

### Limits for Self-Hosted Runners

See [Usage Limits][usage-limits]

### Security Considerations for Using Self-Hosted

Self-hosted runners should only be used with _private_ repositories.

> [!CAUTION]
> If someone fork your public repository, they can do whatever they want with the workflow files in the fork, which may execute the forked workflow in your self-hosted runner.

### Setting Up a Self-Hosted Runner

To set up a self-host runner for a Github repository,

- Go to the repository `Settings` tab / `Code & automation` / `Actions` / `Runners` / `New self-hosted runner`.
- Do as the instructions to `Download`, and `Configure` sections.
- Confirm that the runner is showed in the repository `Settings` - `Runners` page.

### Using a Self-Hosted Runner

To use the self-hosted runner in your workflow, you specify the `runs-on: self-hosted` clause in the workflow's job.

### Using Labels with Self-Hosted Runners

A self-hosted runner automatically gets a set of labels when it is added to GitHub Actions. These labels are:

- `self-hosted`
- `linux`, `macOS`, `windows`
- `x64`, `ARM`, `ARM64`

You can add your custom labels to a self-hosted runner by

- passing the additional labels to the initial config script, e.g.

  ```bash
  ./config.sh --url <REPO_URL> --token <REG_TOKEN> --labels ssd,gpu
  ```

- entering the additional labels to the config script prompt.

### Troubleshooting Self-Hosted Runners

See [Monitoring and troubleshooting self-hosted runners][troubleshoot-self-hosted-runners]

### Removing a Self-Hosted Runner

If a self-hosted runner doesn’t get connected to GitHub Actions for more than 30 days, it will automatically be removed.

To remove a self-hosted runner:

- Go to the repository `Settings` / `Runners` page.
- Click the name of the runner you want to remove.
- Click the `Remove` button and do as the instructions:
  - Remove & clean up the machine
  - Force runner removal

## Autoscaling Self-Hosted Runners

To auto-scale self-hosted runners, you can

- Use Kubernetes's `Actions Runner Controller (ARC)`[^actions-runner-controller] (the recommended way)
- Use cloud provider's scalability, e.g. `terraform-aws-github-runner`[terraform-aws-github-runner]

## Just-in-Time Runners

A self-host runner can be:

- `persistent`: stay across runs of multiple jobs (the default behavior)
- `ephemeral` (aka just-in-time): execute only one job before being automatically removed.

To autoscaling self-hosted runners, you should you ephemeral, just-in-time runner.

## Conclusion

- `Runners` provide the infrastructure to execute `workflows`.
- `Runners` can be hosted by
  - GitHub
  - you
- To choose a particular runner, you use `runs-on` clause for each job in the workflow.
- Self-hosted runners can be made _ephemeral_ to only execute one job, which is desirable for autoscaling solutions such as ARC.

[^github-hosted-runner]: <https://docs.github.com/en/actions/using-jobs/choosing-the-runner-for-a-job#choosing-github-hosted-runners>
[^runner-image-package-manager]: <https://github.com/actions/runner-images?tab=readme-ov-file#package-managers-usage>
[^actions-runner]: <https://github.com/actions/runner>
[^actions-runner-controller]: <https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/quickstart-for-actions-runner-controller>

[runners]: ./chap-02.md#runners
[usage-limits]: <https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#usage-limits>
[troubleshoot-self-hosted-runners]: <https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/monitoring-and-troubleshooting-self-hosted-runners>
[actions-runner-images]: <https://github.com/actions/runner-images>
[terraform-aws-github-runner]: <https://github.com/philips-labs/terraform-aws-github-runner>
