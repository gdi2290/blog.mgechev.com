---
title: Flux in Depth. Store and Communication with Services.
author: minko_gechev
layout: post
categories:
  - JavaScript
  - AngularJS
  - React
  - Flux
  - MVC
  - MVW
tags:
  - Flux
  - JavaScript
  - AngularJS
  - React
  - MVC
  - MVW
---

This is the second, and probably be last, blog post of the series "Flux in Depth". In first post we did a quick overview of flux, took a look at the stateless, pure components, immutable data structures and component communication. This time, we're going to introduce the store and how we can communicate with services through the network via HTTP, WebSocket or WebRTC. Since flux architecture doesn't define a way of communication with external services, here you can find may way of dealing with network communication. If you have any suggestions or opinions, do not hesitate to leave a comment.

## Store

As we said, we have pure components, which are not holding any state. However, our application can't be completely stateless. We have at least business data, UI state and configuration. The store is the place, which keeps this data.

Lets peek at the flux data flow once again:

![High-Level Overview](/images/overview-components/flux-overview.png)

as we can see from the diagram above, the data flow starts in the view, goes to an action, which invokes the dispatcher, after that goes to the store and in the end it arrives in the view again. Alright, so the store is responsible for providing the data to the view. So far so good. How we can make the store deliver the required by the view data? Well, it can throw an event! Observables are getting quite popular recently. For good or bad, they are even going to get into the JavaScript standard. If this makes you angry because you have to learn new things and the language is getting fatter, you can find the guy who stays behind all of this here:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lil4YCCXRYc" frameborder="0" allowfullscreen></iframe>

Observables are good way of building our store. If the view is able to observe the store for changes, it can update itself once the store changes. However, since we want to describe how things are actually happening on low level, we can implement a basic observable object. Before that we need to define a design pattern (yeah, my friends already noticed I'm talking about design patterns constantly and did an intervention for me but didn't help).

![Intervention](/images/intervention.jpg)

### Chain of Responsibility

This is a design pattern, which is exclusively used in the event-driven programming. I bet you know that you can listen for any event on the document, once a user clicks on a button the event will propagate to the root of the DOM tree (eventually) and be caught by your listener:

```javascript
document.addEventListener('click', e => {
  alert(`You actually clicked on element with #${e.target.getAttribute('id')}, not on document`);
  alert(e.currentTarget === document)
}, false);
```
```html
<button id="awesome-button">Click me</button>
```

If you click on the `awesome-button`, you'll see two alert boxes: "You actually clicked on element with #awesome-button, not on document" and "true". If you change the third parameter of `addEventListener`, the event will propagate in the opposite direction (i.e. from top to bottom).

Why I said all of this? Well, this is chain of responsibility:

> In object-oriented design, the chain-of-responsibility pattern is a design pattern consisting of a source of command objects and a series of processing objects.[1] Each processing object contains logic that defines the types of command objects that it can handle; the rest are passed to the next processing object in the chain. A mechanism also exists for adding new processing objects to the end of this chain.

Or if you're have more enterprise taste, here is the UML class diagram:

![Chain of Responsibility](/images/patterns/behavioral/chain-of-responsibilities.svg)

Why we need this pattern? Well, basically our store will be a tree of objects, once a property in an internal node in the tree changes, we'll propagate this change to the root. The root component in our view will listen for events at the store object. Once it detects a change it'll set it's state.