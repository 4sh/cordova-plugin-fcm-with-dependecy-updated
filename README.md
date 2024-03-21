# Google Firebase Cloud Messaging Cordova Push Plugin
> Extremely easy plug&play push notification plugin for Cordova applications with Google Firebase FCM.

Forked from https://github.com/andrehtissot/cordova-plugin-fcm-with-dependecy-updated which is no longer maintained.

[How it works](#how-it-works) | [Installation](#installation) | [Push Payload Configuration](#push-payload-configuration) |  [Features](#features)

## How it works
Send a push notification to a single device or topic.

- Application is in foreground:

  - The notification data is received in the JavaScript callback without notification bar message (this is the normal behavior of mobile push notifications).
  - For Android, to show the notification received on the foreground, it's recommended to use [cordova-plugin-local-notification](https://github.com/katzer/cordova-plugin-local-notifications#readme) as it provides many presentation and interaction features.

- Application is in background or closed:

  - The device displays the notification message in the device notification bar.
  - If the user taps the notification, the application comes to foreground and the notification data is received in the JavaScript callback.
  - If the user does not tap the notification but opens the application, nothing happens until the notification is tapped.
  
### Push Notifications on iOS

On Android, push notifications can be tested from emulators freely.

Unfortunately, Apple is not as nice to work with, requiring:
- The running device to be a real device, no simulators allowed;
- Application has require the `UIBackgroundModes=[remote-notification]` permission (automatically configured by this plugin);
- The user running the application has to manually allow the application to receive push notifications;
- The application must be build with credentials created from a paid account (or team) that is allowed to receive push notifications;
- The build installed has to have come from either Apple Store or TestFlight; Or, be build with a special certificate (https://customersupport.doubledutch.me/hc/en-us/articles/229495568-iOS-How-to-Create-a-Push-Notification-Certificate)

## Installation

Make sure you have ‘google-services.json’ for Android and/or ‘GoogleService-Info.plist’ for iOS in your Cordova project root folder.

#### Preferences

|Preference| Default Value       | Description                                                                                    |
|---|---------------------|------------------------------------------------------------------------------------------------|
|ANDROID_DEFAULT_NOTIFICATION_ICON| @mipmap/ic_launcher | Default notification icon.                                                                     |
|ANDROID_FCM_VERSION| 23.0.8              | Native Firebase Message SDK version.<br>:warning: Replaced by BoM versioning on Gradle >= 3.4. |
|ANDROID_FIREBASE_BOM_VERSION| 29.0.1              | [Firebase BoM](https://firebase.google.com/docs/android/learn-more#bom) version.               |
|ANDROID_GOOGLE_SERVICES_VERSION| 4.3.4               | Native Google Services SDK version.                                                            |
|ANDROID_GRADLE_TOOLS_VERSION| 4.1.0               | Gradle tools version.                                                                          |
|IOS_FIREBASE_MESSAGING_VERSION| ~> 7.4.0            | Native Firebase Message SDK version                                                            |
|IOS_GOOGLE_UTILITIES_VERSION| 7.11.5              | GoogleUtilities pod version                                                                    |

#### Cordova

Default:

```sh
npm uninstall @ionic-native/fcm # Ionic support is included and conflicts with @ionic-native's implementation.
npm install cordova-plugin-fcm-with-dependecy-updated@git+https://github.com/4sh/cordova-plugin-fcm-with-dependecy-updated.git#7.10.0
```

Complete:

```sh
npm uninstall @ionic-native/fcm # Ionic support is included and conflicts with @ionic-native's implementation.
npm install cordova-plugin-fcm-with-dependecy-updated@git+https://github.com/4sh/cordova-plugin-fcm-with-dependecy-updated.git#7.10.0 \
  --variable ANDROID_DEFAULT_NOTIFICATION_ICON="@mipmap/ic_launcher" \
  --variable ANDROID_FIREBASE_BOM_VERSION="26.0.0" \
  --variable ANDROID_GOOGLE_SERVICES_VERSION="4.3.4" \
  --variable ANDROID_GRADLE_TOOLS_VERSION="4.1.0" \
  --variable IOS_GOOGLE_UTILITIES_VERSION="7.11.5" \
  --variable IOS_FIREBASE_MESSAGING_VERSION="~> 7.4.0"
```

## Push Payload Configuration

Besides common FCM configuration (https://firebase.google.com/docs/cloud-messaging/ios/certs), the Push payload should contain "notification" and "data" keys and "click_action" equals to "FCM_PLUGIN_ACTIVITY" within "notification".

Structure expected:
```js
{
  ...,
  "notification": {
    ...
  },
  "data": {
    ...
  },
  "android": {
    "notification": {
      "click_action": "FCM_PLUGIN_ACTIVITY"
    }
  },
  ...,
}
```

Example:
```json
{
  "token": "[FCM token]",
  "notification":{
    "title":"Notification title",
    "body":"Notification body",
    "sound":"default",
  },
  "data":{
    "param1":"value1",
    "param2":"value2"
  },
  "android": {
    "notification": {
      "icon":"fcm_push_icon",
      "click_action": "FCM_PLUGIN_ACTIVITY"
    }
  }
}
```


## Features

- [As its own](#as-its-own)
  - [FCM.clearAllNotifications()](#fcmclearallnotifications)
  - [FCM.createNotificationChannel()](#fcmcreatenotificationchannel)
  - [FCM.deleteInstanceId()](#fcmdeleteinstanceid)
  - [FCM.getAPNSToken()](#fcmgetapnstoken)
  - [FCM.getInitialPushPayload()](#fcmgetinitialpushpayload)
  - [FCM.getToken()](#fcmgettoken)
  - [FCM.hasPermission()](#fcmhaspermission)
  - [FCM.onNotification()](#fcmonnotification)
  - [FCM.onTokenRefresh()](#fcmontokenrefresh)
  - [**FCM.requestPushPermission()**](#fcmrequestpushpermission)
  - [FCM.subscribeToTopic()](#fcmsubscribetotopic)
  - [FCM.unsubscribeFromTopic()](#fcmunsubscribefromtopic)
  - [FCM.eventTarget](#fcmeventtarget)
- [**With Ionic**](#with-ionic)
  - [FCM.onNotification()](#fcmonnotification-1)
  - [FCM.onTokenRefresh()](#fcmontokenrefresh-1)

#### As its own

The JS functions are now as written bellow and do require Promise support. Which, for Android API 19 support, it can be fulfilled by a polyfill.

##### FCM.clearAllNotifications()

Removes existing push notifications from the notifications center.
```typescript
await FCM.clearAllNotifications();
```

##### FCM.createNotificationChannel()

For Android, some notification properties are only defined programmatically.
Channel can define the default behavior for notifications on Android 8.0+.
Once a channel is created, it stays unchangeable until the user uninstalls the app.
```typescript
await FCM.createNotificationChannel({
  id: "urgent_alert", // required
  name: "Urgent Alert", // required
  description: "Very urgent message alert",
  importance: "high", // https://developer.android.com/guide/topics/ui/notifiers/notifications#importance
  visibility: "public", // https://developer.android.com/training/notify-user/build-notification#lockscreenNotification
  sound: "alert_sound", // In the "alert_sound" example, the file should located as resources/raw/alert_sound.mp3
  lights: true, // enable lights for notifications
  vibration: true // enable vibration for notifications
});
```

##### FCM.deleteInstanceId()

Deletes the InstanceId, revoking all tokens.
```typescript
await FCM.deleteInstanceId();
```

##### FCM.getAPNSToken()

Gets iOS device's current APNS token.
```typescript
const apnsToken: string = await FCM.getAPNSToken();
```

##### FCM.getInitialPushPayload()

Retrieves the message that, on tap, opened the app. And `null`, if the app was open normally.
```typescript
const pushPayload: object = await FCM.getInitialPushPayload()
```

##### FCM.getToken()

Gets device's current registration id.
```typescript
const fcmToken: string = await FCM.getToken()
```

##### FCM.hasPermission()

On iOS, returns `true` if runtime permission for remote notifications is granted and enabled in Settings.
On Android, returns `true` if global remote notifications are enabled in the device settings and, from Android 13+, runtime permission for remote notifications is granted.

```typescript
const doesIt: boolean = await FCM.hasPermission()
```

##### FCM.onNotification()

Callback firing when receiving new notifications. It serves as a shortcut to listen to eventTarget's "notification" event.
```typescript
const disposable = FCM.onNotification((payload: object) => {
  // ...
})
// ...
disposable.dispose() // To remove listener
```

:warning: If the subscription to notification events happens after the notification has been fired, it'll be lost. As it is expected that you'd not always be able to catch the notification payload that the opened the app, the `FCM.getInitialPushPayload()` method was introduced.

##### FCM.onTokenRefresh()

Callback firing when receiving a new Firebase token. It serves as a shortcut to listen to eventTarget's "tokenRefresh" event.
```typescript
const disposable = FCM.onTokenRefresh((fcmToken: string) => {
  // ...
})
// ...
disposable.dispose() // To remove listener
```

##### FCM.requestPushPermission()

Grant run-time permission to receive push notifications (will trigger user permission dialog).
iOS & Android 13+ (Android <= 12 does not display any system dialog and immediately returns the value of `hasPermission`).

On Android, the `POST_NOTIFICATIONS` permission has been added to the `AndroidManifest.xml`.

```typescript
const wasPermissionGiven: boolean = await FCM.requestPushPermission({
  ios9Support: {
    timeout: 10,  // How long it will wait for a decision from the user before returning `false`
    interval: 0.3 // How long between each permission verification
  }
})
```

:warning: Without this request, the application won't receive any notification on iOS or Android 13+ (notifications are turned off by default)!

On iOS, the user will only have its permission required once, after that time, this call will only return if the permission was given that time. 
On Android, your app has complete control over when the permission dialog is displayed. Use this opportunity to explain to users why the app needs this permission, encouraging them to grant it. See [best practices](https://developer.android.com/develop/ui/views/notifications/notification-permission#wait-to-show-prompt).

##### FCM.subscribeToTopic()

Subscribes you to a [topic](https://firebase.google.com/docs/notifications/android/console-topics).
```typescript
const topic: string
// ...
await FCM.subscribeToTopic(topic)
```

##### FCM.unsubscribeFromTopic()

Unsubscribes you from a [topic](https://firebase.google.com/docs/notifications/android/console-topics).
```typescript
const topic: string
// ...
await FCM.unsubscribeFromTopic(topic)
```

##### FCM.eventTarget

EventTarget object for native-sourced custom events. Useful for more advanced listening handling.
```typescript
const listener = (data) => {
  const payload = data.detail
  // ...
}
FCM.eventTarget.addEventListener("notification", listener, false);
// ...
FCM.eventTarget.removeEventListener("notification", listener, false);
```

#### With Ionic

Ionic support was implemented as part of this plugin to allow users to have access to newer features with the type support.

- Ionic v5:
```typescript
import {FCM} from 'cordova-plugin-fcm-with-dependecy-updated/ionic';

// ...
private listenToInitialPushNotification(): Subscription {
  return from(FCM.getInitialPushPayload()).pipe(
          filter(data => data && data.wasTapped),
  ).subscribe(data => console.log(data));
}

```

It brings the same behavior as the native implementation, except for `FCM.onNotification()` and `FCM.onTokenRefresh()`, which gain rxjs' Observable support.

To avoid confusion, it's suggested to also remove the redundant `@ionic-native/fcm` package.

##### FCM.onNotification()

Event firing when receiving new notifications.
```typescript
FCM.onNotification().subscribe((payload: object) => {
  // ...
});
```

##### FCM.onTokenRefresh()

Event firing when receiving a new Firebase token.
```typescript
FCM.onTokenRefresh().subscribe((token: string) => {
  // ...
});
```

