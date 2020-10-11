# SSST

A simple static site tool to maintain websites based on markdown and pandoc

# Description

Given a set of posts or pages written in markdown, SSST generates the corresponding HTML files as well as any monthly archives, tag and category files. It also generates a home page summarising recent posts, and a main archive file linking to all existing monthly archives.

Depending in the actual structure of the source tree, SSST mimics the directory structure typically used in WordPress hosted sites. This allows one to use SSST to create a static replacement for a site that was previously managed using WordPress, ensuring that any external links that point to a page remain functional.

The conversion is driven by `pandoc`, using template files (that can be changed to customise website layout).

The resulting website is completely static: no PERL or PHP or whatever is required on the webserver. Also, no JavaScript is used. All you need to do is to push the generated HTML to the webserver. 

SSST does not unnecessarily touch generated output files, ensuring that when mirroring the generated HTML files to the webserver (through rsync or FTP mirroring options), only files whose content have actually changed have fresh timestamps and will be considered for upload.

Even though the website is static, SSST also allows users to comment on posts through mail when they click on automatically generated `mailto` links embedded in posts and comments. These links ensure that the path to the post or comment replied to is automatically included in the subject field. This allows you to quickly insert the comment at the right place after moderation.

SSST also processes LaTeX equations in a post, replacing them with SVG images containing the rendered equation in the output HTML. This uses `pdflatex` and `pdf2svg`. If an equation occurs on a single page more than once, only on image is generated and used for every occurrence. (This does not work accross multiple pages containing the same equation.)

# Usage

SSST processes all files in a source tree rooted at `SOURCE`, and writes its output into a destination tree rooted at `DESTINATION`.

```
usage: ssst [-h] [-s SOURCE] [-d DESTINATION] [-r ROOT] [-t TEMPLATES]
            [-v VERBOSITY] [-l LOGFILE] [-f] [-x] [-z SUMMARYLENGTH]

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Path to the root of the source files tree
  -d DESTINATION, --destination DESTINATION
                        Path to the root of the destination files tree
  -r ROOT, --root ROOT  Root where the site is hosted. If omitted, the root is
                        found using relative addressing, making the site
                        relocateable
  -t TEMPLATES, --templates TEMPLATES
                        Path to directory with templates
  -v VERBOSITY, --verbosity VERBOSITY
                        Verbosity (-1, the default, is silent)
  -l LOGFILE, --logfile LOGFILE
                        Name of file to store all log messages in
  -f, --force           (Re)make everything
  -x, --strict          Abort after warning
  -z SUMMARYLENGTH, --summarylength SUMMARYLENGTH
                        Number of lines in post (including YAML header) to
                        include in a summary
```

# Input

SSST processes markdown files containing posts or pages. SSST expects these files to start with a YAML block containing at least the post/page title (using the `title:` key) and date (using the `date:` key). Values for these keys are  strings (enclosed by single quotes).

The string used as date must either be specified as ISO 8601 dates (like yyyy-mm-dd) or can be specified like this example 'Sun, 19 Apr 2020 08:42:00 +0000'

Any page categories can be specified using the `categories:` key, and any page tags can be specified using the `keywords:` key. The latter two keys expect lists as values, i.e. enumerations separated by comma's enclosed by square brackets. 

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

- Pages are stored in folders with the following structure
  `./<optional path>/title.md`
- Posts are stored in folders with the following structure
  `./yyyy/mm/dd/title/index.md`.
  Make sure the date inside the post is correct
- Comments are stored in the folder containing the posts they belong to
  `./yyyy/mm/dd/title/comment.x.y.mdc`,
  where `x.y.` is the comment numbering scheme. Here .1 is first main comment, .2 second main comment, ...  and so `comment.x.y.mdc` is the y-th response to the x-th main comment.
- media stored with post/page in same directory are copied to the output tree 
  (i.e files with extension .jpg, .gif, .pdf and .png)
  keeping their metadata (i.e modification times)
  
# Generated input

Based on the posts in the input tree, the following additional input is generated (all in markdown format, but with extension `.sst` to indicate they
are generated by SSST and should not be treated like orginal posts)

- monthly archives in `./yyyy/mm/index.sst`
- a list of all monthly archives in `./archives.sst`
- category files listing all posts with a certain category in `./category/<categorystring>/index.sst`
- tag files listing all posts with a certain tag in `./tag/<tagstring>/index.sst`

Care is taken to only overwrite these files (and change its modification time) if their content actually changes (to ensure that the corresponding HTML output file is only (re)generated when necessary.

# Output

SSST generates the following output in the destination tree

- Posts are stored in
  `./yyyy/mm/dd/title/index.html`
  which contains the main content as well as all comments;
  all media referenced (and equations generated) are stored/copied to `./yyyy/mm/dd/title/`
  Pointers to the next and previous post are automatically added, as well as pointers to the home page, the catalogue of all archives, and a comment link.
- Pages (essentially undated posts) are stored in
  `./<optional path>/title/index.html` 
- Tag pages are stored in
  `./tag/<tag>/index.html`
  and list links to all posts tagged with this tag, in chronological order
- Category pages are stored in
  `./category/<category>/index.html`
  and list links to all posts belonging to this category, in chronological order
- Monthly archives are stored in
  `./yyyy/mm/index.html`
- The catalogue of all existing monthly archives is stored in
  `./archives.html`
- The home page  is stored in
  `./index.html`
  and contains a summary of the five most recent posts.
  
Unlike WordPress, any links to posts and pages generated by SSST contain an explicit `index.html` at the end where needed; this allows the generated content to clicked through locally.

# Templates

TBD

# Processing equations

TBD

# Dependencies

This is a program written in Python (version 3), and depends on

- pandoc
- pdf2svg
- PyYaml


