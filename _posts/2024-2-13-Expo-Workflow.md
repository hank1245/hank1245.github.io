---
layout: post
title: Expo Development Workflow Guide
subtitle: Managed to Bare Workflow Transition Lessons
tags: [Expo, React Native, Mobile Development]
comments: true
author: Hank Kim
thumbnail-img: /assets/img/expo.png
---

# Expo Development Workflow Guide

## The Prebuild Mistake

I accidentally overwrote my entire iOS configuration because I didn't understand when to run `expo prebuild`. If you're working with Expo and native libraries, this might help you avoid the same mistake.

It started when I was integrating Google Sign-In with Supabase authentication. What followed was learning about Expo's different development workflows and when to use each command.

## Managed vs Bare Workflow

Expo has two development approaches:

### Managed Workflow

- Everything works in Expo Go
- Use `expo install` for packages
- No access to native code
- Limited to Expo SDK packages

### Bare Workflow

- Full access to `ios/` and `android/` directories
- Can use any React Native library
- Requires building apps locally or with EAS
- More control, more complexity

## Understanding expo prebuild

The `expo prebuild` command does three things:

```bash
expo prebuild
```

1. Generates `ios/` and `android/` directories from your `app.json`
2. Transitions from Managed to Bare Workflow
3. **Overwrites any existing native code**

That third point is important. If you've manually edited files in `ios/` or `android/`, `prebuild` will replace them with generated versions.

### When to Use prebuild

**Safe to use:**

- After `expo install` of native libraries
- When changing `app.json` settings
- Initial transition to Bare Workflow

**Don't use:**

- After manually editing native iOS/Android code
- Just to "refresh" things

## Development Commands

### expo run:android / expo run:ios

```bash
expo run:android
```

This command:

1. Builds the native project (Android Studio/Xcode)
2. Installs the app on device/emulator
3. Starts Metro bundler
4. Connects everything for hot reloading

It's not just a dev server - it's a full native build.

### Development Flow

```bash
# Initial build or after native changes
expo run:android

# For JS-only changes
expo start

# Native code changed? Build again
expo run:android
```

You don't need to run `expo run:android` for every JavaScript change.

## Config Plugins

Config Plugins automatically configure native projects during `prebuild`. They modify `AndroidManifest.xml`, `Info.plist`, and other native files.

Example plugin:

```javascript
// withMyCustomConfig.js
const { withAndroidManifest, withInfoPlist } = require("@expo/config-plugins");

const withCustomAndroidManifest = (config) => {
  return withAndroidManifest(config, (config) => {
    const manifest = config.modResults;
    // Modify manifest here
    return config;
  });
};

const withCustomInfoPlist = (config) => {
  return withInfoPlist(config, (config) => {
    config.modResults.MyCustomKey = "MyCustomValue";
    return config;
  });
};

module.exports = (config) => {
  config = withCustomAndroidManifest(config);
  config = withCustomInfoPlist(config);
  return config;
};
```

Add to `app.json`:

```json
{
  "expo": {
    "plugins": ["./withMyCustomConfig.js"]
  }
}
```

Config Plugins ensure consistent native configuration every time you run `prebuild`.

## Google Sign-In Setup

Here's the actual setup process for Google OAuth:

### 1. Google Cloud Console Setup

You need three client IDs:

- Web client ID (for Supabase backend)
- iOS client ID
- Android client ID

### 2. Environment Variables

React Native can't read `.env` files directly. Install `react-native-dotenv`:

```bash
npm install react-native-dotenv
```

Configure Babel:

```javascript
// babel.config.js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    [
      "module:react-native-dotenv",
      {
        moduleName: "@env",
        path: ".env",
      },
    ],
  ],
};
```

Usage:

```javascript
import { GOOGLE_CLOUD_CLIENT_ID_IOS } from "@env";
```

### 3. GoogleSignin Configuration

```javascript
GoogleSignin.configure({
  iosClientId: "iOS-CLIENT-ID.apps.googleusercontent.com",
  webClientId: "WEB-CLIENT-ID.apps.googleusercontent.com", // iOS needs this too
});
```

iOS requires both `iosClientId` and `webClientId` for different auth flows.

### 4. iOS URL Scheme

Configure in `app.json`:

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourapp.identifier"
    },
    "scheme": "com.yourapp.identifier"
  }
}
```

This allows Google to redirect back to your app after authentication.

## Build Process

### iOS Build

```bash
cd ios
pod install  # Install CocoaPods dependencies
cd ..
expo run:ios
```

### Android Build

```bash
cd android
./gradlew clean  # Clear build cache
cd ..
expo run:android
```

The `gradlew clean` step prevents cached build issues.

## Expo Go vs Dev Client

Once you use native libraries, Expo Go stops working. You'll see "This library is not supported in Expo Go" errors.

Solution is `expo-dev-client`:

```bash
npx expo install expo-dev-client
expo run:android  # Builds custom dev client
```

Dev Client is a custom version of Expo Go that includes your native dependencies.

Comparison:

- **Expo Go**: Quick testing, limited libraries
- **Dev Client**: All libraries, requires building

## EAS Build vs Local Builds

### Local Builds

```bash
expo run:android           # Debug build
expo run:android --variant release  # Release build
```

**Pros:** Fast, full control
**Cons:** Requires local setup, platform-specific

### EAS Build

```bash
eas build --platform android
eas build --platform ios
```

**Pros:** Build iOS on any platform, consistent environment
**Cons:** Requires internet, build queues

## Common Issues

### 1. Prebuild Overwrites Native Code

**Problem:** Lost custom native modifications
**Solution:** Only use `prebuild` for config changes, not after manual native edits

### 2. OAuth Test Users

**Problem:** Google sign-in works in development but fails in production
**Solution:** Add test users in Google Cloud Console OAuth consent screen

### 3. Environment Variables Not Loading

**Problem:** Variables undefined at runtime
**Solution:** Ensure `react-native-dotenv` is properly configured in `babel.config.js`

### 4. Build Cache Issues

**Problem:** Changes not appearing in builds
**Solution:** Clean builds:

```bash
# Android
cd android && ./gradlew clean && cd ..
# iOS
cd ios && rm -rf build && cd ..
```

## Deployment

### Build for Stores

```bash
eas build --platform android --profile production
eas build --platform ios --profile production
```

### Submit to App Stores

```bash
eas submit --platform android
eas submit --platform ios
```

### Over-the-Air Updates

For JavaScript-only changes:

```bash
eas update
```

## Key Takeaways

1. **Understand your workflow**: Managed vs Bare determines which commands work
2. **Respect prebuild**: It overwrites native modifications
3. **Clean builds solve most problems**: When in doubt, clean and rebuild
4. **Dev Client replaces Expo Go**: Once you use native libraries
5. **Environment variables need setup**: React Native doesn't read `.env` automatically

The transition from Managed to Bare Workflow isn't just adding folders - you're taking on native development responsibilities. But the flexibility you gain makes it worthwhile for most real-world apps.
