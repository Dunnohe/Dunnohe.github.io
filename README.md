# Minimal Mistakes

**[Minimal Mistakes](http://mmistakes.github.io/minimal-mistakes)** is a two column responsive Jekyll theme perfect for powering your GitHub hosted blog. 

## Minimal Mistakes is all about:

* Responsive templates. Looking good on mobile, tablet, and desktop.
* Gracefully degrading in older browsers. Compatible with Internet Explorer 8+ and all modern browsers. 
* Minimal embellishments -- content first.
* Optional large feature images for posts and pages.
* Simple and clear permalink structure.
* [Custom 404 page](http://mmistakes.github.io/minimal-mistakes/404.html) to get you started.
* Support for Disqus Comments

![screenshot of Minimal Mistakes theme](http://mmistakes.github.io/minimal-mistakes/images/mm-theme-post-600.jpg)

See a [live version of Minimal Mistakes](http://mmistakes.github.io/minimal-mistakes/) hosted on GitHub.

## Getting Started

Minimal Mistakes takes advantage of Sass and data files to make customizing easier. These features require Jekyll 2.x and will not work with older versions of Jekyll.

To learn how to install and use this theme check out the [Setup Guide](http://mmistakes.github.io/minimal-mistakes/theme-setup/) for more information.

##run
* cd myblog
* ~/myblog $ jekyll serve
* Now browse to http://localhost:4000

##Directory Structure
* minimal-mistakes/
* ├── _includes/
* |    ├── _author-bio.html        # bio stuff layout. pulls optional owner data from _config.yml
* |    ├── _browser-upgrade.html   # prompt to install a modern browser for < IE9
* |    ├── _disqus_comments.html   # Disqus comments script
* |    ├── _footer.html            # site footer
* |    ├── _head.html              # site head
* |    ├── _navigation.html        # site top navigation
* |    ├── _open-graph.html        # Twitter Cards and Open Graph meta data
* |    └── _scripts.html           # site scripts
* ├── _layouts/
* |    ├── home.html               # homepage layout
* |    ├── page.html               # page layout
* |    ├── post-index.html         # post index layout
* |    └── post.html               # single post layout
* ├── _posts/                      # MarkDown formatted posts
* ├── _sass/                       # Sass stylesheets
* ├── _templates/                  # used by Octopress to define YAML variables for new posts/pages
* ├── about/                       # sample about page
* ├── assets/
* |    ├── css/                    # compiled stylesheets
* |    ├── fonts/                  # webfonts
* |    ├── js/
* |    |   ├── _main.js            # main JavaScript file, plugin settings, etc
* |    |   ├── plugins/            # scripts and jQuery plugins to combine with _main.js
* |    |   ├── scripts.min.js      # concatenated and minified _main.js + plugin scripts
* |    |   └── vendor/             # vendor scripts to leave alone and load as is
* |    └── less/
* ├── images/                      # images for posts and pages
* ├── 404.md                       # 404 page
* ├── feed.xml                     # Atom feed template
* ├── index.md                     # sample homepage. lists 5 latest posts
* ├── posts/                       # sample post index page. lists all posts in reverse chronology
* └── theme-setup/                 # theme setup page. safe to remove
