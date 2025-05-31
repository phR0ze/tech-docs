# Office <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Documenting my experience with Office applications in the Linux world

### Quick links
- [.. up dir](..)
* [mdbook](#mdbook)

### Linked pages
- [PDFs](pdfs/README.md)

# mdbook
[mdbook](https://github.com/rust-lang/mdBook) is a Rust utility to create modern online books from 
Markdown files. It's [documentation is self-hosted](https://rust-lang.github.io/mdBook/). Its clean, 
easy to navgate and customizable for product documentaton, tutorals, course materlas etc...

## mdbook overview
***Features***
* Lightweight Markdown syntax that helps you focus on the content
* Integrated search support
* Color syntax highlighting for code blocks for many languages
* Theme files allowing for customizing the formatting of the output
* Preproccessor for custom syntax and content modification extensions
* Automated testing of Rust code samples

### mdbook install
1. Install with
   ```bash
   $ cargo install mdbook
   ```

### mdbook organization
The top level output is a `book` which is organized into `chapters` with each chapter on a separate 
page. Chapters can be nested into a hierarchy of sub-chapters. Typically each chapter will be 
organized into a series of headings to subdivide a chapter.

The sidebar on the left provides a list of all chapters.

* ***book.toml*** is the root of your book which contains settings for describing how to build your 
book.
* ***src/SUMMARY.md*** contains all the chapters in the book. Chapters must be added here to show up. 
Adding a new chapter to the summary will automatically create the associated file. Each chapter is a 
separate markdown file.

## Getting started

### Create your book
1. Create a new book by executing init and answering the questions
   ```bash
   $ mdbook init tech-docs
   ```
2. Serve up the book in a browser. You can leave it running and changes to the files will 
   automatically be rebuilt and shown in the browser
   ```bash
   $ mdbook serve --open
   ```

### Publish your book
Once created you'll want to host your book somewhere. Thus you need to build it which will generate 
HTML in the `book` directory that cna be used on any web server.

<!-- 
vim: ts=2:sw=2:sts=2
-->
