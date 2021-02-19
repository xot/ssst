# SSST

A simple static site tool to maintain websites based on markdown and pandoc

# Description

Given a set of posts or pages written in markdown, SSST generates the corresponding HTML files as well as any monthly archives, tag and category files. It also generates a home page summarising recent posts, a main archive file linking to all existing monthly archives, and an RSS feed containing a summary of the last few posts.

Depending in the actual structure of the source tree, SSST mimics the directory structure typically used in WordPress hosted sites. This allows one to use SSST to create a static replacement for a site that was previously managed using WordPress, ensuring that any external links that point to a page remain functional. (This structure, where paths to posts look like `./yyyy/mm/dd`, is assumed by SSST, and used to distinguish *posts* from *pages*.)

The conversion is driven by `pandoc`, using template files (that can be changed to customise website layout).

The resulting website is completely static: no PERL or PHP or whatever is required on the webserver. Also, no JavaScript is used. All you need to do is to push the generated HTML to the webserver. 

SSST does not unnecessarily touch generated output files, ensuring that when mirroring the generated HTML files to the webserver (through rsync or FTP mirroring options), only files whose content have actually changed have fresh timestamps and will be considered for upload.

Even though the website is static, SSST also allows users to comment on posts through mail when they click on automatically generated `mailto` links embedded in posts and comments. These links ensure that the path to the post or comment replied to is automatically included in the subject field. This allows you to quickly insert the comment at the right place after moderation.

SSST also processes LaTeX equations in a post, replacing them with SVG images containing the rendered equation in the output HTML. This uses `pdflatex` and `pdf2svg`. If an equation occurs on a single page more than once, only on image is generated and used for every occurrence. (This does not work across multiple pages containing the same equation.)

# Usage

## Creating the output

`ssst` processes all files in a source tree rooted at `SOURCE`, and writes its output into a destination tree rooted at `DESTINATION`.

```
usage: ssst [-h] [-s SOURCE] [-d DESTINATION] [-r ROOT] [-t TEMPLATES]
            [-v VERBOSITY] [-l LOGFILE] [-f] [-g] [-k] [-p] [-z SUMMARYLENGTH]

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Path to the root of the source files tree
  -d DESTINATION, --destination DESTINATION
                        Path to the root of the destination files tree
  -r ROOT, --root ROOT  Root where the site is hosted. If omitted, the root is
                        found using relative addressing, making the site
                        relocatable
  -t TEMPLATES, --templates TEMPLATES
                        Path to directory with templates
  -v VERBOSITY, --verbosity VERBOSITY
                        Verbosity (-1, the default, is silent)
  -l LOGFILE, --logfile LOGFILE
                        Name of file to store all log messages in
  -f, --force           (Re)make everything
  -g, --gladtex         Use pandoc --gladtex to process LaTeX equations.
  -k, --keepsimple      Do not process simple LaTex equations
  -p, --pedantic        Abort after warning
  -z SUMMARYLENGTH, --summarylength SUMMARYLENGTH
                        Number of lines in post (including YAML header) to
                        include in a summary
```

## Adding new posts

The `ssst-add` utility adds a markdown post to its appropriate place in the source tree (using the current date or the date in the post header), and does some consistency checks.

```
usage: ssst-add [-h] [-s SOURCE] [-v VERBOSITY] [-p] post

positional arguments:
  post                  Filename of post to add

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Path to the root of the source files tree
  -v VERBOSITY, --verbosity VERBOSITY
                        Verbosity (-1, the default, is silent)
  -p, --pedantic        Abort after warning
```

## Adding a comment

The `ssst-comment` utility ads a comment received by email (as generated when a users submits a comment while clicking on SSST generated comment or reply link) in the appropriate file in the source tree. It uses the From and Date header in the email to set the author and date for the comment. The contents of the email are used as comment text.

```
usage: ssst-comment [-h] [-s SOURCE] [-v VERBOSITY] [-l LOGFILE] [-p] mail

positional arguments:
  mail                  Mail containing comment to add

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Path to the root of the source files tree
  -v VERBOSITY, --verbosity VERBOSITY
                        Verbosity (-1, the default, is silent)
  -l LOGFILE, --logfile LOGFILE
                        Name of file to store all log messages in
  -p, --pedantic        Abort after warning
```

## Replacing a tag/keyword

The `ssst-replacetag` utility replaces all occurrences of a tag/keyword in all posts in the source tree with the supplied replacement.

```
usage: ssst-replacetag [-h] [-s SOURCE] [-v VERBOSITY] [-l LOGFILE] [-p] [-b]
                       keyword replacement

SSST-replacetag. Replace a tag in a post.

positional arguments:
  keyword               Keyword
  replacement           Replacement keyword

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Path to the root of the source files tree
  -v VERBOSITY, --verbosity VERBOSITY
                        Verbosity (1, the default, reports which replacements
                        took place)
  -l LOGFILE, --logfile LOGFILE
                        Name of file to store all log messages in
  -p, --pedantic        Abort after warning
  -b, --backup          Keep bakcup of source file when changed.
```

# Input

SSST processes markdown files containing posts or pages. SSST expects these files to start with a YAML block containing at least the post/page title (using the `title:` key). Posts also have a date (using the `date:` key). Pages do not.
Values for these keys are strings (enclosed by single quotes).

The string used as date must either be specified as ISO 8601 dates (like yyyy-mm-dd) or can be specified like this example 'Sun, 19 Apr 2020 08:42:00 +0000' (which is the format WordPress uses, also in its data dumps).

Any page categories can be specified using the `categories:` key, and any page tags can be specified using the `keywords:` key. The latter two keys expect lists as values, i.e. enumerations separated by comma's enclosed by square brackets (or one entry on a single line, starting with a hyphen). 

The post content follows the YAML header. Below is an example

```
---
title: 'Maintaining a static self-hosted site'
date: 'Sun, 19 Apr 2020 08:42:00 +0000'
categories: [documentation]
tags: [static, self-hosted, markdown]
---
post content
```

# Structure of the source tree

The source tree should be structured as follows

- The root of the source tree should contain a file `./index.md` which drives the generation of the home page (see below). 
- *Pages* are stored in folders with the following structure
  `./<optional path>/title.md`
- *Posts* are stored in folders with the following structure
  `./yyyy/mm/dd/title/index.md`.
  Make sure the date inside the post is correct!
- Comments are stored in the folder containing the posts they belong to
  `./yyyy/mm/dd/title/comment.x.y.mdc`,
  where `x.y.` is the comment numbering scheme. Here .1 is first main comment, .2 second main comment, ...  and so `comment.x.y.mdc` is the y-th response to the x-th main comment.
- Media stored with post/page in the same directory are copied to the output tree (i.e files with extension `.jpg`, `.gif`, `.pdf` and `.png`)
  keeping their metadata (i.e modification times).


# Generated input

Based on the posts in the input tree, the following additional input is generated (all in markdown format, but with extension `.sst` to indicate they
are generated by SSST and should not be treated like original posts)

- monthly archives in `./yyyy/mm/index.sst`
- a list of all monthly archives in `./archives.sst`
- category files listing all posts with a certain category in `./category/<categorystring>/index.sst`
- tag files listing all posts with a certain tag in `./tag/<tagstring>/index.sst`

Care is taken to only overwrite these files (and change their modification time) if their content actually changes (to ensure that the corresponding HTML output file is only (re)generated when necessary.

# Output

SSST generates the following output in the destination tree. All output is HTML generated using `pandoc` using different templates. 

- Posts are stored in
  `./yyyy/mm/dd/title/index.html`
  which contains the main content as well as all comments;
  all media referenced (and equations generated) are stored/copied to `./yyyy/mm/dd/title/`.
  Pointers to the next and previous post are automatically added, as well as pointers to the home page, the catalogue of all archives, and a comment link.
  The conversion uses template `<templates>/ssst-post.html`.
- Pages (essentially undated posts) are stored in
  `./<optional path>/title/index.html`.
  The conversion uses template `<templates>/ssst-page.html`.
- Tag pages are stored in
  `./tag/<tag>/index.html`
  and list links to all posts tagged with this tag, in chronological order
  The conversion uses template `<templates>/ssst-page.html`.
- Category pages are stored in
  `./category/<category>/index.html`
  and list links to all posts belonging to this category, in chronological order
  The conversion uses template `<templates>/ssst-page.html`.
- Monthly archives are stored in
  `./yyyy/mm/index.html`.
  The conversion uses template `<templates>/ssst-page.html`.
- The catalogue of all existing monthly archives is stored in
  `./archives.html`.
  The conversion uses template `<templates>/ssst-archives.html` if it exists, or `<templates>/ssst-page.html` otherwise.
- The catalogue of all existing categories is stored in
  `./categories.html`.
  The conversion uses template `<templates>/ssst-categories.html` if it exists, or `<templates>/ssst-page.html` otherwise.
- The catalogue of all existing tags is stored in
  `./tags.html`.
  The conversion uses template `<templates>/ssst-tags.html` if it exists, or `<templates>/ssst-page.html` otherwise.
- The RSS feed is stored in 
  `./feed/index.html`. 
  The conversion uses template `<templates>/ssst-rss.html` for the  main feed and `<templates>/ssst-rss-entry.html` to generate individual feed items.

If a post contains a `template` key in its YAML header, the value for that key is used to override the default template (as described above).

When generating posts or pages, any LaTeX equations (that are enclosed by $..$ in the markdown input) are automatically processed. Any equation that is 'too complex' to render natively (or any equation when specifying the `-g` or `--gladtex` option) will be converted using `pdflatex` and `pdf2svg`. The conversion process is controlled by a template `<templates>/ssst-eq.tex`, which should contain a complete LaTeX document, where the first occurance of `$$` is replace by the equation to convert. (The standard template uses the LaTeX standalone class.)

**Warning**: *When SSST decides that an item needs to be (re)made, the destination folder is erased (except for any subdirectories it contains). As this may occasionally happen for items in the root folder as well, it is unwise to store important content like style sheets there. Store those in a separate folder.*

Unlike WordPress, any links to posts and pages generated by SSST contain an explicit `index.html` at the end where needed; this allows the generated content to clicked through locally.


## Generating the home page

The home page is generated using as input the file `./index.md` in the root of the source tree. This markdown file, whose title and subtitle field in the YAML header are used as blog title and subtitle, is converted to HTML using pandoc using template `<templates>/ssst-homepage.html`. The home page generated is stored in `./index.html` in the root of the destination tree.
A summary of the five most recent posts is appended. Each summary is converted using template `<templates>/ssst-summary-entry.html` before it is included in the home page. (This also implies that the HTML generated for summaries should not be full, self-contained, HTML documents.)

# Dependencies

This is a program written in Python (version 3), and depends on

- [pandoc](https://pandoc.org) to convert markdown to html.
- [pdflatex](https://www.tug.org/applications/pdftex/) to convert LateX equations to pdf.
- [pdf2svg](https://github.com/dawbarton/pdf2svg) to convert equations in pdf to SVG .
- [PyYaml](https://pyyaml.org) to parse YAML headers.
- The Unix `head` command, to summarise posts.

# Implementation notes

- 
