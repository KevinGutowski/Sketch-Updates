Hello everyone! I'm back again to help document what has changed in the new beta release. 55 has only a few changes compared to the massive release of 54. That being said, definitely be sure to check out the new URL scheme that lets you target particular commands of your plugin. You can even pass in query parameters! Super neat. So neat, I've included a plugin example. Lastly, I wrote a little guide to [getting started writing sketch plugin development](https://medium.com/@kevingutowski/how-to-build-your-first-sketch-plugin-14c0e9e56bf0). If you know any new developers or designers interested in plugin development, be sure to send them that article!

[< Sketch 54 Beta](https://sketchplugins.com/d/1335-what-s-new-sketch-54-beta)

##### Last Edited: May 4, 2019

---

# API Changes

### URL Scheme to launch a plugin to a specific command
##### More details
-  You can use the new url scheme to target a particular command of your plugin 
    - It follows `sketch://plugin/my.plugin.identifier/my.command.identifier`
    - You can also pass in params
    - Note that this is the second URL scheme. The first was to open a particular document with `sketch://path/to/file.sketch?centerOnLayer=LAYER_ID&zoom=ZOOM_LEVEL`

- Note that the action _HandleURL_ will be triggerend with it is opened with the url scheme above
- The action context for this action contains three keys:
    - `url`: the NSURL that triggered this action
    - `path`: path: a string containing everything after `sketch://plugin` (eg. `/my.plugin.identifier/my.command.identifier`).
    - `query`: an object containing the query of the URL (eg. for `sketch://plugin/my.plugin.identifier/my.command.identifier?foo=bar&baz=qux`, `query` will be `{ foo: 'bar', baz: 'quz' }`).

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/460](https://github.com/BohemianCoding/SketchAPI/pull/460)

##### Usage
```
// Need to have manifest file setup properly to be able to trigger a function on the HandleURL Action
{
  "identifier": "com.sketchapp.examples.log-message",
  "compatibleVersion": 3,
  "bundleVersion": 1,
  "icon": "icon.png",
  "commands": [
    {
      "name": "Log Message",
      "identifier": "log-message",
      "script": "./log-message.js",
      "handlers": {
        "actions": {
          "HandleURL": "handleURL"
        }
      }
    }
  ]
}
```

```
// Then in JS File
const sketch = require('sketch')
function handleURL(context) {
    let query = context.actionContext.query
    sketch.UI.message(query.foo)
}
```

Then when the user navigates to the url `sketch://plugin/com.sketchapp.examples.log-message/log-message?foo=hello%20world` a message will appear in the app with the text "Hello World".

Note that the user in this case will need to have the plugin installed and a document already open. You can also made a new document for the user like this:

```
const sketch = require('sketch')
const Document = sketch.Document
const currentDocument = sketch.getSelectedDocument()

function handleURL(context) {
    let query = context.actionContext.query
    if (currentDocument) {
        sketch.UI.message(query.foo)
    } else {
        let document = new Document()
        sketch.UI.message(query.foo)
    }
}
```

Full plugin example can be found at [https://github.com/KevinGutowski/HandleURL_Example](https://github.com/KevinGutowski/HandleURL_Example)

### Add `isSelected` method on a CurvePoint
##### More details
-  In case the user is currently editing a path, you can check if a curve point is selected using the `curvePoint.isSelected()` method.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/445](https://github.com/BohemianCoding/SketchAPI/pull/445)

##### Usage
```
// If the user is selecting a point of a shape you can check it with

shape.points[0].isSelected()
```

Here are is an example for how you might use it in practice
```
let sketch = require('sketch')
let document = sketch.getSelectedDocument()
let selection = document.selectedLayers.layers[0]

let isAnyPointSelected = false
let pointSelectionArray = []
selection.points.forEach(point => {
  if (point.isSelected()) {
    isAnyPointSelected = true
    pointSelectionArray.push(true)
  } else {
    pointSelectionArray.push(false)
  }
})

// I created a rectangle and selected the top two points
console.log(isAnyPointSelected)
// true
console.log(pointSelectionArray)
// [true, true, false, false]
```


###  `getSelectedDocument()` could throw an error when no document was opened. Now it will return `undefined`
##### More details
-  Babel would tranform `[nativeDocument] = NSApplication.sharedApplication().orderedDocuments()` thinking that it's a proper array but it's not, it's an NSArray so it would throw an error.
- This is a great addition with the URL example from earlier. We can confidently check if there is a current document open or not.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/459](https://github.com/BohemianCoding/SketchAPI/pull/459)

##### Usage
```
let sketch = require('sketch')
let document = sketch.getSelectedDocument()

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