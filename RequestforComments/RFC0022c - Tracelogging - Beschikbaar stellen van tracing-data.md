![header](../src/ZinBanner.png "template_header")

# RFC0022c - Tracelogging - Beschikbaar stellen van tracing-data

> versie 0.1 d.d. 18-11-2026 - concept

<font size="4">**SAMENVATTING**</font>

**Huidige situatie:**

De bestaande traceloggingafspraken beschrijven de toepassing van `TraceId` en `SpanId`. RFC0022b werkt de parentrelatie uit en [RFC0022d][rfc22d] beschrijft de inhoud en structuur van tracing-data. Voor ketenbrede incidentanalyse zijn daarnaast afspraken nodig over het selecteren, exporteren en beschikbaar stellen van de gegevens bij de betrokken ketenpartijen.

**Beoogde situatie:**

Ketenpartijen stellen op aanvraag van de operationeel netwerkbeheerder (ONB) de lokaal beschikbare tracing-data van een opgegeven trace beschikbaar. De export bevat uitsluitend de ketenset uit [RFC0022d][rfc22d], in OTLP/JSON-formaat. De ONB kan exports van verschillende ketenpartijen combineren.

<font size="4">**CHANGELOG**</font>

| Versie | Datum | Wijzigingen |
| :-- | :-- | :-- |
| 0.1 | 18-11-2026 | Eerste concept. |
---

**Inhoudsopgave**

- [RFC0022c - Tracelogging - Beschikbaar stellen van tracing-data](#rfc0022c---tracelogging---beschikbaar-stellen-van-tracing-data)
  - [| 0.1 | 18-11-2026 | Eerste concept. |](#-01--18-11-2026--eerste-concept-)
- [1. Inleiding](#1-inleiding)
  - [1.1 Uitgangspunten](#11-uitgangspunten)
  - [1.2 Relatie andere RFC](#12-relatie-andere-rfc)
  - [1.3 Scope](#13-scope)
  - [1.4 Use cases](#14-use-cases)
  - [1.5 Afwegingen](#15-afwegingen)
- [2. Terminologie](#2-terminologie)
- [3. Technische uitwerking](#3-technische-uitwerking)
  - [3.1 Aanvraag](#31-aanvraag)
  - [3.2 Selectie](#32-selectie)
  - [3.3 Exportformaat](#33-exportformaat)
  - [3.4 Gegevensminimalisatie en controle](#34-gegevensminimalisatie-en-controle)
- [4. Impact](#4-impact)
- [Bijlage A. Voorbeeldexport](#bijlage-a-voorbeeldexport)

---

# 1. Inleiding

Bij een ketenincident bevindt de tracing-data zich bij verschillende ketenpartijen. Om de verwerking te kunnen reconstrueren moet de ONB de bijbehorende spans kunnen samenbrengen.

## 1.1 Uitgangspunten

- Tracing-data wordt decentraal bij de ketenpartijen opgeslagen.
- Ketenpartijen stellen tracing-data op aanvraag beschikbaar aan de ONB voor ketenbrede incidentanalyse.
- De export is gebaseerd op OpenTelemetry en sluit aan op [RFC0022d][rfc22d].
- Er wordt geen centrale opslagvoorziening verplicht gesteld.
- Deze RFC stelt geen aanvullende eisen aan het overdrachtskanaal; bestaande beveiligingsafspraken blijven gelden.

De afspraken wijzigen geen bestaande verplichtingen voor bewaring of beveiliging. Beschikbaarstelling vindt plaats op aanvraag; continue aanlevering, automatische doorzending en rechtstreekse toegang van de ONB tot lokale systemen worden niet voorgeschreven.

## 1.2 Relatie andere RFC

| RFC | Onderwerp | Relatie* | Toelichting | Issue |
| :-- | :-- | :-- | :-- | :-- |
| RFC0022a | Tracelogging - TraceID en SpanID | Afhankelijk | Biedt de bestaande trace- en spanidentificatie en kondigt beschikbaarstelling aan. | [#37](https://github.com/iStandaarden/iWlz-RFC/issues/37) |
| RFC0022b | Tracelogging - ParentSpanId | Afhankelijk | De export behoudt de parentrelaties zoals voorgesteld in RFC0022b. | [#251](https://github.com/iStandaarden/iWlz_RequestForChange/issues/251) |
| RFC0022d | Tracelogging - Inhoud en structuur van tracing-data | Afhankelijk | Bepaalt de ketenset, structuur en codering waarop de export aansluit. | [#293](https://github.com/iStandaarden/iWlz_RequestForChange/issues/293) |

\* *Afhankelijk* betekent dat deze RFC voortbouwt op de genoemde RFC.

## 1.3 Scope

Deze RFC beschrijft de aanvraag, selectie, export en beschikbaarstelling van lokaal beschikbare tracing-data aan de ONB, inclusief terugkoppeling over ontbrekende gegevens. De lokale opslagoplossing en de inrichting van een centrale voorziening worden niet uitgewerkt.

## 1.4 Use cases

- Bij een mislukte registerraadpleging vraagt de ONB de tracing-data van de betreffende trace op bij de betrokken ketenpartijen en combineert deze voor incidentanalyse.
- Bij een trage interactie vraagt de ONB de tracing-data op bij de betrokken ketenpartijen en vergelijkt de duur van de spans om te bepalen waar de vertraging optreedt.

## 1.5 Afwegingen

- **Decentrale opslag:** er is geen nieuwe centrale voorziening nodig en de gegevens blijven bij de partij die de bron beheert. De ONB moet daardoor bij meerdere ketenpartijen opvragen.
- **Beschikbaarstelling op aanvraag:** gegevensdeling blijft beperkt tot incidenten en sluit aan op bestaande bewaar- en beveiligingsafspraken. Continue aanlevering of doorzending is niet nodig voor incidentanalyse.
- **OTLP/JSON:** sluit aan op het gebruik van OpenTelemetry (RFC0022a) en is met standaardtooling te verwerken.
- **Zelfstandige JSON-documenten:** ieder bestand is los te valideren en te combineren. JSON Lines, zoals beschreven in de OpenTelemetry File Exporter-specificatie, is niet gekozen omdat die specificatie de status Development heeft en nog kan wijzigen.

# 2. Terminologie

| Terminologie | Omschrijving |
| :-- | :-- |
| `TraceId`, `SpanId` | Identifiers van respectievelijk de trace en een afzonderlijke verwerkingsstap, zoals beschreven in de iWlz-begrippenlijst. De JSON-veldnamen zijn `traceId` en `spanId`. |
| Span | Eén bewerking binnen een trace. |
| Ketenset | De selectie van span- en resourcegegevens die volgens [RFC0022d][rfc22d] ketenbreed beschikbaar mag worden gesteld. |
| Export | De op basis van een aanvraag geselecteerde ketenset in OTLP/JSON-formaat. |
| OTLP/JSON | De JSON-representatie volgens het OpenTelemetry Protocol. |
| ONB | Operationeel netwerkbeheerder; vraagt de gegevens op voor ketenbrede incidentanalyse. |

# 3. Technische uitwerking

## 3.1 Aanvraag

De ONB vermeldt bij de aanvraag de `traceId` waarvan tracing-data wordt gevraagd. De `traceId` wordt aangeleverd in de notatie uit RFC0022a. Voor de selectie is geen `spanId` vereist.

De ONB vermeldt, indien bekend, het tijdstip of de periode waarin het incident optrad om de tracing-data eenvoudiger terug te vinden.

De ketenpartij controleert of de aanvraag voldoende duidelijk is om de gegevens te selecteren. Bij een ontbrekende of ongeldige `traceId` vraagt zij om verduidelijking.

## 3.2 Selectie

De ketenpartij selecteert alle lokaal beschikbare, afgeronde spans die bij de opgegeven `traceId` horen binnen de iWlz-interactie. Dit omvat ook spans uit verschillende eigen diensten die onder dezelfde trace zijn vastgelegd.

De selectie betreft uitsluitend tracing-data waarover de ketenpartij beschikt. De partij hoeft geen gegevens bij andere ketenpartijen op te halen. Zij levert de beschikbare gegevens ook wanneer deze slechts een deel van de trace vormen.

Als de gevraagde tracing-data geheel of gedeeltelijk ontbreekt, meldt de ketenpartij dit aan de ONB.

Spans die nog niet zijn afgerond, worden niet opgenomen. Dit wordt vermeld in de melding over ontbrekende gegevens.

## 3.3 Exportformaat

De ketenpartij levert de geselecteerde gegevens als een JSON-document met de structuur van een OTLP `ExportTraceServiceRequest`, volgens de [OTLP/JSON-codering][json] en de [OTLP-tracevoorbeelden][examples].

[Bijlage A](#bijlage-a-voorbeeldexport) bevat een voorbeeldexport met fictieve waarden, gebaseerd op de ketenset uit [RFC0022d][rfc22d]. Het voorbeeld illustreert de toepassing van het exportformaat; de afspraken in deze RFC en [RFC0022d][rfc22d] blijven leidend.

De export behoudt de standaard OTLP-structuur en bevat uitsluitend de gegevens die volgens [RFC0022d][rfc22d] ketenbreed beschikbaar mogen worden gesteld.

Als de omvang meerdere bestanden vereist, bevat ieder bestand een zelfstandig geldig JSON-document met dezelfde OTLP-structuur en de resourcegegevens van de daarin opgenomen spans.

Exports van verschillende ketenpartijen kunnen worden samengebracht door de resourcegroepen te verzamelen en spans te koppelen op basis van `traceId`, `spanId` en `parentSpanId`.

## 3.4 Gegevensminimalisatie en controle

Voor beschikbaarstelling controleert de ketenpartij dat:

- de opgenomen spans de aangevraagde `traceId` hebben;
- de bestanden geldige JSON-documenten met de voorgeschreven OTLP-structuur zijn;
- de export uitsluitend gegevens uit de ketenset bevat, zonder aanvullende attributen of gevoelige waarden;
- de verplichte en toepasselijke conditionele gegevens uit [RFC0022d][rfc22d] aanwezig zijn;
- identifiers, parentrelaties en tijdstippen behouden zijn.

Als de export niet aan deze controles voldoet, meldt de ketenpartij dit aan de ONB. Ontbrekende brongegevens worden niet aangevuld met geschatte waarden.

# 4. Impact

Ketenpartijen moeten tracing-data op `traceId` kunnen terugvinden, de beschikbare spans selecteren en deze volgens §3.3 en [RFC0022d][rfc22d] kunnen exporteren. Daarnaast moeten zij aanvragen van de ONB kunnen afhandelen en de export vóór beschikbaarstelling controleren.

Ketenpartijen en softwareleveranciers moeten beoordelen of hun huidige opslag- en exportvoorzieningen dit ondersteunen en deze waar nodig aanpassen.

---

# Bijlage A. Voorbeeldexport

Dit voorbeeld toont de export van één ketenpartij: een interne verwerkingsstap met een onderliggende HTTP-aanroep die een foutrespons met statuscode `500` ontvangt. Alle waarden zijn fictief. Het voorbeeld bevat uitsluitend gegevens uit de ketenset van [RFC0022d][rfc22d] en is illustratief; de afspraken in deze RFC en [RFC0022d][rfc22d] blijven leidend.

```json
{
  "resourceSpans": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.namespace",
            "value": {
              "stringValue": "urn:nld:agb:12345678"
            }
          },
          {
            "key": "service.name",
            "value": {
              "stringValue": "zorgadministratie"
            }
          }
        ]
      },
      "scopeSpans": [
        {
          "spans": [
            {
              "traceId": "463ac35c9f6413ad48485a3953bb6124",
              "spanId": "a2fb4a1d1a96d312",
              "name": "raadplegen-register",
              "kind": 1,
              "startTimeUnixNano": "1794916800000000000",
              "endTimeUnixNano": "1794916800260000000",
              "status": {
                "code": 2
              }
            },
            {
              "traceId": "463ac35c9f6413ad48485a3953bb6124",
              "spanId": "b3fc5b2e2ba7e423",
              "parentSpanId": "a2fb4a1d1a96d312",
              "name": "POST",
              "kind": 3,
              "startTimeUnixNano": "1794916800010000000",
              "endTimeUnixNano": "1794916800250000000",
              "status": {
                "code": 2
              },
              "attributes": [
                {
                  "key": "http.response.status_code",
                  "value": {
                    "intValue": "500"
                  }
                },
                {
                  "key": "error.type",
                  "value": {
                    "stringValue": "500"
                  }
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

De resource-attributen gelden voor beide spans. De eerste span is een rootspan en heeft daarom geen `parentSpanId`. Bij de HTTP-aanroep verwijst `parentSpanId` naar de eerste span. `kind: 1` staat voor INTERNAL, `kind: 3` voor CLIENT en `status.code: 2` voor ERROR.

De HTTP-statuscode wordt als integer-attribuut weergegeven met `intValue`; de waarde daarvan is volgens OTLP/JSON een decimale tekenreeks. `error.type` is een string-attribuut en gebruikt `stringValue`. Aanvullende resource- en scopemetadata zijn niet opgenomen.

[json]: https://opentelemetry.io/docs/specs/otlp/#json-protobuf-encoding
[examples]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/examples/trace.json
[rfc22d]: https://github.com/iStandaarden/iWlz_RequestForChange/blob/main/RequestforComments/RFC0022d%20-%20Tracelogging%20-%20Inhoud%20en%20structuur%20van%20tracing-data.md