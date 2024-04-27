---
title: Dispatcher configureren
description: Leer hoe u Dispatcher configureert. Leer over steun voor IPv4 en IPv6, configuratiedossiers, omgevingsvariabelen, noemend de instantie, bepalend landbouwbedrijven, identificerend virtuele gastheren, en meer.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '8854'
ht-degree: 0%

---

# Dispatcher configureren {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher-versies zijn onafhankelijk van AEM. U bent mogelijk omgeleid naar deze pagina als u een koppeling naar de Dispatcher-documentatie hebt gevolgd die is ingesloten in de documentatie voor een vorige versie van AEM.

De volgende secties beschrijven hoe te om diverse aspecten van de Verzender te vormen.

## Ondersteuning voor IPv4 en IPv6 {#support-for-ipv-and-ipv}

Alle elementen van AEM en Dispatcher kunnen in zowel IPv4 als IPv6 netwerken worden geïnstalleerd. Zie [IPV4 en IPV6](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#ipv-and-ipv).

## Dispatcher Configuration Files {#dispatcher-configuration-files}

Standaard wordt de Dispatcher-configuratie opgeslagen in de `dispatcher.any` tekstbestand, hoewel u de naam en locatie van dit bestand tijdens de installatie kunt wijzigen.

Het configuratiedossier bevat een reeks enig-getaxeerde of multi-getaxeerde eigenschappen die het gedrag van Dispatcher controleren:

* Namen van eigenschappen worden voorafgegaan door een slash `/`.
* Met multi-getaxeerde eigenschappen worden onderliggende items tussen accolades geplaatst `{ }`.

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

Bijvoorbeeld, om het dossier myFarm.any in de /farm configuratie te omvatten gebruik de volgende code:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Als u een reeks bestanden wilt opgeven die u wilt opnemen, gebruikt u de asterisk (`*`) als een jokerteken.

Als de bestanden `farm_1.any` tot `farm_5.any` U kunt de configuratie van de landbouwbedrijven 1 tot 5 bevatten, kunt u hen omvatten als volgt:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Omgevingsvariabelen gebruiken {#using-environment-variables}

U kunt omgevingsvariabelen gebruiken in tekenreeksgetaxeerde eigenschappen in het bestand dispatcher.any in plaats van de waarden hard te coderen. Als u de waarde van een omgevingsvariabele wilt opnemen, gebruikt u de indeling `${variable_name}`.

Als het bestand dispatcher.any zich bijvoorbeeld in dezelfde map bevindt als de cachemap, geldt de volgende waarde voor de [docroot](#specifying-the-cache-directory) eigenschap kan worden gebruikt:

```xml
/docroot "${PWD}/cache"
```

Als ander voorbeeld, als u een milieuvariabele creeert die wordt genoemd `PUBLISH_IP` dat hostname van AEM publiceert instantie, de volgende configuratie van [/renders](#defining-page-renderers-renders) eigenschap kan worden gebruikt:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## De instantie Dispatcher een naam geven {#naming-the-dispatcher-instance-name}

Gebruik de `/name` eigenschap om een unieke naam op te geven om uw instantie Dispatcher te identificeren. De `/name` bezit is een top-level bezit in de configuratiestructuur.

## Bedrijven definiëren {#defining-farms-farms}

De `/farms` Deze eigenschap definieert een of meer sets Dispatcher-gedrag, waarbij elke set aan verschillende websites of URL&#39;s is gekoppeld. De `/farms` de eigenschap kan bestaan uit één bedrijf of meerdere landbouwbedrijven :

* Gebruik één landbouwbedrijf wanneer u Dispatcher al uw Web-pagina&#39;s of Websites op de zelfde manier wilt behandelen.
* Maak meerdere boerderijen wanneer verschillende gebieden van uw website of verschillende websites een ander Dispatcher-gedrag vereisen.

De `/farms` bezit is een top-level bezit in de configuratiestructuur. Om een landbouwbedrijf te bepalen, voeg een kindbezit aan toe `/farms` eigenschap. Gebruik een bezitsnaam die uniek het landbouwbedrijf binnen de instantie van de Verzender identificeert.

De `/farmname` Deze eigenschap is gecompileerd en bevat andere eigenschappen die het gedrag Verzender definiëren:

* De URL&#39;s van de pagina&#39;s waarop het landbouwbedrijf van toepassing is.
* Een of meer service-URL&#39;s (doorgaans van AEM publicatieinstanties) die moeten worden gebruikt voor het weergeven van documenten.
* De statistische gegevens die moeten worden gebruikt voor meerdere renderers van documenten die een taakverdeling hebben.
* Verschillende andere gedragingen, zoals welke bestanden in cache moeten worden geplaatst en waar ze moeten worden opgeslagen.

De waarde kan elk alfanumeriek teken (a-z, 0-9) bevatten. Het volgende voorbeeld toont de skeletdefinitie voor twee landbouwbedrijven genoemd `/daycom` en `/docsdaycom`:

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
>Als u meer dan één gebruikt geef landbouwbedrijf terug, wordt de lijst geëvalueerd bottom-up. Deze stroom is relevant bij het definiëren van [Virtuele hosts](#identifying-virtual-hosts-virtualhosts) voor uw websites.

Elk landbouwbedrijfbezit kan de volgende kindeigenschappen bevatten:

| Eigenschapnaam | Beschrijving |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | Standaardstartpagina (optioneel) (alleen IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | De headers van de HTTP-client-aanvraag die moeten worden doorgegeven. |
| [/virtuele hosts](#identifying-virtual-hosts-virtualhosts) | De virtuele hosts van deze boerderij. |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | Ondersteuning voor sessiebeheer en verificatie. |
| [/renders](#defining-page-renderers-renders) | De servers die gerenderde pagina&#39;s leveren (AEM gewoonlijk exemplaren publiceren). |
| [/filter](#configuring-access-to-content-filter) | Bepaalt URLs waaraan de Verzender toegang toelaat. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Vormt toegang tot vanity URLs. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | Steun voor de doorzending van verzoeken om syndicatie. |
| [/cache](#configuring-the-dispatcher-cache-cache) | Vormt caching gedrag. |
| [/statistiek](#configuring-load-balancing-statistics) | Statistische categorieën definiëren voor berekeningen van de taakverdeling. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | De map die kleverige documenten bevat. |
| [/health_check](#specifying-a-health-check-page) | De URL die moet worden gebruikt om de beschikbaarheid van de server te bepalen. |
| [/retryDelay](#specifying-the-page-retry-delay) | De vertraging voordat een mislukte verbinding opnieuw wordt geprobeerd. |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Sancties die van invloed zijn op statistieken voor berekeningen voor taakverdeling. |
| [/failover](#using-the-failover-mechanism) | Verzend verzoeken opnieuw naar verschillende renders wanneer het oorspronkelijke verzoek ontbreekt. |
| [/auth_checker](permissions-cache.md) | Voor toestemming-gevoelige caching, zie [Beveiligde inhoud in cache plaatsen](permissions-cache.md). |

## Een standaardpagina opgeven (alleen IIS) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>De `/homepage`parameter (alleen IIS) werkt niet meer. Gebruik in plaats daarvan de opdracht [IIS URL Rewrite Module](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Als u Apache gebruikt, moet u de `mod_rewrite` -module. Raadpleeg de documentatie bij de Apache-website voor meer informatie over `mod_rewrite` (bijvoorbeeld [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). Wanneer u `mod_rewrite`, is het aan te raden de markering &#39;passthrough|PT&#39; (passthrough naar volgende handler) te gebruiken om de herschrijfengine te dwingen de `uri` interne `request_rec` structuur aan de waarde van `filename` veld.

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

De `/clientheaders` eigenschap definieert een lijst met HTTP-headers die door Dispatcher worden doorgegeven van de HTTP-client-aanvraag naar de renderer (AEM-instantie).

Standaard verzendt Dispatcher de standaard HTTP-headers naar de AEM-instantie. In sommige gevallen wilt u mogelijk andere koppen doorsturen of specifieke koppen verwijderen:

* Voeg kopballen, zoals douanekopballen, toe die uw AEM instantie in de HTTP- aanvraag verwacht.
* Verwijder kopballen, zoals authentificatiekopballen die slechts relevant voor de Webserver zijn.

Als u de reeks kopballen aanpast om over te gaan, moet u een uitvoerige lijst van kopballen, met inbegrip van die kopballen specificeren die normaal inbegrepen door gebrek zijn.

Voor een Dispatcher-instantie die pagina-activeringsverzoeken voor publicatie-instanties afhandelt, is bijvoorbeeld de opdracht `PATH` in de `/clientheaders` sectie. De `PATH` de kopbal laat communicatie tussen de replicatieagent en de Dispatcher toe.

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

De `/virtualhosts` eigenschap definieert een lijst met alle combinaties hostname/URI die Dispatcher accepteert voor dit farm. U kunt de asterisk (`*`) als jokerteken. Waarden voor de `virtualhosts` eigenschap gebruikt de volgende indeling:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Optioneel) Ofwel `https://` of `https://.`
* `host`: De naam of het IP-adres van de hostcomputer en, indien nodig, het poortnummer. (Zie [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Optioneel) Het pad naar de bronnen.

De volgende verzoeken van de voorbeeldconfiguratie om `.com` en `.ch` domeinen van myCompany, en alle domeinen van mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

De volgende configuratiehandgrepen *alles* verzoeken:

```xml
   /virtualhosts
    {
    "*"
    }
```

### De virtuele host oplossen {#resolving-the-virtual-host}

Wanneer de Ontvanger een HTTP of HTTPS verzoek ontvangt, vindt het de virtuele gastheerwaarde die best aanpast `host,` `uri`, en `scheme` kopteksten van de aanvraag. Dispatcher evalueert de waarden in het dialoogvenster `virtualhosts` eigenschappen in de volgende volgorde:

* De afzender begint bij het laagste landbouwbedrijf en vordert omhoog in het dispatcher.any dossier.
* Voor elk landbouwbedrijf, begint de Ontvanger met de hoogste waarde in `virtualhosts` en gaat verder naar de lijst met waarden.

Dispatcher vindt de best-passende virtuele gastheerwaarde op de volgende manier:

* De eerst aangetroffen virtuele host die overeenkomt met alle drie de `host`de `scheme`en de `uri` van het verzoek wordt gebruikt.
* Indien niet `virtualhosts` waarden hebben `scheme` en `uri` onderdelen die overeenkomen met de `scheme` en `uri` van het verzoek, de eerste ontmoet virtuele gastheer die aanpast `host` van het verzoek wordt gebruikt.
* Indien niet `virtualhosts` de waarden hebben een gastheerdeel dat de gastheer van het verzoek aanpast, wordt de hoogste virtuele gastheer van het hoogste landbouwbedrijf gebruikt.

Daarom zou u uw standaard virtuele gastheer bij de bovenkant van moeten plaatsen `virtualhosts` bezit in het hoogste landbouwbedrijf van uw `dispatcher.any` bestand.

### Voorbeeld virtuele hostresolutie {#example-virtual-host-resolution}

In het volgende voorbeeld wordt een fragment uit een `dispatcher.any` dossier dat twee landbouwbedrijven van de Verzender bepaalt, en elk landbouwbedrijf bepaalt een `virtualhosts` eigenschap.

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

## Beveiligde sessies inschakelen - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` Instellen op `"0"` in de `/cache` om deze functie in te schakelen. Zoals in het [In cache plaatsen wanneer verificatie wordt gebruikt](#caching-when-authentication-is-used) -sectie, wanneer u instelt `/allowAuthorized 0 ` verzoeken die authentificatieinformatie omvatten zijn **niet** in cache geplaatst. Als het toestemming-gevoelige caching wordt vereist, zie [Beveiligde inhoud in cache plaatsen](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/permissions-cache) pagina.

Creeer een veilige zitting voor toegang tot teruggeven landbouwbedrijf zodat de gebruikers moeten login om het even welke pagina in het landbouwbedrijf toegang hebben. Na het programma openen, kunnen de gebruikers tot pagina&#39;s in het landbouwbedrijf toegang hebben. Zie [Een gesloten gebruikersgroep maken](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/cug#creating-the-user-group-to-be-used) voor informatie over het gebruiken van deze eigenschap met CUGs. Zie ook de Dispatcher [Beveiligingscontrolelijst](/help/using/security-checklist.md) voordat je live gaat.

De `/sessionmanagement` eigenschap is een subeigenschap van `/farms`.

>[!CAUTION]
>
>Als gedeelten van uw website verschillende toegangsvereisten gebruiken, moet u meerdere boerderijen definiëren.

**/sessionmanagement** heeft verschillende subparameters:

**/directory** (verplicht)

De map waarin de sessiegegevens worden opgeslagen. Als de map niet bestaat, wordt deze gemaakt.

>[!CAUTION]
>
> Wanneer het vormen van de folder subparameter, **niet** verwijzen naar de hoofdmap (`/directory "/"`), omdat dit ernstige problemen kan veroorzaken. Geef altijd het pad op naar de map waarin de sessiegegevens worden opgeslagen. Bijvoorbeeld:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (optioneel)

Hoe de sessiegegevens worden gecodeerd. Gebruiken `md5` voor versleuteling met het md5-algoritme, of `hex` voor hexadecimale codering. Als u de sessiegegevens versleutelt, kan een gebruiker met toegang tot het bestandssysteem de sessie-inhoud niet lezen. De standaardwaarde is `md5`.

**/header** (optioneel)

De naam van de HTTP-header of het cookie waarin de autorisatiegegevens zijn opgeslagen. Als u de informatie opslaat in de http-header, gebruikt u `HTTP:<header-name>`. Als u de gegevens in een cookie wilt opslaan, gebruikt u `Cookie:<header-name>`. Als u geen waarde opgeeft, `HTTP:authorization` wordt gebruikt.

**/timeout** (optioneel)

Het aantal seconden tot de sessietijden uit nadat deze voor het laatst zijn gebruikt. Indien niet opgegeven `"800"` wordt gebruikt, zodat de zittingstijden iets meer dan 13 minuten na het laatste verzoek van de gebruiker uitkomen.

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

De eigenschap /renders definieert de URL waarnaar de afzender een verzoek verzendt om een document te renderen. Het volgende voorbeeld `/renders` -sectie identificeert één AEM voor rendering:

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

De volgende voorbeeldsectie /renders identificeert een AEM instantie die op de zelfde computer zoals Dispatcher loopt:

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

In het volgende voorbeeld /renders-gedeelte worden renderverzoeken gelijkelijk verdeeld over twee AEM:

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

Geeft de time-out van de verbinding op die de AEM instantie benadert, in milliseconden. De standaardwaarde is `"0"`, waardoor de Dispatcher oneindig wacht.

**/receiveTimeout**

Geeft de tijd op in milliseconden die een reactie mag afleggen. De standaardwaarde is `"600000"`, waardoor de Dispatcher 10 minuten wacht. Een instelling van `"0"` Hiermee wordt de time-out verwijderd.

Wanneer de time-out wordt bereikt tijdens het parseren van responsheaders, wordt een HTTP-status van 504 (Bad Gateway) geretourneerd. Als de onderbreking wordt bereikt terwijl het antwoordlichaam wordt gelezen, keert de Dispatcher de onvolledige reactie op de cliënt terug. Het schrapt ook om het even welke caching dossiers die zouden kunnen zijn geschreven.

**/ipv4**

Hiermee wordt opgegeven of Dispatcher de opdracht `getaddrinfo` functie (voor IPv6) of de `gethostbyname` functie (voor IPv4) voor het verkrijgen van het IP adres van teruggeven. Een waarde van 0 oorzaken `getaddrinfo` te gebruiken. Een waarde van `1` oorzaken `gethostbyname` te gebruiken. De standaardwaarde is `0`.

De `getaddrinfo` Deze functie retourneert een lijst met IP-adressen. Dispatcher herhaalt de lijst van adressen tot het een verbinding TCP/IP vestigt. Daarom `ipv4` Het bezit is belangrijk wanneer teruggeven hostname met veelvoudige IP adressen en de gastheer, in antwoord op wordt geassocieerd `getaddrinfo` functie, keert een lijst van IP adressen terug die altijd in de zelfde orde zijn. In deze situatie moet u de `gethostbyname` functie zodat het IP adres dat de Ontvanger met verbindt wordt willekeurig verdeeld.

Amazon Elastic Load Balancing (ELB) is een service die reageert op getaddrinfo met een lijst met IP-adressen die mogelijk dezelfde volgorde heeft.

**/secure**

Als de `/secure` eigenschap heeft een waarde van `"1"`, gebruikt Dispatcher HTTPS om te communiceren met de AEM instantie. Zie voor meer informatie [Dispatcher configureren voor gebruik van SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Met Dispatcher-versie **4.1.6.**, kunt u de `/always-resolve` eigenschap als volgt:

* Wanneer ingesteld op `"1"`, lost het gastheer-naam op elk verzoek (de Verzender plaatst nooit om het even welk IP adres) op. Er kan een lichte prestatiesinvloed toe te schrijven zijn aan de extra vraag die wordt vereist om de gastheerinformatie voor elk verzoek te krijgen.
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

Gebruik de `/filter` om de HTTP-aanvragen op te geven die Dispatcher accepteert. Alle andere aanvragen worden teruggestuurd naar de webserver met een foutcode van 404 (pagina niet gevonden). Indien niet `/filter` -sectie bestaat, worden alle aanvragen geaccepteerd.

**Opmerking:** Verzoeken om [statfile](#naming-the-statfile) worden altijd afgewezen.

>[!CAUTION]
>
>Zie de [Controlelijst voor beveiliging van verzender](security-checklist.md) voor verdere overwegingen wanneer het beperken van toegang gebruikend Dispatcher. Zie ook de [Beveiligingschecklist AEM](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist#security) voor meer veiligheidsdetails betreffende uw AEM installatie.

De `/filter` de sectie bestaat uit een reeks regels die of toegang tot inhoud volgens patronen in het verzoek-lijn deel van het HTTP- verzoek ontkennen of toestaan. Gebruik een strategie voor lijst van gewenste personen voor uw `/filter` sectie:

* Eerst, ontken toegang tot alles.
* Toegang tot inhoud toestaan als dat nodig is.

>[!NOTE]
>
>Wis de cache wanneer de filterregels worden gewijzigd.

### Een filter definiëren {#defining-a-filter}

Elk item in het dialoogvenster `/filter` Deze sectie bevat een type en een patroon die overeenkomen met een specifiek element van de aanvraagregel of de gehele aanvraagregel. Elk filter kan de volgende items bevatten:

* **Type**: De `/type` Hiermee wordt aangegeven of toegang wordt toegestaan of geweigerd voor de aanvragen die overeenkomen met het patroon. De waarde kan `allow` of `deny`.

* **Element van de aanvraagregel:** Inclusief `/method`, `/url`, `/query`, of `/protocol` en een patroon voor het filtreren verzoeken volgens deze specifieke delen van het verzoek-lijn deel van het HTTP- verzoek. Filteren op elementen van de aanvraaglijn (eerder dan op de volledige verzoeklijn) is de aangewezen filtermethode.

* **Geavanceerde elementen van de aanvraagregel:** Vanaf Dispatcher 4.2.0 zijn er vier nieuwe filterelementen beschikbaar voor gebruik. Deze nieuwe elementen zijn `/path`, `/selectors`, `/extension`, en `/suffix` respectievelijk. Neem een of meer van deze items op om de URL-patronen verder te beheren.

>[!NOTE]
>
>Voor meer informatie over welk deel van de verzoeklijn elk van deze elementenverwijzingen, zie [URL-decompositie in verkoop](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki-pagina.

* **glob, eigenschap**: De `/glob` eigenschap wordt gebruikt om overeen te komen met de gehele request-line van de HTTP-aanvraag.

>[!CAUTION]
>
>Filteren met globs is afgekeurd in Dispatcher. Als zodanig moet u voorkomen dat globals in de `/filter` secties omdat dit tot beveiligingsproblemen kan leiden. Dus in plaats van:
>
>`/glob "* *.css *"`
>
>Gebruiken
>
>`/url "*.css"`

#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 definieert de [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) als volgt:

`Method Request-URI HTTP-Version<CRLF>`

De `<CRLF>` tekens staan voor een regelterugloop gevolgd door een regelinvoer. Het volgende voorbeeld is verzoek-lijn die wordt ontvangen wanneer een cliënt om de V.S.-Engelse pagina van de plaats WKND verzoekt:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

Uw patronen moeten de ruimtetekens in verzoek-lijn en de `<CRLF>` tekens.

#### Dubbele aanhalingstekens versus enkele aanhalingstekens {#double-quotes-vs-single-quotes}

Gebruik bij het maken van filterregels dubbele aanhalingstekens `"pattern"` voor eenvoudige patronen. Als u Dispatcher 4.2.0 of hoger gebruikt en uw patroon een reguliere expressie bevat, moet u het regex-patroon insluiten `'(pattern1|pattern2)'` binnen enkele aanhalingstekens.

#### Reguliere expressies {#regular-expressions}

In de versies van de Verzender later dan 4.2.0, kunt u POSIX Uitgebreide Reguliere Uitdrukkingen in uw filterpatronen omvatten.

#### Problemen met filters oplossen {#troubleshooting-filters}

Als de filters niet worden geactiveerd zoals u zou verwachten, schakelt u [Trackregistratie](#trace-logging) op Dispatcher zodat kunt u zien welk filter het verzoek onderschept.

#### Voorbeeldfilter: Alles weigeren {#example-filter-deny-all}

In de volgende voorbeeldfiltersectie worden aanvragen voor alle bestanden door Dispatcher afgewezen. Ontken toegang tot alle bestanden en geef vervolgens toegang tot specifieke gebieden.

```xml
/0001  { /type "deny" /url "*"  }
```

Verzoeken naar een expliciet geweigerd gebied hebben tot gevolg dat een foutcode van 404 (pagina niet gevonden) wordt geretourneerd.

#### Voorbeeldfilter: toegang tot specifieke gebieden weigeren {#example-filter-deny-access-to-specific-areas}

Met filters kunt u ook toegang tot verschillende elementen weigeren, zoals ASP-pagina&#39;s en gevoelige gebieden in een publicatie-instantie. Met het volgende filter krijgt u geen toegang tot ASP-pagina&#39;s:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Voorbeeldfilter: aanvragen voor POSTEN inschakelen {#example-filter-enable-post-requests}

Met het volgende voorbeeldfilter kunt u formuliergegevens verzenden met de methode POST:

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

Als u toegang moet hebben tot enkele pagina&#39;s binnen het beperkte gebied, kunt u deze toegankelijk maken. Voeg bijvoorbeeld de volgende sectie toe om toegang tot het tabblad Archiveren in de workflowconsole toe te staan:

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

Hieronder ziet u een regelvoorbeeld waarin wordt voorkomen dat inhoud wordt opgehaald uit het `/content` pad en de bijbehorende substructuur, met filters voor pad, kiezers en extensies:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### Voorbeeld-/filtersectie {#example-filter-section}

Wanneer het vormen van Dispatcher, zou u externe toegang zoveel mogelijk moeten beperken. In het volgende voorbeeld wordt minimale toegang geboden aan externe bezoekers:

* `/content`
* diverse inhoud, zoals ontwerpen en clientbibliotheken. Bijvoorbeeld:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Nadat u filters hebt gemaakt, [toegang tot testpagina](#testing-dispatcher-security) om ervoor te zorgen dat uw AEM instantie veilig is.

Het volgende `/filter` van de `dispatcher.any` kan als basis worden gebruikt in uw [Dispatcher-configuratiebestand.](#dispatcher-configuration-files)

Dit voorbeeld is gebaseerd op het standaardconfiguratiedossier dat van Dispatcher wordt voorzien en als voorbeeld voor gebruik in een productiemilieu bedoeld is. Objecten met voorvoegsel `#` worden gedeactiveerd (met opmerkingen). Wees voorzichtig als u besluit een van deze items te activeren (door het verwijderen van de `#` op die regel). Dit kan gevolgen hebben voor de beveiliging.

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
>Wanneer u Apache gebruikt, ontwerpt u uw filter-URL-patronen volgens de eigenschap DispatcherUseProcinedURL van de module Dispatcher. (Zie [Apache Web Server - Uw Apache Web Server voor Dispatcher configureren](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

Overweeg de volgende aanbevelingen als u verkiest om toegang uit te breiden:

* Externe toegang tot uitschakelen `/admin` als u CQ-versie 5.4 of een eerdere versie gebruikt.

* Voorzichtigheid is geboden wanneer toegang wordt verleend tot bestanden in `/libs`. Toegang moet op individuele basis worden toegestaan.
* Ontken toegang tot de replicatieconfiguratie zodat kan het niet worden gezien:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Toegang tot de Google Gadgets reverse-proxy weigeren:

   * `/libs/opensocial/proxy*`

Afhankelijk van uw installatie kunnen er meer bronnen onder zijn `/libs`, `/apps` of elders, die beschikbaar moeten worden gesteld. U kunt de `access.log` bestand als een methode om te bepalen welke bronnen extern worden benaderd.

>[!CAUTION]
>
>De toegang tot consoles en folders kan een veiligheidsrisico voor productiemilieu&#39;s vormen. Tenzij u een expliciete motivering hebt, moeten deze worden gedeactiveerd (gemarkeerd als commentaar).

>[!CAUTION]
>
>Als u [rapporten gebruiken in een publicatieomgeving](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/operations/reporting#using-reports-in-a-publish-environment)moet u Dispatcher configureren om toegang tot `/etc/reports` voor externe bezoekers.

### Query-tekenreeksen beperken {#restricting-query-strings}

Sinds Dispatcher versie 4.1.5 kunt u de `/filter` om querytekenreeksen te beperken. Het wordt hoogst geadviseerd uitdrukkelijk vraagkoorden toe te staan en generische toelage door uit te sluiten `allow` filterelementen.

Eén item kan een van de volgende `glob` of een combinatie van `method`, `url`, `query`, en `version`, maar niet beide. In het volgende voorbeeld wordt het volgende `a=*` querytekenreeks en ontkent alle andere querytekenreeksen voor URL&#39;s die worden omgezet in de `/etc` knooppunt:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Als een regel een `/query`, komt het slechts verzoeken aan die een vraagkoord bevatten en het verstrekte vraagpatroon aanpassen.
>
>In het bovenstaande voorbeeld geldt dat als `/etc` die geen vraagkoord ook zouden moeten worden toegestaan, zouden de volgende regels worden vereist:
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Beveiliging van Dispatcher testen {#testing-dispatcher-security}

Dispatcher-filters blokkeren de toegang tot de volgende pagina&#39;s en scripts bij AEM publicatie-instanties. Gebruik een webbrowser om te proberen de volgende pagina&#39;s te openen zoals een bezoeker van de site zou doen en om te controleren of code 404 wordt geretourneerd. Pas de filters aan als er andere resultaten worden verkregen.

U ziet normale paginerendering voor `/content/add_valid_page.html?debug=layout`.

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

Om te bepalen of anonieme schrijf toegang wordt toegelaten, geef het volgende bevel in een terminal of bevelherinnering uit. U zou geen gegevens aan de knoop moeten kunnen schrijven.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Om te proberen om het geheime voorgeheugen van de Verzender ongeldig te maken en ervoor te zorgen dat u code 403 reactie ontvangt, geef het volgende bevel in een terminal of bevelherinnering uit:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Toegang tot URL&#39;s met Vanity inschakelen {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Configureer Dispatcher om toegang in te schakelen tot vanity URL&#39;s die zijn geconfigureerd voor uw AEM pagina&#39;s.

Wanneer toegang tot vanity URLs wordt toegelaten, roept de Verzender periodiek de dienst die op de teruggeeft instantie loopt om een lijst van vanity URLs te verkrijgen. Dispatcher slaat deze lijst op in een lokaal bestand. Wanneer een aanvraag voor een pagina wordt afgewezen vanwege een filter in het dialoogvenster `/filter` , raadpleegt Dispatcher de lijst met vanity URL&#39;s. Als de ontkende URL in de lijst staat, geeft Dispatcher toegang tot de vanity URL.

Als u toegang tot vanity-URL&#39;s wilt inschakelen, voegt u een `/vanity_urls` aan de `/farms` -sectie, vergelijkbaar met het volgende voorbeeld:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

De `/vanity_urls` Deze sectie bevat de volgende eigenschappen:

* `/url`: Het pad naar de service vanity URL die wordt uitgevoerd op de renderinstantie. De waarde van deze eigenschap moet `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Het pad naar het lokale bestand waarin Dispatcher de lijst met vanity URL&#39;s opslaat. Zorg ervoor dat Dispatcher schrijftoegang heeft tot dit bestand.
* `/delay`: (Seconden) De tijd tussen vraag aan de dienst van vanity URL.

>[!NOTE]
>
>Als de rendermethode een instantie van AEM is, moet u de [VanityURLS-Components-pakket van softwaredistributie](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) om de service vanity URL in te schakelen. (Zie [Softwaredistributie](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/contentmanagement/package-manager#software-distribution) voor meer informatie .)

Gebruik de volgende procedure om toegang tot vanity URLs toe te laten.

1. Als uw renderservice een AEM instantie is, installeert u de `com.adobe.granite.dispatcher.vanityurl.content` op de publicatie-instantie (zie de bovenstaande opmerking).
1. Voor elke vanity URL die u voor een AEM of CQ-pagina hebt geconfigureerd, controleert u of de [`/filter`](#configuring-access-to-content-filter) de URL wordt door de configuratie geweigerd. Voeg zo nodig een filter toe dat de URL weigert.
1. Voeg de `/vanity_urls` sectie hieronder `/farms`.
1. Start Apache-webserver opnieuw.

## Door:sturen de Verzoeken van de Syndicatie - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Syndicatieverzoeken zijn alleen bedoeld voor Dispatcher, zodat ze standaard niet naar de renderer worden verzonden (bijvoorbeeld een AEM-instantie).

Indien nodig stelt u de `/propagateSyndPost` eigenschap aan `"1"` om synchronisatieverzoeken door te sturen naar Dispatcher. Indien ingesteld, moet u ervoor zorgen dat de aanvragen voor POSTEN niet worden afgewezen in de filtersectie.

## Het vormen van het Geheime voorgeheugen van de Verzender - /cache {#configuring-the-dispatcher-cache-cache}

De `/cache` sectie bepaalt hoe de Verzender documenten in cache plaatst. Vorm verscheidene subproperties om uw caching strategieën uit te voeren:

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
>Voor plaatsen met bevoegdheid, lezen [Beveiligde inhoud in cache plaatsen](permissions-cache.md).

### De cachemap opgeven {#specifying-the-cache-directory}

De `/docroot` eigenschap identificeert de map waarin in cache opgeslagen bestanden worden.

>[!NOTE]
>
>De waarde moet precies hetzelfde pad hebben als de hoofdmap van het document van de webserver, zodat de zender en de webserver dezelfde bestanden verwerken.\
>De webserver is verantwoordelijk voor het leveren van de juiste statuscode wanneer het cachebestand van de Dispatcher wordt gebruikt. Daarom is het belangrijk dat deze ook kan worden gevonden.

Als u veelvoudige landbouwbedrijven gebruikt, moet elk landbouwbedrijf een verschillende documentwortel gebruiken.

### De naam van het statusbestand wijzigen {#naming-the-statfile}

De `/statfile` eigenschap identificeert het bestand dat als statfile moet worden gebruikt. Dispatcher gebruikt dit bestand om de tijd van de meest recente inhoudsupdate te registreren. Het statusbestand kan elk bestand op de webserver zijn.

De status heeft geen inhoud. Wanneer de inhoud wordt bijgewerkt, werkt Dispatcher de tijdstempel bij. De standaardstatus heet `.stat` en wordt opgeslagen in de hoofdmap van het document. Dispatcher blokkeert de toegang tot het statfile.

>[!NOTE]
>
>Indien `/statfileslevel` is geconfigureerd, negeert de Dispatcher het `/statfile` eigenschap en gebruik `.stat` als de naam.

### Stale documenten verzenden als er fouten optreden {#serving-stale-documents-when-errors-occur}

De `/serveStaleOnError` property controls whether Dispatcher returns invalidate documents when the render server returns an error. Wanneer een statusbestand wordt aangeraakt en cacheinhoud ongeldig wordt gemaakt, verwijdert Dispatcher de inhoud in de cache de volgende keer dat deze wordt opgevraagd.

Indien `/serveStaleOnError` is ingesteld op `"1"`, Verwijdert Dispatcher geen ongeldig gemaakte inhoud uit het cachegeheugen, tenzij de renderserver een succesvol antwoord retourneert. Een 5xx-reactie van AEM of een verbindingstijd zorgt ervoor dat Dispatcher de verouderde inhoud levert en reageert met en HTTP-status van 111 (Revalidation Failed).

### In cache plaatsen wanneer verificatie wordt gebruikt {#caching-when-authentication-is-used}

De `/allowAuthorized` het bezit controleert of de verzoeken die om het even welke volgende authentificatieinformatie bevatten in het voorgeheugen worden opgeslagen:

* De `authorization` header
* Een cookie genaamd `authorization`
* Een cookie genaamd `login-token`

Door gebrek, worden de verzoeken die deze authentificatieinformatie omvatten niet in het voorgeheugen ondergebracht omdat de authentificatie niet wordt uitgevoerd wanneer een caching document aan de cliënt is teruggekeerd. Met deze configuratie voorkomt u dat Dispatcher cachedocumenten kan verzenden aan gebruikers die niet de vereiste rechten hebben.

Als uw vereisten het in cache plaatsen van geverifieerde documenten echter toestaan, stelt u `/allowAuthorized` op één:

`/allowAuthorized "1"`

>[!NOTE]
>
>Om zittingsbeheer toe te laten (gebruikend `/sessionmanagement` eigenschap), `/allowAuthorized` eigenschap moet worden ingesteld op `"0"`.

### Documenten opgeven om in cache te plaatsen {#specifying-the-documents-to-cache}

De `/rules` Deze eigenschap bepaalt welke documenten in de cache worden geplaatst op basis van het documentpad. Ongeacht de `/rules` eigenschap, Dispatcher plaatst een document nooit in de cache in de volgende omstandigheden:

* Verzoek-URI bevat een vraagteken (`?`).
   * Geeft een dynamische pagina aan, zoals een zoekresultaat dat niet in de cache hoeft te worden opgeslagen.
* De bestandsextensie ontbreekt.
   * De webserver heeft de extensie nodig om het documenttype (het MIME-type) te bepalen.
* De verificatieheader wordt ingesteld (configureerbaar).
* Als de AEM instantie met de volgende kopballen antwoordt:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>De methoden GET of HEAD (voor de HTTP-header) kunnen door de Dispatcher in cache worden geplaatst. Zie voor meer informatie over het in cache plaatsen van responsheaders de [HTTP-responsheaders in cache plaatsen](#caching-http-response-headers) sectie.

Elk item in het dialoogvenster `/rules` eigenschap omvat een [`glob`](#designing-patterns-for-glob-properties) patroon en tekst:

* De `glob` wordt gebruikt om het pad van het document aan te passen.
* Het type geeft aan of de documenten die overeenkomen met de `glob` patroon. De waarde kan `allow` (om het document in cache te plaatsen) of `deny` (om het document altijd weer te geven).

Als u geen dynamische pagina&#39;s hebt (buiten die pagina&#39;s die reeds door de bovengenoemde regels worden uitgesloten), kunt u Dispatcher vormen om alles in het voorgeheugen onder te brengen. De sectie Regels ziet er als volgt uit:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Zie voor informatie over eigenschappen van glob [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties).

Als er gedeelten van de pagina dynamisch zijn (bijvoorbeeld een nieuwstoepassing) of zich in een gesloten gebruikersgroep bevinden, kunt u uitzonderingen definiëren:

>[!NOTE]
>
>Sluit gebruikersgroepen niet in cache plaatsen omdat gebruikersrechten niet zijn gecontroleerd op pagina&#39;s in de cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Compressie**

Op Apache-webservers kunt u de documenten in de cache comprimeren. Met compressie kan Apache het document op verzoek van de client in een gecomprimeerd formulier retourneren. Compressie wordt automatisch uitgevoerd door de Apache-module in te schakelen `mod_deflate`, bijvoorbeeld:

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

Gebruik de `/statfileslevel` eigenschap voor het ongeldig maken van cachebestanden volgens het pad:

* Dispatcher maakt `.stat`bestanden in elke map van de hoofdmap naar het niveau dat u opgeeft. De documenthoofdmap is niveau 0.
* Bestanden worden ongeldig gemaakt door op de knop `.stat` bestand. De `.stat` de laatste wijzigingsdatum van het bestand wordt vergeleken met de laatste wijzigingsdatum van een document in de cache. Het document wordt opnieuw ingesteld als de `.stat` is nieuwer.

* Wanneer een bestand op een bepaald niveau ongeldig wordt gemaakt, **alles** `.stat` bestanden uit de hoofdmap **tot** het niveau van het ongeldig gemaakte dossier of gevormd `statsfilevel` (de kleinste) wordt aangeraakt.

   * Als u bijvoorbeeld de instelling `statfileslevel` eigenschap tot en met 6 en een bestand wordt op niveau 5 dan elke `.stat` bestand van docroot naar 5 wordt aangeraakt. Als u doorgaat met dit voorbeeld en een bestand op niveau 7 ongeldig wordt gemaakt, wordt elke `stat` bestand van docroot naar zes wordt gewijzigd (sinds `/statfileslevel = "6"`).

Alleen bronnen **langs het pad** op het ongeldig gemaakte bestand wordt beïnvloed. Neem het volgende voorbeeld: een website gebruikt de structuur `/content/myWebsite/xx/.` Als u `statfileslevel` als 3, a `.stat`bestand wordt als volgt gemaakt:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Wanneer een bestand in `/content/myWebsite/xx` ongeldig is, dan elke `.stat` bestand van docroot naar `/content/myWebsite/xx`is aangeraakt. Dit scenario geldt alleen voor `/content/myWebsite/xx` en niet bijvoorbeeld `/content/myWebsite/yy` of `/content/anotherWebSite`.

>[!NOTE]
>
>Ongeldige validatie kan worden voorkomen door een extra koptekst te verzenden `CQ-Action-Scope:ResourceOnly`. Deze methode kan worden gebruikt om bepaalde middelen te spoelen zonder andere delen van het geheime voorgeheugen ongeldig te maken. Zie [deze pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) en [De Dispatcher-cache handmatig ongeldig maken](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/page-invalidate#configuring) voor meer informatie .

>[!NOTE]
>
>Als u een waarde voor de `/statfileslevel` eigenschap, `/statfile` eigenschap wordt genegeerd.

### Automatisch cachebestanden valideren {#automatically-invalidating-cached-files}

De `/invalidate` Deze eigenschap definieert de documenten die automatisch ongeldig worden gemaakt wanneer de inhoud wordt bijgewerkt.

Met automatische ongeldigmaking verwijdert Dispatcher geen in het cachegeheugen opgeslagen bestanden nadat de inhoud is bijgewerkt, maar controleert de geldigheid van deze bestanden op het moment dat ze de volgende keer worden aangevraagd. Documenten in de cache die niet automatisch ongeldig worden gemaakt, blijven in de cache totdat een inhoudsupdate deze expliciet verwijdert.

Automatische validatie wordt doorgaans gebruikt voor HTML-pagina&#39;s. HTML-pagina&#39;s bevatten vaak koppelingen naar andere pagina&#39;s, waardoor het moeilijk is om vast te stellen of een update van de inhoud van invloed is op een pagina. Als u ervoor wilt zorgen dat alle relevante pagina&#39;s ongeldig worden gemaakt wanneer de inhoud wordt bijgewerkt, maakt u automatisch alle HTML-pagina&#39;s ongeldig. De volgende configuratie maakt alle HTML pagina&#39;s ongeldig:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Zie voor informatie over eigenschappen van glob [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties).

Deze configuratie veroorzaakt de volgende activiteit wanneer `/content/wknd/us/en` is geactiveerd:

* Alle bestanden met patroon en.* worden verwijderd uit de `/content/wknd/us` map.
* De `/content/wknd/us/en./_jcr_content` map wordt verwijderd.
* Alle andere bestanden die overeenkomen met de `/invalidate` configuratie niet onmiddellijk worden verwijderd. Deze bestanden worden verwijderd wanneer de volgende aanvraag wordt uitgevoerd. In het voorbeeld: `/content/wknd.html` wordt niet verwijderd. In plaats daarvan wordt deze verwijderd wanneer `/content/wknd.html` is aangevraagd.

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

De AEM integratie met Adobe Analytics biedt configuratiegegevens in een `analytics.sitecatalyst.js` op uw website. Het voorbeeld `dispatcher.any` Het bestand dat bij Dispatcher wordt geleverd, bevat de volgende validatieregel voor dit bestand:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Aangepaste validatiescripts gebruiken {#using-custom-invalidation-scripts}

De `/invalidateHandler` staat u toe om een manuscript te bepalen dat voor elk ongeldigingsverzoek wordt geroepen dat door Dispatcher wordt ontvangen.

De methode wordt aangeroepen met de volgende argumenten:

* Handgreep - Het ongeldig gemaakte inhoudspad
* Handeling - De replicatiehandeling (bijvoorbeeld Activeren, Deactiveren)
* Toepassingsgebied van handeling - Het bereik van de replicatieactie (leeg, tenzij een koptekst van `CQ-Action-Scope: ResourceOnly` wordt verzonden, zie [In cache geplaatste pagina&#39;s ongeldig maken van AEM](page-invalidate.md) voor details)

Deze methode kan worden gebruikt voor verschillende gebruiksgevallen. Bijvoorbeeld het ongeldig maken van andere toepassingsspecifieke geheime voorgeheugens, of het behandelen van gevallen waar extern URL van een pagina, en zijn plaats in het docroot, niet de inhoudspad aanpast.

In het onderstaande voorbeeld wordt elk ongeldig verzoek in een bestand geregistreerd.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### Voorbeeldscript voor validatiehandlers {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### De clients beperken die de cache kunnen leegmaken {#limiting-the-clients-that-can-flush-the-cache}

De `/allowedClients` eigenschap definieert specifieke clients die de cache mogen leegmaken. De globbende patronen worden aangepast aan IP.

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

Zie voor informatie over eigenschappen van glob [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>U wordt aangeraden de `/allowedClients`.
>
>Als het niet wordt gedaan, kan om het even welke cliënt een vraag uitgeven om het geheime voorgeheugen te ontruimen. Als u dit herhaaldelijk doet, kan dit ernstige gevolgen hebben voor de prestaties van de site.

### URL-parameters worden genegeerd {#ignoring-url-parameters}

De `ignoreUrlParams` In deze sectie wordt gedefinieerd welke URL-parameters worden genegeerd bij het bepalen of een pagina in cache wordt geplaatst of via cache wordt geleverd:

* Wanneer een aanvraag-URL parameters bevat die allemaal worden genegeerd, wordt de pagina in de cache geplaatst.
* Wanneer een aanvraag-URL een of meer parameters bevat die niet worden genegeerd, wordt de pagina niet in de cache opgeslagen.

Wanneer een parameter voor een pagina wordt genegeerd, wordt de pagina in de cache geplaatst de eerste keer dat de pagina wordt aangevraagd. Volgende aanvragen voor de pagina worden op de pagina in de cache geplaatst, ongeacht de waarde van de parameter in het verzoek.

>[!NOTE]
>
>Men adviseert dat u vormt `ignoreUrlParams` op de wijze van de lijst van gewenste personen instellen. Als dusdanig, worden alle vraagparameters genegeerd en slechts worden bekende of verwachte vraagparameters vrijgesteld (&quot;ontkend&quot;) van wordt genegeerd. Zie voor meer informatie en voorbeelden [deze pagina](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner).

Om te specificeren welke parameters worden genegeerd, voeg glob regels aan toe `ignoreUrlParams` eigenschap:

* Als u een pagina in het cachegeheugen wilt plaatsen ondanks het verzoek met een URL-parameter, maakt u een glob-eigenschap waarmee de parameter kan worden genegeerd.
* Als u wilt voorkomen dat de pagina in de cache wordt opgeslagen, maakt u een glob-eigenschap die de parameter weigert (te negeren).

>[!NOTE]
>
>Wanneer het vormen van het glob bezit, zou het de naam van de vraagparameter moeten aanpassen. Als u bijvoorbeeld de parameter &quot;p1&quot; van de volgende URL wilt negeren `http://example.com/path/test.html?p1=test&p2=v2`, dan zou de glob eigenschap moeten zijn:
> `/0002 { /glob "p1" /type "allow" }`

In het volgende voorbeeld worden door Dispatcher alle parameters genegeerd, behalve de parameters `nocache` parameter. Vraag daarom URL&#39;s aan die de `nocache` parameter nooit in cache geplaatst door de Dispatcher:

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

In de context van de `ignoreUrlParams` in het bovenstaande configuratievoorbeeld zorgt de volgende HTTP-aanvraag ervoor dat de pagina in de cache wordt geplaatst omdat de `willbecached` parameter wordt genegeerd:

```xml
GET /mypage.html?willbecached=true
```

In de context van de `ignoreUrlParams` in het configuratievoorbeeld leidt de volgende HTTP-aanvraag ertoe dat de pagina **niet** worden in cache geplaatst omdat de `nocache` parameter wordt niet genegeerd:

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

Zie voor informatie over eigenschappen van glob [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties).

### HTTP-responsheaders in cache plaatsen {#caching-http-response-headers}

>[!NOTE]
>
>Deze functie is beschikbaar bij versie **4.1.11.** van de verzender.

De `/headers` staat u toe om de kopbaltypes te bepalen van HTTP die door de Dispatcher in het voorgeheugen zullen worden opgenomen. Op het eerste verzoek aan een middel uncached, worden alle kopballen die één van de gevormde waarden (zie de configuratiemonster hieronder) aanpassen opgeslagen in een afzonderlijk dossier, naast het geheim voorgeheugendossier. Bij verdere verzoeken aan het caching middel, worden de opgeslagen kopballen toegevoegd aan de reactie.

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
>Tekens voor het samenvoegen van bestanden zijn niet toegestaan. Zie voor meer informatie [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Als u Dispatcher nodig hebt om de eBay-antwoordheaders van AEM op te slaan en te leveren, doet u het volgende:
>
>* Voeg de koptekstnaam toe in het dialoogvenster `/cache/headers`sectie.
>* Voeg het volgende toe [Apache-richtlijn](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) in het gedeelte Verzender:
>
>```xml
>FileETag none
>```

### Machtigingen voor cachebestanden voor verzending {#dispatcher-cache-file-permissions}

De `mode` eigenschap bepaalt welke bestandsmachtigingen worden toegepast op nieuwe mappen en bestanden in de cache. Deze instelling wordt beperkt door de `umask` van het aanroepingsproces. Dit is een octaal getal dat wordt samengesteld uit de som van een of meer van de volgende waarden:

* `0400` Lezen door eigenaar toestaan.
* `0200` Schrijven door eigenaar toestaan.
* `0100` Laat de eigenaar in directory&#39;s zoeken.
* `0040` Lezen door groepsleden toestaan.
* `0020` Schrijven door groepsleden toestaan.
* `0010` Groepsleden mogen in de map zoeken.
* `0004` Lezen door anderen toestaan
* `0002` Schrijven door anderen toestaan.
* `0001` Anderen toestaan in de map te zoeken.

De standaardwaarde is `0755` waarmee de eigenaar de groep en anderen kan lezen, schrijven of doorzoeken.

### Startbestand met Throttling .stat aanraken {#throttling-stat-file-touching}

Met de standaardinstelling `/invalidate` eigenschap, elke activering maakt alle `.html` bestanden (wanneer het pad ervan overeenkomt met het `/invalidate` ). Op een website met aanzienlijk verkeer verhogen meerdere, daaropvolgende activeringen de CPU-belasting op de achtergrond. In een dergelijk scenario is het wenselijk &quot;vertragen&quot; `.stat` het aanraken van bestanden om de website ontvankelijk te houden. U kunt deze handeling uitvoeren met de opdracht `/gracePeriod` eigenschap.

De `/gracePeriod` eigenschap definieert het aantal seconden dat een standaard, automatisch ongeldig gemaakte resource mogelijk nog steeds uit de cache wordt aangeboden na de laatste activering. De eigenschap kan worden gebruikt in een installatie waarbij een batch activeringen anders de gehele cache herhaaldelijk ongeldig zouden maken. De aanbevolen waarde is 2 seconden.

Zie voor meer informatie `/invalidate` en `/statfileslevel`eerder.

### Het vormen tijd-Gebaseerde Invalidatie van het Geheime voorgeheugen - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

Op tijd gebaseerde cachevalidatie is afhankelijk van de `/enableTTL` en de aanwezigheid van normale verloopheaders van de HTTP-standaard. Als u de eigenschap instelt op 1 (`/enableTTL "1"`), worden de antwoordheaders vanaf de achtergrond geëvalueerd. Als de koppen een `Cache-Control`, `max-age` of `Expires` datum, wordt een hulpbestand gemaakt dat leeg is naast het bestand in de cache, waarbij de wijzigingstijd gelijk is aan de vervaldatum. Wanneer het cachebestand na de wijzigingstijd wordt opgevraagd, wordt het automatisch opnieuw opgevraagd vanaf de achtergrond.

Vóór Dispatcher 4.3.5, was de logica van de annulering van TTL gebaseerd slechts op de gevormde waarde van TTL. Met Dispatcher 4.3.5 stelt u beide TTL in **en** de regels voor het ongeldig maken van het cachegeheugen van Dispatcher worden verwerkt. Voor een bestand in de cache:

1. Indien `/enableTTL` is ingesteld op 1, wordt gecontroleerd of het bestand vervalt. Als het bestand volgens de set-TTL is verlopen, worden geen andere controles uitgevoerd en wordt het bestand in de cache opnieuw opgevraagd vanaf de back-end.
2. Als het bestand niet is verlopen, of `/enableTTL` wordt niet gevormd, dan worden de standaardregels van de geheim voorgeheugenongeldigverklaring toegepast zoals die regels die door worden geplaatst [/statfileslevel](#invalidating-files-by-folder-level) en [/invalidate](#automatically-invalidating-cached-files). Deze stroom betekent dat Dispatcher bestanden waarvoor de TTL nog niet is verlopen, ongeldig kan maken.

Deze nieuwe implementatie steunt gebruiksgevallen waar de dossiers langere TTL (bijvoorbeeld, op CDN) hebben maar nog ongeldig kunnen zijn zelfs als TTL niet is verlopen. De Dispatcher geeft de voorkeur aan versheid van inhoud boven cache-hit verhouding.

Omgekeerd, voor het geval dat **alleen** de vervallogica die op een dossier wordt toegepast en dan reeks `/enableTTL` naar 1 en sluit dat bestand uit van het standaardmechanisme voor cachevalidatie. U kunt bijvoorbeeld:

* Als u het bestand wilt negeren, configureert u de [regels voor ongeldigverklaring](#automatically-invalidating-cached-files) in de cachesectie. In het onderstaande fragment worden alle bestanden weergegeven die eindigen in `.example.html` worden genegeerd en verlopen slechts wanneer geplaatst TTL is overgegaan.

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* De inhoudsstructuur zodanig ontwerpen dat u een hoge waarde kunt instellen [/statfilelevel](#invalidating-files-by-folder-level) het bestand wordt dus niet automatisch ongeldig gemaakt.

Zo zorgt u ervoor dat `.stat` de bestandsinvalidatie wordt niet gebruikt en alleen de vervaldatum van TTL is actief voor de opgegeven bestanden.

>[!NOTE]
>
>Onthoud dat instelling `/enableTTL` aan 1 laat TTL caching slechts op de Dispatcher kant toe. Als dusdanig, wordt de informatie van TTL in het extra dossier (zie hierboven) niet verstrekt aan een andere gebruikersagent die om zo een dossiertype van de Dispatcher verzoekt. Als u caching kopballen aan stroomafwaartse systemen zoals CDN of browser wilt verstrekken, zou u moeten vormen `/cache/headers` van toepassing.

>[!NOTE]
>
>Deze functie is beschikbaar in versie **4.1.11.** of later van de Dispatcher.

## Load Balancing configureren - /statistics {#configuring-load-balancing-statistics}

De `/statistics` in deze sectie worden categorieën bestanden gedefinieerd waarvoor Dispatcher de responsiviteit van elke rendermethode scoort. Dispatcher gebruikt de scores om te bepalen welke renderen om een aanvraag te verzenden.

Elke categorie die u maakt, definieert een globaal patroon. Dispatcher vergelijkt de URI van de aangevraagde inhoud met deze patronen om de categorie van de gevraagde inhoud te bepalen:

* De volgorde van de categorieën bepaalt de volgorde waarin ze worden vergeleken met de URI.
* Het eerste categoriepatroon dat overeenkomt met de URI is de categorie van het bestand. Er worden niet meer categoriepatronen geëvalueerd.

Dispatcher ondersteunt maximaal acht statistische categorieën. Als u meer dan acht categorieën definieert, worden alleen de eerste 8 gebruikt.

**Selectie renderen**

Telkens wanneer Dispatcher een teruggegeven pagina vereist, gebruikt het het volgende algoritme om terug te selecteren:

1. Als de aanvraag de rendernaam bevat in een `renderid` cookie, Dispatcher gebruikt die rendering.
1. Als de aanvraag geen `renderid` cookie, Dispatcher vergelijkt de renderstatistieken:

   1. Dispatcher bepaalt de categorie van de aanvraag-URI.
   1. Dispatcher bepaalt welke rendermethode de laagste responsscore voor die categorie heeft en selecteert die rendermethode.

1. Als er nog geen renderbewerking is geselecteerd, gebruikt u de eerste renderbewerking in de lijst.

De score voor de rendercategorie is gebaseerd op vorige responstijden en eerdere mislukte en succesvolle verbindingen die Dispatcher probeert uit te voeren. Voor elke poging, wordt de score voor de categorie van gevraagde URI bijgewerkt.

>[!NOTE]
>
>Als u geen taakverdeling gebruikt, kunt u deze sectie weglaten.

### Categorieën statistieken definiëren {#defining-statistics-categories}

Definieer een categorie voor elk type document waarvoor u statistieken wilt bijhouden voor de renderselectie. De `/statistics` sectie bevat een `/categories` sectie. Als u een categorie wilt definiëren, voegt u een regel onder de `/categories` sectie met de volgende indeling:

`/name { /glob "pattern"}`

De categorie `name` moet uniek zijn voor het bedrijf. De `pattern` wordt beschreven in [Patronen ontwerpen voor globale eigenschappen](#designing-patterns-for-glob-properties) sectie.

Om de categorie van URI te bepalen, vergelijkt de Dispatcher URI met elk categoriepatroon tot een gelijke wordt gevonden. Dispatcher begint met de eerste categorie in de lijst en gaat in de juiste volgorde verder. Plaats daarom eerst categorieën met specifiekere patronen.

Bijvoorbeeld, verzend het gebrek `dispatcher.any` wordt een categorie HTML en een categorie Overige gedefinieerd. De categorie HTML is specifieker en verschijnt dus eerst:

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

De `/unavailablePenalty` het bezit plaatst de tijd (in tiende van een seconde) die op teruggeeft statistieken wordt toegepast wanneer een verbinding aan teruggeeft ontbreekt. De verzender voegt de tijd aan de statistiekcategorie toe die gevraagde URI aanpast.

Bijvoorbeeld, wordt de sanctie toegepast wanneer de verbinding TCP/IP aan aangewezen hostname/haven niet kan worden gevestigd, of omdat AEM niet loopt (en niet luistert) of wegens een netwerk-gerelateerd probleem.

De `/unavailablePenalty` eigenschap is een rechtstreeks onderliggend element van het `/farm` deel (een verwant van de `/statistics` ).

Indien niet `/unavailablePenalty` eigenschap bestaat, een waarde van `"1"` wordt gebruikt.

```xml
/unavailablePenalty "1"
```

## Identificeer een Vaste Omslag van de Verbinding - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

De `/stickyConnectionsFor` eigenschap definieert een map met plakke documenten. Deze eigenschap wordt benaderd via de URL. Dispatcher verzendt alle aanvragen, van één gebruiker in deze map, naar dezelfde renderinstantie. De stevige verbindingen zorgen ervoor dat de zittingsgegevens voor alle documenten aanwezig en verenigbaar zijn. Dit mechanisme gebruikt het `renderid` cookie.

In het volgende voorbeeld wordt een kleverige verbinding met de map /products gedefinieerd:

```xml
/stickyConnectionsFor "/products"
```

Wanneer een pagina bestaat uit inhoud van verschillende inhoudsknooppunten, neemt u de `/paths` eigenschap die de paden naar de inhoud bevat. Een pagina bevat bijvoorbeeld inhoud van `/content/image`, `/content/video`, en `/var/files/pdfs`. De volgende configuratie laat kleverige verbindingen voor alle inhoud op de pagina toe:

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

Wanneer kleverige verbindingen worden toegelaten, plaatst de module van de Verzender `renderid` cookie. Deze cookie bevat niet de `httponly` markering, die moet worden toegevoegd om de veiligheid te verbeteren. U voegt de `httponly` markeren door de `httpOnly` eigenschap in de `/stickyConnections` knooppunt van een `dispatcher.any` configuratiebestand. De waarde van de eigenschap (ofwel `0` of `1`) bepaalt of de `renderid` cookie `HttpOnly` toegevoegd kenmerk. De standaardwaarde is `0`, wat betekent dat het kenmerk niet wordt toegevoegd.

Voor meer informatie over de `httponly` markering, lezen [deze pagina](https://owasp.org/www-community/HttpOnly).

### `secure` {#secure}

Wanneer kleverige verbindingen worden toegelaten, plaatst de module van de Verzender `renderid` cookie. Deze cookie bevat niet de `secure` markering, die moet worden toegevoegd om de veiligheid te verbeteren. U voegt de `secure` markering die de `secure` eigenschap in de `/stickyConnections` knooppunt van een `dispatcher.any` configuratiebestand. De waarde van de eigenschap (ofwel `0` of `1`) bepaalt of de `renderid` cookie `secure` toegevoegd kenmerk. De standaardwaarde is `0`, wat betekent dat het kenmerk wordt toegevoegd **indien** het binnenkomende verzoek is veilig. Als de waarde is ingesteld op `1`, dan wordt de veilige vlag toegevoegd ongeacht of het inkomende verzoek veilig of niet is.

## Renderfouten afhandelen {#handling-render-connection-errors}

Vorm het gedrag van de Verzender wanneer teruggeeft de server een fout 500 terugkeert, of niet beschikbaar is.

### Een pagina voor een health check opgeven {#specifying-a-health-check-page}

Gebruik de `/health_check` eigenschap om een URL op te geven die wordt gecontroleerd wanneer een 500-statuscode plaatsvindt. Als deze pagina ook een 500 statuscode terugkeert, wordt de instantie beschouwd als niet beschikbaar en een configureerbare tijd ( `/unavailablePenalty`) wordt toegepast op de renderbewerking voordat opnieuw wordt geprobeerd.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### De vertraging voor opnieuw proberen van pagina opgeven {#specifying-the-page-retry-delay}

De `/retryDelay` het bezit plaatst de tijd (in seconden) dat de Verzender tussen ronde van verbindingspogingen met het landbouwbedrijf teruggeeft. Voor elke ronde, is het maximumaantal tijden de Verzender probeert een verbinding aan terug te geven het aantal teruggeeft in het landbouwbedrijf.

Dispatcher gebruikt de waarde van `"1"` indien `/retryDelay` is niet expliciet gedefinieerd. De standaardwaarde is geschikt.

```xml
/retryDelay "1"
```

### Het aantal pogingen configureren {#configuring-the-number-of-retries}

De `/numberOfRetries` eigenschap stelt het maximumaantal ronde verbindingspogingen in dat Dispatcher met de renders uitvoert. Als Dispatcher na dit aantal keren geen verbinding kan maken met een renderbewerking, retourneert Dispatcher een mislukte reactie.

Voor elke ronde, is het maximumaantal tijden de Verzender probeert een verbinding aan terug te geven het aantal teruggeeft in het landbouwbedrijf. Daarom is het maximumaantal tijden dat de Verzender een verbinding probeert ( `/numberOfRetries`) x (het aantal renderingen).

Als de waarde niet expliciet wordt gedefinieerd, is de standaardwaarde `5`.

```xml
/numberOfRetries "5"
```

### Het mechanisme Failover gebruiken {#using-the-failover-mechanism}

Om verzoeken aan verschillende terug te sturen geeft terug wanneer het originele verzoek ontbreekt, laat het failovermechanisme op uw landbouwbedrijf van de Verzender toe. Wanneer failover wordt toegelaten, heeft de Dispatcher het volgende gedrag:

* Wanneer een verzoek aan teruggeeft HTTP status 503 (UNAVAILABLE) terugkeert, verzendt de Dispatcher het verzoek naar verschillend teruggeven.
* Wanneer een verzoek aan teruggeeft de status van HTTP 50x (buiten 503) terugkeert, verzendt de Dispatcher een verzoek voor de pagina die voor wordt gevormd `health_check` eigenschap.
   * Als de gezondheidscontrole 500 (INTERNAL_SERVER_ERROR) terugkeert, verzendt de Dispatcher het originele verzoek naar verschillende teruggeven.
   * Als de gezondheidscontrole HTTP status 200 terugkeert, keert de Dispatcher de aanvankelijke fout van HTTP 500 aan de cliënt terug.

Om failover toe te laten, voeg de volgende lijn aan het landbouwbedrijf (of de website) toe:

```xml
/failover "1"
```

>[!NOTE]
>
>Om HTTP- verzoeken opnieuw te proberen die een lichaam bevatten, verzendt de Dispatcher een `Expect: 100-continue` verzoek kopbal aan teruggeven alvorens de daadwerkelijke inhoud te spoolen. CQ 5.5 met CQSE beantwoordt dan onmiddellijk met of 100 (CONTINUE) of een foutencode. Ook andere servlet-containers worden ondersteund.

## Onderbrekingsfouten negeren - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Deze optie is niet nodig. Gebruik dit alleen wanneer u de volgende logberichten ziet:
>
>`Error while reading response: Interrupted system call`

Om het even welk systeem georiënteerde dossiervraag kan worden onderbroken `EINTR` als het object van de systeemaanroep zich op een extern systeem bevindt dat via NFS wordt benaderd. Of deze systeemvraag uit kan tijd of worden onderbroken is gebaseerd op hoe het onderliggende dossiersysteem op de lokale machine werd opgezet.

Gebruik de `/ignoreEINTR` parameter als uw instantie zulk een configuratie heeft en het logboek het volgende bericht bevat:

`Error while reading response: Interrupted system call`

Intern, leest de Dispatcher de reactie van de verre server (namelijk AEM) gebruikend een lijn die als kan worden vertegenwoordigd:

```text
while (response not finished) {  
read more data  
}
```

Dergelijke berichten kunnen worden geproduceerd wanneer `EINTR` komt voor in &quot; `read more data`&quot; en worden veroorzaakt door de ontvangst van een signaal voordat gegevens werden ontvangen.

Als u dergelijke onderbrekingen wilt negeren, kunt u de volgende parameter toevoegen aan `dispatcher.any` (voor `/farms`):

`/ignoreEINTR "1"`

Instelling `/ignoreEINTR` tot `"1"` zorgt ervoor dat Dispatcher blijft proberen gegevens te lezen totdat de volledige reactie is gelezen. De standaardwaarde is `0` en deactiveert de optie.

## Patronen ontwerpen voor globale eigenschappen {#designing-patterns-for-glob-properties}

Verschillende secties in het Dispatcher-configuratiebestand kunnen worden gebruikt `glob` eigenschappen als selectiecriteria voor cliëntverzoeken. De waarden van `glob` eigenschappen zijn patronen die de Verzender met een aspect van het verzoek, zoals de weg van het gevraagde middel, of het IP adres van de cliënt vergelijkt. De items in het dialoogvenster `/filter` sectiegebruik `glob` patronen om de paden te identificeren van de pagina&#39;s waarop Dispatcher reageert of deze weigert.

De `glob` Deze waarden kunnen jokertekens en alfanumerieke tekens bevatten om het patroon te definiëren.

| Jokerteken | Beschrijving | Voorbeelden |
|--- |--- |--- |
| `*` | Komt overeen met nul of meer aaneengesloten instanties van een willekeurig teken in de tekenreeks. Het uiteindelijke teken van de overeenkomst wordt bepaald door een van de volgende situaties: <br/>Een teken in de tekenreeks komt overeen met het volgende teken in het patroon en het patroonteken heeft de volgende kenmerken:<br/><ul><li>Geen *</li><li>Niet een?</li><li>Een letterlijk teken (inclusief een spatie) of een tekenklasse.</li><li>Het einde van het patroon is bereikt.</li></ul>Binnen een tekenklasse wordt het teken letterlijk geïnterpreteerd. | `*/geo*` Komt overeen met elke pagina onder de `/content/geometrixx` en de `/content/geometrixx-outdoors` knooppunt. De volgende HTTP-aanvragen komen overeen met het globale patroon: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>Komt overeen met elke pagina onder de `/content/geometrixx-outdoors` knooppunt. De volgende HTTP-aanvraag komt bijvoorbeeld overeen met het glob-patroon: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Komt overeen met elk willekeurig enkel teken. Gebruik externe tekenklassen. Binnen een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | `*outdoors/??/*`<br/> Komt overeen met de pagina&#39;s voor elke taal in de geometrixx-outdoorsite. De volgende HTTP-aanvraag komt bijvoorbeeld overeen met het glob-patroon: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Het volgende verzoek komt niet overeen met het glob-patroon: <br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | Hiermee wordt het begin en einde van een tekenklasse gedemonstreerd. Tekenklassen kunnen een of meer tekenbereiken en enkele tekens bevatten.<br/>Een overeenkomst treedt op als het doelteken overeenkomt met een van de tekens in de tekenklasse of binnen een gedefinieerd bereik.<br/>Als de accolade sluiten niet is opgenomen, resulteert het patroon niet in overeenkomende waarden. | `*[o]men.html*`<br/> Komt overeen met de volgende HTTP-aanvraag:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Komt niet overeen met de volgende HTTP-aanvraag:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>Komt overeen met de volgende HTTP-aanvragen: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Geeft een tekenbereik aan. Voor gebruik in tekenklassen. Buiten een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | `*[m-p]men.html*` Komt overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Komt niet overeen met de volgende HTTP-aanvraag:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Hiermee wordt het volgende teken of de volgende tekenklasse genegeerd. Alleen gebruiken voor negerende tekens en tekenbereiken binnen tekenklassen. Equivalent met de `^ wildcard`. <br/>Buiten een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | `*[!o]men.html*`<br/> Komt overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Komt niet overeen met de volgende HTTP-aanvraag: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Komt niet overeen met de volgende HTTP-aanvraag:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` of `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Hiermee wordt het volgende teken- of tekenbereik genegeerd. Alleen gebruiken voor negatie van tekens en tekenbereiken binnen tekenklassen. Equivalent met de `!` jokerteken. <br/>Buiten een tekenklasse wordt dit teken letterlijk geïnterpreteerd. | De voorbeelden van de `!` jokerteken is van toepassing en vervangt het `!` tekens in voorbeeldpatronen met `^` tekens. |


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

In de webserverconfiguratie kunt u instellen:

* De locatie van het logbestand voor de verzending.
* Het logniveau.

Raadpleeg de documentatie bij de webserver en het Lees mij-bestand van de Dispatcher-instantie voor meer informatie.

**Apache geroteerd / Logbestanden met pijplijnen**

Als u een **Apache** Webserver, kunt u de standaardfunctionaliteit voor geroteerde logboeken, of door buizen geleide logboeken, of allebei gebruiken. Bijvoorbeeld met behulp van stammen met buizen:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Deze functionaliteit roteert automatisch:

* het Dispatcher-logbestand, met een tijdstempel in de extensie (`logs/dispatcher.log%Y%m%d`).
* wekelijks (60 x 60 x 24 x 7 = 604800 seconden).

Raadpleeg de documentatie bij de Apache-webserver over Log Rotation en Piped Logs. Bijvoorbeeld: [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Na installatie, is het standaardlogboekniveau hoog (namelijk niveau 3 = zuivert), zodat de Dispatcher alle fouten en waarschuwingen registreert. Dit niveau is handig in de beginstadia.
>
>Een dergelijk niveau vereist echter meer middelen. Wanneer de Dispatcher probleemloos werkt *volgens uw vereisten* kunt u het logniveau verlagen.

### Trackregistratie {#trace-logging}

Naast andere verbeteringen voor de Dispatcher, introduceert versie 4.2.0 ook Tracks Logging.

Deze capaciteit is een hoger niveau dan Debug registreren die extra informatie in de logboeken toont. Er wordt logbestanden toegevoegd voor:

* De waarden van de doorgestuurde kopteksten;
* De regel die wordt toegepast voor een bepaalde handeling.

U kunt de Logboekregistratie van het Spoor toelaten door het logboekniveau te plaatsen aan `4` in uw webserver.

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

En een gebeurtenis die wordt geregistreerd wanneer een dossier dat een het blokkeren regel aanpast wordt gevraagd:

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## Basisbewerking bevestigen {#confirming-basic-operation}

U kunt de volgende stappen gebruiken om de basisbewerking en interactie van de webserver, de AEM Dispatcher en de instantie te bevestigen:

1. Stel de `loglevel` tot `3`.

1. Start de webserver. Zo start u ook de Dispatcher.
1. Start de AEM.
1. Controleer het logboek en de foutendossiers voor uw Webserver en de Verzender.
   * Afhankelijk van uw webserver worden berichten weergegeven zoals:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` en
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surf op de website via de webserver. Bevestig dat de inhoud naar wens wordt weergegeven.\
   Bijvoorbeeld op een lokale installatie waar AEM op haven loopt `4502` en de webserver op `80` toegang tot de console van Websites gebruikend allebei:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * De resultaten moeten identiek zijn. Bevestig toegang tot andere pagina&#39;s met het zelfde mechanisme.

1. Controleer of de cachemap wordt gevuld.
1. Activeer een pagina om te controleren of de cache correct wordt leeggemaakt.
1. Als alles correct werkt, kunt u de `loglevel` tot `0`.

## Meerdere verzenders gebruiken {#using-multiple-dispatchers}

In complexe instellingen kunt u meerdere verzenders gebruiken. U kunt bijvoorbeeld het volgende gebruiken:

* één verzender om een website op het Intranet te publiceren
* een tweede verzender, onder een ander adres en met verschillende beveiligingsinstellingen, om dezelfde inhoud op internet te publiceren.

In een dergelijk geval, zorg ervoor dat elk verzoek door slechts één Dispatcher gaat. Een verzender behandelt geen verzoeken die afkomstig zijn van een andere verzender. Zorg er daarom voor dat beide verzenders de AEM website rechtstreeks openen.

## Foutopsporing {#debugging}

Bij het toevoegen van de koptekst `X-Dispatcher-Info` op een verzoek, beantwoordt de Ontvanger of het doel in het voorgeheugen onder was gebracht, van cachegeheugen teruggekeerd, of helemaal niet in het cachegeheugen kon. De responsheader `X-Cache-Info` bevat deze gegevens in leesbare vorm. U kunt deze antwoordkopballen gebruiken om kwesties te zuiveren die reacties impliceren door Dispatcher in het voorgeheugen worden opgenomen.

Deze functionaliteit is niet standaard ingeschakeld, dus voor de responsheader `X-Cache-Info` het bedrijf moet de volgende gegevens bevatten om daarin te worden opgenomen:

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

Ook de `X-Dispatcher-Info` header heeft geen waarde nodig, maar als je `curl` voor het testen moet u een waarde opgeven die naar de koptekst wordt verzonden, zoals:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Hieronder ziet u een lijst met de antwoordheaders die `X-Dispatcher-Info` retourneert:

* **caching**\
  Het doelbestand bevindt zich in de cache en de Dispatcher heeft vastgesteld dat het geldig is om het te leveren.
* **caching**\
  Het doelbestand bevindt zich niet in de cache en de Dispatcher heeft bepaald dat het geldig is om de uitvoer in cache te plaatsen en te leveren.
* **caching: statusbestand is recenter**
Het doelbestand bevindt zich in de cache, maar wordt ongeldig gemaakt door een recentere statusbestand. Dispatcher verwijdert het doelbestand, maakt het opnieuw van de uitvoer en levert het.
* **niet in cache geplaatst: geen hoofdmap van document**
De configuratie van het landbouwbedrijf bevat geen documentwortel (configuratieelement `cache.docroot`).
* **niet in cache geplaatst: pad naar cachebestand is te lang**\
  Het doelbestand - de samenvoeging van het hoofdbestand van het document en het URL-bestand - overschrijdt de langst mogelijke bestandsnaam op het systeem.
* **niet in cache geplaatst: tijdelijk bestandspad is te lang**\
  De sjabloon voor tijdelijke bestandsnamen overschrijdt de langst mogelijke bestandsnaam op het systeem. Dispatcher maakt eerst een tijdelijk bestand voordat het in de cache opgeslagen bestand wordt gemaakt of overschreven. De tijdelijke bestandsnaam is de naam van het doelbestand met de tekens `_YYYYXXXXXX` toegevoegd, waarbij de `Y` en `X` worden vervangen om een unieke naam te maken.
* **niet in cache geplaatst: verzoek-URL heeft geen extensie**\
  De aanvraag-URL heeft geen extensie of er is een pad dat volgt op de bestandsextensie, bijvoorbeeld: `/test.html/a/path`.
* **niet cacheable: verzoek was geen GET of HEAD**
De HTTP-methode is geen GET of HEAD. Dispatcher gaat ervan uit dat de uitvoer dynamische gegevens bevat die niet in de cache mogen worden opgeslagen.
* **niet in cache geplaatst: verzoek bevat een queryreeks**\
  Het verzoek bevatte een queryreeks. Dispatcher veronderstelt dat de output van het gegeven vraagkoord afhangt en daarom niet geheim voorgeheugen plaatst.
* **niet in cache geplaatst: sessiemanager is niet geverifieerd**\
  Het geheime voorgeheugen van het landbouwbedrijf wordt geregeerd door een zittingsmanager (de configuratie bevat een `sessionmanagement` knoop) en het verzoek bevatte niet de aangewezen authentificatieinformatie.
* **niet in cache te plaatsen: aanvraag bevat vergunning**\
  Het landbouwbedrijf wordt niet toegestaan output ( `allowAuthorized 0`) en het verzoek bevat verificatiegegevens.
* **niet in cache geplaatst: doel is een map**\
  Het doelbestand is een map. Deze locatie kan wijzen op een conceptuele fout, waarbij een URL en een subURL beide cacheable-uitvoer bevatten. Als een aanvraag bijvoorbeeld `/test.html/a/file.ext` komt eerst en bevat cacheable output, kan de Dispatcher niet de output van een volgend verzoek in het voorgeheugen onderbrengen aan `/test.html`.
* **niet in cache geplaatst: verzoek-URL heeft een slash aan het einde**\
  De aanvraag-URL heeft een slash.
* **niet in cache geplaatst: verzoek-URL niet in cacheregels**\
  De het geheime voorgeheugenregels van het landbouwbedrijf ontkennen uitdrukkelijk caching de output van één of ander verzoek URL.
* **niet in cache geplaatst: controle van de vergunning geweigerd toegang**\
  De de vergunningscontrole van het landbouwbedrijf ontkende toegang tot het caching dossier.
* **niet in cache geplaatst: sessie is niet geldig**
Het geheime voorgeheugen van het landbouwbedrijf wordt geregeerd door een zittingsmanager (de configuratie bevat een `sessionmanagement` en de sessie van de gebruiker is niet of niet meer geldig.
* **niet in cache geplaatst: reactie bevat`no_cache`**
De externe server heeft een `Dispatcher: no_cache` header, waarbij de Dispatcher wordt verboden de uitvoer in cache te plaatsen.
* **niet in cache geplaatst: lengte van reactieinhoud is nul**
De lengte van de reactie is nul. De verzender maakt geen bestand met een lengte van nul.

