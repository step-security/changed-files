# StepSecurity Maintained changed-files Action

[StepSecurity Maintained Actions](https://docs.stepsecurity.io/actions/stepsecurity-actions) are usually exclusive to StepSecurity enterprise customers. However, in an effort to help the community with [the tj-actions/changed-files security incident](https://www.stepsecurity.io/blog/harden-runner-detection-tj-actions-changed-files-action-is-compromised), we are making this StepSecurity Maintained Action **freely available to everyone**.

## Overview

Effortlessly track all changed files and directories relative to a target branch, the current branch (preceding commit or the last remote commit), multiple branches, or custom commits returning **relative paths** from the project root using this GitHub action.

## Table of contents

*   [Features 🚀](#features-)
*   [Usage 💻](#usage-)
    *   [On `pull_request` 🔀](#on-pull_request-)
        *   [Using local .git directory 📁](#using-local-git-directory-)
        *   [Using Github's API :octocat:](#using-githubs-api-octocat)
    *   [On `push` ⬆️](#on-push-️)
    *   [Other supported events :electron:](#other-supported-events-electron)
*   [Inputs ⚙️](#inputs-️)
*   [Useful Acronyms 🧮](#useful-acronyms-)
*   [Outputs 📤](#outputs-)
*   [Versioning 🏷️](#versioning-️)
*   [Examples 📄](#examples-)
*   [Real-world usage 🌐](#real-world-usage-)
    *   [Open source projects 📦](#open-source-projects-)
    *   [Scalability Example 📈](#scalability-example-)
*   [Important Notice ⚠️](#important-notice-️)
*   [Migration guide 🔄](#migration-guide-)
*   [Credits 👏](#credits-)
*   [Report Bugs 🐛](#report-bugs-)
*   [Contributors ✨](#contributors-)

## Features 🚀

*   Fast execution, averaging 0-10 seconds.
*   Leverages either [Github's REST API](https://docs.github.com/en/rest/reference/repos#list-commits) or [Git's native diff command](https://git-scm.com/docs/git-diff) to determine changed files.
*   Facilitates easy debugging.
*   Scales to handle large/mono repositories.
*   Supports Git submodules.
*   Supports [merge queues](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue) for pull requests.
*   Generates escaped [JSON output for running matrix jobs](https://github.com/step-security/changed-files/blob/main/.github/workflows/matrix-example.yml) based on changed files.
*   Lists changed directories.
    *   Limits matching changed directories to a specified maximum depth.
    *   Optionally excludes the current directory.
*   Writes outputs to a designated `.txt` or `.json` file for further processing.
*   Restores deleted files to their previous location or a newly specified location.
*   Supports fetching a fixed number of commits, which improves performance.
*   Compatible with all platforms (Linux, MacOS, Windows).
*   Supports [GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners).
*   Supports [GitHub Enterprise Server](https://docs.github.com/en/enterprise-server@3.3/admin/github-actions/getting-started-with-github-actions-for-your-enterprise/getting-started-with-github-actions-for-github-enterprise-server).
*   Supports [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners).
*   Lists all files and directories that have changed:
    *   Between the current pull request branch and the last commit on the target branch.
    *   Between the last commit and the current pushed change.
    *   Between the last remote branch commit and the current HEAD.
*   Restricts change detection to a subset of files and directories:
    *   Provides boolean output indicating changes in specific files.
    *   Uses [Glob pattern](https://en.wikipedia.org/wiki/Glob_\(programming\)) matching.
        *   Supports Globstar.
        *   Supports brace expansion.
        *   Supports negation.
    *   Uses [YAML](https://yaml.org/) syntax for specifying patterns.
        *   Supports [YAML anchors & aliases](https://github.com/step-security/changed-files/blob/main/test/changed-files.yml#L8-L12).
        *   Supports [YAML multi-line strings](https://github.com/step-security/changed-files/blob/main/test/changed-files.yml#L13-L16).

And many more...

## Usage 💻

> \[!IMPORTANT]
>
> *   **Push Events**: When configuring [`actions/checkout`](https://github.com/actions/checkout#usage), make sure to set [`fetch-depth`](https://github.com/actions/checkout#usage) to either `0` or `2`, depending on your use case.
> *   **Mono Repositories**: To avoid pulling the entire branch history, you can utilize the default [`actions/checkout`](https://github.com/actions/checkout#usage)'s [`fetch-depth`](https://github.com/actions/checkout#usage) of `1` for [`pull_request`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request) events.
> *   **Quoting Multiline Inputs**: Avoid using single or double quotes for [multiline](https://github.com/step-security/changed-files/blob/main/test/changed-files.yml#L13-L16) inputs. The value is already a string separated by a newline character. Refer to the [Examples](#examples-) section for more information.
> *   **Credentials Persistence**: If [`fetch-depth`](https://github.com/actions/checkout#usage) is not set to 0, make sure to set [`persist-credentials`](https://github.com/actions/checkout#usage)  to `true` when configuring [`actions/checkout`](https://github.com/actions/checkout#usage).
> *   **Matching Files and Folders**: To match all files and folders under a directory, this requires a globstar pattern e.g. `dir_name/**` which matches any number of subdirectories and files.

Visit the [discussions for more information](https://github.com/step-security/changed-files/discussions) or [create a new discussion](https://github.com/step-security/changed-files/discussions/new/choose) for usage-related questions.

### On [`pull_request`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request) 🔀

Detect changes to all files in a Pull request relative to the target branch or since the last pushed commit.

#### Using local .git directory 📁

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

jobs:
  # ------------------------------------------------------------------------------------------------------------------------------------------------
  # Event `pull_request`: Compare the last commit of the main branch or last remote commit of the PR branch -> to the current commit of a PR branch.
  # ------------------------------------------------------------------------------------------------------------------------------------------------
  changed_files:
    runs-on: ubuntu-latest  # windows-latest || macos-latest
    name: Test changed-files
    steps:
      - uses: actions/checkout@v4

      # -----------------------------------------------------------------------------------------------------------
      # Example 1
      # -----------------------------------------------------------------------------------------------------------
      - name: Get changed files
        id: changed-files
        uses: step-security/changed-files@v46
        # To compare changes between the current commit and the last pushed remote commit set `since_last_remote_commit: true`. e.g
        # with:
        #   since_last_remote_commit: true 

      - name: List all changed files
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files.outputs.all_changed_files }}
        run: |
          for file in ${ALL_CHANGED_FILES}; do
            echo "$file was changed"
          done

      # -----------------------------------------------------------------------------------------------------------
      # Example 2
      # -----------------------------------------------------------------------------------------------------------
      - name: Get all changed markdown files
        id: changed-markdown-files
        uses: step-security/changed-files@v46
        with:
          # Avoid using single or double quotes for multiline patterns
          files: |
            **.md
            docs/**.md

      - name: List all changed files markdown files
        if: steps.changed-markdown-files.outputs.any_changed == 'true'
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-markdown-files.outputs.all_changed_files }}
        run: |
          for file in ${ALL_CHANGED_FILES}; do
            echo "$file was changed"
          done

      # -----------------------------------------------------------------------------------------------------------
      # Example 3
      # -----------------------------------------------------------------------------------------------------------
      - name: Get all test, doc and src files that have changed
        id: changed-files-yaml
        uses: step-security/changed-files@v46
        with:
          files_yaml: |
            doc:
              - '**.md'
              - docs/**
            test:
              - test/**
              - '!test/**.md'
            src:
              - src/**
          # Optionally set `files_yaml_from_source_file` to read the YAML from a file. e.g `files_yaml_from_source_file: .github/changed-files.yml`

      - name: Run step if test file(s) change
        # NOTE: Ensure all outputs are prefixed by the same key used above e.g. `test_(...)` | `doc_(...)` | `src_(...)` when trying to access the `any_changed` output.
        if: steps.changed-files-yaml.outputs.test_any_changed == 'true'  
        env:
          TEST_ALL_CHANGED_FILES: ${{ steps.changed-files-yaml.outputs.test_all_changed_files }}
        run: |
          echo "One or more test file(s) has changed."
          echo "List all the files that have changed: $TEST_ALL_CHANGED_FILES"
      
      - name: Run step if doc file(s) change
        if: steps.changed-files-yaml.outputs.doc_any_changed == 'true'
        env:
          DOC_ALL_CHANGED_FILES: ${{ steps.changed-files-yaml.outputs.doc_all_changed_files }}
        run: |
          echo "One or more doc file(s) has changed."
          echo "List all the files that have changed: $DOC_ALL_CHANGED_FILES"

      # -----------------------------------------------------------------------------------------------------------
      # Example 4
      # -----------------------------------------------------------------------------------------------------------
      - name: Get changed files in the docs folder
        id: changed-files-specific
        uses: step-security/changed-files@v46
        with:
          files: docs/*.{js,html}  # Alternatively using: `docs/**`
          files_ignore: docs/static.js

      - name: Run step if any file(s) in the docs folder change
        if: steps.changed-files-specific.outputs.any_changed == 'true'
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files-specific.outputs.all_changed_files }}
        run: |
          echo "One or more files in the docs folder has changed."
          echo "List all the files that have changed: $ALL_CHANGED_FILES"
```

#### Using Github's API :octocat:

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

jobs:
  # -------------------------------------------------------------
  # Event `pull_request`: Returns all changed pull request files.
  # --------------------------------------------------------------
  changed_files:
    # NOTE:
    # - This is limited to pull_request* events and would raise an error for other events.
    # - A maximum of 3000 files can be returned.
    # - For more flexibility and no limitations see "Using local .git directory" above.

    runs-on: ubuntu-latest  # windows-latest || macos-latest
    name: Test changed-files
    permissions:
      pull-requests: read

    steps:
      - name: Get changed files
        id: changed-files
        uses: step-security/changed-files@v46

      - name: List all changed files
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files.outputs.all_changed_files }}
        run: |
          for file in ${ALL_CHANGED_FILES}; do
            echo "$file was changed"
          done
```

### On [`push`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push) ⬆️

Detect changes to files made since the last pushed commit.

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  # -------------------------------------------------------------
  # Using GitHub's API is not supported for push events
  # -------------------------------------------------------------
  # 
  # ----------------------------------------------------------------------------------------------
  # Using local .git history
  # ----------------------------------------------------------------------------------------------
  # Event `push`: Compare the preceding remote commit -> to the current commit of the main branch 
  # ----------------------------------------------------------------------------------------------
  changed_files:
    runs-on: ubuntu-latest  # windows-latest || macos-latest
    name: Test changed-files
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # OR "2" -> To retrieve the preceding commit.

      - name: Get changed files
        id: changed-files
        uses: step-security/changed-files@v46
      # NOTE: `since_last_remote_commit: true` is implied by default and falls back to the previous local commit.

      - name: List all changed files
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files.outputs.all_changed_files }}
        run: |
          for file in ${ALL_CHANGED_FILES}; do
            echo "$file was changed"
          done
      ...
```

### Other supported events :electron:

*   [schedule](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
*   [release](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#release)
*   [workflow\_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch)
*   [workflow\_run](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_run)
*   [merge\_group](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#merge_group)
*   [issue\_comment](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issue_comment)
*   ...and many more

To access more examples, navigate to the [Examples](#examples-) section.



> \[!IMPORTANT]
>
> *   When using `files_yaml*` inputs:
>     *   All keys must start with a letter or `_` and contain only alphanumeric characters, `-`, or `_`.
>
>         For example, `test` or `test_key` or `test-key` or `_test_key` are all valid choices.

## Inputs ⚙️

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

```yaml
- uses: step-security/changed-files@v46
  id: changed-files
  with:
    # Github API URL.
    # Type: string
    # Default: "${{ github.api_url }}"
    api_url: ''

    # Specify a different base commit 
    # SHA or branch used for 
    # comparing changes 
    # Type: string
    base_sha: ''

    # Exclude changes outside the current 
    # directory and show path names 
    # relative to it. NOTE: This 
    # requires you to specify the 
    # top-level directory via the `path` 
    # input. 
    # Type: boolean
    # Default: "true"
    diff_relative: ''

    # Output unique changed directories instead 
    # of filenames. NOTE: This returns 
    # `.` for changed files located 
    # in the current working directory 
    # which defaults to `$GITHUB_WORKSPACE`. 
    # Type: boolean
    # Default: "false"
    dir_names: ''

    # Include only directories that have 
    # been deleted as opposed to 
    # directory names of files that 
    # have been deleted in the 
    # `deleted_files` output when `dir_names` is 
    # set to `true`. 
    # Type: boolean
    # Default: "false"
    dir_names_deleted_files_include_only_deleted_dirs: ''

    # Exclude the current directory represented 
    # by `.` from the output 
    # when `dir_names` is set to 
    # `true`. 
    # Type: boolean
    # Default: "false"
    dir_names_exclude_current_dir: ''

    # File and directory patterns to 
    # include in the output when 
    # `dir_names` is set to `true`. 
    # NOTE: This returns only the 
    # matching files and also the 
    # directory names. 
    # Type: string
    dir_names_include_files: ''

    # Separator used to split the 
    # `dir_names_include_files` input 
    # Type: string
    # Default: "\n"
    dir_names_include_files_separator: ''

    # Limit the directory output to 
    # a maximum depth e.g `test/test1/test2` 
    # with max depth of `2` 
    # returns `test/test1`. 
    # Type: string
    dir_names_max_depth: ''

    # Escape JSON output.
    # Type: boolean
    # Default: "true"
    escape_json: ''

    # Exclude changes to submodules.
    # Type: boolean
    # Default: "false"
    exclude_submodules: ''

    # Fail when the initial diff 
    # fails. 
    # Type: boolean
    # Default: "false"
    fail_on_initial_diff_error: ''

    # Fail when the submodule diff 
    # fails. 
    # Type: boolean
    # Default: "false"
    fail_on_submodule_diff_error: ''

    # Fetch additional history for submodules.
    # Type: boolean
    # Default: "false"
    fetch_additional_submodule_history: ''

    # Depth of additional branch history 
    # fetched. NOTE: This can be 
    # adjusted to resolve errors with 
    # insufficient history. 
    # Type: string
    # Default: "25"
    fetch_depth: ''

    # Maximum number of retries to 
    # fetch missing history. 
    # Type: string
    # Default: "20"
    fetch_missing_history_max_retries: ''

    # File and directory patterns used 
    # to detect changes (Defaults to the entire repo if unset). NOTE: 
    # Multiline file/directory patterns should not 
    # include quotes. 
    # Type: string
    files: ''

    # Source file(s) used to populate 
    # the `files` input. 
    # Type: string
    files_from_source_file: ''

    # Separator used to split the 
    # `files_from_source_file` input. 
    # Type: string
    # Default: "\n"
    files_from_source_file_separator: ''

    # Ignore changes to these file(s). 
    # NOTE: Multiline file/directory patterns should 
    # not include quotes. 
    # Type: string
    files_ignore: ''

    # Source file(s) used to populate 
    # the `files_ignore` input 
    # Type: string
    files_ignore_from_source_file: ''

    # Separator used to split the 
    # `files_ignore_from_source_file` input 
    # Type: string
    # Default: "\n"
    files_ignore_from_source_file_separator: ''

    # Separator used to split the 
    # `files_ignore` input 
    # Type: string
    # Default: "\n"
    files_ignore_separator: ''

    # YAML used to define a 
    # set of file patterns to 
    # ignore changes 
    # Type: string
    files_ignore_yaml: ''

    # Source file(s) used to populate 
    # the `files_ignore_yaml` input. Example: https://github.com/step-security/changed-files/blob/main/test/changed-files.yml 
    # Type: string
    files_ignore_yaml_from_source_file: ''

    # Separator used to split the 
    # `files_ignore_yaml_from_source_file` input 
    # Type: string
    # Default: "\n"
    files_ignore_yaml_from_source_file_separator: ''

    # Separator used to split the 
    # `files` input 
    # Type: string
    # Default: "\n"
    files_separator: ''

    # YAML used to define a 
    # set of file patterns to 
    # detect changes 
    # Type: string
    files_yaml: ''

    # Source file(s) used to populate 
    # the `files_yaml` input. Example: https://github.com/step-security/changed-files/blob/main/test/changed-files.yml 
    # Type: string
    files_yaml_from_source_file: ''

    # Separator used to split the 
    # `files_yaml_from_source_file` input 
    # Type: string
    # Default: "\n"
    files_yaml_from_source_file_separator: ''

    # Include `all_old_new_renamed_files` output. Note this 
    # can generate a large output 
    # See: #501. 
    # Type: boolean
    # Default: "false"
    include_all_old_new_renamed_files: ''

    # Output list of changed files 
    # in a JSON formatted string 
    # which can be used for 
    # matrix jobs. Example: https://github.com/step-security/changed-files/blob/main/.github/workflows/matrix-example.yml 
    # Type: boolean
    # Default: "false"
    json: ''

    # Output changed files in a 
    # format that can be used 
    # for matrix jobs. Alias for 
    # setting inputs `json` to `true` 
    # and `escape_json` to `false`. 
    # Type: boolean
    # Default: "false"
    matrix: ''

    # Apply the negation patterns first. 
    # NOTE: This affects how changed 
    # files are matched. 
    # Type: boolean
    # Default: "false"
    negation_patterns_first: ''

    # Split character for old and 
    # new renamed filename pairs. 
    # Type: string
    # Default: " "
    old_new_files_separator: ''

    # Split character for old and 
    # new filename pairs. 
    # Type: string
    # Default: ","
    old_new_separator: ''

    # Directory to store output files.
    # Type: string
    # Default: ".github/outputs"
    output_dir: ''

    # Output renamed files as deleted 
    # and added files. 
    # Type: boolean
    # Default: "false"
    output_renamed_files_as_deleted_and_added: ''

    # Specify a relative path under 
    # `$GITHUB_WORKSPACE` to locate the repository. 
    # Type: string
    # Default: "."
    path: ''

    # Use non-ASCII characters to match 
    # files and output the filenames 
    # completely verbatim by setting this 
    # to `false` 
    # Type: boolean
    # Default: "true"
    quotepath: ''

    # Recover deleted files.
    # Type: boolean
    # Default: "false"
    recover_deleted_files: ''

    # Recover deleted files to a 
    # new destination directory, defaults to 
    # the original location. 
    # Type: string
    recover_deleted_files_to_destination: ''

    # File and directory patterns used 
    # to recover deleted files, defaults 
    # to the patterns provided via 
    # the `files`, `files_from_source_file`, `files_ignore` and 
    # `files_ignore_from_source_file` inputs or all deleted 
    # files if no patterns are 
    # provided. 
    # Type: string
    recover_files: ''

    # File and directory patterns to 
    # ignore when recovering deleted files. 
    # Type: string
    recover_files_ignore: ''

    # Separator used to split the 
    # `recover_files_ignore` input 
    # Type: string
    # Default: "\n"
    recover_files_ignore_separator: ''

    # Separator used to split the 
    # `recover_files` input 
    # Type: string
    # Default: "\n"
    recover_files_separator: ''

    # Apply sanitization to output filenames 
    # before being set as output. 
    # Type: boolean
    # Default: "true"
    safe_output: ''

    # Split character for output strings.
    # Type: string
    # Default: " "
    separator: ''

    # Specify a different commit SHA 
    # or branch used for comparing 
    # changes 
    # Type: string
    sha: ''

    # Get changed files for commits 
    # whose timestamp is older than 
    # the given time. 
    # Type: string
    since: ''

    # Use the last commit on 
    # the remote branch as the 
    # `base_sha`. Defaults to the last 
    # non-merge commit on the target 
    # branch for pull request events 
    # and the previous remote commit 
    # of the current branch for 
    # push events. 
    # Type: boolean
    # Default: "false"
    since_last_remote_commit: ''

    # Skip initially fetching additional history 
    # to improve performance for shallow 
    # repositories. NOTE: This could lead 
    # to errors with missing history. 
    # It's intended to be used 
    # when you've fetched all necessary 
    # history to perform the diff. 
    # Type: boolean
    # Default: "false"
    skip_initial_fetch: ''

    # Tags pattern to ignore.
    # Type: string
    tags_ignore_pattern: ''

    # Tags pattern to include.
    # Type: string
    # Default: "*"
    tags_pattern: ''

    # GitHub token used to fetch 
    # changed files from Github's API. 
    # Type: string
    # Default: "${{ github.token }}"
    token: ''

    # Get changed files for commits 
    # whose timestamp is earlier than 
    # the given time. 
    # Type: string
    until: ''

    # Use POSIX path separator `/` 
    # for output file paths on 
    # Windows. 
    # Type: boolean
    # Default: "false"
    use_posix_path_separator: ''

    # Force the use of Github's 
    # REST API even when a 
    # local copy of the repository 
    # exists 
    # Type: boolean
    # Default: "false"
    use_rest_api: ''

    # Write outputs to the `output_dir` 
    # defaults to `.github/outputs` folder. NOTE: 
    # This creates a `.txt` file 
    # by default and a `.json` 
    # file if `json` is set 
    # to `true`. 
    # Type: boolean
    # Default: "false"
    write_output_files: ''

```

<!-- AUTO-DOC-INPUT:END -->

## Useful Acronyms 🧮

|  Acronym  |   Meaning    |
|:---------:|:------------:|
|     A     |    Added     |
|     C     |    Copied    |
|     M     |   Modified   |
|     D     |   Deleted    |
|     R     |   Renamed    |
|     T     | Type changed |
|     U     |   Unmerged   |
|     X     |   Unknown    |

> \[!IMPORTANT]
>
> *   When using `files_yaml*` inputs:
>     *   it's required to prefix all outputs with the key to ensure that the correct outputs are accessible.
>
>         For example, if you use `test` as the key, you can access outputs like `added_files`, `any_changed`, and so on by prefixing them with the key `test_added_files` or `test_any_changed` etc.

## Outputs 📤

<!-- AUTO-DOC-OUTPUT:START - Do not remove or modify this section -->

|                                                                     OUTPUT                                                                     |  TYPE  |                                                                                                                                                       DESCRIPTION                                                                                                                                                       |
|------------------------------------------------------------------------------------------------------------------------------------------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                                      <a name="output_added_files"></a>[added\_files](#output_added_files)                                       | string |                                                                                                                                       Returns only files that are <br>Added (A).                                                                                                                                        |
|                             <a name="output_added_files_count"></a>[added\_files\_count](#output_added_files_count)                              | string |                                                                                                                                           Returns the number of `added_files`                                                                                                                                           |
|          <a name="output_all_changed_and_modified_files"></a>[all\_changed\_and\_modified\_files](#output_all_changed_and_modified_files)          | string |                                                                                                                    Returns all changed and modified <br>files i.e. a combination of <br>(ACMRDTUX)                                                                                                                      |
| <a name="output_all_changed_and_modified_files_count"></a>[all\_changed\_and\_modified\_files\_count](#output_all_changed_and_modified_files_count) | string |                                                                                                                                 Returns the number of `all_changed_and_modified_files`                                                                                                                                  |
|                             <a name="output_all_changed_files"></a>[all\_changed\_files](#output_all_changed_files)                              | string |                                                                                                    Returns all changed files i.e. <br>a combination of all added, <br>copied, modified and renamed files <br>(ACMR)                                                                                                     |
|                    <a name="output_all_changed_files_count"></a>[all\_changed\_files\_count](#output_all_changed_files_count)                     | string |                                                                                                                                        Returns the number of `all_changed_files`                                                                                                                                        |
|                            <a name="output_all_modified_files"></a>[all\_modified\_files](#output_all_modified_files)                            | string |                                                                                              Returns all changed files i.e. <br>a combination of all added, <br>copied, modified, renamed and deleted <br>files (ACMRD).                                                                                                |
|                   <a name="output_all_modified_files_count"></a>[all\_modified\_files\_count](#output_all_modified_files_count)                   | string |                                                                                                                                       Returns the number of `all_modified_files`                                                                                                                                        |
|                 <a name="output_all_old_new_renamed_files"></a>[all\_old\_new\_renamed\_files](#output_all_old_new_renamed_files)                  | string | Returns only files that are <br>Renamed and lists their old <br>and new names. **NOTE:** This <br>requires setting `include_all_old_new_renamed_files` to `true`. <br>Also, keep in mind that <br>this output is global and <br>wouldn't be nested in outputs <br>generated when the `*_yaml_*` input <br>is used. (R)  |
|        <a name="output_all_old_new_renamed_files_count"></a>[all\_old\_new\_renamed\_files\_count](#output_all_old_new_renamed_files_count)         | string |                                                                                                                                    Returns the number of `all_old_new_renamed_files`                                                                                                                                    |
|                                      <a name="output_any_changed"></a>[any\_changed](#output_any_changed)                                       | string |                      Returns `true` when any of <br>the filenames provided using the <br>`files*` or `files_ignore*` inputs have changed. This <br>defaults to `true` when no <br>patterns are specified. i.e. *includes a combination of all added, copied, modified and renamed files (ACMR)*.                        |
|                                      <a name="output_any_deleted"></a>[any\_deleted](#output_any_deleted)                                       | string |                                                             Returns `true` when any of <br>the filenames provided using the <br>`files*` or `files_ignore*` inputs have been deleted. <br>This defaults to `true` when <br>no patterns are specified. (D)                                                               |
|                                     <a name="output_any_modified"></a>[any\_modified](#output_any_modified)                                     | string |            Returns `true` when any of <br>the filenames provided using the <br>`files*` or `files_ignore*` inputs have been modified. <br>This defaults to `true` when <br>no patterns are specified. i.e. <br>*includes a combination of all added, copied, modified, renamed, and deleted files (ACMRD)*.             |
|                                     <a name="output_changed_keys"></a>[changed\_keys](#output_changed_keys)                                     | string |                                                                Returns all changed YAML keys <br>when the `files_yaml` input is <br>used. i.e. key that contains <br>any path that has either <br>been added, copied, modified, and <br>renamed (ACMR)                                                                  |
|                                     <a name="output_copied_files"></a>[copied\_files](#output_copied_files)                                     | string |                                                                                                                                      Returns only files that are <br>Copied (C).                                                                                                                                        |
|                            <a name="output_copied_files_count"></a>[copied\_files\_count](#output_copied_files_count)                            | string |                                                                                                                                          Returns the number of `copied_files`                                                                                                                                           |
|                                   <a name="output_deleted_files"></a>[deleted\_files](#output_deleted_files)                                    | string |                                                                                                                                      Returns only files that are <br>Deleted (D).                                                                                                                                       |
|                          <a name="output_deleted_files_count"></a>[deleted\_files\_count](#output_deleted_files_count)                           | string |                                                                                                                                          Returns the number of `deleted_files`                                                                                                                                          |
|                                  <a name="output_modified_files"></a>[modified\_files](#output_modified_files)                                  | string |                                                                                                                                     Returns only files that are <br>Modified (M).                                                                                                                                       |
|                         <a name="output_modified_files_count"></a>[modified\_files\_count](#output_modified_files_count)                         | string |                                                                                                                                         Returns the number of `modified_files`                                                                                                                                          |
|                                   <a name="output_modified_keys"></a>[modified\_keys](#output_modified_keys)                                    | string |                                                               Returns all modified YAML keys <br>when the `files_yaml` input is <br>used. i.e. key that contains <br>any path that has either <br>been added, copied, modified, and <br>deleted (ACMRD)                                                                 |
|                                     <a name="output_only_changed"></a>[only\_changed](#output_only_changed)                                     | string |                                                           Returns `true` when only files <br>provided using the `files*` or `files_ignore*` inputs <br>have changed. i.e. *includes a combination of all added, copied, modified and renamed files (ACMR)*.                                                             |
|                                     <a name="output_only_deleted"></a>[only\_deleted](#output_only_deleted)                                     | string |                                                                                                  Returns `true` when only files <br>provided using the `files*` or `files_ignore*` inputs <br>have been deleted. (D)                                                                                                    |
|                                   <a name="output_only_modified"></a>[only\_modified](#output_only_modified)                                    | string |                                                                                               Returns `true` when only files <br>provided using the `files*` or `files_ignore*` inputs <br>have been modified. (ACMRD).                                                                                                 |
|                          <a name="output_other_changed_files"></a>[other\_changed\_files](#output_other_changed_files)                           | string |                                                                           Returns all other changed files <br>not listed in the files <br>input i.e. includes a combination <br>of all added, copied, modified <br>and renamed files (ACMR).                                                                            |
|                 <a name="output_other_changed_files_count"></a>[other\_changed\_files\_count](#output_other_changed_files_count)                  | string |                                                                                                                                       Returns the number of `other_changed_files`                                                                                                                                       |
|                          <a name="output_other_deleted_files"></a>[other\_deleted\_files](#output_other_deleted_files)                           | string |                                                                                                 Returns all other deleted files <br>not listed in the files <br>input i.e. a combination of <br>all deleted files (D)                                                                                                   |
|                 <a name="output_other_deleted_files_count"></a>[other\_deleted\_files\_count](#output_other_deleted_files_count)                  | string |                                                                                                                                       Returns the number of `other_deleted_files`                                                                                                                                       |
|                         <a name="output_other_modified_files"></a>[other\_modified\_files](#output_other_modified_files)                         | string |                                                                              Returns all other modified files <br>not listed in the files <br>input i.e. a combination of <br>all added, copied, modified, and <br>deleted files (ACMRD)                                                                                |
|                <a name="output_other_modified_files_count"></a>[other\_modified\_files\_count](#output_other_modified_files_count)                | string |                                                                                                                                      Returns the number of `other_modified_files`                                                                                                                                       |
|                                   <a name="output_renamed_files"></a>[renamed\_files](#output_renamed_files)                                    | string |                                                                                                                                      Returns only files that are <br>Renamed (R).                                                                                                                                       |
|                          <a name="output_renamed_files_count"></a>[renamed\_files\_count](#output_renamed_files_count)                           | string |                                                                                                                                          Returns the number of `renamed_files`                                                                                                                                          |
|                            <a name="output_type_changed_files"></a>[type\_changed\_files](#output_type_changed_files)                            | string |                                                                                                                             Returns only files that have <br>their file type changed (T).                                                                                                                               |
|                   <a name="output_type_changed_files_count"></a>[type\_changed\_files\_count](#output_type_changed_files_count)                   | string |                                                                                                                                       Returns the number of `type_changed_files`                                                                                                                                        |
|                                   <a name="output_unknown_files"></a>[unknown\_files](#output_unknown_files)                                    | string |                                                                                                                                      Returns only files that are <br>Unknown (X).                                                                                                                                       |
|                          <a name="output_unknown_files_count"></a>[unknown\_files\_count](#output_unknown_files_count)                           | string |                                                                                                                                          Returns the number of `unknown_files`                                                                                                                                          |
|                                  <a name="output_unmerged_files"></a>[unmerged\_files](#output_unmerged_files)                                  | string |                                                                                                                                     Returns only files that are <br>Unmerged (U).                                                                                                                                       |
|                         <a name="output_unmerged_files_count"></a>[unmerged\_files\_count](#output_unmerged_files_count)                         | string |                                                                                                                                         Returns the number of `unmerged_files`                                                                                                                                          |

<!-- AUTO-DOC-OUTPUT:END -->

## Versioning 🏷️

This GitHub Action follows the principles of [Semantic Versioning](https://semver.org) for versioning releases.

The format of the version string is as follows:

*   major: indicates significant changes or new features that may not be backward compatible.

*   minor: indicates minor changes or new features that are backward compatible.

*   patch: indicates bug fixes or other small changes that are backward compatible.

## Examples 📄

<details>
<summary>Get all changed files in the current branch</summary>

```yaml
...
    - name: Get changed files
      id: changed-files
      uses: step-security/changed-files@v46
...
```

</details>

<details>
<summary>Get all changed files without escaping unsafe filename characters</summary>

```yaml
...
    - name: Get changed files
      id: changed-files
      uses: step-security/changed-files@v46
      with:
        safe_output: false # set to false because we are using an environment variable to store the output and avoid command injection.

    - name: List all added files
      env:
        ADDED_FILES: ${{ steps.changed-files.outputs.added_files }}
      run: |
        for file in ${ADDED_FILES}; do
          echo "$file was added"
        done
...
```

</details>

<details>
<summary>Get all changed files and use a comma separator</summary>

```yaml
...
    - name: Get all changed files and use a comma separator in the output
      id: changed-files
      uses: step-security/changed-files@v46
      with:
        separator: ","
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary> Get all changed files and list all added files</summary>

```yaml
...
    - name: Get changed files
      id: changed-files
      uses: step-security/changed-files@v46

    - name: List all added files
      env:
        ADDED_FILES: ${{ steps.changed-files.outputs.added_files }}
      run: |
        for file in ${ADDED_FILES}; do
          echo "$file was added"
        done
...
```

See [outputs](#outputs) for a list of all available outputs.

</details>

<details>
<summary>Get all changed files and optionally run a step if a file was modified</summary>

```yaml
...
    - name: Get changed files
      id: changed-files
      uses: step-security/changed-files@v46

    - name: Run a step if my-file.txt was modified
      if: contains(steps.changed-files.outputs.modified_files, 'my-file.txt')
      run: |
        echo "my-file.txt file has been modified."
...
```

See [outputs](#outputs) for a list of all available outputs.

</details>

<details>
<summary>Get all changed files and write the outputs to a txt file</summary>

```yaml
...

   - name: Get changed files and write the outputs to a Txt file
     id: changed-files-write-output-files-txt
     uses: step-security/changed-files@v46
     with:
       write_output_files: true

   - name: Verify the contents of the .github/outputs/added_files.txt file
     run: |
       cat .github/outputs/added_files.txt
...
```

</details>

<details>
<summary>Get all changed files and write the outputs to a json file</summary>

```yaml
...
   - name: Get changed files and write the outputs to a JSON file
     id: changed-files-write-output-files-json
     uses: step-security/changed-files@v46
     with:
       json: true
       write_output_files: true

   - name: Verify the contents of the .github/outputs/added_files.json file
     run: |
       cat .github/outputs/added_files.json
...
```

</details>

<details>
<summary>Get all changed files using a list of files</summary>

```yaml
...
    - name: Get changed files
      id: changed-files
      uses: step-security/changed-files@v46
      with:
        files: |
          my-file.txt
          *.sh
          *.png
          !*.md
          test_directory/**
          **/*.sql
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files using a list of files and take action based on the changes</summary>

```yaml
...
    - name: Get changed files
      id: changed-files-specific
      uses: step-security/changed-files@v46
      with:
        files: |
          my-file.txt
          *.sh
          *.png
          !*.md
          test_directory/**
          **/*.sql

    - name: Run step if any of the listed files above change
      if: steps.changed-files-specific.outputs.any_changed == 'true'
      run: |
        echo "One or more files listed above has changed."

    - name: Run step if only the files listed above change
      if: steps.changed-files-specific.outputs.only_changed == 'true'
      run: |
        echo "Only files listed above have changed."

    - name: Run step if any of the listed files above is deleted
      if: steps.changed-files-specific.outputs.any_deleted == 'true'
      env:
        DELETED_FILES: ${{ steps.changed-files-specific.outputs.deleted_files }}
      run: |
        for file in ${DELETED_FILES}; do
          echo "$file was deleted"
        done

    - name: Run step if all listed files above have been deleted
      if: steps.changed-files-specific.outputs.only_deleted == 'true'
      env:
        DELETED_FILES: ${{ steps.changed-files-specific.outputs.deleted_files }}
      run: |
        for file in ${DELETED_FILES}; do
          echo "$file was deleted"
        done
...
```

See [outputs](#outputs) for a list of all available outputs.

</details>

<details>
<summary>Get all changed files using a source file or list of file(s) to populate to files input</summary>

```yaml
...
    - name: Get changed files using a source file or list of file(s) to populate to files input.
      id: changed-files-specific-source-file
      uses: step-security/changed-files@v46
      with:
        files_from_source_file: test/changed-files-list.txt
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get changed files using a source file or list of file(s) to populate to files input and optionally specify more files</summary>

```yaml
...
    - name: Get changed files using a source file or list of file(s) to populate to files input and optionally specify more files.
      id: changed-files-specific-source-file-and-specify-files
      uses: step-security/changed-files@v46
      with:
        files_from_source_file: |
          test/changed-files-list.txt
        files: |
          test.txt
...
```

See [inputs](#inputs) for more information.

</details>

<details>

<summary>Get all changed files using a different SHA</summary>

```yaml
...
    - name: Get changed files using a different SHA
      id: changed-files
      uses: step-security/changed-files@v46
      with:
        sha: ${{ github.event.pull_request.head.sha }}
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files using a different base SHA</summary>

```yaml
...
    - name: Get changed files using a different base SHA
      id: changed-files
      uses: step-security/changed-files@v46
      with:
        base_sha: ${{ github.event.pull_request.base.sha }}
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files between the previous tag and the current tag</summary>

```yaml
...
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed-files
        uses: step-security/changed-files@v46

      - name: List changed files
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files.outputs.all_changed_files }}
        run: |
          echo "List all the files that have changed: $ALL_CHANGED_FILES"

      - name: Get changed files in the .github folder
        id: changed-files-specific
        uses: step-security/changed-files@v46
        with:
          files: .github/**

      - name: Run step if any file(s) in the .github folder change
        if: steps.changed-files-specific.outputs.any_changed == 'true'
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-files-specific.outputs.all_changed_files }}
        run: |
          echo "One or more files in the .github folder has changed."
          echo "List all the files that have changed: $ALL_CHANGED_FILES"
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files for a repository located in a different path</summary>

```yaml
...
    - name: Checkout into dir1
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
        path: dir1

    - name: Run changed-files with defaults in dir1
      id: changed-files-for-dir1
      uses: step-security/changed-files@v46
      with:
        path: dir1

    - name: List all added files in dir1
      env:
        ADDED_FILES: ${{ steps.changed-files-for-dir1.outputs.added_files }}
      run: |
        for file in ${ADDED_FILES}; do
          echo "$file was added"
        done
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files with non-äšćįí characters i.e (Filename in other languages)</summary>

```yaml
...
    - name: Run changed-files with quotepath disabled
      id: changed-files-quotepath
      uses: step-security/changed-files@v46
      with:
        quotepath: "false"

    - name: Run changed-files with quotepath disabled for a specified list of file(s)
      id: changed-files-quotepath-specific
      uses: step-security/changed-files@v46
      with:
        files: test/test-è.txt
        quotepath: "false"
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files using the last successful commit of the base branch</summary>

<ul>
    <li>
        <details>
        <summary>Push event</summary>

```yaml
...
      - name: Get branch name
        id: branch-name
        uses: tj-actions/branch-names@v6

      - uses: nrwl/nx-set-shas@v3
        id: last_successful_commit_push
        with:
          main-branch-name: ${{ steps.branch-name.outputs.current_branch }} # Get the last successful commit for the current branch.
          workflow-id: 'test.yml'

      - name: Run changed-files with the commit of the last successful test workflow run
        id: changed-files-base-sha-push
        uses: step-security/changed-files@v46
        with:
          base_sha: ${{ steps.last_successful_commit_push.outputs.base }}
...
```

</details>
</li>

<li>
<details>
<summary>Pull request events </summary>

```yaml
...
      - name: Get branch name
        id: branch-name
        uses: tj-actions/branch-names@v5

      - uses: nrwl/nx-set-shas@v3
        id: last_successful_commit_pull_request
        with:
          main-branch-name: ${{ steps.branch-name.outputs.base_ref_branch }} # Get the last successful commit on the master or main branch
          workflow_id: 'test.yml'

      - name: Run changed-files with the commit of the last successful test workflow run on the main branch
        id: changed-files-base-sha-pull-request
        uses: step-security/changed-files@v46
        with:
          base_sha: ${{ steps.last_successful_commit_pull_request.outputs.base }}
...
```

</details>
</li>
</ul>

> **Warning**
>
> This setting overrides the commit sha used by setting `since_last_remote_commit` to true.
> It is recommended to use either solution that works for your use case.

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files but only return the directory names</summary>

```yaml
...
    - name: Run changed-files with dir_names
      id: changed-files-dir-names
      uses: step-security/changed-files@v46
      with:
        dir_names: "true"
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files and return JSON formatted outputs</summary>

```yaml
...
    - name: Run changed-files with JSON output
      id: changed-files-json
      uses: step-security/changed-files@v46
      with:
        json: "true"
...
```

See [inputs](#inputs) for more information.

</details>

<details>
<summary>Get all changed files by commits pushed in the past</summary>

```yaml
...
    - name: Get changed-files since 2022-08-19
      id: changed-files-since
      uses: step-security/changed-files@v46
      with:
        since: "2022-08-19"

    - name: Get changed-files until 2022-08-20
      id: changed-files-until
      uses: step-security/changed-files@v46
      with:
        until: "2022-08-20"
...
```

See [inputs](#inputs) for more information.

</details>

*   Free software: [MIT license](LICENSE)
