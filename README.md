<div align="center">
<h6>Manage package.json environment variables</h6>
<h1>🔅 node-noxenv 🔅</h1>

<br />

<p>Set and get environment variables in your node project package.json, across different platforms</p>

<br />

<div align="center">

<!-- prettier-ignore-start -->
[![Version][github-version-img]][github-version-uri]
[![Version][npm-version-img]][npm-version-uri]
[![Build Status][github-tests-img]][github-tests-uri]
[![Downloads][github-downloads-img]][github-downloads-uri]
[![Size][github-size-img]][github-size-img]
[![Last Commit][github-commit-img]][github-commit-img]
[![Contributors][contribs-all-img]](#contributors-)
<!-- prettier-ignore-end -->

</div>

</div>

---

<br />

- [About](#about)
- [Installation](#installation)
- [Usage](#usage)
- [noxenv vs noxenv-shell](#noxenv-vs-noxenv-shell)
- [Windows Users](#windows-users)
- [Troubleshooting](#troubleshooting)
  - [Error: 'noxenv' is not recognized as an internal or external command](#error-noxenv-is-not-recognized-as-an-internal-or-external-command)
- [Contributors ✨](#contributors-)

<br />

---

<br />

## About

`noxenv` is a NodeJS package which allows you to pass environment variables to your project, including run commands within your `package.json`.

<br />

Most Windows-based command prompts / terminals will have issues when you attempt to set env variables utilizing `NODE_ENV=production`; unless you are using [Bash on Windows][win-bash]. Coupled with the issue that Windows and *NIX have different ways of utilizing env variables such as:

- Windows: `%ENV_VAR%`
- *NIX: `$ENV_VAR`

<br />

`noxenv` gives you the ability to have a single command at your disposal; without the hassle of worrying about different platforms. Specify your env variable as you would on *nix, and noxenv will take care of the conversion for Windows users.

<br />

> [!NOTE]
> `noxenv` only supports Node.js v16 and higher. As of May 2025, NodeJS v15 has been sunset.

<br />

---

<br />

## Installation

This module is distributed via [npm][npm] which is bundled with [node][node]. To utilize this moduke, add it to your project's `package.json` -> `devDependencies`:

```json
"devDependencies": {
    "@aetherinox/noxenv": "^1.1.0"
},
```

<br />

To install it via the npm command-line as a `devDependency`:

```
npm i --save-dev @aetherinox/noxenv
```

<br />

---

<br />

## Usage

Some examples have been provided below to show various ways of using noxenv:

<br />

```json
{
    "scripts": {
        "build": "noxenv NODE_ENV=production webpack --config build/webpack.config.js",
        "build-rollup": "noxenv NODE_ENV=production rollup -c",
        "development": "noxenv NODE_ENV=development npm start",
        "production": "noxenv NODE_ENV=production SERVER_IP=http://127.0.0.1 npm start",
        "test": "noxenv BABEL_ENV=test jest test/app",
        "start-dev": "noxenv NODE_ENV=development PORT_ENV=2350 npm run build && node dist/src/main.js",
        "openssl-legacy": "noxenv NODE_OPTIONS=\"--openssl-legacy-provider\" tsc -p tsconfig.json"
    }
}
```

<br />

Inside your module, you can call these env variables with something similar to the below example:

```javascript
const TEST = { api: "f4dcc990-f8f7-4343-b852-a2065b4445d5" };
const PROD = { api: "d1ac1eb8-7194-4095-8976-04be09378611" };

let target;
if (process.env.BABEL_ENV === "development") {
    target = TEST;
} else if (process.env.BABEL_ENV === "production") {
    target = PROD;
}

console.log(`Running ${process.env.BABEL_ENV} mode; API key is ${target}`);
```

<br />

In the above example, variables such as `BABEL_ENV` will be set by `noxenv`.  You can also set multiple environment variables at a time:

```json
{
    "scripts": {
        "release-beta": "noxenv RELEASE=beta ENV=dev PORT=7732 npm run release && npm run start",
    }
}
```

<br />

Additionally; you can split the command into several actions, or separate the env variable declarations from the actual command execution; note the following example:

```json
{
    "scripts": {
        "parent": "noxenv USER_NAME=\"Joe\" npm run child",
        "child": "noxenv-shell \"echo Hello $USER_NAME\""
    }
}
```

<br />

In the above example, `child` stores the actual command to execute, and `parent` sets the env variable that is going to be used. In this case, `Joe` is the user we want to say hello to, so we store Joe's name in within the env variable `USER_NAME`, and then at the end of `parent`, we call `child` which does the actual greeting.

<br />

This means that you only need to call `parent`, and both will be ran. Additionally, it also means that you can also call the env variable using `$USER_NAME` on Windows, even though the typical Windows syntax is `%USER_NAME%`.

<br />

If you wish to pass a JSON string (such as when using [ts-loader]), you may do the following:

<br />

```json
{
    "scripts": {
        "test": "noxenv TS_NODE_COMPILER_OPTIONS={\\\"module\\\":\\\"commonjs\\\"} mocha --config ./test.js"
    }
}
```

<br />

Take note of the `triple backslashes` `(\\\)` before `double quotes` `(")` and the absence of `single quotes` `(')`. This syntax is critical in order for the env var to work on both on Windows and *NIX.

<br />

---

<br />

## noxenv vs noxenv-shell

`noxenv` provides two binary files: `noxenv` and `noxenv-shell`.

<br />

| Binary | Description |
| --- | --- |
| `noxenv` | Executes commands utilizing [`cross-spawn`][cross-spawn] |
| `noxenv-shell` | Executes commands utilizing node's `shell`. Useful when you need an env var to be set across an entire shell script, rather than a single command. Also used when wanting to pass a command that contains special shell characters that you need interpreted |

<br />

If you want to have the env variable apply to several commands in a series, you will need to wrap them in quotes and use `noxenv-shell`, instead of `noxenv`.

```json
{
    "scripts": {
        "salutation": "noxenv-shell SALUTATION=Howdy NAME=Aetherinox \"echo $SALUTATION && echo $NAME\""
    }
}
```

<br />

If you need to handle [signal events](https://nodejs.org/api/process.html#process_signal_events) within your project, use `noxenv-shell`. An example use for this is when you want to capture the `SIGINT` event invoked by pressing `Ctrl + C` within your command-line interface.

<br />

---

<br />

## Windows Users

Note that `npm` uses `cmd` by default and doesn't support command substitution, so if you want to leverage that, then you need to update your `.npmrc` and set `script-shell` to powershell.

<br />

---

<br />

## Troubleshooting

Review the following list of issues and corrective measures you can take to address the issues:

<br />

### Error: 'noxenv' is not recognized as an internal or external command

If you get the following error when running a NodeJS script which utilizes noxenv:

```shell
'noxenv' is not recognized as an internal or external command,
operable program or batch file.
```

<br />

Ensure you install it first locally as a dev dependency:

```shell
npm i --save-dev @aetherinox/noxenv
```

<br />

---

<br />

## Contributors ✨

We are always looking for contributors. If you feel that you can provide something useful to this project, then we'd love to review your suggestion. Before submitting your contribution, please review the following resources:

- [Pull Request Procedure](.github/PULL_REQUEST_TEMPLATE.md)
- [Contributor Policy](CONTRIBUTING.md)

<br />

Want to help but can't write code?

- Review [active questions by our community](https://github.com/Aetherinox/node-noxenv/labels/❔%20Question) and answer the ones you know.

<br />

The following people have helped keep this project going:

<br />

<div align="center">

<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![Contributors][contribs-all-img]](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top"><a href="https://github.com/Aetherinox"><img src="https://avatars.githubusercontent.com/u/118329232?v=4&s=40" width="80px;" alt="Aetherinox"/><br /><sub><b>Aetherinox</b></sub></a><br /><a href="https://github.com/Aetherinox/noxenv/commits?author=Aetherinox" title="Code">💻</a> <a href="#projectManagement-Aetherinox" title="Project Management">📆</a></td>
    </tr>
  </tbody>
</table>
</div>
<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->
<!-- ALL-CONTRIBUTORS-LIST:END -->

<br />
<br />

<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<br />

---

<br />

<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<!-- BADGE > GENERAL -->
  [general-npmjs-uri]: https://npmjs.com
  [general-nodejs-uri]: https://nodejs.org
  [general-npmtrends-uri]: https://npmtrends.com/@aetherinox/noxenv

<!-- BADGE > VERSION > GITHUB -->
  [github-version-img]: https://img.shields.io/github/v/tag/aetherinox/node-noxenv?logo=GitHub&label=Version&color=ba5225
  [github-version-uri]: https://github.com/Aetherinox/node-noxenv/releases

<!-- BADGE > VERSION > NPMJS -->
  [npm-version-img]: https://img.shields.io/npm/v/@aetherinox/noxenv?logo=npm&label=Version&color=ba5225
  [npm-version-uri]: https://npmjs.com/package/@aetherinox/noxenv

<!-- BADGE > VERSION > PYPI -->
  [pypi-version-img]: https://img.shields.io/pypi/v/noxenv
  [pypi-version-uri]: https://pypi.org/project/noxenv/

<!-- BADGE > LICENSE > MIT -->
  [license-mit-img]: https://img.shields.io/badge/MIT-FFF?logo=creativecommons&logoColor=FFFFFF&label=License&color=9d29a0
  [license-mit-uri]: https://github.com/Aetherinox/node-noxenv/blob/main/LICENSE

<!-- BADGE > GITHUB > DOWNLOAD COUNT -->
  [github-downloads-img]: https://img.shields.io/github/downloads/Aetherinox/node-noxenv/total?logo=github&logoColor=FFFFFF&label=Downloads&color=376892
  [github-downloads-uri]: https://github.com/Aetherinox/node-noxenv/releases

<!-- BADGE > NPMJS > DOWNLOAD COUNT -->
  [npmjs-downloads-img]: https://img.shields.io/npm/dw/%40aetherinox%2Fnoxenv?logo=npm&&label=Downloads&color=376892
  [npmjs-downloads-uri]: https://npmjs.com/package/@aetherinox/noxenv

<!-- BADGE > GITHUB > DOWNLOAD SIZE -->
  [github-size-img]: https://img.shields.io/github/repo-size/Aetherinox/node-noxenv?logo=github&label=Size&color=59702a
  [github-size-uri]: https://github.com/Aetherinox/node-noxenv/releases

<!-- BADGE > NPMJS > DOWNLOAD SIZE -->
  [npmjs-size-img]: https://img.shields.io/npm/unpacked-size/@aetherinox/noxenv/latest?logo=npm&label=Size&color=59702a
  [npmjs-size-uri]: https://npmjs.com/package/@aetherinox/noxenv

<!-- BADGE > CODECOV > COVERAGE -->
  [codecov-coverage-img]: https://img.shields.io/codecov/c/github/Aetherinox/node-noxenv?token=MPAVASGIOG&logo=codecov&logoColor=FFFFFF&label=Coverage&color=354b9e
  [codecov-coverage-uri]: https://codecov.io/github/Aetherinox/node-noxenv

<!-- BADGE > ALL CONTRIBUTORS -->
  [contribs-all-img]: https://img.shields.io/github/all-contributors/Aetherinox/node-noxenv?logo=contributorcovenant&color=de1f6f&label=contributors
  [contribs-all-uri]: https://github.com/all-contributors/all-contributors

<!-- BADGE > GITHUB > BUILD > NPM -->
  [github-build-img]: https://img.shields.io/github/actions/workflow/status/Aetherinox/node-noxenv/npm-release.yml?logo=github&logoColor=FFFFFF&label=Build&color=%23278b30
  [github-build-uri]: https://github.com/Aetherinox/node-noxenv/actions/workflows/npm-release.yml

<!-- BADGE > GITHUB > BUILD > Pypi -->
  [github-build-pypi-img]: https://img.shields.io/github/actions/workflow/status/Aetherinox/node-noxenv/release-pypi.yml?logo=github&logoColor=FFFFFF&label=Build&color=%23278b30
  [github-build-pypi-uri]: https://github.com/Aetherinox/node-noxenv/actions/workflows/pypi-release.yml

<!-- BADGE > GITHUB > TESTS -->
  [github-tests-img]: https://img.shields.io/github/actions/workflow/status/Aetherinox/node-noxenv/npm-tests.yml?logo=github&label=Tests&color=2c6488
  [github-tests-uri]: https://github.com/Aetherinox/node-noxenv/actions/workflows/npm-tests.yml

<!-- BADGE > GITHUB > COMMIT -->
  [github-commit-img]: https://img.shields.io/github/last-commit/Aetherinox/node-noxenv?logo=conventionalcommits&logoColor=FFFFFF&label=Last%20Commit&color=313131
  [github-commit-uri]: https://github.com/Aetherinox/node-noxenv/commits/main/

<!-- prettier-ignore-end -->
<!-- markdownlint-restore -->
