# Jest setup guide

> **NOTE: This article previously focused on deprecated `jest-jasmine2` runner setup, and if you nevertheless need to access it, [follow this Git history link](https://github.com/wix/Detox/blob/ef466822129a4befcda71111d02b1a334539889b/docs/Guide.Jest.md).**


This guide describes how to install [Jest](https://jestjs.io) as a test runner to be used by Detox for running the E2E tests.

**Disclaimer:**

1. Here we focus on installing Detox on _new projects_. If you're migrating a project with an existing Detox installation, please apply some common sense while using this guide.
1. These instructions are relevant for `jest-circus@^26.0.1`. They should likely work for the newer `jest-circus` versions too, but for **the older ones** (25.x, 24.x) &mdash; **they will not, due to blocking issues.**

## Introduction

As already mentioned in the [Getting Started](Introduction.GettingStarted.md#step-3-create-your-first-test) guide, Detox itself does not effectively run tests logic, but rather delegates that responsibility onto a test runner. At the moment, Jest is the only recommended choice, for many reasons, including but not limited to parallel test suite execution capability, and complete integration with Detox API.

By the way, Jest itself — much like Detox, also does not effectively run any tests. Instead, it is more of a dispatcher and orchestrator of multiple instances of a delegated runner capable of running in parallel. For more info, refer to [this video](https://youtu.be/3YDiloj8_d0?t=2127) (source: [Jest architecture](https://jestjs.io/docs/en/architecture)).

For its part, Detox supports only one Jest's concrete runner, which is [`jest-circus`](https://www.npmjs.com/package/jest-circus). The former runner, `jest-jasmine2`, is deprecated due to specific bugs in the past, and architectural limitations at present. Moreover, Jest team plans to deprecate `jest-jasmine2` in the upcoming major release 27.0.0 ([see blog post](https://jestjs.io/blog/2020/05/05/jest-26)).

## Installation

### 1. Install Jest

Before starting with Jest setup, be sure to complete the preliminary sections of the [Getting Started](Introduction.GettingStarted.md) guide.

Afterward, install the respective npm packages:

```sh
npm install jest jest-circus --save-dev --no-package-lock
```

If you are already using Jest in your project,
make sure that `jest` and `jest-circus` package versions match (e.g., both are `26.0.1`).

### 2. Set up test-code scaffolds :building_construction:

Run the automated init script:

```sh
detox init -r jest
```
> **Note:** errors occurring in the process may appear in red.

If things go well, you should to have this set up:

- An `e2e/` folder in your project root
- An `e2e/config.json` file; [example](/examples/demo-react-native-jest/e2e/config.json)
- An `e2e/init.js` file; [example](/examples/demo-react-native-jest/e2e/init.js)
- An `e2e/firstTest.e2e.js` file with content similar to [this](/examples/demo-react-native-jest/e2e/app-hello.e2e.js).

### 3. Fix / Verify

Even if `detox init` passes well, and everything is green, we still recommend going over the checklist below. You can also use our example project, [`demo-react-native-jest`](https://github.com/wix/Detox/tree/master/examples/demo-react-native-jest), as a reference in case of ambiguities.

#### .detoxrc.json

| Property               | Value                                          | Description                                                  |
| ---------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `test-runner`    | `"jest"`                                       | *Required.* Should be `"jest"` for the proper `detox test` CLI functioning. |
| `runner-config ` | (optional path to Jest config file)            | *Optional.* This field tells `detox test` CLI where to look for Jest's config file. If omitted, the default value is `e2e/config.json`. |

A typical Detox configuration in `.detoxrc.json` file looks like:

```json
{
  "test-runner": "jest",
  "runner-config": "e2e/config.json",
  "configurations": {
    "ios.sim.release": {
      "type": "ios.simulator",
      "binaryPath": "ios/build/Build/Products/Release-iphonesimulator/example.app",
      "build": "...",
      "device": {
        "type": "iPhone 11 Pro"
      }
    }
  }
}
```

#### e2e/config.json

| Property               | Value                                          | Description                                                  |
| ---------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `testEnvironment `     | `"./environment"`               | *Required.* Needed for the proper functioning of Jest and Detox. See [Jest documentation](https://jestjs.io/docs/en/configuration#testenvironment-string) for more details. |
| `testRunner `          | `"jest-circus/runner"`                           | *Required.* Needed for the proper functioning of Jest and Detox. See [Jest documentation](https://jestjs.io/docs/en/configuration#testrunner-string) for more details.  |
| `testTimeout `          | `120000`                           | *Required*. Overrides the default timeout (5 seconds), which is usually too short to complete a single end-to-end test. |
| `reporters`            | `["detox/runners/jest/streamlineReporter"]`    | *Recommended.* Sets up our streamline replacement for [Jest's default reporter](https://jestjs.io/docs/en/configuration#reporters-array-modulename-modulename-options), which removes Jest's default buffering of `console.log()` output. That is helpful for end-to-end tests since log messages appear on the screen without any artificial delays. For more context, [read Detox 12.7.0 migration guide](Guide.Migration.md#migrating-to-1270-from-older-nonbreaking). |
| `verbose`              | `true`                                         | *Conditional.* Must be `true` if above you have replaced Jest's default reporter with Detox's `streamlineReporter`. Optional otherwise. |

A typical `jest-circus` configuration in `e2e/config.json` file would look like:

```json
{
  "testRunner": "jest-circus/runner",
  "testEnvironment": "./environment",
  "testTimeout": 120000,
  "reporters": ["detox/runners/jest/streamlineReporter"],
  "verbose": true
}
```

#### e2e/environment.js

If you are not familiar with Environment concept in Jest, you could check [their documentation](https://jestjs.io/docs/en/configuration#testenvironment-string).

For Detox, having a `CustomDetoxEnvironment` class derived from `NodeEnvironment` enables implementing cross-cutting concerns such as taking screenshots the exact moment a test function (it/test) or a hook (e.g., beforeEach) fails, skip adding tests if they have `:ios:` or `:android:` within their title, starting device log recordings before test starts and so on.

API of `CustomDetoxEnvironment` is not entirely public in a sense that there's no guide on how to write custom `DetoxCircusListeners` and override `initDetox()` and `cleanupDetox()` protected methods, since this is not likely to be needed for typical projects, but this is under consideration if there appears specific demand.

See [an example](https://github.com/wix/Detox/blob/master/examples/demo-react-native-jest/e2e/init.js) of a custom Detox environment for Jest.

```js
const {
  DetoxCircusEnvironment,
  SpecReporter,
  WorkerAssignReporter,
} = require('detox/runners/jest-circus');

class CustomDetoxEnvironment extends DetoxCircusEnvironment {
  constructor(config) {
    super(config);

    // Can be safely removed, if you are content with the default value (=300000ms)
    this.initTimeout = 300000;

    // This takes care of generating status logs on a per-spec basis. By default, Jest only reports at file-level.
    // This is strictly optional.
    this.registerListeners({
      SpecReporter,
      WorkerAssignReporter,
    });
  }
}

module.exports = CustomDetoxEnvironment;
```

**Notes:**

- The custom `SpecReporter` is recommended to be registered as a listener. It takes care of logging on a per-spec basis (i.e. when `it('...')` functions start and end) — which Jest does not do by default.
- The custom `WorkerAssignReporter` prints for every next test suite which device is assigned to its execution.

This is how a typical Jest log output looks when `SpecReporter` and `WorkerAssignReporter` are enabled in `streamline-reporter` is set up in `config.json` and
`SpecReporter` added in `e2e/environment.js`:

![Streamlined output](img/jest-guide/streamlined_logging.png)

## Writing Tests

There are some things you should notice:

- Don't worry about mocks being used, Detox works on the compiled version of your app.
- Detox exposes it's primitives (`expect`, `device`, ...) globally, it will override Jest's global `expect` object.

## Parallel Test Execution

Through Detox' CLI, Jest can be started with [multiple workers](Guide.ParallelTestExecution.md) that run tests simultaneously, e.g.:

```bash
detox test --configuration <yourConfigurationName> --workers 2
```

In this mode, Jest effectively assigns one worker per each test file.
Per-spec logging offered by the `SpecReporter` mentioned earlier, does not necessarily make sense, as the workers' outputs get mixed up.

By default, we disable `SpecReporter` in a multi-workers environment.
If you wish to force-enable it nonetheless, the [`--jest-report-specs`](APIRef.DetoxCLI.md#test) CLI option can be used with `detox test`, e.g.:

```bash
detox test --configuration <yourConfigurationName> --workers 2 --jest-report-specs
```

## How to run unit and E2E tests in the same project

- Create different Jest configs for unit and E2E tests, e.g. in `e2e/config.json` (for Detox) and `jest.config.js`
(for unit tests). For example, in Jest's E2E config you can set `testRegex` to look for `\.e2e.js$` regexp,
 and this way avoid accidental triggering of unit tests with `.test.js` extension.
- To run your E2E tests, use `detox test` command (or `npx detox test`, if you haven't installed `detox-cli`).
