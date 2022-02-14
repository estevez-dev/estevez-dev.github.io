---
title: 'Testing GitHub Pages Jekyll Website Locally on MacOS'
header:
    og_image: /img/teaser_500x300.jpg
---

It could be a simple task following the [docs on GitHub](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll), but while running this website locally for testing on my Mac I faced some issues. To speed up the process in future and to help others with the same task, I want to describe step by step what I have learned today.

# Homebrew

First of all, you need Homebrew on your macOS. And not only for testing your GitHub Pages or Jekyll website but for making your life easier. Just follow the instructions on [brew.sh](https://brew.sh/) if you still don't have it installed.

# Ruby

The next thing you'll need to run Jekyll locally is Ruby. And it could be not an issue as it is already pre-installed in macOS, but this is not the case. The default Ruby installation would block tools like _bundler_ from modifying needed files. Sure, you can `sudo` these issues away, but this is not a recommended way. We need to install our own Ruby environment to be sure everything works as expected.

We can install ruby with a simple `brew install ruby`, but then we'll be forced to manually deal with two Ruby environments changing PATH variable, restarting terminal sessions and so on... Some good guys from around the internet already faced this inconvenience and decided to automate the process making our lives easier.

We'll use [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install) from [postmodern](https://github.com/postmodern)'s repository:

```
brew install chruby ruby-install
```

After the successful installation, we'll get something like this in the output:
```
Add the following to the ~/.bash_profile or ~/.zshrc file:
  source /usr/local/opt/chruby/share/chruby/chruby.sh

To enable auto-switching of Rubies specified by .ruby-version files,
add the following to ~/.bash_profile or ~/.zshrc:
  source /usr/local/opt/chruby/share/chruby/auto.sh

```

Don't ignore this suggestion if you usually don't want or don't need to switch between different Ruby versions.

I'm using _nano_ text editor here because it reminds me about the good old days spent in Ubuntu. You can install it with `brew install nano` or use any other editor of your choice.
{: .notice--info}

I'm using _zsh_ shell, so I'm editing the `.zshrc` file:
```
nano ~/.zshrc
```
to add those lines:
```
source /usr/local/opt/chruby/share/chruby/chruby.sh
source /usr/local/opt/chruby/share/chruby/auto.sh
```

We need to close the current terminal session (closing the window) and start a new one. Where we finally install Ruby:

```
ruby-install ruby
```

At the end of the installation, we'll see the version of Ruby installed. For example:
```
>>> Successfully installed ruby 3.1.0 into /Users/mccoy/.rubies/ruby-3.1.0
```

Use this version to tell `chruby` which Ruby version you want to be automatically activated for every terminal session:
```
echo "ruby-3.1.0" > ~/.ruby-version
```

Now restart the terminal again and:
```
ruby -v
```

It should show the version of Ruby. Just to make sure.

{% include ads_media.html %}

# Jekyll

Install [_bundler_](https://bundler.io/). Check the link to make sure nothing was changed from the time of writing this post:

```
gem install bundler
```

Then install Jekyll itself, with a simple [`gem install jekyll`](https://jekyllrb.com/docs/installation/), make sure it is not working, go to the [macOS installation instructions](https://jekyllrb.com/docs/installation/macos/) and make everything right. Or just make everything right from the beginning:
```
xcode-select --install
```
to install needed command-line tools. Then:
```
export SDKROOT=$(xcrun --show-sdk-path)
```
and finally:
```
gem install jekyll
```

# Launch

Now in the folder of your website:

```
budle install
```

You can also update all your dependencies with:

```
bundle update
```

And launch!

```
bundle exec jekyll serve
```

Or not. On Ruby 3.1.0 as well as on other Ruby 3 versions you can face [an issue](https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417/3) launching Jekyll site generation:

```
bundler: failed to load command: jekyll (/Users/mccoy/.gem/ruby/3.1.0/bin/jekyll)
/Users/mccoy/.gem/ruby/3.1.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

It could be fixed by downgrading to Ruby 2.x or by adding missed dependencies:
```
bundle add webrick
```

Now `bundle exec jekyll serve` should look like this:
```
Configuration file: /Users/mccoy/src/bones.github.io/_config.yml
            Source: /Users/mccoy/src/bones.github.io
       Destination: /Users/mccoy/src/bones.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
      Remote Theme: Using theme mmistakes/minimal-mistakes
       Jekyll Feed: Generating feed for posts
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
                    done in 24.42 seconds.
 Auto-regeneration: enabled for '/Users/mccoy/src/bones.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```
`Server address` is a place where you should see your generated website ready for testing.





