![header](../src/ZinBanner.png "template_header")

# RFC26288 - Notificeren met CloudEvents

_versie 0.1 d.d. 09-07-2025_

## Samenvatting

### Huidige situatie

De notificaties en meldingen worden op dit moment via GraphQL geregeld zoals beschreven in RFC0008.

### Beoogde situatie

Deze RFC beschrijft de voorgestelde overstap van het huidige GraphQL-systeem voor notificaties naar een event-driven architectuur met behulp van CloudEvents. Deze wijziging is bedoeld om de interoperabiliteit, schaalbaarheid en flexibiliteit van het notificatiesysteem binnen het iWlz-netwerk te verbeteren.

### Status RFC

Volg deze link naar issue [#288](https://github.com/iStandaarden/iWlz_RequestForChange/issues/288) om de actuele status van deze RFC te bekijken.

## Inhoudsopgave

- [RFC26288 - Notificeren met CloudEvents](#rfc26288---notificeren-met-cloudevents)
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
  - [4. Implementatie](#4-implementatie)
    - [Veranderingen bij de Overstap naar CloudEvents](#veranderingen-bij-de-overstap-naar-cloudevents)
    - [SubjectList](#subjectlist)
    - [4.1 TODO/uitzoekwerk](#41-todouitzoekwerk)
  - [5. Referenties](#5-referenties)



## 1. Inleiding

Het doel van een notificatie is het op de hoogte stellen van een deelnemer door een bron over nieuwe (of gewijzigde) informatie die directe of afgeleide betrekking heeft op die deelnemer en daarmee de deelnemer in staat stelt op basis van die notificatie de nieuwe informatie te raadplegen. Een notificatie verloopt altijd van bronhouder naar deelnemer. De reden voor notificatie is altijd de registratie of wijziging van gegevens in een bronregister. Dit is de notificatie-trigger en beschrijft welk CRUD-event in het register leidt tot een notificatie.


Er bestaat een CloudEvent-NL message format dat geïmplementeerd kan worden voor een uniforme berichtgeving in Nederland.


### 1.1 Relatie andere RFC

| RFC | onderwerp | relatie* | toelichting | issue |
| --- | --- | --- | --- | --- |
| [RFC0008](./RFC0008%20-%20Notificaties.md) | Notificaties | gerelateerd | beschrijft de functionele uitwerking van notificaties | #2 |

> [!caution]
> RFC0008 - Notificaties is inmiddels verwerkt in het Afsprakenstelsel iWlz en daarna is de RFC niet meer bijgehouden. Ga voor een actueel inzicht in Notificeren naar [Afsprakenstelsel iWlz > Applicatie > Diensten > Notificeren en Melden](https://istandaarden.github.io/Afsprakenstelsel-iWlz/current/applicatie/diensten/notificeren-en-melden/)



## 2. Huidige situatie GraphQL

### Huidige Situatie

De structuur van de notificatie bevat verschillende gegevens, zoals de tijdstempel, identificatie van de afzender en ontvanger, het type gebeurtenis, en een lijst met onderwerpen die betrekking hebben op de notificatie. Dit stelt de ontvanger in staat om de relevante informatie te raadplegen.
Voorbeeld Notificatie GraphQL

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
 
  ## 3. CloudEvents

  ### 3.1 Introductie

CloudEvents is een specificatie die is ontworpen om de interoperabiliteit van event-gedreven systemen te verbeteren. Het biedt een gestandaardiseerde manier om gebeurtenissen (events) te beschrijven die plaatsvinden binnen een systeem of tussen verschillende systemen. Dit is vooral nuttig in cloud-native omgevingen, waar verschillende applicaties en diensten met elkaar moeten communiceren.

  **Belangrijke kenmerken van CloudEvents:**

  - CloudEvents definieert een uniforme structuur voor het beschrijven van gebeurtenissen, inclusief belangrijke metadata zoals het type gebeurtenis, de tijdstempel, en de bron van de gebeurtenis.
  - Door een gemeenschappelijke standaard te gebruiken, kunnen verschillende systemen en diensten, ongeacht hun technologie of platform, eenvoudig met elkaar communiceren. Dit bevordert samenwerking en integratie tussen verschillende applicaties.
  - CloudEvents kan over verschillende transportprotocollen worden verzonden, zoals HTTP, AMQP, en Kafka. CloudEvents is ontworpen om protocolonafhankelijk te zijn. Dit betekent dat een event zender niet noodzakelijk hetzelfde protocol hoeft te gebruiken als de ontvangers. Dit biedt flexibiliteit in de keuze van technologieën en infrastructuren. Het is mogelijk om meerdere ontvangers te hebben die verschillende protocollen gebruiken. In veel gevallen kan middleware of een event broker worden gebruikt om events te vertalen tussen verschillende protocollen. Dit betekent dat een event zender een gebeurtenis kan verzenden via één protocol, en de middleware kan deze gebeurtenis omzetten naar een ander protocol voor verschillende ontvangers. Hoe dit geregeld gaat worden moet nog worden afgesproken.
  - CloudEvents ondersteunt asynchrone communicatie, wat betekent dat systemen gebeurtenissen kunnen verzenden en ontvangen zonder dat ze op elkaar hoeven te wachten. Dit verbetert de prestaties en schaalbaarheid van applicaties.
  - Veel moderne cloud-platforms en services ondersteunen CloudEvents, waardoor het eenvoudiger wordt om nieuwe functionaliteiten toe te voegen en bestaande systemen uit te breiden.

  ### 3.2 NL-GOV Specificatie

Hier is een voorbeeld van hoe een notificatie zou worden verzonden met het NL-GOV CloudEvents, inclusief de relevante informatie en structuur.
Een CloudEvent-notificatie kan worden opgesteld in JSON-formaat, wat vervolgens via het HTTP protocol verzonden kan worden. Ook is mogelijk gebruik te maken van webhooks. Hier is een voorbeeld van een CloudEvent-notificatie die een nieuwe indicatie voor een zorgkantoor beschrijft.

```json
{
  "specversion": "1.0",
  "type": "nl.istandaarden.iwlz.indicatie.zorgkantoor-nieuw.v1.0.0 ",
  "source": "urn:kvk:62253778:ciz:cloudevents:indicaties",
  "id": "e2d8c3c2-7453-4948-95c8-de86688461e5",
  "time": "2024-07-02T00:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "afzenderID": "62253778",
    "afzenderIDType": "KVK",
    "eventType": "NIEUWE_INDICATIE_ZORGKANTOOR",
    "ontvangerID": "5151",
    "ontvangerIDType": "UZOVI",
    "subjectList": [
      {
        "recordID": "WlzIndicatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6",
        "subject": "WlzIndicatie/ef88ce35-58fa-4e6d-ac7a-6e298dd211d6"
      }
    ]
  }
}
```

- **specversion:** De versie van de CloudEvents-specificatie die wordt gebruikt.
- **type:** Het type van de gebeurtenis, in dit geval een nieuwe indicatie voor een zorgkantoor.
- **source:** De bron van de gebeurtenis, in een uniek URN-formaat dat de afzender identificeert, zoals een Kamer van Koophandel-nummer (KVK) en een CIZ-service.
- **id:** Een unieke identificatie voor deze specifieke gebeurtenis.
- **time:** De tijd waarop de gebeurtenis heeft plaatsgevonden.
- **datacontenttype:** Het type van de gegevens die in de data-sectie zijn opgenomen, in dit geval JSON.
- **data:** De payload van de gebeurtenis, die de relevante informatie bevat, zoals de afzender en ontvanger, en een lijst met onderwerpen.

Zie NL-GOV voor informatie over alle attributen.

## 4. Implementatie

### Veranderingen bij de Overstap naar CloudEvents

Bij de overstap van het huidige GraphQL-systeem naar CloudEvents voor het verzenden van notificaties zullen er verschillende veranderingen plaatsvinden. Ten eerste zal de structuur van de notificatie veranderen. In plaats van een GraphQL-notificatie te genereren, zal er een CloudEvent-notificatie opgesteld worden in JSON-formaat. Deze notificatie bevat gestandaardiseerde velden zoals specversion, type, source, id, time, en data, wat de interoperabiliteit tussen verschillende systemen vergemakkelijkt.
Daarnaast zal de manier van verzenden van notificaties ook veranderen. De bronhouder kan de CloudEvent-notificatie direct verzenden naar de deelnemer via verschillende transportprotocollen, in plaats van een GraphQL-verzoek. Dit vereenvoudigt de communicatie en biedt de mogelijkheid om verschillende transportprotocollen te gebruiken, wat de flexibiliteit van het systeem vergroot.
De autorisatie- en validatiestappen blijven bestaan, alleen het verzenden van de informatie wordt nu op een andere manier gedaan. CloudEvents biedt een meer gestandaardiseerde aanpak, waardoor het mogelijk is om notificaties efficiënter te verwerken.
Tot slot zal het response-mechanisme veranderen. In plaats van een GraphQL 200-response zal de deelnemer een standaard HTTP-response terugsturen, zoals 200 OK of 400 Bad Request, om de status van de ontvangen CloudEvent-notificatie aan te geven. Deze veranderingen zullen bijdragen aan een meer flexibele, schaalbare en efficiënte manier van notificeren binnen het iWlz-netwerk.

### SubjectList

In de huidige situatie wordt er een veld subjectList meegegeven in de notificatie. Dit betekent dat een notificatie meerdere berichten kan bevatten. Wanneer we met CloudEvents gaan werken zou dit betekenen dat er voor iedere subject in de subjectList een CloudEvent wordt aangemaakt en verzonden.

### 4.1 TODO/uitzoekwerk

- Wie zijn de subscribers van een notificatie-event? <=> Wie moeten er op de hoogte gebracht worden van de verandering? Dit is het zorgkantoor van de persoon waarvoor de verandering is opgetreden.
- Hoe wordt dit gerouteerd naar de desbetreffende subscriber? Het zorgkantoor moet zich abonneren op het notificatiechannel van het desbetreffende register. Gebruik een message broker zoals Kafka, NATS of RabbitMQ om de CloudEvents te publiceren. Dit kan een publish/subscribe-model zijn waarbij abonnees zich kunnen inschrijven voor specifieke soorten gebeurtenissen.
- Schematisch te laten zien hoe het per protocol werkt. Hoe het proces van overeenstemming tussen protocollen plaatsvindt, en hoe we dat gaan doen. Dit biedt ontwikkelaars de vrijheid om de meest geschikte transportmethode voor hun specifieke use case te kiezen.
- Welke broker protocol moet gebruikt worden om de CloudEvents te versturen en om te kunnen subscriben? Dit kan bijvoorbeeld Kafka, AMQP.

## 5. Referenties

- [NL-GOV profile for CloudEvents](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/cloudevents-nl/1.1/)
- [CloudEvents-NL Guidelines](https://gitdocumentatie.logius.nl/publicatie/notificatieservices/guidelines/)
- [CloudEvents.io](https://cloudevents.io/)
- [Wat is NL GOV profile for CloudEvents? - Logius](https://www.logius.nl/onze-dienstverlening/gegevensuitwisseling/nl-gov-profile-cloudevents/wat-nl-gov-profile-cloudevents)

