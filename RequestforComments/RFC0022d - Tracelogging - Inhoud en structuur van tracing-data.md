![header](../src/ZinBanner.png "template_header")

# RFC0022d - Tracelogging - Inhoud en structuur van tracing-data.md
<font size="4">**SAMENVATTING**</font>

**Huidige situatie:**  
Met TraceId en SpanId kunnen ketenpartijen verwerkingsstappen binnen één interactie terugvinden. Het onderdeel Tracelogging specificeert nog geen gemeenschappelijke gegevensset waarmee zij die stappen kunnen vergelijken.

**Beoogde situatie:**  
Bij een incident kunnen ketenpartijen per stap de dienst, de bewerking, de duur en een eventuele technische fout bepalen. De ketenbreed beschikbare gegevens blijven beperkt tot wat voor die analyse nodig is.