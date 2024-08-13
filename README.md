# GripSDK
- [GripSDK](#gripsdk)
  - [요구사항](#요구사항)
  - [Gradle 설정](#Gradle-설정)
  - [GripSDK 초기화](#GripSDK-초기화)
  - [홈 피드 연동 (1탭)](#홈-피드-연동-(1탭))
  - [커머스 탭 웹뷰 연동 (3탭)](#커머스-탭-웹뷰-연동-(3탭))
  - [다크 모드 설정](#다크-모드-설정)

## 요구사항
- minSdk 26 이상

## Gradle 설정
- 프로젝트의 Gradle 설정을 아래와 같이 설정하면 Grip Android SDK를 사용할 수 있습니다.

build.gradle(App)
```kt
dependencies {
    implementation("io.github.gripcorp:grip-android:1.0.0")
}
```
settings.gradle
```kt
dependencyResolutionManagement {
    repositories {
        jcenter() // or maven(url = "https://plugins.gradle.org/m2/")
    }
}
```

### Proguard 설정
```kt
-keep class co.gripcorp.** { *; }
-keep public final class com.google.gson.Gson {*;}
```
- proguard-rules.pro 에 추가

### 참고: 외부 라이브러리 의존성
- Grip SDK에서 사용하는 주요 외부라이브러리는 다음과 같습니다. 
    - OkHttp3, RxKotlin, Retrofit2, Glide, ExoPlayer


## GripSDK 초기화
```kt
    GripSdk.init(
        context = applicationContext,
        appKey = "your appkey",
        appName = "kakaoStory",
        appVersion = "your app version",
        isDarkMode = true | false,
        allowAutoPlay = true | false,
        isRealConfig = true | false
        onSuccess = { /* handle launch success */ },
        onError = { throwable -> /* handle launch error */}
    )
```
- GripSdk를 초기화하는 메서드입니다. 동기 작업으로 splash에 추가해야 합니다. 

### 파라미터
- `context: Context`
- `appkey: String`
  - 그립에서 발급해 준 app key 정보
- `appName: String`
  - kakaoStory name
- `appVersion: String`
  - kakaoStory app version
- `isDarkMode: Boolean`
  - true : 다크 모드
  - false : 라이트 모드
- `allowAutoPlay: Boolean`
  - true : 자동 재생 ON
  - false : 자동 재생 OFF
- `isRealConfig: Boolean`
  - true : 그립 real 서버와 연결
  - false : 그립 dev 서버와 연결
- `onSuccess: () -> Unit`
    - GripSdk를 초기화를 성공했을때 호출되는 Callback
- `onError: (Throwable) -> Unit`
    - GripSdk를 초기화가 실패 했을때 호출되는 Callback
    - **실패시 Throwable를 전달하며 에러 처리가 필요**
        - 그립이 노출되는 영역 제거 (홈 컴포넌트, 커머스탭)


## 홈 피드 연동 (1탭)
### 홈 피드 추가
```kt
    GripSdk.getInstance().initFeedContents(
        binding.carouselLayout,
        onContentClick = {
            logger.sendClickLog(...)
        }
        onResult = { feedResult: FeedResult ->
            val carouselVisible = feedResult is FeedResult.Success
            binding.carouselLayout.isVisible = carouselVisible
            binding.errorView.isVisible = !carouselVisible
        },
        onRequestAutoPlay = { commerceDestination: CommerceDestination ->
            Toast.makeText(context, "자동 재생을 ON 하시겠습니까?", Toast.LENGTH_SHORT).show()
            ...
            GripSdk.getInstance().openGripPage(commerceDestination)
        }
    )
```

### initFeedContents
- 그립 feed를 구성하고 추가하는 메서드입니다.

### 파라미터
#### ViewGroup
- 카카오 스토리 앱의 ViewGroup을 받습니다. (ex. 카스앱 홈피드 3번째 구좌의 grip feed contents를 보여줄 viewGroup)

#### onContentClick: () -> Unit
- Feed Content를 클릭했을 때 동작하는 Callback입니다.

#### onResult: (FeedResult) -> Unit
- Feed Contents의 결과를 수신하는 Callback입니다. FeedResult 전달합니다.

##### FeedResult.Success
- Feed Contents의 성공 결과 정보입니다.
    - `topTitle: String`
        - 1단에 표시되는 문구
    - `topIconDrawable: Drawable`
        - 1단에 표시되는 icon drawable
    - `bottomTitle: String`
        - 3단에 표시되는 문구
    - `commerceDestination: CommerceDestination`
        - 암호화 된 webUrl
        - setDestination 메소드의 parameter



##### FeedResult.Fail
- Feed Contents의 실패 결과 정보입니다.
    - `throwable: Throwable`
        - throwable 정보 
    - `message: String`
        - 에러 메세지

#### onRequestAutoPlay: (GripDestination) -> Unit
- 자동 재생 ON 유도 팝업 노출을 위한 Callback으로 GripDestination를 전달합니다.
- openGripPage() 메소드의 parameter
 

### 동영상 자동 재생 설정
```kt
    GripSdk.setAutoPlayEnabled(isEnabled: Boolean)
```
- 자동 재생의 설정 값을 그립앱에 전달합니다. 
#### 파라미터
- `isEnabled: Boolean`
  - true : 자동 재생 ON
  - false : 자동 재생 OFF
 


### 그립 페이지로 이동  
```kt
    Gripsdk.openGripPage(gripDestination: GripDestination)
```
- 그립 앱이 설치 되어 있으면 그립 앱이 실행되며, 그렇지 않으면 외부 브라우저가 실행 됩니다. 

#### 파라미터
- `gripDestination: GripDestination`

### 홈 피드 동작 여부 설정
```kt
    GripSdk.getInstance().setFeedEnabled(isEnabled: Boolean)
```
- `isEnabled: Boolean`
  - true : Feed 를 동작시킴
  - false : Feed 를 멈춤
 

## 커머스 탭 웹뷰 연동 (3탭)
### 커머스 탭 추가
```kt
    GripSdk.getInstance().getGripCommerceFragment(
        onError = { throwable -> /* handle web error */}
    ): Fragment
```
- `onError: (Throwable) -> Unit`
    - WebView 에서 error 발생하거나 서버 장애시 호출되는 Callback
        - 400~500 서버 장애
        - WebResourceError

### 커머스 탭 url 설정
```kt
    GripSdk.getInstance().setDestination(destination: CommerceDestination)
```

### 커머스 탭 WebView 동작 여부 설정
```kt
    GripSdk.getInstance().setGripCommerceViewEnabled(isEnabled: Boolean)
```
- `isEnabled: Boolean`
  - true : WebView 를 동작시킴
  - false : WebView 를 멈춤

## 다크 모드 설정
```kt
    GripSdk.getInstance().setDartMode(isDarkMode: Boolean)
```
- 다크 모드 설정이 변경 될 때 사용합니다.

### 파라미터
- `isDarkMode: Boolean`
  - 다크 모드 상태 값
  - true : 다크 모드
  - false : 라이트 모드
