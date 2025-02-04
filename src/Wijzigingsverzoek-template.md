<!--  
  Template als aanvulling op een RFC of wijzigingsverzoek. 
  Het is gemarkeerd met MarkDown. Ga naar: voor meer informatie hierover
      
  Een mutatie kan zijn: Vervallen / Toegevoegd / Gewijzigd 
-->

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

| **Mutatie** |  **Nieuwe situatie** |
| :--- | :--- | 
| Verwijderen |  `Entiteit` verwijderd |  
| Toevoegen |   `Entiteit` toegevoegd (zie hieronder de samenstelling) |


**Toevoegen Entiteit:**
| **Entiteit** | **Documentatie** |
| :-- | :--- |
| **`Entiteit`** | _Documentatie._ |

**Samenstelling `Entiteit`:**
| **Element** | **ID** | **Verplicht** | **LDT** | **primitive** | **Codelijst** | **Documentatie** |  **Regel** | 
|:---|---|---|---|---|---|---|---|
| **Element** | ja | ja | LDT_UUID | string | | Technische sleutel Entiteit.  | | 




## Wijzigingen GEGEVENS
De volgende wijzigingen zijn nodig op de gegevens (datatypen en codelijsten)


| **Mutatie** | **type** | **Nieuwe situatie** |
| :--- | :--- | :-- |
| Toevoegen | Codelijst |  Codelijst **`CODxx1`** (zie hieronder voor samenstelling) |
| Toevoegen |  LDT |  `LDT_Nieuw` - string(3) -  documentatie: (zie documentatie bij codelijst) |

**Toevoegen Codelijst**
| **Codelijst** | **Documentatie** |
| :-- | :--- |
|  **CODxx1: naam** | _Documentatie_| 

**Samenstelling `CODxx1: naam`**
  **Mutatie** | **Codelijst** | **Id** | **Omschrijving** | **ingangsdatum** | **expiratiedatum** | **mutatiedatum** | 
  |---|---|---|---|---|---|---|
  | toegevoegd | `CODxx1: naam` |  |  |  |  |   |



## Wijzigingen REGELS
De volgende wijzigingen zijn nodig op de regels.

| **Bedrijfsregel** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :--- | :--- | :--- |  :--- | :-- |
|  |  |  |  |  |


| **Gegevensregel** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :--- | :--- | :--- |  :--- | :-- |
|  |  |  |  |  |


| **Autorisatieregel** | **Mutatie** | **Oud** | **Nieuw** | **Opmerking** |
| :--- | :--- | :--- |  :--- | :-- |
|  |  |  |  |  |

## Wijzigingen AUTORISATIEMATRIX: 
De volgende wijzigingen zijn nodig op de Autorisatiematrix.

| **Mutatie** | **Nieuwe situatie** |
| :--- | :--- | 
|  |  |  


## Wijzigingen NOTIFICATIE
De volgende wijzigingen zijn nodig op de notificaties.

| **Mutatie** |  **Nieuwe situatie** |
| :--- | :--- | 
|  |  |


**Inhoud wijziging `Notificatie 1`**:
| **Onderdeel** | **Inhoud** |
| :-- | :-- | 
| **Naam** | | 
| **Documentatie** |
| **Register** | | 
| **Type** | | 
| **Entiteit** | | 
| **Trigger** | | 
| **Trigger documentatie** | | 
| **Van** | | 
| **Naar** | | 
| **afzenderIDType** | | 
| **ontvangerIDType** | | 
| **ontvangerKenmerk** | | 
| **eventType** | | 
| **subject** | | 
| **recordID** | | 


## Wijzigingen PROCES
De volgende wijzigingen zijn nodig op het proces.



## Wijzigingen KOPPELVLAK 
De volgende wijzigingen zijn nodig op het koppelvlak-schema

| **Mutatie** |  **Nieuwe situatie** |
| :--- | :--- | 
| Verwijderen |  type ```Dossierhouder``` inclusief hulp-typen verwijderd |



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

| **Mutatie** |  **Nieuwe situatie** |
| :--- |  :--- |
| Toevoegen |  Een zorgaanbieder mag met het meegeven van ```regiehouderID``` (afkomstig uit notificatie) en de eigen ```agbCode```  alleen die Regiehouder en direct gerelateerde Bemiddeling en Client raadplegen | 



**query-template**

```graphQL
aanpassing

```
