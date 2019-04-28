# Details on the setup 🤓

## Appcenter CodePush (Instant release) 🔫

- CodePush deployment key per app per environment **and** CodePush public key per environment entered in platform-specific configuration files:

  - `ios/yourapp/Info.plist`
  - `android/app/src/main/res/values/strings.xml`

  The keys are entered in the fastlane environment secrets file (`fastlane/.env.<env-name>.secret`) and set in the platform-specific config files per environment when doing a hard deploy.  
  For details see lanes `set_keys` and `ios:build` in the [Fastfile](../fastlane/Fastfile).

  > ⚠️ Don't ever commit one of these two files with an actual staging/production key in it! Only `Fastlane` should be responsible for setting the correct keys here, otherwise our secrets might be unveiled.

- CodePush entry point and config in `src/index.js`

To use Codepush, see [Deployment 🚀](./deployment.md).

## (If used) Firebase (Analytics & Push Notifications) 🔥

Firebase is installed using [React Native Firebase](https://rnfirebase.io/docs/v5.x.x/getting-started).

Two types of secret files constitute the Firebase setup. Both files are contained in the secrets archives (see [Secrets 🔒](./secrets.md)):

- Firebase common configuration file (per platform):

  - `ios/GoogleService-Info.plist` (one file per environment)
  - `android/app/google-services.json` (one file for all environments)

  Can be downloaded from the Firebase Portal

  > ⚠️ Since the iOS file is per environment, the iOS build runs a special step to select the correct environment's file on every build.  
  > To see the content of this step, see _yourapp > Build Phases > Select GoogleService-Info.plist + Clean GoogleService-Info.plist_ in Xcode.  
  > Source: ["Setup Firebase on iOS/Android with multiple environments"](https://github.com/bamlab/dev-standards/blob/master/react-native/setup/setup_firebase_multiple_envs.mo.md).

- Apple Push Notifications backend key (not contained in the app bundle):
  one for both apps, uploaded to Firebase Console. We could use a different one for each app to separate prod and staging secrets even more.  
  Keys can be re-generated in [Apple Developer Portal](https://developer.apple.com/account/ios/authkey/) and re-uploaded to Firebase anytime.

Push notifications are sent through the Firebase Console.  
Analytics can be seen in the Firebase Console, don't forget to filter to get the results for the app that interests you.

## (If used) Appcenter Crashes (App monitoring) 💥

One secret file per platform that contains the corresponding "App Secret". There is one App Secret per platform and per environment.

- `ios/yourapp/AppCenter-Config.plist`
- `android/app/src/main/assets/appcenter-config.json`

The correct keys are entered in the fastlane environment secrets file (`fastlane/.env.<env-name>.secret`) and set in the Appcenter secrest file per environment when doing a hard deploy.  
For details see lanes `set_keys` and `ios:build` in the [Fastfile](../fastlane/Fastfile).

> ⚠️ Don't ever commit one of these two files with an actual staging/production key in it! Only `Fastlane` should be responsible for setting the correct keys here, otherwise our secrets might be unveiled.

To monitor crashes, visit AppCenter Diagnostics
