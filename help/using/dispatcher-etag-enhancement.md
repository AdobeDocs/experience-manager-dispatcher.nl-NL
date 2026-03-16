---
title: Dispatcher ETag Enhancement for CDN Revalidation
description: Beschikbaarheid, ondersteuningsstatus en gedrag van INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT in AEM as a Cloud Service.
source-git-commit: ac0fafd060643903735ff565072ef2c5bee970be
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---

# Dispatcher ETag Enhancement for CDN Revalidation

## Overzicht

Met de markering `INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT` kan Dispatcher de aanvraagheader van `If-None-Match` in cache-hits evalueren. Wanneer de binnenkomende `If-None-Match` -waarde overeenkomt met de waarde in de cache `ETag` , kan Dispatcher `304 Not Modified` retourneren in plaats van `200 OK` .

Dit gedrag wordt ontworpen om onnodige nuttige ladingsoverdracht tussen CDN en Dispatcher te verminderen en voorwaardelijk-geheim voorgeheugenefficiency te verhogen.

## Beschikbaarheid

- Dispatcher-versie: `2.0.264`
- AEM SDK build: `aem-sdk-2026.2.24464.20260214T050318Z-260100`

## AEM as a Cloud Service-ondersteuning

In AEM as a Cloud Service wordt deze mogelijkheid ondersteund voor gebruik door de klant.

Klanten kunnen dit inschakelen door de omgevingsvariabele `INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT` in Cloud Manager in te stellen. Adobe kan het ook namens de klant inschakelen wanneer dat nodig is.

Wanneer deze optie is ingeschakeld en wanneer de CDN `If-None-Match` verzendt en de relevante `ETag` aanwezig is in de Dispatcher-cache, worden hogere `304` responspercentages tussen CDN en Dispatcher verwacht. Deze verhoging is het beoogde resultaat.

## Voorbeeld van configuratie (cachegeheugen ETag-header)

Om deze verbetering effectief te maken, moet u ervoor zorgen dat Dispatcher de antwoordheader `ETag` in cache plaatst en dat uw webserver is geconfigureerd om te voorkomen dat ETags op basis van bestandssysteem worden gegenereerd.

Voorbeeld `dispatcher.any` cache-sectie:

```text
/cache {
  /headers {
    "Cache-Control"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "ETag"
  }
}
```

Voorbeeld van een Apache-instructie in de Dispatcher-hostcontext:

```apache
FileETag none
```

Voor basislijn kopbal-caching begeleiding, zie [&#x200B; Caching de reactiekopballen van HTTP &#x200B;](dispatcher-configuration.md#caching-http-response-headers).

## Validatievoorbeeld

Na het toelaten van de milieuvariabele en het opstellen van config verandert:

1. Vraag één keer om de cache op te warmen en de geretourneerde `ETag` vast te leggen.
1. Nogmaals aanvragen met `If-None-Match: <etag-value>` .
1. Bevestig Dispatcher retourneert `304 Not Modified` voor door cache hit validatiestromen.

## Openbare referentie (gerelateerd gedrag)

Raadpleeg de volgende bronnen voor klantgerichte basisrichtlijnen voor het in cache plaatsen van kopteksten en `ETag` afhandelen in Dispatcher:

- [Dispatcher configureren - HTTP-antwoordheaders in cache plaatsen](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration#caching-http-response-headers)

&quot;Deze mogelijkheid is beschikbaar in Dispatcher `2.0.264` (AEM SDK `2026.2.24464` ). Als deze optie is ingeschakeld, kan Dispatcher `If-None-Match` valideren op basis van in de cache opgeslagen `ETag` -waarden en `304 Not Modified` retourneren bij cacheschtochten. In AEM as a Cloud Service wordt dit ondersteund en kan het worden ingeschakeld via de Cloud Manager-omgevingsconfiguratie.&quot;
