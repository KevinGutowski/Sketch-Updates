Hello everyone! I'm back again to help document what has changed in the new beta release. 55 has only a few changes compared to the massive release of 54. That being said, definitely be sure to check out the new URL scheme that lets you target particular commands of your plugin. You can even pass in query parameters! Super neat. So neat, I've included a plugin example. Lastly, I wrote a little guide to [writing your first sketch plugin](https://medium.com/@kevingutowski/how-to-build-your-first-sketch-plugin-14c0e9e56bf0). If you know any new developers or designers interested in plugin development, be sure to send them that article!

[< Sketch 55 Beta](https://sketchplugins.com/d/1394-what-s-new-sketch-beta-55)

##### Last Edited: May 4, 2019

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

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/492/](https://github.com/BohemianCoding/SketchAPI/pull/492/)

##### Usage
```
test to see how a fixed width text object will break across multiple lines (not using \n)

```


### Improve consistency by deprecating `Fill.fill` in favor of `Fill.fillType`
##### More details
-  This was done to match `Border.fileType` and other types

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/463](https://github.com/BohemianCoding/SketchAPI/pull/463)

##### Usage
```
// Setting a fill is more consistent to setting a border
const style = new Style({
    fills: [
        {
            color: '#1234',
            fillType: Style.FillType.Color
        }
    ]
})

// Was previously
const style = new Style({
    fills: [
        {
            color: '#1234',
            fill: Style.FillType.Color
        }
    ]
})
```

### Some better logging of the `prototype` of wrapped objects
##### More details
- There was a bug in the `util.inspect` algorithm (which console uses) that makes it think the prototype of a wrapped object is a wrapped object and uses the code path for wrapped object. This has been fixed for better logging output.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/451](https://github.com/BohemianCoding/SketchAPI/pull/451)

### Upcoming Additions
- [Fixed] Changing the `pointType` of a CurvePoint wouldn't always restore the control points ([GitHub PR](https://github.com/BohemianCoding/SketchAPI/pull/481))
- [Fixed] Do not default to `ShapeType.Rectangle` if some points are specified when create a new ShapePath ([GitHub PR](https://github.com/BohemianCoding/SketchAPI/pull/468))
- [Improved] Added multiline functionality to string inputs on `UI.getInputFromUser` (I added this in! :D, [GitHub PR](https://github.com/BohemianCoding/SketchAPI/pull/475))

---
As always, if you have any questions or feedback be sure to comment below!