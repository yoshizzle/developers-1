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

To run locally execute the following commands:

*If you don't already have Bundler installed*

```bash
gem install bundler
```

Install [Github-Pages](https://help.github.com/articles/using-jekyll-with-pages/) and start server locally.

```bash
bundler install
bundle exec guard
```
If you install the [LiveReload browser extension](http://livereload.com/extensions/) you can see changes refreshed in the browser as you save your changes.

Now you can view the Docs at [localhost:4000](http://localhost:4000)

## Contributing

Fork this repository and send a pull request, simple as that.
