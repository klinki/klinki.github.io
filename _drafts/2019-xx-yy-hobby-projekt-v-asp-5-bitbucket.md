---
published: false
title: Hobby projekt v ASP.NET Core díl pátý - bitbucket pipelines
---

V dnešním článku se podíváme na to, jak zautomatizovat build a nasazení naší ASP.NET Core aplikace.

Pro hostování svých privátních repozitářů používám už pěkných pár let [Bitbucket](https://bitbucket.org). Nabízí neomezené množství privátních repozitářů zdarma a velice dobře se mi s ním pracuje. Do nedávna jsem ho využíval pouze na hostování git repozitářů, poslední dobou ale začínám používat i další funkcionalitu kterou nabízí, například [Pipelines](https://bitbucket.org/product/features/pipelines). Bitbucket Pipelines je nástroj pro velmi jednoduchou konfiguraci CI/CD. Jedná se o alternativu k provozování separátního build serveru, jako Jenkins apod.

Pipelines se konfigurují pomocí souboru `bitbucket-pipelines.yml`, který je uložený v kořenovém adresáři repozitáře. Konfigurace je velice jednoduchá a vesměs deklarativní. Nadefinujeme, jaké programy potřebujeme (například docker) a popíšeme jednotlivé kroky.

Celá konfigurace může vypadat například takto jednoduše:

```yaml
options:
  docker: true

pipelines:
  default:
    - step:
        script:
          - TAG=$(git tag --contains $BITBUCKET_COMMIT)
          - SHORT_COMMIT=$(git rev-parse --short $BITBUCKET_COMMIT)
          - VERSION=${TAG:-$SHORT_COMMIT}
          - docker login -u $DOCKER_USERNAME -p $DOCKER_PASS registry.gitlab.com
          - docker build -t registry.gitlab.com/klinki/docker-registry/inviser:latest .
          - docker build -t registry.gitlab.com/klinki/docker-registry/inviser:${VERSION} .
          - docker push registry.gitlab.com/klinki/docker-registry/inviser:latest
          - docker push registry.gitlab.com/klinki/docker-registry/inviser:${VERSION}
    - step:
        name: Deploy
        script:
            - cat ./deploy.sh | ssh $DEPLOYMENT_USER@$DEPLOYMENT_SERVER
            - echo "Deploy finished"
```
Celý skript je velmi přímočarý, první 3 řádky konfigurace slouží k určení výsledné verze docker image.
Nejprve se podívám, jestli neexistuje pro daný commit také tag. Pokud ano, použije se tag jako název verze. Pokud ne, použije se zkrácený hash commitu.

Pak následuje login do privátních Docker registrů, které mám hostované na Gitlabu. Každý image builduji se 2 tagy - jednou jako `latest` a jednou s konkrétním číslem verze. To proto, abych se mohl případně snadno vrátit k předchozí verzi, kdyby nastaly nějaké problémy s novou. Zbuildovaný image pushuji do registů a tím končí první fáze pipeline.

Všimněte si proměnné `$BITBUCKET_COMMIT`. Tuto proměnnou Bitbucket automaticky nastaví na hodnotu aktuálního commitu při každém buildu. Proměnných je možné použít celou řadu, [zde je seznam všech které Bitbucket defaultně nastavuje](https://confluence.atlassian.com/bitbucket/variables-in-pipelines-794502608.html). Je možné nadefinovat si vlastní proměnné pro repozitář a pro deployment. Uživatelské proměnné je možné také zašifrovat. Hodnota zašifrovaných proměnných nebude viditelná ani v build logu. (Šifrovaná je například proměnná `$DOCKER_PASS`). Pro pipeline je možné také velice jednoduše vygenerovat SSH klíče, které můžete použít pro přihlášení na server s aplikací.


V druhé fázi je deployment. Ten spočívá v přihlášení na deployment server a spouštení `deploy.sh` scriptu. Ten je také uložený v repozitáři, čímž je velice snadno udržovatelný.
Deploy script nedělá nic složitého - pouze stáhne z dockeru novou verze image a vytvoří kontejner s novou verzí. Jistě by se tu dalo vymyslet i něco zajímavějšího - třeba separátně běžící stará i nová verze routovaná pomocí nějaké reverse proxy. To je ale pro můj případ v tuto chvíli zbytečně složité.


Snadnost konfigurace Bitbucket Pipelines na mě velice zapůsobila. Musím říct že se jedná o jeden z nejsnáze konfigurovatelných CI/CD serverů, se kterými jsem kdy pracoval. V tarifu zdarma je k dispozici 500 minut měsíčně. Pro hobby projekty je to rozhodně více než dost, troufám si říct že by to vystačilo i pro menší komerční projekty. Samotná pipeline lze spustit několika způsoby, například po pushi na server, ručně či automaticky v daných intervalech.
