---
title: De Dispatcher-beveiligingscontrolelijst
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
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '590'
ht-degree: 0%

---

# De Dispatcher-beveiligingscontrolelijst{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe raadt u aan de volgende checklist in te vullen voordat u verdergaat met de productie.

>[!CAUTION]
>
>Vul de lijst Beveiligingscontrole van uw versie van AEM in voordat u live gaat. Zie de overeenkomstige [ documentatie van Adobe Experience Manager ](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist).

## De nieuwste versie van Dispatcher gebruiken {#use-the-latest-version-of-dispatcher}

Installeer de nieuwste beschikbare versie die beschikbaar is voor uw platform. Voer een upgrade uit op uw Dispatcher-exemplaar om de nieuwste versie te gebruiken en zo te profiteren van product- en beveiligingsverbeteringen. Zie [ Installerend Dispatcher ](dispatcher-install.md).

>[!NOTE]
>
>U kunt de huidige versie van uw Dispatcher-installatie controleren door het Dispatcher-logbestand te bekijken.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Als u het logbestand wilt zoeken, controleert u de Dispatcher-configuratie in uw `httpd.conf` .

## Clients beperken die uw cache kunnen leegmaken {#restrict-clients-that-can-flush-your-cache}

Adobe adviseert dat u [ de cliënten beperkt die uw geheime voorgeheugen kunnen leegmaken.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

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

Wanneer het vormen van de Dispatcher, beperking externe toegang zoveel mogelijk. Zie [ Sectie van het Voorbeeld /filter ](dispatcher-configuration.md#main-pars_184_1_title) in de documentatie van Dispatcher.

## Zorg ervoor dat toegang tot administratieve URL&#39;s wordt geweigerd {#make-sure-access-to-administrative-urls-is-denied}

Zorg ervoor dat u filters gebruikt om externe toegang tot eventuele beheerURL&#39;s, zoals de webconsole, te blokkeren.

Zie [ het Testen de Veiligheid van Dispatcher ](dispatcher-configuration.md#testing-dispatcher-security) voor een lijst van URLs die moet worden geblokkeerd.

## Lijsten van gewenste personen gebruiken in plaats van Lijsten van gewezen personen {#use-allowlists-instead-of-blocklists}

Lijsten van gewenste personen zijn een betere manier om toegangscontrole te verlenen aangezien zij inherent, veronderstellen zij dat alle toegangsverzoeken zouden moeten worden ontkend tenzij zij uitdrukkelijk deel van de lijst van gewenste personen uitmaken. Dit model verstrekt meer restrictieve controle over nieuwe verzoeken die nog niet zouden kunnen zijn herzien of tijdens een bepaalde configuratiestadium overwogen.

## Dispatcher uitvoeren met een specifieke systeemgebruiker {#run-dispatcher-with-a-dedicated-system-user}

Zorg er bij het configureren van de Dispatcher voor dat de webserver wordt uitgevoerd door een toegewijde gebruiker met de minste toegangsrechten. U wordt aangeraden alleen schrijftoegang te verlenen tot de Dispatcher-cachemap.

IIS-gebruikers moeten hun website ook als volgt configureren:

1. In het fysieke weg plaatsen voor uw website, uitgezochte **verbind als specifieke gebruiker**.
1. Stel de gebruiker in.

## Ontkenning van service-aanvallen (DoS) voorkomen {#prevent-denial-of-service-dos-attacks}

Een ontkenning van de dienst (Dos) aanval is een poging om een computermiddel niet beschikbaar te maken aan zijn voorgenomen gebruikers.

Op het niveau van Dispatcher, zijn er twee methodes om aanvallen van Dos te verhinderen: [ Filters ](https://experienceleague.adobe.com/en/docs#/filter)

* Gebruik mod_rewrite module (bijvoorbeeld, [ Apache 2.4 ](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) om URL validaties uit te voeren (als de URL patroonregels niet te complex zijn).

* Verhinder Dispatcher URLs met valse uitbreidingen in het voorgeheugen onderbrengen door [ filters ](dispatcher-configuration.md#configuring-access-to-content-filter) te gebruiken.\
  Wijzig bijvoorbeeld de caching-regels om caching te beperken tot de verwachte mime-typen, zoals:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

  Een dossier van de voorbeeldconfiguratie kan voor [ worden gezien die externe toegang ](#restrict-access) beperken. Het bevat beperkingen voor mime-typen.

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

AEM verstrekt a [ kader ](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#verification-steps) gericht op het verhinderen van de aanvallen van de Versmeding van het Verzoek van de Depositovergangen van de Depositovergangen. Om dit kader goed te gebruiken, lijst van gewenste personen symbolische steun CSRF in Dispatcher door het volgende te doen:

1. Een filter maken om het `/libs/granite/csrf/token.json` -pad toe te staan;
1. Voeg de header `CSRF-Token` toe aan de sectie `clientheaders` van de Dispatcher-configuratie.

## Klikaanvallen voorkomen {#prevent-clickjacking}

Om klikaanvallen te voorkomen, adviseert de Adobe dat u uw webserver vormt om `X-FRAME-OPTIONS` kopbal te verstrekken die aan `SAMEORIGIN` wordt geplaatst.

Voor meer informatie bij klikjacking, zie de [ plaats van OWASP ](https://owasp.org/www-community/attacks/Clickjacking).

## Een beveiligingstest uitvoeren {#perform-a-penetration-test}

Adobe beveelt ten zeerste aan om een penetratietest van uw AEM-infrastructuur uit te voeren voordat u verdergaat met de productie.

