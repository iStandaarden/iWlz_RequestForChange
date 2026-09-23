![header](../src/ZinBanner.png "template_header")

# RFC26284 - Authorization Decision Log (ADL)

> versie 0.1 d.d. 24-09-2026 — concept

<font size="4">**SAMENVATTING**</font>

**Huidige situatie:**  
Binnen het iWlz-netwerkmodel worden autorisatiebeslissingen genomen door een Policy Decision Point (PDP). Voor het vastleggen van deze autorisatiebeslissingen is een uniforme afspraak nodig over de wijze waarop deze logging wordt ingericht.

**Beoogde situatie**

Voor het vastleggen van autorisatiebeslissingen binnen het iWlz-netwerkmodel wordt gebruikgemaakt van de **Authorization Decision Log (ADL)-standaard van Logius**.

De autorisatielogging wordt ingericht conform de gegevensstructuur en eisen van deze standaard. Hiermee wordt aangesloten op een bestaande standaard en wordt geen iWlz-specifieke gegevensstructuur voor autorisatielogging geïntroduceerd.

<font size="4">**Status RFC**</font>

Volg deze [link](https://github.com/iStandaarden/iWlz_RequestForChange/issues/284) om de actuele status van deze RFC te bekijken.

---

**Inhoudsopgave**

- [RFC26284 - Authorization Decision Log (ADL)](#rfc26284---authorization-decision-log-adl)
- [1. Inleiding](#1-inleiding)
  - [1.1 Uitgangspunten](#11-uitgangspunten)
  - [1.2 Scope](#12-scope)
- [2. Terminologie](#2-terminologie)
- [3. Authorization Decision Log](#3-authorization-decision-log)
  - [3.1 Doel en afbakening](#31-doel-en-afbakening)
  - [3.2 Standaard](#32-standaard)
  - [3.3 Bewaartermijn](#33-bewaartermijn)
  - [3.4 Verantwoordelijkheid en beschikbaarheid](#34-verantwoordelijkheid-en-beschikbaarheid)
- [4. Impact](#4-impact)
- [5. Voorgestelde wijziging afsprakenstelsel](#5-voorgestelde-wijziging-afsprakenstelsel)
  - [Referenties](#referenties)

---

# 1. Inleiding

Binnen het iWlz-netwerkmodel worden autorisatiebeslissingen genomen door een Policy Decision Point (PDP). Voor het vastleggen van deze autorisatiebeslissingen wordt gebruikgemaakt van de Authorization Decision Log (ADL)-standaard van Logius.

Deze RFC legt deze keuze vast en beschrijft de bijbehorende afspraken over toepassing, bewaartermijn, verantwoordelijkheid en beschikbaarheid van de Authorization Decision Logs.

Autorisatielogging staat los van tracelogging. Tracelogging maakt het mogelijk om verzoeken en verwerkingsstappen door de keten te volgen. Autorisatielogging legt vast welke autorisatiebeslissing door een PDP is genomen en op basis waarvan toegang tot een verzoek is toegestaan of geweigerd.

## 1.1 Uitgangspunten

Voor deze RFC gelden de volgende uitgangspunten:

* Voor autorisatielogging wordt aangesloten op een bestaande standaard.
* De Authorization Decision Log (ADL)-standaard van Logius is leidend voor de gegevensstructuur en eisen aan de autorisatielogging.
* Deze RFC introduceert geen iWlz-specifieke gegevensstructuur voor autorisatielogging.
* Autorisatielogging heeft betrekking op autorisatiebeslissingen van een Policy Decision Point (PDP).
* Autorisatielogging toont niet noodzakelijk aan dat gegevens daadwerkelijk zijn geraadpleegd.

## 1.2 Scope

Binnen scope van deze RFC vallen:

* het gebruik van de Authorization Decision Log (ADL)-standaard van Logius voor het vastleggen van autorisatiebeslissingen;
* afspraken over het bewaren van Authorization Decision Logs;
* de verantwoordelijkheid voor het vastleggen en bewaren van Authorization Decision Logs;
* de beschikbaarheid van Authorization Decision Logs gedurende de bewaartermijn.

# 2. Terminologie

Onderstaande tabel geeft aan of een begrip al in de iWlz-begrippenlijst is opgenomen of nieuw moet worden toegevoegd.

| Terminologie | Omschrijving | Status begrippenlijst |
| :--- | :--- | :--- |
| Authorization Decision Log (ADL) | Vastlegging van een autorisatiebeslissing conform de Authorization Decision Log-standaard van Logius. | Nieuw |
| Policy Decision Point (PDP) | Policy Decision Point (PDP): neemt de beleidsbeslissingen, voert de policy uit. | Bestaand |
| Autorisatielogging | Logging waarin wordt vastgelegd welke autorisatiebeslissingen door een PDP zijn genomen. | Nieuw |

# 3. Authorization Decision Log

## 3.1 Doel en afbakening

Autorisatielogging heeft als doel om vast te leggen welke autorisatiebeslissingen door een Policy Decision Point (PDP) zijn genomen.

Autorisatielogging staat los van tracelogging. Tracelogging maakt het mogelijk om verzoeken en verwerkingsstappen door de keten te volgen; autorisatielogging legt vast op basis waarvan toegang tot een verzoek is toegestaan of geweigerd.

Autorisatielogging toont daarmee niet noodzakelijk aan dat gegevens daadwerkelijk zijn geraadpleegd. Afspraken over raadpleeg- of inzagelogging vallen buiten deze RFC.

## 3.2 Standaard

Voor het vastleggen van autorisatiebeslissingen wordt gebruikgemaakt van de **[Authorization Decision Log (ADL)-standaard van Logius](https://gitdocumentatie.logius.nl/publicatie/ftv/adl/1.0.0/)**.

De autorisatielogging wordt ingericht conform de in deze standaard opgenomen gegevensstructuur en eisen.

Deze RFC bevat daarom geen aanvullende iWlz-specifieke uitwerking van de gegevensstructuur van een Authorization Decision Log.

## 3.3 Bewaartermijn

Authorization Decision Logs worden gedurende een nader vast te stellen minimale bewaartermijn bewaard.

Voorlopig wordt uitgegaan van een minimale bewaartermijn van **vijf jaar**.

> [!IMPORTANT]
> De definitieve minimale bewaartermijn is nog onderwerp van juridische beoordeling. De termijn van vijf jaar geldt daarom op dit moment als voorstel en kan op basis van de juridische beoordeling worden aangepast.

## 3.4 Verantwoordelijkheid en beschikbaarheid

De partij die verantwoordelijk is voor de uitvoering van de PDP draagt zorg voor het vastleggen en bewaren van de bijbehorende Authorization Decision Logs.

De Authorization Decision Logs moeten gedurende de volledige bewaartermijn beschikbaar en raadpleegbaar blijven.

De verantwoordelijkheden voor de uitvoering hiervan worden nader uitgewerkt in de betreffende rol-specifieke serviceafspraken.

# 4. Impact

De Techniek leverancier die verantwoordelijk is voor de uitvoering van de PDP moet ervoor zorgen dat de bijbehorende autorisatiebeslissingen conform de [Authorization Decision Log (ADL)-standaard van Logius](https://gitdocumentatie.logius.nl/publicatie/ftv/adl/1.0.0/) worden vastgelegd.

De implementatie-impact is afhankelijk van de mate waarin de huidige autorisatielogging al aansluit op de ADL-standaard.

Daarnaast moet de Techniek leverancier rekening houden met het bewaren en beschikbaar houden van de Authorization Decision Logs gedurende de afgesproken bewaartermijn.

# 5. Voorgestelde wijziging afsprakenstelsel

Voorgesteld wordt om in het onderdeel **Logging** van het Afsprakenstelsel iWlz-netwerkmodel het volgende onderdeel over autorisatielogging op te nemen:

> ## Autorisatielogging
>
> ### Doel en afbakening
>
> Autorisatielogging heeft als doel om vast te leggen welke autorisatiebeslissingen door een Policy Decision Point (PDP) zijn genomen.
>
> Autorisatielogging staat los van tracelogging. Tracelogging maakt het mogelijk om verzoeken en verwerkingsstappen door de keten te volgen; autorisatielogging legt vast op basis waarvan toegang tot een verzoek is toegestaan of geweigerd.
>
> Autorisatielogging toont daarmee niet noodzakelijk aan dat gegevens daadwerkelijk zijn geraadpleegd. Afspraken over raadpleeg- of inzagelogging vallen buiten dit onderdeel.
>
> ### Standaard
>
> Voor het vastleggen van autorisatiebeslissingen wordt gebruikgemaakt van de **Authorization Decision Log (ADL)-standaard van Logius**.
>
> De autorisatielogging wordt ingericht conform de in deze standaard opgenomen gegevensstructuur en eisen.
>
> zie: [Authorization Decision Log (ADL)](https://gitdocumentatie.logius.nl/publicatie/ftv/adl/1.0.0/)
>
> ### Bewaartermijn
>
> Voor Authorization Decision Logs wordt voorlopig uitgegaan van een minimale bewaartermijn van vijf jaar.
> 
> De definitieve minimale bewaartermijn is nog onderwerp van juridische beoordeling en kan op basis daarvan worden aangepast.
>
> ### Verantwoordelijkheid en beschikbaarheid
>
> De partij die verantwoordelijk is voor de uitvoering van de PDP draagt zorg voor het vastleggen en bewaren van de bijbehorende Authorization Decision Logs.
>
> De Authorization Decision Logs moeten gedurende de volledige bewaartermijn beschikbaar en raadpleegbaar blijven.
>
> De verantwoordelijkheden voor de uitvoering hiervan zijn nader uitgewerkt in de betreffende rol-specifieke serviceafspraken.

---

## Referenties

* [Authorization Decision Log 1.0.0 — Logius](https://gitdocumentatie.logius.nl/publicatie/ftv/adl/1.0.0/)