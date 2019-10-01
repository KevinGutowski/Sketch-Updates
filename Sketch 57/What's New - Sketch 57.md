Hiya everyone! Sketch 57 was released... well a little while ago! I took a little break, played that goose game, and am now back at these updates. You might have noticed the [new developer docs](https://developer.sketch.com) now have a new look! Luckily not much has changed with the Sketch API between 56-58 so I'll keep these updates short!

[< Sketch 56 Beta](https://sketchplugins.com/d/1476-what-s-new-sketch-beta-56) | Sketch 58 >

##### Last Edited: September 30, 2019

---

# API Changes

### Exposed `PointType` enum on `ShapePath`
##### More details
- Quick refresher, `PointType` is referring to how the Bezier handles behave at a point.
[insert image here]
-  This enum makes it easier to figure out what the different options for `PointType` are.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/561](https://github.com/BohemianCoding/SketchAPI/pull/561)

##### Usage
```javascript
let sketch = require('sketch')
let ShapePath = sketch.ShapePath

console.log(ShapePath.PointType)
/* { Undefined: 'Undefined',
	Straight: 'Straight',
	Mirrored: 'Mirrored',
	Asymmetric: 'Asymmetric',
  	Disconnected: 'Disconnected' } */
 
let PointType = ShapePath.PointType
let straight = PointType.Straight

let myLine = new sketch.ShapePath({
  name: 'myLine',
  frame: {x: 10, y: 0, width: 40, height: 100},
  style: { borders: ['#FF0000'] },
  points: [{ point: { x: 0, y: 0 }, pointType: straight}, { point: { x: 1, y: 1 }, pointType: straight}],
  closed: false
})

console.log(myLine.points[0].pointType)
// 'Straight'
```

### Fixed `centerOnLayer` for nested layers
##### More details
-  Previously, `centerOnLayer` wasn't properly repositioning the viewport over a nested layer but now it works!


##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/562](https://github.com/BohemianCoding/SketchAPI/pull/562)

---

Thanks to Jed for these additions! As always if you have any questions or feedback be sure to comment below!