---
layout: post
title: 'Working with Asynchronously Loaded JavaScript Objects'
date: 2013-05-28 14:00
external-url: http://joelencioni.com/blog/2013/05/28/working-with-asynchronously-loaded-javascript-objects/
author: Joe Lencioni
categories:
  - Asynchronous
  - Facebook
  - JavaScript
---

Telling browsers to load large JavaScripts asynchronously can significantly
improve performance. This prevents the browser from blocking the rendering of
the page, allowing it to be viewed more quickly. However, if you are loading
dependent files asynchronously, such as a third-party service's API, making the
scripts work together is not automatic.

At [Causes](http://www.causes.com) we use Facebook's large (nearly 60 <abbr
title="Kibibytes">KiB</abbr> gzipped) JavaScript API on our pages. Although
[they recommend loading it
asynchronously](https://developers.facebook.com/docs/reference/javascript/#loading),
we were already putting our JavaScript at the bottom of the page and weren't
convinced that async would give us much additional benefit. However, after some
non-scientific performance tests it appeared that switching to asynchronously
loading the Facebook API could reduce the time to `DOMContentLoaded` by nearly
a full second on our pages.

<!-- more -->

After the Facebook API has been loaded asynchronously, it executes
`window.fbAsyncInit`, which is where  [they
suggest](https://developers.facebook.com/docs/javascript/gettingstarted/#loading)
placing code that depends on the `FB`.

The trouble is, we wanted our JavaScript that interacts with the Facebook API
to continue to work even though it is dispersed throughout our scripts, and it
may execute before the API was available.

The [answers on
StackOverflow](http://stackoverflow.com/questions/3548493/how-to-detect-when-facebooks-fb-init-is-complete)
may work, but they feel inelegant. For instance, [one
answer](http://stackoverflow.com/a/3549043/18986) addresses this problem by
setting a flag to true in the `fbAsyncInit` callback, and creates a
`fbEnsureInit` method that uses a recursive timeout to poll the flag.

We decided to take a different approach. Thankfully, the Facebook API sets up a
single object, `FB`, for developers to interact with.

So we developed a class, `MethodProxy`, to act as a proxy between the rest of
our code and the `FB` object. It accepts an object to forward calls onto, and
it masquerades as an Array by implementing a `push` method. The `push` method
accepts a single argument of a array whose first position is always a string of
the method name on the `FB` object to be executed, followed by as many
arguments to be given to that method.

```javascript MethodProxy.push()
// payload : ['methodName', arguments*]
this.push = this.forward = function(payload) {
  var methodName = payload.shift().split('.'),
      object     = this.object,
      method;
  while (methodName.length) {
    // dig into the object as many levels as needed (e.g. `FB.XFBML.parse`)
    if (method) {
      object = object[method];
    }
    method = methodName.shift();
  }
  return object[method].apply(object, payload);
};
```

This allows us to initialize a native Array when the page loads to collect
calls to Facebook's API. Then, in the `fbAsyncInit` callback, we Indiana Jones
style swap out the native array with an instantiation of our `MethodProxy`
object, which then consumes the queued events by executing them on the `FB`
object. The end result is a consistent API for our code to use regardless of
the state of Facebook's asynchronously loaded API.

```javascript
MyFB = window.MyFB || [];

MyFB.push(['ui', { ... }]);

window.fbAsyncInit = function() {
  FB.init({
    ...
  });

  MyFB = new MethodProxy(FB, MyFB);
};

MyFB.push(['FBML.parse', ...]);
```

`MethodProxy` is general enough to be used for other similar cases so we've
released it as [method-proxy-js, an open source
project](https://github.com/causes/method-proxy-js) under the MIT license. Pull
requests welcome.
