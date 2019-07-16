- Lots of documentation updates, be sure to read up on the documentation
	- API updates are also documented there https://developer.sketch.com/plugins/updates/new-in-sketch-55 (you might see some similarities with my update posts here ;D)
- Link to Matt's article
- Link out to my post on getting your plugin to subscribe to autoupdates

[< Sketch 55 Beta](https://sketchplugins.com/d/1394-what-s-new-sketch-beta-55)

##### Last Edited: July 11, 2019

---

# API Changes

### Added a `colorSpace` property and a `changeColorSpace` method to `Document`
##### More details
-  Sketch has 3 different color profiles: `Unmanaged`, `sRGB`, and `P3`
-  You can get what the current color profile is and also assign a new one.
-  Be careful with assigning a new color profile as there are two subtle, yet impactful, ways of modifying the document: _Assign_ and _Convert_
	- Assign will apply the current RGB values to the selected profile. This will subtly change the appearance of some colors.
	-  Convert will change the RGB values for the selected profile, but colors will try to appear mostly the same. Green and Red hues will be the most affected.
- For more information on color profiles read the sketch help document on [color management](https://www.sketch.com/docs/other/color-management/).

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/533/files](https://github.com/BohemianCoding/SketchAPI/pull/533/files)

##### Usage
```
let sketch = require('sketch')
let document = sketch.getSelectedDocument()

let documentColorSpace = document.colorSpace
// 'Unmanaged', 'sRGB', or 'P3'

// By default the method assigns a new color space
document.changeColorSpace(ColorSpace.sRGB)

// Pass true as an optional second argument to convert instead of assign
document.changeColorSpace(ColorSpace.P3, true)

// You can set colorSpace but you can only assign a new colorspace this way (you can't convert)
document.colorSpace = ColorSpace.P3

// When creating a new document you can also specify the color profile
let Document = sketch.Document
const p3Doc = new Document({ colorSpace: ColorSpace.P3 })

```

### Logging native structs now have nicer output in DevTools
##### More details
-  Previously, logging something like NSRange would return an unhelpful message and now it returns the location and range as you would expect.

##### Github Commit
- [https://github.com/skpm/console/commit/91791a321fe1643de09884c3ba8d75a44312eddf](https://github.com/skpm/console/commit/91791a321fe1643de09884c3ba8d75a44312eddf)

###  Expose substring in `Text.fragment`
##### More details
-  Now there is more information about how a piece of text breaks across multiple lines.
	-  You'll have access to the `rect`, `baselineOffset`, `range`, and `text` of each line
	-  `baselineOffset` is the distance from the bottom of the line fragment rectangle in which the glyph resides to the baseline

![typographic labels](https://developer.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/Art/glyph_metrics_2x.png)
	- I think its the distance from the baseline to the bottom line (frame) of the text.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/492/](https://github.com/BohemianCoding/SketchAPI/pull/492/)

##### Usage
```
test to see how a fixed width text object will break across multiple lines (not using \n)

```


### `symbolMaster.getParentSymbolMaster` used to throw an error. It will now return `undefined`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/487](https://github.com/BohemianCoding/SketchAPI/pull/487)

##### Usage
```
var sketch = require('sketch')
var document = sketch.getSelectedDocument()
var layer = document.selectedLayers.layers[0]

layer.getParentSymbolMaster()
// used throw an error but now does not!
```

### Fix setting layers of a group when the layers already had a parent
##### More details
- There was a bug with reassigning layers to a group that already had parents. You would need to first remove the parent before assigning the layers to a group.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/486](https://github.com/BohemianCoding/SketchAPI/pull/486)

##### Usage
```
// say you have some layers that have an artboard as their parent
let myLayers = [mySquare, myTriangle, myHexagon, myCircle]

// if you reassign them to be within a group
let myGroup = new Group({
	name: "My Group",
	layers: myLayers,
	parent: myArtboard
})
myGroup.adjustToFit()

// then two references of the layers would be stored, one with parents to myArtboard and one with parents to myGroup
// in order to fix this you would need to remove the reference to the parent on each of the layers before assigning them to a group

// remove the parent for each layer
myLayers.forEach(layer => layer.remove())

// This update makes it so that you can easily assign layers to a group even if those layers already have a parent set
// Now just do this

let myLayers = [mySquare, myTriangle, myHexagon, myCircle]
let myGroup = new Group({
	name: "My Group",
	layers: myLayers,
	parent: myArtboard
})

```

### Changing the `pointType` of a CurvePoint wouldn't always restore the control points
##### More Details
- There was a bug with setting the `pointType` of a line. This made it so that you could only create straight lines rather than curved ones

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/481](https://github.com/BohemianCoding/SketchAPI/pull/481)

##### Usage
```
let sketch = require('sketch')
let document = sketch.getSelectedDocument()
let page = document.selectedPage

page.layers = []

let Artboard = sketch.Artboard
let myArtboard = new Artboard({
  frame: { x: 0, y: 0, width: 48, height: 48 },
  parent: page
})

var point1 = {
    pointType: 'CurvePoint',
    curveFrom: { x: 0, y: 0 },
    curveTo: { x: 0, y: 2 },
    point: { x: 0, y: 1 },
    pointType: 'Disconnected'
}
var point2 = {
    pointType: 'CurvePoint',
    curveFrom: { x: 1, y: 0 },
    curveTo: { x: 1, y: 0 },
    point: { x: 1, y: 0 },
    pointType: 'Straight'
}

let ShapePath = sketch.ShapePath
let path = new ShapePath({
  type: ShapePath.ShapeType.Custom,
  points: [point1, point2],
  frame: { x: 0, y: 0, width: 48, height: 48 },
  style: { fills: [], borders: ['#FF0000']},
  frame: { x: 0, y: 0, width: 48, height: 48 },
  parent: myArtboard,
  closed: false
})

// should create a curved line rather than a straight one
```

### Added multiline functionality to string inputs on `UI.getInputFromUser`
##### More Details
- Previously you could only ask for a single line of input from a user via the JS API. Now you can specify a number of lines so that users can input larger amounts of text.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/475](https://github.com/BohemianCoding/SketchAPI/pull/475) (I made this PR :D)

##### Usage
```
let sketch = require('sketch')
let UI = sketch.UI

UI.getInputFromUser("What's your favorite design tool?", {
  type: UI.INPUT_TYPE.textarea,
  numberOfLines: 10,
  initialValue: 'hi',
}, (err, value) => {
  if (err) {
    // most likely the user canceled the input
    return
  }
  console.log(value)
})
```

![Example popover with a textarea input](https://user-images.githubusercontent.com/4199296/56643841-51009700-662f-11e9-8425-95969f0fd3dc.png)

### `ShapeType.Rectangle` used to be defaulted even if some points are specified when create a new ShapePath
##### More Details
- Previously, you couldn't draw a proper line with the API (you could get close but it didn't quite behave the same as a line that you could draw in Sketch). This has been now fixed.
- Here is a snippet from the API documentation

> You can only set the `shapeType` when creating a new one. Once it is created, the `shapeType` is read-only. If it is not specified and you do not specify any `points`, it will default to `ShapePath.ShapeType.Rectangle` (if you do specify some `points`, it will default to `ShapePath.ShapeType.Custom`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/468](https://github.com/BohemianCoding/SketchAPI/pull/468)

##### Usage
```
let sketch = require('sketch')
let ShapePath = sketch.ShapePath

let myLine = new ShapePath({
  name: 'myLine',
  frame: {x: 10, y: 0, width: 40, height: 100},
  style: { borders: ['#FF0000'] },
  points: [{ point: { x: 0, y: 0 }, pointType: 'Straight'}, { point: { x: 1, y: 1 }, pointType: 'Straight'}],
  parent: myArtboard
})

myLine.shapeType
// would report 'Rectangle' but now will be 'Custom' because we specified some points
// previously this would behave like a rectangle with a path inside the frame but now it behaves like a line as expected
// NEED TO CONFIRM
```
### Improve consistency by deprecating `Fill.fill` in favor of `Fill.fileType` (to match `Border.fileType` and other types)

### Find Endpoint
##### More Details
Todo

### Upcoming Additions
- Need to add


---
As always, if you have any questions or feedback be sure to comment below!
