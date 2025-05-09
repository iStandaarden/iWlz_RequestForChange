<!--  
  Template als aanvulling op een RFC of wijzigingsverzoek. 
  Het is gemarkeerd met MarkDown. Ga naar: voor meer informatie hierover
      
  Een mutatie kan zijn: Vervallen / Nieuw / Gewijzigd 
-->
# Analyse
<!--  
  Zet hieronder de nadere analyse van het wijzigingsverzoek. Huidige situatie en gewenste situatie
-->
Probleemanalyse als aanvulling op het ingediende voorstel nodig is.

# Wijzingsvoorstel
<!--  
  Zet hieronder de functionele toelichting van de voorgestelde wijziging. Benoem voor- en nadelen.
  Als er meer dan 1 wijzigingsvoorstellen zijn, nummer en beschrijf ze onder afzonderlijke hoofdstukken.
-->
Functionele beeschrijving van de benodigde wijziging(en). De detail uitwerking volgt hieronder.



# Uitwerking wijzigingsvoorstel per onderdeel
- [Uitwerking wijzigingsvoorstel per onderdeel](#uitwerking-wijzigingsvoorstel-per-onderdeel)
  - [Wijzigingen ERD](#wijzigingen-erd)
  - [Wijzigingen GEGEVENS](#wijzigingen-gegevens)
  - [Wijzigingen REGELS](#wijzigingen-regels)
  - [Wijzigingen AUTORISATIEMATRIX:](#wijzigingen-autorisatiematrix)
  - [Wijzigingen NOTIFICATIE](#wijzigingen-notificatie)
  - [Wijzigingen PROCES](#wijzigingen-proces)
  - [Wijzigingen KOPPELVLAK](#wijzigingen-koppelvlak)
  - [Wijzigingen Policy \& query-template](#wijzigingen-policy--query-template)


## Wijzigingen ERD
De volgende wijzigingen zijn nodig in het ERD

| **Mutatie** | **Nieuwe situatie**                                    |
| :---------- | :----------------------------------------------------- |
| Verwijderen | `Entiteit` verwijderd                                  |
| Nieuw       | `Entiteit` toegevoegd (zie hieronder de samenstelling) |


**Toevoegen Entiteit:**
| **Entiteit**   | **Documentatie** |
| :------------- | :--------------- |
| **`Entiteit`** | _Documentatie._  |

**Samenstelling `Entiteit`:**
| **Element** | **ID** | **Verplicht** | **LDT**  | **primitive** | **Codelijst** | **Documentatie**             | **Regel** |
| :---------- | ------ | ------------- | -------- | ------------- | ------------- | ---------------------------- | --------- |
| **Element** | ja     | ja            | LDT_UUID | string        |               | Technische sleutel Entiteit. |           |




## Wijzigingen GEGEVENS
De volgende wijzigingen zijn nodig op de gegevens (datatypen en codelijsten)


| **Mutatie** | **type**  | **Nieuwe situatie**                                                       |
| :---------- | :-------- | :------------------------------------------------------------------------ |
| Toevoegen   | Codelijst | Codelijst **`CODxx1`** (zie hieronder voor samenstelling)                 |
| Toevoegen   | LDT       | `LDT_Nieuw` - string(3) -  documentatie: (zie documentatie bij codelijst) |

**Toevoegen Codelijst**
| **Codelijst**    | **Documentatie** |
| :--------------- | :--------------- |
| **CODxx1: naam** | _Documentatie_   |

**Samenstelling `CODxx1: naam`**
  | **Mutatie** | **Codelijst**  | **Id** | **Omschrijving** | **ingangsdatum** | **expiratiedatum** | **mutatiedatum** |
  | ----------- | -------------- | ------ | ---------------- | ---------------- | ------------------ | ---------------- |
  | toegevoegd  | `CODxx1: naam` |        |                  |                  |                    |                  |



## Wijzigingen REGELS
De volgende wijzigingen zijn nodig op de regels.

**Bedrijfsregels**

| **Bedrijfsregel** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :---------------- | :---------- | :------ | :-------- | :------------ |
|                   |             |         |           |               |

**Gegevensregels**

| **Gegevensregel** | **Mutatie** | **Oud** | **Nieuw** | **Bedrijfsregel** | **Opmerking** |
| :---------------- | :---------- | :------ | :-------- | :---------------- | :------------ |
|                   |             |         |           |                   |               |

**Autorisatieregels**

| **Autorisatieregel** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :------------------- | :---------- | :------ | :-------- | :------------ |
|                      |             |         |           |               |

**Invulinstructies**

| **Invulinstructie** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :------------------ | :---------- | :------ | :-------- | :------------ |
|                     |             |         |           |               |

## Wijzigingen AUTORISATIEMATRIX: 
De volgende wijzigingen zijn nodig op de Autorisatiematrix.

| **Mutatie** | **Nieuwe situatie** |
| :---------- | :------------------ |
|             |                     |


## Wijzigingen NOTIFICATIE
De volgende wijzigingen zijn nodig op de notificaties.

| **Mutatie** | **Nieuwe situatie** |
| :---------- | :------------------ |
|             |                     |

<!-- copieer onderstaande tabel voor elke notificatie -->
**Inhoud wijziging `Notificatie 1`**:
| **Onderdeel**        | **Inhoud** |
| :------------------- | :--------- |
| Naam                 |            |
| Documentatie         |
| Register             |            |
| Type                 |            |
| Entiteit             |            |
| Trigger              |            |
| Trigger documentatie |            |
| Van                  |            |
| Naar                 |            |
| afzenderIDType       |            |
| ontvangerIDType      |            |
| ontvangerKenmerk     |            |
| eventType            |            |
| subject              |            |
| recordID             |            |


## Wijzigingen PROCES
De volgende wijzigingen zijn nodig op het proces.



## Wijzigingen KOPPELVLAK 
De volgende wijzigingen zijn nodig op het koppelvlak-schema

| **Mutatie** | **Nieuwe situatie** |
| :---------- | :------------------ |
| Verwijderen |                     |



- **Query type** 

```graphql
aanpassing
```

- **node type**
```graphql
aanpassing
```
Schematisch
    



## Wijzigingen Policy & query-template

**Rego-policy**

| **Mutatie** | **Nieuwe situatie** |
| :---------- | :------------------ |
| Toevoegen   |                     |

<!-- Gherkin template gebruiken? -->


**query-template**

```graphQL
aanpassing

```
