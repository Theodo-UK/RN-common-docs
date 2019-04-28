# Secrets 🔒

## The problem

At the time of this writing, the list of App secrets and their location is the following:

- In `fastlane/.env.<environment>.secret`:
  - Android keystore passwords (for Android code signing)
  - Appcenter Token (for CodePush auth and upload of hard deployed bundles)
  - Codepush deployment keys
  - CodePush Public key (for CodePush code signing)
  - (if used) Appcenter App Secrets (for analytics and crash reporting)
- (if prod env is created) Google Play API auth file at `xxxxxxx/api-xxxxxx.json`
- iOS code signing identities in the `Match` repo, protected by the MATCH PASSWORD. These should only be accessed using [Fastlane Match](https://codesigning.guide/) (see [deployment 🚀](./deployment.md) docs).

As you can see, some of these secrets are keys, some others are entire files => we can't enter them all in Circle-CI's _Environment variables_ section. Also, we need a way to efficiently share them while keeping them secret.

## Solution

Solution adopted is the one suggested [here](https://support.circleci.com/hc/en-us/articles/360006717953-Storing-secret-files-certs-etc-): putting all the secrets in one encrypted archive per environment committed to the repo. This allows to have a different password for each environment.

Two scripts respectively allow to unzip all the secrets to your local environment or conversely to zip them in case you wanted to update them.

#### Unziping all the secrets from the root of the project and copying them all to the right locations in the project ("unpacking")

```sh
cd <whatever>/yourapp
git checkout master
git pull
yarn pack-secrets -e <environment> -p <secretsPasswordForThisEnvironment>
```

#### Zipping all secrets to the encrypted archive from the project root ("packing")

```sh
cd <whatever>/yourapp
git checkout master
git pull
yarn unpack-secrets -e <environment> -p <secretsPasswordForThisEnvironment>
# Do your changes in the secret files/add a secret to pack in secrets-scripts/pack_secrets.sh
yarn pack-secrets -e <environment>
# MAKE SURE YOU ENTER THE SAME PASSWORD AS BEFORE! (Unless you do this specifically to change the password)
```

> ⚠️ When packing the secrets to update them, make sure you unpack them just before, in case someone also modified them since last time you unpacked them, otherwise you will lose their changes!!!  
> ⚠️ Make sure you don't mix-up the passwords when re-packing, this is very sensitive and important!

## Notes

Circle-CI uses the unpacking script to access all the secrets. The environment variables that it has configured are the passwords to unpack, the match password to get the iOS signing identities and some API tokens.
