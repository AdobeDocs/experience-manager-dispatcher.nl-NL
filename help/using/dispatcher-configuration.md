---
title: AEM Dispatcher configureren
description: Leer hoe u de Dispatcher configureert. Leer over steun voor IPv4 en IPv6, configuratiedossiers, omgevingsvariabelen, en het noemen van de instantie. Lees over het bepalen van landbouwbedrijven, het identificeren van virtuele gastheren, en meer.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: a9ef9d7d2fe5c421cd8039579fd84961ea901def
workflow-type: tm+mt
source-wordcount: '8941'
ht-degree: 0%

---

# Dispatcher configureren {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher-versies zijn onafhankelijk van AEM (Adobe Experience Manager). U bent mogelijk omgeleid naar deze pagina als u een koppeling naar de documentatie van Dispatcher hebt gevolgd. Die koppeling is ingesloten in de documentatie voor een vorige versie van AEM.

In de volgende secties wordt beschreven hoe u verschillende aspecten van de Dispatcher kunt configureren.

## Ondersteuning voor IPv4 en IPv6 {#support-for-ipv-and-ipv}

Alle elementen van AEM en Dispatcher kunnen in zowel IPv4 als IPv6 netwerken worden geïnstalleerd. Zie [ IPV4 en IPV6 ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#ipv-and-ipv).

## Dispatcher-configuratiebestanden {#dispatcher-configuration-files}

Standaard wordt de Dispatcher-configuratie opgeslagen in het tekstbestand van `dispatcher.any` , maar u kunt de naam en locatie van dit bestand tijdens de installatie wijzigen.

Het configuratiebestand bevat een reeks enkelvoudige of meergetaxeerde eigenschappen die het gedrag van de Dispatcher bepalen:

* Namen van eigenschappen worden voorafgegaan door een slash `/` .
* Bij eigenschappen met meerdere waarden worden onderliggende items ingesloten met behulp van accolades `{ }` .

Een voorbeeldconfiguratie is als volgt gestructureerd:

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

U kunt andere dossiers omvatten die tot de configuratie bijdragen:

* Als het configuratiebestand groot is, kunt u het opsplitsen in verschillende kleinere bestanden (die eenvoudiger te beheren zijn) en elk bestand opnemen.
* Bestanden opnemen die automatisch worden gegenereerd.

Als u bijvoorbeeld het bestand myFarm.any wilt opnemen in de `/farms` -configuratie gebruikt u de volgende code:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Om een waaier van dossiers te specificeren om te omvatten, gebruik de asterisk (`*`) als vervanging.

Als de bestanden `farm_1.any` tot en met `farm_5.any` bijvoorbeeld de configuratie van de farm 1 tot en met 5 bevatten, kunt u deze als volgt opnemen:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Omgevingsvariabelen gebruiken {#using-environment-variables}

U kunt omgevingsvariabelen gebruiken in tekenreeksgetaxeerde eigenschappen in het bestand dispatcher.any in plaats van de waarden hard te coderen. Als u de waarde van een omgevingsvariabele wilt opnemen, gebruikt u de indeling `${variable_name}` .

Bijvoorbeeld, als het dispatcher.any- dossier in de zelfde folder zoals de geheim voorgeheugenfolder is, kan de volgende waarde voor het [ docroot ](#specifying-the-cache-directory) bezit worden gebruikt:

```xml
/docroot "${PWD}/cache"
```

Als ander voorbeeld kunt u de volgende configuratie van de eigenschap [`/renders`](#defining-page-renderers-renders) gebruiken wanneer u een omgevingsvariabele met de naam `PUBLISH_IP` maakt die de hostnaam van de publicatieinstantie van AEM opslaat:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## De naam van de Dispatcher-instantie wijzigen {#naming-the-dispatcher-instance-name}

Gebruik de eigenschap `/name` om een unieke naam op te geven waarmee uw Dispatcher-instantie kan worden geïdentificeerd. De eigenschap `/name` is een eigenschap op hoofdniveau in de configuratiestructuur.

## Bedrijven definiëren {#defining-farms-farms}

De eigenschap `/farms` definieert een of meer sets Dispatcher-gedrag, waarbij elke set aan verschillende websites of URL&#39;s is gekoppeld. De eigenschap `/farms` kan een of meer boerderijen bevatten:

* Gebruik één landbouwbedrijf wanneer u Dispatcher al uw Web-pagina&#39;s of Websites op de zelfde manier wilt behandelen.
* Maak meerdere boerderijen wanneer verschillende gebieden van uw website of verschillende websites een ander Dispatcher-gedrag vereisen.

De eigenschap `/farms` is een eigenschap op hoofdniveau in de configuratiestructuur. Als u een farm wilt definiëren, voegt u een onderliggende eigenschap toe aan de eigenschap `/farms` . Gebruik een bezitsnaam die het landbouwbedrijf binnen de instantie van Dispatcher uniek identificeert.

De eigenschap `/farmname` is multiwaardeerd en bevat andere eigenschappen die het gedrag van Dispatcher definiëren:

* De URL&#39;s van de pagina&#39;s waarop het landbouwbedrijf van toepassing is.
* Een of meer service-URL&#39;s (doorgaans van AEM-publicatie-instanties) die moeten worden gebruikt voor het weergeven van documenten.
* De statistische gegevens die moeten worden gebruikt voor meerdere renderers van documenten die een taakverdeling hebben.
* Verschillende andere gedragingen, zoals welke bestanden in cache moeten worden geplaatst en waar ze moeten worden opgeslagen.

De waarde kan elk alfanumeriek teken (a-z, 0-9) bevatten. In het volgende voorbeeld wordt de skeletdefinitie getoond voor twee boerderijen met de naam `/daycom` en `/docsdaycom` :

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>Als u meer dan één gebruikt geef landbouwbedrijf terug, wordt de lijst geëvalueerd bottom-up. Deze stroom is relevant wanneer het bepalen van [ Virtuele Gastheren ](#identifying-virtual-hosts-virtualhosts) voor uw websites.

Elk landbouwbedrijfbezit kan de volgende kindeigenschappen bevatten:

| Eigenschapnaam | Beschrijving |
|--- |--- |
| [ /homepage](#specify-a-default-page-iis-only-homepage) | Standaardstartpagina (optioneel) (alleen IIS) |
| [ /clientheaders ](#specifying-the-http-headers-to-pass-through-clientheaders) | De headers van de HTTP-client-aanvraag die moeten worden doorgegeven. |
| [/virtualhosts ](#identifying-virtual-hosts-virtualhosts) | De virtuele hosts van deze boerderij. |
| [ /sessionmanagement ](#enabling-secure-sessions-sessionmanagement) | Ondersteuning voor sessiebeheer en verificatie. |
| [/renders ](#defining-page-renderers-renders) | De servers die gerenderde pagina&#39;s leveren (AEM publiceert instanties doorgaans). |
| [/filter ](#configuring-access-to-content-filter) | Hiermee definieert u de URL&#39;s waartoe de Dispatcher toegang biedt. |
| [/vanity_urls ](#enabling-access-to-vanity-urls-vanity-urls) | Vormt toegang tot vanity URLs. |
| [/propagateSyndPost ](#forwarding-syndication-requests-propagatesyndpost) | Steun voor de doorzending van verzoeken om syndicatie. |
| [/cache ](#configuring-the-dispatcher-cache-cache) | Vormt caching gedrag. |
| [/statistics ](#configuring-load-balancing-statistics) | Statistische categorieën definiëren voor berekeningen van de taakverdeling. |
| [/stickyConnectionsFor ](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | De map die kleverige documenten bevat. |
| [/health_check ](#specifying-a-health-check-page) | De URL die moet worden gebruikt om de beschikbaarheid van de server te bepalen. |
| [/retryDelay ](#specifying-the-page-retry-delay) | De vertraging voordat een mislukte verbinding opnieuw wordt geprobeerd. |
| [/unavailablePenalty ](#reflecting-server-unavailability-in-dispatcher-statistics) | Sancties die van invloed zijn op statistieken voor berekeningen voor taakverdeling. |
| [/failover ](#using-the-failover-mechanism) | Verzend verzoeken opnieuw naar verschillende renders wanneer het oorspronkelijke verzoek ontbreekt. |
| [/auth_checker ](permissions-cache.md) | Voor toestemming-gevoelig caching, zie [ In het voorgeheugen onderbrengend Beveiligde Inhoud ](permissions-cache.md). |

## Een standaardpagina opgeven (alleen IIS) - `/homepage` {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>De `/homepage` parameter (IIS slechts) werkt niet meer. In plaats daarvan, zou u [ IIS URL moeten gebruiken herschrijft Module ](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Gebruik de module `mod_rewrite` als u Apache gebruikt. Zie de Apache- website documentatie voor informatie over `mod_rewrite` (bijvoorbeeld, [ Apache 2.4 ](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). Wanneer u `mod_rewrite` gebruikt, is het aan te raden de markering &#39;passthrough|PT&#39; (doorgeven naar volgende handler) te gebruiken om de engine voor herschrijven te forceren het `uri` veld van de interne `request_rec` structuur in te stellen op de waarde van het `filename` -veld.

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## De HTTP-headers opgeven die moeten worden doorgegeven {#specifying-the-http-headers-to-pass-through-clientheaders}

De eigenschap `/clientheaders` definieert een lijst met HTTP-headers die Dispatcher doorgeeft van de HTTP-client-aanvraag naar de renderer (AEM-instantie).

Standaard stuurt de Dispatcher de standaard HTTP-headers door naar de AEM-instantie. In sommige gevallen wilt u mogelijk extra kopteksten doorsturen of specifieke koppen verwijderen:

* Voeg kopballen, zoals douanekopballen, toe die uw instantie van AEM in de HTTP- aanvraag verwacht.
* Verwijder kopballen, zoals authentificatiekopballen die slechts relevant voor de Webserver zijn.

Als u de reeks kopballen aanpast om over te gaan, moet u een uitvoerige lijst van kopballen, met inbegrip van die kopballen specificeren die normaal inbegrepen door gebrek zijn.

Een Dispatcher-instantie die pagina-activeringsverzoeken voor publicatie-instanties afhandelt, vereist bijvoorbeeld de header `PATH` in de `/clientheaders` -sectie. De header `PATH` maakt communicatie mogelijk tussen de replicatieagent en de Dispatcher.

De volgende code is een voorbeeldconfiguratie voor `/clientheaders`:

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## Virtuele hosts identificeren {#identifying-virtual-hosts-virtualhosts}

De eigenschap `/virtualhosts` definieert een lijst met alle combinaties van hostnaam en URI die Dispatcher accepteert voor deze farm. U kunt het asteriskteken (`*`) als vervanging gebruiken. Waarden voor de eigenschap /`virtualhosts` gebruiken de volgende indeling:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Optioneel) Ofwel `https://` of `https://.`
* `host`: De naam of het IP-adres van de hostcomputer en het poortnummer, indien nodig. (Zie [ https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23 ](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Optioneel) Het pad naar de bronnen.

In het volgende voorbeeld behandelt de configuratie aanvragen voor de domeinen `.com` en `.ch` van myCompany en alle domeinen van mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

De volgende configuratiehandvatten *alle* verzoeken:

```xml
   /virtualhosts
    {
    "*"
    }
```

### De virtuele host oplossen {#resolving-the-virtual-host}

Wanneer Dispatcher een HTTP- of HTTPS-aanvraag ontvangt, wordt de virtuele hostwaarde gevonden die het best overeenkomt met de headers `host,` `uri` en `scheme` van de aanvraag. Dispatcher evalueert de waarden in de eigenschappen `virtualhosts` in de volgende volgorde:

* Dispatcher begint bij het laagste landbouwbedrijf en gaat omhoog in dispatcher.any dossier.
* Dispatcher begint voor elke farm met de bovenste waarde in de eigenschap `virtualhosts` en gaat vervolgens verder in de lijst met waarden.

Dispatcher vindt de best-passende virtuele gastheerwaarde op de volgende manier:

* De eerst aangetroffen virtuele host die overeenkomt met alle drie de `host` , de `scheme` en de `uri` van de aanvraag, wordt gebruikt.
* Als er geen `virtualhosts` -waarden zijn met `scheme` - en `uri` -onderdelen die beide overeenkomen met `scheme` en `uri` van de aanvraag, wordt de eerst aangetroffen virtuele host gebruikt die overeenkomt met `host` van de aanvraag.
* Als geen `virtualhosts` waarden een gastheerdeel hebben dat de gastheer van het verzoek aanpast, wordt de hoogste virtuele gastheer van het hoogste landbouwbedrijf gebruikt.

Plaats daarom de standaard virtuele host boven aan de eigenschap `virtualhosts` . Plaats het in het hoogste landbouwbedrijf van uw `dispatcher.any` dossier.

### Voorbeeld virtuele hostresolutie {#example-virtual-host-resolution}

In het volgende voorbeeld ziet u een fragment uit een `dispatcher.any` -bestand dat twee Dispatcher-farm definieert en elke farm definieert een `virtualhosts` -eigenschap.

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany
    {
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

Gebruikend dit voorbeeld, toont de volgende lijst de virtuele gastheren die voor de bepaalde HTTP- verzoeken worden opgelost:

| Aanvraag-URL | Opgeloste virtuele host |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Beveiligde sessies inschakelen - `/sessionmanagement` {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` Stel dit in op `"0"` in de sectie `/cache` om deze functie in te schakelen. Zoals gedetailleerd in het [ Caching wanneer de Authentificatie wordt gebruikt ](#caching-when-authentication-is-used) sectie, wanneer u `/allowAuthorized 0 ` verzoeken plaatst die authentificatieinformatie omvatten **&#x200B;**&#x200B;niet in het cachegeheugen wordt opgeslagen. Als toestemming-gevoelig caching wordt vereist, zie de [ In het voorgeheugen onderbrengende Beveiligde Inhoud ](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/permissions-cache) pagina.

Creeer een veilige zitting voor toegang tot teruggeven landbouwbedrijf zodat de gebruikers moeten login om het even welke pagina in het landbouwbedrijf toegang hebben. Na het programma openen, kunnen de gebruikers tot pagina&#39;s in het landbouwbedrijf toegang hebben. Zie [ Creërend een Gesloten Groep van de Gebruiker ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/cug#creating-the-user-group-to-be-used) voor informatie over het gebruiken van deze eigenschap met CUGs. Ook, zie Dispatcher [ Controlelijst van de Veiligheid ](/help/using/security-checklist.md) alvorens levend te gaan.

De eigenschap `/sessionmanagement` is een subeigenschap van `/farms` .

>[!CAUTION]
>
>Als gedeelten van uw website verschillende toegangsvereisten gebruiken, moet u meerdere boerderijen definiëren.

**/sessionManagement** heeft verschillende subparameters:

**/directory** (verplicht)

De map waarin de sessiegegevens worden opgeslagen. Als de map niet bestaat, wordt deze gemaakt.

>[!CAUTION]
>
> Wanneer het vormen van de folder subparameter, **richt niet** aan de wortelomslag (`/directory "/"`) aangezien het ernstige problemen kan veroorzaken. Geef altijd het pad op naar de map waarin de sessiegegevens worden opgeslagen. Bijvoorbeeld:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (optioneel)

Hoe de sessiegegevens worden gecodeerd. Gebruik `md5` voor codering met het md5-algoritme of `hex` voor hexadecimale codering. Als u de sessiegegevens versleutelt, kan een gebruiker met toegang tot het bestandssysteem de sessie-inhoud niet lezen. De standaardwaarde is `md5` .

**/header** (optioneel)

De naam van de HTTP-header of het cookie waarin de autorisatiegegevens zijn opgeslagen. Gebruik `HTTP:<header-name>` als u de informatie in de http-header opslaat. Gebruik `Cookie:<header-name>` als u de gegevens in een cookie wilt opslaan. Wanneer u geen waarde opgeeft, wordt `HTTP:authorization` gebruikt.

**/timeout** (optioneel)

Het aantal seconden tot de sessietijden uit nadat deze voor het laatst zijn gebruikt. Als `"800"` niet is opgegeven, wordt de sessie iets langer dan 13 minuten na het laatste verzoek van de gebruiker uitgevoerd.

Een voorbeeldconfiguratie ziet er als volgt uit:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## Paginarenderers definiëren {#defining-page-renderers-renders}

De eigenschap `/renders` definieert de URL waarnaar de Dispatcher een verzoek verzendt om een document te renderen. In de volgende voorbeeldsectie `/renders` wordt één AEM-instantie voor rendering geïdentificeerd:

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

In de volgende voorbeeldsectie `/renders` wordt een AEM-instantie geïdentificeerd die op dezelfde computer als Dispatcher wordt uitgevoerd:

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

In de volgende voorbeeldsectie `/renders` worden renderverzoeken gelijkelijk verdeeld over twee AEM-instanties:

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### Renderopties {#renders-options}

**/timeout**

Geeft de time-out van de verbinding aan die de AEM-instantie benadert, in milliseconden. De standaardwaarde is `"0"` , zodat de Dispatcher oneindig wacht.

**/receiveTimeout**

Geeft de tijd op in milliseconden die een reactie mag afleggen. De standaardwaarde is `"600000"` , zodat Dispatcher 10 minuten wacht. Met de instelling `"0"` wordt de time-out verwijderd.

Wanneer de time-out wordt bereikt tijdens het parseren van responsheaders, wordt een HTTP-status van 504 (Bad Gateway) geretourneerd. Als de time-out wordt bereikt terwijl de hoofdtekst van de reactie wordt gelezen, retourneert de Dispatcher de onvolledige reactie op de client. Het schrapt ook om het even welke caching dossiers die zouden kunnen zijn geschreven.

**/ipv4**

Geeft aan of Dispatcher de functie `getaddrinfo` (voor IPv6) of de functie `gethostbyname` (voor IPv4) gebruikt om het IP-adres van de rendering te verkrijgen. Bij de waarde 0 wordt `getaddrinfo` gebruikt. Bij de waarde `1` wordt `gethostbyname` gebruikt. De standaardwaarde is `0` .

De functie `getaddrinfo` retourneert een lijst met IP-adressen. Dispatcher herhaalt de lijst van adressen tot het een verbinding TCP/IP vestigt. Daarom is het `ipv4` bezit belangrijk wanneer teruggeven hostname met veelvoudige IP adressen wordt geassocieerd. En als reactie op de functie `getaddrinfo` retourneert de host een lijst met IP-adressen die altijd in dezelfde volgorde staan. In dit geval moet u de functie `gethostbyname` gebruiken, zodat het IP-adres waarmee Dispatcher verbinding maakt, willekeurig wordt toegewezen.

Amazon Elastic Load Balancing (ELB) is een service die reageert op getaddrinfo met een lijst met IP-adressen die mogelijk dezelfde volgorde heeft.

**/secure**

Als de eigenschap `/secure` de waarde `"1"` heeft, gebruikt Dispatcher HTTPS om te communiceren met de instantie AEM. Voor meer details, zie [ Vormend Dispatcher om SSL ](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl) te gebruiken.

**/always-resolve**

Met versie van Dispatcher **4.1.6**, kunt u het `/always-resolve` bezit als volgt vormen:

* Wanneer ingesteld op `"1"` , wordt de host-name voor elk verzoek omgezet (de Dispatcher plaatst nooit een IP-adres in cache). Er kan een lichte prestatiesinvloed toe te schrijven zijn aan de extra vraag die wordt vereist om de gastheerinformatie voor elk verzoek te krijgen.
* Als het bezit niet wordt geplaatst, wordt het IP adres in het voorgeheugen ondergebracht door gebrek.

Ook, kan dit bezit worden gebruikt voor het geval u in dynamische IP resolutiekwesties loopt, zoals aangetoond in de volgende steekproef:

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## Toegang tot inhoud configureren {#configuring-access-to-content-filter}

Gebruik de sectie `/filter` om de HTTP-aanvragen op te geven die Dispatcher accepteert. Alle andere aanvragen worden teruggestuurd naar de webserver met een foutcode van 404 (pagina niet gevonden). Als er geen sectie `/filter` bestaat, worden alle aanvragen geaccepteerd.

**Nota:** De verzoeken om [ statfile ](#naming-the-statfile) worden altijd verworpen.

>[!CAUTION]
>
>Zie [ Controlelijst van de Veiligheid van Dispatcher ](security-checklist.md) voor verdere overwegingen wanneer het beperken van toegang gebruikend AEM Dispatcher. Ook, lees de [ Controlelijst van de Veiligheid van AEM ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist#security) voor extra veiligheidsdetails betreffende uw installatie van AEM.

De sectie `/filter` bestaat uit een reeks regels die of toegang tot inhoud volgens patronen in het verzoek-lijn deel van het HTTP- verzoek ontkennen of toestaan. Gebruik een strategie voor de lijst van gewenste personen van uw `/filter` sectie:

* Eerst, ontken toegang tot alles.
* Toegang tot inhoud toestaan als dat nodig is.

>[!NOTE]
>
>Wis de cache wanneer de filterregels worden gewijzigd.

### Een filter definiëren {#defining-a-filter}

Elk item in de sectie `/filter` bevat een type en een patroon die overeenkomen met een specifiek element van de aanvraagregel of de gehele aanvraagregel. Elk filter kan de volgende items bevatten:

* **Type**: `/type` wijst erop of om toegang voor de verzoeken toe te staan of te ontkennen die het patroon aanpassen. De waarde kan `allow` of `deny` zijn.

* **Element van de Regel van het Verzoek:** omvat `/method`, `/url`, `/query`, of `/protocol`. Neem ook een patroon op voor het filteren van aanvragen. Filter ze op basis van specifieke delen van het request-line deel in de HTTP-aanvraag. Filteren op elementen van de aanvraaglijn (eerder dan op de volledige verzoeklijn) is de aangewezen filtermethode.

* **Geavanceerde Elementen van de Lijn van het Verzoek:** Beginnend met Dispatcher 4.2.0, zijn vier nieuwe filterelementen beschikbaar voor gebruik. Deze nieuwe elementen zijn respectievelijk `/path` , `/selectors` , `/extension` en `/suffix` . Neem een of meer van deze items op om de URL-patronen verder te beheren.

>[!NOTE]
>
>Voor meer informatie over welk deel van de verzoeklijn elk van deze elementenverwijzingen, zie [ het Schuiven van de Decomposition URL ](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki- pagina.

* **glob Bezit**: Het `/glob` bezit wordt gebruikt om met de volledige verzoek-lijn van het verzoek van HTTP aan te passen.

>[!CAUTION]
>
>Filteren met globs is afgekeurd in Dispatcher. Als zodanig moet u globaal gebruik in de `/filter` -secties vermijden, omdat dit tot beveiligingsproblemen kan leiden. Dus in plaats van:
>
>`/glob "* *.css *"`
>
>Gebruiken
>
>`/url "*.css"`

#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 bepaalt [ verzoek-lijn ](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) als volgt:

`Method Request-URI HTTP-Version<CRLF>`

De `<CRLF>` -tekens vertegenwoordigen een harde return, gevolgd door een nieuwe regel. Het volgende voorbeeld is verzoek-lijn die wordt ontvangen wanneer een cliënt om de V.S.-Engelse pagina van de plaats WKND verzoekt:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

Uw patronen moeten de spatietekens in de request-line en de `<CRLF>` tekens bevatten.

#### Dubbele aanhalingstekens versus enkele aanhalingstekens {#double-quotes-vs-single-quotes}

Wanneer u filterregels maakt, gebruikt u dubbele aanhalingstekens `"pattern"` voor eenvoudige patronen. Als u Dispatcher 4.2.0 of hoger gebruikt en uw patroon een reguliere expressie bevat, moet u het regex-patroon `'(pattern1|pattern2)'` tussen enkele aanhalingstekens plaatsen.

#### Reguliere expressies {#regular-expressions}

In Dispatcher-versies hoger dan versie 4.2.0 kunt u POSIX Extended Regular Expressions opnemen in uw filterpatronen.

#### Problemen met filters oplossen {#troubleshooting-filters}

Als uw filters niet in de manier teweegbrengen u zou verwachten, laat [ het Registreren van het Spoor ](#trace-logging) op Dispatcher toe zodat kunt u zien welke filter het verzoek onderschept.

#### Voorbeeldfilter: Alles weigeren {#example-filter-deny-all}

In de volgende voorbeeldfiltersectie weigert de Dispatcher aanvragen voor alle bestanden. Ontken toegang tot alle bestanden en geef vervolgens toegang tot specifieke gebieden.

```xml
/0001  { /type "deny" /url "*"  }
```

Verzoeken naar een expliciet geweigerd gebied hebben tot gevolg dat een foutcode van 404 (pagina niet gevonden) wordt geretourneerd.

#### Voorbeeldfilter: toegang tot specifieke gebieden weigeren {#example-filter-deny-access-to-specific-areas}

Met filters kunt u ook toegang tot verschillende elementen weigeren, zoals ASP-pagina&#39;s en gevoelige gebieden in een publicatie-instantie. Met het volgende filter krijgt u geen toegang tot ASP-pagina&#39;s:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Voorbeeldfilter: POST-verzoeken inschakelen {#example-filter-enable-post-requests}

Met het volgende voorbeeldfilter kunt u formuliergegevens verzenden via de POST-methode:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Voorbeeldfilter: Toegang tot de workflowconsole toestaan {#example-filter-allow-access-to-the-workflow-console}

In het volgende voorbeeld wordt een filter getoond dat wordt gebruikt om externe toegang tot de workflowconsole toe te staan:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

Als uw publicatie-instantie gebruikmaakt van een webtoepassingscontext (bijvoorbeeld publiceren), kan deze ook aan uw filterdefinitie worden toegevoegd.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Als u toegang moet hebben tot enkele pagina&#39;s binnen het beperkte gebied, kunt u deze toegankelijk maken. Als u bijvoorbeeld toegang wilt verlenen tot het tabblad Archief in de Workflowconsole, voegt u de volgende sectie toe:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Wanneer meerdere filterpatronen van toepassing zijn op een aanvraag, is het laatst toegepaste filterpatroon effectief.

#### Voorbeeld, filter: Reguliere expressies gebruiken {#example-filter-using-regular-expressions}

Met dit filter schakelt u extensies in mappen met niet-openbare inhoud in met behulp van een reguliere expressie, die hier tussen enkele aanhalingstekens wordt gedefinieerd:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Voorbeeld, filter: extra elementen van een aanvraag-URL filteren {#example-filter-filter-additional-elements-of-a-request-url}

Hieronder ziet u een voorbeeld van een regel die het ophalen van inhoud van het pad `/content` en de substructuur ervan blokkeert met behulp van filters voor paden, kiezers en extensies:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### Voorbeeld `/filter` -sectie {#example-filter-section}

Wanneer het vormen van de Dispatcher, beperking externe toegang zoveel mogelijk. In het volgende voorbeeld wordt minimale toegang geboden aan externe bezoekers:

* `/content`
* diverse inhoud, zoals ontwerpen en clientbibliotheken. Bijvoorbeeld:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Nadat u filters creeert, [ de toegang van de testpagina ](#testing-dispatcher-security) om ervoor te zorgen dat uw instantie van AEM veilig is.

De volgende `/filter` sectie van het `dispatcher.any` dossier kan als basis in uw [ configuratiedossier van Dispatcher worden gebruikt.](#dispatcher-configuration-files)

Dit voorbeeld is gebaseerd op het standaardconfiguratiedossier dat van Dispatcher wordt voorzien en als voorbeeld voor gebruik in een productiemilieu bedoeld is. Items die met `#` zijn voorafgegaan, worden gedeactiveerd (gemarkeerd met opmerkingen). Wees voorzichtig als u besluit een van deze items te activeren (door de `#` op die regel te verwijderen). Dit kan gevolgen hebben voor de beveiliging.

Ontken toegang tot alles en geef vervolgens toegang tot specifieke (beperkte) elementen:

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001  { /type "deny" /url "*"  }

      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console

      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only

#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>Wanneer u Apache gebruikt, ontwerpt u uw filter-URL-patronen volgens de eigenschap DispatcherUseProcessURL van de Dispatcher-module. (Zie {de Server van het Web van 0} Apache - vorm uw Server van het Web Apache voor Dispatcher [&#128279;](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

Overweeg de volgende aanbevelingen als u verkiest om toegang uit te breiden:

* Schakel externe toegang tot `/admin` uit als u CQ-versie 5.4 of een eerdere versie gebruikt.

* Let op wanneer u toegang tot bestanden toestaat in `/libs` . Toegang moet op individuele basis worden toegestaan.
* Ontken toegang tot de replicatieconfiguratie zodat kan het niet worden gezien:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Toegang tot de Google Gadgets reverse-proxy weigeren:

   * `/libs/opensocial/proxy*`

Afhankelijk van de installatie kunnen er meer bronnen onder `/libs` , `/apps` of elders zijn die beschikbaar moeten worden gemaakt. U kunt het `access.log` -bestand gebruiken als een methode om te bepalen welke bronnen extern worden benaderd.

>[!CAUTION]
>
>De toegang tot consoles en folders kan een veiligheidsrisico voor productiemilieu&#39;s vormen. Tenzij u een expliciete motivering hebt, moeten deze worden gedeactiveerd (gemarkeerd als commentaar).

>[!CAUTION]
>
>Als u [ gebruikend rapporten in publiceert milieu ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/operations/reporting#using-reports-in-a-publish-environment) bent, zou u Dispatcher moeten vormen om toegang tot `/etc/reports` voor externe bezoekers te ontkennen.

### Query-tekenreeksen beperken {#restricting-query-strings}

Sinds Dispatcher versie 4.1.5 kunt u de sectie `/filter` gebruiken om querytekenreeksen te beperken. U wordt aangeraden querytekenreeksen expliciet toe te staan en generieke uitzonderingen via `allow` -filterelementen uit te sluiten.

Een enkel item kan `glob` of een combinatie van `method` , `url` , `query` en `version` hebben, maar niet beide. In het volgende voorbeeld wordt de querytekenreeks `a=*` toegestaan en worden alle andere querytekenreeksen voor URL&#39;s die worden omgezet in het knooppunt `/etc` geweigerd:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Als een regel een instructie `/query` bevat, komt deze alleen overeen met aanvragen die een queryreeks bevatten en met het opgegeven querypatroon.
>
>In het bovenstaande voorbeeld geldt dat als aanvragen aan `/etc` die geen queryreeks hebben ook zijn toegestaan, de volgende regels vereist zijn:
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Dispatcher Security testen {#testing-dispatcher-security}

Dispatcher-filters moeten de toegang tot de volgende pagina&#39;s en scripts in AEM-publicatieexemplaren blokkeren. Gebruik een webbrowser om te proberen de volgende pagina&#39;s te openen zoals een bezoeker van de site zou doen en om te controleren of code 404 wordt geretourneerd. Pas de filters aan als er andere resultaten worden verkregen.

U moet de normale rendering van pagina&#39;s zien voor `/content/add_valid_page.html?debug=layout` .

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

Om te bepalen of anonieme schrijf toegang wordt toegelaten, geef het volgende bevel in een terminal of bevelherinnering uit. Het schrijven van gegevens naar het knooppunt moet niet mogelijk zijn.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Als u wilt proberen de Dispatcher-cache ongeldig te maken en ervoor wilt zorgen dat u een code 403-reactie ontvangt, geeft u de volgende opdracht weer in een terminal of opdrachtprompt:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Toegang tot URL&#39;s met Vanity inschakelen {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Configureer de Dispatcher om toegang in te schakelen tot URL&#39;s met ijdelheid die zijn geconfigureerd voor uw AEM-pagina&#39;s.

Wanneer toegang tot vanity URLs wordt toegelaten, roept Dispatcher periodiek de dienst die op de teruggeeft instantie loopt om een lijst van vanity URLs te verkrijgen. Dispatcher slaat deze lijst op in een lokaal bestand. Wanneer een aanvraag voor een pagina wordt afgewezen vanwege een filter in de `/filter` -sectie, raadpleegt Dispatcher de lijst met URL&#39;s met een ijdelheid. Als de ontkende URL in de lijst staat, verleent Dispatcher toegang tot de vanity URL.

Als u toegang tot URL&#39;s met een ijdelheid wilt inschakelen, voegt u een sectie `/vanity_urls` toe aan de sectie `/farms` , net als in het volgende voorbeeld:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

De sectie `/vanity_urls` bevat de volgende eigenschappen:

* `/url`: Het pad naar de service vanity URL die wordt uitgevoerd op de renderinstantie. De waarde van deze eigenschap moet `"/libs/granite/dispatcher/content/vanityUrls.html"` zijn.

* `/file`: Het pad naar het lokale bestand waar Dispatcher de lijst met URL&#39;s van het type vanity opslaat. Zorg ervoor dat de Dispatcher schrijftoegang heeft tot dit bestand.
* `/delay`: (Seconden) De tijd tussen vraag aan de dienst van vanity URL.

>[!NOTE]
>
>Als uw teruggeven een geval van AEM is, moet u het [ VanityURLS-Components- pakket van de Distributie van de Software ](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) installeren om de dienst van vanityURL toe te laten. (Zie [ Distributie van de Software ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/contentmanagement/package-manager#software-distribution) voor meer details.)

Gebruik de volgende procedure om toegang tot vanity URLs toe te laten.

1. Als uw renderservice een AEM-instantie is, installeert u het `com.adobe.granite.dispatcher.vanityurl.content` -pakket op de publicatie-instantie (zie de opmerking hierboven).
1. Voor elke vanity-URL die u voor een AEM- of CQ-pagina hebt geconfigureerd, controleert u of de URL in de configuratie [`/filter`](#configuring-access-to-content-filter) wordt geweigerd. Voeg zo nodig een filter toe dat de URL weigert.
1. Voeg de sectie `/vanity_urls` onder `/farms` toe.
1. Start Apache-webserver opnieuw.

Met Dispatcher **versie 4.3.6** is een nieuwe `/loadOnStartup` parameter toegevoegd. Door deze parameter te gebruiken, kunt u het laden van vanity URLs bij opstarten als volgt vormen:

Door `/loadOnStartup 0` toe te voegen (zie het voorbeeld hieronder) kunt u het laden van vanity-URL&#39;s bij het opstarten uitschakelen.

```
/vanity_urls {
        /url "/libs/granite/dispatcher/content/vanityUrls.html"
        /file "/tmp/vanity_urls"
        /loadOnStartup 0
        /delay 60
      } 
```

Terwijl `/loadOnStartup 1` de vanity-URL&#39;s laadt bij het opstarten. Onthoud dat `/loadOnStartup 1` de huidige standaardwaarde voor deze parameter is.

## Verzoeken om synchronisatie verzenden - `/propagateSyndPost` {#forwarding-syndication-requests-propagatesyndpost}

Syndicatieverzoeken zijn alleen bedoeld voor Dispatcher, zodat ze standaard niet naar de renderer worden verzonden (bijvoorbeeld een AEM-instantie).

Stel, indien nodig, de eigenschap `/propagateSyndPost` in op `"1"` om synchronisatieverzoeken door te sturen naar Dispatcher. Indien ingesteld, moet u ervoor zorgen dat POST-aanvragen niet worden afgewezen in de filtersectie.

## Dispatcher Cache configureren - `/cache` {#configuring-the-dispatcher-cache-cache}

De sectie `/cache` bepaalt hoe Dispatcher documenten in cache plaatst. Vorm verscheidene subproperties om uw caching strategieën uit te voeren:

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

Een voorbeeldgeheim voorgeheugensectie zou als volgt kunnen kijken:

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"
  /allowAuthorized "0"

  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>Voor toestemming-gevoelig caching, lees [ In het voorgeheugen onderbrengend Beveiligde Inhoud ](permissions-cache.md).

### De cachemap opgeven {#specifying-the-cache-directory}

De eigenschap `/docroot` identificeert de map waarin bestanden in de cache worden opgeslagen.

>[!NOTE]
>
>De waarde moet hetzelfde pad hebben als de hoofdmap van het document op de webserver, zodat de Dispatcher en de webserver dezelfde bestanden verwerken.\
>De webserver is verantwoordelijk voor het leveren van de juiste statuscode wanneer het Dispatcher-cachebestand wordt gebruikt. Daarom is het belangrijk dat deze ook kan worden gevonden.

Als u veelvoudige landbouwbedrijven gebruikt, moet elk landbouwbedrijf een verschillende documentwortel gebruiken.

### De naam van het statusbestand wijzigen {#naming-the-statfile}

De eigenschap `/statfile` identificeert het bestand dat als statfile moet worden gebruikt. Dispatcher gebruikt dit bestand om de tijd van de meest recente inhoudsupdate te registreren. Het statusbestand kan elk bestand op de webserver zijn.

De status heeft geen inhoud. Wanneer de inhoud wordt bijgewerkt, werkt de Dispatcher de tijdstempel bij. Het standaardstatusbestand heeft de naam `.stat` en wordt opgeslagen in de hoofdmap van het document. Dispatcher blokkeert de toegang tot het statusbestand.

>[!NOTE]
>
>Als `/statfileslevel` is geconfigureerd, negeert Dispatcher de eigenschap `/statfile` en gebruikt `.stat` als naam.

### Stale documenten verzenden als er fouten optreden {#serving-stale-documents-when-errors-occur}

De eigenschap `/serveStaleOnError` bepaalt of Dispatcher ongeldig gemaakte documenten retourneert wanneer de renderserver een fout retourneert. Wanneer een statusbestand wordt aangeraakt en cacheinhoud ongeldig wordt gemaakt, verwijdert de Dispatcher de inhoud in de cache standaard. Deze actie wordt gedaan de volgende tijd het wordt gevraagd.

Als `/serveStaleOnError` is ingesteld op `"1"` , verwijdert Dispatcher geen ongeldig gemaakte inhoud uit de cache. Dat wil zeggen, tenzij de renderserver een geslaagde reactie retourneert. Een 502-, 503- of 504-reactie van AEM of een verbindingstijd zorgt ervoor dat de Dispatcher de verouderde inhoud aanbiedt en reageert met en HTTP-status 111 (Revalidation Failed).

### In cache plaatsen wanneer verificatie wordt gebruikt {#caching-when-authentication-is-used}

De eigenschap `/allowAuthorized` bepaalt of aanvragen die een van de volgende verificatiegegevens bevatten, in de cache worden geplaatst:

* De header `authorization`
* Een cookie met de naam `authorization`
* Een cookie met de naam `login-token`

Door gebrek, worden de verzoeken die deze authentificatieinformatie omvatten niet in het voorgeheugen ondergebracht omdat de authentificatie niet wordt uitgevoerd wanneer een caching document aan de cliënt is teruggekeerd. Deze configuratie voorkomt dat Dispatcher cachedocumenten kan verzenden aan gebruikers die niet de vereiste rechten hebben.

Als uw vereisten het in cache plaatsen van geverifieerde documenten echter toestaan, stelt u `/allowAuthorized` in op één:

`/allowAuthorized "1"`

>[!NOTE]
>
>Om sessiebeheer in te schakelen (met de eigenschap `/sessionmanagement` ), moet de eigenschap `/allowAuthorized` op `"0"` worden ingesteld.

### Documenten opgeven om in cache te plaatsen {#specifying-the-documents-to-cache}

De eigenschap `/rules` bepaalt welke documenten in de cache worden geplaatst op basis van het documentpad. Ongeacht de eigenschap `/rules` plaatst Dispatcher een document nooit in cache in de volgende omstandigheden:

* Verzoek-URI bevat een vraagteken (`?`).
   * Geeft een dynamische pagina aan, zoals een zoekresultaat dat niet in de cache hoeft te worden opgeslagen.
* De bestandsextensie ontbreekt.
   * De webserver heeft de extensie nodig om het documenttype (het MIME-type) te bepalen.
* De verificatieheader wordt ingesteld (configureerbaar).
* Als de AEM-instantie reageert met de volgende headers:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>De methoden GET of HEAD (voor de HTTP-header) kunnen door de Dispatcher in cache worden geplaatst. Voor extra informatie over reactiekopbal caching, zie de [ Caching sectie van de Kopballen van de Reactie van HTTP ](#caching-http-response-headers).

Elk item in de eigenschap `/rules` bevat een [`glob`](#designing-patterns-for-glob-properties) -patroon en een type:

* Het patroon `glob` wordt gebruikt om het pad van het document te laten overeenkomen.
* Het type geeft aan of de documenten die overeenkomen met het `glob` -patroon in de cache moeten worden geplaatst. De waarde kan `allow` (het document in cache plaatsen) of `deny` (het document renderen) zijn.

Als u geen dynamische pagina&#39;s hebt (naast die pagina&#39;s die reeds door de bovengenoemde regels worden uitgesloten), kunt u Dispatcher vormen om alles in het voorgeheugen onder te brengen. De sectie Regels ziet er als volgt uit:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Voor informatie over eigenschappen van Glob, zie [ het Ontwerpen van Patronen voor Eigenschappen van Glob ](#designing-patterns-for-glob-properties).

Als er gedeelten van de pagina dynamisch zijn (bijvoorbeeld een nieuwstoepassing) of zich in een gesloten gebruikersgroep bevinden, kunt u uitzonderingen definiëren:

>[!NOTE]
>
>Sluit gebruikersgroepen niet in cache plaatsen. De reden hiervoor is dat gebruikersrechten niet zijn gecontroleerd op pagina&#39;s in de cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Compressie**

Op Apache-webservers kunt u de documenten in de cache comprimeren. Met compressie kan Apache het document op verzoek van de client in een gecomprimeerd formulier retourneren. Compressie wordt automatisch uitgevoerd door de Apache-module `mod_deflate` in te schakelen, bijvoorbeeld:

```xml
AddOutputFilterByType DEFLATE text/plain
```

De module wordt standaard geïnstalleerd met Apache 2.x.

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### Bestanden op mapniveau ongeldig maken {#invalidating-files-by-folder-level}

Gebruik de eigenschap `/statfileslevel` om in cache opgeslagen bestanden ongeldig te maken op basis van het pad:

* Dispatcher leidt `.stat` tot dossiers in elke omslag van de documentwortelomslag aan het niveau dat u specificeert. De documenthoofdmap is niveau 0.
* Bestanden worden ongeldig gemaakt door het `.stat` -bestand aan te raken. De laatste wijzigingsdatum van het bestand `.stat` wordt vergeleken met de laatste wijzigingsdatum van een document in de cache. Het document wordt opnieuw ingesteld als het `.stat` -bestand nieuwer is.

* Wanneer een dossier op een bepaald niveau ongeldig wordt gemaakt, **alle** `.stat` dossiers van docroot **aan** het niveau van het ongeldig gemaakte dossier of gevormde `statsfilevel` (welke kleiner is) worden geraakt.

   * Als u bijvoorbeeld de eigenschap `statfileslevel` instelt op 6 en een bestand op niveau 5 ongeldig wordt gemaakt, wordt elk `.stat` -bestand van docroot naar 5 aangeraakt. Als u doorgaat met dit voorbeeld en een bestand op niveau 7 ongeldig wordt gemaakt, wordt elk `stat` -bestand van docroot naar zes aangeraakt (sinds `/statfileslevel = "6"` ).

Slechts worden de middelen **langs de weg** aan het ongeldig gemaakte dossier beïnvloed. Bekijk het volgende voorbeeld: een website gebruikt de structuur `/content/myWebsite/xx/.` Als u `statfileslevel` instelt als 3, wordt een bestand a `.stat` als volgt gemaakt:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Wanneer een bestand in `/content/myWebsite/xx` ongeldig wordt gemaakt, wordt elk `.stat` -bestand van docroot tot `/content/myWebsite/xx` aangeraakt. Dit scenario is alleen van toepassing voor `/content/myWebsite/xx` en niet voor bijvoorbeeld `/content/myWebsite/yy` of `/content/anotherWebSite` .

>[!NOTE]
>
>Ongeldige validatie kan worden voorkomen door een extra koptekst `CQ-Action-Scope:ResourceOnly` te verzenden. Deze methode kan worden gebruikt om bepaalde middelen te spoelen zonder andere delen van het geheime voorgeheugen ongeldig te maken. Zie [ deze pagina ](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) en [ manueel het Valideren van het Geheime voorgeheugen van Dispatcher ](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/page-invalidate#configuring) voor extra details.

>[!NOTE]
>
>Wanneer u een waarde opgeeft voor de eigenschap `/statfileslevel` , wordt de eigenschap `/statfile` genegeerd.

### Automatisch cachebestanden valideren {#automatically-invalidating-cached-files}

De eigenschap `/invalidate` definieert de documenten die automatisch ongeldig worden gemaakt wanneer de inhoud wordt bijgewerkt.

Met automatische ongeldigmaking verwijdert Dispatcher geen in het cachegeheugen opgeslagen bestanden nadat de inhoud is bijgewerkt, maar controleert het of deze geldig zijn wanneer ze de volgende keer worden aangevraagd. Documenten in de cache die niet automatisch ongeldig worden gemaakt, blijven in de cache totdat een inhoudsupdate deze expliciet verwijdert.

Automatische validatie wordt meestal gebruikt voor HTML-pagina&#39;s. HTML-pagina&#39;s bevatten vaak koppelingen naar andere pagina&#39;s, waardoor het moeilijk is om te bepalen of een update van de inhoud van invloed is op een pagina. Als u ervoor wilt zorgen dat alle relevante pagina&#39;s ongeldig worden gemaakt wanneer de inhoud wordt bijgewerkt, maakt u automatisch alle HTML-pagina&#39;s ongeldig. De volgende configuratie maakt alle HTML-pagina&#39;s ongeldig:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Voor informatie over eigenschappen van Glob, zie [ het Ontwerpen van Patronen voor Eigenschappen van Glob ](#designing-patterns-for-glob-properties).

Deze configuratie veroorzaakt de volgende activiteit wanneer `/content/wknd/us/en` wordt geactiveerd:

* Alle bestanden met patroon en.* worden verwijderd uit de map `/content/wknd/us` .
* De map `/content/wknd/us/en./_jcr_content` wordt verwijderd.
* Alle andere bestanden die overeenkomen met de `/invalidate` -configuratie worden niet onmiddellijk verwijderd. Deze bestanden worden verwijderd wanneer de volgende aanvraag wordt uitgevoerd. In het voorbeeld wordt `/content/wknd.html` niet verwijderd, maar verwijderd wanneer `/content/wknd.html` wordt aangevraagd.

Als u automatisch gegenereerde PDF- en ZIP-bestanden aanbiedt om te downloaden, moet u deze bestanden mogelijk ook automatisch ongeldig maken. Een configuratievoorbeeld ziet er als volgt uit:

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

De AEM-integratie met Adobe Analytics levert configuratiegegevens in een `analytics.sitecatalyst.js` -bestand op uw website. Het voorbeeldbestand `dispatcher.any` dat bij Dispatcher wordt geleverd, bevat de volgende regel voor het ongeldig maken van dit bestand:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Aangepaste validatiescripts gebruiken {#using-custom-invalidation-scripts}

Met de eigenschap `/invalidateHandler` kunt u een script definiëren dat wordt aangeroepen voor elk verzoek tot validatie dat door Dispatcher wordt ontvangen.

De methode wordt aangeroepen met de volgende argumenten:

* Handgreep - Het ongeldig gemaakte inhoudspad
* Handeling - De replicatiehandeling (bijvoorbeeld Activeren, Deactiveren)
* Het Werkingsgebied van de actie - het Werkgebied van de replicatieactie (leeg, tenzij een kopbal van `CQ-Action-Scope: ResourceOnly` wordt verzonden, zie [ het Invalideren van Cached Pagina&#39;s van AEM ](page-invalidate.md) voor details)

Deze methode kan worden gebruikt voor verschillende gebruiksgevallen. Bijvoorbeeld het ongeldig maken van andere toepassingsspecifieke geheime voorgeheugens, of het behandelen van gevallen waar extern URL van een pagina, en zijn plaats in het docroot, niet de inhoudspad aanpast.

In het volgende voorbeeldscript worden alle ongeldig gemaakte aanvragen naar een bestand genoteerd.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### Voorbeeldscript voor validatiehandlers {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### De clients beperken die de cache kunnen leegmaken {#limiting-the-clients-that-can-flush-the-cache}

De eigenschap `/allowedClients` definieert specifieke clients die de cache mogen leegmaken. De globbende patronen worden aangepast aan IP.

In het volgende voorbeeld:

1. toegang tot elke client wordt geweigerd
1. verleent uitdrukkelijk toegang tot localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Voor informatie over eigenschappen van Glob, zie [ het Ontwerpen van Patronen voor Eigenschappen van Glob ](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>U wordt aangeraden de `/allowedClients` te definiëren.
>
>Als het niet wordt gedaan, kan om het even welke cliënt een vraag uitgeven om het geheime voorgeheugen te ontruimen. Als u dit herhaaldelijk doet, kan dit ernstige gevolgen hebben voor de prestaties van de site.

### URL-parameters worden genegeerd {#ignoring-url-parameters}

In de sectie `ignoreUrlParams` wordt gedefinieerd welke URL-parameters worden genegeerd wanneer wordt bepaald of een pagina in de cache wordt opgeslagen of via de cache wordt geleverd:

* Wanneer een aanvraag-URL parameters bevat die allemaal worden genegeerd, wordt de pagina in de cache geplaatst.
* Wanneer een aanvraag-URL een of meer parameters bevat die niet worden genegeerd, wordt de pagina niet in de cache opgeslagen.

Wanneer een parameter voor een pagina wordt genegeerd, wordt de pagina in de cache geplaatst de eerste keer dat de pagina wordt aangevraagd. Volgende aanvragen voor de pagina worden naar de pagina in de cache verzonden, ongeacht de waarde van de parameter in het verzoek.

>[!NOTE]
>
>U wordt aangeraden de instelling `ignoreUrlParams` op een manier van lijsten van gewenste personen te configureren. Als dusdanig, worden alle vraagparameters genegeerd en slechts worden bekende of verwachte vraagparameters vrijgesteld (&quot;ontkend&quot;) van wordt genegeerd. Voor meer details en voorbeelden, zie [ deze pagina ](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner).

Als u wilt opgeven welke parameters worden genegeerd, voegt u glob-regels toe aan de eigenschap `ignoreUrlParams` :

* Als u een pagina in het cachegeheugen wilt plaatsen, ongeacht de aanvraag die een URL-parameter bevat, maakt u een glob-eigenschap waarmee de parameter kan worden genegeerd.
* Als u wilt voorkomen dat de pagina in de cache wordt opgeslagen, maakt u een glob-eigenschap die de parameter weigert (te negeren).

>[!NOTE]
>
>Wanneer het vormen van het glob bezit, zou het de naam van de vraagparameter moeten aanpassen. Als u bijvoorbeeld de parameter &quot;p1&quot; van de volgende URL wilt negeren `http://example.com/path/test.html?p1=test&p2=v2` , moet de eigenschap glob als volgt zijn:
> `/0002 { /glob "p1" /type "allow" }`

In het volgende voorbeeld negeert Dispatcher alle parameters, behalve de parameter `nocache` . Als zodanig vraagt Dispatcher nooit URL&#39;s die de parameter `nocache` bevatten, in cache op:

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

In de context van het bovenstaande configuratievoorbeeld `ignoreUrlParams` zorgt de volgende HTTP-aanvraag ervoor dat de pagina in de cache wordt geplaatst omdat de parameter `willbecached` wordt genegeerd:

```xml
GET /mypage.html?willbecached=true
```

In de context van het `ignoreUrlParams` configuratievoorbeeld, veroorzaakt het volgende HTTP- verzoek de pagina **niet** om in het voorgeheugen worden opgeslagen omdat de `nocache` parameter niet wordt genegeerd:

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

Voor informatie over eigenschappen van Glob, zie [ het Ontwerpen van Patronen voor Eigenschappen van Glob ](#designing-patterns-for-glob-properties).

### HTTP-responsheaders in cache plaatsen {#caching-http-response-headers}

>[!NOTE]
>
>Deze eigenschap is beschikbaar met versie **4.1.11** van Dispatcher.

Met de eigenschap `/headers` kunt u de HTTP-headertypen definiëren die Dispatcher in de cache gaat plaatsen. Op het eerste verzoek aan een middel uncached, worden alle kopballen die één van de gevormde waarden (zie de configuratiemonster hieronder) aanpassen opgeslagen in een afzonderlijk dossier, naast het geheim voorgeheugendossier. Bij verdere verzoeken aan het caching middel, worden de opgeslagen kopballen toegevoegd aan de reactie.

Hieronder ziet u een voorbeeld van de standaardconfiguratie:

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>Tekens voor het samenvoegen van bestanden zijn niet toegestaan. Voor meer details, zie [ het Ontwerpen Patronen voor Eigenschappen van Glob ](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Als u de Dispatcher nodig hebt om de eBay-antwoordheaders van AEM op te slaan en te leveren, doet u het volgende:
>
>* Voeg de kopbalnaam in de `/cache/headers` sectie toe.
>* Voeg de volgende [ richtlijn Apache ](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) in de op Dispatcher betrekking hebbende sectie toe:
>
>```xml
>FileETag none
>```

### Dispatcher Cache File Machtigingen {#dispatcher-cache-file-permissions}

De eigenschap `mode` geeft aan welke bestandsmachtigingen worden toegepast op nieuwe mappen en bestanden in de cache. De `umask` van het aanroepingsproces beperkt deze instelling. Dit is een octaal getal dat wordt samengesteld uit de som van een of meer van de volgende waarden:

* `0400` Lezen door eigenaar toestaan.
* `0200` Schrijven door eigenaar toestaan.
* `0100` Laat de eigenaar zoeken in mappen.
* `0040` Lezen door groepsleden toestaan.
* `0020` Schrijven door groepsleden toestaan.
* `0010` Groepsleden mogen in de map zoeken.
* `0004` Lezen door anderen toestaan.
* `0002` Schrijven door anderen toestaan.
* `0001` Anderen mogen in de map zoeken.

De standaardwaarde is `0755` . Hiermee kan de eigenaar lezen, schrijven of zoeken en de groep en anderen lezen of zoeken.

### Startbestand met Throttling .stat aanraken {#throttling-stat-file-touching}

Met de standaardeigenschap `/invalidate` maakt elke activering alle `.html` -bestanden ongeldig (wanneer het pad ervan overeenkomt met de sectie `/invalidate` ). Op een website met aanzienlijk verkeer verhogen meerdere, daaropvolgende activeringen de CPU-belasting op de achtergrond. In een dergelijk scenario is het wenselijk om het tikken van `.stat` -bestanden te &quot;vertragen&quot; om de website responsief te houden. U kunt deze handeling uitvoeren met de eigenschap `/gracePeriod` .

De eigenschap `/gracePeriod` definieert het aantal seconden dat een niet-gevalideerde, niet-gevalideerde resource mogelijk nog steeds uit de cache wordt geladen na de laatste activering. De eigenschap kan worden gebruikt in een installatie waarbij een batch activeringen anders de gehele cache herhaaldelijk ongeldig zouden maken. De aanbevolen waarde is 2 seconden.

Voor meer details, zie `/invalidate` en `/statfileslevel` vroeger.

### Op tijd gebaseerde cachevalidatie configureren - `/enableTTL` {#configuring-time-based-cache-invalidation-enablettl}

Op tijd gebaseerde cachevalidatie is afhankelijk van de eigenschap `/enableTTL` en de aanwezigheid van normale verloopheaders van de HTTP-standaard. Als u het bezit aan 1 (`/enableTTL "1"`) plaatst, evalueert het de reactiekopballen van het achtereind. Als de kopteksten de datum `Cache-Control`, `max-age` of `Expires` bevatten, wordt een leeg hulpdossier naast het caching dossier gecreeerd, met de wijzigingstijd gelijk aan de vervaldatum. Wanneer het cachebestand na de wijzigingstijd wordt opgevraagd, wordt het automatisch opnieuw opgevraagd vanaf de achtergrond.

Vóór Dispatcher 4.3.5 was de logica voor de validatie van TTL alleen gebaseerd op de geconfigureerde TTL-waarde. Met Dispatcher 4.3.5, zowel worden de reeksTTL **als** de regels van de het geheim voorgeheugenongeldigheid van Dispatcher rekenschap gegeven. Voor een bestand in de cache:

1. Als `/enableTTL` is ingesteld op 1, wordt gecontroleerd of het bestand vervalt. Als het bestand volgens de set-TTL is verlopen, worden geen andere controles uitgevoerd en wordt het bestand in de cache opnieuw opgevraagd vanaf de back-end.
2. Als het bestand niet is verlopen of als `/enableTTL` niet is geconfigureerd, worden de standaard regels voor cachevalidatie toegepast, zoals de regels die [`/statfileslevel`](#invalidating-files-by-folder-level) en [`/invalidate`](#automatically-invalidating-cached-files) hebben ingesteld. Deze stroom betekent dat de Dispatcher bestanden waarvoor de TTL niet is verlopen, ongeldig kan maken.

Deze nieuwe implementatie steunt gebruiksgevallen waar de dossiers langere TTL (bijvoorbeeld, op CDN) hebben. Maar dat dossier kan nog ongeldig worden verklaard zelfs als TTL niet is verlopen. Het begunstigt de versheid van de inhoud ten opzichte van de cache-hit verhouding op de Dispatcher.

Omgekeerd, voor het geval u **slechts** nodig hebt wordt de vervallogica toegepast op een dossier dan geplaatst `/enableTTL` aan 1 en sluit dat dossier van het standaardmechanisme van de geheim voorgeheugenongeldigverklaring uit. U kunt bijvoorbeeld:

* Om het dossier te negeren, vorm de [ bevestigingsregels ](#automatically-invalidating-cached-files) in de geheim voorgeheugensectie. In het onderstaande fragment worden alle bestanden die eindigen in `.example.html` alleen genegeerd en verlopen wanneer de ingestelde TTL is doorgegeven.

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* Ontwerp de inhoudsstructuur zodanig dat u een hoge waarde [`/statfilelevel`](#invalidating-files-by-folder-level) kunt instellen zodat het bestand niet automatisch ongeldig wordt gemaakt.

Zo zorgt u ervoor dat `.stat` -bestandsvalidatie niet wordt gebruikt en dat alleen de vervaldatum van TTL actief is voor de opgegeven bestanden.

>[!NOTE]
>
>Houd er rekening mee dat als u `/enableTTL` instelt op 1, TTL alleen in cache wordt geplaatst aan de Dispatcher-zijde. Als zodanig wordt de TTL-informatie in het aanvullende bestand (zie hierboven) niet verstrekt aan een andere gebruikersagent die een dergelijk bestandstype van de Dispatcher aanvraagt. Als u caching kopballen aan stroomafwaartse systemen zoals CDN of browser wilt verstrekken, zou u de `/cache/headers` sectie dienovereenkomstig moeten vormen.

>[!NOTE]
>
>Deze eigenschap is beschikbaar in versie **4.1.11** of recenter van Dispatcher.

## Load Balancing configureren - `/statistics` {#configuring-load-balancing-statistics}

In de sectie `/statistics` worden categorieën bestanden gedefinieerd waarvoor Dispatcher de reactiesnelheid van elke rendermethode scant. Dispatcher gebruikt de scores om te bepalen welke renderen om een aanvraag te verzenden.

Elke categorie die u maakt, definieert een globaal patroon. Dispatcher vergelijkt de URI van de aangevraagde inhoud met deze patronen om de categorie van de gevraagde inhoud te bepalen:

* De volgorde van de categorieën bepaalt de volgorde waarin ze worden vergeleken met de URI.
* Het eerste categoriepatroon dat overeenkomt met de URI is de categorie van het bestand. Er worden niet meer categoriepatronen geëvalueerd.

Dispatcher ondersteunt maximaal acht statistische categorieën. Als u meer dan acht categorieën definieert, worden alleen de eerste 8 gebruikt.

**geeft Selectie** terug

Telkens wanneer de Dispatcher een weergegeven pagina vereist, gebruikt het de volgende algoritme om de rendering te selecteren:

1. Als de aanvraag de rendernaam in een `renderid` -cookie bevat, gebruikt Dispatcher die render.
1. Als de aanvraag geen `renderid` cookie bevat, vergelijkt Dispatcher de renderstatistieken:

   1. Dispatcher bepaalt de categorie van verzoek-URI.
   1. Dispatcher bepaalt welke rendermethode de laagste responsscore voor die categorie heeft en selecteert die rendermethode.

1. Als er nog geen renderbewerking is geselecteerd, gebruikt u de eerste renderbewerking in de lijst.

De score voor de rendercategorie is gebaseerd op vorige responstijden en eerdere mislukte en succesvolle verbindingen die Dispatcher probeert uit te voeren. Voor elke poging, wordt de score voor de categorie van gevraagde URI bijgewerkt.

>[!NOTE]
>
>Als u geen taakverdeling gebruikt, kunt u deze sectie weglaten.

### Categorieën statistieken definiëren {#defining-statistics-categories}

Definieer een categorie voor elk type document waarvoor u statistieken wilt bijhouden voor de renderselectie. De sectie `/statistics` bevat een sectie `/categories` . Als u een categorie wilt definiëren, voegt u onder de sectie `/categories` een regel met de volgende indeling toe:

`/name { /glob "pattern"}`

De categorie `name` moet uniek zijn voor de farm. `pattern` wordt beschreven in [ het Ontwerpen Patronen voor globale Eigenschappen ](#designing-patterns-for-glob-properties) sectie.

Om de categorie van URI te bepalen, vergelijkt Dispatcher URI met elk categoriepatroon tot een gelijke wordt gevonden. Dispatcher begint met de eerste categorie in de lijst en gaat door in de juiste volgorde. Plaats daarom eerst categorieën met specifiekere patronen.

In Dispatcher definieert het standaard `dispatcher.any` -bestand bijvoorbeeld een HTML-categorie en een andere categorie. De categorie HTML is specifieker en wordt dus eerst weergegeven:

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

In het volgende voorbeeld wordt ook een categorie voor zoekpagina&#39;s opgenomen:

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### Weerspiegelen van serveronbeschikbaarheid in Dispatcher-statistieken {#reflecting-server-unavailability-in-dispatcher-statistics}

De eigenschap `/unavailablePenalty` stelt de tijd (in tiende van een seconde) in die wordt toegepast op de renderstatistieken wanneer een verbinding met de renderbewerking mislukt. Dispatcher voegt de tijd toe aan de statistische categorie die overeenkomt met de aangevraagde URI.

Bijvoorbeeld, wordt de sanctie toegepast wanneer de verbinding TCP/IP aan aangewezen hostname/haven niet kan worden gevestigd. De reden is of omdat AEM niet wordt uitgevoerd (en niet luistert) of vanwege een netwerkgerelateerd probleem.

De eigenschap `/unavailablePenalty` is een rechtstreeks onderliggend element van de sectie `/farm` (een neveneffect van de sectie `/statistics` ).

Als er geen eigenschap `/unavailablePenalty` bestaat, wordt de waarde `"1"` gebruikt.

```xml
/unavailablePenalty "1"
```

## Identificeren van een gevoelige verbindingsmap - `/stickyConnectionsFor` {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

De eigenschap `/stickyConnectionsFor` definieert één map die plakke documenten bevat. Deze eigenschap wordt benaderd via de URL. Dispatcher verzendt alle aanvragen, van één gebruiker in deze map, naar dezelfde renderinstantie. De stevige verbindingen zorgen ervoor dat de zittingsgegevens voor alle documenten aanwezig en verenigbaar zijn. Dit mechanisme gebruikt het `renderid` cookie.

In het volgende voorbeeld wordt een kleverige verbinding met de map /products gedefinieerd:

```xml
/stickyConnectionsFor "/products"
```

Wanneer een pagina is samengesteld uit inhoud van verschillende inhoudsknooppunten, neemt u de eigenschap `/paths` op die de paden naar de inhoud opsomt. Een pagina bevat bijvoorbeeld inhoud van `/content/image` , `/content/video` en `/var/files/pdfs` . De volgende configuratie laat kleverige verbindingen voor alle inhoud op de pagina toe:

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### `httpOnly` {#httponly}

Wanneer kleverige verbindingen worden toegelaten, plaatst de module van Dispatcher het `renderid` koekje. Dit cookie heeft niet de markering `httponly` , die moet worden toegevoegd om de beveiliging te verbeteren. U voegt de markering `httponly` toe door de eigenschap `httpOnly` in het knooppunt `/stickyConnections` van een `dispatcher.any` -configuratiebestand in te stellen. De waarde van de eigenschap (ofwel `0` of `1` ) definieert of het cookie `renderid` het kenmerk `HttpOnly` heeft toegevoegd. De standaardwaarde is `0` , wat betekent dat het kenmerk niet wordt toegevoegd.

Voor extra informatie over de `httponly` vlag, lees [ deze pagina ](https://owasp.org/www-community/HttpOnly).

### `secure` {#secure}

Wanneer kleverige verbindingen worden toegelaten, plaatst de module van Dispatcher het `renderid` koekje. Dit cookie heeft niet de markering `secure` , die moet worden toegevoegd om de beveiliging te verbeteren. U voegt de markering `secure` toe die de eigenschap `secure` instelt in het knooppunt `/stickyConnections` van een `dispatcher.any` -configuratiebestand. De waarde van de eigenschap (ofwel `0` of `1` ) definieert of het cookie `renderid` het kenmerk `secure` heeft toegevoegd. De standaardwaarde is `0`, wat betekent wordt het attribuut toegevoegd **als** het inkomende verzoek veilig is. Als de waarde is ingesteld op `1` , wordt de beveiligde vlag toegevoegd, ongeacht of de binnenkomende aanvraag veilig is of niet.

## Renderfouten afhandelen {#handling-render-connection-errors}

Configureer Dispatcher-gedrag wanneer de renderserver een fout van 500 retourneert of niet beschikbaar is.

### Een pagina voor een health check opgeven {#specifying-a-health-check-page}

Gebruik de eigenschap `/health_check` om een URL op te geven die wordt gecontroleerd wanneer een 500-statuscode plaatsvindt. Als deze pagina ook een 500-statuscode retourneert, wordt de instantie als niet beschikbaar beschouwd en wordt een configureerbare tijd ( `/unavailablePenalty` ) toegepast op de rendering voordat u het opnieuw probeert.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### De vertraging voor opnieuw proberen van pagina opgeven {#specifying-the-page-retry-delay}

De eigenschap `/retryDelay` stelt de tijd (in seconden) in die Dispatcher wacht tussen ronde verbindingspogingen met de farm. Voor elke ronde is het maximumaantal keren dat de Dispatcher een verbinding probeert te maken met een renderbewerking het aantal renders in het bedrijf.

Dispatcher gebruikt de waarde `"1"` if `/retryDelay` niet expliciet gedefinieerd. De standaardwaarde is meestal juist.

```xml
/retryDelay "1"
```

### Het aantal pogingen configureren {#configuring-the-number-of-retries}

De eigenschap `/numberOfRetries` stelt het maximumaantal ronde verbindingspogingen in dat Dispatcher met de renderers uitvoert. Als Dispatcher geen verbinding kan maken met een renderbewerking na dit aantal keren, retourneert Dispatcher een mislukte reactie.

Voor elke ronde is het maximumaantal keren dat de Dispatcher een verbinding probeert te maken met een renderbewerking het aantal renders in het bedrijf. Daarom is het maximumaantal keren dat de Dispatcher een verbinding probeert, ( `/numberOfRetries`) x (het aantal renders).

Wanneer de waarde niet expliciet wordt gedefinieerd, is de standaardwaarde `5` .

```xml
/numberOfRetries "5"
```

### Het mechanisme Failover gebruiken {#using-the-failover-mechanism}

Om verzoeken aan verschillende terug te sturen geeft terug wanneer het originele verzoek ontbreekt, laat het failovermechanisme op uw landbouwbedrijf van Dispatcher toe. Wanneer failover wordt toegelaten, heeft Dispatcher het volgende gedrag:

* Wanneer een verzoek om terug te geven HTTP Status 503 (UNAVAILABLE) terugkeert, verzendt Dispatcher het verzoek naar verschillend teruggeven.
* Wanneer een aanvraag naar een rendermethode HTTP Status 50x (behalve 503) retourneert, verzendt Dispatcher een aanvraag naar de pagina die voor de eigenschap `health_check` is geconfigureerd.
   * Als de gezondheidscontrole 500 (INTERNAL_SERVER_ERROR) terugkeert, verzendt Dispatcher het originele verzoek naar verschillende teruggeven.
   * Als de health check HTTP Status 200 retourneert, retourneert de Dispatcher de eerste HTTP 500-fout aan de client.

Om failover toe te laten, voeg de volgende lijn aan het landbouwbedrijf (of de website) toe:

```xml
/failover "1"
```

>[!NOTE]
>
>Als u HTTP-aanvragen met een hoofdtekst opnieuw wilt proberen, stuurt Dispatcher een `Expect: 100-continue` aanvraagheader naar de renderbewerking voordat de werkelijke inhoud wordt gespoold. CQ 5.5 met CQSE beantwoordt dan onmiddellijk met of 100 (CONTINUE) of een foutencode. Ook andere servlet-containers worden ondersteund.

## Onderbrekingsfouten negeren - `/ignoreEINTR` {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Deze optie is niet nodig. Gebruik dit alleen wanneer u de volgende logberichten ziet:
>
>`Error while reading response: Interrupted system call`

Elke systeemaanroep kan worden onderbroken `EINTR` als het object van de systeemaanroep zich op een extern systeem bevindt dat via NFS wordt benaderd. Of deze systeemvraag uit kan tijd of worden onderbroken is gebaseerd op hoe het onderliggende dossiersysteem op de lokale machine werd opgezet.

Gebruik de parameter `/ignoreEINTR` als uw instantie een dergelijke configuratie heeft en het logboek het volgende bericht bevat:

`Error while reading response: Interrupted system call`

Intern leest Dispatcher de reactie van de externe server (AEM) met een lus die kan worden weergegeven als:

```text
while (response not finished) {  
read more data  
}
```

Dergelijke berichten kunnen worden gegenereerd wanneer de instructie `EINTR` voorkomt in de sectie `read more data` . De ontvangst van een signaal voordat er gegevens werden ontvangen, is de oorzaak.

Als u dergelijke onderbrekingen wilt negeren, voegt u de volgende parameter toe aan `dispatcher.any` (vóór `/farms` ):

`/ignoreEINTR "1"`

Als u `/ignoreEINTR` instelt op `"1"` , blijft Dispatcher proberen gegevens te lezen totdat het volledige antwoord wordt gelezen. De standaardwaarde is `0` en deactiveert de optie.

## Patronen ontwerpen voor Glob-eigenschappen {#designing-patterns-for-glob-properties}

In verschillende secties in het Dispatcher-configuratiebestand worden de eigenschappen `glob` gebruikt als selectiecriteria voor clientaanvragen. De waarden van `glob` -eigenschappen zijn patronen die Dispatcher vergelijkt met een aspect van de aanvraag, zoals het pad van de gevraagde bron of het IP-adres van de client. De items in de `/filter` -sectie gebruiken bijvoorbeeld `glob` -patronen om de paden van de pagina&#39;s te identificeren waarop Dispatcher reageert of weigert.

De `glob` -waarden kunnen jokertekens en alfanumerieke tekens bevatten om het patroon te definiëren.

| Jokerteken | Beschrijving | Voorbeelden |
|--- |--- |--- |
| `*` | Komt overeen met nul of meer aaneengesloten instanties van een willekeurig teken in de tekenreeks. Één van beide volgende situaties bepaalt het definitieve karakter van de gelijke: <br/> het karakter van A in het koord past het volgende karakter in het patroon aan, en het patroonkarakter heeft de volgende kenmerken:<br/><ul><li>Geen `*`</li><li>Geen `?`</li><li>Een letterlijk teken (inclusief een spatie) of een tekenklasse.</li><li>Het einde van het patroon is bereikt.</li></ul>Binnen een tekenklasse wordt het teken letterlijk geïnterpreteerd. | `*/geo*` Komt overeen met elke pagina onder het knooppunt `/content/geometrixx` en `/content/geometrixx-outdoors` . De volgende HTTP-aanvragen komen overeen met het glob-patroon: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/> komt om het even welke pagina onder de `/content/geometrixx-outdoors` knoop overeen. De volgende HTTP-aanvraag komt bijvoorbeeld overeen met het glob-patroon: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Komt overeen met elk willekeurig enkel teken. Gebruik externe tekenklassen. Binnen een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | `*outdoors/??/*`<br/> Komt overeen met de pagina&#39;s voor elke taal in de geometrixx-outdoorsite. De volgende HTTP-aanvraag komt bijvoorbeeld overeen met het glob-patroon: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> De volgende aanvraag komt niet overeen met het glob-patroon: <br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | Hiermee wordt het begin en einde van een tekenklasse gedemonstreerd. Tekenklassen kunnen een of meer tekenbereiken en enkele tekens bevatten.<br/> de gelijke van A komt voor als het doelkarakter om het even welke karakters in de karakterklasse, of binnen een bepaalde waaier aanpast.<br/> als de sluitende steun niet inbegrepen is, veroorzaakt het patroon geen gelijken. | `*[o]men.html*`<br/> Komt overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/> het past niet het volgende HTTP- verzoek aan:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/> Komt overeen met de volgende HTTP-aanvragen: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Er wordt een reeks tekens aangegeven. Voor gebruik in tekenklassen. Buiten een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | `*[m-p]men.html*` Komt overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Deze komt niet overeen met de volgende HTTP-aanvraag:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Hiermee wordt het volgende teken of de volgende tekenklasse genegeerd. Alleen gebruiken voor negerende tekens en tekenbereiken binnen tekenklassen. Gelijk aan de `^ wildcard` . <br/> buiten een karakterklasse, wordt dit karakter letterlijk geïnterpreteerd. | `*[ !o]men.html*`<br/> Komt overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> Deze komt niet overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[ !o!/]men.html*`<br/> Deze komt niet overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` of `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Hiermee wordt het volgende teken- of tekenbereik genegeerd. Alleen gebruiken voor negatie van tekens en tekenbereiken binnen tekenklassen. Komt overeen met het jokerteken `!` . <br/> buiten een karakterklasse, wordt dit karakter letterlijk geïnterpreteerd. | De voorbeelden van het jokerteken `!` zijn van toepassing, waarbij de `!` -tekens in de voorbeeldpatronen worden vervangen door `^` -tekens. |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## Logboekregistratie {#logging}

In de de serverconfiguratie van het Web, kunt u plaatsen:

* De locatie van het Dispatcher-logbestand.
* Het logniveau.

Raadpleeg de documentatie bij de webserver en het Lees mij-bestand van uw Dispatcher-instantie voor meer informatie.

**Apache Geroteerde of Door buizen geleid Logboeken**

Als het gebruiken van een **Apache** Webserver, kunt u de standaardfunctionaliteit voor de Omwentelingen van het Logboek, of Pijl Logboeken, of allebei gebruiken. Bijvoorbeeld, het gebruiken van Pijl Logboeken:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Deze functionaliteit roteert automatisch:

* het het logboekdossier van Dispatcher, met een timestamp in de uitbreiding (`logs/dispatcher.log%Y%m%d`).
* wekelijks (60 x 60 x 24 x 7 = 604800 seconden).

Zie de documentatie van de Server van het Web Apache op de Omwenteling van het Logboek en Pijl-Logboeken. Bijvoorbeeld, [ Apache 2.4 ](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Na installatie, is het standaardlogboekniveau hoog (namelijk niveau 3 = zuivert), zodat Dispatcher alle fouten en waarschuwingen registreert. Dit niveau is handig in de beginstadia.
>
>Een dergelijk niveau vereist echter meer middelen. Wanneer Dispatcher regelmatig *volgens uw vereisten* werkt, kunt u het logboekniveau verminderen.

### Trackregistratie {#trace-logging}

Naast andere verbeteringen voor de Dispatcher, introduceert versie 4.2.0 ook Tracks Logging.

Deze capaciteit is een hoger niveau dan Debug registreren die extra informatie in de logboeken toont. Er wordt logbestanden toegevoegd voor:

* De waarden van de doorgestuurde kopteksten;
* De regel die wordt toegepast voor een bepaalde handeling.

U kunt de Registratie van het Spoor toelaten door het logboekniveau aan `4` in uw Webserver te plaatsen.

Hieronder ziet u een voorbeeld van logboeken waarin overtrekken is ingeschakeld:

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

En een gebeurtenis wordt geregistreerd wanneer een dossier dat een blokkerende regel aanpast wordt gevraagd:

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## Basisbewerking bevestigen {#confirming-basic-operation}

U kunt de volgende stappen uitvoeren om de basiswerking en interactie van de webserver, de Dispatcher- en AEM-instantie te bevestigen:

1. Stel de waarde `loglevel` in op `3` .

1. Start de webserver. Zo begint ook de Dispatcher.
1. Start de AEM-instantie.
1. Controleer het logboek en de foutendossiers voor uw Webserver en Dispatcher.
   * Afhankelijk van uw webserver worden berichten weergegeven zoals:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` en
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surf op de website via de webserver. Bevestig dat de inhoud naar wens wordt weergegeven.\
   Op een lokale installatie waar AEM op poort `4502` wordt uitgevoerd en de webserver op `80` toegang heeft tot de websiteconsole via beide:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * De resultaten moeten identiek zijn. Bevestig toegang tot andere pagina&#39;s met het zelfde mechanisme.

1. Controleer of de cachemap wordt gevuld.
1. Activeer een pagina om te controleren of de cache correct wordt leeggemaakt.
1. Als alles correct werkt, kunt u de `loglevel` tot `0` verminderen.

## Meerdere verzenders gebruiken {#using-multiple-dispatchers}

In complexe instellingen kunt u meerdere verzenders gebruiken. U kunt bijvoorbeeld het volgende gebruiken:

* één Dispatcher om een website op het Intranet te publiceren
* een tweede Dispatcher, onder een ander adres en met verschillende beveiligingsinstellingen, om dezelfde inhoud op internet te publiceren.

In dat geval moet u ervoor zorgen dat elke aanvraag slechts door één Dispatcher wordt behandeld. Een Dispatcher behandelt geen verzoeken die afkomstig zijn van een andere Dispatcher. Zorg er daarom voor dat beide verzenders de AEM-website rechtstreeks openen.

## Foutopsporing {#debugging}

Bij het toevoegen van de header `X-Dispatcher-Info` aan een aanvraag antwoordt Dispatcher of het doel in de cache is opgeslagen, is geretourneerd uit de cache of helemaal niet in cache kan worden geplaatst. De responsheader `X-Cache-Info` bevat deze informatie in leesbare vorm. U kunt deze antwoordheaders gebruiken om fouten op te sporen in problemen die betrekking hebben op reacties die in cache zijn geplaatst door de Dispatcher.

Deze functionaliteit wordt niet standaard ingeschakeld, zodat de responsheader `X-Cache-Info` wordt opgenomen, moet het landbouwbedrijf de volgende vermelding bevatten:

```xml
/info "1"
```

Bijvoorbeeld:

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

De header van `X-Dispatcher-Info` heeft ook geen waarde nodig, maar als u `curl` gebruikt om te testen, moet u een waarde opgeven die naar de koptekst moet worden verzonden, zoals:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Hieronder ziet u een lijst met de antwoordheaders die `X-Dispatcher-Info` retourneert:

* **doeldossier caching**\
  Het doelbestand bevindt zich in de cache en de Dispatcher heeft vastgesteld dat het geldig is om het te leveren.
* **caching**\
  Het doelbestand bevindt zich niet in de cache en de Dispatcher heeft bepaald dat het bestand geldig is om de uitvoer in cache te plaatsen en te leveren.
* **caching: Statistische dossier is recenter**
Het doelbestand bevindt zich in de cache. Een recentere statusbestand kan de validatie echter wel ongedaan maken. De Dispatcher verwijdert het doelbestand, maakt het opnieuw van de uitvoer en levert het.
* **niet cacheable: documentwortel niet-bestaand**
De configuratie van het landbouwbedrijf bevat geen documentwortel (configuratieelement `cache.docroot`).
* **niet cacheable: de weg van het geheim voorgeheugendossier te lang**\
  Het doelbestand - de samenvoeging van het hoofdbestand van het document en het URL-bestand - overschrijdt de langst mogelijke bestandsnaam op het systeem.
* **niet cacheable: tijdelijk dossierweg te lang**\
  De sjabloon voor tijdelijke bestandsnamen overschrijdt de langst mogelijke bestandsnaam op het systeem. De Dispatcher maakt eerst een tijdelijk bestand voordat het in de cache opgeslagen bestand wordt gemaakt of overschreven. De tijdelijke bestandsnaam is de naam van het doelbestand met de tekens `_YYYYXXXXXX` eraan toegevoegd, waarbij de tekens `Y` en `X` worden vervangen om een unieke naam te maken.
* **niet cacheable: verzoek URL mist uitbreiding**\
  De aanvraag-URL heeft geen extensie of er is een pad dat volgt op de bestandsextensie, bijvoorbeeld: `/test.html/a/path` .
* **niet cacheable: verzoek moest GET of HEAD zijn**
De HTTP-methode is geen GET of HEAD. De Dispatcher gaat ervan uit dat de uitvoer dynamische gegevens bevat die niet in de cache mogen worden opgeslagen.
* **niet cacheable: het verzoek bevatte een vraagkoord**\
  Het verzoek bevatte een queryreeks. De Dispatcher gaat ervan uit dat de uitvoer afhankelijk is van de opgegeven querytekenreeks en dus niet in cache wordt geplaatst.
* **niet cacheable: de zittingsmanager moet** voor authentiek verklaren\
  Een zittingsmanager (de configuratie bevat a `sessionmanagement` knoop) regeert het geheime voorgeheugen van het landbouwbedrijf en het verzoek bevatte niet de aangewezen authentificatieinformatie.
* **niet cacheable: het verzoek bevat vergunning**\
  Het landbouwbedrijf wordt niet toegestaan om output ( `allowAuthorized 0`) in het voorgeheugen onder te brengen en het verzoek bevat authentificatieinformatie.
* **niet cacheable: het doel is een folder**\
  Het doelbestand is een map. Deze locatie kan wijzen op een conceptuele fout, waarbij een URL en een subURL beide cacheable-uitvoer bevatten. Als een aanvraag naar `/test.html/a/file.ext` bijvoorbeeld als eerste wordt ingediend en cacheable-uitvoer bevat, kan de Dispatcher de uitvoer van een volgende aanvraag naar `/test.html` niet in cache plaatsen.
* **niet cacheable: verzoek URL heeft een het slepen schuine streep**\
  De aanvraag-URL heeft een slash.
* **niet cacheable: verzoek URL mist in geheim voorgeheugenregels**\
  De het geheime voorgeheugenregels van het landbouwbedrijf ontkennen uitdrukkelijk caching de output van één of ander verzoek URL.
* **niet cacheable: de vergunningscontrole ontkende toegang**\
  De de vergunningscontrole van het landbouwbedrijf ontkende toegang tot het caching dossier.
* **niet cacheable: de zitting is ongeldig**
Een sessiemanager (configuratie bevat een knooppunt `sessionmanagement` ) bestuurt het cachegeheugen van het bedrijf en de sessie van de gebruiker is niet of niet langer geldig.
* **niet cacheable: reactie bevat`no_cache`**
De externe server heeft een `Dispatcher: no_cache` -header geretourneerd, waardoor de Dispatcher de uitvoer niet in cache kon plaatsen.
* **niet cacheable: de lengte van de reactieinhoud is nul**
De lengte van de reactie is nul. De Dispatcher maakt geen bestand met een lengte van nul.

