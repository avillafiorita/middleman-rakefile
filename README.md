# What it is

A Rakefile to automate some operations on your [middleman](http://middlemanapp.com) websites.

Main features:

* Make sure `middleman build` is called before `middleman deploy`, if sources have changed since the last build
* Deploy only if needed (e.g., if there are local changes to be published)
* Check the status of your git repository before deploying
* Keep track of what you changes have been made since the last deploy


# Installation

1. Download this repository
2. Put the Rakefile in your [middleman](http://middlemanapp.com) website directory
3. Start using it!


# Typical usage scenario

<pre>
rake preview
rake deploy
</pre>

# Getting help

<pre>
rake --tasks
</pre>

... you can also get in touch by email, of course!


# Determining whether a build is necessary

This Rakefile tries to be efficient with build and deploy commands, invoking `middleman build` and `middleman deploy` only when they are necessary.  The checks are performed as follows:

* a build is necessary if there is a file in `source` or `data` which is more recent than the most recent file in `build`
* a deploy is necessary if there is a fil in `build` which is more recent thatn the `_last_deploy.txt` file.  (The `_last_deploy.txt` file is updated any time a deploy is performed.)

If in doubt, you can use `rake check_timestamps` to see what the Rakefile would do.  If you can always force a build or a deploy with the `rake force_build` and `rake force_deploy` commands, respectively.


# Integration with Git

If you are using `git` to manage the sources of your website, the local repository has to be in sync with the default remote before building and publishing the website.  If this is not the case, in fact, you might deploy an old version of the repository, causing a regression of the published content.  One solution to this problem is using a custom `post-commit` hook, which deploys the website after a successful push:  this is what [Github](http://www.github.com) does.  A second solution is using this Rakefile: in fact it checks whether there are remote changes to be pulled and, if so, it warns the user.  Additionally, the Rakefile can commit and push on deploy, if the variable `$git_autopush` is `true`.

# Change Log

* **version 1.0** is the first released version

# License

Distributed under the terms of the [MIT License](http://opensource.org/licenses/MIT).
