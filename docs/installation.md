# Installation 🔨

## Requirements

#### TL;DR

- Computer with macOS (for running iOS simulator)
- Xcode [setup for React Native](https://facebook.github.io/react-native/docs/getting-started#installing-dependencies)
- Android Studio [setup for React Native](https://facebook.github.io/react-native/docs/getting-started#installing-dependencies-1)
- `node`
- `yarn`

#### How to get these requirements properly setup?

- Download and install both Xcode and Android Studio.
- [Remove all versions of `node` from your computer](https://stackabuse.com/how-to-uninstall-node-js-from-mac-osx/).
- [Install nvm properly](https://github.com/creationix/nvm).
- Make node 10 your default version (or at least `nvm use` node 10 in the project's folder).

  ```sh
  nvm install 10
  nvm alias default 10
  ```

- Install `yarn` to work properly with `nvm`.
  ```sh
  brew install yarn --without-node
  ```
- Install `react-native-cli`.
  ```sh
  yarn global add react-native-cli
  ```
- Follow the official [iOS](https://facebook.github.io/react-native/docs/getting-started#installing-dependencies) and [Android](https://facebook.github.io/react-native/docs/getting-started#installing-dependencies-1) RN setup tutorials. Stop where it asks you to run `react-native init AwesomeProject`.

  ⚠️ ️️️Make sure your SDK path is exported in your env: `export ANDROID_HOME=~/Library/Android/sdk`  
  ⚠️ If you have another version of Java running by default, you will have to change the specific version you want to use:

  - `/usr/libexec/java_home -V` to check the version you have
  - Run in your terminal: export JAVA_HOME=`/usr/libexec/java_home -v <the java version you want to use>`
  - You should also add this line to your `bash` or `zsh` config

## Project setup

- Clone this repo
  ```sh
  git clone git@<your-repo>.git
  cd <your-repo>
  ```
- Install JS dependencies.
  ```sh
  yarn
  ```
- If there is a dev environment, unpack the dev app secrets.
  ```sh
  brew install gpg
  ./unpack_secrets.sh -e dev -p <archivePasswordForDev>
  ```
- Install React Native Debugger.
  ```sh
  brew cask install react-native-debugger
  ```
