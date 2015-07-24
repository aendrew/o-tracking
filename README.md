# Origami Tracking component

This module should be included on your product to make sending tracking requests to the Spoor API easy.

## The Spoor ecosystem
![ScreenShot](https://rawgit.com/Financial-Times/o-tracking/master/resources/images/ngda-system-design.svg)

## Usage

### Requirement: Polyfils & Doesn't cut the mustard
_Applies to both quickstart examples below_


Add the [polyfill](polyfill.io) service to your site to make sure o-tracking runs in as many brwosers as possible.

Require the noscript version of o-tracking incase the browser doesn't cut the mustard.

Checkout the [Origami spec](http://origami.ft.com/docs/developer-guide/using-modules/#core-vs-enhanced-experience) for the correct CTM logic, [html and css changes](http://origami.ft.com/docs/developer-guide/using-modules/#styles-for-fallbacks-and-enhancements) needed.



o-tracking should have the following piece of html added, with the correct server and data filled in.
```
<div class="o-tracking o--if-nojs" data-o-component="o-tracking" style="height:0;line-height:0;overflow:hidden;">
	<div style="background:url('http://server?data={}');"></div>
</div>
```

### Quickstart example 1 - JSON config

Use the build service to load o-tracking and add a json config.

```
<script type="application/json" data-o-tracking-config>
{
	"server": "https://test.spoor-api.ft.com/px.gif",
	"context": {
		"product": "ft.com" // e.g. webapp, next - This is a defined list, send the correct value!
	}
}
</script>
```

### Quickstart example 2 - JS init

Use the build service or require locally to load o-tracking and init manually.
```
var oTracking = require('o-tracking');
```

```
if (cutsTheMustard) {
    // Setup
    oTracking.init({
        server: '...',
        context: {
            product: 'ft.com'
        },
        user: {
            passport_id: passport_id,
            ft_session: ft_session
        }
    });
    // Track page
    oTracking.page({
        content: {
            uuid: uuid,
            hurdle: hurdle
        }
    });
}
```

## Events

o-tracking will listen for 2 events as well as the funcations available above.

- `oTracking.page`
    
    Send a page view event

    ```
    var event = new CustomEvent('oTracking.page', { content: { uuid: 'abc-123', barrier: 'PREMIUM' }});
    window.dispatchEvent(event);
    ```
- `oTracking.event`
    
    Send a normal event  

    ```
    var event = new CustomEvent('oTracking.event', { category: 'video', action: 'play', id: '512346789', pos: '10' });
    window.dispatchEvent(event);
    ```

## Parameters

o-tracking will
* Automatically pickup ftsession from cookies for you.
* Page events automatically pick up the url and the referrer.

### Init
```
{
    server: "...",
    context: {
        product: "..."
    },
    user: {
        passport_id: "...",
        ft_session: "..."
    }
}
```

### Page
```
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
```
{
    category: 'video',
    action: 'play',
    ... any other key-values ...  
    id: '512346789',
    pos: '10'
}
```
