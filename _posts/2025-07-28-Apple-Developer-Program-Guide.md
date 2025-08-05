---
layout: post
title: Apple Developer Program and App Store Submission Guide
subtitle: From account setup to app approval - practical tips for first-time iOS developers
tags: [iOS Development, App Store, Expo, React Native]
comments: true
author: Hank Kim
thumbnail-img: /assets/img/appstore.png
---

# Apple Developer Program and App Store Submission Guide

Submitting an app to the Apple App Store involves several steps that can trip up first-time developers. I recently went through this process and encountered various challenges along the way. Here's a practical guide based on my experience, covering everything from account setup to getting your app approved.

## Account Setup and Identity Verification

The Apple Developer Program enrollment starts straightforward but can hit unexpected snags. After updating my payment method and password, I received a call from Apple support informing me that identity verification was required. This involved:

- Submitting identification documents
- Providing my address in English
- Multiple photo submissions of my ID

The process required persistence, but eventually got approved. If you encounter identity verification, be patient and ensure all documents are clear and legible.

## Internal Distribution with Expo and EAS Build

Once my developer account was active, I tested my app using Expo and EAS Build for iOS internal distribution. The built `.ipa` file would download but wouldn't install on my device because the **Unique Device Identifier (UDID)** wasn't included in the build.

### Finding Your Device's UDID

To test on your device, you first need its UDID:

1. Connect your iPhone/iPad to your Mac via cable
2. Open Finder and select your device in the sidebar
3. Click on the text below the device name (cycles through model, serial number, then UDID)
4. Copy the UDID value

### Registering the UDID

Register your device with Apple:

1. Log in to [Apple Developer website](https://developer.apple.com/account/)
2. Navigate to **Certificates, Identifiers & Profiles** → **Devices**
3. Click the '+' button to add a new device
4. Select **iOS, tvOS, watchOS** platform
5. Paste the UDID and enter a device name
6. Click **Continue** to complete registration

### Configuring eas.json

Set up an internal distribution profile in your `eas.json`:

```json
{
  "cli": {
    "version": ">= 7.6.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "internal": {
      "distribution": "internal",
      "ios": {
        "enterpriseProvisioning": "adhoc"
      }
    },
    "production": {}
  },
  "submit": {
    "production": {}
  }
}
```

### Building and Installing

Build your app with the internal profile:

```bash
eas build --platform ios --profile internal
```

EAS will automatically create an Ad Hoc Provisioning Profile including your registered device and build the app accordingly.

## Developer Mode for iOS 16+

For iOS 16 and later, enable Developer Mode to install apps from sources other than the App Store:

1. Open **Settings** → **Privacy & Security**
2. Scroll to **Developer Mode** at the bottom
3. Toggle Developer Mode on
4. Restart your device when prompted
5. After restart, select **Turn On** and enter your passcode

## App Store Connect Preparation

### Icons and Screenshots

- **App Icons**: Provide a 1024x1024 icon - EAS Build generates all necessary sizes automatically
- **Screenshots**: Apple requires specific sizes:
  - 6.5-inch displays: 1290 x 2796 (iPhone 14 Pro Max, 15 Plus)
  - 6.9-inch displays: 1320 x 2868 (iPhone 15 Pro Max)

**Screenshot tip**: Use Xcode simulators and capture with Cmd+S. Start with the largest size (6.7-inch) as App Store Connect often allows these for smaller displays too.

### Common Requirements

Before submission, ensure you have:

- 13-inch iPad screenshots
- Privacy Policy URL
- Primary app category selected
- Content Rights Information configured
- Age Rating completed
- Support URL (can be a simple GitHub Pages or Notion page)

## Handling Rejections: Unnecessary Sign-ups

My app was rejected for forcing unnecessary user registration. Features that didn't require an account were still demanding sign-up.

### The Solution: Guest Mode + Cloud Sync

I implemented a hybrid approach:

**Step 1**: Allow guest access

- Users can use core features without signing up
- Data stored locally on device
- All basic functionality available

**Step 2**: Incentivize account creation

- "Create an account to back up your data and access it on other devices"
- Option to sync local data to cloud storage once they sign up

**Implementation Flow**:

```
App Launch → [Start as Guest] [Log In]
            ↓
    Use Local Storage
            ↓
    Later: "Back up your data?" prompt
```

This approach complies with App Store guidelines while maintaining the benefits of cloud storage for users who want it.

## Re-submission Tips

When re-submitting after rejection:

- Increment both version number and build number
- Explain modifications in the Resolution Center
- Consider using "Developer Reject" for faster processing

### Build Number Issues

If your `buildNumber` isn't updating on EAS Build, remember that EAS fetches code from your Git repository, not your local machine. Always:

```bash
git add .
git commit -m "Update build number"
git push
```

Then build:

```bash
eas build --platform ios --profile production --clear-cache
```

If issues persist, try:

```bash
expo prebuild
```

This synchronizes your `app.json` changes with native project files.

## Key Takeaways

The Apple Developer Program and App Store submission process has a learning curve, but each challenge teaches valuable lessons. The most important points:

1. **Be patient with identity verification** - it's a necessary security step
2. **Understand UDID registration** for internal testing
3. **Implement guest access** to comply with App Store guidelines
4. **Remember Git workflow** when using EAS Build
5. **Prepare all required assets** before submission

By planning for these common issues, you can navigate the submission process more smoothly and get your app approved faster.
