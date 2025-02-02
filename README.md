# A Standard 'Toast' UI Element

# Introduction

This document proposes a new HTML element for a "toast" pop-up notification.
It is provided as a [built-in module](https://github.com/tc39/proposal-javascript-standard-library/).

This proposal is intended to incubate in the WICG once it gets interest on
[its Discourse thread](https://discourse.wicg.io/t/proposal-toast-ui-element/3634).
After incubation, if it gains multi-implementer interest,
it will graduate to the [HTML Standard](https://html.spec.whatwg.org/multipage/) as a new standard HTML element.

## What is a "toast" pop-up notification?

> "Toasts are pretty common in UX design; refers to popup notifications which typically appear at the bottom of the screen (like a piece of toast in a toaster)."

— [Kyle Decker](https://twitter.com/kybradeck/status/1139006173762531328)

Sneha Munot provides a nice definition in her [uxplanet.org article](https://uxplanet.org/toast-notification-or-dialog-box-ae32ad53106d),
where she compares toasts to dialog boxes.

> [A toast] is a small message that shows up in a box at the bottom of the screen and disappears on its own after few seconds.
> It is a simple feedback about an operation in which current activity remains visible and interactive.
> It basically is to inform the user of something that is not critical and that does not require specific attention and does not prevent the user from using the app device.
>
> For example; on gmail when a mail is send you receive a feedback of “Sending message…” written in the form of toast message.

Another concise definition is found in Ben Brocka's [ux.stackexchange.com response](https://ux.stackexchange.com/a/12000):

> A Toast is a non modal, unobtrusive window element used to display brief, auto-expiring windows of information to a user.

This adds which adds the distinguishing detail of a toast being **auto-expiring**.

In the absence of browser-intrinsic toasts,
the current state of affairs is that many libraries and design systems include toast features.
This repository contains a [study group](https://github.com/jackbsteinberg/std-toast/tree/master/study-group) which surveys and compares toast implementations across the web and other platforms.

Below is an animated image displaying some typical toast behaviors,
drawn from the [Blueprint](https://blueprintjs.com/docs/#core/components/toast) design component library.
The study group contains [a variety of such examples](study-group/Library-Demos.md).

![An animated demo showing several toasts in action, including close buttons, action buttons like "Retry", and multiple toasts stacking on top of, or replacing each other](study-group/images/blueprint-demo.gif)


## Why?

Modern web applications allow users to complete many actions per page,
which necessitates providing clear feedback for each action.
Toast notifications are commonly used to unobtrusively provide this feedback.

Many libraries in a variety of frameworks implement a version of toast
(see [research](./study-group/)),
but the web has no built-in API to address the use case.
By providing a toast API as part of the web platform's standard library,
the web becomes more competitive with other app development platforms,
and web application developers can spend less of their time and bytes
on implementing this pattern.

Toasts are also a deceptively-tricky pattern to get right.
They require special accessibility considerations,
and scenarios involving multiple toasts need special handling.
[Not all libraries account for these subtleties](./study-group/).
By providing a built-in toast control that fully handles these aspects,
we can level up the typical toast experience for both developers and users of the web.

Finally,
the ecosystem can benefit from a shared understanding of how to create and style toasts.
If the platform provides a toast,
then all libraries and components can freely use toasts to communicate to their users.
Whereas,
if toasts can only be found in libraries,
then importing a toast-using component also means importing their opinion on what the best toast library is.
In the worst case,
this can lead to multiple uncoordinated toast libraries acting on a single page,
each of which needs its own styling and tweaks to fit in to the application.
If instead libraries and components all use the standard toast,
the application developer can centrally style and coordinate them.


## Sample code

The standard toast can be used according to two different patterns.

The first defines a `<std-toast>` HTML element,
then shows it with configurations via a method on the element.
This can be used to declaratively predefine toasts the application will need,
and then show them inside the application logic.

```html
<script type="module">
import 'std:elements/toast';
</script>

<std-toast id="sample-toast" type="success">
    Email sent!
</std-toast>
```

```js
document.querySelector('#sample-toast').show({
    duration: 3000
});
```

The second imports the `showToast()` function from the `"std:elements/toast"` module,
which takes in a message and some configurations and creates and shows a toast in the DOM.
This is more convenient for one-off toasts,
or for JavaScript-driven situations,
similar to how the `alert()` function can be used show alerts.

```js
import { showToast } from 'std:elements/toast';

const toast = showToast("Email sent!", {
    type: "success",
    duration: 3000
});
```

## Goals

Across popular toast implementations there are recurring patterns,
which the standard toast aims to accomplish natively.

- The component will be accessible by default;
  native accessibility is a strong priority for toasts,
  as they can be difficult to make properly accessible.

- Toast implementations are often shaped similarly,
  and a goal of the standard toast is to make it as easy as possible for developers
  to build and style toasts that conform to those common shapes.

- The positioning of the toast must be intuitive,
  so the standard toast will come with built-in support for common positions,
  as well as a sensible default.

- To balance ease of use with customization,
  the standard toast will support creating and showing with one JavaScript function,
  as well as writing a custom view with a `<std-toast>` element
  and showing that with a method.

- The standard toast will come with support for showing multiple toasts,
  either by stacking them in the view,
  or queueing them and displaying sequentially.

The standard toast API hopes to provide a base for more opinionated or featureful toast libraries to layer on top of.
It will be designed and built highly extensible,
so library implementations can focus on providing more specific styling, better framework support, or more opinionated defaults.
The intent is that any developer looking to use a toast in their work will use a standard toast,
or a library which provides a wrapper on top of standard toast.

TODO([#14](https://github.com/jackbsteinberg/std-toast/issues/14)): create an example of this layering and link to it here.

## Proposed API

The element is provided as a [built-in module](https://github.com/tc39/proposal-javascript-standard-library/blob/master/README.md),
named `"std:elements/toast"`.
See [whatwg/html#4697](https://github.com/whatwg/html/issues/4697) for more discussion on "pay-for-what-you-use" elements available via module imports.

### The `<std-toast>` element

#### Behavior

The `<std-toast>` element provides a subtle, non-interruptive notification to the user.
The toast will appear at a customized time and position on the screen,
and will typically contain a message and optionally action and dismiss buttons
(though arbitrary markup in the element is supported).
The contents will be announced to a screen reader
(politely or assertively depending on `type` [see [attributes](#attributes)]),
and after a certain duration, the toast will timeout and hide itself
(though this timeout will be suspended while the toast has focus or the mouse is hovering on it).

TODO([#18](https://github.com/jackbsteinberg/std-toast/issues/18), 
[#29](https://github.com/jackbsteinberg/std-toast/issues/29)): 
determine properly accessible behavior,
specifically w.r.t. actions and navigation to / from the toast.

#### Attributes

- [Global attributes](https://html.spec.whatwg.org/multipage/dom.html#global-attributes)
- `open`: a boolean attribute, determining whether the toast is visible or not (according to the default styles).
By default toasts are not shown.
- `type`: an [enumerated attribute](https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#enumerated-attribute) indicating whether the toast is of a special type:
  one of `"success"`, `"warning"`, or `"error"`.
  This is used to convey special [semantics](https://html.spec.whatwg.org/multipage/dom.html#represents) for the toast,
  similar to the distinctions between e.g. `<ol>` / `<ul>` / `<menu>` or `<strong>` / `<em>` / `<small>`.
  - Toasts with no `type=""` set (or `type=""` set to an invalid value) have no special semantics distinction.
  - Like other semantic distinctions,
    authors may want to style based on the distinction.
    The [default styles](#default-styles) only change the border color of the toast,
    but can be overridden by a page or design system to give any desired appearance.
  - By default, `"error"` toasts will be treated as having the ARIA role semantics of [alert](https://rawgit.com/w3c/aria/master/#alert),
    while the other toasts will be treated as having the ARIA role semantics of a [status](https://rawgit.com/w3c/aria/master/#status).
    As with all HTML elements,
    explicitly-specified ARIA attributes can take precedence for exceptional cases,
    e.g. `<std-toast type="success" aria-live="assertive">` could be used to immediately notify the user of a time-sensitive success.
  - TODO: action-containing toasts may also need to be assertive.
- `position`: default position will be ???
    - Options for position:
        - `"top-left"`
        - `"top-center"`
        - `"top-right"`
        - `"center"`
        - `"bottom-left"`
        - `"bottom-center"`
        - `"bottom-right"`
    - TODO([#39](https://github.com/jackbsteinberg/std-toast/issues/39)): Do we need values `"top-stretch"`, `"center-stretch"`, and `"bottom-stretch"` as well? Should this stretching be done automatically on mobile?
The default (if the attribute is omitted or set to an invalid value) is ???.
    - TODO([#13](https://github.com/jackbsteinberg/std-toast/issues/13)): should this positioning be an attribute or a style
- `closebutton`: allows setting the toast's close button content (using `<std-toast closebutton="Dismiss">`),
or leaving it up to the user agent's default (using `<std-toast closebutton>`).
If this attribute is not present,
the toast does not have a close button.
See the ["Appearance customization"](#appearance-customization) section for how to customize the close button when it's present.

#### Properties

##### Reflected properties

All attributes will be reflected as properties on the element's JavaScript interface.
For example:

```js
const toast = document.createElement('std-toast');
console.log(toast.open); // false
```

The `type` enumerated attribute is reflected [limited to only known values](https://html.spec.whatwg.org/multipage/common-dom-interfaces.html#limited-to-only-known-values),
with an invalid value default and missing value default of the default state. The default state does not have a corresponding keyword.

_This means that `toast.type` will return the empty string, if the `type=""` attribute's value is missing, or not a case-insensitive match for `"warning"`, `"error"`, or `"success"`. Otherwise it will return those values, in their canonical lowercase._

##### `closeButton` property

The `closeButton` property allows controlling the element's `closebutton=""` attribute.
It is similar to the reflected properties,
but slightly more complicated to allow both a boolean usage model
(for using the user agent's default close button content)
and a string usage model
(for customizing the close button content).
In detail:

- The getter returns `false` if the attribute is absent,
  `true` if the attribute is present with the empty string as its value,
  and the attribute's value otherwise.
- The setter,
  if given true,
  sets the value of the attribute to the empty string.
  If given false,
  it removes the attribute.
  Otherwise,
  it converts the given value to a string,
  and sets the attribute's value to that string.

To see examples of the `closeButton` setter in use,
read on to the next section.

##### `action` property

There will additionally be an `action` property,
which returns or allows setting an element that provides the toast's action.
In detail:

- The getter returns the first descendant element with the `slot="action"` attribute set.
  If no such element exists,
  it returns null.
- The setter accepts an element,
  appends it to the toast element,
  and sets the `slot="action"` attribute on it.
  If any existing such elements exist,
  the first one (i.e. the one that would be returned by the getter) is replaced.
  (If the provided value is not an `Element`,
  a `TypeError` is thrown.)

To see examples of the `action` getter and setter in use,
read on to the next section.

#### Contents

At a technical level, the `<std-toast>` element can be used as a generic container.
However, the authoring conformance requirements,
as well as the API,
steer the developer in to the following content model:

- All children except an action need to be [phrasing content](https://html.spec.whatwg.org/#phrasing-content-2),
  that is not [interactive content](https://html.spec.whatwg.org/#interactive-content-2).
  These children provide the toast's message.
- There may be zero or one actions,
  which are either `<a>` or `<button>` elements,
  or autonomous custom elements,
  decorated with the `slot="action"` attribute.
  TODO: is this the right restriction on what can be an action?
  What about `<div tabindex="0" class="synthetic-button">`?
  But allowing all interactive content is weird.
  (Note that it is for HTML validators/authors and does not impact the API.)

TODO([#17](https://github.com/jackbsteinberg/std-toast/issues/17)): what about title or icon?
They could potentially also be accomodated, in a similar fashion.

Thus,
the following would all work well out of the box,
and be comformant to the content model:

```html
<std-toast>Hello world!</std-toast>

<std-toast><p>Hello <strong>world</strong>!</p></std-toast>

<std-toast id="toast3">
  Hello world!
  <button slot="action" class="fancy-button">Click me!</button>
</std-toast>
<script type="module">
document.querySelector("#toast3").action.onclick = e => { /*...*/ };
</script>

<std-toast>
  Hello world!
  <a slot="action" href="https://example.com/" target="_blank">Click me!</button>
</std-toast>

<std-toast closebutton>
  Hello world!
</std-toast>

<std-toast closebutton="Dismiss">
  Hello world!
</std-toast>
```

These can equivalently be created via JavaScript:

```js
const toast1 = showToast("Hello world!");

const toast2 = showToast();
toast2.innerHTML = `<p>Hello <strong>world</strong>!</p>`;

const toast3 = showToast("Hello world!", { action: "Click me!" });
toast3.action.className = "fancy-button";
toast3.action.onclick = e => { /*...*/ };

const toast4 = showToast("Hello world!", { action: document.createElement("a") });
Object.assign(toast4.action,
  href: "https://example.com",
  target: "_blank",
  textContent: "Click me!"
});

const toast4SetterDemo = showToast("Hello world!");
toast4SetterDemo.action = document.createElement("a");
Object.assign(toast4SetterDemo.action, /* as before */);

const toast5 = showToast("Hello world!", { closeButton: true });

const toast6 = showToast("Hello world!", { closeButton: "Dismiss" });
```

(Note: because frames are only painted after JavaScript runs to completion,
the setup code in the lines immediately after `showToast()` will be applied before the toast is ever shown to the user.)

More complex toasts,
that don't fit the above content model,
would require custom handling on the part of the developer,
as they are stepping outside of the supported use case:

```html
<!-- Invalid HTML, but will still "work" -->
<std-toast>
  <p>Hello world!</p>

  <textarea slot="action">Action 1</textarea>
  <button>Action 2</button>
  <button slot="action">Action 3</button>
</std-toast>
```

Such an unusual toast would still integrate with other toasts in terms of stacking and positioning behavior,
and some of the default styles that are inherited may be useful.
But the `action` property will only provide a pointer to the `<textarea>`,
and the page author will need to add additional styling to handle the extra contents.

TODO: when we have a prototype, link to/show an example of this in action.

#### Methods

- `.show(options)`: shows the toast element,
by toggling its `open=""` attribute to true.
It will also start or reset the timeout of the toast,
to show for the provided `duration`.
The `options` include:
    - `duration`: how long to show the toast,
      in milliseconds.
      Can be set to `Infinity` to show the toast indefinitely,
      but values ≤ 0 will cause a `RangeError` to be thrown.
      If not provided,
      the default will be `Infinity` for `type="warning"` or `type="error"` toasts,
      and `3000` otherwise.
    - `multiple`: ???
    - `newestOnTop`: ???

TODO([#37](https://github.com/jackbsteinberg/std-toast/issues/37)): should `3000` be the default?
Should it be longer for accessibility reasons? Should it change dependent on the amount of text in the toast?

TODO([#19](https://github.com/jackbsteinberg/std-toast/issues/19)): how do `multiple` and `newestOnTop` work?

- `.hide()`: hides the toast element,
by toggling its `open=""` attribute to false.
- `.toggle([state])`: toggles the toast element,
by hiding it if it's being shown and showing it if it's being hidden,
or alternately adding/removing the `open=""` attribute per `state` if `state` given.

#### Events

A `<std-toast>` element can fire the following events:

- `"show"`: the toast was shown
- `"hide"`: the toast was hidden, either explicitly by the user, or via the timeout.
  (Note: if animations were applied, the toast may not be entirely invisible at the time this event fires)
    - TODO: should we consider separate events for the start and end of any hide animation?
      This seems hard to do correctly if the user customizes the animation, though.
      TODO([#35](https://github.com/jackbsteinberg/std-toast/issues/35)): split `"show"` and `"hide"` into `"will/didShow"` and `"will/didHide"`.

### `showToast(message, options)`

The `"std:elements/toast"` module also exports a convenience function,
`showToast()`,
which allows creating and showing toasts entirely from JavaScript.
Behind the scenes,
`showToast()` creates a `<std-toast>`,
sets it up using the given message and options,
inserts it as the last child of `<body>`,
and then adds the `open=""` attribute,
to make the toast visible.
Finally,
it returns the created `<std-toast>` element,
allowing further manipulation by script.

`message` is a string that will be inserted as a text node into the created `<std-toast>`.

`options` allows configuring both the attributes and contents of the `<std-toast>`,
as well as the options for this particular showing of the toast.
Thus, the possible options are:

- `type`, like the property that reflects the `type=""` attribute.
- `position`, like the property that reflects the `position=""` attribute.
- `closeButton`, like the property that reflects the `closebutton=""` attribute.
  Defaults to `true` if `duration` is `Infinity`,
  including if it defaults to infinity by setting `type` to `"warning"` or `"error"`.
- `duration`, like the `show()` option.
- `multiple`, like the `show()` option.
- `newestOnTop`, like the `show()` option.
- `action`, an `Element` or string.
  An `Element` is treated the same as the `action` property setter.
  Otherwise,
  the result is converted to a string,
  which gives a shortcut for creating a `<button slot="action">` with the string value as its `textContent`.

TODO([#15](https://github.com/jackbsteinberg/std-toast/issues/15)): Should a toast generated by `showToast()`
get removed from the DOM, and if so should there be cases where it remains?

### Default styles

The standard toast will come with these default styles,
which developers will be able to change to customize look and feel.
They are similar to [the default styles for `<dialog>`](https://html.spec.whatwg.org/multipage/rendering.html#flow-content-3),
which are very minimal.

```css
std-toast {
    position: fixed;
    offset-block-end: 1em;
    offset-inline-end: 1em;
    border: solid;
    padding: 1em;
    background: white;
    color: black;
    z-index: 1;
}

std-toast:not([open]) {
    display: none;
}

std-toast[type=success i] {
    border-color: green;
}

std-toast[type=warning i] {
    border-color: orange;
}

std-toast[type=error i] {
    border-color: red;
}
```

TODO: the choice of block/inline-end positioning
(lower-right in left-to-right, horizontal writing modes)
is not final and will likely be changed.
It also interacts with the desire for easier positioning discussed in the attributes section.
Discuss in [#13](https://github.com/jackbsteinberg/std-toast/issues/13) and [#39](https://github.com/jackbsteinberg/std-toast/issues/39).

### Appearance customization

TODO: link to demos.

Appearance can be customized using normal CSS.
For example, to change the colors, you could do

```css
std-toast {
    border-color: blue;
    background-color: yellow;
    color: red;
}
```

To completely remove all default styling (useful for library authors),
use

```css
std-toast {
    all: unset;
}
```

#### Action

To style an action (if present),
use the `[slot=action]` selector:

```css
std-toast [slot=action] {
    color: blue;
    text-transform: uppercase;
}
```

(Note: we could offer an alternate syntax of `std-toast::part(action) { ... }`.
Discuss in [#20](https://github.com/jackbsteinberg/std-toast/issues/20).)

#### Close button

To style the close button (if present via the `closebutton=""` attribute),
use the `::part(closebutton)` selector:

```css
std-toast::part(closebutton) {
    position: absolute;
    top: 0;
    right: 0;
    background: none;
}
```

#### Types

To give custom styling to the different types of toasts,
use the `[type]` selector.
Remember to use the `i` modifier so that your styling works even if someone does
`<std-toast type="ERROR">`.

```css
std-toast[type=error i] {
    background: #c23934;
    color: white;
}

std-toast[type=error i]::before {
    content: "⛔" / "";
}
```

## Accessibility

TODO: determine proper accessibility roles / semantics.
See issues [#25](https://github.com/jackbsteinberg/std-toast/issues/25) and
[#29](https://github.com/jackbsteinberg/std-toast/issues/29) for ongoing initial discussions.

## Common Patterns

### Show existing toast
```html
<std-toast id="sample-toast">
    Hello World!
</std-toast>
```

```js
document.querySelector('#sample-toast').show();
```

### Create and show new toast with options
```js
const toast = showToast("Hello World!", {
    type: "info",
    duration: 3000
});
```

### Show toast in container
```html
<div id="container"></div>
```

```js
const toast = showToast("Hello World!");
document.querySelector("#container").append(toast);
```

### Reusable config object

```js
const configs = {
    type: 'info',
    duration: 3000
}

const toast1 = showToast("number 1", configs);
const toast2 = showToast("number 2", configs);
```

## Security and Privacy Considerations
See [TAG Security / Privacy Self Review](/security-privacy-self-review.md).

## Considered Alternatives

### Extending the Notification API
Toasts are intended to be ephemeral, context-specific messages,
and collecting them in a user's notifications tray doesn't reflect the spirit of the pattern.
[kenchris](https://github.com/kenchris) put this well in [this comment](https://github.com/w3ctag/design-reviews/issues/385#issuecomment-502070938)
on the TAG Design Review:

> the difference is that this is a temporary notification / infobar,
> like you see for PWAs ("new version available, press reload to refresh")
> or like in GMail ("Chat is currently unavailable") etc.
> These are not things you want an actual system notification for...
> They really depend on the surrounding content/situation unlike regular notifications.

The toast is intended to be a completely in-page element,
not a system-level notification the way Notifications and Android toasts are.
Because of this page-level role, toasts do not require any user-granted permissions the way Notifications do.

### Extending the `<dialog>` element
This approach was considered, and is still an open area of exploration,
but our current thoughts are to go a different route for a few reasons.
First and foremost, we felt the accessibility considerations for a toast and a dialog differ,
as the dialog typically engages in the kind of flow-breaking behavior we want toasts to avoid.
Additionally, the popular pattern of adding a call to action button on a toast with a baked-in time limit
necessitates replacing tab-trapping with a less intrusive, easily restored alternative.
The line between the elements becomes blurrier from a functional perspective,
with action-button-containing toasts and non-modal dialogs,
but the intent and semantics are sufficiently different,
similar to `<blockquote>` and `<q>`, or `<meter>` and `<progress>`.

### As a new global element (instead of a built-in module)
The idea of creating new polyfillable HTML elements is being explored in
[this issue](https://github.com/whatwg/html/issues/4696) on the HTML standard,
and the idea of creating new pay-for-what-you-use HTML elements is being explored in
[this issue](https://github.com/whatwg/html/issues/467) on the HTML standard.

### Leaving this up to libraries
The rationale discussed for `std-switch` in [this explanation](https://github.com/tkent-google/std-switch#leaving-this-up-to-libraries)
in the Considered Alternatives section of [tkent's std-switch explainer](https://github.com/tkent-google/std-switch#considered-alternatives)
applies to `std-toast` as well.

## Extra Resources
This proposal comes in parallel with the [proposal for a standard switch element](https://github.com/tkent-google/std-switch),
which has a good [FAQs section](https://github.com/tkent-google/std-switch#faqs), many of which apply to this proposal as well.
There is also currently an [open discussion](https://discourse.wicg.io/t/proposal-toast-ui-element/3634)
on the WICG discourse page to gauge and solicit community interest in moving the explainer to incubate in a WICG repo.
