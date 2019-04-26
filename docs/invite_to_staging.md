# [Inviting new people to the Staging and Beta Apps 👪](./docs/invite_to_staging.md)

There are two apps that can be installed on a given device:

- **Staging** obtained from Appcenter at https://install.appcenter.ms
- **Production** obtained from either:
  - Beta release platform (TestFlight for iOS app, Google Play Alpha release for Android app)
  - Official store (Apple App Store for iOS app, Google Play Store for Android app)

Here is how to get the Staging and Beta apps installed on a device.

## Staging App

### iOS 🍏

**Pre-requisites:**

- Manager account for the app on [Appcenter](https://appcenter.ms)
- Admin account on the [Apple developer portal](https://developer.apple.com/account/).
- Fastlane setup on your computer (see "Setup" [here](./deployment.md). You don't need the "unpack secrets" part).

**Procedure**:

To make the iOS staging app accessible to a new person on a new device, do the following:

- Visit Appcenter iOS "Collaborators" distribution group page.
- Add email of the person to add.
- Ask the person to do as instructed in the email. The most usual way to login is using their Google account.
- Now that the person subscribed to Appcenter and the app, in "Collaborators", go to the _Devices_ section, click on the three dots at the right of the person you just added and copy their _device UDID_.
- Open your app's repo in your IDE, open `ios_devices.txt`.
- **Making sure you respect the spaces and indentations** replace the example lines with your line, adding UDID, name and device of the person.  
  ⚠️ If you try to re-create the indentation yourself, the file will be badly formatted and the command will fail (Apple uses a weird tab format for their file).  
  ⚠️ Don't commit anything: this modification is temporary to add the person. Actual people are not commited in `ios_devices.txt` because the command below can only add people, not remove: it is not the full list of authorised people.
- Run the following command from the root of the project. Enter your Apple ID credentials when prompted (should be admin in dev portal).
  ```sh
  bundle exec fastlane ios add_devices --env=staging
  ```
- Hard deploy staging (simplest is to re-trigger the latest `staging` build on Circle-Ci).

Voilà! After the build succeeds, the person added should be able to download the latest version of the app at https://install.appcenter.ms

**Note:** you can add several people at once in `ios_devices.txt`

**How does it work?**

Apple requires that only a [list of devices defined on their portal](https://developer.apple.com/account/ios/device/) is allowed to install test apps.  
The command above adds the new device to the list of authorized devices in the Apple Developer portal, then automatically generates a new provisioning profile containing that device and uploads it to the `Match` repo.
Next time the app is deployed with Fastlane, it will be built with this new provisioning profile thus authorizing the new person to install it. The easiest way to do that is re-triggering the latest build on Circle-CI.

### Android 🤖

**Pre-requisites:**

- Manager account for the app [on Appcenter](https://appcenter.ms).

**Procedure:**

- Visit Appcenter Android "Collaborators" distribution group page.
- Add email of the person to add.
- Ask the person to do as instructed in the email. The most usual way to login is using their Google account.

The person added should now be able to download the latest version of the app at https://install.appcenter.ms/

## Beta App

### iOS 🍏

**Pre-requisites:**

- App Manager account for your app on [App Store Connect](https://appstoreconnect.apple.com/access/users)

**Procedure:**

- Visit TestFlight
- In the left panel, click on an "External testers" group
- Either share the public link with the person or add them with the +.
- Ask them to do as instructed in the email or link.

⚠️ Check that the group you have put the person in has been allocated the latest build (in **Builds**. This should be automatically done by Circle-CI), otherwise do it.

### Android 🤖

**Pre-requisites:**

- Account for your app on [Google Play console](https://play.google.com/apps/publish/)

**Procedure:**

- Visit Google Play Console Internal track
- Click the arrow on the right of _Manage Testers_ and manage them there.
- Give the _Opt-in URL_ to people you have added to the testers. They will need a Google account.
- Ask them to do as instructed in the link.
