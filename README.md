# developer.linode.com

This is the Linode developer portal. It's a jekyll site hosted on GitHub Pages.
Included here are:

* API documentation
* API guides
* API libraries

## Feedback

Feedback on our API, including feature requests and bug reports, may be
submitted as [GitHub issues](https://github.com/linode/developers/issues/new)
on this repository.

## Cloning

This repository uses git submodules. Use a recursive clone:

    git clone --recursive https://github.com/Linode/developers.git

If you already cloned, but missed this step, try this:

    git submodule update --init

## Running

To run locally, we follow the [Github Pages recommendation](https://help.github.com/articles/using-jekyll-with-pages/) of using Bundler.

1. Ensure you have the latest version of Bundler installed

    ```bash
    gem update --system && gem install bundler
    ```
2. Use Bundler to install all required gems
    
    ```bash
    bundle install
    ```
    * This installs all required gems locally to ./vendor to avoid interfering or colliding with system installed gems  
    
3. Start the [Jekyll](http://jekyllrb.com/) server and [guard-livereload](https://github.com/guard/guard-livereload) plugin.
    
    ```bash
    bundle exec guard
    ```
    
4. Now you can view the Docs at [localhost:4000](http://localhost:4000)
    * Changes to assets will trigger a [LiveReload](http://feedback.livereload.com/knowledgebase/articles/86174-livereload-protocol) event.
    * The browser will auto-reload once Jekyll finishes processing the changes.
    * No browser extensions are required for this to work thanks to [rack-livereload](https://github.com/johnbintz/rack-livereload). 

## Contributing

Fork this repository and send a pull request, simple as that.
