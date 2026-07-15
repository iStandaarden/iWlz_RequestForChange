# iWlz-netwerkmodel : RFC007 OAuth-autorisatie


**VERSIE:** 04-03-2023 LOPENDE VERSIE  
**STATUS:** IN PRODUCTIE

> [!CAUTION]
> De tekst van deze RFC is opgenomen in artikel over de dienst [Autoriseren](). De RFC is gebruikt om tot overeenstemming te komen, de tekst van het artikel Autoriseren is leidend.

**Samenvatting**  
Deze RFC beschrijft het autoriseren van systemen in het iWlz-netwerkmodel met behulp van het OAuth 2.0-framework. OAuth 2.0 is een algemeen geaccepteerd autorisatie framework. Het OAuth 2.0-framework is in hoge mate aanpasbaar en accepteert vele grant-types. De meest gebruikte OAuth-grant-types vereisen dat clients vooraf worden geregistreerd bij de autorisatie server of dat tokens vooraf worden overgedragen en ondersteunen geen zero-knowledge-proofs.

Het iWlz-netwerkmodel gebruikt de authorization code grant-type. Deze RFC beschrijft hoe een systeem kan worden geïdentificeerd om vervolgens met een authorization code een access token kan worden opgehaald dat vervolgens kan worden gebruikt als bearer token voor autorisatie bij de resource server.

# 1. Inleiding

In dit document bieden we een manier om service-API's (bijv. RESTful, GraphQL) te beschermen met behulp van een OAuth 2.0-access token. In tegenstelling tot een reguliere OAuth-flow waarbij een gebruiker wordt doorgestuurd naar een authenticatie server om in te loggen, ontvangt de client(applicatie) een authorisatie code. Dit document beschrijft de methode voor het verkrijgen van het token met behulp van een OAuth2-flow en beschrijft het gebruik van het token tijdens een API-request.

# 2. Terminologie

- Client(applicatie) afnemer: de toepassing die toegang aanvraagt.

- JSON Web Token (JWT): Een string welke een set van claims in de vorm van JSON-objecten beschrijft. Deze kunnen digitaal worden ondertekend en/of MACed(soort van sleutel waarmee de authenticiteit van een bericht kan worden gecontroleerd)  en/of versleuteld

- Claim: Informatie over het subject (onderwerp), weergegeven in name/value paar.

- Resource Server: de applicatie (een beschermde bron) die geautoriseerde toegang tot zijn API's vereist.

- Access token: een OAuth 2-access token, geleverd door een autorisatie server. Dit token wordt aan de client overhandigd, zodat deze zichzelf kan autoriseren bij een resource server. De inhoud van het access token is ondoorzichtig (opaque) voor de client. Dit betekent dat de client niets hoeft te weten over de inhoud of structuur van het token zelf.

- Autorisatie server: de autorisatie server controleert de identiteit en de overlegde AutorisatieCredentials van de gebruiker en maakt het access token aan. De autorisatieserver wordt vertrouwd door de resource server. De autorisatieserver ondersteunt introspectie volgens [RFC7662](https://www.rfc-editor.org/rfc/rfc7662).

- Request context: de context van een aanvraag die wordt geïdentificeerd door het access token. Het access token verwijst naar deze context. De context bestaat uit de autorisator (veelal: bronhouder), aanvrager (veelal: afnemer), service-endpoint: elk geregistreerd service-endpoint heeft een unieke referentie. [RFC003](RFC003-Adresboek_92930049.html) beschrijft registratie van services.

# 3. OAuth flow

Het mechanisme voor het ophalen van een access token wordt gedaan volgens de OAuth 2.0 Authorization Code Flow (gedefinieerd in [OAuth 2.0 RFC 6749, section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)).

![Authorization Code Grant Flow](./images/rfc0007-01-Code-grant-flow.png)

Diagram 1 toont de referentie OAuth authorisation code flow welke bestaat uit drie delen: het verkrijgen van de Authorization Code, het verkrijgen van een access token en het gebruiken van het access token.

## 3.1 Verkrijg authorization code

Stap 1 t/m 4 van de Authorization Code Flow weergegeven in diagram 1 worden in het iWlz-netwerkmodel niet uitgevoerd. In het iWlz-netwerkmodel worden authorization codes niet opgevraagd, maar toegestuurd door de authorization server(stap 5), na initiatie vanuit de LUARunner door het optreden van een event.

De authorization code MOET verlopen kort nadat deze is uitgegeven door de authorisatie server om het risico van misbruik te verkleinen. Een maximum authorisatie code levensduur van 10 minuten is AANBEVOLEN.

**De autorisatie code in het iWlz-netwerkmodel verlopen na 1 minuut.**

## 3.2 Verkrijg access token

De client(applicatie) van de afnemer MOET een access token verkrijgen bij de autorisatieserver voordat toegang tot gegevens kan worden verkregen. Dit ZOU MOETEN worden gedaan vlak voordat het gegevens-request wordt verstuurd. Een maximum levensduur van 3600 sec. is AANBEVOLEN voor een Access Token.

Bij het opvragen van de access token in het netwerkmodel gebruikte grant_type “authorization code” MOET een client zich identificeren met een “client_id en client_secret”. Dit is niet van toepassing bij het bevragen van de afgeschermde resource zelf.

Additioneel op de RFC 6749 vereist de TLS verbinding een clientcertificaat(mTLS). De client(applicatie) MOET een geldig VECOZO-systeemcertificaat meesturen welke geregistreerd is bij de hiervoor genoemde client_id. Ook MOET het IP-adres zijn geregistreerd bij VECOZO waarvan uit het verzoek wordt uitgevoerd.

Als het verzoek om een access token ​​geldig en geautoriseerd is, geeft de autorisatieserver een access token en een refresh token uit. Als de aanvraag door de client(applicatie) mislukt of ongeldig is, stuurt de autorisatieservereen  foutcode terug zoals beschreven in paragraaf 5.2.

De client(applicatie) van de afnemer MOET het access token vernieuwen als er opnieuw, meer of andere gegevens worden opgevraagd.

**Een access token is slechts 1 dag geldig (bepaald door de autorisatie server)**

## 3.3 Access token gebruiken

Bij het opvragen van gegevens MOET de client(application) van de afnemer het access token toevoegen aan de Authorization-header als een Bearer-token, zoals vermeld in [RFC7523](https://tools.ietf.org/html/rfc7523). De resource server MOET het access token valideren bij de autorisatie server. De resource server en autorisatie server MOETEN samen beslissen of een access token geldig is voor de aangevraagde resource.

**Een access token kan maar 1 keer worden gebruikt.**

# 4. Autorisatie-client

## 4.1 Registratie

### 4.1.1 Registratie van client

Bij reguliere OAuth2-flows moet een OAuth-client worden geregistreerd bij de autorisatie server met zijn client-ID en client-secret. Zo weet de autorisatie server welke verzoeken door welke partij worden gedaan. De registratie omvat nu handmatige stappen van registreren en goedkeuren.

# 5. Autorisatie server

### 5.2.2 Foutcodes

Fouten worden geretourneerd zoals beschreven in <https://tools.ietf.org/html/rfc6749#section-5.2:>.

``` syntaxhighlighter-pre
     HTTP/1.1 400 Bad Request
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "error":"invalid_request"
     }
```

## 5.3 Access token

Er is geen beperking op het type access token dat wordt uitgegeven door de autorisatie server. De volgende punten zijn echter van toepassing op alle vormen van access tokens:

- Elk willekeurig getal MOET minimaal 256 bits lang zijn en base64-gecodeerd. De autorisatie server MOET ervoor zorgen dat deze uniek is en niet opnieuw wordt gebruikt voor een andere context.

- De autorisatieserver MOET de context opslaan die aan het token is gekoppeld, zodat de resource server erom kan vragen.

- Tokens MOGEN NIET langer dan 3600 seconden (1 uur) geldig zijn.

- Het access token MOET gemarkeerd zijn om alleen voor een specifieke dienstverlener te werken. Dus wanneer de client(applicatie) van een afnemer resources aanvraagt, kan het access token samen met het TLS-clientcertificaat worden vergeleken met de bekende dienstverlener. Dit voorkomt dat het resulterende access token wordt gekaapt.

## 5.4 Refresh Token

Met de refresh token die de autorisatieserver aan de client heeft uitgegeven, kan de client een vernieuwingsverzoek versturen aan het token endpoint van de autorisatieserver eindpunt door de   volgende parameters met behulp van het "application/x-www-form-urlencoded" formaat.


HTTP request entity-body:
| type | voorwaarde | uitleg |
| :-- | :-- | :-- |
| grant_type | MOET |  Waarde moet worden gezet op "refresh_token". |
| refresh_token | MOET |  De refresh token eerder uitgedeeld door de autorisatieserver.|

Omdat refresh tokens doorgaans langdurige inloggegevens zijn die worden gebruikt om   (aanvullende) toegangstokens aan te vragen, is de refresh token gebonden aan de client(applicatie) aan waaraan het is uitgegeven. De client(applicatie) MOET zich authenticeren met de autorisatieserver zoals beschreven bij het opvragen van de access token.

Een voorbeeld van een HTTP verzoek over TLS verbinding (Met extra regeleinden t.b.v. de leesbaarheid)

``` syntaxhighlighter-pre
   POST /token HTTP/1.1

     Host: [server.example.com](http://server.example.com)
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded
     grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA

``` 
 

De autorisatieserver MOET:

- De client(applicatie) autoriseren op basis van de uitgegeven client credentials (Client_Id en Client_secret)

- Controleren of de Refresh token is afgegeven aan de betreffende client(applicatie).

- Controleren van de geldigheid van de Refresh token

Als aan bovenstaande voorwaarden is voldaan wordt er zowel een nieuw access token als ook een nieuwe refresh token geretourneerd. Als het vernieuwingsverzoek fout gaat of wordt afgewezen zal de autorisatieserver een foutmelding retourneren.

The Client(application) MOET de oude refresh token verwijderen en vervangen door de nieuwe refresh token. De autorisatieserver MOET de oude refresh token revoken.

**Een Refresh Token heeft een lifetime van 7 dagen.**

### 5.4.1 Refresh Token Rotation

Refresh token rotation is een techniek om nieuwe access Tokens op te vragen waarbij de Refresh Token een nieuwe expiration date krijgt.

## 5.5 Rate limiting

Het access token wordt gebruikt in verschillende flows, waaronder geautomatiseerde flows. Dit kan de autorisatie server behoorlijk belasten. Een autorisatie-client MOET daarom waar mogelijk tokens hergebruiken. Wanneer een autorisatie server zwaar wordt belast, KAN deze de statuscode 429 TOO MANY REQUEST (RFC009) retourneren. Het moet dan ook de `Retry-After`-header toevoegen met het aantal seconden dat de autorisatie-client MOET wachten. De resource server KAN deze statuscode ook retourneren als een autorisatie-client om tokens vraagt terwijl eerdere tokens nog geldig zijn. Gezien het feit dat de autorisatie-client mogelijk in een geclusterde omgeving draait, MAG deze niet meer dan 10 overlappende tokens aanvragen. Een token overlapt als de velden `iss`, `sub`, `sid` en `usi` hetzelfde zijn of als alleen de velden `iss` en `sub` hetzelfde zijn in het geval dat een token wordt gebruikt zonder subject-context.

# 6. Resource Server

## 6.1. Access token

Het Access token MOET aanwezig zijn in de Authorization-header als bearer token:

``` syntaxhighlighter-pre
POST  /wlzindicaties/wlzindicatiespersoon HTTP/1.1
     Host: resources.example.com
     Authorization: Bearer eyJhbGciOiJFUzI1NiIsImtpZCI6IjE2In0.
     eyJpc3Mi[...omitted for brevity...].
     J9l-ZhwP[...omitted for brevity...]
```

Een token KAN meerdere keren worden gebruikt, tenzij de geretourneerde foutcode dit verhindert. Een token MAG NIET worden gebruikt als deze is verlopen.

## 6.2 Autorisatie

De resource server MOET de geldigheid van het access token valideren. Het KAN contact opnemen met de autorisatie server om het token te valideren, of het KAN bestaande kennis gebruiken om het token te valideren. Een JWT kan bijvoorbeeld worden gevalideerd door de geregistreerde openbare sleutel van de autorisatie server te gebruiken. De volgende stap is om te valideren of het token mag worden gebruikt om toegang te krijgen tot de gevraagde resource. Er zijn drie verschillende gevallen die MOETEN worden ondersteund:

1.  De gevraagde resource bevat geen persoonsgegevens. Bepaalde resource bevatten geen persoonsgegevens en kunnen daarom zonder gebruikerscontext worden uitgewisseld. Resources die in deze categorie vallen, MOETEN als zodanig worden gemarkeerd in de specifieke use case-specificatie.

2.  De aangevraagde resource bevat persoonsgegevens. In het iWlz-netwerkmodel wordt ervoor gekozen om ook in deze gevallen nog NIET te valideren dat de gebruikerscontext aanwezig is, b.v. er is een access token aangevraagd met het `usi`-veld. De resource server MOET ook verifiëren dat er een AuthorizationCredential is gebruikt in het verzoek om access token voor de combinatie van autorisator, aanvrager, subject en resource.

3.  De aanvrager en autorisator zijn dezelfde. Het kan voorkomen dat een deelnemer gebruik maakt van meerdere dienstverleners. Elke dienstverlener handelt dan namens de deelnemer. Het is niet nodig om gebruikerscontext op te geven. Het is aan de dienstverleners om te zorgen voor de juiste handhaving van rollen en eventuele controletaken. Elk van de dienstverleners (aanvrager en autorisator) KAN verschillende identificatiemiddelen gebruiken voor dezelfde deelnemer.

In de eerste twee gevallen MOET de resource server controleren of het toegangsbeleid de aangevraagde resource dekt.

## 6.3 Foutcodes

Verschillende protocollen retourneren verschillende soorten foutmeldingen. Het formaat zal hoogstwaarschijnlijk ook verschillen. Dit betekent dat foutmeldingen per dienst gespecificeerd moeten worden. Als een foutbericht een op tekst gebaseerde foutcode ondersteunt, moet het de code `illegal_access_token` ondersteunen. Als een client deze foutcode ontvangt, MAG hij het access token NIET opnieuw gebruiken.

------------------------------------------------------------------------