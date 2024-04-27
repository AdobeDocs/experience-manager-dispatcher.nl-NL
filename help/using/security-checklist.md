---
title: De beveiligingscontrolelijst voor verzending
description: Een lijst met beveiligingscontroles die moet worden ingevuld voordat de productie wordt voortgezet.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '591'
ht-degree: 0%

---

# De beveiligingscontrolelijst voor verzending{#the-dispatcher-security-checklist}

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
>Vul de lijst Beveiligingscontrole van uw versie van AEM in voordat u live gaat. Zie het bijbehorende [Adobe Experience Manager-documentatie](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist).

## De nieuwste versie van Dispatcher gebruiken {#use-the-latest-version-of-dispatcher}

Installeer de nieuwste versie die beschikbaar is voor uw platform. Zorg ervoor dat u de Dispatcher-instantie upgradet, zodat u de nieuwste versie gebruikt om te profiteren van de verbeteringen op het gebied van product en beveiliging. Zie [Dispatcher installeren](dispatcher-install.md).

>[!NOTE]
>
>Controleer de huidige versie van uw Dispatcher-installatie door het logboekbestand van Dispatcher te bekijken.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Om het logboekdossier te vinden, inspecteer de configuratie van de Verzender in uw `httpd.conf`.

## Clients beperken die uw cache kunnen leegmaken {#restrict-clients-that-can-flush-your-cache}

Adobe beveelt aan [beperkt de clients die uw cache kunnen leegmaken.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## HTTPS inschakelen voor beveiliging van transportlagen {#enable-https-for-transport-layer-security}

Adobe raadt aan de HTTPS-transportlaag in te schakelen voor zowel de auteur als de publicatieversie.

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

Wanneer het vormen van de Dispatcher, beperking externe toegang zoveel mogelijk. Zie [Voorbeeld-/filtersectie](dispatcher-configuration.md#main-pars_184_1_title) in de Dispatcher documentatie.

## Zorg ervoor dat toegang tot administratieve URL&#39;s wordt geweigerd {#make-sure-access-to-administrative-urls-is-denied}

Zorg ervoor dat u filters gebruikt om externe toegang tot eventuele beheerURL&#39;s, zoals de webconsole, te blokkeren.

Zie [Beveiliging van Dispatcher testen](dispatcher-configuration.md#testing-dispatcher-security) voor een lijst met URL&#39;s die moeten worden geblokkeerd.

## Lijsten van gewenste personen gebruiken in plaats van Lijsten van gewezen personen {#use-allowlists-instead-of-blocklists}

Lijsten van gewenste personen zijn een betere manier om toegangscontrole te verlenen aangezien zij inherent, veronderstellen zij dat alle toegangsverzoeken zouden moeten worden ontkend tenzij zij uitdrukkelijk deel van de lijst van gewenste personen uitmaken. Dit model verstrekt meer restrictieve controle over nieuwe verzoeken die nog niet zouden kunnen zijn herzien of tijdens een bepaalde configuratiestadium overwogen.

## Dispatcher uitvoeren met een specifieke systeemgebruiker {#run-dispatcher-with-a-dedicated-system-user}

Wanneer het vormen van de Dispatcher, zou u moeten ervoor zorgen dat de Webserver door een specifieke gebruiker met minste voorrechten wordt in werking gesteld. Het wordt aanbevolen alleen schrijftoegang te verlenen tot de cachemap van Dispatcher.

IIS-gebruikers moeten hun website ook als volgt configureren:

1. Selecteer in de fysieke padinstelling voor uw website de optie **Verbinding maken als specifieke gebruiker**.
1. Stel de gebruiker in.

## Ontkenning van service-aanvallen (DoS) voorkomen {#prevent-denial-of-service-dos-attacks}

Een ontkenning van de dienst (Dos) aanval is een poging om een computermiddel niet beschikbaar te maken aan zijn voorgenomen gebruikers.

Op het niveau van de Verzender, zijn er [twee methodes om aanvallen van Dos te verhinderen te vormen](https://experienceleaguecommunities.adobe.com/t5/adobe-experience-manager/configure-aem-dispatcher-to-prevent-dos-attacks-aem-community/m-p/447780).

* Gebruik de module mod_rewrite (bijvoorbeeld [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) om URL-validaties uit te voeren (als de URL-patroonregels niet te complex zijn).

* Vermijd dat de Dispatcher URL&#39;s in cache plaatst met onjuiste extensies door [filters](dispatcher-configuration.md#configuring-access-to-conten-tfilter).\
  Wijzig bijvoorbeeld de caching-regels om caching te beperken tot de verwachte mime-typen, zoals:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

  Een voorbeeldconfiguratiebestand is zichtbaar voor [beperking van externe toegang](#restrict-access)Dit omvat beperkingen voor mime-typen.

Om volledige functionaliteit op de publiceer instanties veilig toe te laten, vorm filters om toegang tot de volgende knopen te verhinderen:

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
* `/libs/granite/security/currentuser.json` (**gegevens mogen niet in cache worden geplaatst**)

* `/libs/cq/i18n/*` (Internationalisatie)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Dispatcher configureren om CSRF-aanvallen te voorkomen {#configure-dispatcher-to-prevent-csrf-attacks}

AEM biedt [kader](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#verification-steps) gericht op het voorkomen van aanvallen van smeedstukken tussen verschillende sites. Om dit kader behoorlijk te gebruiken, moet u symbolensteun CSRF in de Verzender lijsten van gewenste personen.
<!-- OLD URL ABOVE USED TO BE https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps -->
U kunt dit bereiken door het volgende te doen:

1. Een filter maken om de `/libs/granite/csrf/token.json` pad;
1. Voeg de `CSRF-Token` aan de `clientheaders` sectie van de configuratie van de Verzender.

## Klikaanvallen voorkomen {#prevent-clickjacking}

Om klikaanvallen te verhinderen, adviseert de Adobe dat u uw webserver vormt om `X-FRAME-OPTIONS` HTTP-header ingesteld op `SAMEORIGIN`.

Voor meer informatie over klikjacking, zie [OWASP-site](https://owasp.org/www-community/attacks/Clickjacking).

## Een beveiligingstest uitvoeren {#perform-a-penetration-test}

Adobe raadt u aan een penetratietest van uw AEM uit te voeren voordat u verdergaat met de productie.
