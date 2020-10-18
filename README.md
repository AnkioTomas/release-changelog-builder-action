<div align="center">
  :octocat:📄🔖📦
</div>
<h1 align="center">
  release-changelog-builder-action
</h1>

<p align="center">
    ... a github action that builds your release notes fast, easy and exactly the way you want.
</p>

<div align="center">
  <a href="https://github.com/mikepenz/release-changelog-builder-action/actions">
		<img src="https://github.com/mikepenz/release-changelog-builder-action/workflows/CI/badge.svg"/>
	</a>
</div>
<br />

-------

<p align="center">
    <a href="#whats-included-">What's included 🚀</a> &bull;
    <a href="#setup">Setup 🛠️</a> &bull;
    <a href="#customization-%EF%B8%8F">Customization 🖍️</a> &bull;
    <a href="#contribute-">Contribute 🧬</a> &bull;
    <a href="#complete-sample-%EF%B8%8F">Complete Sample 🖥️</a> &bull;
    <a href="#license">License 📓</a>
</p>

-------

### What's included 🚀

- Super simple integration
  - ...even on huge repositories with hundreds of tags
- Parallel releases support
- Blazingly fast execution
- Supports any git project
- Highly flexible configuration
- Lightweight
- Supports any branch
- Rich build log output

-------

## Setup

### Configure the workflow

Specify the action as part of your GitHub actions workflow:

```yml
- name: "Build Changelog"
  id: build_changelog
  uses: mikepenz/release-changelog-builder-action@{latest-release}
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

By default the action will try to automatically retrieve the `tag` from the current commit and automtacally resolve the `tag` before. Read more about this here.

### Action outputs

After action execution it will return the `changelog` and additional information as step output. You can use it in any follow up step by referenceing the output by referencing it via the steps id. For example `build_changelog`.

```yml
# ${{steps.{CHANGELOG_STEP_ID}.outputs.changelog}}
${{steps.build_changelog.outputs.changelog}}
```

A full set list of possible output values for this action.

| **Output**          | **Description**                                                                                                           |
|---------------------|---------------------------------------------------------------------------------------------------------------------------|
| `outputs.changelog` | The built release changelog built from the merged pull requests                                                           |
| `outputs.owner`     | Specifies the owner of the repository processed                                                                           |
| `outputs.repo`      | Describes the repository name, which was processed                                                                        |
| `outputs.fromTag`   | Defines the `fromTag` which describes the lower bound to process pull requests for                                        |
| `outputs.toTag`     | Defines the `toTag` which describes the upper bound to process pull request for                                           |
| `outputs.failed`    | Defines if there was an issue with the action run, and the changelog may not have been generated correctly. [true, false] |

## Customization 🖍️

### Changelog Configuration

The action supports flexible configuration options to modify vast areas of its behavior. To do so, provide the configuration file to the workflow using the `configuration` setting.

```yml
- name: "Build Changelog"
  uses: mikepenz/release-changelog-builder-action@{latest-release}
  with:
    configuration: "configuration.json"
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This configuration is a `.json` file in the following format.

```json
{
    "categories": [
        {
            "title": "## 🚀 Features",
            "labels": ["feature"]
        },
        {
            "title": "## 🐛 Fixes",
            "labels": ["fix"]
        },
        {
            "title": "## 🧪 Tests",
            "labels": ["test"]
        }
    ],
    "sort": "ASC",
    "template": "${{CHANGELOG}}\n\n<details>\n<summary>Uncategorized</summary>\n\n${{UNCATEGORIZED}}\n</details>",
    "pr_template": "- ${{TITLE}}\n   - PR: #${{NUMBER}}",
    "empty_template": "- no changes",
    "transformers": [
        {
            "pattern": "[\\-\\*] (\\[(...|TEST|CI|SKIP)\\])( )?(.+?)\n(.+?[\\-\\*] )(.+)",
            "target": "- $4\n  - $6"
        }
    ],
    "max_tags_to_fetch": 200,
    "max_pull_requests": 200,
    "max_back_track_time_days": 90,
    "exclude_merge_branches": [
        "Owner/qa"
    ]
}
```

Any section of the configruation can be ommited to have defaults apply.
Defaults for the configuraiton can be found in the [configuration.ts](https://github.com/mikepenz/release-changelog-builder-action/blob/develop/src/configuration.ts)


### Advanced workflow specification

For advanced usecases additional settings can be provided to the action

```yml
- name: "Complex Configuration"
  id: build_changelog
  if: startsWith(github.ref, 'refs/tags/')
  uses: mikepenz/release-changelog-builder-action@{latest-release}
  with:
    configuration: "configuration_complex.json"
    owner: "mikepenz"
    repo: "release-changelog-builder-action"
    ignorePreReleases: "false"
    fromTag: "0.0.2"
    toTag: "0.0.3"
    token: ${{ secrets.PAT }}
```

💡 All input values are optional. It is only required to privde the `token` either via the input, or as `env` variable.

| **Input**         | **Description**                                                                                                                                                          |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| configuration     | Relative path, to the `configuration.json` file, providing additional configurations                                                                                     |
| owner             | The owner of the repository to generate the changelog for                                                                                                                |
| repo              | Name of the repository we want to process                                                                                                                                |
| fromTag           | Defines the 'start' from where the changelog will consider merged pull requests                                                                                          |
| toTag             | Defines until which tag the changelog will consider merged pull requests                                                                                                 |
| path              | Allows to specify an alternative sub directory, to use as base                                                                                                           |
| token             | Alternative config to specify token. You should prefer `env.GITHUB_TOKEN` instead though                                                                                 |
| ignorePreReleases | Allows to ignore pre-releases for changelog generation (E.g. for 1.0.1... 1.0.0-rc02 <- ignore, 1.0.0 <- pick). Only used if `fromTag` was not specified. Default: false |
| failOnError       | Defines if the action will result in a build failure, if problems occurred. Default: false                                                                               |

💡 `${{ secrets.GITHUB_TOKEN }}` only grants rights to the current repository, for other repos please use a PAT (Personal Access Token).

### PR Template placeholders

Table of supported placeholders allowed to be used in the `template` configuration.

| **Placeholder**  | **Description**                                             |
|------------------|-------------------------------------------------------------|
| `${{NUMBER}}`    | The number referencing this pull request. E.g. 13           |
| `${{TITLE}}`     | Specified title of the merged pull request                  |
| `${{URL}}`       | Url linking to the pull request on GitHub                   |
| `${{MERGED_AT}}` | The ISO time, the pull request was merged at                |
| `${{AUTHOR}}`    | Author creating and opening the pull request                |
| `${{LABELS}}`    | The labels associated with this pull request, joined by `,` |
| `${{MILESTONE}}` | Milestone this PR was part of, as assigned on GitHub        |
| `${{BODY}}`      | Description/Body of the pull request as specified on GitHub |
| `${{ASSIGNEES}}` | Login names of assigned GitHub users, joined by `,`         |
| `${{REVIEWERS}}` | GitHub Login names of specified reviewers, joined by `,`    |

### Template placeholders

Table of supported placeholders allowed to be used in the `pr_template` configuration.

| **Placeholder**      | **Description**                                                                                 |
|----------------------|-------------------------------------------------------------------------------------------------|
| `${{CHANGELOG}}`     | The contents of the changelog, matching the labels as specified in the categories configuration |
| `${{UNCATEGORIZED}}` | All pull requests not matching a specified label in categories                                  |


## Complete Sample 🖥️

Below is a complete example showcasing how to define a build, which is executed when tagging the project. It consists of:
- Prepare tag, via the GITHUB_REF environment variable
- Build changelog, given the tag
- Create release on GitHub - specifying body with constructed changelog
 
```yml
name: 'CI'
on:
  push:
    tags:
      - '*'

  release:
    if: startsWith(github.ref, 'refs/tags/')
    runs-on: ubuntu-latest
    steps:
      - name: Build Changelog
        id: github_release
        uses: mikepenz/release-changelog-builder-action@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Create Release
        uses: actions/create-release@v1
        with:
          tag_name: ${{ github.ref }}
          release_name: ${{ github.ref }}
          body: ${{steps.github_release.outputs.changelog}}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Contribute 🧬

```bash
# Install the dependencies  
$ npm install

# Build the typescript and package it for distribution
$ npm run build && npm run package

# Run the tests, use to debug, and test it out
$ npm test

# Verify lint is happy
$ npm run lint -- --fix
```

It's suggested to export the token to your path before running the tests, so that API calls can be done to github.

```bash
export GITHUB_TOKEN=your_personal_github_pat
```

## Developed By

* Mike Penz
 * [mikepenz.com](http://mikepenz.com) - <mikepenz@gmail.com>
 * [paypal.me/mikepenz](http://paypal.me/mikepenz)

## Credits

Core parts of the PR fetching logic are based on [pull-release-notes](https://github.com/nblagoev/pull-release-notes)
- Nikolay Blagoev - [GitHub](https://github.com/nblagoev/)

## License

   Copyright for portions of pr-release-notes are held by Nikolay Blagoev, 2019-2020 as part of project pull-release-notes. All other copyright for project pr-release-notes are held by Mike Penz, 2020.

## Fork License

All patches and changes applied to the original source are licensed under the Apache 2.0 license.

    Copyright 2020 Mike Penz

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
