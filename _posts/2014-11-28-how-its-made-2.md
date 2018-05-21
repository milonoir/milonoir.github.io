---
title: How it's made? part 2
layout: post
lang: en
ref: how-its-made2
date: 2014-11-28 11:07:51 +0100
categories: archive
---

Since [part one]({% post_url 2014-10-03-how-its-made-1 %}), I’ve been thinking much how to continue. There is a lot of information out there (tutorials and books) about how to set up a *very basic* Django blog, so I just don’t want to line up. In my posts I suppose the reader has already gone through one of them, releasing me from describing the very basics. I’d rather share how-tos of features that helps a blog to make a step forward.

A typical *very basic* blog’s `models.py` file looks something like this:

```python
from django.db import models

class Story(models.Model):
    title = models.CharField(max_length=100)
    slug = models.SlugField()
    content = models.TextField(blank=True)
    created = models.DateTimeField(default=datetime.datetime.now)
```

It can be seen that it’s not too complex. What we need here is a title, a [slug](http://en.wikipedia.org/wiki/Semantic_URL#Slug) field for generating permalinks, the content itself and a timestamp.

The `created` field is filled in automatically by the admin interface whenever we create a new post. The `slug` is generated from the `title` - this is set in `admin.py`:

```python
from django.contrib import admin
from models import Story

class StoryAdmin(admin.ModelAdmin):
    ...
    prepopulated_fields = {'slug': ('title')}

admin.site.register(Story, StoryAdmin)
```

When it comes to the content, the story is getting more interesting. At this point, the `content` returns some plain text, which is, admittedly, quite bare. Let’s say we want to see headings, bold and italics letters, images, embedded videos and program source codes with syntax highlighting and line numbers. Let’s simply call them *formatted content*.

### Attention! Here comes what I intended to share in this post. ###

Since there is no available WYSIWYG editor, in order to get tormatted content we have to put HTML tags and CSS codes into the text. It is better if we have CSS sources separately. But the best is to use some text-to-HTML translator, e.g. [Markdown](https://pypi.python.org/pypi/Markdown). Markdown’s syntax is easy to learn. It’s similar to Wiki’s, but it’s more deliberate and user friendly (for me).

Then a question comes up: when does the text-to-HTML translation happen? We’ve got two options:

* The `content` field contains the current Markdown format text and HTML code is generated on the fly for each incoming request; or
* We reserve another field for the HTML content as well, which is generated each time we save the Markdown content.

The problem with the first is that it generates computation load in our server, although with the second we store duplicated content, which consumes more disk space. In case of R-Pi let’s say that the latter is the lesser of two evils. We’ll modify `models.py` by adding a new *non-editable* content field (and we alter the name of the original `content`, too, in spirit of readability)...:

```python
...
class Story(models.Model):
    ...
    markdown_content = models.TextField(blank=True)
    html_content = models.TextField(editable=False)
    ...
```

... and we have it filled in by the admin interface: when we finish editing (when we save) we call Markdown with `markdown_content` in `models.py` (don’t forget to import Markdown). We have to override the `save` method of `Model` class:

```python
from markdown import markdown
...
class Story(models.Model):
    ...
    def save(self, *args):  # yes, we override it
        self.html_content = markdown(text=self.markdown_content, extensions=['codehilite(linenums = True)'])
        super(Story, self).save()  # don’t forget to call super()
```

The `codehilite(linenums = True)` is responsible for syntax highlighting and line numbering of program code snippets. In order to make it work we must do two things:

1. Install [Pygments](https://pypi.python.org/pypi/Pygments) package.
2. Create or [get](https://github.com/richleland/pygments-css) a CSS file for codehilite and link it in our template. For example: `<link rel="stylesheet" type="text/css" href="./codehilite.css">`

There is only one thing left to do. By default Django does not parse any HTML code stored in variables, so we should tell him to do so with `html_content`. We can do this with ease in our template file using the `safe` flag:

```html
<p id="story-body">
    {{ story.html_content|safe }}
</p>
```

And we’re done! Now we have all the formatting features that Markdown is capable of in hand.

> The contents of `urls.py` and `views.py` are irrelevant from the task’s angle. We can find them in any tutorial, that’s why I do not mention here.
