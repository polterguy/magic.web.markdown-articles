
# Markdown blog plugin for magic.web

This plugin allows you to easily implement Markdown blog support for your dynamically rendered Hyperlambda websites.
Each blog article is written by creating a Markdown file within your _"/etc/blogs/"_ folder, with frontmatter parts
declaring the title of the article, and its content such as follows.

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

This mixin allows you to render a list of blogs in your installation such as follows.

**/index.hl**

```
.list-blogs
   io.file.mixin:/etc/www/.plugins/magic.web.markdown-blogs/list.html
      .root-url:/blog/
   return:x:-
```

The above could be referenced in your HTML such as follows.

**/index.html**

```html
<div class="blogs">
{{*/.list-blogs}}
</div>
```

The above will return a bulleted list of all articles it can find in your _"/etc/blogs/"_ folder.
The **[.root-url]** argument is the root URL of where individual blogs can be found. If you for
instance have a file named _"/etc/blogs/hello-world.md"__ the above will result in the following
root URL _"/blogs/hello-world"_. Below is an example of its output.

```html
<ul>
  <li><a href="/blog/hello-world">Hello World</a></li>
</ul>
```

Notice, to actually resolve individual blogs, you'll need a _"default.hl"_ file and a _"default.html"_
file, that can be found in whatever **[.root-url]** folder you choose to render your blogs within.
