# What's New: Sketch 53 Beta

Hi all, I thought I would collect a list of noteable new user and api features for the Sketch 53 Beta. While the release notes do communicate what changed, I'm hoping this post can help suppliment them and provide us plugin devs a place to look for how to use the new features. Questions or feedback? Ask them below!

### App Changes

The offical list can be found at [https://www.sketchapp.com/beta/](https://www.sketchapp.com/beta/)

##### Main New Features in Sketch 53

- Performance
	- Major Performance improvements when work with complex documents containing many prototyping flows.
- Override Management
	- You can now manage which overridable properties appear directly in the Symbol’s master.
	![Manage Overrides Interface](/Users/kgutowski/Documents/Screenshots/manageoverrides.png)
-  Overrides
	-  Overrides in Symbols can now be selected via the Layer List, and in the Canvas when the Symbol instance is selected.  
	![Selectable Overides in symbol instances](/Users/kgutowski/Documents/Screenshots/Selectable Overrides.gif)
- Snapping Improvements
	- Improved snapping for rotated layers
	- Improved snapping while resizing multiple layers to indicate equal distances between the selection and other nearby layers
	- Improved snapping while resizing Artboards so they can now snap to their contents
	- Improved the way shapes resize after being flipped on their axis
- Redesigned Color Picker
	- Colors can now be named and they are shared in the libraries
	![Named Colors](/Users/kgutowski/Documents/Screenshots/Named Colors.png)
	
##### Key Enhancements

- Grid and Layout visibility can now be toggled for all selected Artboards
- Improved the way available updates to remote Libraries are reported in Sketch
- When a Symbol is detached a background layer is now created to represent the background color of the Symbol master
- Nested imported Symbols can now be detached by pressing the Option key while detaching their parent Symbol
- Added more Artboard presets for Paper Sizes A0 - A3
- Last but not least... You can now open documents and select layers directly via the Sketch url scheme
	- URL Scheme is `sketch:///path/to/file.sketch`
	- To open the file and focus on a layer `sketch:///path/to/file.sketch?centerOnLayer=LAYER_ID`
	- To open the file and set the zoom level `sketch:///path/to/file.sketch?zoom=1`
	- For testing during the beta prepend `sketch-beta` instead of `sketch`

##### Bug Fixes
- I didn't see anything too notable... but there were lots of fixes! See [https://www.sketchapp.com/beta/#bug-fixes](https://www.sketchapp.com/beta/#bug-fixes) for a full detailed list.

##### A few known issues
- Toggling between RGB and HSB color modes in the Inspector doesn’t work as expected on macOS High Sierra (10.13)
- Layer List previews do not properly reflect flip or rotation transforms
- Redesigned Color Popover may be slow to open on occasion (hopefully this will be improved because I has lots of colors!)

##### Reporting bugs

To report bugs or send feedback, simply click on Help in the Sketch Beta menu and choose Contact Customer Support.

One pretty cool thing that I never noticed - If you find a regression in this beta, please submit a detailed bug report and to say thanks, the Sketch team will add add an extra three months to your license! NICE!

---

### API CHANGES

- The document from a library will now have a proper path (either local path or the appcast URL)
	- I believe this is referring to the `.getDocument()` method of a library? I also couldn't find the PR that added this.
- Add _exportFormats_ property on _Layer_ ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/280/files))
	- You can specify `size`, `suffix`, `prefix`, and `fileFomat`
	- Valid fileFormats are 
		- `jpg`
	  - `png`
	  - `tiff`
	  - `eps`
	  - `pdf`
	  - `webp`
	  - `svg`
  - Example:

```
var sketch = require('sketch')
var document = sketch.getSelectedDocument()
var layer = document.selectedLayers.layers[0]

layer.exportFormats = [
    {
      size: '2x',
      suffix: '@2x',
    },
    {
      size: '1x',
      suffix: '@1x',
    }
]

// You will need to reload the inspector to see the changes appear there
document.sketchObject.inspectorController().reload()

console.log(layer)
/* Which logs:
[ { type: 'ExportFormat',
    fileFormat: 'png',
    prefix: undefined,
    suffix: '@2x',
    size: '2x' },
  { type: 'ExportFormat',
    fileFormat: 'png',
    prefix: undefined,
    suffix: '@1x',
    size: '1x' } ]
 */ 
```

- Add method to get the theme of Sketch ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/303))
	- Sketch has 2 themes: `light` and `dark`. If your plugin has some custom UI, it should support both as well.
	- Example:

```
 var theme = UI.getTheme()
  
  if (theme === 'dark') {
    // show UI in dark theme
  } else {
    // show UI in light theme
  }
```

- No need to specify the type when there is no choice (like Document.pages can only contain Pages, Layer.exportFormats can only contain ExportFormats, etc.)",
	- _Not sure what this means and what context it is referring to_

- Add `UI.getInputFromUser` method and deprecate the other input methods ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/276/files))
	- The same UI inputs are there (`String` and `Select`) but its moved over to the method `UI.getInputFromUser`
	- Depreciated Methods
		- `UI.getStringFromUser(message, initialValue)`
		- `UI.getSelectionFromUser(message, items, selectedItemIndex)`
	- Bonus: `Slider`, `Number`, `Color`, and `Path` inputs are coming soon
	- Examples:

```
// Default type .string
UI.getInputFromUser(
	"What's your name?",
	{
      initialValue: 'Appleseed',
    },
	(err, value) => {
		if (err) {
			// most likely the user canceled the input
			return
		}
	}
)

// Type .selection  
UI.getInputFromUser(
	"What's your favorite design tool?",
	{
		type: UI.INPUT_TYPE.selection,
		possibleValues: ['Sketch']
	},
	(err, value) => {
		if (err) {
			// most likely the user canceled the input
			return
		}
	}
) 
```

- Add some `getParent*` methods on Layer ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/284))
	- You can use `getParentPage()`, `getParentArtboard()`, `getParentSymbolMaster()`, and `getParentShape()` to quickly access higher level components
	- You can also use the `parent` property on _Layer_ that was added to go up the layer structure.
	- Examples:

```
var sketch = require('sketch')
var document = sketch.getSelectedDocument()
var layer = document.selectedLayers.layers[0]

layer.getParentPage() // gets the page
layer.getParentArtboard() // gets the containing artboard
layer.getParentSymbolMaster() // you get the idea now...

layer.parent // goes up one level
layer.parent.parent // you can continue to chain all the way up
document.parent // will be undefined

```

- Add support for text styles
	- LOTS of added functionality here
	- Depreciated Methods
		- `Text.systemFontSize`
		- `Text.alignment` (its moved to `Text.style.alignment`)
	- Added properties to `Text.style`
		- `alignment`
			- `left`, `center`, `right`, `justified`
		- `verticalAlignment`
			- `top`, `center`, `bottom`
		- `kerning`
			- default to `null` if there is none set
			- _pretty sure there is a floating point error here when you set this through the UI_
		- `lineHeight`
			- defaults to `null` if nothing is set
			- _would be nice to get the default since it varies from each font_
		- `textColor`
			- note that it can set in various formats `#000`, `#000000`, and the opacity variant `#000000FF`
		- `fontSize`
		- `textTransform`
			- `uppercase`, `lowercase`, and `none`
		- `fontFamily`
		- `fontWeight`
			- Default is 5
			- If you attempt to set the weight to something that the font doesn't support Sketch will attempt to pick the closest one.
		- `fontStyle`
			- `italic` , `normal`
			- default to `undefined`
			- setting this property to `normal` will return `undefined` if you later read it
		- `fontStretch`
			- `condensed`, `normal`
			- default to `undefined	`
			- setting this property to `normal` will return `undefined` if you later read it
		- `textUnderline`
			- `single`, `none`, `double dot`, `dot double`, `thick dash-dot`
			- default to `undefined`
			- setting this property to `double dot` or `dot double` will both return `double dot` if you later read it
			- setting this property to `none` will return `undefined` if you later read it
		- `textStrikethrough`
			- `single`, `none`, `double dot`, `dot double`, `thick dash-dot`
			- default to `undefined`
			- setting this property to `double dot` or `dot double` will both return `double dot` if you later read it
			- setting this property to `none` will return `undefined` if you later read it
	- Examples

```
var sketch = require('sketch')
var document = sketch.getSelectedDocument()
var Text = require('sketch/dom').Text
var Rectangle = require('sketch/dom').Rectangle

const text = new Text({
  text: 'blah',
  frame: new Rectangle(10, 10, 100, 100),
  parent: document.pages[0].layers[0] //If you want to bind it to an artboard
})

// Below are the default props with on the new Text object

text.style
// { type: 'Style',
// ...
// alignment: 'left'
// verticalAlignment: 'top'
// kerning: null
// lineHeight: null
// textColor: '#000000ff'
// fontSize: 12
// textTransform: 'none'
// fontFamily: 'Helvetica'
// fontWeight: 5
// fontStyle: undefined
// fontStretch: undefined
// textUnderline: undefined
// textStrikethrough: undefined
// }

```

- Add some methods to store a session variable ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/302))
	- Session variables let you store a value which is persisted when the plugin finishes to run but is _not_ persisted when Sketch closes. It is useful when you want to keep a value between plugin's runs.
	- Note that you still have `setSettingForKey` which will let you store things after closing Sketch.
	- Examples:

```
var Settings = require('sketch/settings')

Settings.sessionVariable('myVar')
// undefined

Settings.setSessionVariable('myVar', 0.1)
Settings.sessionVariable('myVar')
// 0.1

```

- Allow using setting methods even from the Run Script panel ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/302))
	- Previously you couldn't test anything from `'sketch/settings'` in the script panel and now you can!


##### Things that I couldn't get to work
- Slices ([Github PR](https://github.com/BohemianCoding/SketchAPI/pull/280/files))
	- Should work by `const slice = new Slice({ name: 'Test' })`
	- You can also pass in _exportFormats_ 

```
const artboard = new Slice({
      exportFormats: [
        {
          size: '2x',
          suffix: '@2x',
        },
      ],
    })    
```

And thats about it. Thanks for reading!
    
