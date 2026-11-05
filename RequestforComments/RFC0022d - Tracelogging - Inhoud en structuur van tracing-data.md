![header](../src/ZinBanner.png "template_header")

# RFC0022d - Tracelogging - Inhoud en structuur van tracing-data.md
<font size="4">**SAMENVATTING**</font>

**Huidige situatie:**  
Binnen het iWlz-netwerkmodel ontbreekt het aan gestandaardiseerde tracelogging, waardoor het lastig is om transacties en gebeurtenissen end-to-end te volgen en analyseren.

**Beoogde situatie:**  
- **Traceerbare loggegevens over verschillende registers heen:** Door het implementeren van gestandaardiseerde tracelogging wordt het mogelijk om de volledige keten van gebeurtenissen consistent en eenduidig te volgen binnen het iWlz-netwerkmodel.  
- **Gefaseerde uitbreiding van functionaliteit:** De implementatie van tracelogging is gepland in drie fasen. In de eerste fase ligt de focus op het introduceren van `TraceId` voor basis traceerbaarheid. In latere fasen worden `SpanId`, `ParentSpanId` en mechanismen voor het beschikbaar stellen van loggingdata toegevoegd om de traceerbaarheid en analyse verder te verdiepen.