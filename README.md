# Interpolate
From Unify Community Wiki
Jump to: navigation, search

# Author: Fernando Zapata (fernando@cpudreams.com)
# Contents

    1 Description
    2 Usage
    3 Code
        3.1 Interpolate.js
        3.2 Interpolate.cs

# Description

Interpolation utility functions for easing, Bezier splines, and Catmull-Rom splines. Provides consistent calling conventions across these three interpolation types. Provides low level access via individual easing functions, for example EaseInOutCirc(), Bezier(), and CatmullRom(). Provides high level access using sequence generators, NewEase(), NewBezier(), and NewCatmullRom(). Functionality is available at different levels of abstraction, making the functions easy to use and making your own high level abstractions easy to build.
Usage

You can use the low level functions similar to how you might use Unity's built-in Mathf.Lerp().

```csharp
var start = 0.0;
var distance = 3.0;
var duration = 2.0;
private var elapsedTime = 0.0;
 
function Update() {
    transform.position.y = Interpolate.EaseOutSine(start, distance,
                                                   elapsedTime, duration);
    elapsedTime += Time.deltaTime;
}
```

Instead of hard coding the easing function you can use the Ease(EaseType) function to look up a concrete easing function.

```csharp
// set using Unity's property inspector to any easing type (ex. EaseInCirc)
var easeType : EaseType; 
var ease : Function;
 
function Awake() {
  ease = Interpolate.Ease(easeType); // get easing function by type
   // ease can now be used:
  // transform.position.y = ease(start, distance, elapsedTime, duration);
}
```

You can also use higher level sequence generator functions to quickly build a reusable component. For example, this SplinePath component will move a GameObject smoothly along a path using Catmull-Rom to make sure the GameObject passes through each control point. Using the interpolation utility function this component takes less than ten lines of code (minus the visualization function OnDrawGizmos).

```csharp
// SplinePath.js
var path : Transform[]; // path's control points
var loop : boolean;
// number of nodes to generate between path nodes, to smooth out the path
var betweenNodeCount : int;
private var nodes : IEnumerator;
 
function Awake() {
  nodes = Interpolate.NewCatmullRom(path, betweenNodeCount, loop);
}
 
function Update() {
  if (nodes.MoveNext()) {
    transform.position = nodes.Current;
  }
}
 
// optional, use gizmos to draw the path in the editor
function OnDrawGizmos() {
  if (path && path.length >= 2) {
 
    // draw control points
    for (var i = 0; i < path.length; i++) {
      Gizmos.DrawWireSphere(path[i].position, 0.15);
    }
 
    // draw spline curve using line segments
    var sequence = Interpolate.NewCatmullRom(path, betweenNodeCount, loop);
    var firstPoint = path[0].position;
    var segmentStart = firstPoint;
    sequence.MoveNext(); // skip the first point
    // use "for in" syntax instead of sequence.MoveNext() when convenient
    for (var segmentEnd in sequence) {
      Gizmos.DrawLine(segmentStart, segmentEnd);
      segmentStart = segmentEnd;
      // prevent infinite loop, when attribute loop == true
      if (segmentStart == firstPoint) { break; }
    }
  }
}
```
