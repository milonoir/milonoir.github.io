---
title: Hogyan készült? 3. rész
layout: post
lang: hu
ref: how-its-made3
date: 2014-12-01 19:31:04 +0100
categories: archívum
---

Frissiben találtam megoldást egy problémára, amely pár hete bosszant, ezért erről írok röviden: biztonsági mentés.

Adott ugye a Raspberry Pi, rajta pedig a weboldal beélesítve. Maga a Django kód persze verziókövető rendszerben van, annak nem eshet bántódása. De mi a helyzet az adatbázissal és a statikus tartalmakkal (pl. képek)? Amióta szeptember végén elindítottam az oldalt aggaszt, hogy nincsen backup stratégiám.

Mostanáig! Úgy képzeltem, hogy a nem verziókövetett fájljaimat felsyncelem valamilyen felhő tárhelyre minden nap és akkor nyugodtan alhatok. Ma délután meg is találtam a megoldást [ezen az oldalon](http://alexbelezjaks.com/automatic-backup-to-dropbox-on-raspberry-pi-tutorial/). Csak annyira kötnék bele, hogy az első `wget` lépés nálam nem működött, úgyhogy helyette inkább clone-oztam a repot [innen](https://github.com/andreafabrizi/Dropbox-Uploader), ahogyan az a *Getting started* részben írva vagyon.

Ha jól csináltam, akkor minden reggel 6 órakor egy újabb backup mentésem lesz a Dropboxomban.
