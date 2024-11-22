# Custom Render Objects in Flutter

## When to Use
Custom Render Objects are used when:
- Default widgets are not efficient for your custom design.
- You need fine-grained control over the rendering process, bypassing higher-level widget rebuild logic.

---

## Creating Custom Render Objects

### What is a RenderObject?
A `RenderObject` is the low-level building block in Flutter's rendering system. It controls how UI elements are laid out and painted on the screen.

---

### Practical Example: `AnimatedCircularProgressIndicator`

Let's compare the **before** and **after** implementation of a circular progress indicator using Custom Render Objects.

---

### Before: Using Default Widgets

```dart
import 'package:flutter/material.dart';

class DefaultCircularProgress extends StatelessWidget {
  final double progress;

  const DefaultCircularProgress({required this.progress, Key? key})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 100,
      height: 100,
      child: CircularProgressIndicator(
        value: progress,
        strokeWidth: 8,
        backgroundColor: Colors.grey.shade200,
      ),
    );
  }
}
```

**Issues:**
1. **Inefficiency:** High-level widget rebuilds when progress changes.
2. **Limited Customization:** Stroke behavior and animation are fixed.

---

### After: Using a Custom Render Object

```dart
import 'package:flutter/rendering.dart';
import 'package:flutter/widgets.dart';

class CustomCircularProgress extends LeafRenderObjectWidget {
  final double progress;

  const CustomCircularProgress({required this.progress, Key? key})
      : super(key: key);

  @override
  RenderObject createRenderObject(BuildContext context) {
    return _CircularProgressRenderObject(progress);
  }

  @override
  void updateRenderObject(
      BuildContext context, covariant _CircularProgressRenderObject renderObject) {
    renderObject..progress = progress;
  }
}

class _CircularProgressRenderObject extends RenderBox {
  double _progress;

  _CircularProgressRenderObject(this._progress);

  double get progress => _progress;
  set progress(double value) {
    if (_progress == value) return;
    _progress = value;
    markNeedsPaint();
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    final size = constraints.biggest;

    final Paint backgroundPaint = Paint()
      ..color = Colors.grey.shade200
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;

    final Paint progressPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;

    final double radius = size.width / 2;

    // Draw background circle
    canvas.drawCircle(offset + Offset(radius, radius), radius - 4, backgroundPaint);

    // Draw progress arc
    final double sweepAngle = 2 * 3.141592653589793 * progress;
    canvas.drawArc(
      Rect.fromCircle(center: offset + Offset(radius, radius), radius: radius - 4),
      -3.141592653589793 / 2,
      sweepAngle,
      false,
      progressPaint,
    );
  }

  @override
  void performLayout() {
    size = constraints.biggest;
  }
}
```

---

### Advantages of Custom Render Objects

1. **Fine-Grained Control:** Custom drawing logic for optimized animations and styles.
2. **Efficient Updates:** Avoid unnecessary widget rebuilds by directly updating the RenderObject.

---

### Before vs. After

| Feature                       | Default Widget                    | Custom Render Object              |
|-------------------------------|------------------------------------|------------------------------------|
| Rebuild Efficiency            | Rebuilds entire widget tree       | Updates only the render object    |
| Customization                 | Limited to provided parameters    | Fully customizable                |
| Performance                   | Moderate                          | High                              |
| Use Case                      | General-purpose UI                | Advanced, performance-critical UI |

---
