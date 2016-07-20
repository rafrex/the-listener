# The Listener

Set listeners for mouse, touch and pointer events. Pass in a DOM element along with the mouse and touch event handlers, and `the-listener` will only set listeners for the events that correspond to the device's capabilities, and will automatically set pointer event listeners if needed (uses [`detect-it`](https://github.com/rafrex/detect-it) to determine device capabilities).

- If it's a mouse only device, then only mouse event listeners are set.
- If it's a touch only device, then only touch event listeners are set.
- If it is a hybrid device, then both mouse and touch event listeners are set, but the mouse event handlers are only called when no touch event is fired (touch interactions fire both mouse and touch events, which is one of the problems that `the-listener` solves).
- If a device is touch capable but only supports pointer events, then the corresponding pointer event listeners are automatically set instead of mouse and touch event listeners (this behavior can be prevented if desired).



### Installing `the-listener`
```terminal
$ npm install the-listener
```

### Using `the-listener`

```javascript
import addListener from 'the-listener';
```
```javascript
/**
 * addListener() sets listeners on the domElementTarget
 * based on the options in objWithEventsAndFunctions
 *
 * @param {DOM Element} domElementTarget
 * @param {Object} objWithEventsAndFunctions
 */
addListener(domElementTarget, objWithEventsAndFunctions);
```
```javascript
/*
 * object keys are the event types and options, and the values are the handlers
 * if the same function should be called for multiple event types,
 * then set the key as a space separated string with the multiple event types
 */
const objWithEventsAndFunctions = {
  'one or more events as space separated string': function(event){...},

  // handler will be called for both mousedown and touchstart events,
  'mousedown touchstart': function(event){...},

  // the click event handler is called for both mouse and touch 'clicks' without any delay
  click: function(event){...},

  // add capture to the key string to have the handler called during
  // the capture phase instead of the bubbling phase
  'mouseleave capture': function(event){...},

  // add passive to the key string to set a passive event listener
  'touchmove passive': function(event){...},

  // prevent pointermove listener from being set, will still set other pointer listeners
  // note that the corresponding mousemove listener won't be set
  pointermove: false,
}
```

#### Examples
```javascript

addListener(el1, { click: (e) => alert(e) });

addListener(el2,
  {
    'mousedown touchstart': (e) => (...),
    'mouseup touchend': (e) => (...),
    mouseenter: (e) => (...),
    mouseleave: (e) => (...),

    // separate handlers for mousemove and touchmove
    mousemove: (e) => (...),
    'touchmove capture passive': (e) => (...),

    // never set pointermove listener
    pointermove: false,
  }
);
```

#### Notes
- In the case of `click` events from touch interactions, the `click` event handlers will be called on `touchend` and not wait for the delayed click event to be fired (providing that `touchend` occurs within 500ms of `touchstart`, otherwise the `click` event handler won't be called at all - this is how modern mobile browsers work).

- All mouse and touch event handlers are only called when the event is fired from the respective input. Even though the `mousedown` event is fired after a touch interaction, `the-listener` won't call the `mousedown` handler. For example, if you want a handler to be called on both `mousedown` and `touchstart` then you need to explicitly set both. The only exception is that `click` event handlers are called for both mouse and touch events.

- `the-listener` never calls `preventDefault()`, so it won't effect anything else in your app (you can call `preventDefault()` in side of your handlers if desired).

- Pointer event listeners are only set when the device is touch capable but doesn't support the touch events api. In this case, pointer event listeners are set instead of mouse and touch event listeners (in all other cases, even if the device supports pointer events, only the mouse and touch event listeners are set). If you don't want a specific type of pointer event listener to be set, e.g. `pointermove`, then set `pointermove: false` in the options object. Note that the corresponding mouse event listener will not be set, e.g. the `mousemove` listener will not be set.

- Set passive event listeners by adding `passive` to the key string. Not all browsers support passive event listeners and `the-listener` will only set a listener as passive if the browser supports it, otherwise the listener will be set as a normal listener. See the [passive event listener explained](https://github.com/WICG/EventListenerOptions/blob/gh-pages/explainer.md) for more information.

- Set capture phase listeners by adding `capture` to the key string.
