# deep-modification
A library which allows Chrome web developers to modify webpages without modifying the Document Object Model (DOM). Deep Modification seeks to understand the functionality of Chrome's native highlight function as used in its CTRL-F function. Using this understanding, Deep Modification hopes to enable developers to not just highlight text on the web page, but otherwise alter the web page without modifying the DOM.

## Useful Links
These links are an archive of links that you can peruse should you wish to become better acquainted with the project.

### Relevant Source Code Links
#### Chromium, Webkit, and Blink
A list of source code files which are useful in understanding Chrome's native CTRL + F functionality. All links redirect to some part of https://cs.chromium.org/, the main repository for Chromium's source code.

* [chromium/src/third_party/WebKit/Source/core/editing/finder/](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/)
The directory for Chromium's text finding (CTRL + F functionality). Of particular importance are [FindInPageCoordinates.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/FindInPageCoordinates.cpp) and [TextFinder.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/editing/finder/TextFinder.cpp).
