---
layout: post
title: "Plugin directory missing after deploy"
author: Tom
date: 2012-05-18 15:56
comments: true
categories: 
---
Last week I was working on bringing a sadly neglected Rails application up to date. Among other issues it was stored in a Subversion repository. I didn't care very much about the history (so much for all my carefully worded commit messages), so I just exported the the trunk, copied the exported files into another directory, created a new local Git repo, added everything, and pushed it to a remote repo for a backup. Then I modified the `config/deploy.rb` to reflect Git, and deployed to production (no staging environment since it's a personal project).

I checked the site, and oh no, there was an error - something about class method "acts_as_list" not found. Wha? What I should have done at this point was a quick `cap deploy:rollback`, but instead I ssh'd in and started poking around. This was an old app, so `acts_as_list` was in `vendor/plugins` rather than using the gem. I checked `vendor/plugins/acts_as_list` and it was empty! A quick `cap deploy:upload deploy:restart FILES=vendor` got the site back up, but why was that directory not getting deployed?

Like all these things, it was simple once I figured out the problem. When I had added that plugin in Subversion I had cloned a Git repo and not removed the `.git` directory before committing the code. Thus when I exported the Subversion repo, the `.git` directory came along too. When I did my `git add .` to add all the code to the new Git repo, it didn't add that subdirectory because there was already a repo there. Everything worked fine locally, but that directory wasn't getting pushed to the remote Git repository and thus wasn't included when Capistrano checked out the code. Looking back on it, another way to troubleshoot this would have been to clone the repo from the remote; then I would have seen that the `vendor/plugins/acts_as_list` directory was empty.

I guess the takeaways are 1) having a staging deploy and use it 2) when you add something to Subversion, don't include the `.git` directory and 3) use `cap deploy:rollback` when a deploy goes wrong.

Oh, also - if you're using acts_as_list-rails3, you need a specific require in your Gemfile, i.e.:

``` ruby
gem 'acts_as_list-rails3', :require => 'acts_as_list'
```

That's because the gem name doesn't match the top level file.
