---
title: Hogyan készült? 1. rész
layout: post
lang: hu
ref: how-its-made1
date: 2014-10-03 16:20:19 +0100
categories: archive
---

Ahogy arról már [korábban megemlékeztem]({% post_url 2014-09-30-hello-vilag %}), nagyjából 2 évvel ezelőtt jött az ihlet, hogy én bizony itthon beüzemelek egy *nagyon kicsi* PC-t és azon lakik majd az én weblapom. Mielőtt ezt kitaláltam volna, mindenféle ingyenes és olcsó hosting szolgáltatásokat böngésztem. Teljesen fogalom nélkül voltam, de azt tudtam, hogy nem akarok sokat rákölteni és a lehető legkevesebb hirdetést szeretném látni az oldalamon. Ekkoriban hallottam először a *bankkártya méretű*, alacsony fogyasztású számítógépekről. Rövidesen kiderült, hogy mindent meg tudok valósítani saját magam.


#### Hardver

Választásom az akkoriban nagyon népszerű [Raspberry Pi](http://www.raspberrypi.org/)-re esett, azon belül is a combosabb B variánsra. A gépen egy 700 MHz-es ARM proci dolgozik, 512 MB RAM-mal megtámogatva. Található rajta 2 db USB 2.0 port, egy Ethernet csati, egy kompozit video kimenet, egy 3.5-es jack audio out, egy SDHC memóriakártya-foglalat és egy micro USB bemenet a delejnek. Itt jegyezném meg, hogy egy közönséges mobiltelefon töltőről üzemeltethető, amely képes 700 mA-t produkálni 5 V-on. Ha ezekkel az adatokkal számolunk, akkor folyamatos üzem mellett ez *évi* kb. 30 kWh fogyasztást jelent. A gépet kitben vásároltam az eBay-ről, ezért átlátszó műanyag tokozást is kaptam hozzá. Mivel Kínából jött, szép piros a NYÁK színe. Ezen kívül vettem még egy Class 10-es SanDisk SDHC kártyát 16 GB tárhellyel (azt hiszem szintén eBay-ről) és egy noname micro USB töltőt, ami a Pi-t táplálja. Az egész konstrukciót így megúsztam 16-17 ezer JMF-ből.

![Raspberry Pi](/assets/img/img_raspberrypi.jpg)


#### Op. rendszer

Az üres SD kártyára fel kellett varázsolnom valamilyen operációs rendszert. Ma már rendelhető Málnáéktól előre installált kártya is, de azért ez nem agysebészet. Végigkövettem a [leírást](http://www.raspberrypi.org/help/noobs-setup/), majd a kártyáról bootoltam be a Pi-t. Első indításkor összedugtam a TV-mmel, csatlakoztattam bele billentyűzetet és egeret. A NOOBS (New Out Of the Box Software) felkínál néhány lehetőséget, csak rá kell bökni melyik oprendszert szeretnénk telepíteni. Mivel Ubuntu-féle Linuxokkal érzem magam a legjobban, a nagyon ajánlott Raspbian OS-t installáltam, ami végülis egy ARM-ra optimalizált Debian. Miután a Raspbian működőképessé vált felraktam rá egy SSH-t (meg VNC-t is, bár azt nem szoktam használni) és osztottam neki statikus IP-t (mert ugye közben rácsatlakoztattam az otthoni routerünkre). Innentől a TV-re, billentyűzetre és egérre többé már nem volt szükség.


#### Szoftver

Megszámlálhatatlanul sok fórumot és blogot áttúrtam, hogy megtaláljam az optimális software stacket. Két kiindulási pontom volt: Raspberry Pi és Django (Python 2.7 alapon). Ezek mellé kellett találnom gyors és lightweight adatbázist és webszervert. SQL? NoSQL? A Django teszt célokra és kisebb projektekhez az SQLite3-at ajánlja. Az igazság az, hogy egy blog kiszolgálásához bőven elég ez is. MySQL-t vagy Postgret telepíteni emiatt teljesen felesleges. Ha később mégsem jönne be, akkor MongoDB-vel fogok próbálkozni.

Ha webszerver, akkor a legtöbb embernek az Apache ugrik be legelőször. Kicsit talán ágyúval verébre, de [nem lehetetlen](http://www.raspberrypi.org/documentation/remote-access/web-server/apache.md). Én azért ettől lájtosabbat szerettem volna. [Elolvastam](http://tutos.readthedocs.org/en/latest/source/ndg.html) [néhány](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/) [leírást](http://www.apreche.net/complete-single-server-django-stack-tutorial/), így lett aztán Nginx-Gunicorn kombó. Ebben a felállásban az Nginx HTTP proxyként a beérkező requesteket továbbítja egy statikus könyvtár vagy valamilyen applikáció (Gunicorn) felé. A Gunicorn egy socketen keresztül összeköti az Nginx-et a blogmotorral és ezen keresztül szolgálja ki a dinamikus tartalmakat.

A blogmotor maga pedig egy Django applikáció egy virtualenv-ben. A virtuális környezet különválasztja a blogot a rendszer Pythontól; alapja egy 2.7-es Python és egy Django 1.6-os installáció, melyekhez minden szükséges egyéb library is telepítve van.

Nagy vonalakban így áll össze az egész rendszer.
