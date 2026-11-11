![header](../src/ZinBanner.png "template_header")

# RFC26288 - Notificeren en melden met CloudEvents

_versie 0.4 d.d 11-11-2026_

## Samenvatting

### Huidige situatie

De notificaties en meldingen worden op dit moment via GraphQL geregeld zoals beschreven in het [Afsprakenstelsel iWlz > Notificeren en Melden](https://istandaarden.github.io/Afsprakenstelsel-iWlz/current/applicatie/diensten/notificeren-en-melden/).

### Beoogde situatie

Deze RFC beschrijft de voorgestelde overstap van het huidige GraphQL-systeem voor notificaties en meldingen naar een event-driven architectuur (EDA) met behulp van CloudEvents. Deze wijziging is bedoeld om de interoperabiliteit, schaalbaarheid en flexibiliteit van het notificatiesysteem binnen het Netwerkmodel te verbeteren.

### Status RFC

Volg deze link naar issue [#288](https://github.com/iStandaarden/iWlz_RequestForChange/issues/288) om de actuele status van deze RFC te bekijken.

## Inhoudsopgave

- [RFC26288 - Notificeren en melden met CloudEvents](#rfc26288---notificeren-en-melden-met-cloudevents)
  - [Samenvatting](#samenvatting)
    - [Huidige situatie](#huidige-situatie)
    - [Beoogde situatie](#beoogde-situatie)
    - [Status RFC](#status-rfc)
  - [Inhoudsopgave](#inhoudsopgave)
- [1. Inleiding](#1-inleiding)
    - [1.1 Relatie andere RFC](#11-relatie-andere-rfc)
- [2. Huidige situatie GraphQL](#2-huidige-situatie-graphql)
    - [Huidige Situatie](#huidige-situatie-1)
      - [Voorbeeld Notificatie GraphQL](#voorbeeld-notificatie-graphql)
- [3. CloudEvents](#3-cloudevents)
  - [3.1 Introductie](#31-introductie)
  - [3.2 NL-GOV Specificatie](#32-nl-gov-specificatie)
    - [3.2.1 Vulling *verplichte* Context Attributes voor iWlz Netwerkmodel](#321-vulling-verplichte-context-attributes-voor-iwlz-netwerkmodel)
      - [3.2.1.1 `id`](#3211-id)
      - [3.2.1.2 `source`](#3212-source)
      - [3.2.1.3 `specversion`](#3213-specversion)
      - [3.2.1.4 `type`](#3214-type)
    - [3.2.2 Vulling *optionele* Context Attributes voor Netwerkmodel](#322-vulling-optionele-context-attributes-voor-netwerkmodel)
      - [3.2.2.1 `datacontenttype`](#3221-datacontenttype)
      - [3.2.2.2 `dataschema`](#3222-dataschema)
      - [3.2.2.3 `time`](#3223-time)
    - [3.2.3 Event Data](#323-event-data)
- [4. Verzending](#4-verzending)
  - [4.1 HTTP-transport](#41-http-transport)
  - [4.2 Content-Type](#42-content-type)
    - [4.2.1 `cloudevents+json`](#421-cloudeventsjson)
    - [4.2.2 `application/cloudevents-batch+json` (toekomstig?)](#422-applicationcloudevents-batchjson-toekomstig)
  - [4.3 Response](#43-response)
    - [4.3.1 Geldige situatie](#431-geldige-situatie)
    - [4.3.2 Ongeldige situatie](#432-ongeldige-situatie)
- [5. Implementatie](#5-implementatie)
- [6. Mapping `eventType` en `type`](#6-mapping-eventtype-en-type)
  - [6.1 Vanuit Bemiddelingsregister:](#61-vanuit-bemiddelingsregister)
  - [6.2 Vanuit Indicatieregister:](#62-vanuit-indicatieregister)
  - [6.3 Vanuit Leveringsregister:](#63-vanuit-leveringsregister)
  - [6.4 iWlz foutmelding](#64-iwlz-foutmelding)
- [7. Referenties](#7-referenties)


# 1. Inleiding

Het doel van een notificatie is het op de hoogte stellen van een deelnemer door een bron over nieuwe (of gewijzigde) informatie die directe of afgeleide betrekking heeft op die deelnemer en daarmee de deelnemer in staat stelt op basis van die notificatie de nieuwe informatie te raadplegen. Een notificatie verloopt altijd van bronhouder naar deelnemer. De reden voor notificatie is altijd de registratie of wijziging van gegevens in een bronregister. Dit is de notificatie-trigger en beschrijft welk CRUD-event in het register leidt tot een notificatie. Deze werkwijze volgt een Event-Driven Architecture (EDA)

Andersom heeft een foutmelding als doel een bronhouder op de hoogte te stellen van een fout-constatering in de data van de bronhouder door een raadpleger. 

Zowel de notificatie als de foutmelding zijn dus berichten die bedoeld zijn om de ontvanger te informeren over een gebeurtenis. Een nieuwe, gewijzigde of verwijderde registratie of een fout constatering. 

Voor het doorgeven van een gebeurtenis (event) bestaat er CloudEvents waarvoor een CloudEvent-NL profiel bestaat dat geïmplementeerd kan worden voor een uniforme berichtgeving in Nederland.


### 1.1 Relatie andere RFC

| RFC | onderwerp | relatie* | toelichting | issue |
| --- | --- | --- | --- | --- |
| [RFC0008](./RFC0008%20-%20Notificaties.md) | Notificaties | gerelateerd | beschrijft de functionele uitwerking van notificaties | #2 |

> [!caution]
> RFC0008 - Notificaties is inmiddels verwerkt in het Afsprakenstelsel iWlz en daarna is de RFC niet meer bijgehouden. Ga voor een actueel inzicht in Notificeren naar [Afsprakenstelsel iWlz > Applicatie > Diensten > Notificeren en Melden](https://istandaarden.github.io/Afsprakenstelsel-iWlz/current/applicatie/diensten/notificeren-en-melden/)



# 2. Huidige situatie GraphQL

### Huidige Situatie

De structuur van de notificatie bevat verschillende gegevens, zoals de tijdstempel, identificatie van de afzender en ontvanger, het type gebeurtenis, en een lijst met onderwerpen die betrekking hebben op de notificatie. Dit stelt de ontvanger in staat om de relevante informatie te raadplegen.

#### Voorbeeld Notificatie GraphQL

**Query:**

```graphql
  $afzenderID: String!
  $afzenderIDType: IDTypeEnum!
  $eventType: String!
  $ontvangerID: String!
  $ontvangerIDType: IDTypeEnum!
  $timestamp: DateTime!
  $subjectList: [SubjectEntity!]!
) {
  zendNotificatie(
    notificatieInput: {
      afzenderID: $afzenderID
      afzenderIDType: $afzenderIDType
      eventType: $eventType
      ontvangerID: $ontvangerID
      ontvangerIDType: $ontvangerIDType
      timestamp: $timestamp
      subjectList: $subjectList
    }
  ) {
    notificatieID
  }
}
```

**Variabelen:**

```json
{
  "afzenderID": "62253778",
  "afzenderIDType": "KVK",
  "eventType": "NIEUWE_INDICATIE_ZORGKANTOOR",
  "ontvangerID": "5151",
  "ontvangerIDType": "UZOVI",
  "timestamp": "2024-07-02T00:00:00Z",
  "subjectList": [
    {
      "recordID": "WlzIndicatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
      "subject": "WlzIndicatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
    }
  ]
}
```

**Succesvol response:**

```http
HTTP/1.1 200
```

```json
{
  "data": {
    "zendNotificatie": {
      "notificatieID": "e2d8c3c2-7453-4948-95c8-de86688461e5"
    }
  }
  ```
 
# 3. CloudEvents

## 3.1 Introductie

CloudEvents is een specificatie die is ontworpen om de interoperabiliteit van event-gedreven systemen te verbeteren. Het biedt een gestandaardiseerde uniforme structuur om gebeurtenissen (events) te beschrijven die plaatsvinden binnen een systeem of tussen verschillende systemen. Dit is vooral nuttig in cloud-native omgevingen, waar verschillende applicaties en diensten met elkaar moeten communiceren.

**Abonnementendienst en notificatie catalogus**  
In het verlengde daarvan biedt CloudEvents met die uniforme structuur de elementen om een gestandaardiseerde abonnementservice en notificatiecatalogus te specificeren — twee onderdelen die nu nog ontbreken in het netwerkmodel. De context-attributen uit die structuur vormen de basis voor het zoeken en filteren van notificaties enerzijds, en voor het registreren van abonnementen (subscriptions) anderzijds.

Meer over - [CloudEvents.io](https://cloudevents.io/)

## 3.2 NL-GOV Specificatie

Voor gebruik binnen de Nederlandse overheden is er een [NL GOV profile for CloudEvents](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/) opgesteld om de gebeurtenissen en meldingen daarvan aan partijen in te richten en te standaardiseren. 

Deze Request for Comment gaat uit van dat profiel en geeft aan hoe de verschillende attributen uit het CloudEvent schema toepast worden in het Netwerkmodel. Het CloudEvent schema kent zowel **verplichte** als **optionele** attributen die deels conditioneel zijn. 

### 3.2.1 Vulling *verplichte* Context Attributes voor iWlz Netwerkmodel

De CloudEvent specificatie kent de volgende [verplichte attributen](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#required-attributes)

- `id`
- `source`
- `specversion`
- `type`

Hieronder de specificatie voor gebruik binnen het Netwerkmodel.

#### 3.2.1.1 `id` 

**id:** De unieke identificatie van de specifieke gebeurtenis (event).

In het **Netwerkmodel** zal CloudEvents ingezet worden om de diensten Notificeren en Melden te standaardiseren. Een notificatie of melding bevat een `recordID` en `subject` die gelijk blijven voor elke notificatie soort of melding op dat record. Geen van beide id's komen daarmee in aanmerking om gebruikt te worden voor vulling in `id` omdat het CloudEvent met geen van de id's (in combinatie met de source) uniek is te identificeren, wat het doel is van `id`.

Daarom geldt voor het Netwerkmodel het volgende: 
  - Het CloudEvent `id` **MOET** daarom gevuld worden met een uniek UUID. In de praktijk zal dit een UUID versie 4 zijn. 
  - Daarnaast **MOET** er in het CloudEvent een verwijzing opgenomen worden naar het dataschema waarin duidelijk is gemaakt dat het `id` gevuld wordt met een random waarde die niet gebruikt kan worden om het desbetreffende record op te vragen. Hiervoor zal het attribuut `dataschema` worden toegevoegd met de verwijzing naar de uitleg van het onderdeel in het attribuut `data`.

Voorbeeld:
```json
"id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
"dataschema" : "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json"
```  

Zie [NL GOV profile for CloudEvents - §3.3.1 id](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#id)

#### 3.2.1.2 `source`

**source:** De bron van de gebeurtenis, in een uniek URN-formaat dat de afzender identificeert.

In het **Netwerkmodel** zijn momenteel de volgende organisatie-identifiers afgesproken: 

| Code  | Omschrijving                         | Referentie     | Toepassing                            |
|-------|--------------------------------------|----------------|---------------------------------------|
| AGB   | AGB-code                             | AGB-register   | identificatie Zorgaanbieder           |
| KVK   | Kamer van Koophandel                 | KVK-register   | identificatie Ondernemer ( voor CIZ*) |
| OIN   | Organisatie Identificatienummer      | OIN-register   | identificatie CIZ (toekomstig*)       |
| UZOVI | Unieke ZorgVerzekeraarsIdentificatie | UZOVI-register | identificatie Zorgkantoren            |

(zie: [Afsprakenstelsel iWlz > Notificeren en Melden - §2.2.1 Code afzenderIDtype of ontvangerIDType](https://istandaarden.github.io/Afsprakenstelsel-iWlz/current/applicatie/diensten/notificeren-en-melden/#221-code-afzenderidtype-of-ontvangeridtype))

Daarom geldt voor Netwerkmodel het volgende voor de opbouw van de URN voor `source`: `urn:1:2:3:4:5`
1. Vaste waarde: `nld` (Verplicht volgens NL GOV profile for CloudEvents) 
2. Variabele waarde: de code van organisatie-identifier zoals hierboven is aangegeven
3. Variabele waarde: de unieke ID van de organisatie die de CloudEvent verstuurd
4. Vaste waarde: `events` 
5. Variabele waarde: Gekoppeld aan afzender. 
   1. Voor CIZ: `indicatie`
   2. Voor Zorgkantoor: `bemiddeling`
   3. Voor Aanbieder: `levering`  
  *N.b. het is niet toegestaan hier te verwijzen naar een externe data bron. Vandaar geen register.*

Voorbeeld voor cloudevent vanuit een (zorg-)aanbieder:
```json
"source" : "urn:nld:agb:12345678:events:levering"
```

Zie [NL GOV profile for CloudEvents - §3.3.2 source](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#source-0)

#### 3.2.1.3 `specversion` 

**specversion:** De versie van de CloudEvents-specificatie die wordt gebruikt.

Vaste *verplichte* waarde: `1.0`

```json
"specversion": "1.0"
```

Zie [NL GOV profile for CloudEvents - §3.3.3 specversion](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#specversion)

#### 3.2.1.4 `type`

**type:** Referentie naar de soort gebeurtenis (event).  

In het **netwerkmodel** is `type` afgeleid van het `eventType` van een notificatie of melding. Zo zijn de huidig gedefinieerde notificatie eventtypen direct te gebruiken voor het CloudEvent type. 

Voorwaarden van het CloudEvent `type`: 
1. moet opgebouwd worden volgens het [Reverse domain name notation](https://en.wikipedia.org/wiki/Reverse_domain_name_notation) principe. Voor het netwerkmodel zal dit `"nl.istandaarden"` zijn. 
2. bevat bij voorkeur een verwijzing naar de data source. Voor het netwerkmodel: 
   - `idr`: Indicatieregister, 
   - `bmr`: Bemiddelingsregister, 
   - `lvr`: Leveringsregsiter
3. bevat een unieke verwijzing naar het soort event. Hier zal een extra namespace aanduiding aan worden toegevoegd in de vorm van `eventtypes`. 

> [!CAUTION]
> 
> Formeel hoort de ontvanger niet in het CloudEvent-type, dit is in de specificatie van de huidige notificatie eventType wel het geval.


Voorbeeld op basis van de twee notificaties: 

```json
"eventType" : "NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR",
"eventType" : "NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER"
```

De CloudEvent `type` waarde in het Netwerkmodel zal dan als volgt worden opgebouwd:
```json
"type": "nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie"
```

De volledige mapping van de huidige notificatie-`eventType` en het CloudEvents-`type` staat verderop. CloudEvents-type zal als attribuut worden toegevoegd aan de notificatiebeschrijving. 

Zie [NL GOV profile for CloudEvents - §3.3.4 type](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#type)


### 3.2.2 Vulling *optionele* Context Attributes voor Netwerkmodel

Naast de verplichte CloudEvents Context Attributes zijn er ook een aantal optionele attributen die **WEL** verplicht zijn voor het Netwerkmodel. Reden is dat de huidige notificatie en melding structuur ondergebracht worden in een CloudEvent. Hiervoor is het nodig gebruik te maken van het attribuut `data`. Met het gebruik van `data` zijn een aantal optionele context attributen verplicht. 

Het gaat om de volgende attributen:
- `datacontenttype`
- `dataschema`
- `time`

Hieronder volgt de specificatie van de benodigde Context Attributes die daarmee **VERPLICHT** worden in het Netwerkmodel.

#### 3.2.2.1 `datacontenttype`

**datacontenttype:** Het type van de gegevens die in de data-sectie zijn opgenomen. 

In het **Neterkmodel** zijn de notificatie en de melding beide opgesteld als JSON. De waarde voor `datacontenttype` is voor het Netwerkmodel daarom:

```json
"datacontenttype": "application/json"
```

Zie [NL GOV profile for CloudEvents - §3.4.1 datacontenttype](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#datacontenttype)


#### 3.2.2.2 `dataschema`

**dataschema:** Identificeert (de locatie van) het schema waaraan de gegevens in `data` moet voldoen.

Het is verplicht aanwezig omdat hier een toelichting opgenomen moet worden waarin uitgelegd is dat `id` geen documentwaarde heeft. Er is geen document met de identificatie-waarde uit `id`. (zie ook [`id`](#3211-id))

In het **Netwwerkmodel** staat de eerste versie van het schema (voorlopig gepland) op: https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json

```json
"dataschema" : "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json"
```

**NU** staat de eerste versie van het schema [hier](./cloudevents.schema.json)

Zie [NL GOV profile for CloudEvents - §3.4.2 dataschema](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#dataschema)

#### 3.2.2.3 `time`

- **time:** De tijd waarop de gebeurtenis heeft plaatsgevonden.

In het **Netwerkmodel** toegevoegd om de volgorde van de ontvangen CloudEvents te bepalen. In de huidige specificatie van de notificatie en de melding is `timestamp` gedefinieerd als respectievelijk "Tijdstip waarop de notificatie is aangemaakt", Tijdstip waarop de melding is aangemaakt". Dit kan anders zijn dan het moment waarop de gebeurtenis (bijvoorbeeld de registratie nieuwe indicatie). Daarnaast is in dit RFC ook het voorstel opgenomen om notificaties van hetzelfde type te bundelen in één CloudEvent. 

De waarde van `time` is daarom gebaseerd op **het moment waarop de CloudEvent is aangemaakt**.

Voorbeeld:
```json
"time": "2024-07-02T00:00:00Z"
```
N.b. miliseconden zijn toegestaan mits genoteerd als decimal fraction (.123)

Zie [NL GOV profile for CloudEvents - §3.4.4 time](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#time)

### 3.2.3 Event Data

- **data:** De payload van de gebeurtenis met de relevante metadata.

In het **Netwerkmodel** bestaat de Notificatie (of Melding) JSON uit de volgende attributen: 

bijvoorbeeld: 
```json
{
  "afzenderID": "5050",
  "afzenderIDType": "UZOVI",
  "eventType": "NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER",
  "ontvangerID": "12345678",
  "ontvangerIDType": "AGBCODE",
  "timestamp": "2024-07-02T00:00:00Z",
  "subjectList": [
    {
      "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
      "recordID": "Bemiddelingspecificatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
    }
  ]
}
```

Dit hele JSON-object *kan* in `data` worden geplaatst, maar dan zitten er een aantal redundante gegevens in het object. 

**JSON-object minmaliseren**

Een aantal van de attributen in het notificatie/melding JSON-object zitten ook al in de CloudEvent-header en kunnen daarom worden vervangen door een CloudEvent-attribuut: 

| Notifictie of Melding attribuut | Vervangen door CloudEvent met |
| :------------------------------ | :---------------------------- |
| `afzenderID`                    | [`source`](#3212-source)      |
| `afzenderIDType`                | [`source`](#3212-source)      |
| `eventType`                     | [`type`](#3214-type)          |
| `ontvangerID`                   | handhaven (is dit nodig?)     |
| `ontvangerIDType`               | handhaven (is dit nodig?)     |
| `timestamp`                     | [`time`](#3223-time)          |
| `subjectList`                   | handhaven                     |
| `subject`                       | handhaven                     |
| `recordID`                      | handhaven                     |

Voorbeeld JSON-data notificatie in CloudEvent:
```json
{
  "ontvangerID": "12345678",
  "ontvangerIDType": "AGBCODE",
  "subjectList": [
    {
      "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
      "recordID": "Bemiddelingspecificatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
    }
  ]
}
```

Voorbeeld CloudEvent met notificatie:

```json
{
  "id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
  "source": "urn:nld:uzovi:5050:events:bemiddeling",
  "specversion": "1.0",
  "type": "nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie",
  "datacontenttype": "application/json",
  "dataschema": "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json",
  "time": "2024-07-02T00:00:00Z",
  "data": {
    "ontvangerID": "12345678",
    "ontvangerIDType": "AGB",
    "subjectList": [
      {
        "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
        "recordID": "Bemiddelingspecificatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
      }
    ]
  }
}
```

Zie [NL GOV profile for CloudEvents - §4 Event Data](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/#event-data)

# 4. Verzending

In het **Netwerkmodel** volgen we al GraphQL over HTTP. De huidige notificaties en meldingen zijn op basis van GraphQL en verlopen dus over HTTP. De laagdrempeligste keuze voor de inzet van CloudEvents is daarom transport over **HTTP**.  

Daarnaast is HTTP Protocol Binding ook de aanbevolen methode voor de NL-GOV profile CloudEvents.

## 4.1 HTTP-transport

De CloudEvents worden verzonden in **Structured Content Mode**. Hierbij zitten de metadata (de CloudEvents-attributen) en de data (de notificatie of melding) samen in de HTTP-body als één JSON-object. 

Zie [http protocol binding](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/http-protocol-binding.md#32-structured-content-mode)

## 4.2 Content-Type

Het `Content-type` is bij Structured Content Mode:  
  - `application/cloudevents+json` 
  - `application/cloudevents-batch+json`

### 4.2.1 `cloudevents+json`

Bij het gebruik van `Content-Type`: `application/cloudevents+json` bevat het CloudEvent één notificatie of melding gebeurtenis (van hetzelfde type) voor één ontvanger. De ontvanger is bijvoorbeeld één zorgkantoor of één (zorg)aanbieder. Dit sluit aan bij de huidige werking van notificeren en melden in het netwerkmodel. Een CloudEvent gaat naar één ontvanger (resource-endpoint). De inhoud van het data attribuut hoort alleen bij die ontvanger. 

Bijvoorbeeld Zorgkantoor 5050 stuurt een notificatie aan Zorgaanbieder AGB01 over 2 nieuwe bemiddeingspecificaties voor die zorgaanbieder (niet-normatief): 

```http
POST /webhook HTTP/1.1
Host: resource.end.point.zorgaanbiederAGB01
Content-Type: application/cloudevents+json

{
  "id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
  "source": "urn:nld:uzovi:5050:events:bemiddeling",
  "specversion": "1.0",
  "type": "nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie",
  "datacontenttype": "application/json",
  "dataschema": "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json",
  "time": "2024-07-02T00:00:00Z",
  "data": {
    "ontvangerID": "12345678",
    "ontvangerIDType": "AGB",
    "subjectList": [
      {
        "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
        "recordID": "Bemiddelingspecificatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
      }
    ]
  }
}

```

### 4.2.2 `application/cloudevents-batch+json` (toekomstig?)

> [!IMPORTANT]
> In de eerste release van de CloudEvent zal batch **NIET** ondersteund worden. Hier zijn aanvullende afspraken voor nodig.
>
> Voor de volledigheid is dit wel toegevoegd aan deze RFC met een mogelijke toepassing.

Een voordeel van Structured Content Mode is dat dit het mogelijk maakt om meerdere notificaties (CloudEvents) aan dezelfde ontvanger (endpoint) te bundelen in één HTTP actie. Dit kan een efficiëntie voordeel geven.

Bijvoorbeeld voor een zorgaanbieder registreert het zorgkantoor twee nieuwe bemiddelingspecificaties en wijzigt een bemiddelingspecificatie (onder een andere Bemiddeling). 

Voorbeeld met batch CloudEvent: 
```json
POST /webhook HTTP/1.1
Host: resource.end.point.zorgaanbieders01en02
Content-Type: application/cloudevents-batch+json

[
  {
    "id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
    "source": "urn:nld:uzovi:5050:events:bemiddeling",
    "specversion": "1.0",
    "type": "nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie",
    "datacontenttype": "application/json",
    "dataschema": "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json",
    "time": "2024-07-02T00:00:00Z",
    "data": {
      "ontvangerID": "12345678",
      "ontvangerIDType": "AGB",
      "subjectList": [
        {
          "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
          "recordID": "Bemiddelingspecificatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
        },
        {
          "subject": "Bemiddeling/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
          "recordID": "Bemiddelingspecificatie/20a2e980-03d4-4225-bfe2-ea301b7a240a"
        }
      ]
    }
  },
  {
    "id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
    "source": "urn:nld:uzovi:5050:events:bemiddeling",
    "specversion": "1.0",
    "type": "nl.istandaarden.eventtypes.bmr.gewijzigde-bemiddelingspecificatie",
    "datacontenttype": "application/json",
    "dataschema": "https://github.com/iStandaarden/generiek/blob/main/cloudevents/v1.json",
    "time": "2024-07-02T00:00:01Z",
    "data": {
      "ontvangerID": "12345678",
      "ontvangerIDType": "AGB",
      "subjectList": [
        {
          "subject": "Bemiddeling/df24f57e-143f-4e11-ae37-c14ee42ab8bc",
          "recordID": "Bemiddelingspecificatie/6cf0553d-4827-4ed2-86f0-4938d29a1465"
        }
      ]
    }
  }
]

``` 

De zorgaanbieder ontvangt hiermee twee notificaties.


## 4.3 Response

De CloudEvents HTTP-binding specificatie schrijft geen specifieke response methode voor en wijst daarvoor naar de gebruikelijke HTTP-conventies. 

> [!WARNING]
> De hier beschreven response geldt voor de resource-server. Het verkeer verloopt altijd via een PEP. De PEP die controleert op de autorisatie kan ook een afzonderlijke HTTP response genereren. Bijvoorbeeld als de verzender niet gerechtigd is het type notificatie te versturen (Bijvoorbeeld het zorgkantoor verstuurd een notificatie met betrekking tot een nieuwe indicatie). De mogelijk HTTP response vanuit de PEP op een CloudEvent zal in een andere RFC met betrekking tot autorisatie en de introductie van AuthZEN worden beschreven.
> 


### 4.3.1 Geldige situatie

Net als bij de GraphQL-mutation levert de succesvolle **AFLEVERING** van een CloudEvent bij het resource-endpoint een geldige situatie. 

De response bij een succesvolle aflevering op het endpoint van de ontvanger is:

```http
HTTP/1.1 200 OK
Content-Length: 0
```
of
```http
HTTP/1.1 204 No Content
```

Daarmee is de ontvangst van de notificatie BEVESTIGD.

**Huidige implementatie**

De huidige implementatie van notificeren en melden verloopt via GraphQL over HTTP. De response van de aflevering van een notificatie volgt daarmee ook de bijbehorende HTTP-status codes. 

Omdat de  huidige notificatie met een GraphQL mutation is vormgegeven stuurt de ontvangende GraphQL-server naast de HTTP-status code een extra response body in de vorm van een notificatieID. 

Voorbeeld succesvol response op GraphQL notificatie:

```json
"data": {
  "zendNotificatie": {
    "notificatieID": "e2d8c3c2-7453-4948-95c8-de86688461e5"
  }
}
```

In het geval van de op CloudEvents gebaseerde notificatie zit de `notificatieID` al in de POST body als [`id`](#3211-id). De bepaling van de identificatie verschuift daarmee naar de verzender in plaats van de ontvanger!




### 4.3.2 Ongeldige situatie

In het geval dat het CloudEvent niet succesvol is ontvangen of kan worden geinterpreteerd, stuurt de Resource-server afhankelijk van de een `4xx` of `500` HTTP status-code.

| Situatie                                                                                    | Gebruikelijke code                                |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Body is ongeldige JSON / mist verplichte attributen (`id`, `source`, `type`, `specversion`) | `400 Bad Request`                                 |
| Content-type wordt niet geaccepteerd                                                        | `415 Unsupported Media Type`                      |
| Authenticatie ontbreekt/incorrect                                                           | `401 Unauthorized / 403 Forbidden`                |
| Endpoint bestaat niet                                                                       | `404 Not Found`                                   |
| Ontvanger tijdelijk overbelast                                                              | `429 Too Many Requests / 503 Service Unavailable` |
| Interne verwerkingsfout                                                                     | `500 Internal Server Error`                       |


# 5. Implementatie

**Veranderingen bij de overstap naar CloudEvents**

Bij de overstap van het huidige GraphQL-systeem naar CloudEvents voor het verzenden van notificaties zullen er verschillende veranderingen plaatsvinden. Onder andere:

1. GraphQL mutation vervangen met CloudEvent (envelop)
   1. eventType in de notificatie vervangen met CloudEvent.type
   2. timestamp in de notificatie vervangen met CloudEvent.time
2. GraphQL server endpoint vervangen door CloudEvent endpoint. Nu worden de notificaties en meldingen (vermoedelijk) aan het register endpoint verzonden. 
3. Als een notificatieID of meldingID nodig blijkt dan moet de CloudEvents body daarop aangepast worden




# 6. Mapping `eventType` en `type`

Notificatie eventtype : CloudEvent type 

## 6.1 Vanuit Bemiddelingsregister:
| Notificatie eventtype                                          | CloudEvent type                                                                   |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `GEWIJZIGDE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER`             | `nl.istandaarden.eventtypes.bmr.gewijzigde-bemiddelingspecificatie`               |
| `GEWIJZIGDE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR`               | `nl.istandaarden.eventtypes.bmr.gewijzigde-bemiddelingspecificatie`               |
| `GEWIJZIGDE_OVERDRACHT_ZORGKANTOOR`                            | `nl.istandaarden.eventtypes.bmr.gewijzigde-overdracht`                            |
| `GEWIJZIGDE_REGIEHOUDER_ZORGAANBIEDER`                         | `nl.istandaarden.eventtypes.bmr.gewijzigde-regiehouder`                           |
| `GEWIJZIGDE_REGIEHOUDER_ZORGKANTOOR`                           | `nl.istandaarden.eventtypes.bmr.gewijzigde-regiehouder`                           |
| `INFORMATIEVE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER`           | `nl.istandaarden.eventtypes.bmr.informatieve-bemiddelingspecificatie`             |
| `INFORMATIEVE_GEWIJZIGDE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR`  | `nl.istandaarden.eventtypes.bmr.informatieve-gewijzigde-bemiddelingspecificatie`  |
| `INFORMATIEVE_NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR`      | `nl.istandaarden.eventtypes.bmr.informatieve-nieuwe-bemiddelingspecificatie`      |
| `INFORMATIEVE_VERWIJDERDE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR` | `nl.istandaarden.eventtypes.bmr.informatieve-verwijderde-bemiddelingspecificatie` |
| `NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER`                 | `nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie`                   |
| `NIEUWE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR`                   | `nl.istandaarden.eventtypes.bmr.nieuwe-bemiddelingspecificatie`                   |
| `NIEUWE_OVERDRACHT_ZORGKANTOOR`                                | `nl.istandaarden.eventtypes.bmr.nieuwe-overdracht`                                |
| `NIEUWE_REGIEHOUDER_ZORGAANBIEDER`                             | `nl.istandaarden.eventtypes.bmr.nieuwe-regiehouder`                               |
| `NIEUWE_REGIEHOUDER_ZORGKANTOOR`                               | `nl.istandaarden.eventtypes.bmr.nieuwe-regiehouder`                               |
| `VERWIJDERDE_BEMIDDELINGSPECIFICATIE_ZORGAANBIEDER`            | `nl.istandaarden.eventtypes.bmr.verwijderde-bemiddelingspecificatie`              |
| `VERWIJDERDE_BEMIDDELINGSPECIFICATIE_ZORGKANTOOR`              | `nl.istandaarden.eventtypes.bmr.verwijderde-bemiddelingspecificatie`              |
| `VERWIJDERDE_OVERDRACHT_ZORGKANTOOR`                           | `nl.istandaarden.eventtypes.bmr.verwijderde-overdracht`                           |
| `VERWIJDERDE_REGIEHOUDER_ZORGAANBIEDER`                        | `nl.istandaarden.eventtypes.bmr.verwijderde-regiehouder`                          |
| `VERWIJDERDE_REGIEHOUDER_ZORGKANTOOR`                          | `nl.istandaarden.eventtypes.bmr.verwijderde-regiehouder`                          |


## 6.2 Vanuit Indicatieregister:
| Notificatie eventtype                         | CloudEvent type                                                  |
| --------------------------------------------- | ---------------------------------------------------------------- |
| `GEWIJZIGDE_INDICATIE_ZORGKANTOOR`            | `nl.istandaarden.eventtypes.idr.gewijzigde-indicatie`            |
| `GEWIJZIGDE_VERVALLENGELDIGHEID_ZORGKANTOOR`  | `nl.istandaarden.eventtypes.idr.gewijzigde-vervallengeldigheid`  |
| `NIEUWE_INDICATIE_ZORGKANTOOR`                | `nl.istandaarden.eventtypes.idr.nieuwe-indicatie`                |
| `NIEUWE_VERVALLENGELDIGHEID_ZORGKANTOOR`      | `nl.istandaarden.eventtypes.idr.nieuwe-vervallengeldigheid`      |
| `VERWIJDERDE_INDICATIE_ZORGKANTOOR`           | `nl.istandaarden.eventtypes.idr.verwijderde-indicatie`           |
| `VERWIJDERDE_VERVALLENGELDIGHEID_ZORGKANTOOR` | `nl.istandaarden.eventtypes.idr.verwijderde-vervallengeldigheid` |

## 6.3 Vanuit Leveringsregister:
| Notificatie eventtype                        | CloudEvents type                                                |
| -------------------------------------------- | --------------------------------------------------------------- |
| `GEWIJZIGD_AFSTEL_ZORGKANTOOR`               | `nl.istandaarden.eventtypes.lvr.gewijzigd-afstel`               |
| `GEWIJZIGDE_BEHANDELINGPERIODE_ZORGKANTOOR`  | `nl.istandaarden.eventtypes.lvr.gewijzigde-behandelingperiode`  |
| `GEWIJZIGDE_LEVERINGPERIODE_ZORGKANTOOR`     | `nl.istandaarden.eventtypes.lvr.gewijzigde-leveringperiode`     |
| `GEWIJZIGDE_UITSTELPERIODE_ZORGKANTOOR`      | `nl.istandaarden.eventtypes.lvr.gewijzigde-uitstelperiode`      |
| `NIEUW_AFSTEL_ZORGKANTOOR`                   | `nl.istandaarden.eventtypes.lvr.nieuw-afstel`                   |
| `NIEUW_VERZOEK_ZORGKANTOOR`                  | `nl.istandaarden.eventtypes.lvr.nieuw-verzoek`                  |
| `NIEUW_VERZOEKAANBIEDER_AANBIEDER`           | `nl.istandaarden.eventtypes.lvr.nieuw-verzoekaanbieder`         |
| `NIEUWE_BEHANDELINGPERIODE_ZORGKANTOOR`      | `nl.istandaarden.eventtypes.lvr.nieuwe-behandelingperiode`      |
| `NIEUWE_LEVERINGPERIODE_ZORGKANTOOR`         | `nl.istandaarden.eventtypes.lvr.nieuwe-leveringperiode`         |
| `NIEUWE_UITSTELPERIODE_ZORGKANTOOR`          | `nl.istandaarden.eventtypes.lvr.nieuwe-uitstelperiode`          |
| `VERWIJDERD_AFSTEL_ZORGKANTOOR`              | `nl.istandaarden.eventtypes.lvr.verwijderd-afstel`              |
| `VERWIJDERDE_BEHANDELINGPERIODE_ZORGKANTOOR` | `nl.istandaarden.eventtypes.lvr.verwijderde-behandelingperiode` |
| `VERWIJDERDE_LEVERINGPERIODE_ZORGKANTOOR`    | `nl.istandaarden.eventtypes.lvr.verwijderde-leveringperiode`    |
| `VERWIJDERDE_UITSTELPERIODE_ZORGKANTOOR`     | `nl.istandaarden.eventtypes.lvr.verwijderde-uitstelperiode`     |

## 6.4 iWlz foutmelding
| Melding eventType | CloudEvents type |
| :--- | :--- |
| `IWLZFOUTMELDING` | `nl.istandaarden.eventtypes.iwlzfoutmelding` |

# 7. Referenties

- [NL-GOV profile for CloudEvents](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/)
- [Guidelines for NL-GOV profile CloudEvents 1.0](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/guidelines/)
- [HTTP Protocol Binding for CloudEvents - Version 1.0.1](https://github.com/cloudevents/spec/blob/v1.0.1/http-protocol-binding.md)
- [CloudEvents.io](https://cloudevents.io/)
- [CloudEvents spec](https://github.com/cloudevents/spec)


