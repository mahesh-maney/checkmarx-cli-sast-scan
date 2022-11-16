# Checkmarx CLI SAST Scan

This action will run a Checkmarx SAST Scan via the Checkmarx CLI and generate a PDF report. The options for this action have been tailored to `im-open`'s needs so not all [Checkmarx CLI options] have been enabled.

## Index

- [Usage](#usage)
  - [Excluding Files and Folders](#excluding-files-and-folders)
  - [Build Failures](#build-failures)
  - [Checkmarx Exit Codes](#checkmarx-exit-codes)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [Example](#example)
  - [Using the default values & `continue-on-error` pattern](#using-the-default-values----continue-on-error--pattern)
  - [Override the default values and `if: always()` pattern](#override-the-default-values-and--if--always----pattern)
- [Contributing](#contributing)
  - [Incrementing the Version](#incrementing-the-version)
- [Code of Conduct](#code-of-conduct)
- [License](#license)

## Usage

### Excluding Files and Folders

When the Checkmarx CLI runs, it zips up the contents of the working directory and sends that to Checkmarx. To limit the number of files that are zipped up, it's best to run the scan on a newly checked out repository that has not been built.

These file and folder patterns identify what Checkmarx will ignore when it scans the code:

```sh
# Folders
!**/_cvs/**/*,                  !**/.bzr/**/*,                  !**/.git/**/*,
!**/.github/**/*,               !**/.hg/**/*,                   !**/.idea/**/*,
!**/.svn/**/*,                  !**/.vscode/**/*,               !**/*Tests*/**/*,
!**/backup/**/*,                !**/build/**/*,                 !**/bin/**/*,
!**/coverage-results/**/*,      !**/node_modules/**/*,          !**/obj/**/*,

# Specific Files
!**/devserver.js,               !**/.gitignore,                 !**/webpack.config.js,

# File Extensions
!**/*.3g2,      !**/*.3gp,      !**/*.7z,       !**/*.aac,      !**/*.ahtm,     !**/*.ahtml,
!**/*.aif,      !**/*.asf,      !**/*.asx,      !**/*.avi,      !**/*.bak,      !**/*.bmp,
!**/*.class,    !**/*.dll,      !**/*.DS_Store, !**/*.ear,      !**/*.exe,      !**/*.fhtml,
!**/*.flv,      !**/*.gif,      !**/*.gz,       !**/*.hdm,      !**/*.hdml,     !**/*.hsql,
!**/*.ht,       !**/*.hta,      !**/*.htc,      !**/*.htd,      !**/*.htmls,    !**/*.iff,
!**/*.ihtml,    !**/*.iml,      !**/*.ipr,      !**/*.iws,      !**/*.jar,      !**/*.jpg,
!**/*.m3u,      !**/*.mht,      !**/*.mhtm,     !**/*.mhtml,    !**/*.mid,      !**/*.mov,
!**/*.mp3,      !**/*.mp4,      !**/*.mpa,      !**/*.mpg,      !**/*.pdb,      !**/*.png,
!**/*.psd,      !**/*.ra,       !**/*.rar,      !**/*.rm,       !**/*.ssi,      !**/*.stm,
!**/*.stml,     !**/*.swf,      !**/*.swf,      !**/*.tar,      !**/*.tar.gz,   !**/*.tif,
!**/*.tmp,      !**/*.ttml,     !**/*.txn,      !**/*.vob,      !**/*.war,      !**/*.wav,
!**/*.wma,      !**/*.wmv,      !**/*.xhtm,     !**/*.xhtml,    !**/*.zip
```

This list cannot be changed but additional patterns can be appended with the `excluded-paths` argument. Keep in mind if files or folders matching these patterns are present, they'll still be included in the zip file the CLI sends to Checkmarx, they just won't be included in the scan.

### Build Failures

Normally the Checkmarx CLI will fail if the reported number of vulnerabilities exceed the provided thresholds. The `break-build: false` argument can be passed though which will prevent the build from breaking.

If `break-build: true` the following items may be helpful:

- `continue-on-error`
  - Allows all subsequent steps to run
  - Applied to the Checkmarx step
  - Visually it causes the step/job to look like it has succeeded so you have to look through the logs to determine the actual outcome
- `if: always()`
  - Allows subsequent step(s) to run
  - Applied to each subsequent step that should be run
  - The Checkmarx step and job are still shown as failures

### Checkmarx Exit Codes

This action is only configured to run SAST scans and from the [Checkmarx Documentation] the CLI may return these exit codes:
| Code | Description |
| ---- | ------------------------------------ |
| 0 | Completed successfully |
| 1 | Failed to start scan (general error) |
| 2 | Invalid license for SDLC |
| 4 | Login failed |
| 10 | Failed on threshold SAST HIGH |
| 11 | Failed on threshold SAST Medium |
| 12 | Failed on threshold SAST LOW |
| 18 | Policy is violated |
| 130 | Canceled by user (Ctrl-C) |

## Inputs

| Parameter               | Is Required | Description                                                                                                                   |
| ----------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `checkmarx-server-url`  | true        | The Checkmarx Server URL.                                                                                                     |
| `checkmarx-username`    | true        | The Checkmarx username.                                                                                                       |
| `checkmarx-password`    | true        | The Checkmarx password.                                                                                                       |
| `team`                  | true        | The Checkmarx Team that owns the project.                                                                                     |
| `project`               | true        | The project the scan is for.                                                                                                  |
| `report-name`           | false       | Name of the pdf report to be generated. Defaults to CheckmarxSASTResults.pdf.                                                 |
| `sast-high`             | false       | The maximum number of High Vulnerabilities.                                                                                   |
| `sast-medium`           | false       | The maximum number of Medium Vulnerabilities.                                                                                 |
| `sast-low`              | false       | The maximum number of Low Vulnerabilities.                                                                                    |
| `break-build`           | false       | Flag indicating whether the build should break if any vulnerability threshold is exceeded. Defaults to true.                  |
| `comment`               | false       | Comment to add to the scan (shown in Checkmarx Portal). Defaults to _<WorkflowName>_ run #_<RunNumber>_ on branch _<branch>_. |
| `exclude-paths`         | false       | Comma separated list of file or folder name patterns that will be appended to the default list of patterns to exclude.        |
| `checkmarx-cli-version` | false       | The Checkmarx CLI version. Defaults to 1.1.9.                                                                                 |

## Outputs

| Output                 | Description                                                                           |
| ---------------------- | ------------------------------------------------------------------------------------- |
| report-path            | The path to the generated report. Can be used in a later step like `upload-artifact`. |
| high-vulnerabilities   | The number of High severity vulnerabilities reported                                  |
| medium-vulnerabilities | The number of Medium severity vulnerabilities reported                                |
| low-vulnerabilities    | The number of Low severity vulnerabilities reported                                   |
| info-vulnerabilities   | The number of Info severity vulnerabilities reported                                  |
| results-location       | The url to the results location                                                       |

## Example

### Using the default values & `continue-on-error` pattern

```yml
jobs:
  run-the-scan:
    runs-on: [self-hosted, ubuntu-20.04]
    steps:
      - uses: actions/checkout@v3

      - name: Run Checkmarx with the defaults
        continue-on-error: true
        id: checkmarx
        uses: im-open/checkmarx-cli-sast-scan@v2.1.0
        with:
          checkmarx-server-url: ${{ secrets.CHECKMARX_URL }}
          checkmarx-username: ${{ secrets.CHECKMARX_USERNAME }}
          checkmarx-password: ${{ secrets.CHECKMARX_PASSWORD }}
          team: 'CxServer\SP\Company\Users\MyCheckmarxTeam'
          project: 'HelloWorldApp'

      - name: Upload PDF Report
        uses: actions/upload-artifact@v3
        with:
          name: Checkmarx Report
          path: ${{ steps.scan.outputs.report-path }}

      - name: Fail the build if Checkmarx failed
        if: steps.checkmarx.outcome == 'failure'
        run: exit 1
```

### Override the default values and `if: always()` pattern

```yml
jobs:
  run-the-scan:
    runs-on: [self-hosted, ubuntu-20.04]
    steps:
      - uses: actions/checkout@v3

      - name: Run Checkmarx with the defaults
        id: scan
        uses: im-open/checkmarx-cli-sast-scan@v2.1.0
        with:
          checkmarx-server-url: ${{ secrets.CHECKMARX_URL }}
          checkmarx-username: ${{ secrets.CHECKMARX_USERNAME }}
          checkmarx-password: ${{ secrets.CHECKMARX_PASSWORD }}
          team: 'CxServer\SP\Company\Users\MyCheckmarxTeam'
          project: 'HelloWorldApp'
          report-name: 'Scan-Results-${{ github.run_number }}.pdf'
          sast-high: 1
          sast-medium: 2
          sast-low: 3
          break-build: true
          comment: 'Run from Github Actions'
          exclude-paths: '!**/modernizr-custom.js,!**/_ViewImports.cshtml,!**/.terraform/**/*'
          checkmarx-cli-version: '1.1.9'

      - name: Upload PDF Report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: Checkmarx Report
          path: ${{ steps.scan.outputs.report-path }}
```

## Contributing

When creating new PRs please ensure:

1. For major or minor changes, at least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version](#incrementing-the-version).
1. The action code does not contain sensitive information.

When a pull request is created and there are changes to code-specific files and folders, the `auto-update-readme` workflow will run.  The workflow will update the action-examples in the README.md if they have not been updated manually by the PR author. The following files and folders contain action code and will trigger the automatic updates:

- `action.yml`

There may be some instances where the bot does not have permission to push changes back to the branch though so this step should be done manually for those branches. See [Incrementing the Version](#incrementing-the-version) for more details.

### Incrementing the Version

The `auto-update-readme` and PR merge workflows will use the strategies below to determine what the next version will be.  If the `auto-update-readme` workflow was not able to automatically update the README.md action-examples with the next version, the README.md should be updated manually as part of the PR using that calculated version.

This action uses [git-version-lite] to examine commit messages to determine whether to perform a major, minor or patch increment on merge. The following table provides the fragment that should be included in a commit message to active different increment strategies.
| Increment Type | Commit Message Fragment |
| -------------- | ------------------------------------------- |
| major | +semver:breaking |
| major | +semver:major |
| minor | +semver:feature |
| minor | +semver:minor |
| patch | _default increment type, no comment needed_ |

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).

[git-version-lite]: https://github.com/im-open/git-version-lite
[checkmarx cli options]: https://checkmarx.atlassian.net/wiki/spaces/SD/pages/3053847822/Running+Scans+from+the+CLI
[checkmarx documentation]: https://checkmarx.atlassian.net/wiki/spaces/SD/pages/3053847822/Running+Scans+from+the+CLI
