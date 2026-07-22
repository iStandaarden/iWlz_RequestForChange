![header](/src/ZinBanner.png "template_header")

# Wijzigingsverzoeken
Repository voor het faciliteren en volgen van het wijzigingsproces. Doormiddel van het aanmaken van een issue per wijzigingsverzoek is via het project [iWlz Wijzigingsverzoeken](https://github.com/orgs/iStandaarden/projects/9) inzichtelijk wat de status van het wijzigingsverzoek is. 

## Project en status wijzigingsverzoek
In het project [iWlz Wijzigingsverzoeken](https://github.com/orgs/iStandaarden/projects/9/) is de status van een wijzigingsverzoek te volgen of te bekijken. Het proces is hieronder schematisch weergegeven.


```mermaid
---
config:
    theme: default
    look: neo

---
stateDiagram-v2
direction LR
    [*] --> backlog:01

    state "Backlog" as backlog
    state grpchoice <<choice>>
    note left of grpchoice : Bevinding of ander type?

    bevinding: Bevindingen worden direct opgelost

    backlog --> grpchoice : 02
    grpchoice --> bevinding : 03b - Bevinding
    grpchoice --> backlog: 03c - geen prio

    state "Advies: Referentiegroep / Werkgroep / TAO" as adviesgrp {
        state "Kandidaat" as kandidaat
        state "Bespreken" as bespreken
        state "Uitwerken" as uitwerken
        state "Gereed" as gereed
        state "Vervallen" as vervallen
        
        state refgrp <<choice>>
        note left of refgrp : Bespreking in adviesgroep

        kandidaat --> refgrp 
        uitwerken --> bespreken
        bespreken --> refgrp

        refgrp --> gereed : 04a
        refgrp --> uitwerken : 04b
        refgrp --> vervallen : 04c

        vervallen --> [*] : afgesloten
    }

    state besluit <<choice>> 
    note left of besluit : Besluit of issue in voorgestelde Publicatie (Milestone) kan worden opgenomen
        grpchoice --> kandidaat : 03a - RFC
        refgrp --> backlog : 04d
        gereed --> besluit
        besluit --> backlog : 05b

    state "Akkoord" as akkoord
    state "☑️ Gepubliceerd   ." as gepubliceerd
    state "Besluit: Stuurgroep / Koplopersoverleg" as besluitgrp {
        besluit --> akkoord : 05a
        akkoord --> inplannen : 06b

        inplannen: Inplannen
        inplannen: - Implementatiemoment
        inplannen: - Implementatiedatum
    }

    bevinding --> gepubliceerd
    akkoord --> gepubliceerd : 06a
    gepubliceerd --> [*] : afgesloten
    
    %% kleuren
    classDef dash_lightgreen stroke-width:4px,stroke-dasharray:5,fill:#C8E6C9,stroke:#00C853;
    classDef grey fill:#D3D3D3,stroke:#333
    classDef red fill:#FF4500,stroke:#333
    classDef green fill:#3CB371,stroke:#333
    classDef purple fill:#E1BEE7,stroke:#333
    classDef blue fill:#BBDEFB,stroke:#333
    
    class bevinding dash_lightgreen
    class backlog grey
    class kandidaat grey
    class uitwerken grey
    class bespreken grey
    class vervallen red
    class gereed blue
    class akkoord green
    class gepubliceerd purple
    class inplannen grey   

```

|    # | Toelichting                                                                                                                                   |
| :--- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
|   00 | Nieuwe meldingen worden verzameld bij iStandaarden Beheerteam of kunnen worden aangedragen via [Nieuw issue](https://github.com/iStandaarden/iWlz_RequestForChange/issues/new/choose) en kies het juiste type                                                                                |
|   01 | Alle issues komen op de **`Backlog`** |




## Overzicht wijzigingsverzoeken 
Het volledige overzicht van wijzigingsverzoeken is te vinden onder [Issues](https://github.com/iStandaarden/iWlz_RequestForChange/issues).

## Discussies
Discussies over een wijzigingsverzoek staan in het betreffende [Issue](https://github.com/iStandaarden/iWlz_RequestForChange/issues) of staan onder [Discussions](https://github.com/iStandaarden/iWlz_RequestForChange/discussions) indien van toepassing op het wijzigingsverzoek.





