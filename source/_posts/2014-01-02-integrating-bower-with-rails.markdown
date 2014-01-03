---
layout: post
title: "Integrating Bower with Rails"
date: 2014-01-03 10:00
external-url: http://joelencioni.com/blog/2014/01/03/integrating-bower-with-rails/
comments: false
sharing: false
author: Joe Lencioni
categories:
  - Bower
  - Rails
  - Sprockets
  - Asset Pipeline
---

We tend to prefer convention over configuration, so when we first began using
[Bower] to manage our front-end dependencies, we started with minimal
configuration. While this nicely put the files in the expected places, it was a
hassle actually using the assets in our Rails app.

Our basic initial approach was to symlink the assets that we wanted to
reference from the `bower_components` directory into `vendor/assets` where
Sprockets would notice them. While this worked in the short term, it was
tedious to set up new packages, and was likely to break if the required assets
were moved when upgrading dependencies.

Ultimately, this felt like a hack. We decided that there must be a better way.
Thankfully there is.

<!-- more -->

## Configuring Bower

To get us out of the clunky situation of managing symlinks, we first wanted
Bower to store its files in a Railsy location. The right spot for this seemed
to be `vendor/assets/bower_components`. We chose `vendor/assets` because these
are third-party assets, and `bower_components` because it makes clear that
these assets are managed by Bower and shouldn't manually be meddled with.

Bower allows us to configure the directory it stores its files via the
`.bowerrc` file. Since we didn't have one yet, all we needed to do was add this
as a sibling to our `bower.json` file, which was in our app root:

```json .bowerrc
{
  "directory": "vendor/assets/bower_components"
}
```

## Configuring Rails

Now that the assets were in a logical place, we wanted Rails to be able to find
them easily without us having to manage an army of symlinks. In our application
config, we have access to the list of paths that Sprockets checks for assets
that are imported in manifests. We simply needed to add the Bower components
path to that list:

```ruby config/application.rb
config.assets.paths << Rails.root.join('vendor', 'assets', 'bower_components')
```

## Using Bower Assets

Now that we have wired up the critical pieces, we can reference Bower assets in
manifests, and Sprockets will pull them in for us.

```js Example application.js
//= require my_js_file
//= require bower_component_name/js_file
```

```scss Example application.css.scss
@import 'mixins';
@import 'reset';
@import 'bower_component_name/stylesheet'
```

The assets in `vendor/assets/bower_components` are not precompiled on their own
so they can only be referenced in manifests and cannot be served directly to
clients. However, we wanted to serve a couple of components individually and
not bundled with other assets in a pre-existing manifest, generally either
because of their size ([D3]) or because they were being served only to a subset
of clients ([Respond]). For these cases, we created shim manifests that only
include the single file we want served:

```js Our shim d3.js manifest
// Shim to coax vendor/assets/bower_components/d3/d3.js to be precompiled by
// the asset pipeline
//= require d3/d3
```

## Alternatives

There is a [bower-rails gem] that provides a <acronym title="Domain Specific
Language">DSL</acronym> that allows you to manage Bower components via a
Bowerfile, similarly to how [Bundler] operates. Although it sounds pretty
slick, we were hesitant to add another dependency to our stack when the
alternative outlined above requires so little work to configure.

Additionally, [https://rails-assets.org] can be added as a gem source to your
`Gemfile`, which allows you to reference Bower packages via `gem
'rails-assets-BOWER_PACKAGE_NAME'`. We decided against this approach because it
seemed to add an unnecessary layer of indirection when again, the alternative
outlined above is so simple.

[Bower]: http://bower.io/
[D3]: http://d3js.org/
[Respond]: https://github.com/scottjehl/Respond
[bower-rails gem]: https://github.com/42dev/bower-rails
[Bundler]: http://bundler.io/
[https://rails-assets.org]: https://rails-assets.org/
