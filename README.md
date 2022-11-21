
# Markdown article plugin for magic.web

This plugin allows you to easily implement Markdown article support for your dynamically rendered Hyperlambda
websites. Each article is supplied by creating a Markdown file within some folder, with front matter parts
declaring the title of the article, and its content such as follows.

```markdown
---
title: Hello world
---
This is an example article.

1. This is some numbered list
2. This is another list item

This is a [hyperlink](https://aista.com) leading to Aista's website.
```

The plugin contains several _"mixin"_ components that can be used as follows.

## Listing articles

Assuming you have this plugin inside of _"/etc/plugins/magic.web.markdown-articles/"_, and your actual
articles inside of _"/etc/articles/"_ this mixin allows you to render a list of articles in your
installation such as follows.

**/index.hl**

```
.list-articles
   io.file.mixin:/etc/plugins/magic.web.markdown-articles/list.html
      .root-url:/articles/
      .root-folder:/etc/articles/
   return:x:-
```

The above **[.root-url]** is the root URL of where you want users to be able to read your articles, and
the above **[.root-folder]** parts is the physical folder on disc where you keep your Markdown files.
The above could be referenced in your HTML such as follows.

**/index.html**

```html
<div class="articles">
{{*/.list-articles}}
</div>
```

The above will return a bulleted list of all articles it can find in your _"/etc/articles/"_ folder.
The **[.root-url]** argument is the root URL of where individual blogs can be found. If you for
instance have a file named _"/etc/articles/hello-world.md"_ the above will result in the following
root URL _"/articles/hello-world"_. Below is an example of its output.

```html
<ul>
  <li><a href="/articles/hello-world">Hello World</a></li>
</ul>
```

You can also invoke the list blog mixin with a **[.verbose]** argument with a value of `true`,
at which point the returned HTML will resemble the following.

```html
<ul>
  <li>
    <a href="/articles/hello-world">
      <h3>Hello world</h3>
      <img src="https://aista.com/wp-content/uploads/2022/11/twitter-elon.jpg" alt="Hello world">
      <span>This is an excerpt</span>
    </a>
  </li>
</ul>
```

Notice, if you do, your article Markdown files needs to have at least the following front matter
parts declared.

```markdown
---
title: Hello world
excerpt: This is an excerpt
image: https://aista.com/wp-content/uploads/2022/11/twitter-elon.jpg
---
... article content ...
```

## Displaying articles

To actually resolve individual articles, you'll need a _"default.hl"_ file, a _"default.html"_
file, and an _"interceptor.hl"_ file that can be found in whatever **[.root-url]** folder you choose
to render your articles from. Below is an example of all 3 required files.

**/articles/interceptor.hl**

```
/*
 * Interceptor for articles, doing all the heavy lifting, actually
 * loading articles, by intercepting the main blog Hyperlambda file.
 */

// Executing get-article Hyperlambda file, putting content into [.blog] node.
.article
add:x:@.article
   io.file.execute:/etc/plugins/magic.web.markdown-articles/get-article.hl
      .root-folder:/etc/articles/

// Interceptor node, replaced by default.hl Hyperlambda content.
.interceptor
```

**/articles/default.html**

```
<html>
  <head>
    <title>{{@.article/*/title}}</title>
  </head>
  <body>
    <h1>{{@.article/*/title}}</h1>
    <div>
{{@.article/*/content}}
    </div>
  </body>
</html>
```

**/articles/default.hl**

```
/*
 * Since we don't have any logic in our actual article Hyperlambda file,
 * this file can be empty, since all the heavy lifting is actually done
 * in the interceptor file, and expressions in HTML file are leading
 * directly to nodes in our interceptor file.
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
