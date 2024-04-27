---
title: Dispatcher configureren om CSRF-aanvallen te voorkomen
description: Leer hoe te om AEM Dispatcher te vormen om de aanvallen van de Vervalsing van het Verzoek van de Verkeer van de Departement van de Deite te verhinderen.
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '217'
ht-degree: 0%

---

# Dispatcher configureren om CSRF-aanvallen te voorkomen{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM biedt een raamwerk dat bedoeld is om aanvallen met vervalsingen van verzoeken tussen sites te voorkomen. Als u dit framework op de juiste manier wilt gebruiken, brengt u de volgende wijzigingen aan in de configuratie van Dispatcher:

>[!NOTE]
>
>Ben zeker om de regelaantallen in de hieronder voorbeelden bij te werken die op uw bestaande configuratie worden gebaseerd. Herinner dat de verzenders de laatste passende regel gebruiken om toe te staan of te ontkennen, zodat plaats de regels dichtbij de bodem van uw bestaande lijst.

1. In de `/clientheaders` deel van uw `author-farm.any` en `publish-farm.any`, voegt u de volgende vermelding toe onder aan de lijst:\
   `CSRF-Token`
1. In de /filters sectie van uw `author-farm.any` en `publish-farm.any` of `publish-filters.any` bestand, voeg de volgende regel toe om aanvragen voor `/libs/granite/csrf/token.json` via de Dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. Onder de `/cache /rules` deel van uw `publish-farm.any`voegt u een regel toe om te voorkomen dat de Dispatcher het cachegeheugen van de `token.json` bestand. Auteurs omzeilen caching doorgaans, zodat het niet nodig is om de regel aan uw `author-farm.any`.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Om te bevestigen dat de configuratie werkt, bekijk de dispatcher.log in DEBUG wijze om te bevestigen dat het token.json- dossier niet in de cache wordt opgeslagen en niet door filters wordt geblokkeerd. U zou berichten moeten zien gelijkend op:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

U kunt ook controleren of aanvragen succesvol zijn in uw Apache `access_log`. Verzoeken om &quot;/libs/granite/csrf/token.json zouden een HTTP 200 statuscode moeten terugkeren.
