# Chapter 3. What's in action?

> [!NOTE]
> Workflow vs action
>
> action
> : ~ plug-in/modules in other applications
>
> workflow
> : ~ pipeline/scripts that use these plugins/modules

## The Structure of an action

An action can be

- very simple: a small **shell script**.
- complex: a large set of
  - **implementation code**,
  - **test cases**,
  - workflow files to handle its own CI/CD task,
  - a special file that stores metadata for the action.

Underlying each action is a code base in a GitHub repository.

e.g.

- The `checkout` action's underlying repository is at [github.com/actions/checkout](https://github.com/actions/checkout)

## Interfacing with actions

To be used as an action, a GitHub repository must contain an actions file (`.action.yml`).

`.action.yml` file contains metadata[^action-metadata] about the action itself, which has 4 main parts:

- Basic info: `name`, `description`, `author`
- `inputs`: Input parameters allow you to specify data that the action expects to use during runtime
  - If a parameter is required, your workflow needs to use a `with` statement, when calling the action, to provide a value to pass in for that parameter.
  - GitHub stores input parameters as environment variables.
- `outputs`: Declare data that an action sets [^outputs-docker-container-javascript-action] [^outputs-for-composite-actions].

  > [!NOTE]
  > Actions that run later in a workflow can use the output data set in previously run actions.

- `runs`: Specify the type of action[^type-of-action] & how the action is executed.

## Using actions

To use an action as a step of a job, you use the `uses` clause.

e.g.

```yml
steps:
  - uses: actions/checkout@v4
```

- The path part (e.g. `actions/checkout`) is the relative path to the Github repository after `github.com`
- The version number, part following the `@` symbol, e.g. `v4`, can be expressed in multiple ways:

  - Technically, any Git reference (e.g. a branch, a tag, a commit hash SHA) is valid.
  - However, semantic versioning (`major.minor.patch`) is recommended.

    - By convention, the action's consumer only use a shorter tag (the letter `v` and `major` version), e.g. `v4`

      > [!NOTE]
      > The action's author has the responsibility to point the shorter tag to the current latest version with that `major` version.
      >
      > e.g. If the current version has a tag of `4.5.6`, `v4` tag should also point to `4.5.6`.

    - Or you can be more explicit with the version you want:

      e.g.

      ```yml
      uses: actions/checkout@2.3.5
      ```

      ```yml
      uses: actions/checkout@v4-beta
      ```

## Public actions and the Marketplace

The Actions Marketplace is where creators can publish their actions to share with others in a standard location. It's available at [Actions Marketplace](https://github.com/marketplace?type=actions).

Github provides a lot of actions.

- The repositories of these Github-provided actions is at [github.com/actions](https://github.com/actions)
- If you see these action through Marketplace, you'll see the action's `README.md` in more user-friendly page.

## Conclusion

- `action` is a unit of code that perform a repeated tasks.
- The code for an action is stored in a standard Github repository, along with its metadata (in `action.yml` file) that specifies:
  - inputs
  - outputs
  - environment
- GitHub Actions Marketplace provides a place to share & discover public actions.

[^action-metadata]: <https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions>
[^type-of-action]: <https://docs.github.com/en/actions/creating-actions/about-custom-actions#types-of-actions>
[^outputs-docker-container-javascript-action]: <https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-docker-container-and-javascript-actions>
[^outputs-for-composite-actions]: <https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-composite-actions>
