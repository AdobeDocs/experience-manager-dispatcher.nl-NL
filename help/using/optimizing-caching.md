---
title: Een website optimaliseren voor cacheprestaties
description: Leer hoe u uw website ontwerpt om de voordelen van caching te maximaliseren.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
redirecttarget: https://helpx.adobe.com/nl/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '1128'
ht-degree: 0%

---


# Een website optimaliseren voor cacheprestaties {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher-versies zijn onafhankelijk van AEM. U bent mogelijk omgeleid naar deze pagina als u een koppeling naar de documentatie van Dispatcher hebt gevolgd. Die koppeling is ingesloten in de documentatie voor een vorige versie van AEM.

Dispatcher biedt verschillende ingebouwde mechanismen waarmee u de prestaties kunt optimaliseren. In deze sectie wordt uitgelegd hoe u uw website kunt ontwerpen om de voordelen van caching te maximaliseren.

>[!NOTE]
>
>Het kan u helpen herinneren dat de Dispatcher het geheime voorgeheugen op een standaardWebserver opslaat. Dit betekent dat u:
>
>* kan alles opslaan dat u als pagina kunt opslaan en aanvragen via een URL
>* kan geen andere dingen opslaan, zoals HTTP-headers, cookies, sessiegegevens en formuliergegevens.
>
>In het algemeen, impliceren vele caching strategieën het selecteren van goede URLs en het verlaten van deze extra gegevens.

## Codering van pagina&#39;s consistent gebruiken {#using-consistent-page-encoding}

HTTP-aanvraagheaders worden niet in het cachegeheugen opgeslagen. Er kunnen zich dus problemen voordoen als u pagina-coderingsgegevens opslaat in de header. Wanneer Dispatcher dan een pagina uit het cachegeheugen bedient, wordt de standaardcodering van de webserver gebruikt voor de pagina. Dit probleem kan op twee manieren worden voorkomen:

* Als u slechts één codering gebruikt, moet u ervoor zorgen dat de codering die op de webserver wordt gebruikt, gelijk is aan de standaardcodering van de AEM-website.
* Als u de codering wilt instellen, gebruikt u een `<META>` -tag in de sectie HTML `head` , zoals in het volgende voorbeeld:

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## Gebruik geen URL-parameters {#avoid-url-parameters}

Vermijd indien mogelijk URL-parameters voor pagina&#39;s die u in cache wilt plaatsen. Bijvoorbeeld, als u een beeldgalerij hebt, wordt volgende URL nooit in het voorgeheugen ondergebracht (tenzij Dispatcher dienovereenkomstig [ wordt gevormd ](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

U kunt deze parameters echter als volgt in de pagina-URL plaatsen:

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>Deze URL roept dezelfde pagina en dezelfde sjabloon aan als gallery.html. In de sjabloondefinitie kunt u opgeven welk script de pagina rendert of u kunt hetzelfde script voor alle pagina&#39;s gebruiken.

## Aanpassen via URL {#customize-by-url}

Als u gebruikers toestaat de fontgrootte (of een andere lay-outaanpassing) te wijzigen, moet u ervoor zorgen dat de verschillende aanpassingen worden weerspiegeld in de URL.

Cookies worden bijvoorbeeld niet in de cache opgeslagen. Als u de fontgrootte opslaat in een cookie (of een vergelijkbaar mechanisme), blijft de fontgrootte dus niet behouden voor de pagina in de cache. Als gevolg hiervan retourneert Dispatcher willekeurige documenten van een willekeurige tekengrootte.

Door de tekengrootte als kiezer in de URL op te nemen voorkomt u dit probleem:

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>Voor de meeste lay-outaspecten is het ook mogelijk om stijlbladen, of cliënt-zijmanuscripten, of allebei te gebruiken. Of beide werken goed met caching.
>
>Deze methode is ook handig voor een afdrukversie, waar u bijvoorbeeld een URL kunt gebruiken:
>
>`www.myCompany.com/news/main.print.html`
>
>Met behulp van de scriptglobling van de sjabloondefinitie kunt u een afzonderlijk script opgeven dat de afdrukpagina&#39;s rendert.

## Afbeeldingsbestanden die als titels worden gebruikt, ongeldig maken {#invalidating-image-files-used-as-titles}

Als u paginatitels of andere tekst hebt weergegeven als afbeeldingen, slaat u de bestanden op zodat deze worden verwijderd bij een update van de inhoud op de pagina:

1. Plaats de afbeelding in dezelfde map als de pagina.
1. Gebruik de volgende naamgevingsindeling voor het afbeeldingsbestand:

   `<page file name>.<image file name>`

U kunt bijvoorbeeld de titel van de pagina myPage.html opslaan in het bestand myPage.title.gif. Dit bestand wordt automatisch verwijderd als de pagina wordt bijgewerkt. Wijzigingen in de paginatitel worden dus automatisch doorgevoerd in de cache.

>[!NOTE]
>
>Het afbeeldingsbestand bestaat niet noodzakelijkerwijs in de AEM-instantie. U kunt een script gebruiken waarmee het afbeeldingsbestand dynamisch wordt gemaakt. Dispatcher slaat het bestand vervolgens op de webserver op.

## De voor navigatie gebruikte afbeeldingsbestanden ongeldig maken {#invalidating-image-files-used-for-navigation}

Als u foto&#39;s gebruikt voor de navigatie-items, is de methode in principe hetzelfde als bij titels, maar dan is deze iets complexer. Sla alle navigatieafbeeldingen op de doelpagina&#39;s op. Als u twee afbeeldingen gebruikt voor normaal en actief, kunt u de volgende scripts gebruiken:

* Een script waarmee de pagina als normaal wordt weergegeven.
* Een script dat `.normal` verwerkt, vraagt om het normale beeld en retourneert dit.
* Een script dat `.active` verwerkt, vraagt om het geactiveerde beeld en retourneert dit.

Het is belangrijk dat u deze afbeeldingen maakt met dezelfde naamgevingsgreep als de pagina om ervoor te zorgen dat deze afbeeldingen en de pagina worden verwijderd bij het bijwerken van de inhoud.

Voor pagina&#39;s die niet worden gewijzigd, blijven de afbeeldingen in het cachegeheugen staan, hoewel de pagina&#39;s zelf automatisch ongeldig worden gemaakt.

## Personalization {#personalization}

De Dispatcher kan gepersonaliseerde gegevens niet in het cachegeheugen opslaan. Het wordt daarom aangeraden de personalisatie te beperken tot waar dat nodig is. Ter illustratie:

* Als u een vrij aanpasbare startpagina gebruikt, moet die pagina telkens worden samengesteld wanneer een gebruiker erom vraagt.
* Als u daarentegen een keuze hebt uit tien verschillende startpagina&#39;s, kunt u elk van deze in cache plaatsen, waardoor de prestaties verbeteren.

>[!NOTE]
>
>Als u elke pagina personaliseert (bijvoorbeeld door de naam van de gebruiker in de titelbalk te plaatsen) kunt u deze niet in cache plaatsen, wat een grote invloed op de prestaties kan hebben.
>
>U kunt echter wel het volgende doen:
>
>* Gebruik iFrames om de pagina op te splitsen in één onderdeel dat voor alle gebruikers hetzelfde is en één onderdeel dat voor alle pagina&#39;s van de gebruiker hetzelfde is. Vervolgens kunt u beide onderdelen in de cache plaatsen.
>* gebruik client-side JavaScript om persoonlijke gegevens weer te geven. U moet er echter voor zorgen dat de pagina nog steeds correct wordt weergegeven als een gebruiker JavaScript uitschakelt.
>

## Vaste verbindingen {#sticky-connections}

[ de Vaste verbindingen ](dispatcher.md#TheBenefitsofLoadBalancing) zorgen ervoor dat de documenten voor één gebruiker allen op de zelfde server samengesteld zijn. Als een gebruiker deze map verlaat en er later weer naar terugkeert, blijft de verbinding behouden. Definieer één map zodat deze alle documenten kan bevatten waarvoor kleverige verbindingen voor de website nodig zijn. Probeer er geen andere documenten in op te nemen. Dit is van invloed op de taakverdeling als u gepersonaliseerde pagina&#39;s en sessiegegevens gebruikt.

## MIME-typen {#mime-types}

Er zijn twee manieren waarop een browser het type bestand kan bepalen:

1. Door de extensie (bijvoorbeeld .html, .gif en .jpg)
1. Door het MIME-type dat de server met het bestand verzendt.

Voor de meeste bestanden wordt het MIME-type geïmpliceerd in de bestandsextensie:

1. Door de extensie (bijvoorbeeld .html, .gif en .jpg)
1. Door het MIME-type dat de server met het bestand verzendt.

Als de bestandsnaam geen extensie heeft, wordt deze weergegeven als onbewerkte tekst.

Het MIME-type maakt deel uit van de HTTP-header en wordt daarom niet in de cache opgeslagen door Dispatcher. Uw AEM-toepassing kan bestanden retourneren die geen herkende bestandsextensie hebben. Als de bestanden in plaats daarvan afhankelijk zijn van het MIME-type, worden deze bestanden mogelijk onjuist weergegeven.

Volg de onderstaande richtlijnen om ervoor te zorgen dat bestanden correct in het cachegeheugen worden opgeslagen:

* Zorg ervoor dat bestanden altijd de juiste extensie hebben.
* Vermijd generische de manuscripten van de dossierserver, die URLs zoals download.jsp?file=2214 hebben. Schrijf het script opnieuw, zodat het URL&#39;s gebruikt die de bestandsspecificatie bevatten. Voor het vorige voorbeeld is dit `download.2214.pdf` .

