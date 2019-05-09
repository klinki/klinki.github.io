---
published: false
title: Hobby projekt v ASP.NET Core
---

Poslední dobou doma po večerech pracuji na drobném hobby projektu pro správu investičního portfolia. Mým hlavním cílem je vykreslit si pár pěkných grafů a tabulek s přehledem jak si která investice vede. Původně jsem si myslel, že se zaměřím primárně na vývoj frontendu (ten mimochodem píšu ve frameworku Aurelia) a backend bude nějaké co nejjednodušší CRUD řešení.

Bylo mi celkem jedno v čem backend bude, tak jsem jako první šáhl po PHP - už jen z důvodu dostupnosti a ceny hostingu. Chtěl jsem se také podívat co se v PHP světě odehrálo za dobu co jsem ho nepoužíval. Nejprve jsem trošku experimentoval s projektem [Api Platform](https://api-platform.com/) - líbila se mi myšlenka co možná nejvíce kódu vygenerovat. Nicméně poté co jsem propálil spousty hodin a bezesných večerů kvůli různým nesmyslným problémům s autentikací apod. jsem se rozhodl že tudy cesta nevede.

Chvilku jsem zkoušel udělat nějaké řešení postavené na Nette s použitím API Frameworku [Apitte](https://github.com/apitte/core). Celkem se mi líbilo, jaký pokrok PHP udělalo a jak se postupně propracovalo k middlewarové architektuře. Nicméně po nějaké době jsem zjistil, že už nejsem schopen psát PHP kód. Chyběly mi datové typy, generiky, jednoduché - pokud možno automatické mapování entit apod. Také neustále něco nefungovalo, musel jsem přemazávat cache, občas se objevovaly různé náhodné chyby které při refreshi stránky zmizely. A co teprve šílený FTP deployment nahrávající tisíce souborů a trvající celou věčnost...

Zkrátka a dobře jsem si jednoduše experimentálně ověřil, že pro psaní REST API není PHP zrovna nejvhodnější volba. (Tedy alespoň ne pro mě).

Přemýšlel jsem po čem tedy sáhnout. Jistě, z hlediska pohodlnosti se nabýzela Java. Spring framework je celkem fajn, používám jí v práci a mám jí celkem v malíku. Navíc jsem se nemusel omezovat jen na Spring - mohl jsem si pohrát s Microprofile, VertX a dalšími zajímavými frameworky postavenými na Javě.

Nicméně již dlouhou dobu jsem velkým fanouškem C# a velice obdivuju čeho Microsoft dosáhl u ASP.NET Core. Výkon Kestrelu je neuvěřitelný a neustále se zlepšuje. Velké díky patří také Open Source komunitě, která přispívá k jeho vývoji.

V následující sérii článku bych se s vámi rád podělil o to, jak mé první setkání s ASP.NET Core probíhalo.


## Zajímavé odkazy
Zde je pár zajimavých odkazů o výkonu ASP.NET Core.

- [ASP.NET Core – 2300% More Requests Served Per Second](https://www.ageofascent.com/2016/02/18/asp-net-core-exeeds-1-15-million-requests-12-6-gbps/)

- [ASP.NET Core: Saturating 10GbE at 7+ million request/s](https://www.ageofascent.com/2019/02/04/asp-net-core-saturating-10gbe-at-7-million-requests-per-second/)
