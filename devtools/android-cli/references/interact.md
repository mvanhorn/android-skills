Run `android layout --help` and `android screen --help`.

## UI Dump

`android layout` returns a nested JSON tree of the UI elements on screen.
Elements nest under `children`. `android layout --flat` returns a flat JSON
list instead of a tree.

`android layout --full` returns the full tree, including non-interactive and
hidden elements. `android layout --no-idle` does not wait for an idle state.
When layout asks you to ignore layout events, re-run with `--no-idle`.

Each JSON object represents a UI element in the Android app. The following
properties may be present:

- `class` - the Android class of the element
- `text` - any literal text the element contains
- `resource-id` - the Android resource id used to refer to the element
- `content-desc` - a description of a UI element for use by accessibility tools
- `interactions` - the set of user interactions the element supports. May contain one or more of: `CHECKABLE`, `CLICKABLE`, `FOCUSABLE`, `SCROLLABLE`, `LONG_CLICKABLE`, `PASSWORD`, `EDITABLE`, `TEXT_SELECTABLE`
- `state` - the set of states the element is in. May contain one or more of `CHECKED`, `FOCUSED`, `SELECTED`
- `bounds` - the screen coordinates of the bounding rectangle of the element, in the format `[min X,min Y][max X, max Y]`
- `center` - the screen coordinates of the center of the element, in the format `[x,y]`
- `off-screen` - if true, the element is in the UI hierarchy but not visible; it may require scrolling to view.
- `hidden` - if true, the element is hidden. Included when using `--full`
- `children` - nested child elements in the default tree. Omitted by `--flat`

Use `layout` as a primary means of examining an Android app. The default output
is a nested tree. Use `--flat` for a flat list, and `--full` when you need
non-interactive or hidden elements.

`layout` may fail due to the app displaying a WebView or animation; in these
cases, use `android screen capture --annotate` to inspect the app. This failure
will likely resolve after navigating away from the current screen. If layout
events never go idle, re-run with `android layout --no-idle`.

## Screenshot

`android screen capture -o <file path>` saves a PNG of the current device screen
to `<file path>`

Use `screen capture` as a secondary means of examining an Android app
Examples:

- Understanding the content of an on-screen image
- Looking at a `WebView` (web content does not always appear in the ui dump)
- Trying to find a UI element by its visual appearance

**IMPORTANT** : Always *VISUALLY* examine the PNG image returned from `android
screen` BEFORE doing anything else.

## Annotated Screenshot

`android screen capture --annotate -o <file path>`
`android screen resolve --screen <path> --string <string>`

The `--annotate` command adds numerical labels and bounding boxes around UI
elements. Use this command to locate UI elements that cannot be located in the
`layout` output.

**IMPORTANT** : When using `android screen --annotate`, always *VISUALLY* examine
the resulting PNG file.

To refer to these labels in input commands, use `screen resolve` to convert
labels into coordinates:

`android screen resolve --screen <file path> --string "#3"` returns `<x coord of
region 3> <y coord of region 3>`

To save turns, you can combine shell commands:

`adb shell input $(android screen resolve --screen screen.png --string "tap #34")`

This command taps on region #34 from `screen.png`

## Input

Use `adb shell input` for interacting with Android devices. Refer to the
`"interactions"` property of an element for what interactions can be performed
on a particular element.

Interact with UI elements with their `center` coordinate or their `bounds`
coordinates:
`json
{
"class": "android.widget.Button",
"interactions": ["CLICKABLE"],
"bounds": "[138,9][167,38]",
"center": "[152,23]"
}`

To tap on this button, you would execute `adb shell input tap 152 23`. This taps
the center.

    {
      "class": "com.example.ui.ScrollableList",
      "interactions": ["SCROLLABLE"],
      "bounds": "[100,200][400,600]",
      "center": "[250,400]"
    }

To scroll down on this list, you would execute `adb shell input swipe 250 400
250 200 500`. This swipes from the center to the top over 500ms.

### Text Input

To enter text, ensure the field has `"FOCUSED"` in its `"state"` list and
execute `adb shell input text "<text>"`. An `"EDITABLE"` value in
`"interactions"` identifies a text field.

- **Spaces (`%s`):** Replace spaces with `%s` (Android ignores text after raw spaces): `adb shell input text "Hello%sworld"` (enters `"Hello world"`).
- **Special characters:** Escape shell metacharacters (`&`, `$`, `(`, `)`, `!`): `adb shell input text "AT\&T"` or `adb shell input text 'Price:\$10'`.
- **Submit / Enter:** `adb shell input keyevent 66`

## Android Interaction Rules

1. Always ensure text input fields have `"FOCUSED"` in their `"state"` list before entering text. An `"EDITABLE"` value in `"interactions"` identifies a text field.
2. If an element has `"SCROLLABLE"` in its `"interactions"` list, try scrolling it when looking for missing UI elements.
3. Always scroll slowly when executing scroll inputs. The 5th argument to `adb
   shell input swipe` controls scroll duration.
4. Content may take time to load; if a `layout` is missing information after you take an action, wait a few seconds, then run `layout` again. If layout events never go idle, run `layout --no-idle`.