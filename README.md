# Vidiff

[Vidiff](https://vidiff.com) is a visual regression testing tools that compares screenshots of your app between different stages (eg: production vs staging) accross major operating systems, browsers and screen resolutions.

## Prerequisites

Vidiff assumes your application is mature enough to be deployed on at least two stages. It also assumes that each stage corresponds to a git repository branch.

Example:
- Production (branch master): https://example.com
- Staging (branch dev): https://staging.example.com

Once those prerequisites are defined Vidiff starts comparing changes across both stages.

## Getting started

First, [create your Vidiff account](https://app.vidiff.com/signup).

You will be prompted for an integration:
- If you choose GitHub, your repositories will be imported.
- If you choose no integration, create a new Project that points to your app's repository:

![image](https://user-images.githubusercontent.com/4154003/60613839-fe302200-9dcb-11e9-8881-d9b8cb46f85e.png)

If you do not want to share your code with Vidiff, read the [section about hidding your source](#hidding-your-source).

Then to try Vidiff out checkout a new branch:
```
git checkout -b install-vidiff
```

Vidiff needs at least two files to work: a `.vidiffrc` configuration file and a scenario test file. You can create both using:
```
npx vidiff init --url https://your.application.com --entry path/to/scenario.js
```
Then commut and push your work.

To launch your first build, grab the API token in your account page and the project ID in the project page. Then you can do:
```
npx vidiff create-build -t API_TOKEN -p PROJECT_ID
```

This will launch a build on a single branch, that is a build without comparison accross branches/stages. Good enough to start.

## Going further

To use Vidiff in full there are a few more steps you should take:

- Enhance your scenario test file with a tour of your application's pages and features. [Here's how to](#defining-a-scenario-function).
- Install Vidiff in another branch (or merge the branch you created into another), add a `branchToUrlMapping` entry in your `.vidiffrc` file in the new branch to tell Vidiff which URL to test your scenario against.
- Launch a build that compares two stages using `npx vidiff create-build -t API_TOKEN -p PROJECT_ID --baselineBranch master --comparisonBranch dev`. Vidiff must be installed in both branches.
- When you develop a new feature, adapt your scenario test file to make new functionnal tests and take new screenshots.

## Defining a configuration

The configuration file must be placed at the root of your repository. It must be named `.vidiffrc`. It must be present on both considered branches.

```json
{
  "branchToUrlMapping": {
    "master": "https://example.com",
    "dev": "https://staging.example.com"
  },
  "scenarios": [
    {
      "name": "Website tour",
      "entry": "test/visual-regression-test.js",
      "capabilities": [
        {
          "platform": "WIN10",
          "browserName": "chrome",
          "version": "75"
        },
        {
          "platform": "MOJAVE",
          "browserName": "safari",
          "version": "12"
        },
        {
          "platform": "LINUX",
          "browserName": "firefox",
          "version": "67",
          "screen-resolution": "800x600"
        },
        {
          "platform": "WIN10",
          "browserName": "microsoftedge",
          "version": "18"
        },
        {
          "platform": "WIN10",
          "browserName": "internet explorer",
          "version": "11"
        },
        {
          "platform": "ANDROID",
          "platformName": "Android",
          "deviceName": "Pixel 2",
          "browserName": "Chrome",
          "version": "7.1"
        }
      ]
    }
  ]
}
```

### branchToUrlMapping

Define for each branch which URL to run tests against.

You can also define `branchToUrlMapping` under `scenarios[]` to specify particular URLs for each scenario.

### pullRequestUrlTemplate

For GitHub integration users only.

Set for each pull request an URL. The value `${n}` will be replaced by the pull request number.

example:
```js
"pullRequestUrlTemplate": "https://pr-${n}.example.com" // PR 123 --> https://pr-123.example.com
```

You can also define `pullRequestUrlTemplate` under `scenarios[]` to specify a particular pull request URL templates for each scenario.

### scenarios

You can run multiple scenarios in one build.

`name`: A unique name for your scenario. Required.

`entry`: The scenario function path relative to the root of your repository.

`capabilities`: An array describing the browsers you want your scenario to be tested uppon.

The complete list of capabilities can be found [here](https://testingbot.com/support/getting-started/browsers.html).

The list of available `screen-resolution` for desktop capabilities can be found [here](https://testingbot.com/support/other/test-options#screenresolution). The default value is `1920x1080`.

## Defining a scenario function

It corresponds to the `entry` parameter in your scenario definition in your `.vidiffrc` file. It must be present on both compared branches but can be different as new features often mean new visual and functionnal tests. The function will be executed once for every capability you set.

```js
async function scenario(browser, baseUrl, log) {
  // Go to any URL
  // The baseUrl is defined in your .vidiffrc file under "branchToUrlMapping"
  await browser.get(baseUrl + '/signin')

  // Take your first screenshot
  await takeScreenshot('Signin', 'The signin page')

  // Get DOM elements
  const emailInput = await browser.elementByCssSelector('#email')
  const passwordInput = await browser.elementByCssSelector('#password')
  const submitButton = await browser.elementByCssSelector('#submit')

  // Type data into inputs
  await emailInput.type('user@vidiff.com'.split(''))
  await passwordInput.type('password1234'.split(''))

  // Click a button
  await submitButton.click()

  // Take the second screenshot
  await takeScreenshot('Home', 'The main route')

  // Transition page again
  const aboutLink = await browser.elementByCssSelector('nav>ul>li:nth-child(2)>a')

  await aboutLink.click()

  // Take the third screenshot
  await takeScreenshot('About', 'The about page')

  // Log anything
  log('This log will appear on the build page')

  // Conditionnal testing
  if (browser.capability.platformName === 'iOS') {
    // ...
  }
}

module.exports = scenario
```

At the moment scenario functions cannot import external modules (installed with `npm install`). In a future version it will be possible.

### Scenario function arguments

#### browser

See the [wd documentation](https://github.com/admc/wd). For security reasons we removed some of the methods.

##### browser.capability

The considered capability as defined in your `.vidiffrc` file.

##### browser.takeScreenshot(name<string>, description<string>)

Records a screenshot to be compared by Vidiff.

**name**: Required. An identifier for your screenshot. Must be unique. Screenshots with the same name in both the baseline and comparison stage will be compared.

**description**: Optional. A description of your screenshot.

#### baseUrl

A string containing the URL your scenario will execute against. Use it to transition routes. It should be different accross stages.

#### log(...messages)

A log function. Logs appear on the build page in our app. Logs are prefixed with a string to identify the capability.

## Using Vidiff to run functionnal tests

You can also use Vidiff to run your application tests on multiple platforms and browsers with statements like `assert` and `expect` or `should`.

At the moment, the [wd driver](https://github.com/admc/wd) and the `assert` NodeJS library allow such statements.

## Launching your first build

For every project, three options are available:

- Use the [REST API](#rest-api)
- Use the [`vidiff` NPM package](https://www.npmjs.com/package/vidiff).
- Launch a build manually in the Vidiff app:

![image](https://user-images.githubusercontent.com/4154003/60613909-2455c200-9dcc-11e9-9a0d-973efefb1e3e.png)

For GitHub-integrated projects, pushing a commit to an opened pull request between branches with Vidiff installed will trigger a new build.

## Hidding your source

At Vidiff we only clone your repository, checkout the two branches you provided, read the configuration file and import the scenarios handlers. The repository folder is then deleted, the rest of your source is untouched. We never write to your repository.

If you want to hide your source from us you can create a repository containing only the `.vidiffrc` configuration file and the scenario functions since that's all we need.

## REST API

To launch builds automatically, a REST API is available.

To authenticate, use the account API token available in [your account page](https://app.vidiff.com/account).

The project id is available in the project page.

### Create build

`POST https://api.vidiff.com/rest/build`

#### Request body

```json
{
  "accountToken": "...",
  "projectId": "...",
  "baselineBranch": "master",
  "comparisonBranch": "dev"
}
```

#### Response

`202 Accepted`

```json
{
  "message": "Build queued",
  "buildUrl": "https://app.vidiff.com/project/~/vidiff-demo-website/build/1"
}
```

## Bugs and new feature requests

Please improve Vidiff by reporting bugs and requesting new features. To do so, [start a new issue](https://github.com/vidiff/vidiff-documentation-and-issues/issues/new) describing the bug or feature need.

## Contact

Get it touch at [hello@vidiff.com](mailto:hello@vidiff.com) or [start a new issue](https://github.com/vidiff/vidiff-documentation-and-issues/issues/new). We love to here news from our users.
