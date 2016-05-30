# What it is

A Rakefile to automate some operations on your
[middleman](http://middlemanapp.com) websites.

Main features:

* Support for different environments: `middleman deploy[staging]` and
  `middleman deploy[production]`.  (Slightly more powerful than
  `middleman-deploy`, without going all the way to capistratno-based
  solutions, such as `statistrano` and `capistrano-middleman`.)
* Make sure `middleman build` is called before `middleman deploy`, if sources
  have changed since the last build
* Deploy only if needed (e.g., if there are local changes to be published)
* Check the status of your remote git repository before deploying (elimitaing
  the risk of regressions on your website, if your website is managed by
  different people, versioned with git, but not deployed with a post-commit
  hook).
* Keep track of what changes have been made since the last *deploy* (sometimes
  different from what changed since last commit).
* Flexible syntax for deployment


# Installation

1. Download this repository
2. Put the Rakefile in your [middleman](http://middlemanapp.com) website directory
3. Configure it for deployment (see the "Configuration" section)
4. Start using it!


# Typical usage scenario

    rake preview
    rake deploy

# Getting help

    rake --tasks

... you can also get in touch by email, of course!

# Configuration

Before using the `Rakefile` for deployment, you need to set `@deploy_cmd` to
the command used to deploy your website.  Edit the Rakefile and follow the
instructions in the file or the examples below.

**Example 1.** If you are using the `middleman-deploy` gem, you can set this
variable to:

    @deploy_cmd = "middleman deploy"

**Example 2.** If you prefer to invoke `rsync` directly, specify how you want
to inkove the command, using the special symbol `__build_dir__` to indicate
the build directory.  (At runtime, `__build_dir__` will be replaced by the
appropriate build dir.)

For instance:

    @deploy_cmd = "rsync -avz __build_dir__ user@example.com:/dest/dir/"
    
will tell the `Rakefile` that the website is to be deployed in the directory
`/dest/dir` of `example.com`, using the credentials of the user `user`.

Setting this variable in the `Rakefile` simplifies the use of different
environments for deployment.

# Support for different environments

This `Rakefile` supports different environments for building and deploying.

## Environments for building

[Environments](https://middlemanapp.com/basics/upgrade-v4/) are a feature
consolidated in Middleman 4, that allows one to define different settings for
building (or previewing) your website.

To use this feature with this `Rakefile` simply pass the environment name as
argument to the `build` command.  For instance:

    middleman build[production]

will tell middleman to build using the settings for the `production`
environment (which are defined in `config.rb`).

Different from `middleman`, the `Rakefile` generates different build
directories for different environments.  Thus:

    middleman build[staging]
    
will generate the files in the `build-staging` directory.  

The directory used for the `production` environment is `build`, the same you
would expect from running `middleman build` directly.

## Enviroments for deployment

This `Rakefile` allows you to specify different settings for deploying.  This
is useful, for instance, if you want to *stage* your website on a server
before going live.

To use this feature, create a file `_env-<environment name>.rb` containing the
specific settings for `<environment name>` and then invoke `deploy` with the
name of the environment.

For instance:

    middleman deploy[production]
    
will read the settings from `_env-production.rb` and run the deploy task.  You
will typically specify the value of `@deploy_cmd` in these files.

## An Example

My [homepage](http://www.ict4g.net/adolfo) is written in Middleman.  Before
deploying a new version I have defined a `staging` area, where I can test any
modification I make before going live.  (Most of the errors I make and have
difficulties testing with the `preview` task are related to URLs pointing to
assets and other pages.)

To test my website before going live, I have defined a `staging` and a
`production` environment.  The former is where I deploy before going live';
the latter is where my website lives.

The two environments are set as follows:

1. specific build settings in `config.rb`
2. specific deployment commands in `_env-<env>.rb` files

More in details, I have the following snippets in my `config.rb` file:

    configure :production do
        set :http_prefix, 'http://ict4g.net/adolfo'
    end

    configure :staging do
        set :http_prefix, 'http://ict4g.net/adolfo-staging'
    end

which set the `:http_prefix` variable to the URLs used by the two enviroments.

I have also the following two files:

    _env-production.rb
    _env-staging.rb

which define the `@deploy_cmd` for the two environments.

Now the command

    rake deploy[staging]
    
deploys my website in the staging area, where I can test everything works as
expected.  When I am happy I do:

    rake deploy[production]
    
(and then manually delete the `staging` area.)


# Determining whether a build is necessary

This Rakefile tries to be efficient with build and deploy commands, invoking
`middleman build` and `middleman deploy` only when they are necessary.  The
checks are performed as follows:

* a build is necessary if there is a file in `source` or `data` which is more
  recent than the most recent file in `build`
* a deploy is necessary if there is a fil in `build` which is more recent
  thatn the `_last_deploy.txt` file.  (The `_last_deploy.txt` file is updated
  any time a deploy is performed.)

If in doubt, you can use `rake check_timestamps` to see what the Rakefile
would do.  If you can always force a build or a deploy with the `rake
force_build` and `rake force_deploy` commands, respectively.


# Integration with Git

If you are using `git` to manage the sources of your website, the local
repository has to be in sync with the default remote before building and
publishing the website.  If this is not the case, in fact, you might deploy an
old version of the repository, causing a regression of the published content.
One solution to this problem is using a custom `post-commit` hook, which
deploys the website after a successful push: this is what
[Github](http://www.github.com) does.  A second solution is using this
Rakefile: in fact it checks whether there are remote changes to be pulled and,
if so, it warns the user.  Additionally, the Rakefile can commit and push on
deploy, if the variable `$git_autopush` is `true`.

# Change Log

* **version 1.1** introduces support for environments and eliminates the
  dependency from the `middleman-deploy` gem for deployment (you can still use
  it, if you want)
* **version 1.0** is the first released version

# License

Distributed under the terms of the [MIT License](http://opensource.org/licenses/MIT).
