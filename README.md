<p align="center">
  <a href="https://sentry.io/?utm_source=github&utm_medium=logo" target="_blank">
    <picture>
      <source srcset="https://sentry-brand.storage.googleapis.com/sentry-logo-white.png" media="(prefers-color-scheme: dark)" />
      <source srcset="https://sentry-brand.storage.googleapis.com/sentry-logo-black.png" media="(prefers-color-scheme: light), (prefers-color-scheme: no-preference)" />
      <img src="https://sentry-brand.storage.googleapis.com/sentry-logo-black.png" alt="Sentry" width="280">
    </picture>
  </a>
</p>

# Sentry Release GitHub Action

Automatically create a Sentry release in a workflow.

A release is a version of your code that can be deployed to an environment. When you give Sentry information about your releases, you unlock a number of new features:

- Determine the issues and regressions introduced in a new release
- Predict which commit caused an issue and who is likely responsible
- Resolve issues by including the issue number in your commit message
- Receive email notifications when your code gets deployed

Additionally, releases are used for applying [source maps](https://docs.sentry.io/platforms/javascript/sourcemaps/) to minified JavaScript to view original, untransformed source code. You can learn more about releases in the [releases documentation](https://docs.sentry.io/workflow/releases).

## What's new

Version 3 is out with improved support for source maps. We highly recommend upgrading to `getsentry/action-release@v3`.

Please refer to the [release page](https://github.com/getsentry/action-release/releases) for the latest release notes.

## Prerequisites

See how to [set up the prerequisites](https://docs.sentry.io/product/releases/setup/release-automation/github-actions/#prerequisites) for the Action to securely communicate with Sentry.

## Usage

Adding the following to your workflow will create a new Sentry release and tell Sentry that you are deploying to the `production` environment.

> [!IMPORTANT]
> Make sure you are using at least v3 of [actions/checkout](https://github.com/actions/checkout) with `fetch-depth: 0`, issues commonly occur with older versions.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- name: Create Sentry release
  uses: getsentry/action-release@v3
  env:
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    SENTRY_ORG: ${{ secrets.SENTRY_ORG }}
    SENTRY_PROJECT: ${{ secrets.SENTRY_PROJECT }}
    # SENTRY_URL: https://sentry.io/
  with:
    environment: production
```

### Inputs

#### Environment Variables

|name|description|default|
|---|---|---|
|`SENTRY_AUTH_TOKEN`|**[Required]** Authentication token for Sentry. See [installation](#create-a-sentry-internal-integration).|-|
|`SENTRY_ORG`|**[Required]** The slug of the organization name in Sentry.|-|
|`SENTRY_PROJECT`|The slug of the project name in Sentry. One of `SENTRY_PROJECT` or `projects` is required.|-|
|`SENTRY_URL`|The URL used to connect to Sentry. (Only required for [Self-Hosted Sentry](https://develop.sentry.dev/self-hosted/))|`https://sentry.io/`|

#### Parameters

| name                      | description                                                                                                                                                                                                                 |default|
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---|
| `environment`             | Set the environment for this release. E.g. "production" or "staging". Omit to skip adding deploy to release.                                                                                                                |-|
| `sourcemaps`              | Space-separated list of paths to JavaScript sourcemaps. Omit to skip uploading sourcemaps.                                                                                                                                  |-|
| `inject`                  | Injects Debug IDs into source files and source maps to ensure proper un-minifcation of your stacktraces. Does nothing if `sourcemaps` was not set.                                                                          |`true`|
| `finalize`                | When false, omit marking the release as finalized and released.                                                                                                                                                             |`true`|
| `ignore_missing`          | When the flag is set and the previous release commit was not found in the repository, will create a release with the default commits count instead of failing the command.                                                  |`false`|
| `ignore_empty`            | When the flag is set, command will not fail and just exit silently if no new commits for a given release have been found.                                                                                                   |`false`|
| `dist`                    | Unique identifier for the distribution, used to further segment your release. Usually your build number.                                                                                                                    |-|
| `started_at`              | Unix timestamp of the release start date. Omit for current time.                                                                                                                                                            |-|
| `release`                 | Identifier that uniquely identifies the releases. Should match the `release` property in your Sentry SDK init call if one was set._Note: the `refs/tags/` prefix is automatically stripped when `version` is `github.ref`._ |<code>${{&nbsp;github.sha&nbsp;}}</code>|
| `version`                 | Deprecated: Use `release` instead.                                                                                                                                                                                          |<code>${{&nbsp;github.sha&nbsp;}}</code>|
| `release_prefix`          | Value prepended to auto-generated version. For example "v".                                                                                                                                                                 |-|
| `version_prefix`          | Deprecated: Use `release_prefix` instead.                                                                                                                                                                                 |-|
| `set_commits`             | Specify whether to set commits for the release. Either "auto" or "skip".                                                                                                                                                    |"auto"|
| `projects`                | Space-separated list of paths of projects. When omitted, falls back to the environment variable `SENTRY_PROJECT` to determine the project.                                                                                  |-|
| `url_prefix`              | Adds a prefix to source map urls after stripping them.                                                                                                                                                                      |-|
| `strip_common_prefix`     | Will remove a common prefix from uploaded filenames. Useful for removing a path that is build-machine-specific.                                                                                                             |`false`|
| `working_directory`       | Directory to collect sentry release information from. Useful when collecting information from a non-standard checkout directory.                                                                                            |-|
| `disable_telemetry`       | The action sends telemetry data and crash reports to Sentry. This helps us improve the action. You can turn this off by setting this flag.                                                                                  |`false`|
| `disable_safe_directory`  | The action needs access to the repo it runs in. For that we need to configure git to mark the repo as a safe directory. You can turn this off by setting this flag.                                                         |`false`|


### Examples

- Create a new Sentry release for the `production` environment, inject Debug IDs into JavaScript source files and source maps and upload them from the `./dist` directory.

    ```yaml
    - uses: getsentry/action-release@v3
      with:
        environment: 'production'
        sourcemaps: './dist'
    ```

- Create a new Sentry release for the `production` environment of your project at version `v1.0.1`.

    ```yaml
    - uses: getsentry/action-release@v3
      with:
        environment: 'production'
        release: 'v1.0.1'
    ```

- Create a new Sentry release for [Self-Hosted Sentry](https://develop.sentry.dev/self-hosted/)

    ```yaml
    - uses: getsentry/action-release@v3
      env:
        SENTRY_URL: https://sentry.example.com/
    ```


## Contributing

See the [Contributing Guide](./CONTRIBUTING.md).

## License

See the [License File](./LICENSE)

## Troubleshooting

Suggestions and issues can be posted on the repository's
[issues page](https://github.com/getsentry/action-release/issues).

- Forgetting to include the required environment variables
  (`SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, and `SENTRY_PROJECT`), yields an error that looks like:

    ```text
    Environment variable SENTRY_ORG is missing an organization slug
    ```

- Building and running this action locally on an unsupported environment yields an error that looks like:

    ```text
    Syntax error: end of file unexpected (expecting ")")
    ```

- When adding the action, make sure to first check out your repo with `actions/checkout@v4`.
Otherwise, it could fail at the `propose-version` step with the message:

    ```text
    error: Could not automatically determine release name
    ```

- In `actions/checkout@v4` the default fetch depth is 1. If you're getting the error message:

    ```text
    error: Could not find the SHA of the previous release in the git history. Increase your git clone depth.
    ```

    you can fetch all history for all branches and tags by setting the `fetch-depth` to zero like so:

    ```yaml
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    ```

- Not finding the repository

  ```text
  Error: Command failed: /action-release/node_modules/@sentry/cli-linux-x64/bin/sentry-cli --header sentry-trace:ab7a03b5cd8ce324103b3ced985de08b-2bf7fecfb8a1e812-1 --header baggage:sentry-environment=production-sentry-github-action,sentry-release=1.10.5,sentry-public_key=<truncated>,sentry-trace_id=ab7a03b5cd8ce324103b3ced985de08b,sentry-sample_rate=1,sentry-transaction=sentry-github-action-execution,sentry-sampled=true releases set-commits action-test --auto
  error: could not find repository at '.'; class=Repository (6); code=NotFound (-3)
  ```

  Ensure you use `actions/checkout` before running the action

  ```yaml
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - uses: getsentry/action-release@v3
      with:
        environment: 'production'
        release: 'v1.0.1'
    ```
