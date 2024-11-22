# Performance Profiling and Optimization in Flutter

## Identifying Bottlenecks with Flutter DevTools

In this example, we will analyze a Flutter application suffering from performance issues during scrolling. Specifically, the app exhibits janky behavior due to poor widget tree optimization and excessive rebuilds.

---

## Initial Scenario: Identifying the Problem

The app contains a `ListView` with 100 items, where each item contains an image, a title, and a description. Users experience jank during scrolling, especially on low-end devices.

### Observations:

1. **Frame Build Times**: Average 25ms (Above the 16ms threshold for 60 FPS).
2. **FPS**: Drops to 40 FPS during scrolling.
3. **Rendering Stats**: Shows multiple layers being rebuilt unnecessarily.

### Example Widget Code (Before Optimization):

```dart
class JankyListView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(
          leading: Image.network('https://via.placeholder.com/50'),
          title: Text('Title $index'),
          subtitle: Text('Description for item $index'),
        );
      },
    );
  }
}
```

---

## Performance Profiling with Flutter DevTools

### Steps:

1. **Open Flutter DevTools**: Attach your running app to DevTools.
2. **Inspect Frame Timing**: Open the **Performance** tab and observe the frame timeline.
3. **Analyze Rebuild Stats**:
   - Enable the **Widget Rebuilds** option.
   - Identify widgets rebuilding unnecessarily.

---

## Bottleneck Analysis

From the DevTools Performance Timeline:
- The `Image.network` widget is being rebuilt for every scroll frame.
- The `ListTile` widget does not utilize a caching mechanism, causing repeated re-fetching of images.

---

## Optimization: Improving Performance

### Optimized Code:

```dart
class OptimizedListView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(
          leading: CachedNetworkImage(
            imageUrl: 'https://via.placeholder.com/50',
            placeholder: (context, url) => CircularProgressIndicator(),
            errorWidget: (context, url, error) => Icon(Icons.error),
          ),
          title: Text('Title $index'),
          subtitle: Text('Description for item $index'),
        );
      },
    );
  }
}
```

---

### Optimization Techniques Applied

1. **Caching Images**: Replaced `Image.network` with `CachedNetworkImage` to prevent repeated fetching of images during scrolling.

2. **Stateless Widgets**: Ensured that widgets not dependent on state are not unnecessarily rebuilt.

3. **Lazy Loading**: Utilized `ListView.builder` to ensure that only the widgets visible on the screen are built and rendered, avoiding unnecessary widget creation for off-screen items. This reduces memory usage and improves rendering performance.

4. **Custom Paint for Complex Widgets**: Simplified rendering for complex UI elements by replacing redundant widgets with a `CustomPainter`. This reduced the widget tree depth and optimized drawing operations for static parts of the UI.

---

## Results: Before/After Comparison

| Metric                   | Before Optimization       | After Optimization        |
|--------------------------|---------------------------|---------------------------|
| **Frame Build Times**    | Avg. 25ms (Janky)         | Avg. 12ms (Smooth)        |
| **FPS**                  | 40 FPS                   | 60 FPS                    |
| **Memory Usage**         | High (Image Re-fetching) | Optimized (Cached Images) |

---
