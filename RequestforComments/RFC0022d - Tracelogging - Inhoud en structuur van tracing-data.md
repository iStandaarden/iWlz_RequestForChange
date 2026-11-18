![header](../src/ZinBanner.png "template_header")

# RFC0022d - Tracelogging - Inhoud en structuur van tracing-data

> versie 0.2 d.d. 18-11-2026 - concept

<font size="4">**SAMENVATTING**</font>

**Huidige situatie:**

Met `TraceId` en `SpanId` kunnen ketenpartijen verwerkingsstappen binnen één interactie terugvinden. Het onderdeel Tracelogging specificeert nog geen gemeenschappelijke gegevensset waarmee zij die stappen kunnen vergelijken.

**Beoogde situatie:**

Bij een incident kunnen ketenpartijen per stap de dienst, de bewerking, de duur en een eventuele technische fout bepalen. De ketenbreed beschikbare gegevens blijven beperkt tot wat voor die analyse nodig is.

<font size="4">**CHANGELOG**</font>

| Versie | Datum | Wijzigingen |
| :-- | :-- | :-- |
| 0.2 | 18-11-2026 | §3.1: datatype `http.response.status_code` gecorrigeerd van `string` naar `int` (OTLP/JSON: `intValue`), met verwijzing naar `AnyValue`.<br>§3.1: tekst boven de tabel aangevuld met "ontvangen of verzonden" statuscode.<br>§3.1: voorbeelden tonen nu de OTLP/JSON-weergave. |
| 0.1 | 16-11-2026 | Eerste concept. |

---

**Inhoudsopgave**

- [RFC0022d - Tracelogging - Inhoud en structuur van tracing-data](#rfc0022d---tracelogging---inhoud-en-structuur-van-tracing-data)
- [1. Inleiding](#1-inleiding)
  - [1.1 Uitgangspunten](#11-uitgangspunten)
  - [1.2 Relatie andere RFC](#12-relatie-andere-rfc)
  - [1.3 Scope](#13-scope)
  - [1.4 Use cases](#14-use-cases)
- [2. Terminologie](#2-terminologie)
- [3. Technische uitwerking](#3-technische-uitwerking)
  - [3.1 Gegevensstructuur en codering](#31-gegevensstructuur-en-codering)
  - [3.2 Gegevensminimalisatie](#32-gegevensminimalisatie)
- [4. Impact](#4-impact)


---

# 1. Inleiding

Bij een technisch incident moeten ketenpartijen kunnen vaststellen welke dienst een verzoek heeft verwerkt en hoe die verwerking is verlopen. Deze RFC beschrijft welke tracing-data daarvoor ketenbreed beschikbaar wordt gesteld.

## 1.1 Uitgangspunten

- Voor de identificatie van een trace en de bijbehorende spans gelden de bestaande traceloggingafspraken.
- De gegevensstructuur sluit aan op het OpenTelemetry-spanmodel. De ketenset bevat uitsluitend de gegevens die nodig zijn voor technische traceerbaarheid en incidentanalyse.

## 1.2 Relatie andere RFC

Deze RFC heeft een relatie met de volgende RFC’s:

| RFC | Onderwerp | Relatie* | Toelichting | Issue |
| :-- | :-- | :-- | :-- | :-- |
| RFC0022a | Tracelogging - TraceID en SpanID | Afhankelijk | RFC0022d gebruikt de in RFC0022a beschreven trace- en spanidentificatie. | [#37](https://github.com/iStandaarden/iWlz-RFC/issues/37) |
| RFC0022b | Tracelogging - ParentSpanId | Afhankelijk | RFC0022d sluit aan op de parentrelatie zoals voorgesteld in RFC0022b. | [#251](https://github.com/iStandaarden/iWlz_RequestForChange/issues/251) |
| RFC0022c | Tracelogging - Beschikbaar stellen van tracing-data | Voorwaardelijk | RFC0022d bepaalt de inhoud, structuur en codering. RFC0022c werkt de wijze van beschikbaar stellen uit. | [#49](https://github.com/iStandaarden/iWlz-RequestForComment/issues/49) |
| RFC26288 | Notificeren en melden met CloudEvents | Gerelateerd | Voor de invulling van `service.namespace` sluit RFC0022d aan op de organisatie-identificatie uit §3.2.1.2 van RFC26288. | [#288](https://github.com/iStandaarden/iWlz_RequestForChange/issues/288) |

\* *Afhankelijk* betekent dat deze RFC voortbouwt op de genoemde RFC. *Voorwaardelijk* betekent dat de genoemde RFC op deze RFC voortbouwt.

## 1.3 Scope

Deze RFC bepaalt de inhoud, betekenis en codering van de tracing-data per afgeronde span binnen één iWlz-interactie.

## 1.4 Use cases

- Bij een trage registerraadpleging vergelijkt de operationeel netwerkbeheerder de duur van de betrokken spans.
- Bij een mislukte aanroep kan een ketenpartij onderscheid maken tussen een ontvangen HTTP-foutrespons en een time-out voordat een respons werd ontvangen.

# 2. Terminologie

| Terminologie | Omschrijving |
| :-- | :-- |
| Span | Eén bewerking binnen een trace. |
| Resource | De bron van de span, hier de dienst die de bewerking uitvoert. |
| Ketenset | De selectie van span- en resourcegegevens die ketenbreed beschikbaar mag worden gesteld. |
| `TraceId`, `SpanId` | Identifiers van respectievelijk de trace en de span, zoals beschreven in de iWlz-begrippenlijst. |
| `ParentSpanId` | De `SpanId` van de direct bovenliggende span, zoals uitgewerkt in RFC0022b. |

# 3. Technische uitwerking

## 3.1 Gegevensstructuur en codering

Het [OpenTelemetry-spanmodel][model] vormt het uitgangspunt voor de structuur van de tracing-data. De ketenbreed gedeelde tracing-data wordt vastgelegd in [OTLP/JSON-formaat][json].

De onderstaande tabel bepaalt welke gegevens per afgeronde span worden vastgelegd voor ketenbrede incidentanalyse. Verplichte gegevens zijn voor iedere afgeronde span van toepassing. Conditioneel verplichte gegevens worden opgenomen zodra aan de genoemde voorwaarde is voldaan en anders weggelaten. Voor standaardwaarden gelden de weglatingsregels van OTLP/JSON. Aanvullende iWlz-afspraken zijn expliciet vermeld.

OTLP/JSON gebruikt het Protobuf-schema als gegevensmodel en de [OTLP/JSON-regels][json] voor de JSON-weergave.


Bij afronding van de verwerkingsstap worden de eindtijd en het resultaat in de span vastgelegd. Bij een HTTP-aanroep of -verwerking wordt de ontvangen of verzonden statuscode vastgelegd in de betreffende span.

| Veld | Verplichting | Datatype | Beschrijving | Voorbeeld |
| :-- | :-- | :-- | :-- | :-- |
| `traceId` | Verplicht | `string` (hexadecimaal) | Identifier van de trace (16 bytes). Zie ook [RFC0022a][rfc22a]. | `"463ac35c9f6413ad48485a3953bb6124"` |
| `spanId` | Verplicht | `string` (hexadecimaal) | Identifier van de span (8 bytes). Zie ook [RFC0022a][rfc22a]. | `"a2fb4a1d1a96d312"` |
| `parentSpanId` | Conditioneel verplicht: als de span een parent heeft. | `string` (hexadecimaal) | Identifier van de parent (8 bytes). Bij een rootspan wordt dit veld weggelaten. Zie ook [RFC0022b][rfc22b]. | `"a2fb4a1d1a96d312"` |
| `name` | Verplicht | `string` | Naam van de bewerking die de span beschrijft, conform [OpenTelemetry Span][api]. Voor HTTP-spans gelden de [HTTP-naamgevingsconventies][http-name]. | `"POST"` |
| `kind` | Verplicht | `number` (geheel getal) | Rol van de span conform [OpenTelemetry SpanKind][kind]. Toegestane waarden volgens de [OTLP-enum SpanKind][proto] zijn `1` INTERNAL, `2` SERVER, `3` CLIENT, `4` PRODUCER en `5` CONSUMER. | `3` |
| `startTimeUnixNano` | Verplicht | `string` (decimaal getal) | Starttijd van de span, conform `start_time_unix_nano` in de [OTLP-spandefinitie][proto], gecodeerd volgens [OTLP/JSON][json]. | `"1794916800000000000"` |
| `endTimeUnixNano` | Verplicht | `string` (decimaal getal) | Eindtijd van de span, conform `end_time_unix_nano` in de [OTLP-spandefinitie][proto], gecodeerd volgens [OTLP/JSON][json]. De eindtijd is gelijk aan of later dan de starttijd. | `"1794916800250000000"` |
| `status.code` | Verplicht; de standaardwaarde UNSET mag volgens OTLP/JSON worden weggelaten. | `number` (geheel getal) | Spanstatus conform [OpenTelemetry Set Status][status]. De [OTLP-definitie Status.StatusCode][proto] bepaalt de waarden: `0` UNSET, `1` OK en `2` ERROR. Voor HTTP-spans gelden de [HTTP-statusconventies][http-status]. | `2` |
| `service.namespace` | Verplicht | `string` | Resource-attribuut dat de namespace van `service.name` bepaalt, conform [OpenTelemetry Service Namespace][namespace]. De invulling sluit aan op de organisatie-identificatie uit [RFC26288 §3.2.1.2][rfc26288-source]. | `{ "stringValue": "urn:nld:agb:12345678" }` |
| `service.name` | Verplicht | `string` | Logische naam van de dienst, uniek binnen `service.namespace`, conform [OpenTelemetry Service][service]. | `{ "stringValue": "zorgadministratie" }` |
| `http.response.status_code` | Conditioneel verplicht: als een HTTP-statuscode is ontvangen of verzonden. | `int` (in OTLP/JSON als decimale string in `intValue`) | [HTTP-responsstatuscode][http-statuscodes], conform de [OpenTelemetry HTTP-attribuutdefinitie][http-attribute]. Codering van `intValue` volgens [AnyValue][anyvalue]. | `{ "intValue": "200" }` |
| `error.type` | Conditioneel verplicht: als de HTTP-aanroep volgens de HTTP-conventies met een fout is geëindigd. | `string` | Foutklasse of de toepasselijke HTTP-code als tekenreeks, conform [Error attributes][error] en de toepassingsregels uit [HTTP spans][http]. | `{ "stringValue": "timeout" }`; `{ "stringValue": "500" }` |

De tabel bepaalt welke gegevens ketenbreed worden gedeeld. Voor de lokale vastlegging kunnen de OpenTelemetry HTTP-conventies aanvullende attributen vereisen.

Voor `service.namespace` sluiten we aan op de organisatie-identificatie uit [RFC26288 §3.2.1.2][rfc26288-source]. We gebruiken het deel `urn:nld:<identificatietype>:<organisatie-ID>` van de daar beschreven `source`. Het onderdeel `:events:…` wordt niet overgenomen.

## 3.2 Gegevensminimalisatie

De tracing-data die ketenbreed wordt gedeeld, bevat uitsluitend de gegevens uit §3.1. De veldwaarden in deze gegevens mogen geen cliëntidentificatoren, berichtinhoud, IP-adressen, authenticatiegegevens of andere gevoelige informatie bevatten. Dit geldt ook voor spannamen en foutklassen.

# 4. Impact

Het gebruik van OpenTelemetry is al voorgeschreven in RFC0022a. Deze RFC vereist dat de tracing-data ook de aanvullende gegevens uit §3.1 bevat. Afhankelijk van de bestaande inrichting vraagt dit om configuratie of uitbreiding van de OpenTelemetry-instrumentatie.

Ketenpartijen en softwareleveranciers moeten hun tracelogging waar nodig aanvullen en aansluiten op de voorgeschreven definities en formaten. De omvang van de aanpassingen hangt af van de huidige inrichting. Een implementatie die uitsluitend `TraceId` en `SpanId` vastlegt, vraagt meer uitbreiding dan een implementatie die al volledige spans registreert.

[model]: https://opentelemetry.io/docs/specs/otel/overview/#spans
[api]: https://opentelemetry.io/docs/specs/otel/trace/api/#span
[status]: https://opentelemetry.io/docs/specs/otel/trace/api/#set-status
[kind]: https://opentelemetry.io/docs/specs/otel/trace/api/#spankind
[json]: https://opentelemetry.io/docs/specs/otlp/#json-protobuf-encoding
[proto]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/trace/v1/trace.proto
[anyvalue]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/common/v1/common.proto
[http]: https://opentelemetry.io/docs/specs/semconv/http/http-spans/#http-client-span
[http-name]: https://opentelemetry.io/docs/specs/semconv/http/http-spans/#name
[http-status]: https://opentelemetry.io/docs/specs/semconv/http/http-spans/#status
[service]: https://opentelemetry.io/docs/specs/semconv/resource/service/#service
[namespace]: https://opentelemetry.io/docs/specs/semconv/resource/service/#service-namespace
[error]: https://opentelemetry.io/docs/specs/semconv/registry/attributes/error/#error-attributes
[rfc22a]: https://github.com/iStandaarden/iWlz-RFC/issues/37
[rfc22b]: https://github.com/iStandaarden/iWlz_RequestForChange/issues/251
[http-attribute]: https://opentelemetry.io/docs/specs/semconv/registry/attributes/http/
[http-statuscodes]: https://www.rfc-editor.org/rfc/rfc9110.html#name-status-codes
[rfc26288-source]: https://github.com/iStandaarden/iWlz_RequestForChange/blob/main/RequestforComments/RFC26288%20-%20Notificeren%20en%20melden%20met%20CloudEvents.md#3212-source