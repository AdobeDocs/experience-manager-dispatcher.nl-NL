---
title: Adobe Experience Manager Dispatcher configureren om CSRF-aanvallen te voorkomen
description: Leer hoe te om de Dispatcher van Adobe Experience Manager te vormen om de aanvallen van het Verzoek van de Vervalsing van de Verkeer van de Deposito's van de Intersite te verhinderen.
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '235'
ht-degree: 0%

---

# Adobe Experience Manager Dispatcher configureren om CSRF-aanvallen te voorkomen{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM (Adobe Experience Manager) biedt een raamwerk dat bedoeld is om aanvallen met vervalsingen door verschillende sites te voorkomen. Als u op de juiste manier gebruik wilt maken van dit framework, brengt u de volgende wijzigingen aan in uw configuratie van Dispatcher:

>[!NOTE]
>
>Ben zeker om de regelaantallen in de hieronder voorbeelden bij te werken die op uw bestaande configuratie worden gebaseerd. Herinner dat de Ontvangers de laatste passende regel gebruiken om toe te staan of te ontkennen, zodat plaats de regels dichtbij de bodem van uw bestaande lijst.

1. In de `/clientheaders` deel van uw `author-farm.any` en `publish-farm.any`, voegt u de volgende vermelding toe onder aan de lijst:\
   `CSRF-Token`
1. In de /filters sectie van uw `author-farm.any` en `publish-farm.any` of `publish-filters.any` bestand, voeg de volgende regel toe om aanvragen voor `/libs/granite/csrf/token.json` via de Dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`

1. Onder de `/cache /rules` deel van uw `publish-farm.any`voegt u een regel toe om te voorkomen dat de Dispatcher het cachegeheugen van de `token.json` bestand. Auteurs omzeilen caching doorgaans, dus u hoeft de regel niet toe te voegen aan uw `author-farm.any`.

   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Om te bevestigen dat de configuratie werkt, bekijk de dispatcher.log in DEBUG wijze. Het kan u helpen bevestigen dat `token.json` om ervoor te zorgen dat deze niet in cache wordt geplaatst of wordt geblokkeerd door filters. U zou berichten moeten zien gelijkend op:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

U kunt ook controleren of aanvragen succesvol zijn in uw Apache `access_log`. Verzoeken om &quot;/libs/granite/csrf/token.json zouden een HTTP 200 statuscode moeten terugkeren.
