# Vidiff

[Vidiff](https://vidiff.com) is a visual regression testing tools that compares screenshots of your app between different stages (eg: production vs staging) accross major operating systems, browsers and screen resolutions.

## Prerequisites

Vidiff assumes your application is mature enough to be deployed on at least two stages. It also assumes that each stage corresponds to a git repository branch.

Example:
- Production (branch master): https://example.com
- Staging (branch dev): https://staging.example.com

Once those prerequisites are defined Vidiff starts comparing changes on both stages.

## Getting started

First, [create your Vidiff account](https://app.vidiff.com/signup). Whether you want to use GitHub or not is up to you. If you choose not to use GitHub, create a new Project that points to your app's repository.

If you do not want to share your code with Vidiff, read the [section about hidding your source](#foo).

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
          "platform": "VISTA",
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

### scenarios

You can run multiple scenarios in one build.

`name`: A unique name for your scenario. Required.

`entry`: The scenario function path relative to the root of your repository.

`capabilities`: An array describing the browsers you want your scenario to be tested uppon.

The complete list of `platform`, `browsername` and `version` can be found [here](https://testingbot.com/support/getting-started/browsers.html).

The list of available `screen-resolution` can be found [here](https://testingbot.com/support/other/test-options#screenresolution). The default value is `1920x1080`.

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

You can also define `pullRequestUrlTemplate` under `scenarios[]` to specify a particular pull request URL template for each scenario.

## Defining a scenario function

It corresponds to the `entry` parameter in your scenario definition in your `.vidiffrc` file. It must be present on both compared branches but can be different as new features often mean new visual and functionnal tests.

```js
async function scenario(browser, takeScreenshot, baseUrl) {
  // Go to any URL
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
}

module.exports = scenario
```

At the moment scenario functions cannot import external modules (installed with `npm install`). In a future version it will be possible.

### Scenario function arguments

#### browser

See the [wd documentation](https://github.com/admc/wd). In a future version a custom driver will be available.

#### takeScreenshot(name<string>, description<string>)

Records a screenshot to be compared by Vidiff.

**name**: Required. An identifier for your screenshot. Must be unique. Screenshots with the same name in both the baseline and comparison stage will be compared.

**description**: Optional. A description of your screenshot.

### baseUrl

A string containing the URL your scenario will execute against. Use it to transition routes. It should be different accross stages.

## Using Vidiff to run functionnal tests

You can also use Vidiff to run your application tests on multiple platforms and browsers with statements like `assert` and `expect` or `should`.

At the moment, the [wd driver](https://github.com/admc/wd) and the `assert` NodeJS library allow such statements.

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

## Contact

Get it touch at [hello@vidiff.com](mailto:hello@vidiff.com). We love to here news from our users.
