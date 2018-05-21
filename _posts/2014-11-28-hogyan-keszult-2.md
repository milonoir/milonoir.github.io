---
title: Hogyan készült? 2. rész
layout: post
lang: hu
ref: how-its-made2
date: 2014-11-28 11:07:51 +0100
categories: archívum
---

Sokat gondolkoztam az [első rész]({% post_url 2014-10-03-hogyan-keszult-1 %}) óta a folytatás mikéntjén. Temérdek információ fellelhető tutorial és könyv formában arról, hogy hogyan kell egy *nagyon alap* Django blogot megcsinálni, úgyhogy én nem szeretnék beállni ebbe a sorba. Írásaimban feltételezem, hogy az olvasó már egy ilyen kalauzon végigment, így az alapokkal nem foglalkozom, sokkal inkább olyan feature leírásokat szeretnék közölni, amelyekkel egy blog továbbléphet erről a szintről.

Egy tipikus *nagyon alap* blog `models.py` állománya nagyjából így szokott kinézni:

```python
from django.db import models

class Story(models.Model):
    title = models.CharField(max_length=100)
    slug = models.SlugField()
    content = models.TextField(blank=True)
    created = models.DateTimeField(default=datetime.datetime.now)
```

Mint látható egy ez nem egy túlkomplikált dolog. Amire szükség van az egy cím, egy [slug](http://en.wikipedia.org/wiki/Semantic_URL#Slug) mező permalink generáláshoz, maga a bejegyzés tartalma és egy időbélyeg.

A `created` mezőt az admin felület automatikusan kitölti majd nekünk amikor új bejegyzést kreálunk. A `slug`-ot a `title`-ből generáltatjuk - ez a beállítás `admin.py`-ból látszik:

```python
from django.contrib import admin
from models import Story

class StoryAdmin(admin.ModelAdmin):
    ...
    prepopulated_fields = {'slug': ('title')}

admin.site.register(Story, StoryAdmin)
```

Ami a tartalmat illeti, a történet itt már érdekesebbé válik. Ebben a formában a `content` csak egyszerű szöveget fog visszaadni, ami azért valljuk be, elég kopár. Mondjuk azt, hogy szeretnénk látni ilyesmiket is: címsorokat, félkövér és dőlt betűket, képeket, beágyazott videókat és programkódokat lehetőleg szintaxis kiemeléssel és a sorok megszámozásával. Hívjuk ezt most egyszerűen *formázott tartalom*nak.

### Figyelem! Most érkeztünk el oda, amit a mai poszt mondanivalójának szántam. ###

Jelen esetben egy WYSIWYG tartalomszerkesztő nem áll rendelkezésre, ígyhát ahhoz, hogy formázott tartalmat kapjunk a szöveget tele kell pakolnunk HTML tagekkel és CSS kódokkal. Ettől egyel jobb, ha csak HTML tageljük a szöveget és a CSS-t valahonnan source-oljuk. A legjobb azonban, ha egy text-to-HTML fordítót használunk, pl. a  [Markdown](https://pypi.python.org/pypi/Markdown)t. A Markdown szintaxisa egyszerűen elsajátítható, hasonló a Wikiéhez, de attól sokkal átgondoltabb és felhasználóbarátabb (szerintem).

Felmerül a kérdés, hogy mikor történik meg a text-HTML konverzió? Erre két megoldás kínálkozik:

* A `content` mező tartalmazza a mindenkori Markdown formátumú szöveget, amiből minden lekéréskor generáltatjuk a HTML kódot; vagy
* Fenntartunk egy másik mezőt a HTML kódnak is, amit minden szerkesztés után frissítünk.

Az első megoldással az a probléma, hogy számítási feladattal terheljük a szerverünket, a másodikkal pedig az, hogy duplikálva tároljuk a kontentot, tehát az adatbázisunk nagyobbra hízik. Az R-Pi esetében tegyük fel, hogy az utóbbi a kisebbik rossz. Ezek után a `models.py` úgy módosul, hogy felveszünk egy újabb *nem szerkeszthető* content mezőt (az eredeti `content` nevét is megváltoztatjuk a readability szellemében)...:

```python
...
class Story(models.Model):
    ...
    markdown_content = models.TextField(blank=True)
    html_content = models.TextField(editable=False)
    ...
```

... majd ezt úgy töltetjük ki az admin felülettel, hogy a szerkesztés végeztével (mentéskor) ráhívjuk a Markdownt a `markdown_content` tartalmára - szintén a `models.py`-ban (természetesen a Markdownt nem felejtjük el importálni). Ehhez a `Model` class `save` metódusát írjuk felül:

```python
from markdown import markdown
...
class Story(models.Model):
    ...
    def save(self, *args):  # yes, we override it
        self.html_content = markdown(text=self.markdown_content, extensions=['codehilite(linenums = True)'])
        super(Story, self).save()  # don’t forget to call super()
```

A `codehilite(linenums = True)` rész lesz felelős a kontentben megjelenő programkódok szintaxisának színezéséért és sorszámozásáért. Ahhoz, hogy ez működjön két dolgot kell megtennünk:

1. Telepítsük a [Pygments](https://pypi.python.org/pypi/Pygments) csomagot.
2. Készítsünk vagy [szerezzünk be](https://github.com/richleland/pygments-css) egy CSS fájlt a codehilite számára, amit linkelünk a template-ünkben. Pl.: `<link rel="stylesheet" type="text/css" href="./codehilite.css">`

Már csak egy dolog van hátra: default felállásban a Django semmilyen változóból származó HTML kódot nem hajlandó értelmezni biztonsági okokból, ezért külön szólnunk kell neki, hogy mi megbízunk a `html_content` tartalmában és legyen szíves azt HTML kódnak látni. Ezt valamelyik vonatkozó template fájlban tehetjük meg a `safe` flag segítségével. Például:

```html
<p id="story-body">
    {{ story.html_content|safe }}
</p>
```

Ezzel kész is vagyunk, most már mindenféle formázási lehetőség birtokában vagyunk, amit a Markdown támogat.

> *Az `urls.py` és `views.py`, valamint a template-ek tartalma a fenti feladat megoldása szempontjából irreleváns, bármelyik tutorialban választ kaphatunk rá. Éppen ezért itt nem is szerepeltetem.*
