---
title:  "Newton: Captivating Flutter Particle Effects"
subtitle: "Easily add visually stunning effects like rain, smoke, pulse, explosions, and more to your Flutter apps using Newton's highly configurable particle emitter package."
date:  2023-08-06 08:00:00
categories: [Flutter Programming]
image: /images/2023-08-06-introducing-newton-particles/newton-social-card.png
tags:

- Flutter
- Flutter lib
- Particles

---

I was playing around with the Flutter casual game toolkit and the [confetti animation](https://github.com/flutter/samples/blob/main/game_template/lib/src/style/confetti.dart) caught my eye.
Wouldn't be cool to easily achieve this kind of effect in a flutter app? Today, i'm eager
to introduce:
![Newton](/images/2023-08-06-introducing-newton-particles/newton-light.png)

Newton is a highly configurable particle emitter package for Flutter that allows you to create
captivating animations such as rain, smoke, explosions, and more. With Newton, you can easily add
visually stunning effects to your Flutter applications.

|                                 Rain                                  |                                  Smoke                                  |                                  Pulse                                  |                                   Explode                                   |                                   Fountain                                    |
|:---------------------------------------------------------------------:|:-----------------------------------------------------------------------:|:-----------------------------------------------------------------------:|:---------------------------------------------------------------------------:|:-----------------------------------------------------------------------------:|
| ![Rain](https://raw.github.com/tguerin/newton/main/graphics/rain.gif) | ![Smoke](https://raw.github.com/tguerin/newton/main/graphics/smoke.gif) | ![Pulse](https://raw.github.com/tguerin/newton/main/graphics/pulse.gif) | ![Explode](https://raw.github.com/tguerin/newton/main/graphics/explode.gif) | ![Fountain](https://raw.github.com/tguerin/newton/main/graphics/fountain.gif) |

## Installation

To use Newton, simply add it as a dependency in your `pubspec.yaml` file:

```yaml
dependencies:
  newton_particles: use the latest version
```

Then, run `flutter pub get` to fetch the package.

## Usage

Import the Newton package:

```dart
import 'package:newton_particles/newton_particles.dart';
```

Create a `Newton` widget and add it to your Flutter UI with the desired effects:

```dart
Newton(
    // Add any kind of effects to your UI
    // For example:
    activeEffects: [
        RainEffect(
            particleConfiguration: ParticleConfiguration(
            shape: CircleShape(),
            size: const Size(5, 5),
            color: const SingleParticleColor(color: Colors.black),
        ),
    )
],
)
```

## Enjoy the doc

A complete documentation is available [here](https://newton.7omtech.fr/), you'll find out how to create custom effects
and all the configuration properties. 

What's come next is even better!

## Configure your effect

Newton provides an effect configurator to help you tweak the effects and achieved the desired result:

<iframe width="650" height="800" src="https://newton.7omtech.fr/configure/#/"></iframe>

## Feedback

The lib is still wip, don’t hesitate to give your feedback directly on [github](https://github.com/tguerin/newton/).

Really hope you'll enjoy adding cool effects to your app. If you would like specific effects let me know!
