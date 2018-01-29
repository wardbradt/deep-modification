# Relevant Links

## [`context_menus_api.h`](https://cs.chromium.org/chromium/src/chrome/browser/extensions/api/context_menus/context_menus_api.h?sq=package:chromium)
This is the source code for the header file for [chrome.contextMenus](https://cs.chromium.org/chromium/src/chrome/browser/extensions/api/context_menus/context_menus_api.h?sq=package:chromium), one of the very important chrome.* APIs used in creating Chrome extensions. It gives an idea of the main purposes of a header file for a chrome.* API. These purposes will be explained further below. Note: while most of the headers for the Chrome.* APIs have similar purposes, some (such as chrome.runtime) differ.

chrome.contextMenus provides four methods: `create`, `update`, `remove`, and `removeAll`.

`context_menus_api.h` declares four `class`es (all within the `namespace` `extensions`).
```
class ContextMenusCreateFunction : public UIThreadExtensionFunction {
 public:
  DECLARE_EXTENSION_FUNCTION("contextMenus.create", CONTEXTMENUS_CREATE)

 protected:
  ~ContextMenusCreateFunction() override {}

  // ExtensionFunction:
  ResponseAction Run() override;
};
```
The next three classes are nearly identical, with update/ Update/ UPDATE, remove/ Remove/ REMOVE, and removeAll/ RemoveAll/ REMOVEALL replacing create/ Create/ CREATE, respectively.

The classes defined in the header files of many other chrome.* APIs are similar, extending `public UIThreadExtensionFunction`, and replacing abcyXyz/ AbcXyz/ ABCXYZ with create/ Create/ CREATE.

## [`HTMLElement.cpp`](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/html/HTMLElement.cpp?sq=package:chromium)
## [Find Bar Design Documents](https://www.chromium.org/developers/design-documents/find-bar)
