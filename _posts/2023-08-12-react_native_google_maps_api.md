---
layout: post
title: React Native에서 Google Map API 사용하기
date: 2023-08-12 22:32 +0900
categories: [react_native, map]
tags: [react_native, react_native_maps, google_map]
---

React Native 프로젝트에 구글맵을 추가해야할 일이 생겼다. 요구사항은 다음과 같았다.

- [ ] 구글맵 렌더링
- [ ] 사용자 현재 위치로 맵 이동하기
- [ ] 검색한 위치로 맵 이동하기
- [ ] 클릭한 위치에 마커 표시하기
- [ ] 마커의 위,경도 값을 이용하여 해당 위치의 주소 가져오기

간단한 요구사항이었지만 구글맵 설정 과정에서 몇 가지 유의사항이 있어 문서화하기로 결정했다.

## react-native-maps 설치하기

React Native에서 구글 맵을 사용하기 위해서 `react-native-maps` 라이브러리가 필요하다.

자세한 설치 방법은 [Github 레포지토리](https://github.com/react-native-maps/react-native-maps/blob/master/docs/installation.md)에서 확인 할 수 있으며, 요약하자면 다음과 같다.

#### 패키지 설치

NPM에서 `react-native-maps` 패키지를 설치한다.

```shell
yarn add react-native-maps
```
#### Android 설정

`android/app/src/main/AndroidManifest.xml` 파일에 다음의 코드를 추가한다.

```xml
<application>
   <!-- You will only need to add this meta-data tag, but make sure it's a child of application -->
   <meta-data
     android:name="com.google.android.geo.API_KEY"
     android:value="Your Google maps API Key Here"/>
</application>
```

#### iOS 설정

`AppDelegate.mm` 파일에 다음의 코드를 추가한다.

```objc
...

#import <GoogleMaps/GoogleMaps.h> // 추가

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [GMSServices provideAPIKey:@"_YOUR_API_KEY_"]; // 추가

  ...
```

> `[GMSServices provideAPIKey]` 는 메서드의 첫번째 줄에 추가 해야한다.
{:.prompt-warning}

> Google Maps SDK는 iOS 13 버전이 필요하기 때문에 `deployment target` 을 13 이상으로 설정해야 한다.
{:.prompt-warning}


## Google Map API Key 생성하기

`AndroidManifest.xml` 파일과 `AppDelegate.mm` 파일에 API 키를 추가해야 구글맵을 사용할 수 있다.

API 키는 [Google Cloud 콘솔](https://cloud.google.com/cloud-console)에서 생성할 수 있다.

먼저 Google Cloud 콘솔에 접속하여 콘솔을 시작한다.

![1](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/1.png?updatedAt=1692211976698)

상단 헤더에서 버튼을 클릭하여 `새 프로젝트` 를 생성한다.

![2](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/2.png?updatedAt=1692212167868)

프로젝트 이름을 추가하고 `만들기` 버튼을 클릭한다.

![3](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/3.png?updatedAt=1692212323449)

프로젝트가 생성되면 헤더에서 생성한 프로젝트를 선택할 수 있다.

![4](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/4.png?updatedAt=1692212464156)

이제 생성한 프로젝트에서 사용할 API를 선택해야 한다. 죄측의 메뉴에서 `라이브러리` 탭을 선택한다.

![5](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/5.png?updatedAt=1692212691130)

map으로 검색하면 지도와 관련된 여러 API를 확인할 수 있다. 이중 `Maps SDK for Android, iOS`를 선택하여 활성화 한다.

![6](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/6.png?updatedAt=1692635034864)

이제 사용자 인증 정보 탭에서 API Key를 확인 할 수 있다.

![7](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/7.png?updatedAt=1692635375030)

API Key를 코드에 추가하기 전에 보안을 위해서 `애플리케이션 제한사항 설정`과 `API 제한 사항`을 설정한다. 

`애플리케이션 제한 사항` 은 API Key를 사용할 수 있는 애플리케이션을 제한한다.

Android의 경우 환경별 키스토어(`debug.keystore` 등...)를 사용하여 SHA-1 인증서를 만들어서 등록하면 된다. 인증서를 만드는 방법은 해당 페이지 우측에 나온대로 코드를 실행해주면 된다.

```bash
# 개발 환경용 
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android

# 배포 환경용
keytool -list -v -keystore your_keystore_name -alias your_alias_name
```

> 위에서 등록한 SHA-1 인증서는 개발 환경과 배포 환경 2가지 모두 apk 파일에만 적용된다. 앱을 스토어 실제 배포하는 경우에는 aab 파일을 등록하게 되는데 이 경우 추가로 aab용 SHA-1 인증서를 추가해야 실제 배포된 앱에서 지도를 사용할 수 있다. 이 인증서는 구글 플레이 콘솔에서 확인할 수 있다. 자세한 것은 [여기](https://stackoverflow.com/questions/71973211/google-map-not-working-with-aab-file-but-is-working-with-apk)를 확인해 보자.
{:.prompt-info}

iOS는 훨씬 간단하다. 앱의 번들 아이디만 추가해주면 된다.

![8](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/8.png?updatedAt=1692635430849)

`API 제한 사항` 에서는 해당 API Key가 접근할 수 있는 API를 제한할 수 있다. 다른 API는 접근할 수 없도록 필요한 API만 찾아서 설정해주면 된다.

![9](https://ik.imagekit.io/vigor13/2023.08.12-react-native-google-map-example/9.png?updatedAt=1692635706201)

## App API vs Web API

요구 사항 중에는 검색 기능과 위, 경도 값을 사용하여 주소 값을 가져오는 기능이 있었다. 이를 위해서 다음의 2가지 API를 더 추가해야 했다.

- `Geocoding API` : 위,경도 값을 주소로 변환하거나 주소를 위경도 값으로 변환할 수 있다.
- `Places API` : 장소 검색 기능을 제공한다. 

앞서 생성한 API Key에 `Geocoding API` 와 `Places API` 를 추가해도 API가 작동하지 않는 오류가 발생했는데 이유는 `애플리케이션 제한 사항 설정` 때문이다. `애플리케이션 제한 설정`은 말 그대로 애플리케이션에만 적용 가능한 설정이다 ([참조](https://stackoverflow.com/questions/40938504/why-google-places-api-key-not-works)). `Geocoding API` 와 `Places API` 는 웹 서비스이기 때문에 사용할 수 없다. 보통 이런 API들은 백엔드 서버에 추가하여 서버 엔드포인를 통해서 사용하는 것이 일반적이다. 

하지만 당장 서버를 통해서 API를 사용하는 것은 불가능했다. Geocoding API의 경우 위치 데이터를 서버에 전송하는 것 자체가 문제가 되었다. 위치 정보는 민감한 데이터이기 때문에 서버에 전송하여 사용하려면 관련 법령을 검토하여 절차를 밟아야 한다고 한다. ([참조](https://www.lbsc.kr/front/content/contentViewer.do?contentId=CONTENT_0000061))

Places API의 경우에는 `react-native-google-places-autocomplete` 라이브러를 사용했는데 클라이언트에 직접 API Key를 등록하여 사용해야 했다.

결국 하는 수 없이 `Geocoding API` 와 `Places API` 를 위한 새 API Key를 생성하고 사용량 제한을 걸어 사용하기로 결정하여 마무리 했다.