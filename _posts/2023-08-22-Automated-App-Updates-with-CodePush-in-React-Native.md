---
layout: post
title: Automated App Updates with CodePush in React Native
subtitle: Skip the App Store Review Process
tags: [React Native, Mobile Development, DevOps]
comments: true
author: Hank Kim
---

# Automated App Updates with CodePush in React Native

I remember the frustration of finding a critical bug in production and knowing it would take 2-3 days minimum to get a fix through the App Store review process. Sometimes it was even worse - Apple would reject the update for minor issues, adding another few days to the timeline. That's when I discovered CodePush, and it completely changed how I think about mobile app updates.

CodePush allows you to push JavaScript updates directly to users' devices without going through app store reviews. It's been a game-changer for the React Native apps I've worked on, especially when dealing with urgent bug fixes or small feature updates.

## Understanding the Update Scenarios

When working with CodePush, it's important to understand what types of changes can be updated and how:

### Scenario 1: JavaScript and Asset Changes (The Sweet Spot)

This is where CodePush really shines. When I modify React Native JavaScript code or update assets like images, CodePush can push these changes directly to users' devices. However, I learned an important lesson the hard way: **the updated code won't take effect until the app is restarted**.

I spent hours debugging why my updates weren't showing up, only to realize that React Native's JavaScript runtime keeps the compiled code cached in memory. The updates are downloaded successfully, but they only become active when the JavaScript runtime restarts.

This led me to explore two approaches:
- **Passive approach**: Let users discover the update naturally when they next open the app
- **Active approach**: Use CodePush's `restartApp()` function to force an immediate restart

I've used both depending on the urgency of the update. For critical bug fixes, I use the active approach. For feature updates, I usually go with the passive approach to avoid interrupting the user experience.

### Scenario 2: Native Code Changes (Back to the App Store)

Here's where I hit the limitations of CodePush. When I needed to modify native code - whether it was updating a native module, changing iOS/Android permissions, or adding new native dependencies - CodePush couldn't help me.

I learned this during a project where I needed to add camera permissions. Even though the change seemed small, it required modifying the native Android and iOS code. This meant:
- Going back to the traditional app store submission process
- Waiting for review approval
- Hoping users would actually update their apps

The `restartApp()` function I mentioned earlier only restarts the JavaScript runtime, not the entire native application. So for native changes, you're back to square one with app store distribution.

## What is CodePush?

CodePush is an open-source service from Microsoft that allows React Native and Apache Cordova developers to deploy updates instantly without going through app store approval processes.

The service works by:
1. Uploading JavaScript bundles and assets to App Center (Microsoft's central repository)
2. Mobile clients checking for updates when the app starts
3. Automatically downloading and synchronizing new versions

**Note**: App Center is Microsoft's comprehensive platform that provides CodePush functionality along with monitoring, reporting, and other mobile development tools.

## Important Limitations

CodePush can only manage changes in JavaScript code. Any modifications related to native code must be distributed through traditional app store channels.

## My Implementation Journey

Let me walk you through exactly how I set up CodePush for one of my React Native projects. I'll share the gotchas I encountered and how to avoid them.

### Step 1: Install App Center CLI
```bash
npm install -g appcenter-cli
```

**Note from experience**: Make sure you have the latest version of Node.js. I ran into compatibility issues with an older Node version that took me a while to debug.

### Step 2: Create App Center Account and Register App

This step is crucial and took me longer than expected. You need to:
1. Create an account on [App Center](https://appcenter.ms/)
2. Create separate apps for iOS and Android (yes, they need to be separate!)
3. Get your deployment keys for both staging and production environments

**Pro tip**: Write down your deployment keys immediately. You'll need them for the SDK configuration, and they're not always easy to find in the UI later.

### Step 3: Install CodePush SDK
In your app's root directory, install the CodePush SDK:
```bash
npm install --save react-native-code-push
```

### Step 4: Integrate CodePush SDK
Wrap your root component with the CodePush HOC (Higher-Order Component) to enable the app to check for and download updates from the CodePush server:

```javascript
import React from "react";
import CodePush from "react-native-code-push";
import { Provider } from 'react-redux';
import { store } from "./src/store";
import RootNavigator from "./src/navigator/RootNavigator";

const App = () => (
  <Provider store={store}>
    <RootNavigator />
  </Provider>
);

export default CodePush(App);
```

### Step 5: Deploy Updates
Use the App Center CLI to upload new updates to the CodePush server:

```bash
appcenter codepush release-react -a {username}/{app-name} -d {deployment}
```

To force an update (users cannot skip it), add the `-m` option:
```bash
appcenter codepush release-react -a {username}/{app-name} -d {deployment} -m
```

## Advanced: Programmatic App Restart

The `codePush.restartApp()` function allows you to restart the app programmatically when updates are available:

```javascript
codePush.restartApp(onlyIfUpdateIsPending: Boolean = false): void;
```

When called with `true`, the app only restarts if there's a pending update. This is particularly useful for implementing different update strategies:

### Strategy 1: Immediate Updates
Install updates using `LocalPackage.install()` with `sync` mode or `ON_NEXT_RESUME`, then use `ON_NEXT_RESTART` for immediate effect. This approach is best for critical updates but may interrupt the user experience.

### Strategy 2: User-Controlled Updates
Present users with an update notification, allowing them to choose when to apply the update. This provides better user experience but may delay the rollout of important fixes.

## Example Implementation

Here's a complete example of how to implement CodePush with user-controlled updates:

```javascript
import React, { useEffect, useState } from 'react';
import { Alert } from 'react-native';
import CodePush from 'react-native-code-push';

const App = () => {
  const [updateAvailable, setUpdateAvailable] = useState(false);

  useEffect(() => {
    CodePush.checkForUpdate().then((update) => {
      if (update) {
        setUpdateAvailable(true);
        Alert.alert(
          'Update Available',
          'A new version is available. Would you like to update?',
          [
            { text: 'Later', style: 'cancel' },
            { 
              text: 'Update', 
              onPress: () => {
                CodePush.sync({
                  installMode: CodePush.InstallMode.IMMEDIATE
                });
              }
            }
          ]
        );
      }
    });
  }, []);

  // Your app components here
  return (
    // Your app UI
  );
};

export default CodePush(App);
```

## Update Process Flow

1. **Upload**: Developer uploads new JavaScript bundle and assets to CodePush server
2. **Check**: App automatically checks for updates when started or brought to foreground
3. **Download**: If updates are available, they're downloaded in the background
4. **Apply**: Updates are applied either on next app start or immediately (depending on configuration)

## Best Practices

### Update Strategies
- **Critical fixes**: Use mandatory updates (`-m` flag) for security fixes
- **Feature updates**: Allow users to choose when to update
- **Gradual rollout**: Deploy to a subset of users first to catch issues

### Testing
- Always test CodePush updates on development builds before production
- Have a rollback strategy in case updates cause issues
- Monitor app performance after updates

### User Experience
- Provide clear communication about updates
- Don't force updates during critical user tasks
- Consider app usage patterns when scheduling updates

## Conclusion

CodePush revolutionizes the React Native development workflow by enabling instant updates without app store dependencies. While it has limitations around native code changes, it's incredibly powerful for JavaScript and asset updates.

The key to successful CodePush implementation is understanding when and how to restart the app, choosing appropriate update strategies, and maintaining good communication with users about the update process. When implemented thoughtfully, CodePush can significantly improve your app's maintainability and user experience.