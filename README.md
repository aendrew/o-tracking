# Origami Tracking component

This module should be included on your product to make sending tracking requests to the Spoor API easy.

## The Spoor ecosystem
![ScreenShot](https://rawgit.com/Financial-Times/o-tracking/master/resources/images/ngda-system-design.svg)

## Usage

### Requirement: Polyfills & Doesn't cut the mustard
_Applies to both quickstart examples below_


Add the [polyfill](https://cdn.polyfill.io/) service to your site to make sure o-tracking runs in as many browsers as possible.

Require the noscript version of o-tracking incase the browser doesn't cut the mustard.

Checkout the [Origami spec](http://origami.ft.com/docs/developer-guide/using-modules/#core-vs-enhanced-experience) for the correct CTM logic, [html and css changes](http://origami.ft.com/docs/developer-guide/using-modules/#styles-for-fallbacks-and-enhancements) needed.



o-tracking should have the following piece of html added, with the correct server and data filled in.
```html
<div class="o-tracking o--if-no-js" data-o-component="o-tracking">
	<div style="background:url('http://server?data={}');"></div>
</div>
```

### Recommended implementation using the build service

Use an onload handler to check when o-tracking has loaded and then init.

```js
// CTM
if (cutsTheMustard) {
    var o = document.createElement('script');
    o.async = o.defer = true;
    o.src = 'https://build.origami.ft.com/v2/bundles/js?modules=o-tracking';
    var s = document.getElementsByTagName('script')[0];
    if (o.hasOwnProperty('onreadystatechange')) {
        o.onreadystatechange = function() {
            if (o.readyState === "loaded") {
                otrackinginit();
            }
        };
    } else {
        o.onload = otrackinginit;
    }
    s.parentNode.insertBefore(o, s);
}
```

The `otrackinginit` function, used above, would have function calls to setup o-tracking and likely send a page view event. e.g.

```js
function otrackinginit() {
    var oTracking = Origami['o-tracking'];
    // Setup
    oTracking.init({...config...});
    // Page
    oTracking.page({...config...});
}
```

### Alternative implementation using require

Use the build service or require locally to load o-tracking and init manually.
```js
var oTracking = require('o-tracking');
```

```js
if (cutsTheMustard) {
    // Setup
    oTracking.init({...config...});
    // Page
    oTracking.page({...config...});
}
```

### Example implementations

- [ft.com](docs/ftcom_example.md)
- [membership](docs/membership_example.md)

#### Full example
```html
<!DOCTYPE html>
<!-- Add core class to head tag -->
<head class="core">
<head>
    <title>Full example</title>
    <!-- Add CTM styles -->
    <style type="text/css">
        /* Hide any enhanced experience content when in core mode, and vice versa. */
        .core .o--if-js,
        .enhanced .o--if-no-js { display: none !important; }
    </style>
    <!-- Add CTM check -->
    <script>
        var cutsTheMustard = ('querySelector' in document && 'localStorage' in window && 'addEventListener' in window);
        if (cutsTheMustard) {
        // Swap the 'core' class on the HTML element for an 'enhanced' one
        // We're doing it early in the head to avoid a flash of unstyled content
        document.documentElement.className = document.documentElement.className.replace(/\bcore\b/g, 'enhanced');
        }
    </script>
    <!-- Add Polyfil service -->
    <script src="https://cdn.polyfill.io/v1/polyfill.min.js"></script>
    <!-- INIT and make a page request -->
    <script>
        function otrackinginit() {
            var config_data = {
                server: 'https://spoor-api.ft.com/px.gif',
                context: {
                    product: 'ft.com'
                },
                user: {
                    ft_session: otracking.utils.getValueFromCookie(/FTSession=([^;]+)/)
                }
            }
            // oTracking
            var oTracking = Origami['o-tracking'];
            // Setup
            oTracking.init(config_data);
            // Page
            otracking.page({
                content: {
                    asset_type: 'page'
                }
            });
        }
        if (cutsTheMustard) {
            var o = document.createElement('script');
            o.async = o.defer = true;
            o.src = 'https://build.origami.ft.com/v2/bundles/js?modules=o-tracking';
            var s = document.getElementsByTagName('script')[0];
            if (o.hasOwnProperty('onreadystatechange')) {
                o.onreadystatechange = function() {
                    if (o.readyState === "loaded") {
                        otrackinginit();
                    }
                };
            } else {
                o.onload = otrackinginit;
            }
            s.parentNode.insertBefore(o, s);
        }
    </script>
</head>
<body>
<!-- Add fallback if browsers don't cut the mustard -->
<div class="o-tracking o--if-no-js" data-o-component="o-tracking">
    <div style="background:url('https://spoor-api.ft.com/px.gif?data=%7B%22category%22:%22page%22,%20%22action%22:%22view%22,%20%22system%22:%7B%22apiKey%22:%22qUb9maKfKbtpRsdp0p2J7uWxRPGJEP%22,%22source%22:%22o-tracking%22,%22version%22:%221.0.0%22%7D,%22context%22:%7B%22product%22:%22ft.com%22,%22content%22:%7B%22asset_type%22:%22page%22%7D%7D%7D');"></div>
</div>
<!-- Send an event -->
<button onclick="document.body.dispatchEvent(new CustomEvent('oTracking.event', { detail: { category: 'element', action: 'click' }, bubbles: true}));">Send an event</button>
</body>
</html>
```

## Events

o-tracking will listen for 2 events as well as the funcations available above.

- `oTracking.page`

    Send a page view event

    ```js
    document.body.dispatchEvent(new CustomEvent('oTracking.page', { detail: { content: { uuid: 'abc-123', barrier: 'PREMIUM' }}, bubbles: true}));
    ```
- `oTracking.event`

    Send a normal event

    ```js
    document.body.dispatchEvent(new CustomEvent('oTracking.event', { detail: { category: 'video', action: 'play', id: '512346789', pos: '10' }, bubbles: true}));
    ```

## Parameters

o-tracking will
* Automatically pickup ftsession from cookies for you.
* Page events automatically pick up the url and the referrer.

### Init
```js
{
    server: "...",
    context: {
        product: "..."
    },
    user: {
        ft_session: "..."
    }
}
```

### Page
```js
{
    url: "...",
    referrer: "..."
    content: {
        uuid: "...",
        hurdle: "..."
    }
    ... any other key-values ...
}
```

### Event

__Important__: To decide how to name the category and action fields, consider this sentance (square brackets denote that part is optional):

#### A user can {action} a {category}[ with/having a {context}[ to {destination}]]

For example:
* A user can view a page having a uuid.
* A user can play a video having a video ID.
* A user can share a comment having a comment ID to Facebook.
* A user can share an article having a uuid to email.
* A user can scroll a page with a asset type.
* A user can open an email.
* A user can click an element.

```js
{
    category: 'video',
    action: 'play',
    ... any other key-values ...
    id: '512346789',
    pos: '10'
}
```

[Look at all the properties](docs/event.md) available for an event.

[Code Doc](http://codedocs.webservices.ft.com/v1/jsdoc/o-tracking/Tracking.html)
