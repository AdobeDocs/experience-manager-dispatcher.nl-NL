---
title: Problemen met verzending van problemen oplossen
description: Leer problemen met Dispatcher oplossen.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '538'
ht-degree: 0%

---

# Problemen met verzending van problemen oplossen {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>De versies van de Verzender zijn onafhankelijk van AEM, nochtans wordt de documentatie van de Verzender ingebed in de AEM documentatie. Gebruik altijd de Dispatcher-documentatie die is ingesloten in de documentatie voor de meest recente versie van AEM.
>
>U bent mogelijk omgeleid naar deze pagina als u een koppeling naar de Dispatcher-documentatie hebt gevolgd die is ingesloten in de documentatie voor een vorige versie van AEM.

>[!NOTE]
>
>Controleer de [Dispatcher Knowledge Base](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [Problemen met het leegmaken van de Dispatcher oplossen](https://experienceleague.adobe.com/search.html?lang=en#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager])en de [Veelgestelde vragen over de meest voorkomende problemen met Verzender](dispatcher-faq.md) voor nadere informatie.

## Controleer de basisconfiguratie {#check-the-basic-configuration}

Zoals altijd zijn de eerste stappen het controleren van de grondbeginselen:

* [Basisbewerking bevestigen](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Controleer alle logbestanden op uw webserver en Dispatcher. Indien nodig, verhoog de dosis `loglevel` gebruikt voor de Dispatcher [logboekregistratie](/help/using/dispatcher-configuration.md#logging).

* [Controleer uw configuratie](/help/using/dispatcher-configuration.md):

   * Hebt u meerdere verzenders?

      * Hebt u bepaald welke Dispatcher de website/pagina verwerkt die u onderzoekt?

   * Hebt u filters geïmplementeerd?

      * Hebben deze filters invloed op de zaak die u onderzoekt?

## IIS Diagnostic Tools {#iis-diagnostic-tools}

IIS verstrekt diverse spoorhulpmiddelen, afhankelijk van de daadwerkelijke versie:

* IIS 6 - diagnostische hulpmiddelen IIS kunnen worden gedownload en worden gevormd
* IIS 7 - het vinden is volledig geïntegreerd

Deze hulpmiddelen kunnen u helpen activiteit controleren.

## IIS en 404 niet gevonden {#iis-and-not-found}

Wanneer u IIS gebruikt, kunt u `404 Not Found` teruggegeven in verschillende scenario&#39;s. Zo ja, zie de volgende artikelen in de Knowledge Base.

* [IIS 6/7: de methode van de POST keert 404 terug](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6: Verzoeken naar URL&#39;s die het basispad bevatten `/bin` een `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

Controleer ook of de hoofdmap van de Dispatcher-cache en de hoofdmap van het IIS-document zijn ingesteld op dezelfde map.

## Problemen bij het verwijderen van workflowmodellen {#problems-deleting-workflow-models}

**Symptomen**

Problemen bij het verwijderen van workflowmodellen wanneer een AEM auteur-instantie wordt benaderd via de Dispatcher.

**Stappen om te reproduceren:**

1. Meld u aan bij de instantie van de auteur (bevestig dat aanvragen worden gerouteerd via de Dispatcher).
1. Maak een workflow, bijvoorbeeld met Titel ingesteld op workflowToDelete.
1. Controleer of de workflow is gemaakt.
1. Selecteer en klik met de rechtermuisknop op de workflow en klik vervolgens op **Verwijderen**.

1. Klikken **Ja** ter bevestiging.
1. Er wordt een foutbericht weergegeven met het volgende:\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**Resolutie**

Voeg de volgende kopteksten aan toe `/clientheaders` deel van uw `dispatcher.any` bestand:

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

In dit proces wordt beschreven hoe de Dispatcher communiceert met `mod_dir` in de Apache-webserver, omdat dit tot verschillende, mogelijk onverwachte effecten kan leiden:

### Apache 1.3 {#apache}

In Apache 1.3: `mod_dir` handelt elke aanvraag af waar de URL wordt toegewezen aan een map in het bestandssysteem.

Het zal ofwel:

* doorsturen van het verzoek naar een bestaande `index.html` file
* een mappenlijst genereren

Wanneer de Dispatcher wordt toegelaten, verwerkt het dergelijke verzoeken door zich als manager voor het inhoudstype te registreren `httpd/unix-directory`.

### Apache 2.x {#apache-x}

In Apache 2.x zijn de dingen anders. Een module kan verschillende stadia van het verzoek, zoals correctie URL behandelen. De `mod_dir` handelt dit werkgebied af door een aanvraag (wanneer de URL naar een map is toegewezen) om te leiden naar de URL met een `/` toegevoegd.

Dispatcher onderschept het `mod_dir` correctie, maar behandelt het verzoek volledig aan opnieuw geleide URL (namelijk met `/` toegevoegd). Dit proces zou een probleem kunnen vormen als de verre server (bijvoorbeeld, AEM) verzoeken aan behandelt `/a_path` anders dan voor verzoeken aan `/a_path/` (wanneer `/a_path` verwijst naar een bestaande map).

Als dit gebeurt, moet u:

* disable `mod_dir` voor de `Directory` of `Location` substructuur die wordt afgehandeld door de Dispatcher

* gebruiken `DirectorySlash Off` om te vormen `mod_dir` niet toevoegen `/`
