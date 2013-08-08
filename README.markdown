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

Writing always happens in the `source` branch. To create a new post, run the
following command:

    bundle exec rake new_post['My Post Title']

This will create a .markdown file in `source/_posts/` that includes the current
date and a slugified version of the post title.

### Previewing posts

To generate a copy of the static files:

    bundle exec rake generate

To watch for changes to `sass/` and `source/` and re-generate as needed:

    bundle exec rake watch

To watch and fire up a local server on port 4000:

    bundle exec rake preview

When you are ready for your post to be reviewed, push to gerrit:

    git push gerrit HEAD:refs/publish/source

To make pushing to the "source" branch the default:

    git config remote.gerrit.push HEAD:refs/publish/source

### Publishing

When you are ready to see your new content on the blog, run the following
command:

    bundle exec rake deploy

This will push straight to production, bypassing code review. Any local posts
that aren't marked as unpublished, even ones that are untracked, will be live
on the website.
