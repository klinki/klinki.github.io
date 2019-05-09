---
published: false
title: Hobby projekt v ASP.NET Core díl čtvrtý - nasazení
---

Jelikož mám rád docker a cloudové technologie, hledal jsem možnost kam aplikaci nasadit primárně mezi cloudovými providery. Zaujal mě [DigitalOcean](https://www.digitalocean.com/) a jelikož mi ho ještě vychvaloval kamarád, rozhodl jsem se, že
ho vyzkouším. Našel jsem promo kód se 100$ a 2 měsíční platností, takže jsem měl nějaký ten čas na experimentování.

Vytvořil jsem si nový předpřipravený docker droplet. Na zkoušku jsem zvolil nejlevnější nabízený, za 5$ / měs. Hardwarové parametry jsou celkem slušné: 1GB RAM, 1 vCPU (bohužel nevím výkon), 25 GB SSD disk a 1 TB transfer.
Na hobby projekt více než postačující.

Na vytvořený server je možné připojit se pomocí webové konzole v prohlížeči, nebo pomocí ssh. Mám rád ssh, takže jsem zvolil to. Navíc webová konzole mi ani moc spolehlivě nefungovala, prohlížeč (Chrome) z ní byl nějaký zmatený.

Volám `ssh root@ip-adresa-serveru` a zjišťuji, že si nepamatuji heslo :-) Naštěstí ho DigitalOcean umožňuje snadno resetovat, takže volím v detailu dropletu položku *Access/Reset root password* a vzápětí mi dorazí nové heslo mailem. Fajn, ssh funguje. Zkouším základní příkazy jako `docker --version` a `docker-compose --version`, abych ověřil že je docker skutečně korektně nainstalovaný. Je nainstalovaný a funguje. Super :)

`Docker version 18.06.1-ce, build e68fc7a`
`docker-compose version 1.22.0, build f46880fe`

Tak a teď otázka za 3 bludišťáky, jak aplikaci dostat do dropletu? Musím vytvořit docker image a ten nějak dostat ze svého notebooku na DigitalOcean. Bylo mi jasné že nejlepším řešením jsou docker registry a tak jsem chvíli googlil, zda je možné někde hostovat privátní image. Naštěstí ano, záchrana přichází z Gitlabu, mého oblíbeného nástroje pro správu git repozitářů. Stačí založit nový repozitář a zapnout v konfiguraci hostování docker registrů - více informací [v dokumentaci](https://docs.gitlab.com/ee/user/project/container_registry.html). V době psaní článku byl limit 10GB, což je na hobby projekt opět více než postačující.

Pro přihlášení do registrů je potřeba použít příkaz `docker login registry.gitlab.com`. Zde je možné použít username a password do Gitlabu a nebo vygenerovat public token. Gitlab má také podporu pro [deploy tokeny](https://gitlab.com/help/user/project/deploy_tokens/index#read-container-registry-images), které je možné využít ke čtení registrů a stažení image.

Po přihlášení do registrů je potřeba zbuildit kontejner pomocí příkazu `docker build -t registry.gitlab.com/klinki/docker-registry .` (klinki/docker-registry je název repozitáře na Gitlabu a zároveň název registrů).
Docker chvíli pracuje a hotovo. Build je připraven na odeslání. Spouštím tedy příkaz `docker push registry.gitlab.com/klinki/docker-registry` a sestavený docker image putuje z mého notebooku do Gitlab registrů.
Po chvíli ho vidím v administraci.

Nyní příjde zkouška ohněm - půjde stáhnout z DigitalOceanu? Zkouším. Přihlašuji se k registrům pomocí deploy klíče (opět `docker login registry.gitlab.com`). Na zkoušku pouštím `docker-compose up`. Funguje to! Stahuje!

Práce s registry byla nakonec jednodušší než jsem očekával. Jsem rád že jsem se opět naučil pár nových věcí - práci s DigitalOcean a s docker registry na Gitlabu. Příště se podíváme na to, jak zautomatizovat deployment.
