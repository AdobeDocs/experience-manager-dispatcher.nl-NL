---
title: Beveiligde inhoud in cache plaatsen
description: Leer hoe het in cache plaatsen met toestemming werkt in Dispatcher.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '910'
ht-degree: 0%

---

# Beveiligde inhoud in cache plaatsen {#caching-secured-content}

Met machtigingsgevoelige caching kunt u beveiligde pagina&#39;s in het cachegeheugen plaatsen. Dispatcher controleert de toegangsmachtigingen van de gebruiker voor een pagina voordat de pagina in de cache wordt geleverd.

De afzender omvat de module AuthChecker die toestemming-gevoelig caching uitvoert. Wanneer de module wordt geactiveerd, roept de Dispatcher een AEM servlet aan om gebruikersauthentificatie en vergunning voor de gevraagde inhoud uit te voeren. De servletreactie bepaalt of de inhoud aan Webbrowser van het geheime voorgeheugen of niet wordt geleverd.

Omdat de methodes van authentificatie en vergunning voor de AEM plaatsing specifiek zijn, wordt u vereist om servlet tot stand te brengen.

>[!NOTE]
>
>Gebruiken `deny` filters om algemene beveiligingsbeperkingen af te dwingen. Gebruik toestemming-gevoelige caching voor pagina&#39;s die worden gevormd om toegang tot een ondergroep van gebruikers of groepen toe te staan.

De volgende diagrammen illustreren de orde van gebeurtenissen die voorkomen wanneer Webbrowser om een pagina vraagt waarvoor toestemming-gevoelig caching wordt gebruikt.

## Pagina is in cache geplaatst en de gebruiker is geautoriseerd {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher bepaalt dat de aangevraagde inhoud in de cache is opgeslagen en geldig is.
1. Dispatcher verzendt een aanvraagbericht naar renderen. De sectie HEAD bevat alle kopregels uit de browseraanvraag.
1. Renderen roept de auth checker servlet aan om de veiligheidscontrole uit te voeren en aan Dispatcher antwoordt. Het antwoordbericht bevat de HTTP-statuscode 200 om aan te geven dat de gebruiker geautoriseerd is.
1. Dispatcher stuurt een antwoordbericht naar de browser dat bestaat uit de koptekstregels van de renderreactie en de inhoud in de cache.

## De pagina is niet in cache geplaatst en de gebruiker is geautoriseerd {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher bepaalt dat de inhoud niet in het cachegeheugen is opgeslagen of moet worden bijgewerkt.
1. Verzender stuurt het oorspronkelijke verzoek door naar de rendermethode.
1. Renderen roept AEM autorisator servlet (dit is niet de servlet van AuthChcker van de Dispatcher) om een veiligheidscontrole uit te voeren. Wanneer de gebruiker wordt geautoriseerd, omvat teruggeven de teruggegeven pagina in het lichaam van het antwoordbericht.
1. De verzender stuurt de reactie door naar de browser. Dispatcher voegt de hoofdtekst van het reactiebericht van de render aan het geheime voorgeheugen toe.

## Gebruiker is niet geautoriseerd {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher controleert de cache.
1. Dispatcher verstuurt een aanvraagbericht naar de render die alle koptekstregels uit de browseraanvraag bevat.
1. Renderen roept de server van de Controleur van de Auth om een veiligheidscontrole uit te voeren die ontbreekt, en teruggeeft door:sturen het originele verzoek aan Dispatcher.
1. Verzender stuurt het oorspronkelijke verzoek door naar de rendermethode.
1. Renderen roept AEM autorisator servlet (dit is niet de servlet van AuthChcker van de Dispatcher) om een veiligheidscontrole uit te voeren. Wanneer de gebruiker wordt geautoriseerd, omvat teruggeven de teruggegeven pagina in het lichaam van het antwoordbericht.
1. De verzender stuurt de reactie door naar de browser. Dispatcher voegt de hoofdtekst van het reactiebericht van de render aan het geheime voorgeheugen toe.

## Het uitvoeren van toestemming-gevoelige caching {#implementing-permission-sensitive-caching}

Om toestemming-gevoelig caching uit te voeren, voer de volgende taken uit:

* Ontwikkelen van een servlet die authentificatie en vergunning uitvoert
* De Dispatcher configureren

>[!NOTE]
>
>Beveiligde bronnen worden doorgaans in een aparte map opgeslagen dan onbeveiligde bestanden. Bijvoorbeeld: /content/secure/

>[!NOTE]
>
>Wanneer er een CDN (of een ander geheime voorgeheugen) vóór de Dispatcher is, dan zou u de caching kopballen dienovereenkomstig moeten plaatsen zodat CDN niet de privé inhoud in het voorgeheugen onderbrengt. Bijvoorbeeld: `Header always set Cache-Control private`.
>Voor AEM as a Cloud Service, zie [Caching](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/content-delivery/caching) voor meer informatie over het instellen van headers voor persoonlijke caching.

## De servlet Auth Checker maken {#create-the-auth-checker-servlet}

Maak en implementeer een servlet die de verificatie en autorisatie uitvoert van de gebruiker die de webinhoud aanvraagt. servlet kan om het even welke authentificatie en vergunningsmethode, zoals de AEM gebruikersrekening en bewaarplaats ACLs, of een opzoekdienst gebruiken LDAP. U stelt servlet aan de AEM instantie op die Dispatcher als teruggeeft gebruikt.

De servlet moet toegankelijk zijn voor alle gebruikers. Daarom moet uw servlet de `org.apache.sling.api.servlets.SlingSafeMethodsServlet` klasse, die alleen-lezen toegang tot het systeem biedt.

servlet ontvangt slechts HEAD- verzoeken van teruggeven, zodat moet u slechts uitvoeren `doHead` methode.

Renderen omvat URI van het gevraagde middel als parameter van het HTTP- verzoek. Een verificatieserver is bijvoorbeeld toegankelijk via `/bin/permissioncheck`. Als u een beveiligingscontrole wilt uitvoeren op de pagina /content/geometrixx-outdoors/en.html, bevat de render de volgende URL in de HTTP-aanvraag:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Het servlet-antwoordbericht moet de volgende HTTP-statuscodes bevatten:

* 200: Verificatie en autorisatie geslaagd.

Het volgende voorbeeldservlet verkrijgt URL van het gevraagde middel van het HTTP- verzoek. De code gebruikt SCR Felix `Property` aantekening om de waarde van de `sling.servlet.paths` eigenschap naar /bin/permissioncheck. In de `doHead` methode, verkrijgt servlet het zittingsvoorwerp en gebruikt `checkPermission` methode om de juiste responscode te bepalen.

>[!NOTE]
>
>De waarde van de eigenschap sling.servlet.paths moet zijn ingeschakeld in de service Sling Servlet Resolver (org.apache.sling.servlets.resolver.SlingServletResolver).

### Voorbeeld-servlet {#example-servlet}

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

import javax.jcr.Session;

@Component(metatype=false)
@Service
public class AuthcheckerServlet extends SlingSafeMethodsServlet {
 
    @Property(value="/bin/permissioncheck")
    static final String SERVLET_PATH="sling.servlet.paths";
    
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    public void doHead(SlingHttpServletRequest request, SlingHttpServletResponse response) {
     try{ 
      //retrieve the requested URL
      String uri = request.getParameter("uri");
      //obtain the session from the request
      Session session = request.getResourceResolver().adaptTo(javax.jcr.Session.class);     
      //perform the permissions check
      try {
       session.checkPermission(uri, Session.ACTION_READ);
       logger.info("authchecker says OK");
       response.setStatus(SlingHttpServletResponse.SC_OK);
      } catch(Exception e) {
       logger.info("authchecker says READ access DENIED!");
       response.setStatus(SlingHttpServletResponse.SC_FORBIDDEN);
      }
     }catch(Exception e){
      logger.error("authchecker servlet exception: " + e.getMessage());
     }
    }
}
```

## Dispatcher configureren voor caching met bevoegdheden {#configure-dispatcher-for-permission-sensitive-caching}

>[!NOTE]
>
>Als uw vereisten het caching van voor authentiek verklaarde documenten toestaan, plaats het /allowAuthorized bezit onder de /cache sectie aan `/allowAuthorized 1`. Zie het onderwerp [In cache plaatsen wanneer verificatie wordt gebruikt](/help/using/dispatcher-configuration.md) voor meer informatie .

De sectie auth_checker van de dispatcher.any file controleert het gedrag van toestemming-gevoelige caching. De sectie auth_checker bevat de volgende subsecties:

* `url`: De waarde van de `sling.servlet.paths` eigenschap van de servlet die de beveiligingscontrole uitvoert.

* `filter`: Filters die de mappen opgeven waarop machtigingsgevoelige caching wordt toegepast. Doorgaans wordt een `deny` filter wordt toegepast op alle mappen, en `allow` filters worden toegepast op beveiligde mappen.

* `headers`: Geeft de HTTP-headers aan die het verificatieserver in de reactie bevat.

Wanneer de Verzender begint, omvat het het logboekdossier van de Verzender het volgende debug-vlakke bericht:

`AuthChecker: initialized with URL 'configured_url'.`

De volgende voorbeeldsectie auth_checker vormt Dispatcher om servlet van het vorige onderwerp te gebruiken. De filtersectie veroorzaakt toestemmingscontroles die slechts op veilige middelen van HTML worden uitgevoerd.

### Voorbeeldconfiguratie {#example-configuration}

```xml
/auth_checker
  {
  # request is sent to this URL with '?uri=<page>' appended
  /url "/bin/permissioncheck"
      
  # only the requested pages matching the filter section below are checked,
  # all other pages get delivered unchecked
  /filter
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "/content/secure/*.html"
      /type "allow"
      }
    }
  # any header line returned from the auth_checker's HEAD request matching
  # the section below will be returned as well
  /headers
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "Set-Cookie:*"
      /type "allow"
      }
    }
  }
```
