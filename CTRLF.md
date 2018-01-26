# The Path of CTRL F
This document is for reference to how Chrome's CTRL + F function works at the WebKit/ Blink layer as is relevant to its native highlighting function. Should anyone find any errors in this document, please reach out to me ([Ward Bradt](github.com/wardbradt)) or, better yet, create a pull request correcting any inaccuracies.

### [`RequestFind`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2239)
A function of `WebLocalFrameImpl`.

If `IsFocused()` ([?](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=575&gsn=IsFocused)) or `options.find_next`<sup>f_n</sup>, the variable `result` is set to `Find(identifier, search_text, options, false /* wrapWithinFrame */, &active_now)`. This `Find` function returns whether the WebFrame contains `search_text`<sup>s_t</sup>. Further processes occur in this method, however our attention should now be directed to the `Find` function.
* f_n: if this is a "find next request" - a request to search for the next occurence of 'search_text' in the web page
* s_t: if the current web page contains the text enterered into the browser's CTRL + F tab

If 'result' is true and `options.find_next` is false, `Client()->ReportFindInPageMatchCount(identifier, 1 /* count */, false /* finalUpdate */);` is called.

### [`Find`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2298)
A function of `WebLocalFrameImpl`.

`Find` first runs two statements: `if (!GetFrame()) return false;` and `DCHECK(!GetFrame()->GetPage());`. Neither of these are relevant to the highlighting functionality.

It runs `GetFrame()->GetDocument()->UpdateStyleAndLayoutIgnorePendingStylesheets();` because an "up-to-date, clean tree is required for finding text in page, since it relies on TextIterator to look over the text." (According to the linked source code).

It returns `EnsureTextFinder().Find(identifier, search_text, options, wrap_within_frame, active_now);`.

#### [`EnsureTextFinder`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2413&gsn=EnsureTextFinder)
A function of `WebLocalFrameImpl`.

Checks if this `WebLocalFrameImpl` object contains a `TextFinder` object with `if (!text_finder_)`. If not, creates a `TextFinder` object by calling `text_finder_ = TextFinder::Create(*this)`.

Returns `text_finder_`.

### [`Find`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=114)
A function of `TextFinder`.

If `!options.find_next`<sup>f_n</sup>, calls [`UnmarkAllTextMatches()`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=803&gsn=UnmarkAllTextMatches).
* f_n: `!options.find_next` is true if a new string is searched for. If a new string is searched for, clears the stored text matches with `UnmarkAllTextMatches()`.

Else, calls `SetMarkerActive(active_match_.Get(), false);`<sup>[`SetMarkerActive` source](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)</sup>
#### [`SetMarkerActive`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)
A function of `TextFinder`.
If `!range || range->collapsed()`, returns `false`.
Returns `OwnerFrame().GetFrame()->GetDocument()->Markers().SetTextMatchMarkersActive(EphemeralRange(range), active);`
#### [`SetTextMatchMarkersActive`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/markers/DocumentMarkerController.cpp?gsn=SetMarkerActive&l=751)
A function of [`DocumentMarkerController`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/markers/DocumentMarkerController.h?gsn=SetMarkerActive&l=51). Takes two arguments: `const EphemeralRange& range` and `bool active`.

