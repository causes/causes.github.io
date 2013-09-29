## causes.github.io

Source for <http://causes.github.io>

### Setup

To start blogging, clone this repo.

Once the repo has been cloned, you'll want to do the usual `bundle` to get
the necessary gems.

You may also want to set up your _deploy directory so that you can build the
site without Jenkins' help.  To do so, run `bin/setup-deploy`. This creates the
_deploy subdirectory that `rake generate` (from Octopress) will populate with
the built site. You always want to be on the `master` branch in _deploy, since
that's where the static files get committed.

Writing always happens in the `source` branch. To set up your repo to push to
the `source` branch on Gerrit the default, run:

    git config remote.gerrit.push HEAD:refs/publish/source

### Creating posts

You should always be in the `source` branch when writing posts. If you are not
already in the `source` branch, check it out:

    git checkout source

To create a new post, run the following command:

    bundle exec rake new_post['My Post Title']

This will create a .markdown file in `source/_posts/` that includes the current
date and a slugified version of the post title.

### Previewing posts

To generate a copy of the static files:

    bundle exec rake generate

To watch for changes to `sass/` and `source/` and re-generate as needed, and
fire up a local server on port 4000:

    bundle exec rake preview

When you are ready for your post to be reviewed, push to gerrit:

    git push gerrit HEAD:refs/publish/source

Or, if you made pushing to the `source` branch the default earlier by running
`git config remote.gerrit.push HEAD:refs/publish/source`, then you can just
push to gerrit without specifying the target branch:

    git push gerrit

### Publishing

After your post has had adequate review in Gerrit and you are ready to see your
new content on the blog, run the following command:

    bundle exec rake deploy

This will push straight to production, bypassing code review. Any local posts
that aren't marked as unpublished, even ones that are untracked, will be live
on the website.
