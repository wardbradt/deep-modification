# The Path of CTRL F
This document is for reference to how Chrome's CTRL + F function works at the WebKit/ Blink layer as is relevant to its native highlighting function. Should anyone find any errors in this document, please reach out to me ([Ward Bradt](github.com/wardbradt)) or, better yet, create a pull request correcting any inaccuracies.

### [StartFinding](https://cs.chromium.org/chromium/src/chrome/browser/ui/find_bar/find_tab_helper.h?gsn=FindTabHelper&l=28)
Called when a user presses CTRL + F (or the user-designated hotkey assigned to the find function) 


### [IPC_MESSAGE_ROUTED3(FrameMsg_Find, int /* request_id */, base::string16 /* search_text */, blink::WebFindOptions)](https://cs.chromium.org/chromium/src/content/common/frame_messages.h?l=1066)
This line defines the message that is sent when the user wants to search for a word on the page (enters text into the CTRL F tab in Chrome).

This line declares a routed IPC message which takes three arguments: `int`, `base::string16`, and `blink::WebFindOptions` objects. These three arguments will later be referenced as `request_id`, `search_text`, and `options`, respectively. These arguments will be sent to the `FrameMsg_Find` function.

#### [IPC_MESSAGE_HANDLER(FrameMsg_Find, OnFind)](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=1776)
This line handles the `IPC_MESSAGE_ROUTED3` message described previously. It then calls `OnFind` with the arguments included in the message.

For more information on Chromium's Inter-process Communication (IPC) system, read [the section of Chromium's design documents on IPC](https://www.chromium.org/developers/design-documents/inter-process-communication).

### [OnFind](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=6122)
The declaration appears as such:
```
void RenderFrameImpl::OnFind(int request_id,
                             const base::string16& search_text,
                             const WebFindOptions& options)
```
As previously mentioned, `OnFind`'s parameters `request_id`, `search_text`, and `options` are equal to the `int`, `base::string16`, and `blink::WebFindOptions` parameters from `IPC_MESSAGE_ROUTED3(FrameMsg_Find, int /* request_id */, base::string16 /* search_text */, blink::WebFindOptions)`.

The (summarized) flow of `OnFind` is as follows:

\- Calls ```blink::WebPlugin* plugin = GetWebPluginForFind();```.

\- If [`GetWebPluginForFind()`](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=6127) ("if the plugin still exists in the document"): 
  
-- If this is a find next request, navigates to the the next result.

-- Else if `!plugin->StartFind(WebString::FromUTF16(search_text), options.match_case, request_id)`, calls `SendFindReply(request_id, 0 /* match_count */, 0 /* ordinal */, gfx::Rect(), true /* final_status_update */)`, which "send\[s\] 'no results.'"

-- Returns

\- Else:

-- Calls `frame_->RequestFind(request_id, WebString::FromUTF16(search_text), options)`.


#### [GetWebPluginForFind](https://cs.chromium.org/chromium/src/content/renderer/render_frame_impl.cc?l=7080)

The flow of `GetWebPluginForFind` is as follows:

\- If `frame_->GetDocument().IsPluginDocument()`, returns `frame_->GetDocument().To<WebPluginDocument>().Plugin()`.

\- If `ENABLE_PLUGIN` is set to true in one of the header files included in render_frame_impl.cc and `plugin_find_handler_`, returns `plugin_find_handler_->container()->Plugin()`.

\- Else, returns `nullptr`.

### [RequestFind](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2239)
A function of `WebLocalFrameImpl`.

If `IsFocused()` ([?](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=575&gsn=IsFocused)) or `options.find_next`<sup>f_n</sup>, the variable `result` is set to `Find(identifier, search_text, options, false /* wrapWithinFrame */, &active_now)`. This `Find` function returns whether the WebFrame contains `search_text`<sup>s_t</sup>. Further processes occur in this method, however our attention should now be directed to the `Find` function.
* f_n: if this is a "find next request" - a request to navigate to the next occurence of `search_text` in the web page when it is known that there is at least one occurence of `search_text` in the web page.
* s_t: if the current web page contains the text enterered into the browser's CTRL + F tab

If 'result' is true and `options.find_next` is false, `Client()->ReportFindInPageMatchCount(identifier, 1 /* count */, false /* finalUpdate */);` is called.

### [Find](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2298)
`Find` first runs two statements: `if (!GetFrame()) return false;` and `DCHECK(!GetFrame()->GetPage());`. Neither of these are relevant to the highlighting functionality.

It runs `GetFrame()->GetDocument()->UpdateStyleAndLayoutIgnorePendingStylesheets();` because an "up-to-date, clean tree is required for finding text in page, since it relies on TextIterator to look over the text." (According to the linked source code).

It returns `EnsureTextFinder().Find(identifier, search_text, options, wrap_within_frame, active_now);`.

#### [EnsureTextFinder](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?l=2413&gsn=EnsureTextFinder)
Checks if this `WebLocalFrameImpl` object contains a `TextFinder` object with `if (!text_finder_)`. If not, creates a `TextFinder` object by calling `text_finder_ = TextFinder::Create(*this)`.

Returns `text_finder_`.

### [Find](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=114)
If `!options.find_next`<sup>f_n</sup>, calls [`UnmarkAllTextMatches()`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=803&gsn=UnmarkAllTextMatches).
* f_n: `!options.find_next` is true if a new string is searched for. If a new string is searched for, clears the stored text matches with `UnmarkAllTextMatches()`.

Else, calls `SetMarkerActive(active_match_.Get(), false);`<sup>[`SetMarkerActive` source](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)</sup>
#### [SetMarkerActive](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp?l=793&gsn=SetMarkerActive)
If `!range || range->collapsed()`, returns `false`.
Returns `OwnerFrame().GetFrame()->GetDocument()->Markers().SetTextMatchMarkersActive(EphemeralRange(range), active);`
#### [SetTextMatchMarkersActive](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/markers/DocumentMarkerController.cpp?gsn=SetMarkerActive&l=751)
Takes two arguments: `const EphemeralRange& range` and `bool active`.

### [OnFindReply](https://cs.chromium.org/chromium/src/content/browser/find_request_manager.cc?type=cs&q=OnFindReply&l=287)
The function that is called when the renderer wants to tell the browser it has found instances of `search_text` in the web page.

----
##### Note:
Some processes' descriptions are skipped between `OnFindReply` and `FindReply`.

### [FindReply](https://cs.chromium.org/chromium/src/content/public/browser/web_contents_delegate.h?l=405)
A function declared in `web_delegate.h` and called from [`NotifyFindReply`](https://cs.chromium.org/chromium/src/content/browser/web_contents/web_contents_impl.cc?l=5829) in web_contents_impl.cc and [`FindReply`](https://cs.chromium.org/chromium/src/extensions/browser/guest_view/web_view/web_view_guest.cc?l=564) in web_view_guest.cc.

`FindReply` is sent the result of a string search in the page. (I believe it is called when a new match is found or the search is over) The last parameter, `final_update` indicates if this is the call indicating search is over/ no more results will follow.

`FindReply` is overridden in seven places, four of which https://cs.chromium.org displays, and two of which are not in test files. These two are in [chrome/browser/ui/browser.cc](https://cs.chromium.org/chromium/src/chrome/browser/ui/browser.cc?l=1808) and [extensions/browser/guest_view/web_view/web_view_guest.cc](https://cs.chromium.org/chromium/src/extensions/browser/guest_view/web_view/web_view_guest.cc?l=564).

#### [Browser.cc's FindReply](https://cs.chromium.org/chromium/src/chrome/browser/ui/browser.cc?l=1808)
Retrieves `web_contents`'s `FindTabHelper` as `find_tab_helper` and, after checking `find_tab_helper` is not null, calls 
```
find_tab_helper->HandleFindReply(request_id,
                                   number_of_matches,
                                   selection_rect,
                                   active_match_ordinal,
                                   final_update)
```

### [HandleFindReply](https://cs.chromium.org/chromium/src/chrome/browser/ui/find_bar/find_tab_helper.cc?l=157)

\- Ensures that `number_of_matches`, `selection_rect`, and `active_match_ordinal`, have valid values.
\- Notifies "the UI, automation and any other observers that a find result was found" using the following code:
```
    last_search_result_ = FindNotificationDetails(
        request_id, number_of_matches, selection, active_match_ordinal,
        final_update);
    content::NotificationService::current()->Notify(
        chrome::NOTIFICATION_FIND_RESULT_AVAILABLE,
        content::Source<WebContents>(web_contents()),
        content::Details<FindNotificationDetails>(&last_search_result_));
```
Note that the automation provider, according to the Design Documents, is "used for UI tests for Find In Page."

----
##### Note for Later: 
Because installing extensions in Chrome's guest mode is difficult and uncommon, only the call in `web_contents_impl.cc` should be researched. 

## Notes
Here I will note functions that I believe are relevant to this process but which I have not yet encountered in my investigation of the process.

### [CreateMarkupInRect](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/frame/WebLocalFrameImpl.cpp?sq=package:chromium&l=2553)
This is a function in WebLocalFrameImpl.cc. The definition appears as such:
```
static String CreateMarkupInRect(LocalFrame* frame,
                                 const IntPoint& start_point,
                                 const IntPoint& end_point)
```

### [FrameSelection.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/FrameSelection.cpp)
### [Find Bar Design Documents](https://www.chromium.org/developers/design-documents/find-bar)
