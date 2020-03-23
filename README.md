# KEX-Angebote-API

Die Schnittstelle ermöglicht die Ermittlung von Ratenkredit-Angeboten.

> :warning: Diese Schnittstelle wird kontinuierlich weiterentwickelt. Daher erwarten wir 
> von allen Nutzern dieser Schnittstelle, dass sie das "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)" nutzen, d.h. 
> tolerant gegenüber kompatiblen API-Änderungen beim Lesen und Prozessieren der Daten sind:
>
> 1. unbekannte Felder dürfen keine Fehler verursachen
>
> 2. Strings mit eingeschränktem Wertebereich (Enums) müssen mit neuen, unbekannten Werten umgehen können
>
> 3. sinnvoller Umgang mit HTTP-Statuscodes, die nicht explizit dokumentiert sind  
> 
 
<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->
 
# Table of Contents

* [Allgemeines](#allgemeines)
* [Authentifizierung](#authentifizierung)
* [TraceId zur Nachverfolgbarkeit von Requests](#traceid-zur-nachverfolgbarkeit-von-requests)
* [Content-Type](#content-type)
* [Beispiel](#beispiel)
   * [POST Request](#post-request)
   * [POST Response](#post-response)
* [Fehlercodes](#fehlercodes)
   * [HTTP-Status Errors](#http-status-errors)
   * [weitere Fehler](#weitere-fehler)
* [Query bestesAngebot](#query-bestesAngebot)
    * [BestesAngebot - Request Format](#bestesangebot-request-format)
    * [BestesAngebot - Typen der Parameter](#bestesangebot-typen-der-parameter)
    * [BestesAngebot - gewünschte Felder](#bestesangebot-gewnschte-felder)
    * [BestesAngebot - Response Format](#bestesangebot-response-format)
* [Query grenzen](#query-grenzen)
    * [Grenzen - Request Format](#grenzen-request-format)
    * [Grenzen - Typen der Parameter](#grenzen-typen-der-parameter)
    * [Grenzen - gewünschte Felder](#grenzen-gewnschte-felder)
    * [Grenzen - Response Format](#grenzen-response-format)
* [Tools](#tools)

## Allgemeines

Angebote können über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Auslesen von Vorgängen ist:

    https://kex-angebote.kreditsmart.api.europace.de/prod/angebote
    
Die gewünschten Properties werden als JSON im Body des POST Requests übermittelt.  
Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **200 SUCCESS**.  
Die angeforderten Daten werden ebenfalls als JSON übermittelt.


## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über einen HTTP Header.

| Request Header Name | Beschreibung                                        |
|---------------------|-----------------------------------------------------|
| X-Authentication    | API JWT der Vertriebsorganisation oder des Partners |


Das API JWT (JSON Web Token) erhalten Sie von Ihrem Ansprechpartner im KreditSmart-Team. Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional,
wenn er nicht gesetzt ist, wird eine ID vom System generiert.
Hilfreich für die Analyse ist es, wenn die TraceId mit einem System-Kürzel beginnt (im Beispiel unten 'sys').

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige Id für jeden Request | sys12345678 |

## Content-Type

Die Schnittstelle akzeptiert Daten mit Content-Type "**application/json**".  
Entsprechend muss im Request der Content-Type Header gesetzt werden. Zusätzlich das Encoding, wenn es nicht UTF-8 ist.

| Request Header Name |   Header Value   |
|---------------------|------------------|
| Content-Type        | application/json |

## Beispiel 

### POST Request

    POST https://kex-angebote.kreditsmart.api.europace.de/prod/angebote
    X-Authentication: xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query bestesAngebot($partnerId: String!, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int!) { 
         bestesAngebot(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten) {
           produktanbieter
           gesamtbetrag
           nettokreditbetrag
           sollzins
           effektivzins
           monatlicheRate
         }
      }",
      "variables": {
        "partnerId": "ABC12",
        "auszahlungsbetrag": 10000,
        "laufzeitInMonaten": 72
      }
    }
        
### POST Response

    {
        "data": {
            "bestesAngebot": {
                "produktanbieter": "Testbank AG",
                "produktbezeichnung": "Testprodukt Ratenkredit",
                "gesamtbetrag": 10916.88,
                "nettokreditbetrag": 10000.00,
                "laufzeitInMonaten": 72,
                "sollzins": 2.95,
                "effektivzins": 2.99,
                "monatlicheRate": 153.67,
                "letzteRate": 153.42
            }
        }
    }

## Fehlercodes

Die Besonderheit in GraphQL ist u.a., dass die meisten Fehler nicht über HTTP-Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es das Attribut `errors` in der Response.

### HTTP-Status Errors

| Fehlercode | Nachricht       | weitere Attribute          | Erklärung                            |
|------------|-----------------|----------------------------|--------------------------------------|
| 401        | Unauthorized    | -                          | Authentifizierung ist fehlgeschlagen |

### Weitere Fehler
Wenn der Request nicht erfolgreich verarbeitet werden konnte, liefert die Schnittstelle eine 200, aber in dem Attribut `errors` sind Fehlerdetails zu finden

    {
      "data": {},
      "errors": [
        {
          "message": MESSAGE,
          "status": STATUS_CODE
        }
      ]
    }

## Query bestesAngebot  

Für einen Partner wird unter Berücksichtigung seiner Handelsbeziehungen das beste Angebot ermittelt.

### BestesAngebot: Request Format  

Die Angaben werden als JSON im Body des Requests gesendet.
Die Attribute können in beliebiger Reihenfolge angegeben werden.  
Die angegebenen Query-Parameter sind typisiert - ein Ausrufezeichen (!) kennzeichnet die Pflicht-Parameter.
Die konkreten Argumente für die Anfrage werden im **variables**-Teil übergeben.  

    {
        "query": "query bestesAngebot($partnerId: String!, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int!, $finanzierungszweck: Finanzierungszweck, $datenkontext: Datenkontext) { 
            bestesAngebot(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten, finanzierungszweck: $finanzierungszweck, datenkontext: $datenkontext) {
                <gewünschte Felder>
            }
        }",
        "variables": {
            "partnerId": "ABC12",
            "auszahlungsbetrag": 10000,
            "laufzeitInMonaten": 72,
            "finanzierungszweck": "FREIE_VERWENDUNG",
            "datenkontext": "TESTUMGEBUNG"
        }
    }

### BestesAngebot: Typen der Parameter

* partnerId - String
  * die PartnerId ist 5-stellig und identifiziert eine Plakette aus dem Europace-Partnermanagement
  * die angegebene PartnerId muss unterhalb der PartnerId des JWTs liegen oder mit ihr identisch sein.
* auszahlungsbetrag - BigDecimal
  * die erlaubten Werte sind eingeschränkt - siehe dazu die Query *grenzen*  
* laufzeitInMonaten - Int
  * die erlaubten Werte sind eingeschränkt - siehe dazu die Query *grenzen*  
* finanzierungszweck - String
  * hierbei handelt es sich um eine Aufzählung (ENUM) mit folgenden Ausprägungen
    * "UMSCHULDUNG"
    * "FREIE_VERWENDUNG"
    * "FAHRZEUGKAUF"
    * "MODERNISIEREN"  
  * wenn nicht angegeben, wird das beste Angebot über alle Finanzierungszwecke hinweg ermittelt
* datenkontext - String
  * hierbei handelt es sich um eine Aufzählung (ENUM) mit folgenden Ausprägungen
    * "ECHTGESCHAEFT"
    * "TESTUMGEBUNG"
  * wenn nicht angegeben, wird ECHTGESCHAEFT angenommen

### BestesAngebot: gewünschte Felder
    
    Whitespace-separierte Liste folgender Felder

       | Feldname           | Typ                                   |
       | ------------------ | ------------------------------------- |
       | produktanbieter    | String                                |   
       | produktbezeichnung | String                                |     
       | gesamtbetrag       | BigDecimal: Betrag in Euro            |  
       | nettokreditbetrag  | BigDecimal: Betrag in Euro            | 
       | laufzeitInMonaten  | Int                                   | 
       | sollzins           | BigDecimal: 100-basierter Prozentsatz |  
       | effektivzins       | BigDecimal: 100-basierter Prozentsatz |  
       | monatlicheRate     | BigDecimal: Betrag in Euro            |    
       | letzteRate         | BigDecimal: Betrag in Euro            |    
       | ------------------ | ------------------------------------- |
    
### BestesAngebot: Response Format

Die erfragten Felder werden - sofern vorhanden- als JSON im Body der Response gesendet. Nicht befüllte Felder werden nicht zurückgegeben.

    { 
      "data": {
        "bestesAngebot": {
          << ANGEFRAGTE FELDER >>
        }
      },
      "errors": [
        << EVENTUELL AUFGETRETENE FEHLER >>
      ]
    }

## query grenzen  

Ermittlung der gültigen Grenzen von Laufzeit und Auszahlungsbetrag.
Die Grenzen können sich ändern - sie sind abhängig von den Handelsbeziehungen des Vertriebs und den konkreten Produkten der Bank-Partners.

### Grenzen: Request Format  

Die Angaben werden als JSON im Body des Requests gesendet.
Die Attribute können in beliebiger Reihenfolge angegeben werden.  
Die angegebene PartnerId muss unterhalb der PartnerId des JWTs liegen oder mit ihr identisch sein. 
Die angegebenen Query-Parameter sind typisiert - ein Ausrufezeichen (!) kennzeichnet die Pflicht-Parameter.
Die konkreten Argumente für die Anfrage werden im *variables*-Teil übergeben.  

    {
        "query": "query grenzen($partnerId: String!) { 
            grenzen(partnerId: $partnerId) { 
                <gewünschte Felder>
            }
        }",
        "variables": {
            "partnerId": "ABC12"
        }
    }

### Grenzen: Typen der Parameter

* partnerId - String
  * die PartnerId ist 5-stellig und identifiziert eine Plakette aus dem Europace-Partnermanagement
  * die angegebene PartnerId muss unterhalb der PartnerId des JWTs liegen oder mit ihr identisch sein.

### Grenzen: gewünschte Felder
    
    Whitespace-separierte Liste folgender Felder

       | Feldname                      | Typ                        |
       | ----------------------------- | -------------------------- |
       | auszahlungsbetragMin          | BigDecimal: Betrag in Euro |  
       | laufzeitInMonatenSchrittweite | BigDecimal: Betrag in Euro |  
       | auszahlungsbetragMax          | BigDecimal: Betrag in Euro |  
       | laufzeitInMonatenMin          | Int                        | 
       | laufzeitInMonatenSchrittweite | Int                        | 
       | laufzeitInMonatenMax          | Int                        | 
       | ----------------------------- | -------------------------- |

### Grenzen: Response Format

Die erfragten Felder werden - sofern vorhanden- als JSON im Body der Response gesendet. Nicht befüllte Felder werden nicht zurückgegeben.

    { 
      "data": {
        "grenzen": {
          << ANGEFRAGTE FELDER >>
        }
      },
      "errors": [
        << EVENTUELL AUFGETRETENE FEHLER >>
      ]
    }

## Tools

Das GraphQL-Schema kann man z.B. mit dem Tool [GraphiQL](https://electronjs.org/apps/graphiql) analysieren 
und sich per Autocomplete bequem die Query zusammenbauen.


[]: #BestesAngebot-Request-Format
