---
title: Opmerkingen bij de release AEM Dispatcher
description: Opmerkingen bij de release specifiek voor Adobe Experience Manager Dispatcher
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '1062'
ht-degree: 1%

---

# Opmerkingen bij de release van AEM Dispatcher{#aem-dispatcher-release-notes}

## Gegevens vrijgeven {#release-information}

|  |  |
|--- |--- |
| Producten | Adobe Experience Manager (AEM) Dispatcher |
| Versie | 4.3.7. |
| Type | Kleine release |
| Datum | 27 maart 2024 |
| URL downloaden | <ul><li>[ Apache 2.4 ](#apache)</li><li>[ Microsoft® Internet Information Services (IIS) ](#iis)</li></ul> |
| Compatibiliteit | AEM 6.1 of hoger |

## Systeemvereisten en -vereisten {#system-requirements-and-prerequisites}

Zie [ Gesteunde Platforms ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#dispatcher-platforms-web-servers) voor meer informatie over vereisten en eerste vereisten.

Adobe raadt aan de nieuwste versie van AEM Dispatcher te gebruiken om te profiteren van de nieuwste functionaliteit, de meest recente opgeloste problemen en de best mogelijke prestaties.

## Installatie-instructies {#installation-instructions}

Voor gedetailleerde instructies, zie [ Installerend Dispatcher ](dispatcher-install.md).

## Releasegeschiedenis {#release-history}

### Release 4.3.7 (2024-maart-27) {#march}

**Verbeteringen**:

* DISP-1009 - het plaatsen van kopballengte opnieuw
* DISP-1013 - Openssl 3.0-ondersteuning toevoegen voor Linux®
* DISP-1014 - response.location processing leading to invalid redirect
* DISP-1017 - veranderende definitie DTD

### Release 4.3.6 (2023-juli-25) {#jyly}

**Verbeteringen**:

* DISP-911 AEM-05 - X-Edge-Key kan worden gelekt in disp_apache2.c.
* DISP-937 het registreren van alle selecteurs.
* DISP-998 makend lading van ijdelheid urls bij opstarten configureerbaar.

### Release 4.3.5 (2022-apr-04) {#apr}

**Verbeteringen**:

* DISP-954 - De ongeldigverklaring van de steun zelfs als het verlopen niet is overgegaan
* DISP-949 - Dispatcher keert 200 in plaats van 404 terug zelfs als de filter POST verzoek blokkeert

### Release 4.3.4 (2021-nov-29) {#nov}

**Bugfixes**:

* DISP-833 - x-Door:sturen-Gastheerkopballen kunnen een lijst van komma-gescheiden hostnames bevatten.
* DISP-835 - DispatcherUseForwardedHost absorbeert de kopbal van de Gastheer als het laatste komt.

**Verbeteringen**:

* DISP-874 - Creeert een configuratie van Dispatcher om implementatie van DISP-818 of aan of uit door een vlag te draaien `DispatcherRestrictUncacheableContent`. De standaardwaarde is Uit. Wanneer Aan, verwijdert het om het even welke caching kopballen die door mod worden geplaatst verlopen voor uncacheable inhoud. Deze instelling is anders dan het gedrag in versie 4.3.3 (waar de standaardinstelling Aan was), maar hetzelfde als versies ouder dan 4.3.3 (waar de standaardinstelling Uit was). De standaardinstelling van `DispatcherRestrictUncacheableContent` uit houden is de aanbevolen aanpak, zodat de browsercache meer flexibiliteit heeft. Wanneer u een upgrade uitvoert van versie 4.3.3 naar 4.3.4, moet u `DispatcherRestrictUncacheableContent` expliciet instellen op Aan als u hetzelfde gedrag wilt behouden als in versie 4.3.3.
* DISP-841 - Dispatcher respecteert geen /serverStaleOnError voor 504-antwoordcode
* DISP-874 - creeer een configuratie van Dispatcher om implementatie van DISP-818 of weg te draaien
* DISP-883 - Spoor tonend URL verzoek Decomposition in Dispatcher
* DISP-944 - voorladings vanity urls

### Release 4.3.3 (2019-okt-18) {#october}

**Bugfixes**:

* DISP-739 - LogLevel Dispatcher: **niveau** werkt niet
* DISP-749 - Alpine Linux® Dispatcher crasht met tracklogniveau

**Verbeteringen**:

* DISP-813 - Steun in Dispatcher voor openssl 1.1.x
* DISP-814 - Apache 40x-fouten tijdens cachefoppen
* DISP-818 - mod_validate voegt cache-Control kopballen voor uncacheable inhoud toe
* DISP-821 - Sla logcontext niet op in de socket
* DISP-822 - Dispatcher moet `ppoll` gebruiken in plaats van `pselect`
* DISP-824 - Secure DispatcherUseForwardedHost
* DISP-825 - Logboek een speciaal bericht wanneer er geen ruimte meer op de schijf is
* DISP-826 - Steunt refetch URIs met een vraagkoord

**Nieuwe Eigenschappen**:

* DISP-703 - Farm Specific Cache Hit Ratio
* DISP-827 - Lokale server voor het testen
* DISP-828 - Een testdockerafbeelding voor Dispatcher maken

### Release 4.3.2 (2019-jan-31) {#jan}

**Bugfixes**:

* DISP-734 - Dispatcher veroorzaakt neerstorting in insert_output_filter als niet geplaatst als manager
* DISP-735 - REs werkt niet op Alpine Linux®
* DISP-740 - Dispatcher laden in macOS Mojave is standaard uitgeschakeld
* DISP-742 - Geblokkeerde verzoeken kunnen informatie aan auth checker beschermde middelen lekken

**Verbeteringen**:

* DISP-746 - Tekenreeksen zonder label in dispatcher.any moet een waarschuwing genereren

**Nieuwe Eigenschap**:

* DISP-747 - Geef aanvraaginformatie op in de Apache-omgeving

### Release 4.3.1 (2018-okt-16) {#oct}

**Bugfixes**:

* DISP-656 - Dispatcher heeft onjuiste ETag Header
* DISP-694 - onderdruk waarschuwingen wanneer het houden van levende verbindingen stale gaat
* DISP-714 - het gebaseerde zittingsbeheer van Cookie werkt niet in IIS
* DISP-715 - Veilige vlag voor teruggegeven koekje
* DISP-720 - Tijdelijke bestanden niet gesloten, kan tot uitputting (teveel geopende bestanden) leiden
* DISP-721 - Dispatcher onderbreekt opiniepeiling() wanneer Apache het onderliggende item op elegante wijze opnieuw start
* DISP-722 - De dossiers van het geheime voorgeheugen worden gecreeerd met octale wijze 0600
* DISP-723 - impliciete timeout van 10 minuten (en probeer opnieuw) wanneer Render timeouts aan 0 worden geplaatst
* DISP-725 - Navolgende karakters na koorden worden stil omgezet in een naamloze waarde
* DISP-726 - Logboek een waarschuwing wanneer geen landbouwbedrijf eigenlijk de inkomende gastheer aanpast
* DISP-727 - Dispatcher controleert de lengte van de verzoekinhoud voor lege geheim voorgeheugendossiers
* DISP-730 - 404 wanneer het proberen om tot een kopbaldossier over Dispatcher toegang te hebben
* DISP-731 - Dispatcher is kwetsbaar voor Log Injection
* DISP-732 - Dispatcher moet opeenvolgende &#39;/&#39; in de URL verwijderen
* DISP-733 - Dispatcher moet een paginakoptekst instellen (berekenen)

**Verbeteringen**:

* DISP-656 - Dispatcher heeft onjuiste ETag Header
* DISP-694 - onderdruk waarschuwingen wanneer het houden van levende verbindingen stale gaat
* DISP-715 - Veilige vlag voor teruggegeven koekje
* DISP-722 - De dossiers van het geheime voorgeheugen worden gecreeerd met octale wijze 0600
* DISP-726 - Logboek een waarschuwing wanneer geen landbouwbedrijf eigenlijk de inkomende gastheer aanpast

### Release 4.3.0 (2018-jun-13) {#jun}

**Bugfixes**:

* DISP-682 - Numeriek logniveau onjuist toegepast
* DISP-685 - 32 - beetje de binaire getallen van Solaris™ SPARC® hebben een ongedefinieerde verwijzing naar __divdi3
* DISP-688 - Dispatcher retourneert geen &quot;X-Cache-Info&quot; header bij 404-reactie
* DISP-690 - Last-Modified header is not cacheable
* DISP-691 - de Overtredingen van de toegang in w3wp.exe
* DISP-693 - Moet de architecturale details voor servers Solaris™ op de downloadpagina van Dispatcher bijwerken
* DISP-695 - Probleem met DispatcherLog-niveau in Dispatcher module 4.2.3
* DISP-698 - Dispatcher TTL moet s-maxage en privé richtlijnen steunen
* DISP-700 - Module werkt niet correct op Alpine Linux®
* DISP-704 - Browser verzoeken die %2b bevatten worden verzonden naar de uitgever niet-gecodeerd
* DISP-705 - Dispatcher loopt vast als gevolg van dubbele vrijheid of corruptie (fasttop)
* DISP-706 - tijdens ongeldigverklaring, volgt Dispatcher achterverwijzingssymlinks die een oneindige lijn kunnen veroorzaken
* DISP-709 - blokkeer sommige vanity URL uitbreidingen
* DISP-710 - Builds for Linux® not utable on Cent OS 6

**Verbeteringen**:

* DISP-652 - Dispatcher heeft onjuiste Date-header

## Nuttige bronnen {#helpful-resources}

* [AEM Dispatcher - Overzicht](dispatcher.md)

## Downloads {#downloads}

### Apache 2.4 {#apache}

| Platform | Architectuur | OpenSSL-ondersteuning | Klik om te downloaden |
|---|---|---|---|
| Linux® | i686 (32 bits) | Geen | [`dispatcher-apache2.4-linux-i686-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.7.tar.gz) |
| Linux® | i686 (32 bits) | 1,0 | [`dispatcher-apache2.4-linux-i686-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.7.tar.gz) |
| Linux® | i686 (32 bits) | 1,1 | [`dispatcher-apache2.4-linux-i686-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.7.tar.gz) |
| Linux® | i686 (32 bits) | 3,0 | [`dispatcher-apache2.4-linux-i686-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl3.0-4.3.7.tar.gz) |
| Linux® | x86_64 (64-bits) | Geen | [`dispatcher-apache2.4-linux-x86_64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.7.tar.gz) |
| Linux® | x86_64 (64-bits) | 1,0 | [`dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.7.tar.gz) |
| Linux® | x86_64 (64-bits) | 1,1 | [`dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.7.tar.gz) |
| Linux® | x86_64 (64-bits) | 3,0 | [`dispatcher-apache2.4-linux-x86_64-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl3.0-4.3.7.tar.gz) |
| Linux® | aarch64 (64-bits) | Geen | [`dispatcher-apache2.4-linux-aarch64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-4.3.7.tar.gz) |
| Linux® | aarch64 (64-bits) | 1,0 | [`dispatcher-apache2.4-linux-aarch64-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl1.0-4.3.7.tar.gz) |
| Linux® | aarch64 (64-bits) | 1,1 | [`dispatcher-apache2.4-linux-aarch64-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl1.1-4.3.7.tar.gz) |
| Linux® | aarch64 (64-bits) | 3,0 | [`dispatcher-apache2.4-linux-aarch64-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl3.0-4.3.7.tar.gz) |
| macOS | arm64 (64-bits) | Geen | [`dispatcher-apache2.4-darwin-arm64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-arm64-4.3.7.tar.gz) |
| macOS | x86_64 (64-bits) | Geen | [`dispatcher-apache2.4-darwin-x86_64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.7.tar.gz) |

### IIS {#iis}

| Platform | Architectuur | OpenSSL-ondersteuning | Klik om te downloaden |
|---|---|---|---|
| Windows | x86 (32-bits) | Geen | [`dispatcher-iis-windows-x86-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.7.zip) |
| Windows | x86 (32-bits) | 1,0 | [`dispatcher-iis-windows-x86-ssl1.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.7.zip) |
| Windows | x86 (32-bits) | 1,1 | [`dispatcher-iis-windows-x86-ssl1.1-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.7.zip) |
| Windows | x64 (64-bits) | Geen | [`dispatcher-iis-windows-x64-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.7.zip) |
| Windows | x64 (64-bits) | 1,0 | [`dispatcher-iis-windows-x64-ssl1.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.7.zip) |
| Windows | x64 (64-bits) | 1,1 | [`dispatcher-iis-windows-x64-ssl1.1-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.7.zip) |
| Windows | x64 (64-bits) | 3,0 | [`dispatcher-iis-windows-x64-ssl3.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl3.0-4.3.7.zip) |