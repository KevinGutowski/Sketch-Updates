Hiya everyone! Sketch 56 Beta is here (ya, I'm a little late however better late than never!) and I'm here to share what is new with the Sketch JS API. Now, before I dive into the details I wanted to share a few links to some helpful resources.

There have been lots and lots of documentation updates! Be sure to check them out at [https://developer.sketch.com](https://developer.sketch.com) (File format for example, has been getting some love)!  Also, my API updates are also being recorded there as well ([https://developer.sketch.com/plugins/updates/new-in-sketch-55](https://developer.sketch.com/plugins/updates/new-in-sketch-55). 

Matt Curtis (@matt_sven) wrote a fantastic two part article about Sketch plugin development and I encourage you all to check it out! He has been working hard on it and it really shows! I definitely learned a couple tricks!
[https://www.smashingmagazine.com/2019/07/build-sketch-plugin-javascript-html-css-part-1/](https://www.smashingmagazine.com/2019/07/build-sketch-plugin-javascript-html-css-part-1/)

Lastly, I wrote a post here on sketchplugins about how to publish updates to your plugin with Github Releases. It covers all about how to setup that `appcast.xml` file. [https://sketchplugins.com/d/1465-how-to-publish-updates-to-your-plugin-with-github-releases](https://sketchplugins.com/d/1465-how-to-publish-updates-to-your-plugin-with-github-releases)

If you wrote a helpful article about sketch plugin development, shoot me a message on Twitter ([@kevgski](https://twitter.com/kevgski)) and I'll include along with the next API update notes. With that being said, let's get on with the show!

[< Sketch 55 Beta](https://sketchplugins.com/d/1394-what-s-new-sketch-beta-55)

##### Last Edited: July 18, 2019

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
```javascript
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
-  Previously, logging things like `NSRange` would return an unhelpful message and now it returns the location and range as you would expect.

##### Github Commit
- [https://github.com/skpm/console/commit/91791a321fe1643de09884c3ba8d75a44312eddf](https://github.com/skpm/console/commit/91791a321fe1643de09884c3ba8d75a44312eddf)

###  Expose substring in `Text.fragment`
##### More details
-  Now there is more information about how a piece of text breaks across multiple lines.
	-  You'll have access to the `rect`, `baselineOffset`, `range`, and `text` of each line
	-  `baselineOffset` is the distance from the bottom of the line fragment rectangle in which the glyph resides to the baseline (here is a graphic to help visualize this)

![typographic labels](https://developer.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/Art/glyph_metrics_2x.png)
I think its the distance from the baseline to the bottom line (frame) of the text. Ultimately, `Text.fragment` is most useful to see how a fixed text width text object will break across multiple lines
	

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/492/](https://github.com/BohemianCoding/SketchAPI/pull/492/)

##### Usage
```javascript
let sketch = require('sketch')
let Text = sketch.Text
let Artboard = sketch.Artboard

let document = sketch.getSelectedDocument()
let page = document.selectedPage

let myArtboard = new Artboard({
  frame: {x: 0, y: 0, width: 400, height: 400},
  parent: page
})

let myText = new Text({
  text: "Our planet is a lonely speck in the great enveloping cosmic dark.",
  frame: {x: 50, y: 50, width: 100, height: 100},
  fixedWidth: true,
  parent: myArtboard
})

console.log(myText.fragments)
// [ { rect: 
//      { x: [Getter/Setter],
//        y: [Getter/Setter],
//        width: [Getter/Setter],
//        height: [Getter/Setter] },
//     baselineOffset: 3,
//     range: NSRange { location: 0, length: 36 },
//     text: 'Our planet is a lonely speck in the ' },
//   { rect: 
//      { x: [Getter/Setter],
//        y: [Getter/Setter],
//        width: [Getter/Setter],
//        height: [Getter/Setter] },
//     baselineOffset: 3,
//     range: NSRange { location: 36, length: 29 },
//     text: 'great enveloping cosmic dark.' } ]
```


### `symbolMaster.getParentSymbolMaster` used to throw an error. It will now return `undefined`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/487](https://github.com/BohemianCoding/SketchAPI/pull/487)

##### Usage
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
let sketch = require('sketch')
let ShapePath = sketch.ShapePath
let Artboard = sketch.Artboard

let document = sketch.getSelectedDocument()
let page = document.selectedPage

let myArtboard = new Artboard({
  frame: {x: 0, y: 0, width: 400, height: 400},
  parent: page
})

let myLine = new ShapePath({
  name: 'myLine',
  frame: {x: 10, y: 0, width: 40, height: 100},
  style: { borders: ['#FF0000'] },
  points: [{ point: { x: 0, y: 0 }, pointType: 'Straight'}, { point: { x: 1, y: 1 }, pointType: 'Straight'}],
  parent: myArtboard
})

console.log(myLine.shapeType)
// would report 'Rectangle' but now will be 'Custom' because we specified some points
// previously this would behave like a rectangle with a path inside the frame but now it behaves like a line as expected
```
### Improve consistency by deprecating `Fill.fill` in favor of `Fill.fileType`
##### More Details
- This change was made to match `Border.fileType` and other types

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/463](https://github.com/BohemianCoding/SketchAPI/pull/463)

##### Usage
```javascript
let sketch = require('sketch')
let Style = sketch.Style

const style = new Style({
  fills: [{
    color: '#1234',
    fill: Style.FillType.Color,
  }]
})

console.log(style.fills[0])
// Fill {
//   fillType: 'Color', //used to be 'fill'
//   color: '#11223344',
//   ...
//   enabled: true }
```

### Added a Find Method to easily query a scope of a document
##### More Details
- Last but not least is a new way to find objects that meet various criteria. It kind of reminds me a little of jquery selectors.
- The find method can take two arguments
	- A selector (the properties or criteria that you are trying to find)
	- The scope (what part of the sketch document do you want to search - by default it is the current document)
- Selectors are of type `string` and can be of the following
	- name
	- id
	- frame
	- frame.x
	- frame.y
	- frame.width
	- frame.height
	- locked
	- hidden
	- selected
	- type
	- style.fills.color
	- (would love to see these selectors expanded upon!)
- You can use these selectors in conjunction with an operator
	- `=` (equal)
	- `*=` (contains) 
	- `$=` (endswith)
	- `!=` (not equal)
	- `^=` (begins with)
	- `>=` (greater than or equal)
	- `=<` (less than or equal)
	- `>` (greater than)
	- `<` (less than)
- An example of this would be `find('[name="Rectangle"]', document)`. Some Selectors have shorthand notation
	- type: `find('ShapePath', document)`
	- id: ``find(`#${layer_id}`, document)``or `find("#91EC1D70-6A97-...-DEE84160C4F4", document)`
	- all others: `find('[{{type}}="Something"]', document)`
- Also, by default the scope is the current document so you can drop the scope if you like
	- `find('[name="Rectangle"]')`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/357/](https://github.com/BohemianCoding/SketchAPI/pull/357/)

##### Usage
```javascript
let sketch = require('sketch')
let document = sketch.getSelectedDocument()
let page = document.selectedPage

// find all Shapes in the current Document
sketch.find('Shape')

// find all Layers in the first Artboard of the selected Page
let artboard = page.layers[0]
sketch.find('*', artboard)

// find all the Layers named "Recipe"
sketch.find('[name="Recipe"]')

// More examples
document.pages = [{
	layers: [
		{ type: 'Artboard', name: 'myArtboard', frame: { x: 300 } },
		{ type: 'ShapePath', name: 'test' , frame: { x: 400 } },
		{ type: 'ShapePath', name: 'test2', frame: { x: 100 } }
	]
}]

// find by name containing
sketch.find('[name*="test"]', document)
// [ ShapePath( { name: 'test', ... }), ShapePath( { name: 'test2', ... } ]

// find by different name
sketch.find('[name!="test"]', document)
// [ Page(...), Artboard( { name: 'myArtboard', ... } ), ShapePath( { name: 'test2', ... } ]

// find by name ending with
sketch.find('[name$="2"]', document)
// [ ShapePath( { name: 'test2', ... } ]

// find by name beginning with
sketch.find('[name^="test"]', document)
// [ ShapePath( { name: 'test', ... }), ShapePath( { name: 'test2', ... } ]

// find by frame.x greater than 300
sketch.find('[frame.x>300]', document)
// [ ShapePath( { name: 'test', ... } ]

// find by frame.x greater than 200 and is a 'ShapePath'
sketch.find('ShapePath, [frame.x>200]', document)
// [ ShapePath( { name: 'test', ... } ]

// find with id
find(`#${document.pages[0].layers[1].id}`, document)
// [ ShapePath( { name: 'test', ... } ]
```

### Upcoming Additions
- Expose `PointType` enum on `ShapePath` ([https://github.com/BohemianCoding/SketchAPI/pull/561](https://github.com/BohemianCoding/SketchAPI/pull/561))
- `centerOnLayer` wasn't centering on nested layers ([https://github.com/BohemianCoding/SketchAPI/pull/562](https://github.com/BohemianCoding/SketchAPI/pull/562))

---
Thanks to Mathieu (@mathieudutour) for lots of these additions! As always, if you have any questions or feedback be sure to comment below!