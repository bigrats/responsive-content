# Responsive Content

A jQuery plugin that helps you serve different content to different devices. 

Responsive Content doesn't use User Agent detection, relying only on window/screen width 
and secondary device capabilities. HTML page content is loaded as a single fragment by Ajax
and inserted into a container element. Each Ajax fragment request carries device-dependent 
query values which can be referred to on the server side to tailor the returned HTML fragment 
to the current device.

Whilst it can be used in parallel with CSS Responsive Design, it does something else completely. 
For a given screen size Responsive Design applies different styling to _identical_ content, whereas
Reponsive Content will actually load _different_ content. It can be used subtly - for instance to load 
smaller images on smaller devices - or to deliver radically different markup to distinct ranges of window width.  


### Client Side

Call it on a jQuery selector wrapper that returns a single content-container element. 

```javascript
$('#contentContainer').responsiveContent({
  widths: [ 0, 481, 768, 1025 ],
  afterLoad: function(){}, 
  forceLoad: false,
  resizeLoad: false,
  linkSelector: 'a'
})
```
Theses settings are all optional. They are shown above with their default values.

* **widths**: An array of screen width break-points. First element should be 0.

* **afterLoad**: A callback function to run secondary logic after each load. This function is called once
per logical "page", i.e. for each of these scenarios: (1) an initial page requires no ajax reload because the screen width is small 
thus the default content is sufficient; (2) an initial entails makes an ajax reload for "wider" content; (3) a subsequent click loads in new content; (4) a page is resized (and `resizeLoad:true`)

* **forceLoad**: Force an initial fragment load. Normally this only happens if getMinWidth > widths[0], but you may need to force a reload of you do not provide
default content.

* **resizeLoad**: Reload the fragment when window is resized beyond a width breakpoint. If false, the screen-width is used as the metric. If true, the window-width is used as the metric.

* **linkSelector**: The jQuery selector for which anchors to pjaxify. Must only select html A tags.

### Server Side

The query parameters passed by each Ajax request are:
```
_rescon            // Indicates that a HTML fragment should be served.
_rescon_reload     // Equals 1 when the request is a 'same page' reload, i.e. screen width exceeds the lowest (non-zero) breakpoint. 
_rescon_width      // The lower breakpoint of the window width, from a set of width breakpoints. Integer. 
_rescon_touch      // The device has touch/swipe capability. true|false
_rescon_pixelratio // 1 for normal screen. 2 for some iPads etc. Probably an integer...
```
Use these as appropriate to alter the HTML fragment that you return. How you do this is entirely up to you. 

If the `_rescon` query parameter is present in a request, render a fragment that
omits all surrounding HTML - and especially the '$('#contentContainer').responsiveContent()' function call. 
This is important in order to prevent perpetual request loops.

Make sure that the above `_rescon*` query parameters do not leak through and reappear in anchor 
href URLs in the returned HTML fragment. 

### "Mobile First" Content Flow

If the window width exceeds the 
first (non zero) breakpoint in your set of defined breakpoints, an Ajax call will be made 
and its response will replace the "default content" in the container element. 

Default content is the "smallest" version of your content, i.e. as suitable for smartphones. Alternatively, 
if you don't include such default content, you can use the `forceLoad: true` option to force an initial Ajax load.

Subsequent clicks on links will cause new "pages" (i.e. fragments, device tailored) to be loaded into the 
container element using [Pjax](https://github.com/defunkt/jquery-pjax). This causes the address bar and
history state to be updated with the link's href URL, ensuring correct back/forward button behaviour. Pjax also 
caches DOM fragments so that post-Ajax page state is maintained when navigating the history. 

NOTE: the latter paragraph only applies to browsers that support `history.pushState`. IE9 does not, for example. 
In this case the fallback behaviour is to always load the entire page as normal including its default content, 
followed by device-tailored content via Ajax if the window width exceeds the first (non zero) breakpoint.

### Cache Considerations

The approach is cache-friendly. URLs for pages and HTML fragments (with their query parts) are deterministic 
and independent of User Agent or cookies, and are thus effective as regular cache keys. 

### Window Resizing

If you set `resizeLoad:true` and resize the window width across the defined breakpoints, Ajax calls will be made and the content will 
be reloaded. This is useful in development, to check how device width variously affects the content that is loaded. 

### Example Sites

[The Chap Magazine](http://thechapmagazine.co.uk/). The 
number of articles on the front page decreases on lower window sizes, as does the actual size of 
images in articles.

[Metro Blogs](http://blogs.metro.co.uk/). Sidebars are not delivered to small screens mobile. Images in the front page top section
and in articles decrease in actual size for smaller screens. Section pages have fewer article links on smaller screens.

(These sites also use Responsive Design.)

***
Responsive Content is a fork of [Pjax](https://github.com/defunkt/jquery-pjax).
