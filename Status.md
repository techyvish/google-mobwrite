# Tested browsers #

  * Firefox 1.5+
  * Chrome 1
  * Safari 3 & iPhone
  * Opera 9
  * Android
  * Microsoft Internet Explorer 6 & 7

These are just the _tested_ browsers.  As long as a browser can make asynchronous HTTP requests (Ajax) it should work fine.

# Known issues #

  * When updating a textarea with remote changes, the scrollbar restores to the same percentage it had before the update. This works pretty well, but can cause drift in some cases. Need to restore based on relative position to the cursor.
  * Rare case with backwards selections. When a text input or textarea updates, any selected text in that field should remain selected. However, if the cursor was at the left side of the selection (or right side of the selection if in RTL mode), then the cursor will jump to opposite side of the selection. This is a limitation of the browser's API; one can create a selection but one can't tell it which side to place the cursor on.

# Future work #

  * MobWrite includes support for a rich-text editor widget. Although only used for synchronizing plain-text, the rich-text editor allows for yellow highlighting of incoming changes as well as optional syntax highlighting when editing code. Currently this rich-text editor exposes bugs in IE which corrupt whitespace in the data. The code is available in the 'iframe' branch of SVN.
  * A custom undo/redo stack needs to be written. If Alice makes a change, then Bob makes a change, then Alice presses undo, it should be Alice's change which is removed, not Bob's. Naturally the algorithm will have to be flexible about applying undo on a best-effort basis, since Bob's changes may conflict with Alice's undo.
  * Support for [Google Gears](http://gears.google.com/).  The differencing, matching and patching code can be fairly CPU-intensive.  When running in a browser, this may freeze the interface momentarily.  Google Gears allows this work to be pushed into a [WorkerPool](http://code.google.com/apis/gears/api_workerpool.html) so that it executes in the background.
  * The daemon version of MobWrite uses Berkeley DB.  Currently this is being used in single-thread mode.  More development would be needed to make this safely multi-threaded.

# Support #

The best place to get support is in the [MobWrite Support Group](http://groups.google.com/group/mobwrite).