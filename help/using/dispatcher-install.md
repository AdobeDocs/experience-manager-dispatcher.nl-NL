---
title: Dispatcher installeren
description: Leer hoe u de Dispatcher-module installeert op Microsoft&reg; Internet Information Server, Apache Web Server en Sun Java&trade; Web Server-iPlanet.
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '3748'
ht-degree: 0%

---

# Dispatcher installeren {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

Gebruik de [ pagina van de Nota&#39;s van de Versie van Dispatcher ](release-notes.md) om het recentste de installatiedossier van Dispatcher voor uw werkend systeem en Webserver te verkrijgen. Dispatcher-releasenummers zijn onafhankelijk van de Adobe Experience Manager-releasenummers en zijn compatibel met Adobe Experience Manager 6.x-, 5.x- en Adobe CQ 5.x-releases.

>[!NOTE]
>
>Voor Adobe Experience Manager 6.5 is Dispatcher versie 4.3.2 of hoger vereist. Dat gezegd hebbende, zijn de versies van Dispatcher onafhankelijk van AEM, bijvoorbeeld, is Dispatcher versie 4.3.2 ook compatibel met Adobe Experience Manager 6.4.

De volgende naamgevingsconventie voor bestanden wordt gebruikt:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Bijvoorbeeld, bevat het `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` dossier Dispatcher versie 4.3.1 voor een Apache 2.4 Webserver die op Linux® i686 loopt en gebruikend het **teer** formaat verpakt is.

De volgende tabel bevat de id van de webserver die wordt gebruikt in bestandsnamen voor elke webserver:

| Webserver | Installatiekit |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4** -&lt;other parameters> |
| Microsoft® Internet Information Server 7.5, 8, 8.5, 10 | dispatcher-**is** -&lt;other parameters> |
| Sun Java™ Web Server iPlanet | verzender-**ns** -&lt;other parameters> |

>[!CAUTION]
>
>Installeer de nieuwste versie van Dispatcher die beschikbaar is voor uw platform. Voer jaarlijks een upgrade uit op uw Dispatcher-exemplaar om de nieuwste versie te gebruiken om te profiteren van productverbeteringen.

>[!NOTE]
>
>Klanten die specifiek een upgrade uitvoeren van versie 4.3.3 naar versie 4.3.4, moeten een ander gedrag opmerken voor de manier waarop in cache geplaatste koppen worden ingesteld voor niet-cachebare inhoud. Om meer over deze verandering te lezen, zie de [ pagina van de Nota&#39;s van de Versie ](/help/using/release-notes.md#nov).

Elk archief bevat de volgende bestanden:

* de Dispatcher-modules
* een voorbeeldconfiguratiebestand
* het README-bestand met installatie-instructies en informatie van het laatste moment
* het dossier van WIJZIGINGEN dat van kwesties een lijst maakt die in huidige en vroegere versies worden opgelost

>[!NOTE]
>
>Controleer het README-bestand op eventuele laatste wijzigingen/platformspecifieke opmerkingen voordat u de installatie start.

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific Web server installation procedures.</p>

 -->

## Microsoft® Internet Information Server {#microsoft-internet-information-server}

Voor informatie over hoe te om deze server van Web te installeren, zie de volgende middelen:

* Microsoft®&#39;s eigen documentatie op de Internet Information Server
* [ &quot;De officiële plaats van Microsoft® IIS&quot;](https://www.iis.net/)

### Vereiste IIS-componenten {#required-iis-components}

IIS versies 8.5 en 10 vereisen dat de volgende componenten IIS worden geïnstalleerd:

* ISAPI-extensies

Ook, moet u de de serverrol van het Web (IIS) toevoegen. Gebruik Serverbeheer om de rol en de componenten toe te voegen.

## Microsoft® IIS - De Dispatcher-module installeren {#microsoft-iis-installing-the-dispatcher-module}

Het vereiste archief voor het Microsoft® Internet Information System is:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

Het ZIP-bestand bevat de volgende bestanden:

| Bestand | Beschrijving |
|--- |--- |
| `disp_iis.dll` | Het Dispatcher-bibliotheekbestand voor dynamische koppelingen. |
| `disp_iis.ini` | Het dossier van de configuratie voor IIS. Dit voorbeeld kan met uw vereisten worden bijgewerkt. **Nota**: Het ini dossier moet de zelfde naam-wortel zoals dll hebben. |
| `dispatcher.any` | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| `author_dispatcher.any` | Een voorbeeldconfiguratiebestand voor Dispatcher dat werkt met de auteurinstantie. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Nota**: Controleer dit dossier alvorens de installatie te beginnen. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en eerdere versies. |

Gebruik de volgende procedure om de Dispatcher-bestanden naar de juiste locatie te kopiëren.

1. Gebruik Windows Explorer om de map `<IIS_INSTALLDIR>/Scripts` te maken, bijvoorbeeld `C:\inetpub\Scripts` .

1. Pak de volgende bestanden uit het Dispatcher-pakket uit in deze map Scripts:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * Een van de volgende bestanden is afhankelijk van het feit of de Dispatcher werkt met een AEM instantie van de auteur of een publicatie-instantie:
      * Instantie van auteur: `author_dispatcher.any`
      * Publish-instantie: `dispatcher.any`

## Microsoft® IIS - Dispatcher INI-bestand configureren {#microsoft-iis-configure-the-dispatcher-ini-file}

Bewerk het `disp_iis.ini` -bestand om de Dispatcher-installatie te configureren. De basisindeling van het `.ini` -bestand is als volgt:

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

In de volgende tabel wordt elke eigenschap beschreven.

| Parameter | Beschrijving |
|--- |--- |
| `configpath` | De locatie van `dispatcher.any` in het lokale bestandssysteem (absoluut pad). |
| `logfile` | De locatie van het `dispatcher.log` -bestand. Als deze plaats niet wordt geplaatst, dan gaan de logboekberichten naar het de gebeurtenislogboek van Vensters. |
| `loglevel` | Definieert het logniveau dat wordt gebruikt voor het uitvoeren van berichten naar het gebeurtenislogboek. De volgende waarden kunnen op het logboekniveau voor het logboekdossier worden gespecificeerd: <br/> 0 - foutenmeldingen slechts. <br/> 1 - fouten en waarschuwingen. <br/> 2 - fouten, waarschuwingen, en informatieberichten <br/> 3 - fouten, waarschuwingen, informatieberichten, en zuivert berichten. <br/>**Nota**: Plaats het logboekniveau aan 3 tijdens installatie en het testen, toen aan 0 wanneer het lopen in een productiemilieu. |
| `replaceauthorization` | Geeft aan hoe machtigingsheaders in de HTTP-aanvraag worden verwerkt. De volgende waarden zijn geldig:<br/> 0 - de kopballen van de Vergunning worden niet gewijzigd. <br/> 1 - vervangt om het even welke kopbal genoemd &quot;Vergunning&quot;buiten &quot;Basis&quot;met zijn `Basic <IIS:LOGON\_USER>` equivalent.<br/> |
| `servervariables` | Definieert hoe servervariabelen worden verwerkt.<br/> 0 - De IIS servervariabelen worden niet verzonden naar Dispatcher of AEM. <br/> 1 - alle Iis servervariabelen (zoals `LOGON\_USER, QUERY\_STRING, ...`) worden verzonden naar Dispatcher, samen met de verzoekkopballen (en ook naar de AEM instantie als niet caching).  <br/> de variabelen van de Server omvatten `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` en vele anderen. Zie de documentatie IIS voor de volledige lijst van variabelen, met details. |
| `enable_chunked_transfer` | Bepaalt of (1) of (0) segmentoverdracht voor de cliëntreactie toelaten onbruikbaar te maken. De standaardwaarde is 0. |

Een voorbeeldconfiguratie:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Microsoft® IIS configureren {#configuring-microsoft-iis}

Configureer IIS om de Dispatcher ISAPI-module te integreren. In IIS gebruikt u toewijzing van jokertekens voor toepassingen.

### Het vormen Anonieme Toegang - IIS 8.5 en 10 {#configuring-anonymous-access-iis-and}

De standaard Flush replicatieagent op de instantie van de Auteur wordt gevormd zodat het geen veiligheidsgeloofsbrieven met flush verzoeken verzendt. Daarom moet de website waarop u het Dispatcher-cachegeheugen gebruikt anonieme toegang toestaan.

Als uw website een authentificatiemethode gebruikt, moet de Flush replicatieagent dienovereenkomstig worden gevormd.

1. Open IIS Manager en selecteer de website die u als Dispatcher-cache gebruikt.
1. Gebruikend de wijze van de Mening van Eigenschappen, in de sectie IIS tweemaal klikken Authentificatie.
1. Als Anonieme verificatie niet is ingeschakeld, selecteert u Anonieme verificatie en klikt u in het gedeelte Handelingen op Inschakelen.

### De Dispatcher ISAPI Module integreren - IIS 8.5 en 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Gebruik de volgende procedure om de Dispatcher ISAPI module aan IIS toe te voegen.

1. Open IIS Manager.
1. Selecteer de website die u als Dispatcher Cache gebruikt.
1. Gebruikend de wijze van de Mening van Eigenschappen, in de sectie IIS tweemaal klikken Handler Mappings.
1. Klik in het deelvenster Handelingen van de pagina Handler Mappings op Toevoegen van jokerscript, voeg de volgende eigenschapswaarden toe en klik op OK:

   * Aanvraagpad: &#42;
   * Uitvoerbaar: het absolute pad van het bestand disp_is.dll, bijvoorbeeld `C:\inetpub\Scripts\disp_iis.dll` .
   * Naam: een beschrijvende naam voor de handlertoewijzing, bijvoorbeeld `Dispatcher` .

1. In de dialoogdoos die verschijnt, om de bibliotheek disp_is.dll aan de lijst van Beperkingen ISAPI en CGI toe te voegen, klik ja ****.

   Voor IIS 7.0 en 7.5, is de configuratie volledig. Ga met de resterende stappen verder als u IIS 8.0 vormt.

1. (IIS 8.0) Selecteer in de lijst Handler Mappings de afbeelding die u hebt gemaakt en klik in het gedeelte Handelingen op Bewerken.
1. (IIS 8.0) Klik in het dialoogvenster Scriptkaart bewerken op de knop Beperkingen aanvragen.
1. (IIS 8.0) om ervoor te zorgen dat de manager voor dossiers en omslagen wordt gebruikt die nog niet in het voorgeheugen onder worden gebracht, schrap **aanhaalt slechts Behandelaar als het Verzoek aan** wordt toegewezen. Klik **OK**.
1. (IIS 8.0) Klik in het dialoogvenster Scripttoewijzing bewerken op OK.

### Het vormen Toegang tot het geheime voorgeheugen - IIS 8.5 en 10 {#configuring-access-to-the-cache-iis-and}

Geef de standaardgebruiker van de App Pool schrijftoegang tot de map die wordt gebruikt als de Dispatcher-cache.

1. Klik met de rechtermuisknop op de hoofdmap van de website die u als Dispatcher-cache gebruikt en klik op Eigenschappen, zoals `C:\inetpub\wwwroot` .
1. Klik op het tabblad Beveiliging op Bewerken en klik vervolgens in het dialoogvenster Machtigingen op Toevoegen. Er wordt een dialoogvenster geopend waarin u gebruikersaccounts kunt selecteren. Klik op de knop Locaties, selecteer de naam van de computer en klik op OK.

   Zorg dat dit dialoogvenster geopend blijft terwijl u de volgende stap uitvoert.

1. in Manager IIS, selecteer de plaats IIS die u voor het geheime voorgeheugen van Dispatcher gebruikt, en op de rechterkant van het venster, klik Geavanceerde Montages.
1. Selecteer de waarde van het bezit van de Pool van de Toepassing en kopieer het aan het klembord.
1. Ga terug naar het geopende dialoogvenster. Typ `IIS AppPool\` in het vak Voer de objectnamen in die u wilt selecteren en plak vervolgens de inhoud van het klembord. De waarde moet er als volgt uitzien:

   `IIS AppPool\DefaultAppPool`

1. Klik op de knop Namen controleren. Klik op OK als Windows het gebruikersaccount oplost.
1. In het de dialoogvakje van Toestemmingen voor de omslag van Dispatcher, selecteer de rekening die u enkel toevoegde, laat alle toestemmingen voor de rekening **behalve Volledige Controle** toe en klik O.K. Klik op OK om het dialoogvenster Eigenschappen van map te sluiten.

### Registreren van het JSON Mime-type - IIS 8.5 en 10 {#registering-the-json-mime-type-iis-and}

Gebruik de volgende procedure om het JSON MIME-type te registreren wanneer u wilt dat de Dispatcher JSON-aanroepen toestaat.

1. Selecteer uw website in IIS Manager en gebruik de weergave Eigenschappen en dubbelklik op MIME-typen.
1. Als de JSON-extensie niet in de lijst voorkomt, klikt u in het deelvenster Handelingen op Toevoegen, voert u de volgende eigenschapswaarden in en klikt u op OK:

   * Bestandsnaamextensie: `.json`
   * MIME-type: `application/json`

### Het verborgen segment van de bin verwijderen - IIS 8.5 en 10 {#removing-the-bin-hidden-segment-iis-and}

Gebruik de volgende procedure om het verborgen segment `bin` te verwijderen. Websites die niet nieuw zijn, kunnen dit verborgen segment bevatten.

1. Selecteer in IIS Manager uw website en gebruik de weergave Functies en dubbelklik op Verzoek filteren.
1. Selecteer het `bin` -segment, klik op Verwijderen en klik in het bevestigingsdialoogvenster op Ja.

### Het registreren IIS Berichten aan een Dossier - IIS 8.5 en 10 {#logging-iis-messages-to-a-file-iis-and}

Gebruik de volgende procedure om Dispatcher-logberichten naar een logbestand te schrijven in plaats van naar het Windows-gebeurtenislogboek. Configureer de Dispatcher om het logbestand te gebruiken en geef IIS schrijftoegang tot het bestand.

1. Gebruik Windows Verkenner om een map met de naam `dispatcher` onder de logboekmap van de IIS-installatie te maken. Het pad van deze map voor een standaardinstallatie is `C:\inetpub\logs\dispatcher` .

1. Klik de omslag van Dispatcher met de rechtermuisknop aan en klik **Eigenschappen**.
1. Voor het lusje van de Veiligheid, geeft de klik **** uit.
1. In het de dialoogvakje van Toestemmingen, voegt de klik **** toe. Er wordt een dialoogvenster geopend waarin u gebruikersaccounts kunt selecteren. Klik op de knop Locaties, selecteer de naam van de computer en klik op OK.

   Zorg dat dit dialoogvenster geopend blijft terwijl u de volgende stap uitvoert.

1. in Manager IIS, selecteer de plaats IIS die u voor het geheime voorgeheugen van Dispatcher gebruikt, en op de rechterkant van het venster, klik Geavanceerde Montages.
1. Selecteer de waarde van het bezit van de Pool van de Toepassing en kopieer het aan het klembord.
1. Ga terug naar het geopende dialoogvenster. Typ `IIS AppPool\` in het vak Voer de objectnamen in die u wilt selecteren en plak vervolgens de inhoud van het klembord. De waarde moet er als volgt uitzien:

   `IIS AppPool\DefaultAppPool`

1. Klik op de knop Namen controleren. Klik op OK als Windows het gebruikersaccount oplost.
1. In het de dialoogvakje van Toestemmingen voor de omslag van Dispatcher, selecteer de rekening die u enkel toevoegde, laat alle toestemmingen voor de rekening **behalve Volledige Controle,** toe en klik O.K. Klik op OK om het dialoogvenster Eigenschappen van map te sluiten.
1. Open het `disp_iis.ini` -bestand met een teksteditor.
1. Als u de locatie van het logbestand wilt configureren, voegt u een tekstregel toe die lijkt op het volgende voorbeeld en slaat u het bestand op:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Volgende stappen {#next-steps}

Voordat u de Dispatcher kunt gaan gebruiken, moet u het volgende weten:

* [ vorm ](dispatcher-configuration.md) Dispatcher
* [ vorm AEM ](page-invalidate.md) om met Dispatcher te werken.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>De instructies voor installatie onder zowel **Vensters** als **UNIX®** zijn hier behandeld. Wees voorzichtig wanneer u de stappen uitvoert.

### Apache Web Server installeren {#installing-apache-web-server}

Voor Informatie over hoe te om een Server van het Web te installeren Apache leest de installatiehandleiding - of [ online ](https://httpd.apache.org/) of in de distributie.

>[!CAUTION]
>
>Als u een binair Apache-bestand maakt door de bronbestanden te compileren, schakelt u **`dynamic modules support`** in. Het toelaten van deze optie kan worden gedaan gebruikend om het even welke **- toelaten-gedeelde** opties. Neem minimaal de module `mod_so` op.
>
>Meer informatie vindt u in de installatiehandleiding van Apache Web Server.

Zie ook de Server van Apache HTTP [ Tips van de Veiligheid ](https://httpd.apache.org/docs/2.4/misc/security_tips.html) en [ Rapporten van de Veiligheid ](https://httpd.apache.org/security_report.html).

### Apache-webserver - Voeg de Dispatcher-module toe {#apache-web-server-add-the-dispatcher-module}

De Dispatcher is als volgt:

* **Vensters**: een Dynamische Bibliotheek van de Verbinding (DLL)
* **UNIX®**: Een Dynamisch Gedeeld Voorwerp (DSO)

De archiefbestanden van de installatie bevatten de volgende bestanden, afhankelijk van of u Windows of UNIX® hebt geselecteerd:

| Bestand | Beschrijving |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows: Het Dispatcher-bibliotheekbestand voor dynamische koppelingen. |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | UNIX®: Het bibliotheekbestand voor gezamenlijke Dispatcher-objecten. |
| mod_dispatcher.so | UNIX®: Een voorbeeldkoppeling. |
| http.conf.disp&lt;x> | Een voorbeeldconfiguratiebestand voor de Apache-server. |
| dispatcher.any | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Nota**: Controleer dit dossier alvorens de installatie te beginnen. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en vorige versie. |

Ga als volgt te werk om de Dispatcher aan uw Apache Web Server toe te voegen:

1. Plaats het Dispatcher-bestand in de juiste map in de Apache-module:

   * **Vensters**: Plaats `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **UNIX®**: bepaal de plaats van of de `<APACHE_ROOT>/libexec` of `<APACHE_ROOT>/modules` folder volgens uw installatie.\
     Kopieer `dispatcher-apache<options>.so` naar deze map.\
     Om het langetermijnonderhoud te vereenvoudigen, kunt u ook een symbolische koppeling met de naam `mod_dispatcher.so` maken naar de Dispatcher:\
     `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Kopieer het bestand dispatcher.any naar de map `<APACHE_ROOT>/conf` .

   **Nota:** u kunt dit dossier in een verschillende plaats plaatsen, zolang het bezit DispatcherLog van de module van Dispatcher dienovereenkomstig wordt gevormd. (Zie Dispatcher-specifieke configuratiegegevens verderop.)

### Apache Web Server - SELinux-eigenschappen configureren {#apache-web-server-configure-selinux-properties}

Als u Dispatcher uitvoert op Red Hat® Linux® Kernel 2.6 met SELinux ingeschakeld, kunt u foutberichten zoals deze weergeven in het Dispatcher-logbestand.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Deze fout is waarschijnlijk het gevolg van een ingeschakelde SELinux-beveiliging. Zo ja, voer de volgende taken uit:

* Configureer de SELinux-context van het Dispatcher-modulebestand.
* Schakel HTTP-scripts en -modules in om netwerkverbindingen te maken.
* Vorm de context SELinux van de docroot, waar de caching dossiers worden opgeslagen.

Voer de volgende opdrachten in een terminalvenster in, waarbij u `[path to the dispatcher.so file]` vervangt door het pad naar de Dispatcher-module die u op Apache Web Server hebt geïnstalleerd en *`path to the docroot`* door het pad waar de hoofdmap zich bevindt (bijvoorbeeld `/opt/cq/cache` ):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_rw_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_rw_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Apache Web Server configureren voor Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

De Apache Web Server moet worden geconfigureerd met `httpd.conf` . Zoek in de Dispatcher-installatiekit naar een voorbeeld-configuratiebestand met de naam `httpd.conf.disp<x>` .

Deze stappen zijn verplicht:

1. Navigeer naar `<APACHE_ROOT>/conf` .
1. Open `httpd.conf` voor het uitgeven.
1. De volgende configuratieingangen moeten, in de vermelde orde worden toegevoegd:

   * **LoadModule** om de module bij opstarten te laden.
   * Dispatcher-specifieke configuratieingangen, met inbegrip van **DispatcherConfig, DispatcherLog**, en **DispatcherLogLevel**.
   * **SetHandler** om Dispatcher te activeren. **LoadModule**.
   * **ModMimeUsePathInfo** om het gedrag van **te vormen mod_mime**.

1. (Optioneel) U wordt aangeraden de eigenaar van de map htdocs te wijzigen:

   * De Apache-server begint als root, maar de onderliggende processen beginnen als daemon (voor beveiligingsdoeleinden). DocumentRoot (`<APACHE_ROOT>/htdocs`) moet tot de gebruikersdaemon behoren:

     ```xml
     cd <APACHE_ROOT>  
     chown -R daemon:daemon htdocs
     ```

**LoadModule**

De volgende lijst maakt een lijst van voorbeelden die kunnen worden gebruikt; de nauwkeurige ingangen zijn volgens uw specifieke Server van het Web Apache:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| UNIX® (symbolische koppeling wordt gebruikt) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>De eerste parameter van elke instructie moet precies zo worden geschreven als in de bovenstaande voorbeelden.
>
>Zie de dossiers van de voorbeeldconfiguratie en de documentatie van de Server van het Web Apache voor volledige details over dit bevel worden verstrekt die.

**Dispatcher specifieke configuratieingangen**

De Dispatcher-specifieke configuratieingangen worden geplaatst na de ingang LoadModule. De volgende lijst maakt een lijst van een voorbeeldconfiguratie die voor zowel UNIX® als Vensters van toepassing is:

**Vensters &amp; UNIX®**

```
...
<IfModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

>[!NOTE]
>
>Klanten die specifiek een upgrade uitvoeren van versie 4.3.3 naar versie 4.3.4, moeten een ander gedrag opmerken voor de manier waarop in cache geplaatste koppen worden ingesteld voor niet-cachebare inhoud. Om meer over deze verandering te lezen, zie de [ pagina van de Nota&#39;s van de Versie ](/help/using/release-notes.md#nov).

De individuele configuratieparameters:

| Parameter | Beschrijving |
|--- |--- |
| DispatcherConfig | Locatie en naam van het Dispatcher-configuratiebestand. <br/> wanneer dit bezit in de belangrijkste serverconfiguratie is, erven alle virtuele gastheren de bezitswaarde. Virtuele hosts kunnen echter een eigenschap DispatcherConfig opnemen om de hoofdserverconfiguratie te overschrijven. |
| DispatcherLog | Locatie en naam van het logbestand. |
| DispatcherLogLevel | Het niveau van het logboek voor het logboekdossier: <br/> 0 - Fouten <br/> 1 - Waarschuwingen <br/> 2 - <br/> <br/>**Nota** zuiveren: Plaats het logboekniveau aan 3 tijdens installatie en het testen, toen aan 0 wanneer het lopen in een productiemilieu. |
| DispatcherNoServerHeader | *Deze parameter wordt afgekeurd en inefficiënt.*<br/><br/> Definieert de serverkoptekst die moet worden gebruikt: <br/><ul><li>ongedefinieerd of 0 - De HTTP-serverheader bevat de AEM versie. </li><li>1 - De header van de Apache-server wordt gebruikt.</li></ul> |
| DispatcherDeclineRoot | Bepaalt of om verzoeken aan de wortel &quot;/&quot; te verwerpen: <br/>**0** - keur verzoeken aan / <br/>**1** goed - Dispatcher behandelt geen verzoeken aan /. Gebruik in plaats daarvan mod_alias voor de juiste toewijzing. |
| DispatcherUseProcessURL | Bepaalt of om voorverwerkte URLs voor al verdere verwerking door Dispatcher te gebruiken: <br/>**0** - gebruik originele URL die tot de Webserver wordt overgegaan. <br/>**1** - Dispatcher gebruikt reeds URL die door de managers wordt verwerkt die Dispatcher (namelijk `mod_rewrite`) voorafgaan in plaats van originele URL die tot de Webserver wordt overgegaan. Het origineel of de verwerkte URL komt bijvoorbeeld overeen met Dispatcher-filters. De URL wordt ook gebruikt als basis voor de structuur van het cachebestand. Raadpleeg de documentatie bij de Apache-website voor informatie over mod_rewrite, bijvoorbeeld Apache 2.4. Wanneer het gebruiken van mod_rewrite, gebruik de vlag &quot;passthrough&quot;(ga door tot de volgende manager) om de herschrijfmotor te dwingen om het gebied van URI van de interne request_rec structuur aan de waarde van het filename gebied te plaatsen. |
| DispatcherPassError | Bepaalt hoe te om foutencodes voor ErrorDocument behandeling te steunen: <br/>**0** - Dispatcher spoelt alle foutenreacties aan de cliënt. <br/>**1** - Dispatcher spool geen foutenreactie aan de cliënt (waar de statuscode groter of gelijk is dan 400). In plaats daarvan wordt de statuscode doorgegeven aan Apache, waardoor een ErrorDocument-instructie een dergelijke statuscode kan verwerken. <br/>**Waaier van de Code** - specificeer een waaier van foutencodes waarvoor de reactie aan Apache wordt overgegaan. Andere foutcodes worden doorgegeven aan de client. De volgende configuratie geeft bijvoorbeeld reacties voor fout 412 door aan de client en alle andere fouten worden doorgegeven aan Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Geeft de time-out bij &#39;houden in leven&#39; in seconden aan. Vanaf Dispatcher versie 4.2.0 is de standaardwaarde voor het in leven houden 60. Met de waarde 0 wordt het in leven houden uitgeschakeld. |
| DispatcherNoCanonURL | Als u deze parameter instelt op Aan, wordt de onbewerkte URL doorgegeven aan de achterkant in plaats van de gecanonicaliseerde URL en worden de instellingen van DispatcherUseProcessesURL genegeerd. De standaardwaarde is Uit. <br/>**Nota**: De filterregels in de configuratie van Dispatcher worden altijd geëvalueerd tegen ontsmette URL niet ruwe URL. |

>[!NOTE]
>
>Paditems zijn relatief ten opzichte van de hoofdmap van de Apache Web Server.

>[!NOTE]
>
>De standaardinstellingen voor de serverkoptekst zijn:
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>Dit toont de AEM versie voor statistische doeleinden. Als u het beschikbaar zijn van dergelijke informatie in de kopbal wilt onbruikbaar maken, kunt u het volgende plaatsen:
>
>`ServerTokens Prod`
>
>Zie de [ Documentatie Apache over de richtlijn ServerTokens (bijvoorbeeld, voor Apache 2.4) ](https://httpd.apache.org/docs/2.4/mod/core.html) voor meer informatie.

**SetHandler**

Na deze ingangen moet u a **SetHandler** verklaring aan de context van uw configuratie ( `<Directory>`, `<Location>`) voor Dispatcher toevoegen om de inkomende verzoeken te behandelen. In het volgende voorbeeld wordt de Dispatcher geconfigureerd voor het afhandelen van aanvragen voor de volledige website:

**Vensters en UNIX®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

In het volgende voorbeeld wordt de Dispatcher geconfigureerd voor het afhandelen van aanvragen voor een virtueel domein:

**Vensters**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**UNIX®**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
>
>De parameter van de **SetHandler** verklaring moet *precies het zelfde als de bovengenoemde voorbeelden* worden geschreven omdat het de naam van de manager is die in de module wordt bepaald.
>
>Zie de dossiers van de voorbeeldconfiguratie en de documentatie van de Server van het Web Apache voor volledige details over dit bevel worden verstrekt die.

**ModMimeUsePathInfo**

Na **SetHandler** verklaring, zou u ook de **ModMimeUsePathInfo** definitie moeten toevoegen.

>[!NOTE]
>
>Gebruik en configureer de parameter `ModMimeUsePathInfo` alleen als u Dispatcher versie 4.0.9 of hoger gebruikt.
>
>Dispatcher versie 4.0.9 is in 2011 uitgebracht. Als u een oudere versie gebruikt, kunt u een upgrade naar een recente Dispatcher-versie uitvoeren.

De **parameter 0} ModMimeUsePathInfo `On` zou voor alle configuraties Apache moeten worden geplaatst:**

`ModMimeUsePathInfo On`

De module mod_mime (bijvoorbeeld, [ Apache Module mod_mime ](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) wordt gebruikt om inhoudsmeta-gegevens aan de inhoud toe te wijzen die voor een reactie van HTTP wordt geselecteerd. De standaardinstelling houdt in dat `mod_mime` het inhoudstype bepaalt. Als zodanig wordt alleen het deel van de URL in aanmerking genomen dat aan een bestand of map is toegewezen.

Wanneer `On`, specificeert de `ModMimeUsePathInfo` parameter dat `mod_mime` het inhoudstype moet bepalen dat op *wordt gebaseerd volledige* URL; dit betekent dat de virtuele middelen meta-informatie hebben die op hun uitbreiding wordt toegepast.

Het volgende voorbeeld activeert **ModMimeUsePathInfo**:

**Vensters en UNIX®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### Ondersteuning inschakelen voor HTTPS (UNIX® en Linux®) {#enable-support-for-https-unix-and-linux}

Dispatcher gebruikt OpenSSL om veilige communicatie via HTTP te implementeren. Beginnend van versie van Dispatcher **4.2.0**, worden OpenSSL 1.0.0 en OpenSSL 1.0.1 gesteund. Dispatcher gebruikt standaard OpenSSL 1.0.0. Als u OpenSSL 1.0.1 wilt gebruiken, gebruikt u de volgende procedure om symbolische koppelingen te maken, zodat de Dispatcher de geïnstalleerde OpenSSL-bibliotheken gebruikt.

1. Open een terminal en wijzig de huidige map in de map waarin de OpenSSL-bibliotheken zijn geïnstalleerd, bijvoorbeeld:

   ```shell
   cd /usr/lib64
   ```

1. Voer de volgende opdrachten in om de symbolische koppelingen te maken:

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>Als u een aangepaste versie van Apache gebruikt, zorg ervoor Apache en Dispatcher gebruikend de zelfde versie van [ OpenSSL ](https://www.openssl.org/source/) worden gecompileerd.

### Volgende stappen {#next-steps-1}

Voordat u de Dispatcher kunt gaan gebruiken, moet u nu het volgende doen:

* [ vorm ](dispatcher-configuration.md) Dispatcher
* [ vorm AEM ](page-invalidate.md) om met Dispatcher te werken.

## Sun Java™ System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>Hier worden instructies voor zowel Windows- als UNIX®-omgevingen besproken.
>
>Wees voorzichtig bij het selecteren welke moet worden uitgevoerd.

### Sun Java™ System Web Server / iPlanet - Uw webserver installeren {#sun-java-system-web-server-iplanet-installing-your-web-server}

Raadpleeg de documentatie bij de desbetreffende webservers voor volledige informatie over het installeren van deze webservers:

* Sun Java™ System Web Server
* iPlanet-webserver

### Sun Java™ System Web Server / iPlanet - Voeg de Dispatcher-module toe {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

De Dispatcher is als volgt:

* **Vensters**: een Dynamische Bibliotheek van de Verbinding (DLL)
* **UNIX®**: Een Dynamisch Gedeeld Voorwerp (DSO)

De archiefbestanden van de installatie bevatten de volgende bestanden, afhankelijk van of u Windows of UNIX® hebt geselecteerd:

| Bestand | Beschrijving |
|---|---|
| `disp_ns.dll` | Windows: Het Dispatcher-bibliotheekbestand voor dynamische koppelingen. |
| `dispatcher.so` | UNIX®: Het bibliotheekbestand voor gezamenlijke Dispatcher-objecten. |
| `dispatcher.so` | UNIX®: Een voorbeeldkoppeling. |
| `obj.conf.disp` | Een voorbeeld van een configuratiebestand voor de iPlanet/Sun Java™-webserver. |
| `dispatcher.any` | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Nota:** controleer dit dossier alvorens de installatie te beginnen. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en vorige versie. |

Ga als volgt te werk om de Dispatcher aan uw webserver toe te voegen:

1. Plaats het Dispatcher-bestand in de map `plugin` van de webserver:

### Sun Java™ System Web Server / iPlanet - Configureren voor de Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

De webserver moet worden geconfigureerd met `obj.conf` . Zoek in de Dispatcher-installatiekit naar een voorbeeld-configuratiebestand met de naam `obj.conf.disp` .

1. Navigeer naar `<WEBSERVER_ROOT>/config` .
1. Open `obj.conf` voor het uitgeven.
1. Kopieer de regel die begint:\
   `Service fn="dispService"`\
   van `obj.conf.disp` naar de initialisatiesectie van `obj.conf` .

1. Sla de wijzigingen op.
1. Open `magnus.conf` voor bewerking.
1. Kopieer de twee regels die beginnen:\
   `Init funcs="dispService, dispInit"`\
   en\
   `Init fn="dispInit"`\
   van `obj.conf.disp` naar de initialisatiesectie van `magnus.conf` .

1. Sla de wijzigingen op.

>[!NOTE]
>
>De volgende configuraties moeten allemaal op één regel staan. De `$(SERVER_ROOT)` en `$(PRODUCT_SUBDIR)` moeten ook worden vervangen door hun respectievelijke waarden.

**Init**

De volgende tabel bevat voorbeelden die kunnen worden gebruikt. De exacte vermeldingen zijn gebaseerd op uw specifieke webserver:

**Vensters en UNIX®**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

Waarbij:

| Parameter | Beschrijving |
|--- |--- |
| `config` | Locatie en naam van het configuratiebestand `dispatcher.any.` |
| `logfile` | Locatie en naam van het logbestand. |
| `loglevel` | Het niveau van het logboek voor wanneer het schrijven van berichten aan het logboekdossier: <br/>**0** Fouten <br/>**1** Waarschuwing <br/>**2** Info <br/>**** zuivert <br/>**Nota:** plaats het logboekniveau aan 3 tijdens installatie en het testen en aan 0 wanneer het lopen in een productiemilieu. |
| `keepalivetimeout` | Geeft de time-out bij &#39;houden in leven&#39; in seconden aan. Vanaf Dispatcher versie 4.2.0 is de standaardwaarde voor het in leven houden 60. Met de waarde 0 wordt het in leven houden uitgeschakeld. |

Afhankelijk van uw vereisten kunt u de Dispatcher definiëren als service voor uw objecten. Als u de Dispatcher voor uw gehele website wilt configureren, bewerkt u het standaardobject:

**Vensters**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**UNIX®**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### Volgende stappen {#next-steps-2}

Voordat u de Dispatcher kunt gaan gebruiken, moet u nu:

* [ vorm ](dispatcher-configuration.md) Dispatcher
* [ vorm AEM ](page-invalidate.md) om met Dispatcher te werken.
