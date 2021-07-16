---
title: 플랫폼 별 코드 작성
short-title: 플랫폼 별 코드
description: 앱에서 커스텀하게 플랫폼 별 코드를 작성하는 방법을 배워보세요.
---

이 가이드는 플랫폼 별 코드를 어떻게 작성하는지 설명합니다. 어떤 플랫폼 별 기능은 
이미 있는 패키지를 통해 사용이 가능합니다. 
[using packages](/docs/development/packages-and-plugins/using-packages)를 
참조하세요.

Flutter는 Android에서의 Java와 Kotlin, 
iOS에서의 Objective-C와 Swift에서 가능한 플랫폼 별 api를
사용하게 해주는 유연한 시스템을 사용합니다.

Fluttr의 플랫폼 별 API는 코드 생성에 의존하고 있지 않고, 
유연한 메시지 전달 스타일을 사용합니다.

* 앱의 Flutter 부분은 플랫폼 채널을 통해서 iOS나 Android가 될 수 있는 
  *호스트* 에게 메시지를 보냅니다.

* The *host* listens on the platform channel, and receives the message.
  It then calls into any number of platform-specific APIs&mdash;using
  the native programming language&mdash;and sends a response back to the
  *client*, the Flutter portion of the app.

{{site.alert.note}}
  This guide addresses using the platform channel mechanism if you need
  to use the platform's APIs or libraries in Java/Kotlin/Objective-C or Swift.
  But you can also write platform-specific Dart code in your Flutter app
  by inspecting the
  [defaultTargetPlatform]({{site.api}}/flutter/foundation/defaultTargetPlatform.html)
  property. [Platform adaptations](/docs/resources/platform-adaptations)
  lists some platform-specific adaptations that Flutter automatically does
  for you in the framework.
{{site.alert.end}}

## 아키텍쳐 훝어 보기: 플랫폼 채널 {#architecture}

클라이언트(UI)와 호스트(플랫폼)이 플랫폼 채널에서 사용하는 메시지 형태는 
아래 다이어그램에 그려져있습니다.

![Platform channels architecture](/images/PlatformChannels.png)

메시지와 응답은 반응성 좋은 사용자 인터페이스를 위해 
비동기적으로 전달됩니다.

{{site.alert.note}}
  Even though Flutter sends messages to and from Dart asynchronously,
  whenever you invoke a channel method, you must invoke that method on the
  platform's main thread. See the
  [section on threading](#channels-and-platform-threading) for more information.
{{site.alert.end}}

클라이언트 단에서는, `MethodChannel` ([API][MethodChannel])이 메시지를 그에 상응하는
메서드로 보낼 수 있도록 해줍니다. 플랫폼 단에서는, Android는 `MethodChannel`([API][MethodChannelAndroid]), iOS는 `FlutterMethodChannel`
([API][MethodChanneliOS])들이 메시지를 받는 것과 응답을 가능하게 합니다. 이 클래스들은 아주 적은
코드만으로도 플랫폼 플러그인을 개발할 수 있게 해줍니다.

*참고*: 원한다면 메서드 호출은 반대의 방향으로도 보내질 수 있습니다.
(Dart로 구현된 메서드의 클라이언트 역할을 하는 플랫폼일 때) 예시는 
이 플러그인([`quick_actions`]({{site.api}}/packages/quick_actions))을 봐주세요.

[MethodChannel]: {{site.api}}/flutter/services/MethodChannel-class.html
[MethodChannelAndroid]: {{site.api}}/javadoc/io/flutter/plugin/common/MethodChannel.html
[MethodChanneliOS]: {{site.api}}/objcdoc/Classes/FlutterMethodChannel.html

### 플랫폼 채널 지원 데이터형과 코덱 {#codec}

표준 플랫폼 채널은 간단한 json 형태의 효율적인 바이너리 직렬화를 지원하는 
boolean, numbers, Strings, byte butters, List, Map등의 
표준 메시지 코덱을 사용합니다 
(참고: [`StandardMessageCodec`]({{site.api}}/flutter/services/StandardMessageCodec-class.html)).
이 값들에 대한 메시지 직렬화와 역직렬화는 당신이 값을 보내고 받을 때 자동으로 이루어집니다.

아래 표는 dart의 자료형이 플랫폼에서 어떻게 받고 보내지는지 보여줍니다.

| Dart        | Android             | iOS
|-------------|---------------------|----
| null        | null                | nil (NSNull when nested)
| bool        | java.lang.Boolean   | NSNumber numberWithBool:
| int         | java.lang.Integer   | NSNumber numberWithInt:
| int, if 32 bits not enough | java.lang.Long | NSNumber numberWithLong:
| double      | java.lang.Double    | NSNumber numberWithDouble:
| String      | java.lang.String    | NSString
| Uint8List   | byte[]   | FlutterStandardTypedData typedDataWithBytes:
| Int32List   | int[]    | FlutterStandardTypedData typedDataWithInt32:
| Int64List   | long[]   | FlutterStandardTypedData typedDataWithInt64:
| Float64List | double[] | FlutterStandardTypedData typedDataWithFloat64:
| List        | java.util.ArrayList | NSArray
| Map         | java.util.HashMap   | NSDictionary

<br>
## 예시: 플랫폼 채널을 이용해서 iOS와 Android 코드 호출하기 {#example}

아래 코드에선 현재 배터리 레벨를 표시하기 위해 플랫폼 특화 API를 사용하는 법을 설명합니다. 
Android의 `BatteryManager`, iOS의 `device.batteryLevel` API를 
`getBatteryLevel` 이라는 단일 플랫폼 메시지로 사용합니다.

해당 예시는 메인 앱 자체에 플랫폼 별 코드를 추가합니다. 만약 당신이 다양한 앱에서 플랫폼 특화 코드를 재사용하고
싶다면, 프로젝트 시작 방법이 약간 다릅니다([developing
packages](/docs/development/packages-and-plugins/developing-packages#plugin) 참고).
하지만 플랫폼 채널 코드는 여전히 같은 방법으로 작성됩니다.

*참고*: 이 예시의 실행가능한 전체 코드는 여기서 확인할 수 있습니다. 
[`/examples/platform_channel/`]({{site.github}}/flutter/flutter/tree/master/examples/platform_channel)
Android는 Java로 되어 있고 iOS 는 Objective-C 로 되어있습니다. 
Swift 와 iOS에 대한 예제는 다음을 참고하세요.
[`/examples/platform_channel_swift/`]({{site.github}}/flutter/flutter/tree/master/examples/platform_channel_swift)

### 1단계: 새로운 앱 프로젝트 만들기 {#example-project}

새로운 앱을 만들어서 시작하세요.

* 터미널에서 실행: `flutter create batterylevel`

기본적으로 Android는 Java, iOS는 Objective-C를 사용해서 작성하는 템플릿으로 지원됩니다. Kotlin 이나 Swift를
사용하려면 `-i` 혹은 `-a` 플래그를 사용하세요.

By default our template supports writing Android code using Kotlin, or iOS code
using Swift. To use Java or Objective-C, use the `-i` and/or `-a` flags:

* In a terminal run: `flutter create -i objc -a java batterylevel`

앱의 `State` 클래스가 현재 앱 상태를 저장합니다. 
현재 배터리 상태를 저장하려면 이를 상속해야 합니다.

먼저, 채널을 생성합니다. 배터리 레벨을 반환하는 
플랫폼 메서드를 가지고 있는 `MethodChannel`을 사용할 것입니다.


채널의 클라이언트와 호스트는 채널 생성자를 통해 전달된 채널 이름으로 연결됩니다. 
하나의 앱에서 사용하는 모든 채널 이름은 유일해야 하기 때문에, 
유일한 도메인 접두사를 사용해서 채널 이름 앞에 추가하세요. 
예) `samples.flutter.io/battery`

<!-- skip -->
<?code-excerpt "main.dart" title?>
```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
...
class _MyHomePageState extends State<MyHomePage> {
  static const platform = const MethodChannel('samples.flutter.dev/battery');

  // 배터리 레벨을 가져옵니다.
}
```

다음으로, `getBatteryLevel` 구분자를 통해 특정 메서드를 지정해서 
메서드 채널에서 메서드를 실행합니다.
호출은 실패할 수도 있습니다. 
예를 들어 특정 플랫폼이 해당 API를 지원하지 않는 경우(시뮬레이터에서의 실행 등)가 있습니다.
그러므로 `invokeMethod` 호출을 try-catch 문으로 감싸주세요.


반환된 결과를 `setState` 함수 안에서 사용해서 `_batteryLevel` 변수가 들고있는 UI 상태를 업데이트 하세요.

<!-- skip -->
<?code-excerpt "main.dart" title?>
```dart
  // 배터리 레벨을 가져옵니다.
  String _batteryLevel = 'Unknown battery level.';

  Future<void> _getBatteryLevel() async {
    String batteryLevel;
    try {
      final int result = await platform.invokeMethod('getBatteryLevel');
      batteryLevel = 'Battery level at $result % .';
    } on PlatformException catch (e) {
      batteryLevel = "Failed to get battery level: '${e.message}'.";
    }

    setState(() {
      _batteryLevel = batteryLevel;
    });
  }
```

마지막으로, 문자열로 배터리 상태를 표시하는 작은 UI와 
그 값을 새로고침하는 버튼을 생성하도록 
기존 템플릿의 `build` 메소드를 수정하세요.

<!-- skip -->
<?code-excerpt "main.dart" title?>
```dart
  @override
  Widget build(BuildContext context) {
    return Material(
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            RaisedButton(
              child: Text('Get Battery Level'),
              onPressed: _getBatteryLevel,
            ),
            Text(_batteryLevel),
          ],
        ),
      ),
    );
  }
```

### Step 3: Add an Android platform-specific implementation

{% samplecode android-channel %}
{% sample Java %}
Start by opening the Android host portion of your Flutter app in Android Studio:

1. Android 스튜디오 실행

1. 메뉴에서 **File > Open...** 선택

1. Flutter 앱이 있는 디렉토리로 이동해서, 
   그 안의 **android** 폴더를 선택합니다. **OK** 클릭하세요.

1. 프로젝트 뷰에서 
   **java** 폴더 안에 있는 `MainActivity.java` 를 엽니다.

다음으로, `MethodChannel` 을 만들고 `onCreate()` 메서드 안에서 
`MethodCallHandler` 를 설정합니다.
Flutter 클라이언트 측과 같은 채널 이름이 사용되었는지 확인해주세요.

<?code-excerpt "MainActivity.java" title?>
```java
import androidx.annotation.NonNull;
import io.flutter.embedding.android.FlutterActivity;
import io.flutter.embedding.engine.FlutterEngine;
import io.flutter.plugin.common.MethodChannel;
import io.flutter.plugins.GeneratedPluginRegistrant;

public class MainActivity extends FlutterActivity {
  private static final String CHANNEL = "samples.flutter.dev/battery";

  @Override
  public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
    GeneratedPluginRegistrant.registerWith(flutterEngine);
    new MethodChannel(flutterEngine.getDartExecutor().getBinaryMessenger(), CHANNEL)
        .setMethodCallHandler(
          (call, result) -> {
            // Note: this method is invoked on the main thread.
            // TODO
          }
        );
  }
}
```


배터리 레벨을 가져오기 위해 Android 배터리 API를 사용하는 
실제 Android Java 코드를 추가하세요. 
해당 코드는 네이티브 Android 앱에서 사용하던 것과 완전히 같습니다.

첫번째로, 필요한 import 들을 파일 상단에 추가해주세요.

<?code-excerpt "MainActivity.java" title?>
```java
import android.content.ContextWrapper;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.BatteryManager;
import android.os.Build.VERSION;
import android.os.Build.VERSION_CODES;
import android.os.Bundle;
```

Then add the following as a new method in the activity class,
below the `configureFlutterEngine()` method:

<?code-excerpt "MainActivity.java" title?>
```java
  private int getBatteryLevel() {
    int batteryLevel = -1;
    if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
      BatteryManager batteryManager = (BatteryManager) getSystemService(BATTERY_SERVICE);
      batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY);
    } else {
      Intent intent = new ContextWrapper(getApplicationContext()).
          registerReceiver(null, new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
      batteryLevel = (intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100) /
          intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
    }

    return batteryLevel;
  }
```

Finally, complete the `setMethodCallHandler()` method added earlier.
You need to handle a single platform method, `getBatteryLevel()`,
so test for that in the `call` argument. The implementation of
this platform method calls the Android code written
in the previous step, and returns a response for both
the success and error cases using the `response` argument.
If an unknown method is called, report that instead.

마지막으로, 먼저 추가한 `onMethodCall()` 메서드를 완성합니다. 
사용할 플랫폼 메서드는 `getBatteryLevel()`이며, 
이는 `call` 인자 안에서 가져와 테스트 해볼 수 있습니다. 
이 플랫폼 메서드는 단순히 이전 단계에서 작성한 Android 코드를 호출합니다.
그리고 `response` 매개변수를 통해 성공과 에러를 응답으로 돌려줍니다. 
이 플랫폼 메서드는 전 단계에서 작성한 Android 코드를 호출하며, 
`response` 인자를 통해 성공과 에러를 응답으로 돌려줍니다. 
만약 알 수 없는 메서드가 호출된다면, 예외처리가 필요합니다.

아래 코드를:

<?code-excerpt "MainActivity.java" title?>
```java
          (call, result) -> {
            // Note: this method is invoked on the main thread.
            // TODO
          }
```

이렇게 변경하세요:

<?code-excerpt "MainActivity.java" title?>
```java
          (call, result) -> {
            // Note: this method is invoked on the main thread.
            if (call.method.equals("getBatteryLevel")) {
              int batteryLevel = getBatteryLevel();

              if (batteryLevel != -1) {
                result.success(batteryLevel);
              } else {
                result.error("UNAVAILABLE", "Battery level not available.", null);
              }
            } else {
              result.notImplemented();
            }
          }
```
{% sample Kotlin %}
Start by opening the Android host portion of your Flutter app in Android Studio:

안드로이드 스튜디오에서 Flutter 앱의 안드로이드 부분을 열어서 시작하세요:

1. 안드로이드 스튜디오 실행

1. 메뉴에서 **File > Open...** 선택

1. Flutter 앱이 있는 디렉토리로 이동해서, 
   그 안의 **android** 폴더를 선택합니다. **OK**를 클릭합니다.

1. 프로젝트 뷰에서 **kotlin** 폴더 안에 있는 `MainActivity.kt` 파일을 열어주세요. 
   (참고: 만약 안드로이드 스튜디오 2.3 버전을 사용한다면, 
   **kotlin** 폴더가 **java** 폴더로 보이는 것에 유의하세요.)


`onCreate` 메서드 안에서 `MethodChannel`을 만들고 
`setMethodCallHandler` 를 호출하세요.
Flutter 클라이언트 쪽에서 사용한 것과 같은 채널 이름을 사용했는지 확인해주세요.

<?code-excerpt "MyActivity.kt" title?>
```kotlin
import androidx.annotation.NonNull
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {
  private val CHANNEL = "samples.flutter.dev/battery"

  override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler {
      call, result ->
      // Note: this method is invoked on the main thread.
      // TODO
    }
  }
}
```

다음으로, 배터리 레벨을 가져오기 위해 안드로이드 배터리 API를 사용하는 
안드로이드 Kotlin 코드를 추가합니다. 해당 코드는
네이티브 안드로이드 앱에서 사용하던 것과 완전히 같습니다.

첫 번째로, 필요한 import 들을 파일 상단에 추가해주세요.

<?code-excerpt "MyActivity.kt" title?>
```kotlin
import android.content.Context
import android.content.ContextWrapper
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import android.os.Build.VERSION
import android.os.Build.VERSION_CODES
```

Next, add the following method in the `MainActivity` class,
below the `configureFlutterEngine()` method:

<?code-excerpt "MyActivity.kt" title?>
```kotlin
  private fun getBatteryLevel(): Int {
    val batteryLevel: Int
    if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
      val batteryManager = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
      batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
    } else {
      val intent = ContextWrapper(applicationContext).registerReceiver(null, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
      batteryLevel = intent!!.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100 / intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
    }

    return batteryLevel
  }
```

Finally, complete the `setMethodCallHandler()` method added earlier. You need to
handle a single platform method, `getBatteryLevel()`, so test for that in the
`call` argument. The implementation of this platform method calls the
Android code written in the previous step, and returns a response for both
the success and error cases using the `response` argument.
If an unknown method is called, report that instead.

마지막으로, 먼저 추가한 `onMethodCall()` 메서드를 완성합니다. 
사용할 플랫폼 메서드는 `getBatteryLevel()` 이며, 
이는 `call` 매개변수 안에서 가져와 테스트 해볼 수 있습니다. 
이 플랫폼 메서드는 단순히 이전 단계에서 작성한 안드로이드 코드를 호출합니다.
그리고 `response` 매개변수를 통해 성공과 에러를 응답으로 돌려줍니다. 
만약 알 수 없는 메서드가 호출된다면, 예외처리가 필요합니다.

아래 코드를:

<?code-excerpt "MyActivity.kt" title?>
```kotlin
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler {
      call, result ->
      // Note: this method is invoked on the main thread.
      // TODO
    }
```

이렇게 변경하세요:

<?code-excerpt "MyActivity.kt" title?>
```kotlin
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler {
      // Note: this method is invoked on the main thread.
      call, result ->
      if (call.method == "getBatteryLevel") {
        val batteryLevel = getBatteryLevel()

        if (batteryLevel != -1) {
          result.success(batteryLevel)
        } else {
          result.error("UNAVAILABLE", "Battery level not available.", null)
        }
      } else {
        result.notImplemented()
      }
    }
```
{% endsamplecode %}

You should now be able to run the app on Android. If using the Android
Emulator, set the battery level in the Extended Controls panel
accessible from the **...** button in the toolbar.

### Step 4: Add an iOS platform-specific implementation

{% samplecode ios-channel %}
{% sample Objective-C %}
Start by opening the iOS host portion of the Flutter app in Xcode:

1. 메뉴에서 **File > Open...** 선택

1. Flutter 앱이 위치한 디렉토리고 이동하고, 
   그 안의 **ios** 폴더를 선택합니다. **OK** 를 클릭합니다.

1. Xcode 프로젝트가 에러 없이 빌드되는지 확인하세요.

1. 프로젝트 네비게이터에서 **Runner > Runner** 에 있는
   `AppDelegate.m` 파일을 엽니다.

`FlutterMethodChannel`을 만들고 
`applicationDidFinishLaunchingWithOptions:` 메서드 안에 
핸들러를 추가해주세요. Flutter 클라이언트 단과 같은 채널 이름이 사용되었는지 확인해주세요.


<?code-excerpt "AppDelegate.m" title?>
```objectivec
#import <Flutter/Flutter.h>
#import "GeneratedPluginRegistrant.h"

@implementation AppDelegate
- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
  FlutterViewController* controller = (FlutterViewController*)self.window.rootViewController;

  FlutterMethodChannel* batteryChannel = [FlutterMethodChannel
                                          methodChannelWithName:@"samples.flutter.dev/battery"
                                          binaryMessenger:controller];

  [batteryChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
    // Note: this method is invoked on the UI thread.
    // TODO
  }];

  [GeneratedPluginRegistrant registerWithRegistry:self];
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}
```

다음으로, 배터리 레벨을 가져오기 위해 iOS 배터리 API를 사용하는 
iOS ObjectiveC 코드를 추가합니다. 해당 코드는
네이티브 iOS 앱에서 사용하던 것과 완전히 같습니다.

`AppDelegate` 클래스에서 `@end` 바로 전에 아래 코드를 새로운 메서드로 추가해주세요.

<?code-excerpt "AppDelegate.m" title?>
```objectivec
- (int)getBatteryLevel {
  UIDevice* device = UIDevice.currentDevice;
  device.batteryMonitoringEnabled = YES;
  if (device.batteryState == UIDeviceBatteryStateUnknown) {
    return -1;
  } else {
    return (int)(device.batteryLevel * 100);
  }
}
```

마지막으로, 먼저 추가한 `setMethodCallHandler()` 메서드를 완성합니다. 
사용할 플랫폼 메서드는 `getBatteryLevel()` 이며, 
이는 `call` 매개변수 안에서 가져와 테스트 해볼 수 있습니다. 
이 플랫폼 메서드는 단순히 이전 단계에서 작성한 iOS 코드를 호출합니다.
그리고 `response` 매개변수를 통해 성공과 에러를 응답으로 돌려줍니다. 
만약 알수 없는 메서드가 호출된다면, 예외처리가 필요합니다.

<?code-excerpt "AppDelegate.m" title?>
```objectivec
__weak typeof(self) weakSelf = self;
[batteryChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
  // Note: this method is invoked on the UI thread.
  if ([@"getBatteryLevel" isEqualToString:call.method]) {
    int batteryLevel = [weakSelf getBatteryLevel];

    if (batteryLevel == -1) {
      result([FlutterError errorWithCode:@"UNAVAILABLE"
                                 message:@"Battery info unavailable"
                                 details:nil]);
    } else {
      result(@(batteryLevel));
    }
  } else {
    result(FlutterMethodNotImplemented);
  }
}];
```
{% sample Swift %}
Start by opening the iOS host portion of your Flutter app in Xcode:

1. Xcode 실행

1. 메뉴에서 **File > Open...** 선택

1. Flutter 앱이 위치한 디렉토리고 이동하고, 
   그 안의 `ios` 폴더를 선택합니다. OK 를 클릭합니다.

그리고 기본 템플릿에 Objective-C를 사용하는 Swift 에 대한 지원을 추가하세요.

1. 프로젝트 네비게이터에서 **Runner > Runner**을 여세요.

1. 프로젝트 네비게이터 안 **Runner > Runner** 에 위치한 
   `AppDelegate.swift` 파일을 여세요

다음으로, `application:didFinishLaunchingWithOptions:` 함수를 오버라이드 하고 
`samples.flutter.dev/battery` 를 채널 이름으로 사용하는 
`FlutterMethodChannel` 를 생성하세요. 

<?code-excerpt "AppDelegate.swift" title?>
```swift
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    let batteryChannel = FlutterMethodChannel(name: "samples.flutter.dev/battery",
                                              binaryMessenger: controller.binaryMessenger)
    batteryChannel.setMethodCallHandler({
      (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
      // Note: this method is invoked on the UI thread.
      // 배터리 메시지를 처리
    })

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

다음으로, 배터리 레벨을 가져오기 위해 iOS 배터리 API를 사용하는  
iOS의 Swift 코드를 추가합니다. 해당 코드는 
네이티브 iOS 앱에서 사용하던 것과 완전히 같습니다.

아래 코드를 `AppDelegate.swift`  맨 밑에 새로운 메서드로 추가하세요.

<?code-excerpt "AppDelegate.swift" title?>
```swift
private func receiveBatteryLevel(result: FlutterResult) {
  let device = UIDevice.current
  device.isBatteryMonitoringEnabled = true
  if device.batteryState == UIDevice.BatteryState.unknown {
    result(FlutterError(code: "UNAVAILABLE",
                        message: "Battery info unavailable",
                        details: nil))
  } else {
    result(Int(device.batteryLevel * 100))
  }
}
```

마지막으로, 먼저 추가한 `setMethodCallHandler()` 메서드를 완성합니다. 
사용할 플랫폼 메서드는 `getBatteryLevel()` 이며, 이는 `call` 매개변수 안에서 가져와 
테스트 해볼 수 있습니다. 이 플랫폼 메서드는 단순히 이전 단계에서 작성한 iOS 코드를 호출합니다.
그리고 `response` 매개변수를 통해 성공과 에러를 응답으로 돌려줍니다. 
만약 알수 없는 메서드가 호출된다면, 예외처리가 필요합니다.

<?code-excerpt "AppDelegate.swift" title?>
```swift
batteryChannel.setMethodCallHandler({
  [weak self] (call: FlutterMethodCall, result: FlutterResult) -> Void in
  // Note: this method is invoked on the UI thread.
  guard call.method == "getBatteryLevel" else {
    result(FlutterMethodNotImplemented)
    return
  }
  self?.receiveBatteryLevel(result: result)
})
```
{% endsamplecode %}

You should now be able to run the app on iOS. If using the iOS Simulator,
note that it does not support battery APIs,
and the app displays 'battery info unavailable'.

## 플랫폼 별 코드를 UI 코드에서 분리하기 {#separate}

만약 당신의 플랫폼 별 코드를 다양한 Flutter 앱에서 사용할 것 같다면, 
메인 어플리케이션 바깥 디렉토리에 플랫폼 플러그인으로 분리하는것이 유용합니다. 
자세한 내용은 
[developing packages](/docs/development/packages-and-plugins/developing-packages)를 
참고하세요.

## 플랫폼 특화 코드를 패키지로 배포하기 {#publish}

플랫폼 특화 코드를 Flutter 생태계의 다른 개발자들과 공유하고 싶다면, [publishing
packages](/docs/development/packages-and-plugins/developing-packages#publish)를 
참조하세요.

## 커스텀 채널과 코덱

위에 언급한 `MethodChannel` 외에도, 더 기본적이며 비동기적인 
커스텀 메시지 코덱을 사용해서 메시지를 전달하는
[`BasicMessageChannel`][BasicMessageChannel] 를 사용할 수도 있습니다. 
또한 특화된 [`BinaryCodec`][BinaryCodec], [`StringCodec`][StringCodec],
[`JSONMessageCodec`][JSONMessageCodec] 클래스들을 사용하거나, 
직접 코덱을 만드세요.

[BasicMessageChannel]: {{site.api}}/flutter/services/BasicMessageChannel-class.html
[BinaryCodec]: {{site.api}}/flutter/services/BinaryCodec-class.html
[StringCodec]: {{site.api}}/flutter/services/StringCodec-class.html
[JSONMessageCodec]: {{site.api}}/flutter/services/JSONMessageCodec-class.html

## Channels and Platform Threading

Invoke all channel methods on the platform's main thread when writing code on
the platform side. On Android, this thread is sometimes called the "main
thread", but it is technically defined as [the UI thread]. Annotate methods that
need to be run on the UI thread with `@UiThread`. On iOS, this thread is
officially referred to as [the main thread].

[the UI thread]: https://developer.android.com/guide/components/processes-and-threads#Threads
[the main thread]: https://developer.apple.com/documentation/uikit?language=objc

### Jumping to the UI thread in Android

To comply with channels' UI thread requirement, you may need to jump from a
background thread to Android's UI thread to execute a channel method. In
Android this is accomplished by `post()`ing a `Runnable` to Android's UI
thread `Looper`, which will cause the `Runnable` to execute on the main thread
at the next opportunity.

In Java:

```java
new Handler(Looper.getMainLooper()).post(new Runnable() {
  @Override
  public void run() {
    // Call the desired channel message here.
  }
});
```

In Kotlin:

```kotlin
Handler(Looper.getMainLooper()).post {
  // Call the desired channel message here.
}
```

### Jumping to the main thread in iOS

To comply with channel's main thread requirement, you may need to jump from a
background thread to iOS's main thread to execute a channel method. In iOS this
is accomplished by executing a [block] on the main [dispatch queue]:

In Objective-C:

```objectivec
dispatch_async(dispatch_get_main_queue(), ^{
  // Call the desired channel message here.
});
```

In Swift:

```swift
DispatchQueue.main.async {
  // Call the desired channel message here.
}
```

[block]: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html

[dispatch queue]: https://developer.apple.com/documentation/dispatch/dispatchqueue
