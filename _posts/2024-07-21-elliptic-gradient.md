---
title: "CSS-like Gradient in Flutter"
subtitle: "Transform Your Flutter UI with Elegant CSS-like Gradients"
date: 2024-07-21 08:00:00
categories: [ Flutter Programming ]
image: /images/flutter-logo.jpeg
tags:

  - Flutter
  - Gradient
  - Ellipse

---

In one of my recent projects, we encountered the challenge of integrating gradients, but not the usual linear or radial types—this time, 
it was elliptical gradients. An elliptical gradient is essentially a radial gradient altered with a matrix transformation.
However, accurately translating these from Figma into Flutter proved challenging;
while the CSS code rendered perfectly, Flutter’s code generation didn’t quite hit the mark. 
Here’s a glimpse of what the CSS version looked like:

```css
.radial-gradient {
    background: radial-gradient(60% 45% at 50% 50%, rgba(1, 246, 187, 0.80) 0%, rgba(1, 246, 187, 0.00) 100%);
    height: 400px;
    width: 200px;
}
```

It looks like this:

<style>
.radial-gradient {
    background: radial-gradient(60% 45% at 50% 50%, rgba(1, 246, 187, 0.80) 0%, rgba(1, 246, 187, 0.00) 100%);
    height: 400px;
    width: 200px;
}
</style>

<div class="radial-gradient"></div>

Given the complexities, we decided to develop an API that would allow us to easily implement the gradient using the same parameters.

To tackle this, we first dissected the CSS command:


_radial-gradient(60% 45% at 50% 50%, rgba(1, 246, 187, 0.80) 0%, rgba(1, 246, 187, 0.00) 100%);_


Breaking it down, it’s pretty straightforward:

* 60% 45% defines the ellipse dimensions relative to the container’s width and height.
* 50% 50% positions the center of the ellipse in the middle of the container.
* The color stops transition from a semi-transparent green to fully transparent.

Now, let’s translate this into Flutter code:

```dart
EllipticalGradient(
  backgroundColor: Color(0xFFFFFFFF),
  ellipseRelativeCenter: const Offset(0.5, 0.5),
  ellipseScale: const Scale(widthFactor: 0.6, heightFactor: 0.45),
  colors: [
    Color(0xFF01F6BB).withOpacity(0.8),
    Color(0xFF01F6BB).withOpacity(0),
  ],
  stops: const [0.0, 1.0],
)
```

For the implementation, rather than creating a custom shader—which would be overkill—we utilized Flutter’s RadialGradient and applied a GradientTransform. 
This method modifies only the gradient’s appearance without altering the entire canvas.

Here’s the core of our EllipticalGradient class:

```dart
class EllipticalGradient extends Gradient {
  final Color backgroundColor;

  /// This offset is relative to top left of the widget, [0,0] means top left and [1,1] bottom right.
  ///
  /// The range is not limited to [0,1] for both x and y axis.
  final Offset ellipseRelativeCenter;
  final Scale ellipseScale;

  const EllipticalGradient({
    required super.colors,
    required this.backgroundColor,
    required this.ellipseRelativeCenter,
    required this.ellipseScale,
    super.stops,
  });

  @override
  Shader createShader(Rect rect, {TextDirection? textDirection}) {
    return RadialGradient(
      center: Alignment(
        ellipseRelativeCenter.dx * 2 - 1,
        ellipseRelativeCenter.dy * 2 - 1,
      ),
      colors: colors
          .map((color) => Color.alphaBlend(color, backgroundColor))
          .toList(),
      radius: 1,
      stops: stops,
      transform: _EllipseTransform(
        ellipseRelativeCenter: ellipseRelativeCenter,
        ellipseScale: ellipseScale,
      ),
    ).createShader(rect, textDirection: textDirection);
  }

  @override
  EllipticalGradient scale(double factor) {
    return EllipticalGradient(
      backgroundColor: backgroundColor,
      colors: colors
          .map<Color>((Color color) => Color.lerp(null, color, factor)!)
          .toList(),
      ellipseRelativeCenter: ellipseRelativeCenter,
      ellipseScale: ellipseScale * factor,
    );
  }
}
```

There are 3 important things to note in the previous code:
* This operation `ellipseRelativeCenter.dx * 2 - 1` is just to ensure our align value will be between -1 and 1. -1 meaning start and 1 end of the view.
* The radius property is 1, this property is quite tricky cause the radius is percentage of the shortest side. Let’s imagine we have a box of width 300px and height 200px
the size of the radial gradient will be 200px. (This behavior will have an impact on how we compute the width and height factor)
* Last but not least we blend the background color with gradients color so the final result looks like there is a ColoredBox behind our gradient
without the need of a Stack

Now comes the "Mathematical" part. The signature of the transform method is:
```dart
@override
Matrix4? transform(Rect bounds, {TextDirection? textDirection})
```

The initial step involves determining the correct dimensions for width and height, guided by the widthFactor and heightFactor. 
Given that the radius is set to 1, the radius of the radial gradient will equal the length of the shortest side of the container. Consequently, we must accurately adjust these size factors to maintain the desired proportions of the elliptical gradient:

```dart
final double widthFactor;
final double heightFactor;
if (bounds.width > bounds.height) {
  heightFactor = ellipseScale.heightFactor;
  widthFactor = ellipseScale.widthFactor * bounds.width / bounds.height;
} else {
  heightFactor = ellipseScale.heightFactor * bounds.height / bounds.width;
  widthFactor = ellipseScale.widthFactor;
}
final transformMatrix = Matrix4.identity()..scale(widthFactor, heightFactor);
```

Once the size factors and the transformation matrix are initialized, the next challenge is to accurately position the ellipse. 
This is the most intricate part of the process. Initially, we determine the ellipse’s position as though no transformation has been applied. 
Subsequently, we calculate the center’s position using the scaled matrix. The discrepancy between these values indicates the necessary translation of the matrix. 
It’s crucial to note, however, that this offset is influenced by scaling, so you must adjust the translation by dividing the scaled offset by the respective size factors. With these adjustments, the ellipse will be perfectly positioned.
```dart
final Offset originalCenterOffset = Offset(
  bounds.left + bounds.width * ellipseRelativeCenter.dx,
  bounds.top + bounds.height * ellipseRelativeCenter.dy,
);

final List<double> offsetLocation = transformMatrix.applyToVector3Array([originalCenterOffset.dx, originalCenterOffset.dy, 0.0]);
final dx = originalCenterOffset.dx - offsetLocation[0];
final dy = originalCenterOffset.dy - offsetLocation[1];
return transformMatrix..translate(dx / widthFactor, dy / heightFactor);
```

You can play with this dartpad to see how it behaves.

<iframe width="800" height="800" src="https://dartpad.dev/?id=b4c4290b9356eba7a76e2827d270c10e?theme=light"></iframe>


I won’t create a dart package just for this class, so if you feel it could fit your needs, feel free to copy/paste
this code, but don’t forget to like the article if you do so ;).

Last but not least, while implementing this behavior I opened my first [bug](https://github.com/flutter/flutter/issues/145481) on Impeller, which was since fixed by the Flutter team.
Thanks guys, keep the good work!

Cheers