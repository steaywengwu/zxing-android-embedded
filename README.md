赖癌患者，安卓端直接用这个，省的跑到Zxing中裁剪 修改样式等操作
![Image text](https://github.com/steaywengwu/zxing-android-embedded/blob/master/show.png)

BarcodeView就是背景
ViewfinderView就是扫描框
TextView为下方提示文字

了解了这三个View的作用之后我们就可以开始我们的自定义了,自定义界面的步骤：

1.新建一个Activity

2.把CaptureManager(改造里面的returnResult方法，不让扫码一次就关掉了界面)和DecoratedBarcodeView复制到我们自定义的Activity中

3.设置setCaptureActivity(CustomCaptureActivity.class)为我们自己的Activity

--------------------- 
调用方式：
new IntentIntegrator(this)
        // 自定义Activity，重点是这行----------------------------
        .setCaptureActivity(CustomCaptureActivity.class)
        .setDesiredBarcodeFormats(IntentIntegrator.ONE_D_CODE_TYPES)// 扫码的类型,可选：一维码，二维码，一/二维码
        .setPrompt("请对准二维码")// 设置提示语
        .setCameraId(0)// 选择摄像头,可使用前置或者后置
        .setBeepEnabled(false)// 是否开启声音,扫完码之后会"哔"的一声
        .setBarcodeImageEnabled(true)// 扫完码之后生成二维码的图片
        .initiateScan();// 初始化扫码
--------------------- 






# ZXing Android Embedded

Barcode scanning library for Android, using [ZXing][2] for decoding.

The project is loosely based on the [ZXing Android Barcode Scanner application][2], but is not affiliated with the official ZXing project.

Features:

1. Can be used via Intents (little code required).
2. Can be embedded in an Activity, for advanced customization of UI and logic.
3. Scanning can be performed in landscape or portrait mode.
4. Camera is managed in a background thread, for fast startup time.

A sample application is available in [Releases](https://github.com/journeyapps/zxing-android-embedded/releases).

By default, Android SDK 19+ is required because of `zxing:core` 3.3.2.
To support SDK 14+, downgrade `zxing:core` to 3.3.0 -
see [instructions](#adding-aar-dependency-with-gradle).

## Adding aar dependency with Gradle

From version 3.6.0, only Android SDK 19+ is supported by default.

Add the following to your `build.gradle` file:

```groovy
repositories {
    jcenter()
}

dependencies {
    implementation 'com.journeyapps:zxing-android-embedded:3.6.0'
    implementation 'com.android.support:appcompat-v7:25.3.1'   // Minimum 23+ is required
}

android {
    buildToolsVersion '27.0.3' // Older versions may give compile errors
}

```

For Android 14+ support, downgrade `zxing:core` to 3.3.0 or earlier:

```groovy
repositories {
    jcenter()
}

dependencies {
    implementation('com.journeyapps:zxing-android-embedded:3.6.0') { transitive = false }
    implementation 'com.android.support:appcompat-v7:25.3.1'   // Version 23+ is required
    implementation 'com.google.zxing:core:3.3.0'
}

android {
    buildToolsVersion '27.0.3' // Older versions may give compile errors
}

```

## Hardware Acceleration

Hardware accelation is required since TextureView is used.

Make sure it is enabled in your manifest file:

```xml
    <application android:hardwareAccelerated="true" ... >
```

## Usage with IntentIntegrator

Launch the intent with the default options:
```java
new IntentIntegrator(this).initiateScan(); // `this` is the current Activity


// Get the results:
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    IntentResult result = IntentIntegrator.parseActivityResult(requestCode, resultCode, data);
    if(result != null) {
        if(result.getContents() == null) {
            Toast.makeText(this, "Cancelled", Toast.LENGTH_LONG).show();
        } else {
            Toast.makeText(this, "Scanned: " + result.getContents(), Toast.LENGTH_LONG).show();
        }
    } else {
        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

Use from a Fragment:
```java
IntentIntegrator.forFragment(this).initiateScan(); // `this` is the current Fragment

// If you're using the support library, use IntentIntegrator.forSupportFragment(this) instead.
```

Customize options:
```java
IntentIntegrator integrator = new IntentIntegrator(this);
integrator.setDesiredBarcodeFormats(IntentIntegrator.ONE_D_CODE_TYPES);
integrator.setPrompt("Scan a barcode");
integrator.setCameraId(0);  // Use a specific camera of the device
integrator.setBeepEnabled(false);
integrator.setBarcodeImageEnabled(true);
integrator.initiateScan();
```

See [IntentIntegrator][5] for more options.

### Generate Barcode example

While this is not the primary purpose of this library, it does include basic support for
generating some barcode types:

```java
try {
  BarcodeEncoder barcodeEncoder = new BarcodeEncoder();
  Bitmap bitmap = barcodeEncoder.encodeBitmap("content", BarcodeFormat.QR_CODE, 400, 400);
  ImageView imageViewQrCode = (ImageView) findViewById(R.id.qrCode);
  imageViewQrCode.setImageBitmap(bitmap);
} catch(Exception e) {

}
```

### Changing the orientation

To change the orientation, specify the orientation in your `AndroidManifest.xml` and let the `ManifestMerger` to update the Activity's definition.

Sample:

```xml
<activity
		android:name="com.journeyapps.barcodescanner.CaptureActivity"
		android:screenOrientation="fullSensor"
		tools:replace="screenOrientation" />
```

```java
IntentIntegrator integrator = new IntentIntegrator(this);
integrator.setOrientationLocked(false);
integrator.initiateScan();
```

### Customization and advanced options

See [EMBEDDING](EMBEDDING.md).

For more advanced options, look at the [Sample Application](https://github.com/journeyapps/zxing-android-embedded/blob/master/sample/src/main/java/example/zxing/MainActivity.java),
and browse the source code of the library.

## Android Permissions

The camera permission is required for barcode scanning to function. It is automatically included as
part of the library. On Android 6 it is requested at runtime when the barcode scanner is first opened.

When using BarcodeView directly (instead of via IntentIntegrator / CaptureActivity), you have to
request the permission manually before calling `BarcodeView#resume()`, otherwise the camera will
fail to open.

## Building locally

    ./gradlew assemble

To deploy the artifacts the your local Maven repository:

    ./gradlew publishToMavenLocal

You can then use your local version by specifying in your `build.gradle` file:

    repositories {
        mavenLocal()
    }

## Sponsored by

[JourneyApps][1] - Creating business solutions with mobile apps. Fast.


## License

Licensed under the [Apache License 2.0][7]

	Copyright (C) 2012-2018 ZXing authors, Journey Mobile

	Licensed under the Apache License, Version 2.0 (the "License");
	you may not use this file except in compliance with the License.
	You may obtain a copy of the License at

	    http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.



[1]: http://journeyapps.com
[2]: https://github.com/zxing/zxing/
[3]: https://github.com/zxing/zxing/wiki/Scanning-Via-Intent
[4]: https://github.com/journeyapps/zxing-android-embedded/blob/2.x/README.md
[5]: zxing-android-embedded/src/com/google/zxing/integration/android/IntentIntegrator.java
[7]: http://www.apache.org/licenses/LICENSE-2.0
