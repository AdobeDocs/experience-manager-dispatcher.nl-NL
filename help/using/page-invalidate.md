---
title: In cache geplaatste pagina's ongeldig maken vanuit AEM
description: Leer hoe u de interactie tussen Dispatcher en AEM configureert voor een effectief cachebeheer.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '1407'
ht-degree: 0%

---

# Pagina&#39;s in cache ongeldig maken vanuit AEM {#invalidating-cached-pages-from-aem}

Wanneer u Dispatcher gebruikt met AEM, moet de interactie zo worden geconfigureerd dat een effectief cachebeheer wordt gegarandeerd. Afhankelijk van uw milieu, kan de configuratie prestaties ook verhogen.

## AEM-gebruikersaccounts instellen {#setting-up-aem-user-accounts}

De standaard `admin` gebruikersrekening wordt gebruikt om de replicatieagenten voor authentiek te verklaren die door gebrek worden geïnstalleerd. Creeer een specifieke gebruikersrekening voor gebruik met replicatieagenten.

Voor meer informatie, zie de sectie [ replicatie en vervoergebruikers ](https://experienceleague.adobe.com/nl/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#VerificationSteps) van de Controlelijst van de Veiligheid van AEM vormen.

<!-- OLD URL from above https://helpx.adobe.com/nl/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps -->

## De Dispatcher-cache ongeldig maken vanuit de ontwerpomgeving {#invalidating-dispatcher-cache-from-the-authoring-environment}

Een replicatieagent op de AEM-auteurinstantie verzendt een verzoek tot ongeldigmaking van het cachegeheugen naar Dispatcher wanneer een pagina wordt gepubliceerd. Dispatcher vernieuwt het bestand uiteindelijk in de cache wanneer nieuwe inhoud wordt gepubliceerd.

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

Gebruik de volgende procedure om een replicatieagent op de auteursinstantie van AEM te vormen. De configuratie maakt het Dispatcher-cachegeheugen ongeldig na activering van de pagina:

1. Open de AEM Tools Console. (`https://localhost:4502/miscadmin#/etc`)
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten op auteur. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik uitgeven, en op het lusje van Montages zorgen ervoor dat **Toegelaten** wordt geselecteerd.

1. (facultatief) om alias of verzoeken van de de ongeldig wordingsweg van de ijl toe te laten selecteren de **optie van de Update van de Alias**.
1. Ga op het tabblad Vervoer naar Dispatcher door de URI in te voeren.

   Als u de standaardDispatcher Flush agent gebruikt, werk hostname en haven bij; bijvoorbeeld, https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **Nota:** voor de Uitlijningsagenten van Dispatcher, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*` . U kunt een URL van `/path1/invalidate.cache` gebruiken om het eerste landbouwbedrijf te richten en `/path2/invalidate.cache` om het tweede landbouwbedrijf te richten. Voor meer informatie, zie [ Gebruikend Dispatcher met Veelvoudige Domeinen ](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Klik op OK om de agent te activeren.

Alternatief, kunt u tot de Vlek van Dispatcher agent van de [ Aanraak UI van AEM ook toegang hebben en vormen ](https://experienceleague.adobe.com/nl/docs/experience-manager-65/content/implementing/deploying/configuring/replication#configuring-a-dispatcher-flush-agent).

Voor meer details op hoe te om toegang tot ijdelheid URLs toe te laten, zie [ Toelatend Toegang tot Vanity URLs ](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>De agent voor het leegmaken van het geheime voorgeheugen van Dispatcher vereist geen gebruikersnaam en wachtwoord, maar als gevormd worden zij verzonden met basisauthentificatie.

Er zijn twee mogelijke problemen met deze aanpak:

* De Dispatcher moet bereikbaar zijn vanuit de ontwerpinstantie. Als uw netwerk (bijvoorbeeld, de firewall) dusdanig wordt gevormd dat de toegang tussen twee beperkt is, kan deze situatie niet het geval zijn.

* De openbaarmaking en de ongeldigmaking van de cache vinden tegelijkertijd plaats. Afhankelijk van de timing kan een gebruiker een pagina aanvragen vlak nadat deze uit de cache is verwijderd en vlak voordat de nieuwe pagina wordt gepubliceerd. AEM retourneert nu de oude pagina en de Dispatcher plaatst deze opnieuw in het cachegeheugen. Deze situatie is eerder een probleem voor grote sites.

## De Dispatcher-cache ongeldig maken via een instantie Publish {#invalidating-dispatcher-cache-from-a-publishing-instance}

Onder bepaalde omstandigheden kunnen de prestaties worden verbeterd door cachebeheer over te brengen van de ontwerpomgeving naar een publicatie-instantie. Het is dan de publicatieomgeving (niet de AEM-ontwerpomgeving) die een verzoek tot het ongeldig maken van de cache naar Dispatcher verzendt wanneer een gepubliceerde pagina wordt ontvangen.

Deze omstandigheden omvatten:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Het verhinderen van mogelijke timingconflicten tussen AEM Dispatcher en publiceer instantie (zie [ het ongeldig maken van het geheime voorgeheugen van Dispatcher van het Authoring Milieu ](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Het systeem bevat verschillende publicatieinstanties die zich op servers met hoge prestaties bevinden, en slechts één ontwerpinstantie.

>[!NOTE]
>
>Een ervaren beheerder van AEM zou het besluit moeten nemen om deze methode te gebruiken.

Een replicatieagent die op de publicatieinstantie werkt, bestuurt de Dispatcher flush. Nochtans, wordt de configuratie gemaakt in het auteursmilieu en dan overgebracht door de agent te activeren:

1. Open de AEM Tools Console.
1. Open de vereiste replicatieagent onder Hulpmiddelen/replicatie/Agenten bij publiceren. U kunt de Dispatcher Flush-agent gebruiken die standaard is geïnstalleerd.
1. Klik uitgeven, en op het lusje van Montages zorgen ervoor dat **Toegelaten** wordt geselecteerd.
1. (facultatief) om alias of verzoeken van de de ongeldig wordingsweg van de ijl toe te laten selecteren de **optie van de Update van de Alias**.
1. Ga op het tabblad Vervoer naar Dispatcher door de benodigde URI in te voeren.\
   Als u de standaard Dispatcher Flush-agent gebruikt, werkt u de hostnaam en -poort bij, bijvoorbeeld `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Nota:** voor de Uitlijningsagenten van Dispatcher, wordt het bezit van URI gebruikt slechts als u op weg-gebaseerde virtuele gastheeringangen gebruikt om tussen landbouwbedrijven te onderscheiden. U gebruikt dit gebied om het landbouwbedrijf te richten om ongeldig te maken. farm #1 heeft bijvoorbeeld een virtuele host van `www.mysite.com/path1/*` en farm #2 heeft een virtuele host van `www.mysite.com/path2/*` . U kunt een URL van `/path1/invalidate.cache` gebruiken om het eerste landbouwbedrijf te richten en `/path2/invalidate.cache` om het tweede landbouwbedrijf te richten. Voor meer informatie, zie [ Gebruikend Dispatcher met Veelvoudige Domeinen ](dispatcher-domains.md).

1. Configureer desgewenst andere parameters.
1. Meld u aan bij de publicatieinstantie en valideer de configuratie van de spoelagent. Zorg er ook voor dat deze functie is ingeschakeld.
1. Herhaal deze bewerking voor elke betrokken publicatie-instantie.

Na het vormen, wanneer u een pagina van auteur activeert om te publiceren, stelt deze agent een standaardreplicatie in werking. Het logboek bevat berichten die verzoeken aangeven die afkomstig zijn van uw publicatieserver, vergelijkbaar met het volgende voorbeeld:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## De Dispatcher-cache handmatig ongeldig maken {#manually-invalidating-the-dispatcher-cache}

Als u de Dispatcher-cache ongeldig wilt maken (of wilt leegmaken) zonder een pagina te activeren, kunt u een HTTP-aanvraag naar de Dispatcher verzenden. U kunt bijvoorbeeld een AEM-toepassing maken waarmee beheerders of andere toepassingen de cache kunnen leegmaken.

Door de HTTP-aanvraag verwijdert de Dispatcher specifieke bestanden uit de cache. De Dispatcher vernieuwt desgewenst de cache met een nieuwe kopie.

### In cache opgeslagen bestanden verwijderen {#delete-cached-files}

Geef een HTTP- verzoek uit dat de Dispatcher veroorzaakt om dossiers van het geheime voorgeheugen te schrappen. Dispatcher plaatst de bestanden alleen opnieuw in cache wanneer het een clientverzoek voor de pagina ontvangt. Het verwijderen van cachebestanden op deze manier is geschikt voor websites die waarschijnlijk geen gelijktijdige aanvragen voor dezelfde pagina ontvangen.

De HTTP-aanvraag heeft de volgende vorm:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher verwijdert (verwijdert) de bestanden en mappen in de cache die een naam hebben die overeenkomt met de waarde van de header `CQ-Handler` . Een `CQ-Handle` van `/content/geomtrixx-outdoors/en` komt bijvoorbeeld overeen met de volgende items:

* Alle bestanden (van een willekeurige bestandsextensie) met de naam `en` in de map `geometrixx-outdoors`

* Elke map met de naam `_jcr_content` onder de map `en` (die, indien aanwezig, in de cache opgeslagen renderingen van subknooppunten van de pagina bevat)

Alle andere bestanden in de Dispatcher-cache (of tot een bepaald niveau, afhankelijk van de instelling `/statfileslevel` ) worden ongeldig gemaakt door op het `.stat` -bestand te tikken. De laatste wijzigingsdatum van dit bestand wordt vergeleken met de laatste wijzigingsdatum van een document in de cache en het document wordt opnieuw opgehaald als het `.stat` -bestand nieuwer is. Zie [ het Invalideren van Dossiers door het Niveau van de Omslag ](dispatcher-configuration.md#main-pars_title_26) voor details.

Ongeldige validatie (dat wil zeggen het aanraken van .stat-bestanden) kan worden voorkomen door een extra koptekst te verzenden `CQ-Action-Scope: ResourceOnly` . Deze functionaliteit kan worden gebruikt om bepaalde bronnen te leegmaken. Alles zonder andere delen van de cache ongeldig te maken, zoals JSON-gegevens. Deze gegevens worden dynamisch gemaakt en moeten regelmatig worden leeggemaakt, onafhankelijk van de cache. Bijvoorbeeld, die gegevens vertegenwoordigen die van een derdesysteem worden verkregen om nieuws, voorraadtikkers, etc. te tonen.

### Bestanden verwijderen en opnieuw plaatsen {#delete-and-recache-files}

Geef een HTTP-aanvraag uit die ervoor zorgt dat de Dispatcher cachebestanden verwijdert en het bestand direct ophaalt en opnieuw plaatst. U kunt bestanden verwijderen en onmiddellijk opnieuw in cache plaatsen wanneer websites waarschijnlijk gelijktijdige clientverzoeken voor dezelfde pagina ontvangen. Met Direct recaching zorgt Dispatcher ervoor dat de pagina slechts één keer wordt opgehaald en in cache wordt geplaatst in plaats van één keer voor elk van de gelijktijdige clientverzoeken.

**Nota:** het schrappen en het in het voorgeheugen onderbrengen van dossiers zou op de het publiceren instantie slechts moeten worden uitgevoerd. Wanneer uitgevoerd vanaf de auteurinstantie, komen de rasvoorwaarden voor wanneer pogingen om middelen terug te winnen voorkomen alvorens zij zijn gepubliceerd.

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

De paginapaden die onmiddellijk opnieuw in cache moeten worden geplaatst, worden weergegeven op afzonderlijke regels in de berichttekst. De waarde van `CQ-Handle` is het pad van een pagina die de pagina&#39;s ongeldig maakt om opnieuw te worden gemaakt. (Zie de `/statfileslevel` parameter van het [ 2&rbrace; configuratiepunt van het Geheime voorgeheugen &lbrace;.) In het volgende voorbeeld wordt het HTTP-aanvraagbericht verwijderd en doorgehaald ](dispatcher-configuration.md#main-pars_146_44_0010) :`/content/geometrixx-outdoors/en.html page`

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Voorbeeld spoelservlet {#example-flush-servlet}

De volgende code implementeert een servlet die een verzoek tot validatie naar Dispatcher verzendt. De servlet ontvangt een aanvraagbericht dat `handle` en `page` parameters bevat. Deze parameters geven respectievelijk de waarde van de header `CQ-Handle` en het pad van de pagina die moet worden teruggedraaid. De servlet gebruikt de waarden om de HTTP-aanvraag voor Dispatcher samen te stellen.

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
