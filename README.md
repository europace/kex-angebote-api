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
* [Beispiele](#beispiele)
   * [Query bestesAngebot](query-bestesAngebot)
       * [POST Request](#query-bestesAngebot-post-request)
       * [POST Response](#query-bestesAngebot-post-response)
   * [Query angebote](query-angebote)
       * [POST Request](#query-angebote-post-request)
       * [POST Response](#query-angebote-post-response)
   * [Query grenzen](query-grenzen)
       * [POST Request](#query-grenzen-post-request)
       * [POST Response](#query-grenzen-post-response)
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
* [Nutzungsbedingungen](#nutzungsbedingungen)

## Allgemeines

Angebote können über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Ermitteln von Angeboten ist:

    https://kex-angebote.kreditsmart.api.europace.de/angebote
    
Die gewünschten Properties werden als JSON im Body des POST Requests übermittelt.  
Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **200 SUCCESS**.  
Die angeforderten Daten werden ebenfalls als JSON übermittelt.


## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über den OAuth 2.0 Client-Credentials Flow. 

| Request Header Name | Beschreibung           |
|---------------------|------------------------|
| Authorization       | OAuth 2.0 Bearer Token |


Das Bearer Token kann über die [Authorization-API](https://github.com/europace/authorization-api) angefordert werden. 
Dazu wird ein Client benötigt der vorher von einer berechtigten Person über das Partnermanagement angelegt wurde, 
eine Anleitung dafür befindet sich im [Help Center](https://europace2.zendesk.com/hc/de/articles/360012514780).

Damit der Client für diese API genutzt werden kann, muss im Partnermanagement die Berechtigung **Kreditsmartangebote ermitteln** aktiviert sein.  
 
Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

Hat der Client nicht die benötigte Berechtigung um die Resource abzurufen, erhält der Aufrufer eine HTTP Response mit Statuscode **403 FORBIDDEN**.

## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional. 
Wenn er nicht gesetzt ist, wird eine ID vom System generiert.
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

## Beispiele 

### Query bestesAngebot

#### POST Request

    POST https://kex-angebote.kreditsmart.api.europace.de/angebote
    X-Authentication: xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query bestesAngebot($partnerId: String!, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int!) { 
         bestesAngebot(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten) {
            ratenkredit {
                produktanbietername
            }
            gesamtkonditionen {
                sollzins
                effektivzins
                gesamtkreditbetrag
            }
         }
      }",
      "variables": {
        "partnerId": "ABC12",
        "auszahlungsbetrag": 10000,
        "laufzeitInMonaten": 72
      }
    }
        
#### POST Response

    {
        "data": {
            "bestesAngebot": {
                "ratenkredit" :{
                    "produktanbietername": "Testbank AG",
                },
                "gesamtkonditionen": {
                    "sollzins": 2.95,
                    "effektivzins": 2.99,
                    "gesamtbetrag": 10916.88
                }
            }
        }
    }

### Query angebote

#### POST Request

    POST https://kex-angebote.kreditsmart.api.europace.de/angebote
    X-Authentication: xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query angebote($partnerId: String!, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int!) { 
         angebote(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten) {
            ratenkredit {
                produktanbietername
            }
            gesamtkonditionen {
                sollzins
                effektivzins
                gesamtkreditbetrag
            }
         }
      }",
      "variables": {
        "partnerId": "ABC12",
        "auszahlungsbetrag": 10000,
        "laufzeitInMonaten": 72
      }
    }
        
#### POST Response

    {
        "data": {
            "angebote": [
                {
                    "ratenkredit" :{
                        "produktanbietername": "Testbank1 AG",
                    },
                    "gesamtkonditionen": {
                        "sollzins": 3.95,
                        "effektivzins": 3.99,
                        "gesamtbetrag": 10916.88
                    }
                },
                {
                    "ratenkredit" :{
                        "produktanbietername": "Testbank2 AG",
                    },
                    "gesamtkonditionen": {
                        "sollzins": 2.95,
                        "effektivzins": 2.99,
                        "gesamtbetrag": 10916.88
                    }
                }
            ]
        }
    }

### Query grenzen

#### POST Request

    POST https://kex-angebote.kreditsmart.api.europace.de/grenzen
    X-Authentication: xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query grenzen($partnerId: String) { 
        grenzen(partnerId: $partnerId) { 
            auszahlungsbetragMin 
            laufzeitInMonatenMin 
        } 
      }",
      "variables": {
        "partnerId": "ABC12"
      }
    }
        
#### POST Response

    {
        "data": {
            "grenzen": {
                "auszahlungsbetragMin": 1000.0,
                "laufzeitInMonatenMin": 12,
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
  * wenn nicht angegeben, wird TESTUMGEBUNG angenommen

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

## Query grenzen  

Ermittlung der gültigen Grenzen von Laufzeit und Auszahlungsbetrag.
Die Grenzen können sich ändern - sie sind abhängig von den Handelsbeziehungen des Vertriebs und den konkreten Produkten der Bank-Partner.

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
       | auszahlungsbetragSchrittweite | BigDecimal: Betrag in Euro |  
       | auszahlungsbetragMax          | BigDecimal: Betrag in Euro |  
       | laufzeitInMonatenMin          | Int                        | 
       | laufzeitInMonatenSchrittweite | Int                        | 
       | laufzeitInMonatenMax          | Int                        | 
       | ----------------------------- | -------------------------- |

### Grenzen: Response Format

Die erfragten Felder werden - sofern vorhanden - als JSON im Body der Response gesendet. Nicht befüllte Felder werden nicht zurückgegeben.

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


## Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://developer.europace.de/terms/) zur Verfügung gestellt
