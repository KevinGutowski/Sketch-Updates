[TODO] Intro here...
[TODO] Add examples

[< Sketch 54 Beta](https://sketchplugins.com/d/1335-what-s-new-sketch-54-beta)

##### Last Edited: April 21, 2019

---

# API Changes

### URL Scheme to launch a plugin to a specific command
##### More details
-  Need more details from Sketch team but something like this `sketch://plugin/my.plugin.identifier/my.command.identifier`
- Note that the action _HandleURL_ will be triggerend with it is opened with the url scheme above
- The action context for this action contains three keys:
    - `url`: the NSURL that triggered this action
    - `path`: path: a string containing everything after `sketch://plugin` (eg. `/my.plugin.identifier/my.command.identifier`).
    - `query`: an object containing the query of the URL (eg. for `sketch://plugin/my.plugin.identifier/my.command.identifier?foo=bar&baz=qux`, `query` will be `{ foo: 'bar', baz: 'quz' }`).

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/460](https://github.com/BohemianCoding/SketchAPI/pull/460)

##### Usage
```
// example here
```

### Add `isSelected` method on a CurvePoint
##### More details
-  In case the user is currently editing a path, you can check if a curve point is selected using the `curvePoint.isSelected()` method.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/445](https://github.com/BohemianCoding/SketchAPI/pull/445)

##### Usage
```
// example here
```


###  `getSelectedDocument()` could throw an error when no document was opened. Now it will return `undefined`
##### More details
-  Babel would tranform `[nativeDocument] = NSApplication.sharedApplication().orderedDocuments()` thinking that it's a proper array but it's not, it's an NSArray so it would throw an error.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/459](https://github.com/BohemianCoding/SketchAPI/pull/459)

##### Usage
```
// example here
```


### Improve consistency by deprecating `Fill.fill` in favor of `Fill.fileType`
##### More details
-  This was done to match `Border.fileType` and other types

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/463](https://github.com/BohemianCoding/SketchAPI/pull/463)

##### Usage
```
// example here
```

### Some better logging of the `prototype` of wrapped objects
##### More details
- There was a bug in the `util.inspect` algorithm (which console uses) that makes it think the prototype of a wrapped object is a wrapped object and uses the code path for wrapped object. This has been fixed for better logging output.

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/451](https://github.com/BohemianCoding/SketchAPI/pull/451)

##### Usage
```
// example here
```


### If some points are specified when creating a new _ShapePath_ it will no longer default to  `ShapeType.Rectangle`
##### More details
-  If you do specify some `points` when creating a new _ShapePath_ object, it will default to `ShapePath.ShapeType.Custom`

##### Github PR
- [https://github.com/BohemianCoding/SketchAPI/pull/468](https://github.com/BohemianCoding/SketchAPI/pull/468)

##### Usage
```
// example here
```

### Upcoming Additions
