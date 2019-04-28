# Development 💻

## Running the application

To have the RN `js` packager run separately and controlled by you, run it in a separate terminal.

```sh
yarn start
```

### Simulator

#### iOS 🍏

```sh
# Specifying simulator is optional. Replace by whatever device you want to use.
react-native run-ios --simulator="iPhone 8"
```

#### Android 🤖

- Open an Android simulator (can be done through Android Studio).
- Run the building command:
  ```sh
  react-native run-android
  ```

### On Device

#### iOS 🍏

- Pre-requisites:
  - Fastlane configured on your local environment (see "Setup" section [here](./deployment.md))
  - Your device [has been added for access to the staging app](./invite_to_staging.md)
  - `match` password
- Run the following command to get the latest signing identity for running on device:
  ```sh
  # Enter the match password when asked.
  bundle exec fastlane ios get_certificates_and_profiles --env=<e.g. dev or staging>
  ```
- Open the project in Xcode (`ios/yourapp.xcworkspace`).
- Under code signing, select the `match Development` provisioning profile that works (debug or release depending on what you want to do).
- Select your physical device on the top left corner next to the play button and then click the play button.

> ⚠️ For React Native debugging to work, you will need your computer and phone to be on the same wifi network.

#### Android 🤖

With your device connected and no simulator turned on, run

```sh
react-native run-android
```

## Debugging features/hot reloading/style inspection etc.

- iOS Simulator: cmd + D
- Android Simulator: cmd + M
- Devices: shake

=> Most useful button is "Debug JS remotely".  
=> **To debug network calls**, right click anywhere in the React Native Debugger and click _Enable Network Inspect_ from the menu that appears.

## List of useful VSCode extensions

- `ESLint`
- `Prettier`
- `TSLint`
- `Atom keymap` for Atom lovers
- `Color highlight`
- `DotENV`
- `GitLens`
- `VS Live Share`

Make sure you have `formatOnSave` option enabled in your user settings.

### Commands/tools useful to unblock yourself if the app doesn't build/doesn't work in simulator/RN packager shows errors

- Make sure you have Xcode > 10.1
- Start RN packager with no cache.
  ```sh
  yarn start --reset-cache
  ```
- Reset watchman.
  ```sh
  watchman watch-del-all
  ```
- Re-build.
  ```sh
  react-native run-<ios/android>
  ```
- In the simulator menus, click _Hardware -> Erase all contents and settings_. Then re-build.
- In Xcode, clean (cmd + shift + k). Then re-build.
- Re-run `yarn`

- Delete your node_modules and run `yarn`.

  ```sh
  rm -rf node_modules
  yarn
  ```

- Make sure you unpacked the dev/staging app secrets.
  ```sh
  ./unpack_secrets.sh -e dev -p <stagingSecretsPassword>
  ```
- Delete iOS build folder
  ```sh
  rm -rf ios/build
  ```
- Make sure Pods are correctly installed
  ```sh
  cd ios && pod install
  ```

Often, you will need a combination of the above to make the thing work again.  
Below is the most complete combination for iOS, in order:

```sh
### 1. Erase all contents and settings in your simulator

### 2. Make sure you have Xcode > 10.1 and run Cmd + shift + k in Xcode

### 2. First terminal window
rm -rf node_modules
yarn
watchman watch-del-all
yarn start --reset-cache

### 3. Second terminal window
cd ios && pod install && cd ..
rm -rf ios/build
react-native run-ios --simulator="iPhone 8"
```
