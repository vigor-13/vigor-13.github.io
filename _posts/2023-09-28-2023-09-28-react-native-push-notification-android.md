---
layout: post
title: React Native에서 안드로이드 푸시 알림 팝업 노출 안되는 이슈
date: 2023-09-28 07:25 +0900
categories: [react_native, notification]
tags: [react_native, push_notification, android]
---

React Native 안드로이드 백그라운드 모드에서 푸시 알림 팝업이 안뜨는 이슈가 발생했다. 푸시 알림 자체는 오고 있었지만 팝업만 노출이 안되서 애를 먹었다.

당시 앱에서 푸시 알림 기능을 위해서 사용중이던 스택은 다음과 같다.

- @react-native-firebase/messaging
- react-native-push-notification
- @react-native-community/push-notification-ios

알림 자체는 정상적으로 오고 있었기 때문에 파이어베이스 메시징쪽 문제는 아니라고 판단했고, `react-native-push-notification` 깃헙 레포지토리 이슈 트래커에서 관련 내용을 검색해본 결과 문제를 해결할 수 있었다.

## Notification을 보내기 전에는 반드시 채널이 생성되어 있어야 한다.

안드로이드에서는 notification을 보내기 전에 반드시 채널이 생성되어 있어야 한다고 한다.

> Hi, Has written in the Readme, channel must be created before sending a notification. It's an Android requirement. The fcm default channel might be created by the Firebase Sdk in some cases. Highly recommend to handle it yourself.

코드를 확인해본 결과 채널은 정상적으로 생성하고 있었다. 

```typescript
if (Platform.OS === 'android') {
  const channelId = this.getChannelId();

  if (channelId) {
    PushNotification.createChannel(
      {
        channelId,
        channelName: '채널명',
      },
      () => {
        console.log('noti channelId is created');
      },
    );
  }
}
```

## AndroidManifest.xml 파일에 메타데이터 추가

- [https://github.com/zo0r/react-native-push-notification/issues/1395](https://github.com/zo0r/react-native-push-notification/issues/1395)
- [https://github.com/zo0r/react-native-push-notification#channel-management-android](https://github.com/zo0r/react-native-push-notification#channel-management-android)

이슈 트레커를 더 뒤져본 결과 `AndroidManifest.xml` 파일에 채널명을 메타데이터로 추가해야 한다고 한다.

```xml
<meta-data
  android:name="com.google.firebase.messaging.default_notification_channel_id"
  tools:replace="android:value" android:value="채널명" />
```
{: file="android > app > src > main > AndroidManifest.xml" }

정리
채널은 안드로이드 요구사항이기 때문에 반드시 생성해야 한다. 기본적으로 파이어베이스에서 디폴트 채널을 생성해주지만 디폴트 채널이 아닌 다른 채널을 생성해서 사용하는 경우 이를 애플리케이션 메타데이터에 입력하여 알려줘야 한다. 



## 🔗 참조
---
- [https://github.com/zo0r/react-native-push-notification/issues/2189](https://github.com/zo0r/react-native-push-notification/issues/2189)
- [https://github.com/zo0r/react-native-push-notification/issues/1395](https://github.com/zo0r/react-native-push-notification/issues/1395)
- [https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko)
- [https://blog.ysoft.kr/48](https://blog.ysoft.kr/48)
