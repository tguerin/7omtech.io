---
title: "Newton Evolution"
subtitle: "Newton particles are now more performant and allow widget interactions, copy-paste your code directly from the configurator."
date: 2025-12-30 08:00:00
categories: [Flutter Programming]
image: /images/2023-08-06-introducing-newton-particles/newton-social-card.png
tags:
    - Flutter
    - Flutter lib
    - Particles
---

Recently I received an email from a developer asking me if he could take over
the maintenance of Newton Particles. On one side I was happy cause someone was
interested enough to keep the flame of Newton, on the other hand, I was thinking
"damn" I have so many features I still want to implement to make Newton a
rockstar particle library. If you read
[my previous post](https://7omtech.fr/2025/12/mowgli/) I had quite hell for 2
years building a poker app for Betclic. Now I'm moving to another project, I
wanted to enjoy the calm of the end of the year to get back to work on Newton.

Today I'm happy to announce Newton 0.3.0 is released.

## Particles interaction with Widgets

Now particles can interact with your app widgets. For example, if you want your
particles to bounce off your buttons, it's doable and easy. Just use a
`NewtonCollider` and wrap your widget with it and that's all:

```dart
Newton(
  effectConfigurations: [RainPreset().toConfiguration()],
  child: Stack(
    children: [
      NewtonCollider(
        borderRadius: BorderRadius.circular(20),
        child: Container(
          width: 200,
          height: 100,
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(20),
          ),
        ),
      ),
    ],
  ),
)
```

<video width="600" controls style="max-width: 100%; height: auto;">
  <source src="/videos/2025-12-30-newton-evolution/newton_collider.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Copy code from configurator

From the beginning you can directly test building an animation with the
configurator. There are still some limitations, you cannot add a post effect
animation but now you can directly copy the code when you are satisfied with an
animation.

## Introduction of Presets

To be honest my main concern when developing Newton was to have a highly
configurable but also easy to use library. The first objective seems to be met
but not the second one. You needed to tweak 30+ properties to get the perfect
animation. Having presets allows you to have a good starting point to start an
animation. If you have an idea for animations feel free to open a PR.

```dart
Newton(
  effectConfigurations: [
    RainPreset().toConfiguration(),
  ],
)
```

## API coherence

The configuration API was a bit messy. You had properties scattered around and
it wasn't always clear where to find what you needed. I reorganized everything
into logical groups like `PhysicsProperties`, `VisualProperties`,
`EmissionProperties`, and so on. It makes the code more type-safe and easier to
navigate.

I also renamed `RelativisticEffect` to `PhysicsEffect` because let's be honest,
the name was confusing.

# Performance improvements

I pushed `Box2D` to reach the limits from 150 to nearly 300 particles with
constant FPS. I won't be able to go further with `Box2D` but to be honest I'm
not satisfied...

But I did add viewport culling, which means particles outside the visible area
are automatically removed from calculations. This helps a lot when you have
particles flying off screen. I also optimized the rendering pipeline and
improved how colliders report their positions to reduce unnecessary work.

There's also a new particle pool system that reuses particles instead of
constantly creating and destroying them. It's more memory efficient and should
help with performance on lower-end devices.

## What's next

To be honest when I started this library I wanted to be able to have 1000+
particles to be able to add really neat and unique animations to your app.
`Box2D` seems to have reached a hard cap and
[I'm investigating other engines to
reach this goal](https://github.com/tguerin/newton/issues/50).

If you have ideas or want to help with this investigation, feel free to jump in.
I'm also open to suggestions for new presets or features that would make Newton
even better.

You can check out the full documentation at
[https://newton.7omtech.fr/](https://newton.7omtech.fr/) and the source code on
[GitHub](https://github.com/tguerin/newton). As always, feedback and
contributions are welcome!
