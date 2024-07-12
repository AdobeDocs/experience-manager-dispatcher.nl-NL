---
title: Adobe Experience Manager Dispatcher configureren om CSRF-aanvallen te voorkomen
description: Leer hoe u de Adobe Experience Manager Dispatcher configureert om aanvallen met smeedstukken tussen verschillende sites te voorkomen.
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '235'
ht-degree: 0%

---

# Adobe Experience Manager Dispatcher configureren om CSRF-aanvallen te voorkomen{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM (Adobe Experience Manager) biedt een raamwerk dat bedoeld is om aanvallen met vervalsingen door verschillende sites te voorkomen. Als u dit framework op de juiste manier wilt gebruiken, brengt u de volgende wijzigingen aan in uw Dispatcher-configuratie:

>[!NOTE]
>
>Ben zeker om de regelaantallen in de hieronder voorbeelden bij te werken die op uw bestaande configuratie worden gebaseerd. Herinner dat de Ontvangers de laatste passende regel gebruiken om toe te staan of te ontkennen, zodat plaats de regels dichtbij de bodem van uw bestaande lijst.

1. Voeg in de sectie `/clientheaders` van de `author-farm.any` en `publish-farm.any` de volgende vermelding toe onder aan de lijst:\
   `CSRF-Token`
1. Voeg in de sectie /filters van het `author-farm.any` - en `publish-farm.any` - of `publish-filters.any` -bestand de volgende regel toe om aanvragen voor `/libs/granite/csrf/token.json` via de Dispatcher toe te staan.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`

1. Voeg onder de sectie `/cache /rules` van uw `publish-farm.any` een regel toe om te voorkomen dat de Dispatcher het `token.json` -bestand in de cache plaatst. Auteurs omzeilen caching doorgaans, dus u hoeft de regel niet aan uw `author-farm.any` toe te voegen.

   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Om te bevestigen dat de configuratie werkt, bekijk de dispatcher.log in DEBUG wijze. U kunt hiermee controleren of het `token.json` -bestand wordt opgeslagen in de cache of geblokkeerd door filters. U zou berichten moeten zien gelijkend op:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

U kunt ook controleren of aanvragen succesvol zijn in uw Apache `access_log` . Verzoeken om &quot;/libs/granite/csrf/token.json zouden een HTTP 200 statuscode moeten terugkeren.
