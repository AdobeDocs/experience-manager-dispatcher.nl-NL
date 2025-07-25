---
title: Beveiligde inhoud in cache opslaan
description: Leer hoe plaatsen met toestemming werkt in Dispatcher.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '923'
ht-degree: 0%

---

# Beveiligde inhoud in cache opslaan {#caching-secured-content}

Met machtigingsgevoelige caching kunt u beveiligde pagina&#39;s in het cachegeheugen plaatsen. Dispatcher controleert de toegangsmachtigingen van de gebruiker voor een pagina voordat de pagina in de cache wordt geleverd.

Dispatcher omvat de module AuthChecker die toestemming-gevoelig caching uitvoert. Wanneer de module wordt geactiveerd, roept de Dispatcher een servlet van AEM aan om gebruikersauthentificatie en vergunning voor de gevraagde inhoud uit te voeren. De servletreactie bepaalt of de inhoud aan Webbrowser van het geheime voorgeheugen of niet wordt geleverd.

Omdat de methodes van authentificatie en vergunning voor de plaatsing van AEM specifiek zijn, wordt u vereist om servlet te creëren.

>[!NOTE]
>
>Gebruik `deny` -filters om algemene beveiligingsbeperkingen in te stellen. Gebruik toestemming-gevoelige caching voor pagina&#39;s die worden gevormd om toegang tot een ondergroep van gebruikers of groepen toe te staan.

De volgende diagrammen illustreren de orde van gebeurtenissen die voorkomen wanneer Webbrowser om een pagina vraagt waarvoor toestemming-gevoelig caching wordt gebruikt.

## Pagina is in cache geplaatst en de gebruiker is geautoriseerd {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher bepaalt dat de aangevraagde inhoud in de cache wordt opgeslagen en geldig is.
1. Dispatcher verzendt een aanvraagbericht naar de render. De sectie HEAD bevat alle kopregels uit de browseraanvraag.
1. Renderen roept de automatische controleerserver aan om de veiligheidscontrole uit te voeren en antwoordt aan Dispatcher. Het antwoordbericht bevat de HTTP-statuscode 200 om aan te geven dat de gebruiker geautoriseerd is.
1. Dispatcher stuurt een antwoordbericht naar de browser dat bestaat uit de koptekstregels van de renderreactie en de inhoud in de cache.

## De pagina is niet in cache geplaatst en de gebruiker is geautoriseerd {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher bepaalt dat de inhoud niet in de cache wordt geplaatst of moet worden bijgewerkt.
1. Dispatcher stuurt het oorspronkelijke verzoek door naar de rendermethode.
1. Renderen roept AEM authorizer servlet (dit servlet is niet de servlet van Dispatcher AuthChcker) aan om een veiligheidscontrole uit te voeren. Wanneer de gebruiker wordt geautoriseerd, omvat teruggeven de teruggegeven pagina in het lichaam van het antwoordbericht.
1. Dispatcher stuurt de reactie door naar de browser. Dispatcher voegt de hoofdtekst van het antwoordbericht van de renderbewerking toe aan de cache.

## Gebruiker is niet geautoriseerd {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher controleert de cache.
1. Dispatcher verzendt een aanvraagbericht naar de rendermethode die alle headerregels uit het verzoek van de browser bevat.
1. Renderen roept de server van de Controleur van de Auth om een veiligheidscontrole uit te voeren, die ontbreekt, en teruggeeft door:sturen het originele verzoek aan Dispatcher.
1. Dispatcher stuurt het oorspronkelijke verzoek door naar de rendermethode.
1. Renderen roept AEM authorizer servlet (dit servlet is niet de servlet van Dispatcher AuthChcker) aan om een veiligheidscontrole uit te voeren. Wanneer de gebruiker wordt geautoriseerd, omvat teruggeven de teruggegeven pagina in het lichaam van het antwoordbericht.
1. Dispatcher stuurt de reactie door naar de browser. Dispatcher voegt de hoofdtekst van het antwoordbericht van de renderbewerking toe aan de cache.

## Voer toestemmingsgevoelige caching uit {#implementing-permission-sensitive-caching}

Om toestemming-gevoelig caching uit te voeren, voer de volgende taken uit:

* Ontwikkelen van een servlet die authentificatie en vergunning uitvoert
* De Dispatcher configureren

>[!NOTE]
>
>Beveiligde bronnen worden doorgaans in een aparte map opgeslagen dan onbeveiligde bestanden. Bijvoorbeeld: /content/secure/

>[!NOTE]
>
>Wanneer er een CDN (of een ander geheime voorgeheugen) vóór de Dispatcher is, dan zou u de in het voorgeheugen onderbrengende kopballen dienovereenkomstig moeten plaatsen zodat CDN niet de privé inhoud in het voorgeheugen onderbrengt. Bijvoorbeeld: `Header always set Cache-Control private` .
>&#x200B;>Voor AEM as a Cloud Service, zie de [ Caching ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/implementing/content-delivery/caching) pagina voor meer details over hoe te om privé caching kopballen te plaatsen.

## De servlet Auth Checker maken {#create-the-auth-checker-servlet}

Maak en implementeer een servlet die de verificatie en autorisatie uitvoert van de gebruiker die de webinhoud aanvraagt. De servlet kan om het even welke authentificatie gebruiken. Zij kan ook elke vergunningsmethode gebruiken. Het kan bijvoorbeeld de AEM-gebruikersaccount en ACL&#39;s in de gegevensopslagruimte gebruiken. Of u kunt hiervoor een LDAP-opzoekservice gebruiken. U implementeert de servlet op de AEM-instantie die Dispatcher gebruikt als de render.

De servlet moet toegankelijk zijn voor alle gebruikers. Daarom moet uw servlet de klasse `org.apache.sling.api.servlets.SlingSafeMethodsServlet` uitbreiden, die alleen-lezen toegang tot het systeem biedt.

De servlet ontvangt alleen HEAD-aanvragen van de render, dus u moet de methode `doHead` alleen implementeren.

Renderen omvat URI van het gevraagde middel als parameter van het HTTP- verzoek. Een verificatieserver is bijvoorbeeld toegankelijk via `/bin/permissioncheck` . Als u een beveiligingscontrole wilt uitvoeren op de pagina /content/geometrixx-outdoors/en.html, bevat de render de volgende URL in de HTTP-aanvraag:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Het servlet-antwoordbericht moet de volgende HTTP-statuscodes bevatten:

* 200: Verificatie en autorisatie geslaagd.

Het volgende voorbeeldservlet verkrijgt URL van het gevraagde middel van het HTTP- verzoek. De code gebruikt de SCR-annotatie Felix `Property` om de waarde van de eigenschap `sling.servlet.paths` in te stellen op /bin/permissioncheck. In de methode `doHead` verkrijgt de servlet het sessieobject en gebruikt de methode `checkPermission` om de juiste antwoordcode te bepalen.

>[!NOTE]
>
>De waarde van de eigenschap sling.servlet.paths moet zijn ingeschakeld in de service `Sling` Servlet Resolver (org.apache.sling.servlets.resolver.SlingServletResolver).

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
>Als uw vereisten het caching van voor authentiek verklaarde documenten toestaan, plaats het /allowAuthorized bezit onder de /cache sectie aan `/allowAuthorized 1`. Zie het onderwerp [ Caching wanneer de Authentificatie ](/help/using/dispatcher-configuration.md) voor meer details wordt gebruikt.

De sectie auth_checker van de dispatcher.any file controleert het gedrag van toestemming-gevoelige caching. De sectie auth_checker bevat de volgende subsecties:

* `url`: De waarde van de eigenschap `sling.servlet.paths` van de servlet die de beveiligingscontrole uitvoert.

* `filter`: Filters die de mappen opgeven waarop machtigingsgevoelige caching wordt toegepast. Doorgaans wordt een `deny` -filter toegepast op alle mappen en worden `allow` -filters toegepast op beveiligde mappen.

* `headers` - Geeft de HTTP-headers op die de verificatieserver in de reactie bevat.

Wanneer Dispatcher wordt gestart, bevat het Dispatcher-logbestand het volgende bericht op foutopsporingsniveau:

`AuthChecker: initialized with URL 'configured_url'.`

De volgende voorbeeldsectie auth_checker vormt Dispatcher om servlet van het vorige onderwerp te gebruiken. De filtersectie zorgt ervoor dat machtigingscontroles alleen worden uitgevoerd op beveiligde HTML-bronnen.

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
