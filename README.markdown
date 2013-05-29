## causes.github.io

Source for <http://causes.github.io>

### Setup

You'll want to do the usual `bundle` to get the necessary gems. The first time
you clone this repo, you may want to set up your _deploy directory so that you
can build the site without Jenkins' help.

To do so, run `bin/setup-deploy`. This creates the _deploy subdirectory that
`rake generate` (from Octopress) will populate with the built site. You always
want to be on the `master` branch in _deploy, since that's where the static
files get committed.

### Creating posts

    rake new_post['My Post Title']

### Publishing

   rake deploy
