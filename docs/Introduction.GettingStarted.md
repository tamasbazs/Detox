# Getting Started

**Welcome to Detox! :tada:**

In this guide, we will walk you through setting Detox up in your project, one step at a time.

You will find that some steps are longer than the others: some are just one-paragraph long, while for others we have a dedicated multistep guide worked out. Bear with us - it is all necessary, and once set-up, it is easy to move forward with writing test very rapidly.

## Step 1: Environment Setup

### Install [Node.js](https://nodejs.org/en/)

`Node.js` is the JavaScript runtime Detox will run on. **Install Node `8.3.0` or above**.

There's more than one way to install node.js:

1. Download from the [official download page](https://nodejs.org/en/download/)
2. Use [homebrew](https://formulae.brew.sh/formula/node)
3. Use `nvm` - if you need to allow for several versions to be installed on a single machine

We recommened using homebrew:

 ```sh
brew update && brew install node
 ```

> Tip: Verify installation succeeded by typing in `node -v` in the terminal to output current node version. Should be `8.3.0` or higher.

### Install Detox command line tools (`detox-cli`)

This package makes it easier to operate Detox from the command line. `detox-cli` should be installed globally, enabling usage of the command line tools outside of your npm scripts. `detox-cli` is merely a script that passes commands through to a the command line tool shipped inside `detox` package (in `node_modules/.bin/detox`).

  ```sh
npm install -g detox-cli
  ```

### Install platform-specific dependencies, tools and dev-kits

Depending on the platform/s you're aiming at (iOS, Android), take the time to run through these environment setup guides:

* [Android](Introduction.AndroidDevEnv.md)
* [iOS](Introduction.IosDevEnv.md)

## Step 2: Add Detox to your project

### Install the Detox node-module

If you have a React Native project, go to its root folder (where `package.json` is found) and type the following command:

```sh
npm install detox --save-dev
```

If you have a project without Node integration (such as a native project), add the following `package.json` file to the root folder of your project:

```json
{
  "name": "<your_project_name>",
  "version": "0.0.1"
}
```

Name your project in `package.json` and then run the following command:

```sh
npm install detox --save-dev --no-package-lock
```

**You should now have Detox available in `node_modules/detox`**

> Tip: Remember to add the `node_modules` folder to your git-ignore file (e.g. `.gitignore`).

### Set Up a Test Runner :running_man:

Detox delegates the actual Javascript test-code execution to a dedicated test-runner. It supports the popular `Jest` and `Mocha` out of the box. You need to choose and set up one of them now, but it *is* possible to switch later on, should you change your mind.

* **[Jest](https://jestjs.io/) is the recommended choice**, since it provides parallel test execution and a complete lifecycle integration for Detox. To set up, follow [our comprehensive guide for Jest](Guide.Jest.md).
* [Mocha](https://mochajs.org/), albeit its integration is less complete, is still lightweight, and a bit easier to set up. To set up, follow [our guide for Mocha](Guide.Mocha.md).

> **Note:** Detox is coupled to neither Mocha or Jest, nor with a specific directory structure. Both runners are just a recommendation — with some effort, they can be replaced without touching the internal implementation of Detox itself.

### Apply Detox Configuration

If you've completed the test-runner setup successfully using `detox init`, you should have a `.detoxrc.json` file containing a skeletal configuration for Detox to use. This configuration is only half-baked and needs to be set up properly. You now need to either create or edit that file, and apply the actual configuration suitable for your specific project.

*Note that Detox scans for a configuration through multiple files. It starts from the current working directory, and runs over the following options, in this order:*

1. `.detoxrc.js`
1. `.detoxrc.json`
1. `.detoxrc`
1. `detox.config.js`
1. `detox.config.json`
1. `package.json` (`"detox"` section)

*If you prefer to use something other than `.detoxrc.json` - for example, would like to keep all project configs in one place, you can create a `detox` section in your`package.json`. If you otherwise prefer separating configs, all of the other options are valid.*

Depending on the platform/s you're aiming at (iOS, Android), for a complete, platform-specific configuration explanation (and a bunch of other platform-specific things, where applicable), refer to the dedicated guides:

* [Android](Introduction.Android.md)
* [iOS](Introduction.Ios.md)

## Step 3: Build your app and run Detox tests

#### 1. Build your app

Use a convenience method in Detox command line tools to build your project easily:

```sh
detox build
```

> Tip: Notice that the actual build command was specified in the Detox configuration (e.g. in `.detoxrc.json` or `package.json` ).
>
> See `"build": "xcodebuild -project ..."` inside `ios.sim.debug`, or `./gradlew assembleDebug assembleAndroidTest` in `android.emu.debug`.

#### 2. Run the tests (finally!) :beers:

Use the Detox command line tools to test your project easily:

```sh
detox test
```

That's it. Your first failing Detox test is running!

Next, we'll go over usage and how to make this test [actually pass](Introduction.WritingFirstTest.md).


