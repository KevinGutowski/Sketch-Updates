Sketch 54 Beta launched last week and I wanted to continue with documenting new additions to the API. Lots of great additions but I think the two biggest highlights of the release are that colors, gradients, shared text styles, and shared text styles have been added to `document` and colors & gradients have been added to the new `globalAssets` property! If you find anything that I missed, add them below in the comments. Also, if you have any questions with how things work be sure to ask!

[<- Sketch 53 Beta](https://sketchplugins.com/d/1162-what-s-new-sketch-53-beta)

##### Last Edited: March 24, 2019

# API Changes

### Add `colors` and `gradients` properties on Document and `globalAssets`
##### More details
- `sketch.globalAssets` property was added
- Two new asset types were added
	- `ColorAsset`
		- name, type: String (can be null)
		- color, type: String
	- `GradientAsset`
		- name, type: String (can be null)
		- gradient, type: [Gradient](https://developer.sketchapp.com/reference/api/#gradient)

##### Github PRs
- [https://github.com/BohemianCoding/SketchAPI/pull/345](https://github.com/BohemianCoding/SketchAPI/pull/345)
- [https://github.com/BohemianCoding/SketchAPI/pull/398](https://github.com/BohemianCoding/SketchAPI/pull/398)

##### Usage 

```
var sketch = require('sketch')
var colors = sketch.globalAssets.colors
// [ { type: 'ColorAsset', name: null, color: "#febe10ff", ... }]
```

Setting Global Colors (Be sure to save a copy of your global assets before playing with this!)

- Default assets can be found here (#TODO gist link)


```
var globalAssets = sketch.globalAssets
globalAssets.colors = ['#FFFFFF']

// You can set a global color with a variety of objects
// - A native MSColorAsset
// - An MSColor
// - An MSImmutableColor
// - An NSColor
// - A hex color string

// To include a name with one of the color objects use a dictionary with `color` and `name`.
globalAssets.colors = [{name: 'white', color: '#FFFFFF'}]
```

Somewhat surprisingly this doesn't work?

```
var globalColors = sketch.globalAssets.colors
globalColors = ['#FFFFFF']
```

Other useful actions

```
// Appending/Pushing a Global Color
gloablAssets.colors = []
globalAssets.colors.push('#000000')

// globalAssets.colors returns
// [ { type: 'ColorAsset', name: null, color: '#000000ff' } ]


// Removing a Color
globalAssets.colors = [{name: 'white', color: '#FFFFFF'}, '#000000']
globalAssets.colors.splice(1,1)

// globalAssets.colors returns
// [ { type: 'ColorAsset', name: 'white', color: '#ffffffff' } ]

```

Similarily for Gradients

```
var sketch = require('sketch')
var globalGradients = sketch.globalAssets.gradients

// [ { type: 'GradientAsset',
//  name: null,
//  gradient: 
//   { gradientType: 'Linear',
//     from: [Object],
//     to: [Object],
//     aspectRatio: 1,
//     stops: [Array] } },
// ...
// ]
```

Setting Global Gradients

```
var sketch = require('sketch')
var Style = require('sketch/dom').Style
var globalAssets = sketch.globalAssets

const myGradient = {
  gradientType: Style.GradientType.Linear,
  from: { x: 0.5, y: 0},
  to: { x: 0.5, y: 1},
  stops: [
  	{ position: 0, color: '#c0ffee'},
  	{ position: 0.5, color: '#0ff1ce'},
  	{ position: 1, color: '#bada55', },
  ]
}

globalAssets.gradients = [{
  name: 'My Gradient',
  gradient: myGradient,
}]

// You can set a global gradient with a variety of objects
// - A native MSGradientAsset
// - An MSGradient
// - An dictionary of Gradient properties.
```

Moreover, these can all be found on `Document`

```
var sketch = require('sketch')
var selectedDocument = sketch.getSelectedDocument()

let documentColors = selectedDocument.colors
let documentGradients = selectedDocument.gradients
```

--

### Shared styles are now `document` properties and can be mutated

##### More Details
- Two new properties on `document`
	- `sharedLayerStyles`
	- `sharedTextStyles`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/360/files](https://github.com/BohemianCoding/SketchAPI/pull/360/files)

##### Usage

```
var sketch = require('sketch')
var selectedDocument = sketch.getSelectedDocument()

selectedDocument.sharedLayerStyles
// [ { type: 'SharedStyle',
//  id: '9CA2825D-DFE9-4A49-A3AB-1EBCA431DED9',
//  styleType: 'Style',
//  name: 'layer style example',
//  style: 
//   { type: 'Style',
//     id: 'EE3D0AA4-83C4-4049-BF91-43392DD4D4C9',
//     opacity: 1,
//  ...

selectedDocument.sharedTextStyles
// [ { type: 'SharedStyle',
//  id: 'C6F284FF-8D18-4B4B-AF08-1A9055C03B1E',
//  styleType: 'Style',
//  name: 'text style example',
//  style: 
//   { type: 'Style',
//     id: '25A3B94C-9872-4057-8F2B-97BAE4AA0E63',
//     opacity: 1,
//	...
```

Setting a Shared Style

```
var sketch = require('sketch')
var selectedDocument = sketch.getSelectedDocument()


// Setting a shared layer style
selectedDocument.sharedLayerStyles = [] // remove all layer styles

selectedDocument.sharedLayerStyles.push({
  name: 'Layer Style 1',
  style: { fills: ['#000'] }
})


// Setting a shared text style
var Text = require('sketch/dom').Text

var text = new Text({
  text: 'my text',
  style: { alignment: Text.Alignment.center }
})

selectedDocument.sharedTextStyles = [] // remove all text styles

selectedDocument.sharedTextStyles.push({
  name: 'Style 1',
  style: text.style
})
```

--

### `layer.index` can now be set

##### More Details
You can set the index of the layer to move it in the hierarchy. Note that you also have `layer.moveToFront()`, `layer.moveForward()`, `layer.moveToBack()`, and `layer.moveBackward()`

##### Github PR 
- [https://github.com/BohemianCoding/SketchAPI/pull/399](https://github.com/BohemianCoding/SketchAPI/pull/399)

##### Usage

```
var sketch = require('sketch')
var Group = sketch.Group
var page = sketch.getSelectedDocument().selectedPage

const group1 = new Group({
  parent: page
})
const group2 = new Group({
  parent: page
})
const group3 = new Group({
  parent: page
})

console.log(group1.index, group2.index, group3.index)
// 0 1 2

group1.index = 2
console.log(group1.index, group2.index, group3.index)
// 2 0 1

```

--

### Add `aspectRatio` property to Gradient

##### More Details

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/396](https://github.com/BohemianCoding/SketchAPI/pull/396)

##### Usage

--

### Add `selected` property and `getFrame` method on an Override

##### More Details

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/394](https://github.com/BohemianCoding/SketchAPI/pull/394)

##### Usage

--

### `layer.duplicate` now works on a layer with no parent

##### More Details

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/390](https://github.com/BohemianCoding/SketchAPI/pull/390)

##### Usage
--

### `symbolInstance.master` now works on an immutable instance

##### More Details

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/390](https://github.com/BohemianCoding/SketchAPI/pull/390)

##### Usage

--

### Setting `flow` as `undefined` on a Layer

##### More Details
Previously you couldn't remove a flow target. Now you can!

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/373](https://github.com/BohemianCoding/SketchAPI/pull/373)

##### Usage
```
var sketch = require('sketch')
var Artboard = require('sketch').Artboard
var Group = require('sketch').Group
var Rectangle = require('sketch').Rectangle
var selectedDocument = sketch.getSelectedDocument()

const artboard = new Artboard({
  name: 'Test1',
  frame: new Rectangle(0,0,100,100),
  parent: selectedDocument.selectedPage
})
const artboard2 = new Artboard({
  name: 'Test2',
  frame: new Rectangle(200,0,100,100),
  parent: selectedDocument.selectedPage
})

const rect = new Group({
  parent: artboard,
  frame: new Rectangle(0,0,50,50),
  flow: {
    targetId: artboard2.id,
  }
})

rect.flow = undefined
```

--

### Add `noise` and `pattern` properties on Fill

##### More Details

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/363](https://github.com/BohemianCoding/SketchAPI/pull/363)

##### Usage