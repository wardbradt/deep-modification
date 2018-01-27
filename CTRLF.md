# The Path of CTRL F
This document is for reference to how Chrome's CTRL + F function works at the WebKit/ Blink layer as is relevant to its native highlighting function. Should anyone find any errors in this document, please reach out to me ([Ward Bradt](github.com/wardbradt)) or, better yet, create a pull request correcting any inaccuracies.

#### [`IPC_MESSAGE_ROUTED3(FrameMsg_Find, int /* request_id */, base::string16 /* search_text */, blink::WebFindOptions)`](https://cs.chromium.org/chromium/src/content/common/frame_messages.h?l=1066)
An implementation of [`IPC_MESSAGE_ROUTED3`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h?l=428) in [`frame_messages.h`](https://cs.chromium.org/chromium/src/content/common/frame_messages.h). 

This line defines the message that is sent when the user wants to search for a word on the page (enters text into the CTRL F tab in Chrome).

##### [`IPC_MESSAGE_ROUTED3`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h?l=428)
A (constant) inter-process communication message defined in [`ipc_message_macros.h`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h).

[`IPC_MESSAGE_ROUTED3`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h?l=428) is an IPC (Inter-process Communication) message. For information on how Chrome implements IPC, here is [Chromium's section of the design documents on IPC](https://www.chromium.org/developers/design-documents/inter-process-communication). For general information on IPC, here is the [Wikipedia page on IPC](https://en.wikipedia.org/wiki/Inter-process_communication).

#### [`IPC_MESSAGE_HANDLER(FrameMsg_Find, OnFind)`](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=1776)

A (constant) implementation of [`IPC_MESSAGE_HANDLER`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h?l=354) defined in [`render_frame_impl.cc`](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=1776).

##### [`IPC_MESSAGE_HANDLER`](https://cs.chromium.org/chromium/src/ipc/ipc_message_macros.h?l=354)
The definition of `IPC_MESSAGE_HANDLER` contains only the following two lines:
```
#define IPC_MESSAGE_HANDLER(msg_class, member_func) \
  IPC_MESSAGE_FORWARD(msg_class, this, _IpcMessageHandlerClass::member_func)
```

### [`OnFind`](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=6151)
The declaration appears as such:
```
void RenderFrameImpl::OnFind(int request_id,
                             const base::string16& search_text,
                             const WebFindOptions& options)
```
Although I am not certain of the intricacies of [Chromium's IPC system](https://www.chromium.org/developers/design-documents/inter-process-communication), I can intuitively surmise that `OnFind`'s parameters `request_id`, `search_text`, and `options` are equal to the `int`, `base::string16`, and `blink::WebFindOptions` parameters from `IPC_MESSAGE_ROUTED3(FrameMsg_Find, int /* request_id */, base::string16 /* search_text */, blink::WebFindOptions)`.

The (summarized) flow of `OnFind` is as follows:

Firstly:

```blink::WebPlugin* plugin = GetWebPluginForFind();```

##### [`GetWebPluginForFind`](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=7112)


### [`RequestFind`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2239)
A function of `WebLocalFrameImpl`.

If `IsFocused()` ([?](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=575&gsn=IsFocused)) or `options.find_next`<sup>f_n</sup>, the variable `result` is set to `Find(identifier, search_text, options, false /* wrapWithinFrame */, &active_now)`. This `Find` function returns whether the WebFrame contains `search_text`<sup>s_t</sup>. Further processes occur in this method, however our attention should now be directed to the `Find` function.
* f_n: if this is a "find next request" - a request to search for the next occurence of 'search_text' in the web page
* s_t: if the current web page contains the text enterered into the browser's CTRL + F tab

If 'result' is true and `options.find_next` is false, `Client()->ReportFindInPageMatchCount(identifier, 1 /* count */, false /* finalUpdate */);` is called.

### [`Find`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2298)
`Find` first runs two statements: `if (!GetFrame()) return false;` and `DCHECK(!GetFrame()->GetPage());`. Neither of these are relevant to the highlighting functionality.

It runs `GetFrame()->GetDocument()->UpdateStyleAndLayoutIgnorePendingStylesheets();` because an "up-to-date, clean tree is required for finding text in page, since it relies on TextIterator to look over the text." (According to the linked source code).

It returns `EnsureTextFinder().Find(identifier, search_text, options, wrap_within_frame, active_now);`.

#### [`EnsureTextFinder`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2413&gsn=EnsureTextFinder)
Checks if this `WebLocalFrameImpl` object contains a `TextFinder` object with `if (!text_finder_)`. If not, creates a `TextFinder` object by calling `text_finder_ = TextFinder::Create(*this)`.

Returns `text_finder_`.

### [`Find`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=114)
If `!options.find_next`<sup>f_n</sup>, calls [`UnmarkAllTextMatches()`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=803&gsn=UnmarkAllTextMatches).
* f_n: `!options.find_next` is true if a new string is searched for. If a new string is searched for, clears the stored text matches with `UnmarkAllTextMatches()`.

Else, calls `SetMarkerActive(active_match_.Get(), false);`<sup>[`SetMarkerActive` source](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)</sup>
#### [`SetMarkerActive`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)
If `!range || range->collapsed()`, returns `false`.
Returns `OwnerFrame().GetFrame()->GetDocument()->Markers().SetTextMatchMarkersActive(EphemeralRange(range), active);`
#### [`SetTextMatchMarkersActive`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/markers/DocumentMarkerController.cpp?gsn=SetMarkerActive&l=751)
Takes two arguments: `const EphemeralRange& range` and `bool active`.

