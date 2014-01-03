# Mottle
=============

Mottle is a simple event manager that takes care of a few bits and pieces I would often run into during development jsPlumb and the jsPlumb Toolkit. Briefly, its features
are:

- automatic mapping of mouse events to their touch equivalents on touch enabled devices
- "smart" click handling: not posting a click event if the mouse has moved between mousedown and mouseup
- simulation of click/dblclick/contextmenu events for touch devices
- event delegation
- mouseenter and mouseexit event support
- event triggering

### Browser Support

Currently, all desktop browsers and iOS devices are supported.  Support for Android will be coming shortly.

#### Basic Events

The following events can be bound, using the `on` function:

- mousedown
- mouseup
- mousemove
- click
- dblclick
- mouseover
- mouseout
- contextmenu


```javascript
var m = new Mottle();
m.on("someElementId", "click", function(e) { ... });
m.on(someElement, "mouseout", function(e) { ... });
```

#### Smart clicks

The smart click handler provides `click` and `dblclick` events that are not fired if the mouse has moved between `mousedown` and `mouseup` (the default 
browser functionality is to fire a click even if the mouse has moved). This handler works by registering a single `mousedown`, `mouseup` and `click` 
event handler on any element for which a smart click (or dblclick) event has been registered.  The down and up handlers store the mouse position. The 
click handler checks to see if these two positions are identical, and if they are, the event is fired. The handler maintains a list of click listeners 
and a separate list of dblclick listeners, and whenever a click or dblclick should be fired, these lists are iterated and each function in them is fired 
one by one. Of course if you register a click or dblclick on some element that has already been configured for smart clicks, the handler is just added directly
to the appropriate existing list, at which time its index in the list is stored as the `__taSmartClickIndex` property on the function itself.  
Then, if the user unbinds the given function, we have a hook to remove it from the list, which prevents memory leaks.

To enable smart clicks, you set it on Mottle's constructor:

```javascript
var m = new Mottle({ smartClicks:true });
m.on("someElementId", "click", function(e) { ... });
```

#### Synthesized mouseenter/mouseexit

Mouseenter and mousexit are proprietary Internet Explorer events that are kind of useful, and for that reason Mottle offers a wrapper for them in other
browsers. The basic concept is that the event only fires when the mouse enters or exits the exact element on which the listener was 
bound. This differs from the `mouseover`/`mouseout` events, which are fired on some element regardless of whether the event occurred on 
that specific element or on one of its descendants. For this, a `mouseout` and a `mouseover` listener are registered on
the element on which you wish to bind to `mouseenter`/`mouseexit`.  Then, an object is set on the element that contains a flag indicating 
whether the mouse is currently over the element, along with a list of `mouseenter` and `mouseexit` listeners. When a `mouseover` event is fired - remember 
they are fired by the element and all of its descendants - the target is checked. If it is the element itself and the `over` flag is not set, then a 
`mouseenter` has occurred, and all the `mouseenter` listeners are fired. A complication with this simple explanation is that these handlers work with event 
delegation - so in fact at any point in time there may be multiple elements on which the mouse is considered to have entered. but you needn't worry about that.  
To determine `mouseexit`, we test, in the `mouseout` listener, whether the event target is one of the elements that mouse is currently over, and if so, whether 
the "related target" (the element to which the mouse has gone) is NOT a descendant of that element. If both of these conditions are true then we have a `mouseexit`.

#### Synthesized click/dblclick/contextmenu

Mottle offers synthesized `click`, `dblclick` and `contextmenu` events for touch devices. These work by adding a `touchstart`
and `touchend` listener to the element. On `touchstart`, a timer is started. If the timer finishes before `touchend`, then no 
event is fired. If the `touchend` event occurs before the timer finishes, then an event is fired.

#### Touch Event Mapping

On touch devices, Mottle maps mouse events to their touch equivalents, allowing you to bind to `mousedown`, `mouseup`
and `mousemove` in your code and be confident it will still work on a touch device. This is achieved through a simple mapping of 
event name: when you register a `mousedown` listener, it is actually bound as a `touchstart`, for example. There are some devices on the 
market that are both touch and mouse devices; for those, the event listener is simply bound to both the mouse event you asked for, and 
its touch equivalent.

The three basic touch events are mapped in this way:

- __touchstart__ -> __mousedown__
- __touchend__ -> __mouseup__
- __touchmove__ -> __mousemove__


##Event Mapping


In addition,Mottle supports `click`, `dblclick` and `contextmenu`, by starting a timer on touchstart and then checking if the touchend event happens within a certain threshold afterwards.  For the `contextmenu` event, touch-adapter looks for a touchstart+touchend using two fingers (this is one way you can do a right-click on the Mac's trackpad).


##Usage

### 0.2

#### Constructor

	var mottle = new Mottle(PARAMS);

Allowed constructor parameters are:

- **smartClicks** If true, will not report click events if the mouse has moved between mousedown and mouseup.
- **clickThreshold** Amount of time, in milliseconds, inside of which a mousedown should be followed by a mouseup in order to be considered as a click in a touch device. Default is 150ms.
- **dblClickThreshold** Amount of time, in milliseconds, inside of which two consecutive clicks must occur in order to be considered a double click in a touch device. Default is 250ms.

#### Event Binding

To directly bind an event handler on some element, use `bind`:

	mottle.bind(someElement, "click", aFunction);
	mottle.bind(someElement, "dblclick", anotherFunction);	

To subsequently unbind, use `unbind`. Note you have to supply the original function:

	mottle.unbind(someOtherElement, "dblclick", anotherFunction);

#### Event Delegation

To have some element act as an event handling delegate for some set of its child elements, use `on`:

	mottle.on(someElement, "div.foo, div.bar", "click", aFunction);

To remove the event delegation, use `off`:

	mottle.off(someElement, "click", aFunction);

Note that the `off` function does not take a list of selectors as argument. It removes all event delegation for the set of child selectors with which the given function was registered.

---







