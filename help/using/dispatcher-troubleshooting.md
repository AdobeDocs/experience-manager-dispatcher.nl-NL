---
title: Problemen met Dispatcher oplossen
description: Leer problemen met Dispatcher op te lossen.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '539'
ht-degree: 0%

---

# Problemen met Dispatcher oplossen {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher-versies zijn onafhankelijk van AEM. De Dispatcher-documentatie is echter ingesloten in de AEM documentatie. Gebruik altijd de Dispatcher-documentatie die is ingesloten in de documentatie voor de meest recente versie van AEM.
>
>U bent mogelijk omgeleid naar deze pagina als u een koppeling naar de documentatie van Dispatcher hebt gevolgd. Deze koppeling is ingesloten in de documentatie voor een vorige versie van AEM.

>[!NOTE]
>
>Controleer de [ Kennisbank van Dispatcher ](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [ het Oplossen van problemen van Dispatcher die ](https://experienceleague.adobe.com/search.html?lang=nl-NL#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager]), en [ Veelgestelde Veelgestelde vragen van de Kwesties van Dispatcher Hoogste ](dispatcher-faq.md) voor verdere informatie leegmaken.

## Controleer de basisconfiguratie {#check-the-basic-configuration}

Zoals altijd, zijn de eerste stappen de grondbeginselen te controleren:

* [Basisbewerking bevestigen](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Controleer alle logbestanden voor uw webserver en Dispatcher. Indien noodzakelijk, verhoog `loglevel` dat voor het [ registreren van Dispatcher ](/help/using/dispatcher-configuration.md#logging) wordt gebruikt.

* [ controleer uw configuratie ](/help/using/dispatcher-configuration.md):

   * Hebt u meerdere verzenders?

      * Hebt u vastgesteld welke Dispatcher de website/pagina verwerkt die u onderzoekt?

   * Hebt u filters geïmplementeerd?

      * Hebben deze filters invloed op de zaak die u onderzoekt?

## IIS Diagnostic Tools {#iis-diagnostic-tools}

IIS verstrekt diverse spoorhulpmiddelen, afhankelijk van de daadwerkelijke versie:

* IIS 6 - diagnostische hulpmiddelen IIS kunnen worden gedownload en worden gevormd
* IIS 7 - het vinden is volledig geïntegreerd

Deze hulpmiddelen kunnen u helpen activiteit controleren.

## IIS en 404 niet gevonden {#iis-and-not-found}

Wanneer u IIS gebruikt, kan het zijn dat `404 Not Found` in verschillende scenario&#39;s wordt geretourneerd. Zo ja, zie de volgende artikelen in de Knowledge Base.

* [ IIS 6/7: De methode van de POST keert 404 ](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html) terug
* [ IIS 6: Verzoeken aan URLs die de basisweg `/bin` bevatten terugkeer a `404 Not Found` ](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

Controleer ook of de Dispatcher cache-hoofdmap en de hoofdmap van het IIS-document op dezelfde map zijn ingesteld.

## Problemen bij het verwijderen van workflowmodellen {#problems-deleting-workflow-models}

**Symptomen**

Problemen bij het verwijderen van workflowmodellen bij het openen van een AEM auteur-instantie via de Dispatcher.

**Stappen om te reproduceren:**

1. Meld u aan bij de instantie van uw auteur (bevestig dat aanvragen worden gerouteerd via de Dispatcher).
1. Maak een workflow, bijvoorbeeld met Titel ingesteld op workflowToDelete.
1. Controleer of de workflow is gemaakt.
1. Selecteer en klik het werkschema met de rechtermuisknop aan, dan klik **Schrapping**.

1. Klik **ja** om te bevestigen.
1. Er wordt een foutbericht weergegeven met het volgende:\
   `ERROR 'Could not delete workflow model!!`.

**Resolutie**

Voeg de volgende kopteksten aan de `/clientheaders` sectie van uw `dispatcher.any` dossier toe:

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## Interferentie met mod_dir (Apache) {#interference-with-mod-dir-apache}

In dit proces wordt beschreven hoe de Dispatcher communiceert met `mod_dir` binnen de Apache-webserver, aangezien dit tot verschillende, mogelijk onverwachte effecten kan leiden:

### Apache 1.3 {#apache}

In Apache 1.3 handelt `mod_dir` elke aanvraag af waarbij de URL wordt toegewezen aan een map in het bestandssysteem.

Het zal ofwel:

* doorsturen van de aanvraag naar een bestaand `index.html` -bestand
* een mappenlijst genereren

Wanneer de Dispatcher is ingeschakeld, worden dergelijke verzoeken verwerkt door zichzelf te registreren als handler voor het inhoudstype `httpd/unix-directory` .

### Apache 2.x {#apache-x}

In Apache 2.x zijn de dingen anders. Een module kan verschillende stadia van het verzoek, zoals correctie URL behandelen. `mod_dir` handelt dit werkgebied af door een aanvraag (wanneer de URL naar een map wordt toegewezen) om te leiden naar de URL met een toevoeging van `/` .

Dispatcher onderschept de correctie `mod_dir` niet, maar handelt de aanvraag volledig af naar de omgeleide URL (met `/` toegevoegd). Dit proces kan problemen opleveren als de externe server (bijvoorbeeld AEM) aanvragen aan `/a_path` anders afhandelt dan aanvragen aan `/a_path/` (wanneer `/a_path` wordt toegewezen aan een bestaande map).

Als dit gebeurt, moet u:

* disable `mod_dir` voor de `Directory` - of `Location` -substructuur die door de Dispatcher wordt afgehandeld

* use `DirectorySlash Off` gebruiken om `mod_dir` not to append `/` te configureren
