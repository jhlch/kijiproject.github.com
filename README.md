(c) Copyright 2012 WibiData, Inc.

kiji-docs
=========

This repository contains documentation for the [KijiProject](http://www.kiji.org).
This documentation is compiled into a static website using configuration
files Jekyll.

## How words become websites

Our docs will be transformed into readable pages using Jekyll. Jekyll
allows us to write our documentation in markdown, and host it as
static pages in github. We are particularly using
jekyll-bootstrap. [Learn more about jekyll-bootstrap.](http://www.jekyllbootstrap.com)


## Creating and editing documentation
You can write everything in markdown (see markdown_styleguide.md for more
information.) and do code highlighting inline with backticks `code` or
in blocks with the template:

    {% highlight java %}
    block of code goes here
    {% endhighlight %}

Java in the above template can be any short name for a language from
[this list.](http://pygments.org/languages/) 

To add a new file, find the user guide section or article you want and
create a file named YYYY-MM-DD-title.md under the _posts directory,
and write it using markdown syntax. In the file, you should include
the following at the top of the file (include the dashes):

    ---
    layout: post
    title: Delete Contacts
    categories: [tutorials, phonebook-tutorial, 1.0.0-rc4]
    tags: [article]
    order: 8
    description: Examples of Point deletions.
    ---

The above is YAML Front Matter syntax that instructs Jekyll what to do
with the file when compiling it into a static site. The tag allows us to collate
articles and userguides.


## Previewing Changes
In order to preview what your changes look like, you will need to have
Ruby and Jekyll installed. It is highly recommended that you control your
ruby version using rvm. Check out instructions at
http://github.com/mojombo/jekyll.


To see how your local version of kiji-docs renders, run `rake preview`. This
command turns off google analytics, so as not to inflate our stats, and runs 
`jekyll --no-auto --server --pygments --no-lsi --safe`. Since we use the no-auto
parameter, you will need to rerun preview to see new changes, but trust us,
it is better this way.

Note that the pygments highlighting of codeblocks will only work if you have
pygments installed.

## Contributing Documentation

* Fork the [documentation project](https://github.com/kijiproject/kijiproject.github.com).
* File a JIRA for the change you want to make on the [DOCS project](http://jira.kiji.org).
* Create a topic branch: git checkout -b my_fix.
* Refer to markdown_styleguide.md in the parent directory for more on the syntax.
* Make your changes.
* Reference the jira in the commit message (e.g., "DOCS-1: Subscribe buttons to the mailing lists on the website are broken")
* Push your branch: git push origin my_fix.
* Use [pull requests](https://help.github.com/articles/using-pull-requests) to contribute your changes once you are done.

## Maintaing Docs


In order to maintain the docs repo, you need to know:
* How Jekyll works. We use a particular framework called [Jekyll Bootstrap](http://jekyllbootstrap.com/)
  Their docs are great, so use them. 
* How we use front matter on posts. The most important(/hackiest uses) labels and their uses are:
  ** categories : This defines the prefix of a url. For example a post with categories = [a, b, c] and
     file name 2012-01-01-title, will have the full url of {{ site.BASE_URL }}/a/b/c/title.
  ** order : Putting sections of the userguide in the correct order is tricky, and done in a very
     silly way. See _includes/themes/twitter/post.html and _includes/side-toc.html for examples of
     where the order label is used. Jekyll is not made for our use case, so we have some hilarious
     ways of bending its behavior to our will. In the long run, the way we generate the side-toc is
     going to slow down the generation of the site considerably. This is one reason that the current
     Jekyll based docs site is going to need to be replaced with something more resilient in the future.
* The directory structure inside of the _posts doesn't affect the urls that get generated for posts,
  but we make it match the urls as much as possible. Specificly, that means that if the overview
  section of a userguide has the categories tag [userguides, schema, 1.0.0-rc4], then the markdown
  file should be located in _posts/userguides/schema/1.0.0-rc4. This is just a convention that helps
  with organization and navigation.
* Sometimes weird errors happen, adding whitespace will help, about half of the time.
