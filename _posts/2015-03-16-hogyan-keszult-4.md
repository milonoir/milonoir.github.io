---
title: Hogyan készült? 4. rész
layout: post
lang: hu
ref: how-its-made4
date: 2015-03-16 16:32:34 +0100
categories: archívum
---

Egy blogban keresni több módon is lehet. Az egyik legegyszerűbb, ha témakör alapján kategóriákat hozunk létre, majd ezekkel felcímkézzük a bejegyzéseinket. Ezután a kategóriákat vagy címkéket felsoroljuk mondjuk egy oldalsávban és ezekre kattintva egy szűrést végzünk a posztok listáján. Először nézzük meg hogyan lehet megcsinálni a címkézést, aztán pedig, hogy hogyan tudjuk velük a szűrést intézni!

### Címkézés

A legegyszerűbb megoldás, ha egy egyszerű `CharField` mezőt hozzácsapunk a meglévő modellünkhöz:

```python
from django.db import models

class Story(models.Model):
    title = models.CharField(max_length=100)
    slug = models.SlugField()
    category = models.CharField(max_length=50)
    content = models.TextField(blank=True)
    created = models.DateTimeField(default=datetime.datetime.now)
```

Ezzel csak egy probléma van: maximum 1 db kategóriát tudunk hozzárendelni egy bejegyzéshez. Szerencsére létezik már több kész megoldás a problémára, ezekből az egyik a [django-taggit](https://django-taggit.readthedocs.org/en/latest/), amit én is használok. Sajnos a django-taggit önmagában nem tartalmaz template tageket, azonban később jól jöhetnek, így én javaslom a [django-taggit-templatetags](https://github.com/feuervogel/django-taggit-templatetags/
) telepítését is. Nincs különösebb varázslat, a Taggit elintéz mindent. Először is hozzá kell adni az appokhoz a `settings.py`-ban:

```python
INSTALLED_APPS = (
    ...
    'taggit',
    'taggit_templatetags',
    ...
)
```

A modellünkbe importáljuk be a `TaggableManager` osztályt és cseréljük le vele az egyszerű `category` mezőt (nevezzük inkább `tags`-nek mostantól):

```python
from taggit.managers import TaggableManager
from django.db import models
...

class Story(models.Model):
...
tags = TaggableManager()
```

Ha ez megvan rögtön futtassunk is egy migrálást, hogy az adatbázisunk sémája frissüljön:

```bash
python manage.py migrate
```

Innentől kezdve az admin oldalon vesszővel elválasztva bármennyi címkét hozzárendelhetünk a postjainkhoz. A tagek egyszerűen megjeleníthetőek az oldalon az alábbi template részlet segítségével:

{% highlight html %}
{% raw %}
{% for tag in story.tags.all %}
    <a href="{% url 'blog-tagged' tag.slug %}">{{ tag.name }}</a>
{% endfor %}
{% endraw %}
{% endhighlight %}

A fenti példában a `story` template object egy darab posztra mutat, a `blog-tagged` URL-ről a szűrésnél lesz szó. Minden `tag`-nek van egy `name` mezője, ami a konkrét megjelenítendő címke neve lesz és egy `slug` mezője, ami ugyanazt a szerepet játsza, mint a saját Story modellünkben. A `story.tags.all` lekérdezi az összes, adott story-hoz tartozó taget és egy listában adja vissza.

### Szűrés

Szűrésnél az utolsó lekérdezést fordítjuk meg: az adott taghez milyen story-k tartoznak? Ehhez már saját view-t és URL patternt kell írnunk. A view:

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

Először egy [`get_object_or_404`](https://docs.djangoproject.com/en/1.6/topics/http/shortcuts/#get-object-or-404) hívással a címke egyedi `slug` attribútuma alapján kiválasztjuk a megfelelő taget, majd az összes `Story` közül kiszűrjük azokat, amelyekben ez megtalálható. Végül az egészet egy erre alkalmas [template](https://docs.djangoproject.com/en/1.6/ref/templates/api/#basics)-tel a kész HTML [válasszá rendereljük](https://docs.djangoproject.com/en/1.6/topics/http/shortcuts/#render-to-response).

> A __context__ is a “variable name” -> “variable value” mapping that is passed to a template.
>
> A template __renders__ a context by replacing the variable “holes” with values from the context and executing all block tags.

A URL pattern:

```python
url(r'^tags/(?P<slug>[-\w]+)/$', 'tagged', name='blog-tagged')
```

A fenti URL pattern három részből áll:

1. Egy reguláris kifejezés, amelyre egy beérkező URL request illeszkedhet. Ebben a `(P<slug>[-\w]+)` rész egy nevesített csoport, amely `-` és alfanumerikus karakterek kombinációjára illeszkedik. A csoport a slug nevet kapja.
2. A view neve (lásd a view kódjában a függvény neve is `tagged`): ezt a view-t kell hívni ha a regexpre van illeszkedő találat.
3. A `name` argumentum pedig egy alias, amelyet a template-ben használtunk (lásd fent a template kódját). További értelmezés [itt](https://docs.djangoproject.com/en/1.6/topics/http/urls/#reverse-resolution-of-urls).

Az alábbi template részlettel felsorolható az összes témába vágó poszt:

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
