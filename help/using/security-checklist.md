---
title: Dispatcher Security Checklist
description: Klik hier als je wilt weten wat de Dispatcher Security Checklist is die moet worden voltooid voordat je gaat produceren.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '582'
ht-degree: 0%

---

# Dispatcher Security Checklist{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe raadt je aan de volgende checklist in te vullen voordat je gaat produceren.

>[!CAUTION]
>
>Voltooi de lijst Beveiligingscontrole van je versie van AEM voordat je live gaat. Zie de overeenkomstige [&#x200B; documentatie van Adobe Experience Manager &#x200B;](https://experienceleague.adobe.com/nl/docs/experience-manager-65/content/security/security-checklist).

## De nieuwste versie van Dispatcher gebruiken {#use-the-latest-version-of-dispatcher}

Installeer de nieuwste beschikbare versie die beschikbaar is voor uw platform. Voer een upgrade uit op uw Dispatcher-exemplaar om de nieuwste versie te gebruiken en zo te profiteren van product- en beveiligingsverbeteringen. Zie [&#x200B; Installerend Dispatcher &#x200B;](dispatcher-install.md).

>[!NOTE]
>
>U kunt de huidige versie van uw Dispatcher-installatie controleren door het Dispatcher-logbestand te bekijken.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Als u het logbestand wilt zoeken, controleert u de Dispatcher-configuratie in uw `httpd.conf` .

## Beperk clients die uw cache kunnen leegmaken {#restrict-clients-that-can-flush-your-cache}

Adobe adviseert dat u [&#x200B; de cliënten beperkt die uw geheime voorgeheugen kunnen leegmaken.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## HTTPS inschakelen voor beveiliging van transportlagen {#enable-https-for-transport-layer-security}

Adobe raadt aan de HTTPS-transportlaag in te schakelen voor zowel auteur- als publicatieinstanties.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## Toegang beperken {#restrict-access}

Wanneer het vormen van de Dispatcher, beperking externe toegang zoveel mogelijk. Zie [&#x200B; Sectie van het Voorbeeld /filter &#x200B;](dispatcher-configuration.md#main-pars_184_1_title) in de documentatie van Dispatcher.

## Zorg ervoor dat toegang tot administratieve URL&#39;s wordt geweigerd {#make-sure-access-to-administrative-urls-is-denied}

Zorg ervoor dat u filters gebruikt om externe toegang tot eventuele beheerURL&#39;s, zoals de webconsole, te blokkeren.

Zie [&#x200B; het Testen de Veiligheid van Dispatcher &#x200B;](dispatcher-configuration.md#testing-dispatcher-security) voor een lijst van URLs die moet worden geblokkeerd.

## Lijsten van gewenste personen gebruiken in plaats van Lijsten van gewezen personen {#use-allowlists-instead-of-blocklists}

Lijsten van gewenste personen zijn een betere manier om toegangscontrole te verlenen aangezien zij inherent, veronderstellen zij dat alle toegangsverzoeken zouden moeten worden ontkend tenzij zij uitdrukkelijk deel van de lijst van gewenste personen uitmaken. Dit model verstrekt meer restrictieve controle over nieuwe verzoeken die nog niet zouden kunnen zijn herzien of tijdens een bepaalde configuratiestadium overwogen.

## Dispatcher uitvoeren met een toegewezen systeemgebruiker {#run-dispatcher-with-a-dedicated-system-user}

Configureer de Dispatcher zodanig dat een toegewijde, minst geprivilegieerde gebruikersaccount de webserver uitvoert. Adobe raadt u aan alleen schrijftoegang te verlenen tot de cachemap van Dispatcher.

IIS-gebruikers moeten hun website ook als volgt configureren:

1. In het fysieke weg plaatsen voor uw website, uitgezochte **verbind als specifieke gebruiker**.
1. Stel de gebruiker in.

## Ontkenning van service-aanvallen (DoS) voorkomen {#prevent-denial-of-service-dos-attacks}

Een ontkenning van de dienst (Dos) aanval is een poging om een computermiddel niet beschikbaar te maken aan zijn voorgenomen gebruikers.

Op het niveau van Dispatcher, zijn er twee methodes om aanvallen van Dos te verhinderen: [&#x200B; Filters &#x200B;](https://experienceleague.adobe.com/nl/docs#/filter)

* Gebruik mod_rewrite module (bijvoorbeeld, [&#x200B; Apache 2.4 &#x200B;](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) om URL validaties uit te voeren (als de URL patroonregels niet te complex zijn).

* Verhinder Dispatcher URLs met valse uitbreidingen in het voorgeheugen onderbrengen door [&#x200B; filters &#x200B;](dispatcher-configuration.md#configuring-access-to-content-filter) te gebruiken.\
  Wijzig bijvoorbeeld de caching-regels om caching te beperken tot de verwachte mime-typen, zoals:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

  Een dossier van de voorbeeldconfiguratie kan voor [&#x200B; worden gezien die externe toegang &#x200B;](#restrict-access) beperken. Het bevat beperkingen voor mime-typen.

Als u volledige functionaliteit wilt inschakelen voor de publicatie-instanties, configureert u filters om toegang tot de volgende knooppunten te voorkomen:

* `/etc/`
* `/libs/`

Configureer vervolgens filters om toegang tot de volgende knooppuntpaden toe te staan:

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS, CSS en JSON)
* `/libs/cq/security/userinfo.json` (CQ-gebruikersgegevens)
* `/libs/granite/security/currentuser.json` (**gegevens moeten niet in het voorgeheugen ondergebracht worden**)

* `/libs/cq/i18n/*` (Internationalisatie)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Dispatcher configureren om CSRF-aanvallen te voorkomen {#configure-dispatcher-to-prevent-csrf-attacks}

AEM verstrekt a [&#x200B; kader &#x200B;](https://experienceleague.adobe.com/nl/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#verification-steps) gericht op het verhinderen van de aanvallen van het Verzoek van de Versmederij van de Depositovergangen van de Depositovergangen. Om dit kader goed te gebruiken, lijst van gewenste personen symbolische steun CSRF in Dispatcher door het volgende te doen:

1. Een filter maken om het `/libs/granite/csrf/token.json` -pad toe te staan;
1. Voeg de header `CSRF-Token` toe aan de sectie `clientheaders` van de Dispatcher-configuratie.

## Klikaanvallen voorkomen {#prevent-clickjacking}

Om te voorkomen dat wordt geklikt, raadt Adobe u aan uw webserver te configureren om de `X-FRAME-OPTIONS` HTTP-header die is ingesteld op `SAMEORIGIN` , op te geven.

Voor meer informatie bij klikjacking, zie de [&#x200B; plaats van OWASP &#x200B;](https://owasp.org/www-community/attacks/Clickjacking).

## Een penetratietest uitvoeren {#perform-a-penetration-test}

Adobe raadt u aan een penetratietest van uw AEM-infrastructuur uit te voeren voordat u verdergaat met de productie.

