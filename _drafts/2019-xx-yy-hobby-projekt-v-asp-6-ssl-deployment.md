---
published: false
title: Hobby projekt v ASP.NET Core díl šestý - SSL a produkční deployment
---
V dnešní době je dobrým zvykem zabezpečit web pomocí SSL certifikátu. Toho jsem chtěl samozřejmě docílit i v případě tohoto projektu.
SSL certifikát je možné zakoupit u důvěryhodné certifikační autority a nebo je možné zdarma využít [Let's Encrypt](https://letsencrypt.org/). Tuto službu jsem se rozhodl využít a zde si ukážeme jak vypadá můj deployment s podporou SSL certifikátu.

Kestrel samotný sice podporuje SSL certifikáty přímo, ale Microsoft sám v určitých případech doporučuje použít reverse proxy. Jedním z těchto případů kdy je reverse proxy vhodná je právě potřeba SSL. Proxy může pomoci také pro limitování počtu requestů, load balancing atd.

Jako reverse proxy jsem se rozhodl využít nginx. Jednak z důvodu že s ním mám dobré zkušenosti práce, ale také kvůli tomu že jsem našel repozitář [JrCs/docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) který popisuje jak jednoduše nasadit infrastrukturu pro let's encrypt.

Výsledný deployment se skládá z těchto částí:
- nginx reverse proxy
- docker-gen
- let's encrypt proxy companion
- samotná aplikace

Celé to funguje následujícím způsobem:

1. Let's encrypt companion kontejner vygeneruje validní SSL certifikát
2. docker-gen kontejner monitoruje certifikáty a aplikační kontejnery. V případě, že se něco změní, vygeneruje novou konfiguraci pro nginx
3. nginx reverse proxy je oficiální kontejner nginxu konfigurovaný pomocí `docker-gen` kontejneru
4. Aplikační kontejner obsahuje běžící ASP.NET Core aplikaci

![Schéma deploymentu](https://raw.githubusercontent.com/JrCs/docker-letsencrypt-nginx-proxy-companion/master/schema.png)

Existuje také varianta se 3 kontejnery -  nginx a docker-gen jsou spojeny do jednoho kontejneru. Tato konfigurace je blíže [popsána na githubu](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/wiki/Basic-usage). Já preferuji variantu se 4 kontejnery což mi umožňuje použít oficiální nginx image.


## Konfigurace domény
Doménu mám zakoupenou u [Wedosu](wedos.cz). Pro její použití pro aplikaci je nutné nakonfigurovat Wedos i DigitalOcean.

### Wedos
Wedos je nutné nasměrovat na nameservery DigitalOceanu. Já používám konfiguraci pro subdomény a ty se konfigurují následujícím způsobem:

1. Přihlásit do správy domény
2. Přejít do správy DNS záznamů
3. Přidat záznamy pro DigitalOCean name servery. Jedná se o celkem 3 záznamy typu `NS` vedoucí na `ns1.digitalocean.com`, `ns2.digitalocean.com` a `ns3.digitalocean.com`.
Tyto záznamy říkají, že pro jejich překlad se má použít name server DigitalOcean.

### DigitalOcean
Na DigitalOcean je nutné vytvořit doménu a nakonfigurovat na jaký droplet povede.

1. Přihlásit se do administrace DigitalOcean
2. V záložce Networking/Domains přidat novou doménu a spárovat jí s projektem
3. Ve správě domény nastavit hostname a na jaký droplet povede. Místo hostname lze použít `@`, který značí že nastavujeme kořenový záznam.

Pro ověření správnosti konfigurace domény můžeme použít například `ping`. Změna konfigurace nějakou dobu trvá (může to být i hodinu), je možné že bude potřeba počkat, než se DNS záznamy zaktualizují.


Zdroje:
- https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion
- https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-2.2#when-to-use-kestrel-with-a-reverse-proxy
