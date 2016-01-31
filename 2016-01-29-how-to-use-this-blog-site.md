---
title: "How to Use this Blog Site"
created_at: 2016-01-29 18:27:58 +0800
kind: article
---

TL;DR

0. Download this repo (as zip or clone)
0. Install Dependencies: `rake bootstrap`
0. Configure `nanoc.yaml` (just 3 lines)
0. Write with your favorite editor
0. Generate site with one line: `nanoc`
0. **DONE!**

Too easy for anyone!

<!-- more -->

### Setup:

First you need to have Ruby, Python and Bundler working.

If you have installed `rake`, run the following command to install dependencies:

```sh
rake b # b for bootstrap
# It installs the dependencies with `pip` and `bundle`
# See Rakefile for more details

# Go to next section for configuring Nanoc Blog Site
```

Otherwise, install dependencies with `bundler` and `pip`:

```sh
pip install Pygments
bundle install
```

### Configure

You will find `nanoc.yaml` under the root folder.

To use Nanoc Blog Site, you only need to set the `title`, `tagline` and
`description` in the `site` section to your preference.

```yaml
# nanoc.yaml

site:
  title: "Nanoc Blog Site"
  tagline: 'Created by <a href="https://github.com/NicholasTD07">Nicholas T.</a>'
  description: |
    Create a site with Nanoc.<br>
    Write in Markdown and HAML.<br>
```

### Usage:

* Use `rake new["Post Name"]` to create new posts in `content/posts/`
* Edit the new post with your favorite editor
* Run `bundle exec guard` to re-gen the site when there's any change to the source of the site
* Run `nanoc view` to create a local server which serves the generated site

### How was it built?

See [this post](../2016-01-25-new-blog-site-with-nanoc/) for more details of how
Nanoc Blog Site was built.
