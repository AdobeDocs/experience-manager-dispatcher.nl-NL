---
title: In cache geplaatste pagina's ongeldig maken van AEM
seo-title: Invalidating Cached Pages From Adobe AEM
description: Leer hoe u de interactie tussen Dispatcher en AEM configureert voor een effectief cachebeheer.
seo-description: Learn how to configure the interaction between Adobe AEM Dispatcher and AEM to ensure effective cache management.
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: 25f8569bdeb6b675038bea02637900e9d0fc1f27
workflow-type: tm+mt
source-wordcount: '1404'
ht-degree: 0%

---

# In cache geplaatste pagina&#39;s ongeldig maken van AEM {#invalidating-cached-pages-from-aem}

Wanneer u Dispatcher gebruikt met AEM, moet de interactie zodanig zijn geconfigureerd dat een effectief cachebeheer wordt gegarandeerd. Afhankelijk van uw milieu, kan de configuratie prestaties ook verhogen.

## AEM gebruikersaccounts instellen {#setting-up-aem-user-accounts}

De standaardwaarde `admin` de gebruikersrekening wordt gebruikt om de replicatieagenten voor authentiek te verklaren die door gebrek geïnstalleerd zijn. U zou een specifieke gebruikersrekening voor gebruik met replicatieagenten moeten tot stand brengen.

Zie voor meer informatie de [Replicatie- en transportgebruikers configureren](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) van de lijst AEM beveiligingscontrole.

## Dispatcher Cache van de ontwerpomgeving ongeldig maken {#invalidating-dispatcher-cache-from-the-authoring-environment}

Een replicatieagent op de AEM auteurinstantie verzendt een verzoek van de geheim voorgeheugenongeldigheid naar Dispatcher wanneer een pagina wordt gepubliceerd. De aanvraag zorgt ervoor dat Dispatcher het bestand uiteindelijk vernieuwt in de cache terwijl nieuwe inhoud wordt gepubliceerd.

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

Gebruik de volgende procedure om een replicatieagent op de AEM auteursinstantie te vormen voor het ongeldig maken van het geheime voorgeheugen van de Verzender op paginaconactivering:

1. Open de console AEM Tools. (`https://localhost:4502/miscadmin#/etc`)
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten op auteur. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik op Bewerken en controleer of op het tabblad Instellingen **Ingeschakeld** is geselecteerd.

1. (optioneel) Selecteer de optie **Alias-update** optie.
1. Op het lusje van het Vervoer, ga URI nodig in om tot Verzender toegang te hebben.\
   Als u de standaardagent van de Vlek van de Dispatcher gebruikt zult u waarschijnlijk hostname en haven moeten bijwerken; bijvoorbeeld https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **Opmerking:** Voor de agenten van de Vlek van de Verzender, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*`. U kunt een URL gebruiken van `/path1/invalidate.cache` om de eerste boerderij te richten en `/path2/invalidate.cache` om de tweede boerderij te richten. Zie voor meer informatie [Dispatcher gebruiken met meerdere domeinen](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Klik op OK om de agent te activeren.

U kunt ook de Dispatcher Flush-agent openen en configureren vanuit de [AEM Touch UI](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/configuring/replication.html#configuring-a-dispatcher-flush-agent).

Zie voor meer informatie over het inschakelen van toegang tot vanity-URL&#39;s [Toegang tot URL&#39;s met Vanity inschakelen](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>De agent voor het flushing dispatchergeheime voorgeheugen moet geen gebruikersnaam en wachtwoord hebben, maar als gevormd zullen zij met basisauthentificatie worden verzonden.

Er zijn twee mogelijke problemen met deze aanpak:

* De Dispatcher moet bereikbaar zijn vanuit de ontwerpinstantie. Als uw netwerk (b.v. de firewall) dusdanig wordt gevormd dat de toegang tussen twee beperkt is, kan dit niet het geval zijn.

* De openbaarmaking en de ongeldigmaking van de cache vinden tegelijkertijd plaats. Afhankelijk van de timing kan een gebruiker een pagina aanvragen vlak nadat deze uit de cache is verwijderd en vlak voordat de nieuwe pagina wordt gepubliceerd. AEM retourneert nu de oude pagina en de Dispatcher slaat deze opnieuw in het cachegeheugen op. Dit is meer een probleem voor grote sites.

## Dispatcher Cache van een publicatie-instantie ongeldig maken {#invalidating-dispatcher-cache-from-a-publishing-instance}

Onder bepaalde omstandigheden kunnen de prestaties worden verbeterd door cachebeheer over te brengen van de ontwerpomgeving naar een publicatie-instantie. Het zal dan het het publiceren milieu (niet het AEM auteursmilieu) zijn die een verzoek van de geheim voorgeheugenongeldigverklaring naar Dispatcher verzendt wanneer een gepubliceerde pagina wordt ontvangen.

Deze omstandigheden omvatten:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Mogelijke timingconflicten tussen Dispatcher en het publicatieexemplaar voorkomen (zie [De Dispatcher-cache ongeldig maken vanuit de ontwerpomgeving](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Het systeem bevat verschillende publicatieinstanties die zich op krachtige servers bevinden, en slechts één ontwerpinstantie.

>[!NOTE]
>
>Het besluit om deze methode te gebruiken moet worden genomen door een ervaren AEM beheerder.

Het leegmaken van de verzender wordt gecontroleerd door een replicatieagent die op de publicatieinstantie werkt. Nochtans, wordt de configuratie gemaakt op het auteursmilieu en dan overgebracht door de agent te activeren:

1. Open de console AEM Tools.
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten bij publiceren. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik op Bewerken en controleer of op het tabblad Instellingen **Ingeschakeld** is geselecteerd.
1. (optioneel) Selecteer de optie **Alias-update** optie.
1. Op het lusje van het Vervoer, ga URI nodig in om tot Verzender toegang te hebben.\
   Als u de standaardagent van de Vlek van de Dispatcher gebruikt zult u waarschijnlijk hostname en haven moeten bijwerken; bijvoorbeeld: `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Opmerking:** Voor de agenten van de Vlek van de Verzender, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*`. U kunt een URL gebruiken van `/path1/invalidate.cache` om de eerste boerderij te richten en `/path2/invalidate.cache` om de tweede boerderij te richten. Zie voor meer informatie [Dispatcher gebruiken met meerdere domeinen](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Herhaal deze bewerking voor elke betrokken publicatie-instantie.

Na het vormen, wanneer u een pagina van auteur activeert om te publiceren, stelt deze agent een standaardreplicatie in werking. Het logboek bevat berichten die verzoeken aangeven die afkomstig zijn van uw publicatieserver, vergelijkbaar met het volgende voorbeeld:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## De Dispatcher-cache handmatig ongeldig maken {#manually-invalidating-the-dispatcher-cache}

Als u de Dispatcher-cache ongeldig wilt maken (of wilt leegmaken) zonder een pagina te activeren, kunt u een HTTP-aanvraag naar de dispatcher verzenden. U kunt bijvoorbeeld een AEM maken waarmee beheerders of andere toepassingen de cache kunnen leegmaken.

Door de HTTP-aanvraag verwijdert Dispatcher specifieke bestanden uit de cache. De Dispatcher vernieuwt desgewenst de cache met een nieuwe kopie.

### In cache opgeslagen bestanden verwijderen {#delete-cached-files}

Geef een HTTP- verzoek uit dat Dispatcher ertoe brengt om dossiers van het geheime voorgeheugen te schrappen. Dispatcher plaatst de bestanden alleen opnieuw in het cachegeheugen als het een clientverzoek voor de pagina ontvangt. Het verwijderen van cachebestanden op deze manier is geschikt voor websites die waarschijnlijk geen gelijktijdige aanvragen voor dezelfde pagina ontvangen.

De HTTP-aanvraag heeft de volgende vorm:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher spoelt (verwijdert) de cachebestanden en mappen met namen die overeenkomen met de waarde van de `CQ-Handler` header. Bijvoorbeeld een `CQ-Handle` van `/content/geomtrixx-outdoors/en` komt overeen met de volgende items:

* Alle bestanden (van een willekeurige bestandsextensie) met de naam `en` in de `geometrixx-outdoors` directory

* Elke map met de naam &quot; `_jcr_content`&quot; onder de map en (die, indien aanwezig, renderingen van subknooppunten van de pagina in de cache bevat)

Alle andere bestanden in de cache van de verzender (of tot een bepaald niveau, afhankelijk van de `/statfileslevel` instellen) ongeldig worden gemaakt door op de knop `.stat` bestand. De laatste wijzigingsdatum van dit bestand wordt vergeleken met de laatste wijzigingsdatum van een document in de cache en het document wordt opnieuw opgehaald als het `.stat` is nieuwer. Zie [Bestanden op mapniveau ongeldig maken](dispatcher-configuration.md#main-pars_title_26) voor meer informatie.

De ongeldigmaking (d.w.z. het aanraken van .stat dossiers) kan worden verhinderd door een extra Kopbal te verzenden `CQ-Action-Scope: ResourceOnly`. Dit kan worden gebruikt om bepaalde middelen te spoelen zonder andere delen van het geheime voorgeheugen ongeldig te maken, zoals gegevens JSON die dynamisch worden gecreeerd en regelmatig het spoelen onafhankelijk van het geheime voorgeheugen vereisen (b.v. die gegevens vertegenwoordigen die van een derdesysteem worden verkregen om nieuws, voorraadtikkers, enz. te tonen).

### Bestanden verwijderen en opnieuw plaatsen {#delete-and-recache-files}

Geef een HTTP-aanvraag uit die ervoor zorgt dat Dispatcher cachebestanden verwijdert en het bestand direct ophaalt en opnieuw plaatst. U kunt bestanden verwijderen en onmiddellijk opnieuw in cache plaatsen wanneer websites waarschijnlijk gelijktijdige clientverzoeken voor dezelfde pagina ontvangen. Onmiddellijk het recaching zorgt ervoor dat de Dispatcher de pagina terugwint en in het voorgeheugen onderbrengt slechts één keer, in plaats van één voor elk van de gelijktijdige cliëntverzoeken.

**Opmerking:** Het verwijderen en in de cache plaatsen van bestanden mag alleen op het publicatieexemplaar worden uitgevoerd. Wanneer uitgevoerd vanaf de auteurinstantie, komen de rasvoorwaarden voor wanneer pogingen om middelen terug te winnen voorkomen alvorens zij zijn gepubliceerd.

De HTTP-aanvraag heeft de volgende vorm:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

De paginaden die direct opnieuw worden getekend, worden weergegeven op afzonderlijke regels in de berichttekst. De waarde van `CQ-Handle` Dit is het pad van een pagina die de pagina&#39;s ongeldig maakt om opnieuw te tekenen. (Zie de `/statfileslevel` parameter van de [Cache](dispatcher-configuration.md#main-pars_146_44_0010) configuratie-item.) In het volgende voorbeeld wordt het HTTP-aanvraagbericht verwijderd en wordt de `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Voorbeeld spoelservlet {#example-flush-servlet}

De volgende code implementeert een servlet die een ongeldige aanvraag naar Dispatcher verzendt. servlet ontvangt een verzoekbericht dat bevat `handle` en `page` parameters. Deze parameters geven de waarde van de `CQ-Handle` en het pad van de pagina naar de rechthoek. De servlet gebruikt de waarden om de HTTP- aanvraag voor Dispatcher samen te stellen.

Wanneer servlet aan de publicatieinstantie wordt opgesteld, veroorzaakt volgende URL Dispatcher om de /content/geometrixx-outdoors/en.html pagina te schrappen en dan een nieuw exemplaar in het voorgeheugen onder te brengen.

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>Dit voorbeeldservlet is niet veilig en toont slechts het gebruik van het HTTP Post- verzoekbericht aan. Uw oplossing zou toegang tot servlet moeten beveiligen.

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```
