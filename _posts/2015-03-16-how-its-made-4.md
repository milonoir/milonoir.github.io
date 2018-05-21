---
title: How it's made? part 4
layout: post
lang: en
ref: how-its-made4
date: 2015-03-16 16:32:34 +0100
categories: archive
---

There are more ways to find something in a blog. The most simple method is to create various categories and use them as tags in our posts. We can list these tags in the sidebar on our page and we can filter our posts by clicking on them. First let’s see how we can do the tagging and then how to make the filtering.

### Tagging

The easiest solution is to add a `CharField` attribute to our model:

```python
from django.db import models

class Story(models.Model):
    title = models.CharField(max_length=100)
    slug = models.SlugField()
    category = models.CharField(max_length=50)
    content = models.TextField(blank=True)
    created = models.DateTimeField(default=datetime.datetime.now)
```

However, there is a problem with that: we can only assign only one category to a post. Thank God, there are already numerous remedies exist and one of these is [django-taggit](https://django-taggit.readthedocs.org/en/latest/), which I also use. Unfortunately, django-taggit itself does not contain template tags, which might come in handy so I recommend installing [django-taggit-templatetags](https://github.com/feuervogel/django-taggit-templatetags/
), as well. There is no magic; Taggit do all the work here. Let’s add it to the apps in `settings.py`:

```python
INSTALLED_APPS = (
    ...
    'taggit',
    'taggit_templatetags',
    ...
)
```

Import `TaggableManager` class in our model and replace our simple `category` field (you’d better call it `tags` now):

```python
from taggit.managers import TaggableManager
from django.db import models
...

class Story(models.Model):
...
tags = TaggableManager()
```

If this is done run a migration in order to refresh our database scheme:

```bash
python manage.py migrate
```

From now on we can assign as many tags as we want to our posts in the admin page. The tags can be shown on our page using the following snippet:

{% highlight html %}
{% raw %}
{% for tag in story.tags.all %}
    <a href="{% url 'blog-tagged' tag.slug %}">{{ tag.name }}</a>
{% endfor %}
{% endraw %}
{% endhighlight %}

In the example above the `story` template objects holds one post. The `blog-tagged` URL will be mentioned later in the filtering part. All of the tags have a `name` attribute, which is the string to be shown on the page, and a `slug` field, which plays the same role as in our Story model. The `story.tags.all` is a query which returns a list that contains all tags belonging to the same story.

### Filtering

Under the term of filtering, we will reverse the previous query: which stories belong to the same tag? We have to create a view and a URL pattern for this. The view:

```python
from django.shortcuts import render_to_response, get_object_or_404
from django.template.context import RequestContext
from taggit.models import Tag
from models import Story

def tagged(request, slug):
    tag = get_object_or_404(Tag, slug=slug)
    story_list = Story.objects.filter(tags__name__in=[tag.name])
    return render_to_response('story_list.html', locals(), context_instance=RequestContext(request))
```

First we obtain the tag by its `slug` attribute using the [`get_object_or_404`](https://docs.djangoproject.com/en/1.6/topics/http/shortcuts/#get-object-or-404) call and then we filter all our stories by keeping only those that contain the tag. Finally we pass these to an appropriate [template](https://docs.djangoproject.com/en/1.6/ref/templates/api/#basics) to render an [HTML response](https://docs.djangoproject.com/en/1.6/topics/http/shortcuts/#render-to-response).

> A __context__ is a “variable name” -> “variable value” mapping that is passed to a template.
>
> A template __renders__ a context by replacing the variable “holes” with values from the context and executing all block tags.

The URL pattern:

```python
url(r'^tags/(?P<slug>[-\w]+)/$', 'tagged', name='blog-tagged')
```

The URL pattern above consists of three parts:

1. A regular expression, which can match to an incoming URL request. The `(P<slug>[-\w]+)` part is a named group, which matches to a combination of `-` and alphanumeric characters and gets the name: slug.
2. The name of the view (see above in the view code we used `tagged` for the function name): this view has to be called if the regexp matches.
3. The `name` argument is an alias, which we used in the template (see above in the template code). Further explanation [here](https://docs.djangoproject.com/en/1.6/topics/http/urls/#reverse-resolution-of-urls).

With the following template snippet all posts related to the topic will be shown:

{% highlight html %}
{% raw %}
{% if tag %}
    <p>{{ tag.name }}</p>
{% endif %}
...
{% for story in story_list %}
    <p><a href="{{ story.get_absolute_url }}">{{ story.title }}</a></p>
{% empty %}
    <p>No posts.</p>
{% endfor %}
{% endraw %}
{% endhighlight %}
