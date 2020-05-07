
# 19장 브로드캐스트 리시버와 알림
## 목차
[19.1.](#191-브로드캐스트-리시버) 브로드캐스트 리시버
  - [19.1.1.](#1911-브로드캐스트-리시버-이해) 브로드캐스트 리시버 이해
  - [19.1.2.](#1912-브로드캐스트-리시버-작성-방법) 브로드캐스트 리시버 작성 방법
  - [19.1.3.](#1913-시스템-브로드캐스트-인텐트) 시스템 브로드캐스트 인텐트
  - [19.1.4.](#1914-백그라운드-서비스-제한) 백그라운드 서비스 제한
  
[19.2.](#192-알림) 알림
  - [19.2.1.](#1921-알림의-기본-구성) 알림의 기본 구성
  - [19.2.2.](#1922-NotificationChannel) NotificationChannel
  - [19.2.3.](#1923-기본적인-알림-구성) 기본적인 알림 구성
  - [19.2.4.](#1924-알림의-다양한-구성) 알림의 다양한 구성

## 19.1. 브로드캐스트 리시버
- 메인 소프트웨어 아키택처는 컴포넌트 기반의 개발.
- 4개의 컴포넌트로 앱을 구성하는데, 우리는 이제껏 액티비티에 대해서만 살펴봄.
- 액티비티의 이용비율이 가장 높으나 서비스, 브로드캐스트 리시버, 콘텐츠 프로바이더에 대한 이해도 필수.
- 작성 방법도 중요하지만, 내부 수행 원리를 이해하는 것도 중요.

### 19.1.1. 브로드캐스트 리시버 이해
- 실제 개발하다 보면 4개의 컴포넌트 중 가장 비번하게 이용.
- 앱 내에서 브로드캐스트 리시버를 작성하거나 실행도 많이 하며, 특히 시스템의 특정 상황을 대부분 브로드캐스트 리시버로 알려줍니다.

### 19.1.2. 브로드캐스트 리시버 작성 방법
- 브로드캐스트 리시버는 BroadcastReceiver를 상속받아 작성하는 클래스.
- 클래스 내에 onReceive()라는 함수만 정의해 주면 됨.
- 브로드캐스트 리시버가 인텐트로 인해 수행될 때 자동으로 호출되는 함수.

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

- 컴포넌트 클래스이므로 AndroidManifest.xml 파일에 \<receiver\> 태크로 등록.
- \<activity\>와 마찬가지로 암시적 인텐트에 의해 실행되어야 한다면, <intent-filter>를 사용 가능.
```xml
<receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
        </receiver>
```

- sendBroadcast() 함수로 실행.
- 인텐트 부분은 다른 컴포넌트와 같음, putExtra() 함수를 이용하여 데이터를 넘길 수도 있음.
```java
Intent intent = new Intent(this, MyReceiver.class);
sendBroadcast(intent);
```

### 19.1.3. 시스템 브로드캐스트 인텐트
- 브로드캐스트 리시버는 앱 내부에서 사용하려고 자주 만들지만 시스템에서 실행하는 인텐트에 반응하여 각종 시스템 상황을 감지하려고도 자주 사용.
- 시스템에서 많은 브로드캐스트 리시버 인텐트를 발생시키는데, 이곳에서는 자주 이용되는 몇 가지를 살펴보겠음.

#### 부팅 완료
- 스마트폰의 부팅 완료 시점에 앱이 동작하여 간단한 업무를 수행하거나 서비스 등을 구동해야 할 때가 있음.
- 부팅 완료 시점에 시스템에서 발생시키는 브로드캐스트 인텐트에 반응할 브로드캐스트리시버만 정의해주면 쉽게 구현할 수 있습니다.

```xml
<receiver 
            android:name=".MyReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>
```
- 시스템에서 띄우는 인텐트의 Action 문자열과 같은 문자열로 intent-filter를 구성하여 등록.

```xml
<user-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```
- 부팅 완료 시점에 브로드캐스트 리시버가 동작하게 하려면 퍼미션이 등록되어 있어야 함.

#### 화면 On/Off
- 사용자 스마트폰의 화면이 On/Off 되는 상황은 빈번하며 개발 시 이 상황을 체크해서 특정 로직의 수행을 구동하거나 정지해야 하는 경우도 많음.
- 화면이 On/Off되는 상황에 시스템에서 브로드캐스트 인텐트를 발생해 주며 앱에서 브로드캐스트 리시버를 이용하여 이 상황을 감지할 수 있음.
- 다른 브로드캐스트 리시버와 다르게 AndroidManifest.xml 파일에 <receiver> 태그로 등록하면 실행되지 않습니다.
- 액티비티, 서비스 등의 코드에서 동적으로 등록해야만 실행됩니다.

```java
BroadcastReceiver brOn = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d("kkang", "screen on...");
    }
};
```
- 액티비티나 서비스 클래스 내부의 코드로 보면 됩니다.
- 자바 코드에서 브로드캐스트 리시버를 정의한 후 registerReceiver() 함수로 시스템에 등록하면 됩니다.

```java
registerReceiver(brOn, new IntentFilter(Intent.ACTION_SCREEN_ON));
```
- 시스템에서 띄우는 인텐트의 Action 문자열은 android.intent.action.SCREEN_ON, android.intent.action.SCREEN_OFF.
- IntentFilter 객체를 만들어 registerReceiver() 함수로 동적 등록.

```java
unregisterReceiver(brOn);
```
- 동적으로 등록한 브로드캐스트리시버는 동적으로 해제도 가능.

#### 전화 수신/발신
- 사용자가 스마트폰에서 전화를 수신 또는 발신하는 상황은 스마트폰 관점에서 중요한 상황.
- 개발자 앱이 전화 기능을 제공하지 않는다고 하더라도 이 상황을 감지해야 할 때가 있음.
***
- 우선 퍼미션 선언.
```xml
<user-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
        <user-permission android:name="android.permission.READ_PHONE_STATE"/>
```
- 각각 발신, 수신을 감지하는 퍼미션.
***
```xml
<receiver android:name=".MyReceiver">
            <intent-filter>
                <action android:name="android.intent.action.NEW_OUTGOING_CALL" />
                <action android:name="android.intent.action.PHONE_STATE" />
            </intent-filter>
        </receiver>
```
- 발신 상황이 발생하면 시스템에서 android.intent.action.NEW_OUTGOING_CALL이라는 action 문자열로 브로드캐스트 인텐트를 발생해 줍니다.
- 수신 상황이 발생하면 시스템에서 android.intent.action.PHONE_STATE라는 action 문자열로 브로드캐스트 인텐트를 발생해 줍니다.
***
```java
if (action.equals("android.intent.action.NEW_OUTGOING_CALL")) {
            String phoneNumber = intent.getStringExtra(Intent.EXTRA_PHONE_NUMBER);
        }
        else if (action.equals("android.intent.action.PHONE_STATE")) {
            Bundle bundle = intent.getExtras();
            String state = bundle.getString(TelephonyManager.EXTRA_STATE);
            String phoneNumber = bundle.getString(Intent.EXTRA_PHONE_NUMBER);
        }
```
- 발신 전화번호는 intent.getStringExtra(Intent.EXTRA_PHONE_NUMBER)로 얻을 수 있음.
- 수신 전화번호는 Bundle 객체를 통해 얻을 수 있음.
***
#### 배터리
- 스마트폰에 USB 케이블 등으로 전원이 공급되고 있는 상황과 전원 공급이 끊어진 상황 등을 앱에서 인지해야 할 때도 있습니다.
- 배터리와 관련된 각종 상황이 발생할 때 시스템에서는 앱에서 이런 상황을 인지할 수 있도록 브로드캐스트 인텐트를 발생해 줍니다.
- 배터리와 관련된 브로드캐스트 인텐트의 action 문자열은 여러 가지입니다.

문자열 | 의미
| --- | --- |
| android.intent.action.BATTERY_LOW | 스마트폰의 배터리가 낮은 상태가 되었을 때 |
| android.intent.action.BATTERY_OKAY | 스마트폰 배터리가 낮은 상태에서 벗어날 때 |
| android.intent.action.BATTERY_CHANGED | 스마트폰 배터리의 충전 상태가 변경되었을 때 |
| android.intent.action.ACTION_POWER_CONNECTED | 케이블 등으로 외부 전원공급이 연결되었을 때 |
| android.intent.action.ACTION_POWER_DISCONNECTED | 외부 전원공급이 끊어질 때 |

```java
registerReceiver(batteryReceiver, new IntentFilter(Intent.ACTION_POWER_CONNECTED));
registerReceiver(batteryReceiver, new IntentFilter(Intent.ACTION_POWER_DISCONNECTED));
```

- 여러 action 문자열을 등록하여 실행되는 브로드캐스트 리시버에서는 onReceive()함수에서 action 문자열을 추출하여 자신이 어떤 인텐트 정보로 인해 실행된 것인지 구분할 수 있습니다.
```java
@Override
public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action.equals("android.intent.action.ACTION_POWER_CONNECTED")) {
            // 
        }
        else if (action.equals("android.intent.action.ACTION_POWER_DISCONNECTED")) {
            //
        }
    }
```

- 스마트폰에서 배터리가 워낙 중요한 내용이므로 배터리와 관련된 이야기를 조금 더 살펴보겠습니다.
- 앞에서 살펴본 배터리 상태 파악은 브로드캐스트 리시버를 이용한 이벤트 모델.
- 어떨 때는 특정 코드(액티비티, 서비스)가 실행되면서 배터리 상황을 파악해야 할 때도 있습니다.
```java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
Intent batteryStatus = registerReceiver(null, ifilter);
```
- 위의 코드도 registerReceiver() 함수로 브로드캐스트 리시버를 등록하는 구문이지만, 브로드캐스트 리시버 객체 부분이 null.
- 실제 개발자가 준비한 브로드캐스트 리시버를 등록하는 구문이 아니라, 시스템의 배터리 상태의 정보값만 얻기 위한 구문.
- 이렇게 얻은 인텐트 객체에 다양한 배터리 정보가 담기게 됨.
- 가장 많이 얻는 정보가 스마트폰에 전원이 공급되고 있는지에 대한 정보.

- 다음은 현재 스마트폰에 전원이 공급되고 있는지를 파악하기 위한 구문.
```java
int status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING;
```
- isCharging 값이 true면 전원이 공급되고 있는 상황.

```java
int chargePlug = batteryStatus.getInExtra(BatteryManager.EXTRA_PLUGGED, -1);
boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
```
- 전원이 공급되고 있을 때 USB를 이용하는 건지 아니면 AC를 이용하는 건지를 파악해야 할 때도 있음.

```java
int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
float batteryPct = (level / (float)scale) * 100;
```
- 스마트폰의 배터리가 몇 퍼센트 충전된 상황인지를 파악하는 구문.

### 19.1.4. 백그라운드 서비스 제한
- 멀티 태스킹을 지원, 다른 앱이 백그라운드에서 실행될 수 있는 구조, 이는 사용자에게 의도치 않게 좋지 않은 영향을 미칠 수도 있음.
- Android Oreo부터는 백그라운드 실행에 제한을 가하고 있음.
- Android Oreo 이상의 스마트폰에서 실행될 앱이라면 백그라운드 실행 제한을 고려하여 앱을 개발해야 함.
***
- 백그라운드 실행 제한은 크게 두 가지, 백그라운드 서비스 제한과 브로드캐스트 제한.
- 이번 장이 브로드캐스트 리시버를 다루는 곳이므로 리시버와 관련된 제약사항만 살펴보겠음.
***
- 브로드캐스트 리시버의 백그라운드 제약은 암시적 인턴트에 의한 리시버의 실행 제한입니다.
- 모두 제한된 건 아니고, 앱에서 암시적 방법으로 BroadcastReceiver를 실행하는 부분만 제한되었습니다.
- 일반적으로 BroadcastReceiver는 시스템의 특정 상황(부팅 완료, SMS 수신 등)을 파악하기 위해 많이 사용되며, 이때 시스템에서 발생시키는 인텐트에 의해 실행되므로 AndroidManifest.xml 파일에 암시적 방법으로 등록해 놓습니다.
- 이 부분은 전혀 제한이 없으며 기존에 이용하던 방법대로 등록하여 사용할 수 있습니다.
```xml
<receiver
            android:name=".BootStartReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
```
- 부팅 완료 시점에 실행될 BroadcastReceiver를 AndroidManifest.xml 파일에 등록한 것이고 암시적 방법으로 실행됨.
- 위의 BroadcastReceiver는 부팅 완료 시점에 문제없이 잘 실행됨.
- 제한이 걸린 부분은 앱에서 BroadcastReceiver를 실행하기 위해 암시적 방법으로 인텐트를 발생시키는 경우.
***
```xml
<receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.example.ACTION_MY_RECEIVER" />
            </intent-filter>
        </receiver>
```
- 위와 같이 AndroidManifest.xml 파일에 등록된 리시버가 있다고 가정.
- 이를 앱의 코드에서 인텐트로 실행해 보겠습니다.
```java
Intent intent = new Intent("com.example.ACTION_MY_RECEIVER");
sendBroadcast(intent);
```
- 암시적 방법으로 리시버를 실행하는 코드.
- 그런데 Android Oreo부터는 위의 코드로는 리시버가 실행되지 않음.
- 암시적 방법으로의 인텐트 발생은 허용하지 않겠다는 로그가 표시됨.
***
- 물론 위의 AndroidManifest.xml에 등록된 리시버는 명시적 방법으로 잘 실행됩니다.
```java
Intent intent = new Intent(this, MyReceiver.class);
sendBroadcast(intent);
```
- 명시적 방법으로 실행한 예이며, 잘 실행됨.
- 결국 개발자 코드에서 암시적 인텐트로 리시버 실행을 금지한 것인데, 완전히 금지한 것은 아님.
- Android Oreo 변경사항의 주목적은 백그라운드 실행 제한.
- 즉 리시버가 백그라운드에서 암시적인 방법으로 실행되는 것을 제한한 것.
- 리시버가 백그라운드가 아니라면 암시적 인텐트로도 잘 실행됨.
***
```java
registerReceiver(new MyReceiver(), new IntentFilter("com.example.ACTION_REGISTER_RECEIVER"));
```
- 위의 코드는 리시버가 AndroidManifest.xml 파일에 등록된 것이 아니라 코드에서 registerReceiver()함수를 이용해 등록한 예.
- 즉 액티비티 같은 특정 컴포넌트가 실행되면서 등록되는 것.
- 그러므로 리시버 혼자 백그라운드에서 실행되는 것이 아니며 특정 컴포넌트 작업과 관계되어 있음.
- 그렇다면 백그라운드 실행으로 판단하지 않음.
- 따라서 위와 같은 코드에서 registerReceiver()함수로 등록한 리시버는 암시적 인텐트로 실행할 수 있음.

## 19.2. 알림
- 알림은 앱의 각종 상황을 사용자에게 알릴 목적으로 이용하는 기능.
- 보통은 액티비티 화면이 출력되어 사용자 화면에 보이고 있다면 각종 상황이나 데이터를 액티비티 화면에서 보여주면 되므로 액티비티에서는 알림이 자주 이용되지 않음.
- 하지만 서비스 컴포넌트나 브로드캐스트 리시버에서는 화면으로 사용자에게 특정 상황이나 데이터를 보여줄 수 없으므로 알림 이용 비율이 높음.

### 19.2.1. 알림의 기본 구성
- 안드로이드 버전이 변경되면서 크고 작은 내용이 계속 추가되었으며 Android Oreo에서는 NotificationChannel이라는 개념이 필수 구성요소로 추가됨.
- 또한, 안드로이드 파이에서 Person이 추가되어 MessageStyle의 Notification을 띄우는 작업이 한결 쉬워짐.
- 이처럼 내용이 자주 변경되거나 추가되는 경우 하위 호환성 문제가 생기기 마련.
- 이를 위해 support 라이브러리 NotificationCompat 클래스를 제공.
- 알림을 띄우기 위한 기본 클래스는 NotificationManager와 Notification입니다.

클래스 | 내용
| --- | --- |
| NotificationManager | 알림을 시스템에 발생시키는 SystemService |
| Notification | 알림 구성 정보를 가지는 객체 |
| NotificationCompat.Builder | 알림을 다양한 정보로 생성 |
| NotificationChannel | 알림의 관리 단위 |
- 결국 Notification 객체에 각종 정보를 담고 이 객체를 NotificationManager로 시스템에 등록하는 구조.
- 우선 NotificationManager를 getSystemService() 함수를 이용해 얻습니다.
```java
NotificationManager manager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
```

### 19.2.2. NotificationChannel
- Notification 객체는 직접 생성되지 않으며 NotificationCompat.Builder로 생성.
- Android Oreo부터는 Builder를 만드는 방법이 변경됨.
- NotificationChannel에 의해서 Builder가 생성됨.
- NotificationChannel은 일종의 알림에 대한 관리 단위로 생각하면 됨.
***
- 채널을 적용해 알림을 발생시키는 방법에 대해 살펴보겠습니다.
- NotificationChannel은 API Level 26부터 제공되므로 하위 버전에서 실행되지 않게 Build.VERSION.SDK_INT를 이용하여 버전 분기 프로그램을 작성해야 함.
```java
NotificationManager manager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
```
- Builder를 이용하여 Notification을 만들고 NotificationManager로 알림을 발생시키는 구조는 동일.
- API Level 26 이상의 폰을 목적으로 한다면 Builder 생성 시 NotificationChannel을 등록해 주어야 함.
- builder = new NotificationCompat.Builder(this, channelId);에 의해 Builder가 만들어지므로 Builder 생성 이전에 위의 코드처럼 NotificationChannel을 준비하여 적용.

### 19.2.3. 기본적인 알림 구성
- Builder의 각종 setter 함수를 이용하여 알림의 구성 정보를 명시해 주면 됨.
- 다양한 정보를 담을 수 있는데, 우선 가장 기본적인 형태의 알림을 위한 구성을 보겠음.

함수 | 내용
| --- | --- |
| setSmallIcon | 작은 아이콘 이미지 지정 |
| setWhen | 시간 |
| setContentTitle | 확장 내용의 타이틀 문자열 |
| setContentText | 확장 내용의 본문 문자열 |
| setDefaults | DEFAULT_SOUND, DEFAULT_VIBRATE, DEFAULT_LIGHTS을 함께 지정 가능 |
| setAutoCancel | 터치 시 자동 삭제 여부, true 값이 지정되면 터치 시 삭제됨 |
| setOngoing | 진행표시 여부, true 값이 설정되면 사용자가 손가락으로 밀어서 삭제 불가 |

### 19.2.4. 알림의 다양한 구성
- 
