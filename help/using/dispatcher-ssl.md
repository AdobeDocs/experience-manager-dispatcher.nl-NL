---
title: SSL gebruiken met Dispatcher
description: Leer hoe u Dispatcher configureert voor communicatie met AEM via SSL-verbindingen.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
index: y
internal: n
snippet: y
exl-id: ec378409-ddb7-4917-981d-dbf2198aca98
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '1302'
ht-degree: 0%

---

# SSL gebruiken met Dispatcher {#using-ssl-with-dispatcher}

Gebruik SSL-verbindingen tussen Dispatcher en de renderingcomputer:

* [Eenvoudige SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [Wederzijdse SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>Bewerkingen in verband met de SSL-certificaten zijn gebonden aan producten van derden. Zij vallen niet onder het Adobe Platinum Maintenance and Support-contract.

## SSL gebruiken wanneer Dispatcher verbinding maakt met AEM {#use-ssl-when-dispatcher-connects-to-aem}

Configureer Dispatcher voor communicatie met de AEM- of CQ-renderinstantie met behulp van SSL-verbindingen.

Voordat u Dispatcher configureert, configureert u AEM of CQ om SSL te gebruiken:

* AEM 6.2 [HTTP via SSL inschakelen](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* AEM 6.1: [HTTP via SSL inschakelen](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* Oudere AEM versies: zie [deze pagina](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions).

### SSL-gerelateerde aanvraagheaders {#ssl-related-request-headers}

Wanneer Dispatcher een HTTPS-verzoek ontvangt, neemt Dispatcher de volgende koppen op in het volgende verzoek dat het naar AEM of CQ verzendt:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

Een verzoek via Apache-2.4 met `mod_ssl` omvat kopballen die aan het volgende voorbeeld gelijkaardig zijn:

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### Dispatcher configureren voor gebruik van SSL {#configuring-dispatcher-to-use-ssl}

Als u Dispatcher wilt configureren voor verbinding met AEM of CQ via SSL, moet u [dispatcher.any](dispatcher-configuration.md) bestand vereist de volgende eigenschappen:

* Een virtuele host die HTTPS-aanvragen afhandelt.
* De `renders` in de virtuele host bevat een item dat de hostnaam en -poort van de CQ- of AEM-instantie identificeert die HTTPS gebruikt.
* De `renders` item bevat een eigenschap met de naam `secure` van waarde `1`.

Opmerking: maak indien nodig een andere virtuele host voor de afhandeling van HTTP-aanvragen.

Het volgende voorbeeld `dispatcher.any` Het bestand bevat de eigenschapwaarden voor het verbinden met HTTPS met een CQ-instantie die op de host wordt uitgevoerd `localhost` en poort `8443`:

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requests
         "https://*"
      }
      /renders
      {
      /0001
         {
            # hostname or IP of the render
            /hostname "localhost"
            # port of the render
            /port "8443"
            # connect via HTTPS
            /secure "1"
         }
      }
     # the rest of the properties are omitted
   }

   /non-secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTP requests
         "http://*"
      }
      /renders
      {
         /0001
      {
         # hostname or IP of the render
         /hostname "localhost"
         # port of the render
         /port "4503"
      }
   }
    # the rest of the properties are omitted
}
```

## Wederzijdse SSL configureren tussen Dispatcher en AEM {#configuring-mutual-ssl-between-dispatcher-and-aem}

Als u Wederzijdse SSL wilt gebruiken, configureert u de verbindingen tussen Dispatcher en de rendercomputer (doorgaans een AEM- of CQ-publicatie-instantie):

* Dispatcher maakt verbinding met de renderinstantie via SSL.
* De renderinstantie verifieert de geldigheid van het certificaat van Dispatcher.
* Dispatcher controleert of de CA van het certificaat van de renderinstantie vertrouwd is.
* (Optioneel) Dispatcher controleert of het certificaat van de renderinstantie overeenkomt met het serveradres van de renderinstantie.

Voor het configureren van wederzijdse SSL hebt u certificaten nodig die zijn ondertekend door een vertrouwde certificeringsinstantie (CA). Zelfondertekende certificaten zijn niet geschikt. U kunt of als CA dienst doen of de diensten van derde CA gebruiken om uw certificaten te ondertekenen. Voor het configureren van wederzijdse SSL hebt u de volgende items nodig:

* Ondertekende certificaten voor de renderinstantie en de Dispatcher
* Het certificaat van CA (als u als CA dienst doet)
* OpenSSL-bibliotheken voor het genereren van CA-, certificaten- en certificaataanvragen.

Voer de volgende stappen uit om wederzijdse SSL te configureren:

1. [Installeren](dispatcher-install.md) de nieuwste versie van Dispatcher voor uw platform. Gebruik een binaire Dispatcher die SSL ondersteunt (SSL staat in de bestandsnaam), zoals `dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar`).
1. [Certificaat met CA-handtekening maken of verkrijgen](dispatcher-ssl.md#main-pars-title-3) voor Dispatcher en de render-instantie.
1. [Een sleutelarchief met een rendercertificaat maken](dispatcher-ssl.md#main-pars-title-6) en configureer de dienst van HTTP van render.
1. [De webservermodule Dispatcher configureren](dispatcher-ssl.md#main-pars-title-4) voor wederzijdse SSL.

### Certificaten met CA-handtekening maken of verkrijgen {#creating-or-obtaining-ca-signed-certificates}

Creeer of verkrijg de CA-ondertekende certificaten die de het publiceren instantie en de Verzender voor authentiek verklaren.

#### Uw CA maken {#creating-your-ca}

Als u als CA handelt, gebruik [OpenSSL](https://www.openssl.org/) om de certificeringsinstantie te maken die de server- en clientcertificaten ondertekent. (U moet de OpenSSL-bibliotheken hebben geïnstalleerd.) Als u een derde CA gebruikt, voer deze procedure niet uit.

1. Open een terminal en wijzig de huidige map in de map die de `CA.sh` bestand, zoals `/usr/local/ssl/misc`.
1. Om CA tot stand te brengen, ga het volgende bevel in en verstrek dan waarden wanneer ertoe aangezet:

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >Verschillende eigenschappen in het deelvenster `openssl.cnf` besturen de werking van het script CA.sh. Bewerk dit bestand naar wens voordat u uw CA maakt.

#### Certificaten maken {#creating-the-certificates}

Gebruik OpenSSL om de certificaataanvragen te maken die u naar de derde CA wilt verzenden of die u met uw CA wilt ondertekenen.

Wanneer u een certificaat maakt, gebruikt OpenSSL de eigenschap Common Name om de certificaathouder te identificeren. Voor het certificaat van de renderinstantie, gebruik de de gastheernaam van de instantiecomputer als Gemeenschappelijke Naam als u Verzender vormt om het certificaat goed te keuren. Doe dit slechts als het hostname van de het Publiceren instantie aanpast. Zie de [DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11) eigenschap.

1. Open een terminal en wijzig de huidige map in de map met het CH.sh-bestand van uw OpenSSL-bibliotheken.
1. Voer de volgende opdracht in en geef waarden op wanneer u hierom wordt gevraagd. Indien nodig, gebruik de gastheernaam van de het publiceren instantie als Gemeenschappelijke Naam. De gastheernaam is DNS-oplosbare naam voor het IP adres van teruggeven:

   ```shell
   ./CA.sh -newreq
   ```

   Als u een derde CA gebruikt, verzend het newreq.pem- dossier naar CA om te ondertekenen. Als u als CA handelt, ga verder met stap 3.

1. Als u het certificaat wilt ondertekenen met het certificaat van uw CA, voert u de volgende opdracht in:

   ```shell
   ./CA.sh -sign
   ```

   Twee bestanden met naam `newcert.pem` en `newkey.pem` worden gemaakt in de map die uw CA-beheerbestanden bevat. Deze twee bestanden zijn respectievelijk het openbare certificaat en de persoonlijke sleutel voor de renderingcomputer.

1. Naam wijzigen `newcert.pem` tot `rendercert.pem`en hernoemen `newkey.pem` tot `renderkey.pem`.
1. Herhaal stap 2 en 3 om een certificaat en een openbare sleutel voor de module Dispatcher te maken. Zorg ervoor dat u een algemene naam gebruikt die specifiek is voor de instantie Dispatcher.
1. Naam wijzigen `newcert.pem` tot `dispcert.pem`en hernoemen `newkey.pem` tot `dispkey.pem`.

### SSL configureren op de rendercomputer {#configuring-ssl-on-the-render-computer}

SSL configureren op de renderinstantie met behulp van `rendercert.pem` en `renderkey.pem` bestanden.

#### Het rendercertificaat omzetten in de JKS-indeling (Java™ KeyStore) {#converting-the-render-certificate-to-jks-format}

Gebruik de volgende opdracht om het rendercertificaat, een PEM-bestand, om te zetten in een PKCS#12-bestand. Neem ook het certificaat op van de CA die het rendercertificaat heeft ondertekend:

1. Wijzig in een terminalvenster de huidige map in de locatie van het rendercertificaat en de persoonlijke sleutel.
1. Als u het rendercertificaat (een PEM-bestand) wilt converteren naar een PKCS#12-bestand, voert u de volgende opdracht in. Neem ook het certificaat op van de CA die het rendercertificaat heeft ondertekend:

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. Als u het PKCS#12-bestand wilt converteren naar de JKS-indeling (Java™ KeyStore), voert u de volgende opdracht in:

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Het Java™-sleutelarchief wordt gemaakt met een standaardalias. Wijzig desgewenst de alias:

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### Het Cert van CA toevoegen aan de Render Truststore {#adding-the-ca-cert-to-the-render-s-truststore}

Als u als CA dienst doet, voer uw certificaat van CA in keystore in. Configureer vervolgens de JVM die de renderinstantie uitvoert, om het sleutelarchief te vertrouwen.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. Gebruik een teksteditor om het bestand cacert.pem te openen en alle tekst te verwijderen die aan de volgende regel voorafgaat:

   `-----BEGIN CERTIFICATE-----`

1. Gebruik de volgende opdracht om het certificaat te importeren in een sleutelarchief:

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. Gebruik de volgende systeemeigenschap om de JVM te configureren die uw renderinstantie uitvoert om het sleutelarchief te vertrouwen:

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   Als u bijvoorbeeld het script crx-quickstart/bin/quickstart gebruikt om uw publicatieinstantie te starten, kunt u de eigenschap CQ_JVM_OPTS wijzigen:

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### De renderinstantie configureren {#configuring-the-render-instance}

Als u de HTTP-service van de renderinstantie wilt configureren voor het gebruik van SSL, gebruikt u het rendercertificaat met de instructies in het dialoogvenster *`Enable SSL on the Publish Instance`* sectie:

* AEM 6.2 [HTTP via SSL inschakelen](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* AEM 6.1: [HTTP via SSL inschakelen](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* Oudere AEM versies: zie [deze pagina.](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)

### SSL configureren voor de Dispatcher Module {#configuring-ssl-for-the-dispatcher-module}

Als u Dispatcher wilt configureren voor gebruik van wederzijdse SSL, bereidt u het certificaat Dispatcher voor en configureert u vervolgens de module Webserver.

### Het creëren van een Verenigd Certificaat van de Verzender {#creating-a-unified-dispatcher-certificate}

Combineer het Dispatcher-certificaat en de niet-gecodeerde persoonlijke sleutel tot één PEM-bestand. Een teksteditor of de `cat` gebruiken om een bestand te maken dat lijkt op het volgende voorbeeld:

1. Open een terminal en wijzig de huidige map in de locatie van het bestand dispkey.pem.
1. Voer de volgende opdracht in om de persoonlijke sleutel te decoderen:

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. Een teksteditor of de `cat` gebruiken om de niet-gecodeerde persoonlijke sleutel en het certificaat te combineren in één bestand dat vergelijkbaar is met het volgende voorbeeld:

   ```xml
   -----BEGIN RSA PRIVATE KEY-----
   MIICxjBABgkqhkiG9w0B...
   ...M2HWhDn5ywJsX
   -----END RSA PRIVATE KEY-----
   -----BEGIN CERTIFICATE-----
   MIIC3TCCAk...
   ...roZAs=
   -----END CERTIFICATE-----
   ```

### Het certificaat opgeven dat voor Dispatcher moet worden gebruikt {#specifying-the-certificate-to-use-for-dispatcher}

Voeg de volgende eigenschappen toe aan de [Configuratie van de module Dispatcher](dispatcher-install.md#main-pars-55-35-1022) (in `httpd.conf`):

* `DispatcherCertificateFile`: Het pad naar het uniforme certificaatbestand van Dispatcher dat het openbare certificaat en de niet-gecodeerde persoonlijke sleutel bevat. Dit bestand wordt gebruikt wanneer de SSL-server om het Dispatcher-clientcertificaat vraagt.
* `DispatcherCACertificateFile`: Het pad naar het CA-certificaatbestand dat wordt gebruikt als de SSL-server een CA presenteert die niet wordt vertrouwd door een basisinstantie.
* `DispatcherCheckPeerCN`: Of moet worden ingeschakeld ( `On`) of uitschakelen ( `Off`) hostnaam controleren op externe servercertificaten.

De volgende code is een voorbeeldconfiguratie:

```xml
<IfModule disp_apache2.c>
  DispatcherConfig conf/dispatcher.any
  DispatcherLog    logs/dispatcher.log
  DispatcherLogLevel 3
  DispatcherNoServerHeader 0
  DispatcherDeclineRoot 0
  DispatcherUseProcessedURL 0
  DispatcherPassError 0
  DispatcherCertificateFile disp_unified.pem
  DispatcherCACertificateFile cacert.pem
  DispatcherCheckPeerCN On
</IfModule>
```
