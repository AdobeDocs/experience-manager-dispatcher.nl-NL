---
title: Dispatcher installeren
description: Leer hoe u de Dispatcher-module installeert op Microsoft&reg; Internet Information Server, Apache Web Server en Sun Java &Trade; Web Server-iPlanet.
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

Gebruik de [Opmerkingen bij de release Dispatcher](release-notes.md) om het nieuwste Dispatcher-installatiebestand voor uw besturingssysteem en webserver op te halen. De versienummers van de Dispatcher zijn onafhankelijk van de Adobe Experience Manager-releasenummers en zijn compatibel met de Adobe Experience Manager 6.x-, 5.x- en Adobe CQ 5.x-releases.

>[!NOTE]
>
>Voor Adobe Experience Manager 6.5 is Dispatcher versie 4.3.2 of hoger vereist. Dat gezegd hebbende, zijn de versies van de Verzender onafhankelijk van AEM, bijvoorbeeld, is Dispatcher versie 4.3.2 ook compatibel met Adobe Experience Manager 6.4.

De volgende naamgevingsconventie voor bestanden wordt gebruikt:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Bijvoorbeeld de `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` Het bestand bevat Dispatcher versie 4.3.1 voor een Apache 2.4-webserver die wordt uitgevoerd op Linux® i686 en die is verpakt met de **teer** gebruiken.

De volgende tabel bevat de id van de webserver die wordt gebruikt in bestandsnamen voor elke webserver:

| Webserver | Installatiekit |
|--- |--- |
| Apache 2.4 | verzender-apache **2,4**-&lt;other parameters=&quot;&quot;> |
| Microsoft® Internet Information Server 7.5, 8, 8.5, 10 | verzender-**iis**-&lt;other parameters=&quot;&quot;> |
| Sun Java™ Web Server iPlanet | verzender-**ns**-&lt;other parameters=&quot;&quot;> |

>[!CAUTION]
>
>Installeer de nieuwste versie van Dispatcher die beschikbaar is voor uw platform. Voer jaarlijks een upgrade uit op de Dispatcher-instantie om de nieuwste versie te gebruiken om te profiteren van productverbeteringen.

>[!NOTE]
>
>Klanten die specifiek een upgrade uitvoeren van versie 4.3.3 naar versie 4.3.4, moeten een ander gedrag opmerken voor de manier waarop in cache geplaatste koppen worden ingesteld voor niet-cachebare inhoud. Voor meer informatie over deze wijziging raadpleegt u de [Opmerkingen bij de release](/help/using/release-notes.md#nov) pagina.

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
* [&quot;De officiële Microsoft® IIS-site&quot;](https://www.iis.net/)

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
| `disp_iis.dll` | Het bibliotheekbestand voor dynamische koppelingen voor Dispatcher. |
| `disp_iis.ini` | Het dossier van de configuratie voor IIS. Dit voorbeeld kan met uw vereisten worden bijgewerkt. **Opmerking**: Het ini-bestand moet dezelfde naam-hoofdmap hebben als de dll. |
| `dispatcher.any` | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| `author_dispatcher.any` | Een voorbeeldconfiguratiebestand voor Dispatcher dat werkt met de auteurinstantie. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Opmerking**: Controleer dit bestand voordat u de installatie start. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en eerdere versies. |

Gebruik de volgende procedure om de Dispatcher-bestanden naar de juiste locatie te kopiëren.

1. Windows Verkenner gebruiken om de `<IIS_INSTALLDIR>/Scripts` map, bijvoorbeeld `C:\inetpub\Scripts`.

1. Pak de volgende bestanden uit het pakket Dispatcher uit in deze map Scripts:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * Een van de volgende bestanden is afhankelijk van het feit of de Dispatcher werkt met een AEM auteur-instantie of -publicatie-instantie:
      * Instantie van auteur: `author_dispatcher.any`
      * Instantie publiceren: `dispatcher.any`

## Microsoft® IIS - Het INI-bestand Dispatcher configureren {#microsoft-iis-configure-the-dispatcher-ini-file}

Als u de installatie van Dispatcher wilt configureren, bewerkt u de `disp_iis.ini` bestand. Het basisformaat van de `.ini` bestand is als volgt:

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
| `logfile` | De locatie van de `dispatcher.log` bestand. Als deze plaats niet wordt geplaatst, dan gaan de logboekberichten naar het de gebeurtenislogboek van Vensters. |
| `loglevel` | Definieert het logniveau dat wordt gebruikt voor het uitvoeren van berichten naar het gebeurtenislogboek. De volgende waarden kunnen op het logniveau voor het logbestand worden opgegeven: <br/>0 - alleen foutberichten. <br/>1 - fouten en waarschuwingen. <br/>2 - fouten, waarschuwingen en informatieve berichten <br/>3 - fouten, waarschuwingen, informatieberichten, en zuivert berichten. <br/>**Opmerking**: Stel het logniveau in op 3 tijdens de installatie en het testen en op 0 tijdens het uitvoeren in een productieomgeving. |
| `replaceauthorization` | Geeft aan hoe machtigingsheaders in de HTTP-aanvraag worden verwerkt. De volgende waarden zijn geldig:<br/>0 - De machtigingsheaders worden niet gewijzigd. <br/>1 - Vervangt elke koptekst met de naam &quot;Autorisatie&quot;, behalve &quot;Standaard&quot;, door de bijbehorende `Basic <IIS:LOGON\_USER>` equivalent.<br/> |
| `servervariables` | Definieert hoe servervariabelen worden verwerkt.<br/>0 - IIS-servervariabelen worden niet verzonden naar de Dispatcher of de AEM. <br/>1 - alle IIS-servervariabelen (zoals `LOGON\_USER, QUERY\_STRING, ...`) worden samen met de aanvraagheaders naar de Dispatcher verzonden (en ook naar de AEM instantie als deze niet in cache is geplaatst).  <br/>Servervariabelen omvatten `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` en vele anderen. Zie de documentatie IIS voor de volledige lijst van variabelen, met details. |
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

Vorm IIS om de Dispatcher ISAPI module te integreren. In IIS gebruikt u toewijzing van jokertekens voor toepassingen.

### Het vormen Anonieme Toegang - IIS 8.5 en 10 {#configuring-anonymous-access-iis-and}

De standaard Flush replicatieagent op de instantie van de Auteur wordt gevormd zodat het geen veiligheidsgeloofsbrieven met flush verzoeken verzendt. Daarom moet de website waarop u de Dispatcher-cache gebruikt anonieme toegang toestaan.

Als uw website een authentificatiemethode gebruikt, moet de Flush replicatieagent dienovereenkomstig worden gevormd.

1. Open IIS Manager en selecteer de website die u als Dispatcher geheim voorgeheugen gebruikt.
1. Gebruikend de wijze van de Mening van Eigenschappen, in de sectie IIS tweemaal klikken Authentificatie.
1. Als Anonieme verificatie niet is ingeschakeld, selecteert u Anonieme verificatie en klikt u in het gedeelte Handelingen op Inschakelen.

### Integratie van de Dispatcher ISAPI Module - IIS 8.5 en 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Gebruik de volgende procedure om de Dispatcher ISAPI module aan IIS toe te voegen.

1. Open IIS Manager.
1. Selecteer de website die u als Dispatcher Cache gebruikt.
1. Gebruikend de wijze van de Mening van Eigenschappen, in de sectie IIS tweemaal klikken Handler Mappings.
1. Klik in het deelvenster Handelingen van de pagina Handler Mappings op Toevoegen van jokerscript, voeg de volgende eigenschapswaarden toe en klik op OK:

   * Pad aanvragen: &#42;
   * Uitvoerbaar: het absolute pad van het bestand disp_is.dll, bijvoorbeeld `C:\inetpub\Scripts\disp_iis.dll`.
   * Naam: een beschrijvende naam voor de handlertoewijzing, bijvoorbeeld `Dispatcher`.

1. In het dialoogvenster dat wordt weergegeven, klikt u op **Ja**.

   Voor IIS 7.0 en 7.5, is de configuratie volledig. Ga met de resterende stappen verder als u IIS 8.0 vormt.

1. (IIS 8.0) Selecteer in de lijst Handler Mappings de afbeelding die u hebt gemaakt en klik in het gedeelte Handelingen op Bewerken.
1. (IIS 8.0) Klik in het dialoogvenster Scriptkaart bewerken op de knop Beperkingen aanvragen.
1. (IIS 8.0) Om ervoor te zorgen dat de manager voor dossiers en omslagen wordt gebruikt die nog niet caching zijn, schrap **Alleen handler aanroepen als de aanvraag is toegewezen aan**. Klikken **OK**.
1. (IIS 8.0) Klik in het dialoogvenster Scripttoewijzing bewerken op OK.

### Het vormen Toegang tot het geheime voorgeheugen - IIS 8.5 en 10 {#configuring-access-to-the-cache-iis-and}

Geef de standaardgebruiker van de App Pool schrijftoegang tot de map die wordt gebruikt als Dispatcher-cache.

1. Klik met de rechtermuisknop op de hoofdmap van de website die u gebruikt als de Dispatcher-cache en klik op Eigenschappen, zoals `C:\inetpub\wwwroot`.
1. Klik op het tabblad Beveiliging op Bewerken en klik vervolgens in het dialoogvenster Machtigingen op Toevoegen. Er wordt een dialoogvenster geopend waarin u gebruikersaccounts kunt selecteren. Klik op de knop Locaties, selecteer de naam van de computer en klik op OK.

   Zorg dat dit dialoogvenster geopend blijft terwijl u de volgende stap uitvoert.

1. in Manager IIS, selecteer de plaats IIS die u voor het geheime voorgeheugen van de Verzender gebruikt, en op de rechterkant van het venster, klik Geavanceerde Montages.
1. Selecteer de waarde van het bezit van de Pool van de Toepassing en kopieer het aan het klembord.
1. Ga terug naar het geopende dialoogvenster. Typ in het vak Geef de objectnamen op die u wilt selecteren `IIS AppPool\` en plakt u vervolgens de inhoud van het klembord. De waarde moet er als volgt uitzien:

   `IIS AppPool\DefaultAppPool`

1. Klik op de knop Namen controleren. Klik op OK als Windows het gebruikersaccount oplost.
1. Selecteer in het dialoogvenster Machtigingen voor de map Dispatcher de account die u zojuist hebt toegevoegd, alle machtigingen voor de account inschakelen **behalve Volledige controle** en klik op OK. Klik op OK om het dialoogvenster Eigenschappen van map te sluiten.

### Registreren van het JSON Mime-type - IIS 8.5 en 10 {#registering-the-json-mime-type-iis-and}

Gebruik de volgende procedure om het JSON MIME-type te registreren wanneer u wilt dat de Dispatcher JSON-aanroepen toestaat.

1. Selecteer uw website in IIS Manager en gebruik de weergave Eigenschappen en dubbelklik op MIME-typen.
1. Als de JSON-extensie niet in de lijst voorkomt, klikt u in het deelvenster Handelingen op Toevoegen, voert u de volgende eigenschapswaarden in en klikt u op OK:

   * Bestandsnaamextensie: `.json`
   * MIME-type: `application/json`

### Het verborgen segment van de bin verwijderen - IIS 8.5 en 10 {#removing-the-bin-hidden-segment-iis-and}

Gebruik de volgende procedure om de `bin` verborgen segment. Websites die niet nieuw zijn, kunnen dit verborgen segment bevatten.

1. Selecteer in IIS Manager uw website en gebruik de weergave Functies en dubbelklik op Verzoek filteren.
1. Selecteer de `bin` klikt u op Verwijderen en klikt u in het bevestigingsdialoogvenster op Ja.

### Het registreren IIS Berichten aan een Dossier - IIS 8.5 en 10 {#logging-iis-messages-to-a-file-iis-and}

Gebruik de volgende procedure om het logboekberichten van de Ontvanger aan een logboekdossier in plaats van aan het logboek van de Gebeurtenis van Vensters te schrijven. Vorm de Verzender om het logboekdossier te gebruiken, en IIS van schrijven-toegang tot het dossier te voorzien.

1. Windows Verkenner gebruiken om een map te maken met de naam `dispatcher` onder de logbestanden van de IIS-installatie. Het pad van deze map voor een standaardinstallatie is `C:\inetpub\logs\dispatcher`.

1. Klik met de rechtermuisknop op de map Dispatcher en klik op **Eigenschappen**.
1. Klik op het tabblad Beveiliging op **Bewerken**.
1. Klik in het dialoogvenster Machtigingen op **Toevoegen**. Er wordt een dialoogvenster geopend waarin u gebruikersaccounts kunt selecteren. Klik op de knop Locaties, selecteer de naam van de computer en klik op OK.

   Zorg dat dit dialoogvenster geopend blijft terwijl u de volgende stap uitvoert.

1. in Manager IIS, selecteer de plaats IIS die u voor het geheime voorgeheugen van de Verzender gebruikt, en op de rechterkant van het venster, klik Geavanceerde Montages.
1. Selecteer de waarde van het bezit van de Pool van de Toepassing en kopieer het aan het klembord.
1. Ga terug naar het geopende dialoogvenster. Typ in het vak Geef de objectnamen op die u wilt selecteren `IIS AppPool\` en plakt u vervolgens de inhoud van het klembord. De waarde moet er als volgt uitzien:

   `IIS AppPool\DefaultAppPool`

1. Klik op de knop Namen controleren. Klik op OK als Windows het gebruikersaccount oplost.
1. Selecteer in het dialoogvenster Machtigingen voor de map Dispatcher de account die u zojuist hebt toegevoegd, alle machtigingen voor de account inschakelen **behalve voor volledige controle,** en klik op OK. Klik op OK om het dialoogvenster Eigenschappen van map te sluiten.
1. Een teksteditor gebruiken om het dialoogvenster `disp_iis.ini` bestand.
1. Als u de locatie van het logbestand wilt configureren, voegt u een tekstregel toe die lijkt op het volgende voorbeeld en slaat u het bestand op:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Volgende stappen {#next-steps}

Voordat u de Dispatcher kunt gaan gebruiken, moet u het volgende weten:

* [Configureren](dispatcher-configuration.md) Dispatcher
* [AEM configureren](page-invalidate.md) om met Dispatcher te werken.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Instructies voor installatie onder beide **Windows** en **UNIX®** zijn hier behandeld. Wees voorzichtig wanneer u de stappen uitvoert.

### Apache Web Server installeren {#installing-apache-web-server}

Lees de installatiehandleiding voor informatie over het installeren van een Apache Web Server - ofwel [online](https://httpd.apache.org/) of in de distributie.

>[!CAUTION]
>
>Als u een binair Apache-bestand maakt door de bronbestanden te compileren, moet u ervoor zorgen dat **`dynamic modules support`**. U kunt deze optie inschakelen met een van de **—enable-shared** opties. Neem minimaal de `mod_so` -module.
>
>Meer informatie vindt u in de installatiehandleiding van Apache Web Server.

Zie ook de Apache HTTP Server [Beveiligingstips](https://httpd.apache.org/docs/2.4/misc/security_tips.html) en [Beveiligingsrapporten](https://httpd.apache.org/security_report.html).

### Apache Web Server - Add the Dispatcher Module {#apache-web-server-add-the-dispatcher-module}

De Dispatcher wordt geleverd als:

* **Windows**: een Dynamic Link Library (DLL)
* **UNIX®**: een dynamisch gezamenlijk object (DSO)

De archiefbestanden van de installatie bevatten de volgende bestanden, afhankelijk van of u Windows of UNIX® hebt geselecteerd:

| Bestand | Beschrijving |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows: Het bibliotheekbestand voor de dynamische koppeling van Dispatcher. |
| verzender-apache&lt;x.y>-&lt;rel-nr>.so | UNIX®: Het bibliotheekbestand van het gezamenlijke object Dispatcher. |
| mod_dispatcher.so | UNIX®: Een voorbeeldkoppeling. |
| http.conf.disp&lt;x> | Een voorbeeldconfiguratiebestand voor de Apache-server. |
| dispatcher.any | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Opmerking**: Controleer dit bestand voordat u de installatie start. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en vorige versie. |

Ga als volgt te werk om de Dispatcher toe te voegen aan uw Apache Web Server:

1. Plaats het Dispatcher-bestand in de juiste Apache-modulemap:

   * **Windows**: Place `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **UNIX®**: Zoek een van de `<APACHE_ROOT>/libexec` of `<APACHE_ROOT>/modules`volgens uw installatie.\
     Kopiëren `dispatcher-apache<options>.so` in deze map.\
     Om het onderhoud op lange termijn te vereenvoudigen, kunt u ook een symbolische verbinding tot stand brengen genoemd `mod_dispatcher.so` aan de verzender:\
     `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Kopieer het bestand dispatcher.any naar de `<APACHE_ROOT>/conf` directory.

   **Opmerking:** U kunt dit dossier in een verschillende plaats plaatsen, zolang het bezit DispatcherLog van de module van de Verzender dienovereenkomstig wordt gevormd. (Zie Verzendspecifieke configuratiegegevens verderop.)

### Apache Web Server - SELinux-eigenschappen configureren {#apache-web-server-configure-selinux-properties}

Als u Dispatcher uitvoert op Red Hat® Linux® Kernel 2.6 met SELinux ingeschakeld, kunt u foutberichten zoals deze weergeven in het Dispatcher-logbestand.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Deze fout is waarschijnlijk het gevolg van een ingeschakelde SELinux-beveiliging. Zo ja, voer de volgende taken uit:

* Configureer de SELinux-context van het Dispatcher-modulebestand.
* Schakel HTTP-scripts en -modules in om netwerkverbindingen te maken.
* Vorm de context SELinux van de docroot, waar de caching dossiers worden opgeslagen.

Voer de volgende opdrachten in een terminalvenster in en vervang deze `[path to the dispatcher.so file]` met het pad naar de module Dispatcher die u op Apache Web Server hebt geïnstalleerd, en *`path to the docroot`* met het pad waar de hoofdmap zich bevindt (bijvoorbeeld `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_rw_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_rw_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Apache Web Server voor Dispatcher configureren {#apache-web-server-configure-apache-web-server-for-dispatcher}

De Apache-webserver moet zijn geconfigureerd, met `httpd.conf`. Zoek in de installatiekit Dispatcher naar een voorbeeldconfiguratiebestand met de naam `httpd.conf.disp<x>`.

Deze stappen zijn verplicht:

1. Navigeren naar `<APACHE_ROOT>/conf`.
1. Openen `httpd.conf`voor bewerken.
1. De volgende configuratieingangen moeten, in de vermelde orde worden toegevoegd:

   * **LoadModule** om de module bij het opstarten te laden.
   * Dispatcher-specifieke configuratieingangen, inclusief **DispatcherConfig, DispatcherLog**, en **DispatcherLogLevel**.
   * **SetHandler** om de Dispatcher te activeren. **LoadModule**.
   * **ModMimeUsePathInfo** om het gedrag van te vormen **mod_mime**.

1. (Optioneel) U wordt aangeraden de eigenaar van de map htdocs te wijzigen:

   * De Apache-server begint als root, maar de onderliggende processen beginnen als daemon (voor beveiligingsdoeleinden). De DocumentRoot (`<APACHE_ROOT>/htdocs`) moet bij de gebruikersdaemon horen:

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

**Specifieke configuratiegegevens van Dispatcher**

De Dispatcher-specifieke configuratieingangen worden geplaatst na de ingang LoadModule. De volgende lijst maakt een lijst van een voorbeeldconfiguratie die voor zowel UNIX® als Vensters van toepassing is:

**Windows &amp; UNIX®**

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
>Klanten die specifiek een upgrade uitvoeren van versie 4.3.3 naar versie 4.3.4, moeten een ander gedrag opmerken voor de manier waarop in cache geplaatste koppen worden ingesteld voor niet-cachebare inhoud. Voor meer informatie over deze wijziging raadpleegt u de [Opmerkingen bij de release](/help/using/release-notes.md#nov) pagina.

De individuele configuratieparameters:

| Parameter | Beschrijving |
|--- |--- |
| DispatcherConfig | Locatie en naam van het Dispatcher-configuratiebestand. <br/>Wanneer dit bezit in de belangrijkste serverconfiguratie is, erven alle virtuele gastheren de bezitswaarde. Virtuele hosts kunnen echter een eigenschap DispatcherConfig opnemen om de hoofdserverconfiguratie te overschrijven. |
| DispatcherLog | Locatie en naam van het logbestand. |
| DispatcherLogLevel | Logniveau voor het logbestand: <br/>0 - Fouten <br/>1 - Waarschuwingen <br/>2 - Infos <br/>3 - Foutopsporing <br/>**Opmerking**: Stel het logniveau in op 3 tijdens de installatie en het testen en op 0 tijdens het uitvoeren in een productieomgeving. |
| DispatcherNoServerHeader | *Deze parameter is vervangen en is ineffectief.*<br/><br/> Hiermee definieert u de serverkoptekst die moet worden gebruikt: <br/><ul><li>ongedefinieerd of 0 - De HTTP-serverheader bevat de AEM versie. </li><li>1 - De header van de Apache-server wordt gebruikt.</li></ul> |
| DispatcherDeclineRoot | Bepaalt of verzoeken aan de wortel &quot;/&quot; te weigeren: <br/>**0** - aanvragen accepteren voor / <br/>**1** - De verzender verwerkt geen aanvragen naar /. Gebruik in plaats daarvan mod_alias voor de juiste toewijzing. |
| DispatcherUseProcessURL | Hiermee wordt gedefinieerd of vooraf verwerkte URL&#39;s moeten worden gebruikt voor alle verdere verwerking door Dispatcher: <br/>**0** - gebruik de oorspronkelijke URL die aan de webserver is doorgegeven. <br/>**1** - de Dispatcher de URL gebruikt die al is verwerkt door de handlers die voorafgaan aan de Dispatcher (dat wil zeggen: `mod_rewrite`) in plaats van de oorspronkelijke URL die aan de webserver is doorgegeven. Het origineel of de verwerkte URL wordt bijvoorbeeld gekoppeld aan de Dispatcher-filters. De URL wordt ook gebruikt als basis voor de structuur van het cachebestand. Raadpleeg de documentatie bij de Apache-website voor informatie over mod_rewrite, bijvoorbeeld Apache 2.4. Wanneer het gebruiken van mod_rewrite, gebruik de vlag &quot;passthrough&quot;(ga door tot de volgende manager) om de herschrijfmotor te dwingen om het gebied van URI van de interne request_rec structuur aan de waarde van het filename gebied te plaatsen. |
| DispatcherPassError | Definieert hoe foutcodes voor ErrorDocument-afhandeling worden ondersteund: <br/>**0** - Dispatcher spoolt alle foutreacties naar de client. <br/>**1** - Dispatcher spookt geen foutreactie op de client (waarbij de statuscode groter of gelijk is aan 400). In plaats daarvan wordt de statuscode doorgegeven aan Apache, waardoor een ErrorDocument-instructie een dergelijke statuscode kan verwerken. <br/>**Codebereik** - Geef een bereik foutcodes op waarvoor het antwoord wordt doorgegeven aan Apache. Andere foutcodes worden doorgegeven aan de client. De volgende configuratie geeft bijvoorbeeld reacties voor fout 412 door aan de client en alle andere fouten worden doorgegeven aan Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Geeft de time-out bij &#39;houden in leven&#39; in seconden aan. Vanaf Dispatcher versie 4.2.0 is de standaardwaarde voor het in leven houden 60. Met de waarde 0 wordt het in leven houden uitgeschakeld. |
| DispatcherNoCanonURL | Als u deze parameter instelt op Aan, wordt de onbewerkte URL doorgegeven aan de achterkant in plaats van de gecanonicaliseerde URL en worden de instellingen van DispatcherUseProcessesURL genegeerd. De standaardwaarde is Uit. <br/>**Opmerking**: De filterregels in de configuratie Dispatcher worden altijd geëvalueerd op basis van de ontsmette URL en niet op basis van de onbewerkte URL. |

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
>Zie de [Apache-documentatie over ServerTokens-richtlijn (bijvoorbeeld voor Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) voor meer informatie .

**SetHandler**

Na deze vermeldingen moet u een **SetHandler** verklaring aan de context van uw configuratie ( `<Directory>`, `<Location>`) voor de Dispatcher om de binnenkomende aanvragen af te handelen. In het volgende voorbeeld wordt de Dispatcher geconfigureerd voor het afhandelen van aanvragen voor de volledige website:

**Windows en UNIX®**

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

In het volgende voorbeeld wordt de Dispatcher geconfigureerd om aanvragen voor een virtueel domein af te handelen:

**Windows**

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
>De parameter van **SetHandler** instructie moet worden geschreven *exact hetzelfde als de bovenstaande voorbeelden* omdat het de naam van de manager is die in de module wordt bepaald.
>
>Zie de dossiers van de voorbeeldconfiguratie en de documentatie van de Server van het Web Apache voor volledige details over dit bevel worden verstrekt die.

**ModMimeUsePathInfo**

Na de **SetHandler** moet u ook de instructie **ModMimeUsePathInfo** definitie.

>[!NOTE]
>
>Gebruik en vorm slechts `ModMimeUsePathInfo` als u Dispatcher versie 4.0.9 of hoger gebruikt.
>
>Dispatcher versie 4.0.9 is in 2011 uitgebracht. Als u een oudere versie gebruikt, kunt u een upgrade uitvoeren naar een recente versie van Dispatcher.

De **ModMimeUsePathInfo** parameter moet worden ingesteld `On` voor alle Apache-configuraties:

`ModMimeUsePathInfo On`

De module mod_mime (bijvoorbeeld [Mod_mime voor Apache-module](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) wordt gebruikt om metagegevens van de inhoud toe te wijzen aan de inhoud die is geselecteerd voor een HTTP-reactie. De standaardinstelling houdt in dat `mod_mime` bepaalt het inhoudstype. Als zodanig wordt alleen het deel van de URL in aanmerking genomen dat aan een bestand of map is toegewezen.

Wanneer `On`de `ModMimeUsePathInfo` parameter specificeert dat `mod_mime` bepaalt het inhoudstype op basis van de *complete* URL; dit betekent dat voor virtuele bronnen metagegevens zijn toegepast op basis van hun extensie.

In het volgende voorbeeld wordt geactiveerd: **ModMimeUsePathInfo**:

**Windows en UNIX®**

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

Dispatcher gebruikt OpenSSL om veilige communicatie via HTTP te implementeren. Vanaf de Dispatcher-versie **4.2.0.**, OpenSSL 1.0.0 en OpenSSL 1.0.1 worden ondersteund. Dispatcher gebruikt standaard OpenSSL 1.0.0. Als u OpenSSL 1.0.1 wilt gebruiken, gebruikt u de volgende procedure om symbolische koppelingen te maken, zodat de Dispatcher de geïnstalleerde OpenSSL-bibliotheken gebruikt.

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
>Als u een aangepaste versie van Apache gebruikt, moet u ervoor zorgen dat Apache en Dispatcher zijn gecompileerd met dezelfde versie van [OpenSSL](https://www.openssl.org/source/).

### Volgende stappen {#next-steps-1}

Voordat u de Dispatcher kunt gaan gebruiken, moet u nu het volgende doen:

* [Configureren](dispatcher-configuration.md) Dispatcher
* [AEM configureren](page-invalidate.md) om met Dispatcher te werken.

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

### Sun Java™ System Web Server / iPlanet - Voeg de Dispatcher Module toe {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

De Dispatcher wordt geleverd als:

* **Windows**: een Dynamic Link Library (DLL)
* **UNIX®**: een dynamisch gezamenlijk object (DSO)

De archiefbestanden van de installatie bevatten de volgende bestanden, afhankelijk van of u Windows of UNIX® hebt geselecteerd:

| Bestand | Beschrijving |
|---|---|
| `disp_ns.dll` | Windows: Het bibliotheekbestand voor de dynamische koppeling van Dispatcher. |
| `dispatcher.so` | UNIX®: Het bibliotheekbestand van het gezamenlijke object Dispatcher. |
| `dispatcher.so` | UNIX®: Een voorbeeldkoppeling. |
| `obj.conf.disp` | Een voorbeeld van een configuratiebestand voor de iPlanet/Sun Java™-webserver. |
| `dispatcher.any` | Een voorbeeldconfiguratiebestand voor de Dispatcher. |
| README | Leesmij-bestand met installatie-instructies en informatie van het laatste moment. **Opmerking:** Controleer dit bestand voordat u de installatie start. |
| WIJZIGINGEN | Hiermee wijzigt u een bestand waarin de problemen worden vermeld die zijn opgelost in de huidige en vorige versie. |

Ga als volgt te werk om de Dispatcher aan uw webserver toe te voegen:

1. Plaats het Dispatcher-bestand in de `plugin` map:

### Sun Java™ System Web Server / iPlanet - Configureren voor Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

De webserver moet worden geconfigureerd met `obj.conf`. Zoek in de installatiekit Dispatcher naar een voorbeeldconfiguratiebestand met de naam `obj.conf.disp`.

1. Navigeren naar `<WEBSERVER_ROOT>/config`.
1. Openen `obj.conf`voor bewerken.
1. Kopieer de regel die begint:\
   `Service fn="dispService"`\
   van `obj.conf.disp` naar de initialisatiesectie van `obj.conf`.

1. Sla de wijzigingen op.
1. Openen `magnus.conf` voor bewerken.
1. Kopieer de twee regels die beginnen:\
   `Init funcs="dispService, dispInit"`\
   en\
   `Init fn="dispInit"`\
   van `obj.conf.disp` naar de initialisatiesectie van `magnus.conf`.

1. Sla de wijzigingen op.

>[!NOTE]
>
>De volgende configuraties moeten allemaal op één regel staan. Ook de `$(SERVER_ROOT)` en `$(PRODUCT_SUBDIR)` moeten worden vervangen door hun respectieve waarden.

**Init**

De volgende tabel bevat voorbeelden die kunnen worden gebruikt. De exacte vermeldingen zijn gebaseerd op uw specifieke webserver:

**Windows en UNIX®**

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
| `loglevel` | Logniveau voor het schrijven van berichten naar het logbestand: <br/>**0** Fouten <br/>**1** Waarschuwing <br/>**2** Infos <br/>**3** Foutopsporing <br/>**Opmerking:** Stel het logniveau in op 3 tijdens installatie en testen en op 0 wanneer u in een productieomgeving werkt. |
| `keepalivetimeout` | Geeft de time-out bij &#39;houden in leven&#39; in seconden aan. Vanaf Dispatcher versie 4.2.0 is de standaardwaarde voor het in leven houden 60. Met de waarde 0 wordt het in leven houden uitgeschakeld. |

Afhankelijk van uw vereisten kunt u de Dispatcher definiëren als service voor uw objecten. Als u de Dispatcher voor uw gehele website wilt configureren, bewerkt u het standaardobject:

**Windows**

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

* [Configureren](dispatcher-configuration.md) Dispatcher
* [AEM configureren](page-invalidate.md) om met Dispatcher te werken.
