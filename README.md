
# Markdown blog plugin for magic.web

This plugin allows you to easily implement Markdown blog support for your dynamically rendered Hyperlambda websites.
Each blog article is written by creating a Markdown file within some folder, with frontmatter parts
declaring the title of the article and its content such as follows.

```markdown
---
title: Hello world
---
This is an example blog.

1. This is some numbered list
2. This is another list item

This is a [hyperlink](https://aista.com) leading to Aista's website.
```

The plugin contains several _"mixin"_ components that can be used as follows.

## Listing blogs

Assuming you have this plugin inside of _"/etc/plugins/magic.web.markdown-blogs/"_, and your actual
articles inside of _"/etc/articles/"_ this mixin allows you to render a list of blogs in your installation
such as follows.

**/index.hl**

```
.list-blogs
   io.file.mixin:/etc/plugins/magic.web.markdown-blogs/list.html
      .root-url:/blog/
      .root-folder:/etc/articles/
   return:x:-
```

The above **[.root-url]** is the root URL of where you want users to be able to read your blogs, and
the above **[.root-folder]** parts is the physical folder on disc where you keep your Markdown files.
The above could be referenced in your HTML such as follows.

**/index.html**

```html
<div class="blogs">
{{*/.list-blogs}}
</div>
```

The above will return a bulleted list of all articles it can find in your _"/etc/articles/"_ folder.
The **[.root-url]** argument is the root URL of where individual blogs can be found. If you for
instance have a file named _"/etc/articles/hello-world.md"_ the above will result in the following
root URL _"/blogs/hello-world"_. Below is an example of its output.

```html
<ul>
  <li><a href="/blog/hello-world">Hello World</a></li>
</ul>
```

## Displaying blogs

To actually resolve individual blogs, you'll need a _"default.hl"_ file, a _"default.html"_
file, and an _"interceptor.hl"_ file that can be found in whatever **[.root-url]** folder you choose
to render your blogs from. Below is an example of all 3 required files.

**/blog/interceptor.hl**

```
/*
 * Interceptor for blogs, doing all the heavy lifting, actually
 * loading articles, by intercepting the main blog Hyperlambda file.
 */

// Executing get-article Hyperlambda file, putting content into [.blog] node.
.blog
add:x:@.blog
   io.file.execute:/etc/plugins/magic.web.markdown-blogs/get-article.hl
      .root-folder:/etc/blogs/

// Interceptor node, replaced by default.hl Hyperlambda content.
.interceptor
```

**/blog/default.html**

```
<html>
  <head>
    <title>{{@.blog/*/title}}</title>
  </head>
  <body>
    <h1>{{@.blog/*/title}}</h1>
    <div>
{{@.blog/*/content}}
    </div>
  </body>
</html>
```

**/blog/default.hl**

```
/*
 * Since we don't have any logic in our actual article Hyperlambda file,
 * this file can be empty, since all the heavy lifting is actually done
 * in the interceptor file, and expressions in HTML file are leading
 * directly to to nodes in our interceptor file.
 *
 * However, the file still needs to exist on disc, otherwise the endpoint
 * resolver will load the file as static content, and never return the file
 * as a mixin.
 */
```

The point with the above, is that your interceptor loads the article's content once,
and then adds the semantic content from your Markdown file into its **[.blog]** node,
before the default URL resolver executes, which at that point can reference values
from your blog, such as its content, title, etc.
