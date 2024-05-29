---
title: In cache geplaatste pagina's ongeldig maken van AEM
description: Leer hoe u de interactie tussen Dispatcher en AEM configureert voor een effectief cachebeheer.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '1407'
ht-degree: 0%

---

# In cache geplaatste pagina&#39;s ongeldig maken van AEM {#invalidating-cached-pages-from-aem}

Wanneer u Dispatcher gebruikt met AEM, moet de interactie zodanig zijn geconfigureerd dat een effectief cachebeheer wordt gegarandeerd. Afhankelijk van uw milieu, kan de configuratie prestaties ook verhogen.

## AEM gebruikersaccounts instellen {#setting-up-aem-user-accounts}

De standaardwaarde `admin` de gebruikersrekening wordt gebruikt om de replicatieagenten voor authentiek te verklaren die door gebrek geïnstalleerd zijn. Creeer een specifieke gebruikersrekening voor gebruik met replicatieagenten.

Zie de klasse [Replicatie- en transportgebruikers configureren](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#VerificationSteps) van de lijst AEM beveiligingscontrole.

<!-- OLD URL from above https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps -->

## Dispatcher Cache van de ontwerpomgeving ongeldig maken {#invalidating-dispatcher-cache-from-the-authoring-environment}

Een replicatieagent op de AEM auteurinstantie verzendt een verzoek van de geheim voorgeheugenongeldigheid naar Dispatcher wanneer een pagina wordt gepubliceerd. Dispatcher vernieuwt het bestand uiteindelijk in de cache wanneer nieuwe inhoud wordt gepubliceerd.

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

Gebruik de volgende procedure om een replicatieagent op de AEM auteursinstantie te vormen. De configuratie maakt de Dispatcher-cache ongeldig na activering van de pagina:

1. Open de console AEM Tools. (`https://localhost:4502/miscadmin#/etc`)
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten op auteur. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik op Bewerken en controleer of op het tabblad Instellingen **Ingeschakeld** is geselecteerd.

1. (optioneel) Selecteer de optie **Alias-update** -optie.
1. Voor het lusje van het Vervoer, ga tot Dispatcher door URI in te gaan.

   Als u de standaard Dispatcher Flush-agent gebruikt, werkt u de hostnaam en poort bij, bijvoorbeeld https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **Opmerking:** Voor de agenten van de Vlek van de Verzender, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*`. U kunt een URL gebruiken van `/path1/invalidate.cache` om de eerste boerderij te richten en `/path2/invalidate.cache` om de tweede boerderij te richten. Zie voor meer informatie [Dispatcher gebruiken met meerdere domeinen](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Klik op OK om de agent te activeren.

U kunt ook de Dispatcher Flush-agent openen en configureren vanuit de [AEM Touch UI](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/configuring/replication#configuring-a-dispatcher-flush-agent).

Voor meer informatie over hoe te om toegang tot ijdelheid URLs toe te laten, zie [Toegang tot URL&#39;s met Vanity inschakelen](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>De agent voor het leegmaken van het geheime voorgeheugen van de Verzender vereist geen gebruikersnaam en wachtwoord, maar als gevormd worden zij verzonden met basisauthentificatie.

Er zijn twee mogelijke problemen met deze aanpak:

* De Dispatcher moet bereikbaar zijn vanuit de ontwerpinstantie. Als uw netwerk (bijvoorbeeld, de firewall) dusdanig wordt gevormd dat de toegang tussen twee beperkt is, kan deze situatie niet het geval zijn.

* De openbaarmaking en de ongeldigmaking van de cache vinden tegelijkertijd plaats. Afhankelijk van de timing kan een gebruiker een pagina aanvragen vlak nadat deze uit de cache is verwijderd en vlak voordat de nieuwe pagina wordt gepubliceerd. AEM retourneert nu de oude pagina en de Dispatcher slaat deze opnieuw in het cachegeheugen op. Deze situatie is eerder een probleem voor grote sites.

## Dispatcher Cache van een publicatie-instantie ongeldig maken {#invalidating-dispatcher-cache-from-a-publishing-instance}

Onder bepaalde omstandigheden kunnen de prestaties worden verbeterd door cachebeheer over te brengen van de ontwerpomgeving naar een publicatie-instantie. Het is dan het het publiceren milieu (niet het AEM auteursmilieu) dat een verzoek van de geheim voorgeheugenongeldigverklaring naar Dispatcher verzendt wanneer een gepubliceerde pagina wordt ontvangen.

Deze omstandigheden omvatten:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Mogelijke timingconflicten tussen AEM Dispatcher en het publicatieexemplaar voorkomen (zie [De Dispatcher-cache ongeldig maken vanuit de ontwerpomgeving](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Het systeem bevat verschillende publicatieinstanties die zich op servers met hoge prestaties bevinden, en slechts één ontwerpinstantie.

>[!NOTE]
>
>Een ervaren AEM beheerder zou het besluit moeten nemen om deze methode te gebruiken.

Een replicatieagent die op publiceert instantie werkt controleert de Dispatcher flush. Nochtans, wordt de configuratie gemaakt in het auteursmilieu en dan overgebracht door de agent te activeren:

1. Open de console AEM Tools.
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten bij publiceren. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik op Bewerken en controleer of op het tabblad Instellingen **Ingeschakeld** is geselecteerd.
1. (optioneel) Selecteer de optie **Alias-update** -optie.
1. Voor het lusje van het Vervoer, ga tot Dispatcher door nodig URI in te gaan.\
   Als u de standaard Dispatcher Flush agent gebruikt, werk hostname en haven bij; bijvoorbeeld `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Opmerking:** Voor de agenten van de Vlek van de Verzender, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*`. U kunt een URL gebruiken van `/path1/invalidate.cache` om de eerste boerderij te richten en `/path2/invalidate.cache` om de tweede boerderij te richten. Zie voor meer informatie [Dispatcher gebruiken met meerdere domeinen](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Meld u aan bij de publicatieinstantie en valideer de configuratie van de spoelagent. Zorg er ook voor dat deze functie is ingeschakeld.
1. Herhaal deze bewerking voor elke betrokken publicatie-instantie.

Na het vormen, wanneer u een pagina van auteur activeert om te publiceren, stelt deze agent een standaardreplicatie in werking. Het logboek bevat berichten die verzoeken aangeven die afkomstig zijn van uw publicatieserver, vergelijkbaar met het volgende voorbeeld:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## De Dispatcher-cache handmatig ongeldig maken {#manually-invalidating-the-dispatcher-cache}

Als u de Dispatcher-cache ongeldig wilt maken (of wilt leegmaken) zonder een pagina te activeren, kunt u een HTTP-aanvraag naar de Dispatcher verzenden. U kunt bijvoorbeeld een AEM maken waarmee beheerders of andere toepassingen de cache kunnen leegmaken.

De HTTP-aanvraag zorgt ervoor dat de Dispatcher specifieke bestanden uit de cache verwijdert. De Dispatcher vernieuwt desgewenst de cache met een nieuwe kopie.

### In cache opgeslagen bestanden verwijderen {#delete-cached-files}

Geef een HTTP- verzoek uit dat de Dispatcher ertoe brengt om dossiers van het geheime voorgeheugen te schrappen. Dispatcher plaatst de bestanden alleen opnieuw in het cachegeheugen als het een clientverzoek voor de pagina ontvangt. Het verwijderen van cachebestanden op deze manier is geschikt voor websites die waarschijnlijk geen gelijktijdige aanvragen voor dezelfde pagina ontvangen.

De HTTP-aanvraag heeft de volgende vorm:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher spoelt (verwijdert) de cachebestanden en mappen met namen die overeenkomen met de waarde van de `CQ-Handler` header. Bijvoorbeeld een `CQ-Handle` van `/content/geomtrixx-outdoors/en` komt overeen met de volgende items:

* Alle bestanden (van een willekeurige bestandsextensie) met de naam `en` in de `geometrixx-outdoors` directory

* Elke benoemde map `_jcr_content` onder de `en` map (die, indien aanwezig, in cache opgeslagen renderingen van subknooppunten van de pagina bevat)

Alle andere bestanden in de Dispatcher-cache (of tot een bepaald niveau, afhankelijk van de `/statfileslevel` instellen) ongeldig worden gemaakt door het `.stat` bestand. De laatste wijzigingsdatum van dit bestand wordt vergeleken met de laatste wijzigingsdatum van een document in de cache en het document wordt opnieuw opgehaald als het `.stat` is nieuwer. Zie [Bestanden op mapniveau ongeldig maken](dispatcher-configuration.md#main-pars_title_26) voor meer informatie.

De ongeldigmaking (namelijk het aanraken van .stat dossiers) kan worden verhinderd door een extra Kopbal te verzenden `CQ-Action-Scope: ResourceOnly`. Deze functionaliteit kan worden gebruikt om bepaalde bronnen te leegmaken. Alles zonder andere delen van de cache ongeldig te maken, zoals JSON-gegevens. Deze gegevens worden dynamisch gemaakt en moeten regelmatig worden leeggemaakt, onafhankelijk van de cache. Bijvoorbeeld, die gegevens vertegenwoordigen die van een derdesysteem worden verkregen om nieuws, voorraadtikkers, etc. te tonen.

### Bestanden verwijderen en opnieuw plaatsen {#delete-and-recache-files}

Geef een HTTP-aanvraag uit die ervoor zorgt dat de Dispatcher in het cachebestand opgeslagen bestanden verwijdert en het bestand direct ophaalt en opnieuw plaatst. U kunt bestanden verwijderen en onmiddellijk opnieuw in cache plaatsen wanneer websites waarschijnlijk gelijktijdige clientverzoeken voor dezelfde pagina ontvangen. Onmiddellijk het recaching zorgt ervoor dat de Dispatcher de pagina terugwint en in het voorgeheugen onderbrengt slechts één keer, in plaats van één voor elk van de gelijktijdige cliëntverzoeken.

**Opmerking:** Het verwijderen en in de cache plaatsen van bestanden mag alleen op de publicatie-instantie worden uitgevoerd. Wanneer uitgevoerd vanaf de auteurinstantie, komen de rasvoorwaarden voor wanneer pogingen om middelen terug te winnen voorkomen alvorens zij zijn gepubliceerd.

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

De paginapaden die onmiddellijk opnieuw in cache moeten worden geplaatst, worden weergegeven op afzonderlijke regels in de berichttekst. De waarde van `CQ-Handle` Dit is het pad van een pagina die de pagina&#39;s ongeldig maakt om opnieuw te tekenen. (Zie de `/statfileslevel` parameter van de [Cache](dispatcher-configuration.md#main-pars_146_44_0010) configuratie-item.) In het volgende voorbeeld wordt het HTTP-aanvraagbericht verwijderd en wordt de `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Voorbeeld spoelservlet {#example-flush-servlet}

De volgende code implementeert een servlet die een validatieverzoek naar Dispatcher verzendt. servlet ontvangt een verzoekbericht dat bevat `handle` en `page` parameters. Deze parameters geven de waarde van de `CQ-Handle` en het pad van de pagina naar de rechthoek. De servlet gebruikt de waarden om de HTTP- aanvraag voor Dispatcher samen te stellen.

Wanneer servlet aan de publicatieinstantie wordt opgesteld, veroorzaakt volgende URL de Dispatcher om de /content/geometrixx-outdoors/en.html pagina te schrappen en dan een nieuw exemplaar in het voorgeheugen onder te brengen.

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>Dit voorbeeldservlet is niet veilig en toont slechts het gebruik van het HTTP Post- verzoekbericht aan. Uw oplossing zou toegang tot servlet moeten beveiligen.
>

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
