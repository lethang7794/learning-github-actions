# Chap 14. Migrating to GitHub Actions

GitHub Actions can be implemented to replace almost any current automation framework.

You can use _GitHub Actions Importer_ to plan, test and **_automate_ migration** to GitHub Actions from the following platforms:

- Azure DevOps
- Bamboo
- Bitbucket Pipelines
- **CircleCI**
- GitLab
- **Jenkins**
- **Travis CI**

and can achieve an **80% conversion** rate for every workflow.

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

## Azure Pipelines

## CircleCI

## GitLab CI/CD

## Jenkins

## Travis CI

## GitHub Actions Importer

### Authentication

### Planning

### Build Steps and Related

### Manual Tasks

### File Manifest

### Forecasting

### Doing a Dry Run

### Creating Custom Transformers for the Importer

### Doing the Actual Migration

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
