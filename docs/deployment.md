# Deployment 🚀

## Manual - understanding the commands

We use `Fastlane` + `CodePush` to build and deploy the app in a reproducible, automated and environment-specific way.

### Definitions

There are two types of deployment:

- **Hard deploy**: deployment that builds an archive with both the `native` and `js` code and then uploads it to _AppCenter/Beta App Store/Production App store_ (see [here](./invite_to_staging.md) to know what these are). Getting the content of this deployment on a phone requires the user to update the app from https://install.appcenter.ms or the corresponding App Store.

- **Soft deploy**: deployment that only builds the `js` code and publishes it to the _CodePush_ server. Any user opening the app with a _Version Number_ matching the target of the CodePushed bundle will immediately receive the update to their phone. Getting this update doesn't require any action from the user.

**Environment**: `dev` (if setup), `staging` or `production`. Configuration of the app that matches the purpose of its deployment. For example, the iOS production app will be deployed to Test Flight and App Store with production code signing and the keys corresponding to (if setup) Firebase production, Appcenter production, CodePush production etc.

**Platform**: iOS or Android.

=> There is one app per environment and per platform.

### TL;DR

```sh
# Deployment command. If -o is not specified, apps on both platforms will be deployed.
yarn deploy -t <hard/soft> -e <staging/production> -o <ios/android>


############
# Examples #
############

# Build both Native apps + js code and deploy to both Appcenter staging apps
yarn deploy -t hard # `-e staging` not necessary because staging env is default

# Same but Android only
yarn deploy -t hard -o android

# Same but iOS only
yarn deploy -t hard -o ios

# Build both Native apps + js code and deploy to Appcenter production apps AND TestFlight/Google PlayStore Internal.
yarn deploy -t hard -e production

# Build js code only and deploy with CodePush to both staging apps
yarn deploy # (-e staging -t soft are default)

# Same Android only
yarn deploy -o android

# Same iOS only and targeting production app
yarn deploy -o ios -e production
```

### Setup

- Install Fastlane.

  ```sh
  sudo gem install fastlane -NV
  ```

- Install Fastlane project's Ruby dependencies.
  ```sh
  bundle install
  ```
- Make sure you have _Developer_ permission on the Appcenter App that you need to deploy to.
- Install Appcenter CLI and login.
  ```sh
  yarn global add appcenter-cli
  appcenter login
  ```
- Unpack the secrets corresponding to the environment(s) you will deploy to (see [Secrets 🔒](./secrets.md) for details).
  ```sh
  brew install gpg
  ./unpack_secrets.sh -e <targetEnvironment> -p <archivePasswordForTheEnvironment>
  ```

=> You can now run any of the deployment commands (see "TL;DR;" section above).

### Debug mode

For debugging purposes, you can add the option `-d <whatever>` to any of the above deployment commands. This will skip all the "code cleanness" checks, allowing to quickly deploy any code with no obstacles.

### Deployment script

The TL;DR; commands are all run by [`deploy.sh`](../deploy.sh).
The script starts by doing some sanity checks on the code (no un-commited stuff, branch checked-out corresponds to environment) and then does hard or soft deploy on the correct platform and environment.

### Hard deploy with Fastlane

Fastlane automates building, signing and deploying the app to the correct location with a per-environment and per-platform config.  
The different steps run by Fastlane for build, code signing and deployment are written in [`fastlane/Fastfile`](../fastlane/Fastfile).

The `deploy` lane does the following:

**iOS 🍏**

- Change build number to current UNIX timestamp
- Get latest code signing identity by running `match`, in case someone changed the iOS signing config of the app.
- Replace `environment/index.js` with `environment/index.<environment>.js` to set the correct environment variables in JS code.
- Update following values in native code config from `fastlane/.env`, `fastlane/.env.<environment>` and `fastlane/.env.<environment>.secret`:
  - App name
  - Version number
  - Build number
  - Codepush deployment/public key
  - (if used) Appcenter analytics + crashes app secret
- Build and sign the app for this environment.
- Deploy to Appcenter with correct symbols and source maps (and Test Flight as well if `production` build).

**Android 🤖**:

- Change build number to current UNIX timestamp
- Replace `environment/index.js` with `environment/index.<environment>.js` to set the correct environment variables in JS code.
- Update following values in native code config from `.env`, `fastlane/.env.<environment>` and `fastlane/.env.<environment>.secret`:
  - App name
  - Version number
  - Build number
  - Codepush deployment/public key
  - (if used) Appcenter analytics + crashes app secret
- Build and sign the app for this environment.
- Deploy to Appcenter (and Google Play Store Alpha as well if `production` build).

### Zoom on Code signing

**iOS 🍏**  
Code signing makes sure that you are allowed to release the iOS app as a staging or production app. It also makes sure that the app doesn't contain any feature that the signing doesn't allow.  
_For example, you can't add Apple Pay in the app if your code signing isn't configured to allow it. This enables Apple to enforce some Apple Pay-specific restrictions._

iOS code signing is composed of 2 parts:

- correct certificate's private key (certificates are handled by [Apple Developer Portal](https://developer.apple.com/account) admins and make sure that the person is authorised to publish on behalf of your company)
- correct provisioning profile, also created in [Dev Portal](https://developer.apple.com/account) which itself contains information on:
  - App ID: which app the provisioning profile is allowed to deploy
  - App entitlements (whether app is allowed to have Apple Pay, Push notifications, Deep linking etc.)
  - Deployment type: App Store (for production) or Ad Hoc (for staging).
  - (Ad Hoc app only) List of authorised devices. Ad Hoc apps can only be installed on individual devices manually authorised in Apple Dev Portal.
  - Certificate: what specific certificate's private key the deployer needs to have.

Every time one of the above changes, the provisioning profile has to be re-generated and the app re-built with the new provisioning profile in order to get the change live.

As you can see, iOS code signing is tricky and changes often, so sharing signing identities between the team members and the CI server is very cumbersome. Fortunately, [Fastlane Match](https://codesigning.guide/) enables us to automate that easily.  
In a nutshell: every time a code signing change is required, this change is done, saved and shared automatically by Fastlane.

How it works:

- someone with admin access to the Apple dev portal runs `bundle exec ios <setup/add_devices> --env=<environment>`.
- the new provisioning profile (or certificate) is automatically generated in the Apple dev portal and the old one deleted.
- they are then encrypted with the MATCH PASSWORD (see [Secrets 🔒](./secrets.md) ) and uploaded to the Match repo.
- from now on, everytime the app is built by anything using Fastlane (whether it's a dev or Circle-CI), the new signing identity will be downloaded from the `Match` repo and used.

**Android 🤖**  
Code signing makes sure that you are allowed to release the Android app as a staging or production app.

Android Code signing is composed of one keystore file per environment located at: `android/app/yourapp.<environment>.keystore`.  
You need to unpack secrets to get that file in the right location.

> ⚠️ If you lose the production keystore file or its password (contained in `fastlane/.env.production.secret`), you won't be able to publish the Android App to production ever again. You can't delete the app either.  
> => You would need to create a new app in Google Play with a new bundle identifier (since `com.yourapp.production` is taken).

### Soft-deployment with CodePush

For configuration, see [Details 🤓](./details.md).  
CodePush and its parameters are called in [`deploy.sh`](./deploy.sh):

- `--target-binary-version` makes sure that only users with the matching _Version Number_ will receive the update. Currently, this is configured so that users with a version **equal** to `<IOS_VERSION/ANDROID_VERSION_NAME>` found in `fastlane/.env` will get the update.
- `-m` specifies if the update should be _mandatory_ or not. Mandatory means customers will get the update on their next start/restore of the app at the expense of seeing a flashing and their app restart (this interrupts their experience).

### Why hard and soft deploy?

Ideally, all users would immediately download the latest version of the hard deployed app when it's published so we would only do hard deploys.  
This doesn't happen because:

- Apple and Google do a review of the hard deployed app before it can get on their store, meaning that there is a delay (up to 3 days with Apple) before it is available for the users to download.
- Even once the app is published, users don't necessarily update their app immediately.

Soft deploy (CodePush) overcomes these two problems with the following two limitations:

- it can only be used if the changes published are `js` only
- update will be either:
  - _mandatory_: users will see their app flash and restart next time they open it, slightly interrupting their experience.
  - _normal_: users will get the update only after their app has stayed at least 2 minutes in the background (or closed).

If you CodePush `js` code targeting a _Version Number_ that doesn't contain the latest native code, you might cause the app to crash as the pushed `js` code will invoke native code that doesn't exist in that app's version (note: fortunately in that case, CodePush will rollback the update automatically).

### How should a deployment happen with the current setup?

#### Production

Every time you decide to do a deployment, the code should be:

- first, soft deployed **if there are no native code change**, so that current users get the update as soon as possible.  
  If the deployment is a super important bugfix the soft deployment can be made `mandatory` in the Codepush Appcenter portals so that customers get the update immediately.
- then hard deployed so that, once the app is approved by Apple/Google, new customers get the very latest code from the beginning without having to wait for the CodePush after they launch the app for the first time.

_Note_: If the Codepush contains the same `js` as the hard deployed bundle on the phone, the user won't see any CodePush i.e. no interruption of experience.

#### Staging

Same thing except that there are no checks by Apple/Google and that the CodePush doesn't take 2 minutes of inactivity to get to the user's phone.

### How should version bumping be done?

There are 2 types of version bumping:

- _Version Number_ (`IOS_VERSION/ANDROID_VERSION_NAME` in `fastlane/.env`): version number given to the native + js bundle if hard deploy. This is the version number that customers will see in the App Store.

  To be bumped every time the App is submitted to Apple and Google. The easiest way to get that is to bump to a new value just after doing it and commit.

These processes can be automated to avoid human errors and save time. See Circle-CI section below.

## Automatic - How Circle-CI uses these commands to test, build & deploy on every push.

### Useful: https://circleci.com/circleci-react/

### Config

- Steps: `.circleci/config.yml`.  
  Start with `workflows` section at the bottom:
  - a `node` job is triggered that tests JS code and runs CodePush if `staging` or `production` branch.
  - if that job passes then both an `android` and an `ios` jobs are triggered that build the native code properly and release the app to AppCenter/TestFlight/Google Play Alpha.
- Hooks on github (or other) should prevent merrges if CircleCI builds didn't succeed

## Process to release

### Production

- Create a PR from `master` to production.
- In the PR, in `fastlane/.env`, make sure that the _Version Numbers_ are bumped compared to latest release.
  If not, get that fixed in `master` first.
- Get the PR reviewed and approved.
- Merge the PR (you need to be admin). Wait for all Circle-CI jobs to succeed. Your features have been CodePushed by now.
- Review the App in Test Flight / Google Play Store Alpha.
- Publish it by completing the last manual, in online console steps.

### Staging

- Merge your code in `staging` branch.  
  The code will be CodePushed and hard deployed to Appcenter.
