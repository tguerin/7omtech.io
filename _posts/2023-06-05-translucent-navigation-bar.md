---
title:  "Flutter: Translucent Navigation"
subtitle: "Unlocking Immersive Design: Flutter's Translucent Navigation and Android Optimization - Let's Dive In!"
date:  2023-06-04 15:04:23
image: /images/flutter-logo.jpeg
categories: [Flutter]
tags:
- Flutter
- Flutter Development
---

While working on [Factify Facts](https://factify.7omtech.fr), I wanted to have a translucent navigation (status and navigation) for an immersive design in light and dark mode.
Flutter provides everything you need to achieve this goal. However having a smooth experience on Android requires some
adjustments. Let's dive in!

## Quick Takeout

If you are just interested by the solution, everything is available [here](https://github.com/tguerin/flutter_playground/tree/article-translucent-navigation). 
All commits are organized so you can understand the progression.

## Setup the Sample

First we will need to define a simple light and dark theme:

```dart
import 'package:flutter/material.dart';

class AppTheme {
  ThemeData light() {
    var colorScheme = ColorScheme.fromSeed(
      seedColor: Colors.red,
      brightness: Brightness.light,
    );
    return ThemeData(
      brightness: Brightness.light,
      appBarTheme: AppBarTheme(
        backgroundColor: colorScheme.inversePrimary,
      ),
      colorScheme: colorScheme,
      useMaterial3: true,
    );
  }

  ThemeData dark() {
    var colorScheme = ColorScheme.fromSeed(
      seedColor: Colors.red,
      brightness: Brightness.dark,
    );
    return ThemeData(
      brightness: Brightness.dark,
      appBarTheme: AppBarTheme(
        backgroundColor: colorScheme.inversePrimary,
      ),
      colorScheme: colorScheme,
      useMaterial3: true,
    );
  }
}
```

As you can see nothing fancy, we will also need a theme provider to notify when the theme is switching from light to dark:

```dart
import 'package:flutter/widgets.dart';

class ThemeProvider extends ChangeNotifier {
  bool _darkMode = false;

  bool get darkMode => _darkMode;

  set darkMode(bool value) {
    if (_darkMode != value) {
      _darkMode = value;
      notifyListeners();
    }
  }

  void invertCurrentThemeMode() {
    darkMode = !darkMode;
  }
}
```

To finish, the app will just have an AppBar, a text with the current theme, and a floating action button to invert the theme:

```dart
@override
Widget build(BuildContext context) {
  return Consumer<ThemeProvider>(builder: (context, themeProvider, child) {
    SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              "The current theme is :",
            ),
            Text(themeProvider.darkMode ? "Dark" : "Light"),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          themeProvider.invertCurrentThemeMode();
        },
        tooltip: 'Invert current theme mode',
        child: const Icon(Icons.change_circle),
      ),
    );
  });
}
```
The `SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge)` means that the navigation bar and status bar
will be drawn on top of the application.

When playing on the emulator we could imagine that everything is working fine:

{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/working-sdk-29-plus.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*Gesture navigation*
{: refdef}

However some devices do not use the gesture navigation, and if you change the settings to display 
the navigation bar:

{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/sdk-29-plus-not-working-navigation-bar.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*System navigation bar not translucent*
{: refdef}

The result is quite ok but not what we really expect.

# Going Translucent

The easiest way we can find on Internet to go translucent is to call this method:

```dart
SystemChrome.setSystemUIOverlayStyle(const SystemUiOverlayStyle(
  systemNavigationBarContrastEnforced: false,
  systemNavigationBarDividerColor: Colors.transparent,
  systemNavigationBarIconBrightness: themeProvider.darkMode ? Brightness.light : Brightness.dark,
  systemNavigationBarColor: Colors.transparent,
));
```

We can see different parameters we need to adjust:
* `systemNavigationBarContrastEnforced` -> this one means that the system “may” add a contrast color in order
to ensure the navigation bar is visible. We want to be fully transparent so better to deactivate it.
* `systemNavigationBarDividerColor` -> let define the color of the divider located a the top of the system navigation bar.
* `systemNavigationBarIconBrightness` -> as you decided to override the contrast property, you need to apply the right brightness
for the system bar navigation icons. Light brightness for dark mode and dark for light mode.
* `systemNavigationBarColor` -> this one is pretty obvious.

Let's go back to the emulator:

{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/issue-brightness-updated.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*Navigation bar icons brightness issue*
{: refdef}

As you can see the system navigation bar is transparent BUT the icon brightness is not updating properly.

## Introducing AnnotatedRegion

To solve this issue you need to use an `AnnotatedRegion`, this will ensure the layer tree will use the updated `SystemUiOverlayStyle`.
I will write a dedicated post to explain in details why you need to do this.

```dart
@override
  Widget build(BuildContext context) {
    return Consumer<ThemeProvider>(builder: (context, themeProvider, child) {
      return AnnotatedRegion(
          value: SystemUiOverlayStyle(
            systemNavigationBarContrastEnforced: false,
            systemNavigationBarDividerColor: Colors.transparent,
            systemNavigationBarIconBrightness: themeProvider.darkMode ? Brightness.light : Brightness.dark,
            systemNavigationBarColor: Colors.transparent,
          ),
          child: Scaffold(...
```
{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/working-sdk-29-plus-navigation-bar.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*System navigation bar is translucent on Android 29+*
{: refdef}

Job is done, I finished my day! Hold on, hold on, does it work for all API levels? Let's test on 
api level 28.

{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/not-working-sdk-28-navigation-bar.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*System navigation bar not working on Android 28 and less*
{: refdef}


Hmm not working as expected. The reason behind this issue is quite simple and can be found in the documentation of
`SystemUiMode.edgeToEdge`.

```dart
/**
 * Available starting at SDK 29 or Android 10. Earlier versions of Android 
 * will not be affected by this setting.
 */
```

This means that the app will not be drawn below the system navigation bar if API level < 29.

## Faking Translucence

To be able to have a similar design for all API levels there is a simple trick. We are going to have
the system navigation bar the same color as the background only for API levels < 29. As i don't like to have reference to 
sdk levels in my code, i will have a DeviceFeature wrapper class that will let me know if the `edgeToEdge` feature is available.

For this i'll need to use the `device_info_plus` plugin:

```dart
import 'dart:io';

import 'package:device_info_plus/device_info_plus.dart';

class DeviceFeature {

  DeviceFeature._internal();

  static final DeviceFeature _singleton = DeviceFeature._internal();

  factory DeviceFeature() {
    return _singleton;
  }

  final _deviceInfoPlugin = DeviceInfoPlugin();

  AndroidDeviceInfo? _androidDeviceInfo;

  Future<void> init() async {
    _androidDeviceInfo = await _deviceInfoPlugin.androidInfo;
  }

  bool isEdgeToEdgeAvailable() {
    return Platform.isAndroid && _androidDeviceInfo != null && _androidDeviceInfo!.version.sdkInt >= 29;
  }
}
```

The final code for our `AnnotatedRegion` becomes:

```dart
@override
Widget build(BuildContext context) {
  return Consumer<ThemeProvider>(builder: (context, themeProvider, child) {
    final edgeToEdgeAvailable = DeviceFeature().isEdgeToEdgeAvailable();
    final navigationBarColor = edgeToEdgeAvailable
        ? Colors.transparent
        : Theme.of(context).colorScheme.background;
    return AnnotatedRegion(
        value: SystemUiOverlayStyle(
          systemNavigationBarContrastEnforced: false,
          systemNavigationBarDividerColor: Colors.transparent,
          systemNavigationBarIconBrightness: themeProvider.darkMode ? Brightness.light : Brightness.dark,
          systemNavigationBarColor: navigationBarColor,
        ),
      ...
```

Tadaa everything works fine for all API levels!

{:refdef: style="text-align: center;"}
![](/images/2023-06-05-translucent-navigation-bar/working-sdk-28-navigation-bar.gif){:style="display:block; margin-left:auto; margin-right:auto"}
*System navigation bar working on Android 28 and less*
{: refdef}

## Theme is Everything

Even if everything is working well, there is something i can't get rid off my head. I don’t like to have 
theme attributes hard coded in my UI. Isn't there a better way to handle this issue?

There is a `systemOverlayStyle` attribute in the `AppBarTheme`. We can leverage this and define everything at the theme level.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class AppTheme {
  ThemeData light(bool edgeToEdgeAvailable) {
    var colorScheme = ColorScheme.fromSeed(
      seedColor: Colors.red,
      brightness: Brightness.light,
    );
    return ThemeData(
      brightness: Brightness.light,
      appBarTheme: AppBarTheme(
        backgroundColor: colorScheme.inversePrimary,
        systemOverlayStyle: SystemUiOverlayStyle(
          statusBarIconBrightness: Brightness.dark,
          statusBarColor: Colors.transparent,
          systemStatusBarContrastEnforced: false,
          systemNavigationBarContrastEnforced: false,
          systemNavigationBarDividerColor: Colors.transparent,
          systemNavigationBarIconBrightness: Brightness.dark,
          systemNavigationBarColor:
              edgeToEdgeAvailable ? Colors.transparent : colorScheme.background,
        ),
      ),
      colorScheme: colorScheme,
      useMaterial3: true,
    );
  }

  ThemeData dark(bool edgeToEdgeAvailable) {
    var colorScheme = ColorScheme.fromSeed(
      seedColor: Colors.red,
      brightness: Brightness.dark,
    );
    return ThemeData(
      brightness: Brightness.dark,
      appBarTheme: AppBarTheme(
        backgroundColor: colorScheme.inversePrimary,
        systemOverlayStyle: SystemUiOverlayStyle(
          statusBarIconBrightness: Brightness.light,
          statusBarColor: Colors.transparent,
          systemStatusBarContrastEnforced: false,
          systemNavigationBarContrastEnforced: false,
          systemNavigationBarDividerColor: Colors.transparent,
          systemNavigationBarIconBrightness: Brightness.light,
          systemNavigationBarColor:
              edgeToEdgeAvailable ? Colors.transparent : colorScheme.background,
        ),
      ),
      colorScheme: colorScheme,
      useMaterial3: true,
    );
  }
}
```

As you can see we also have to define the status bar color and contrast enforcement to have a smooth integration
for status and navigation bar. Now our main build is as simple as:

```dart
@override
Widget build(BuildContext context) {
  return Consumer<ThemeProvider>(builder: (context, themeProvider, child) {
    final edgeToEdgeAvailable = DeviceFeature().isEdgeToEdgeAvailable();
    final themeMode =
      themeProvider.darkMode ? ThemeMode.dark : ThemeMode.light;
    final appTheme = AppTheme();
    SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
    return MaterialApp(
      title: 'Translucent Navigation',
      themeMode: themeMode,
      theme: appTheme.light(edgeToEdgeAvailable),
      darkTheme: appTheme.dark(edgeToEdgeAvailable),
      home: const MyHomePage(title: 'Translucent Navigation'),
    );
  });
}
```

I hope you enjoyed this article. Don't hesitate to leave a comment or ask questions!

May Flutter be with you!
