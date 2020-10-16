# KEX-Angebote-API
 
# Table of Contents

* [Allgemeines](#allgemeines)
   * [Schaufensterkonditionen](#schaufensterkonditionen)
* [Beispiele](#beispiele)
   * [Query topSchaufensterkondition](#query-topSchaufensterkondition)
      * [POST Request](#post-request)
      * [POST Response](#post-response)
   * [Query schaufensterkonditionen](#query-schaufensterkonditionen)
      * [POST Request](#post-request-1)
      * [POST Response](#post-response-1)
* [Request](#request)
   * [Authentifizierung](#authentifizierung)
   * [Nachverfolgbarkeit von Requests](#nachverfolgbarkeit-von-requests)
   * [Format](#format)
   * [Request Parameter](#request-parameter)
      * [Top Schaufensterkondition](#query-topSchaufensterkondition-1)
      * [Schaufensterkonditionenliste](#query-schaufensterkonditionen-1)
      * [Partner-ID](#partner-id)
      * [Datenkontext](#datenkontext)
      * [Finanzierungszweck](#finanzierungszweck)
   * [Anfragbare Felder](#anfragbare-felder)
      * [Schaufensterkondition](#schaufensterkondition)
         * [Gesamtkonditionen](#gesamtkonditionen)
            * [Zinsgrenzen](#zinsgrenzen)
         * [Ratenkredit](#ratenkredit)
            * [Produktanbieter](#produktanbieter)
* [Fehlercodes](#fehlercodes)
   * [HTTP-Status Errors](#http-status-errors)
   * [Weitere Fehler](#weitere-fehler)
* [Tools](#tools)
* [Nutzungsbedingungen](#nutzungsbedingungen)

# Allgemeines

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

## Schaufensterkonditionen

Schaufensterkonditionen, sowohl die Top-Schaufensterkondition als auch eine komplette Liste, können über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Ermitteln von Schaufensterkonditionen ist:

    https://kex-angebote.ratenkredit.api.europace.de/schaufenster
    

# Beispiele 

## Query topSchaufensterkondition

### POST Request

    POST https://kex-angebote.ratenkredit.api.europace.de/schaufenster
    Authorization: Bearer xxxx
    Content-Type: application/json

    {
      "query": "query topSchaufensterkondition($partnerId: String, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int, $finanzierungszweck: Finanzierungszweck) { 
         topSchaufensterkondition(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten, finanzierungszweck: $finanzierungszweck) {
            ratenkredit {
                produktanbieter {
                    name
                }
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
        "laufzeitInMonaten": 72,
        "finanzierungszweck": "UMSCHULDUNG"
      }
    }
        
### POST Response

    {
        "data": {
            "topSchaufensterkondition": {
                "ratenkredit": {
                    "produktanbieter": {
                        "name": "Testbank AG"
                    }
                },
                "gesamtkonditionen": {
                    "sollzins": 2.95,
                    "effektivzins": 2.99,
                    "gesamtbetrag": 10916.88
                }
            }
        }
    }

## Query schaufensterkonditionen

### POST Request

    POST https://kex-angebote.ratenkredit.api.europace.de/schaufenster
    Authorization: Bearer xxxx
    Content-Type: application/json

    {
      "query": "query schaufensterkonditionen($partnerId: String, $auszahlungsbetrag: Euro!, $laufzeitInMonaten: Int, $finanzierungszweck: Finanzierungszweck) { 
         schaufensterkonditionen(partnerId: $partnerId, auszahlungsbetrag: $auszahlungsbetrag, laufzeitInMonaten: $laufzeitInMonaten, finanzierungszweck: $finanzierungszweck) {
            ratenkredit {
                produktanbieter {
                    name
                }
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
        "laufzeitInMonaten": 72,
        "finanzierungszweck": "UMSCHULDUNG"
      }
    }
        
### POST Response

    {
        "data": {
            "schaufensterkonditionen": [
                {
                    "ratenkredit": {
                        "produktanbieter": {
                            "name": "Testbank1 AG"
                        }
                    },
                    "gesamtkonditionen": {
                        "sollzins": 3.95,
                        "effektivzins": 3.99,
                        "gesamtbetrag": 10916.88
                    }
                },
                {
                    "ratenkredit": {
                        "produktanbieter": {
                            "name": "Testbank2 AG"
                        }
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

# Request

Die Angaben werden als JSON mit UTF-8 Encoding im Body des Requests gesendet.  
Die Attribute innerhalb eines Blocks können in beliebiger Reihenfolge angegeben werden.  

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

## Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional. 
Wenn er nicht gesetzt ist, wird eine ID vom System generiert.
Hilfreich für die Analyse ist es, wenn die TraceId mit einem System-Kürzel beginnt (im Beispiel unten 'sys').

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige Id für jeden Request | sys12345678 |


## Format
Die Schnittstelle unterstützt alle gängigen GraphQL Formate.
Ein Beispiel ist das folgende Format (siehe auch den [Beispiel Requests](#beispiele)):

    <schaufensterkonditionen/topSchaufensterkondition>(partnerId: <partnerId>, auszahlungsbetrag: <auszahlungsbetrag>, laufzeitInMonaten: <laufzeitInMonaten>, finanzierungszweck: <finanzierungszweck>, datenkontext: <datenkontext>){
        <gewünschte Felder>
    }
    
    
## Request Parameter

### Query topSchaufensterkondition

| Parametername      | Typ                | Default                           |
|--------------------|--------------------|-----------------------------------|
| partnerId          | Partner-ID         | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!              | Pflichtfeld                       | 
| laufzeitInMonaten  | Int                | -                                 | 
| finanzierungszweck | Finanzierungszweck | Alle Finanzierungszwecke          |
| datenkontext       | Datenkontext       | TESTUMGEBUNG                      |

### Query schaufensterkonditionen

| Parametername      | Typ                | Default                           |
|--------------------|--------------------|-----------------------------------|
| partnerId          | Partner-ID         | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!              | Pflichtfeld                       |
| laufzeitInMonaten  | Int                | -                                 |
| finanzierungszweck | Finanzierungszweck | FREIE_VERWENDUNG                  |
| datenkontext       | Datenkontext       | TESTUMGEBUNG                      |

### Partner-ID

Dieser Typ ist ein 5-stelliger String und identifiziert eine Plakette aus dem Europace-Partnermanagement.  
Die angegebene Partner-ID muss unterhalb der Partner-ID des API-Clients liegen oder mit ihr identisch sein.

### Datenkontext 

Dieser Typ ist ein String, der aktuell folgende Werte annehmen kann
* TESTUMGEBUNG
* ECHTGESCHAEFT

### Finanzierungszweck

Dieser Typ ist ein String, der aktuell folgende Werte annehmen kann
* UMSCHULDUNG
* FREIE_VERWENDUNG
* FAHRZEUGKAUF
* MODERNISIEREN


## Anfragbare Felder

Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen.  
Es gibt die Scalare `Euro` und `Prozent`, die jeweils Wrapper für BigDecimal sind.

    
### Schaufensterkondition

    {
        gesamtkonditionen: Gesamtkonditionen
        ratenkredit: Ratenkredit
    }

#### Gesamtkonditionen

    {
        effektivzins: Prozent,
        gesamtkreditbetrag: Euro,
        laufzeitInMonaten: Int,
        nettokreditbetrag: Euro,
        rateMonatlich: Euro,
        sollzins: Prozent,
        zinsgrenzen: Zinsgrenzen
    }

##### Zinsgrenzen

    {
        maximalerEffektivzins: Prozent,
        maximalerSollzins: Prozent,
        minimalerEffektivzins: Prozent,
        minimalerSollzins: Prozent
    }

#### Ratenkredit

    {
        produktanbieter: Produktanbieter,
        produktbezeichnung: String,
        schlussrate: Euro
    }

##### Produktanbieter

    {
        name: String
        anschrift: Anschrift
        logo: Logo
    }

##### Anschrift

    {
        strasse: String
        hausnummer: String
        plz: String
        ort: String
    }
    
##### Logo
    
    {
        svg: String
    }    
    
Das Property `svg` enthält die URL auf das SVG.

# Fehlercodes

Die Besonderheit in GraphQL ist u.a., dass die meisten Fehler nicht über HTTP-Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es das Attribut `errors` in der Response.

## HTTP-Status Errors

| Fehlercode | Nachricht             | Erklärung                                                                                                   |
|------------|-----------------------|-------------------------------------------------------------------------------------------------------------|
| 400        | Bad Request           | Request Format ist ungültig, z.B. Pflichtfelder fehlen, Parameternamen, -typen oder -werte sind falsch, ... |
| 401        | Unauthorized          | Authentifizierung ist fehlgeschlagen                                                                        |
| 403        | Forbidden             | Der API-Client besitzt nicht den richtigen Scope                                                            |
| 415        | Unsupported MediaType | Es wurde ein anderer content-type angegeben                                                                 |

## Weitere Fehler
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
    
# Tools

Das GraphQL-Schema kann man z.B. mit dem Tool [GraphiQL](https://electronjs.org/apps/graphiql) analysieren 
und sich per Autocomplete bequem die Query zusammenbauen.


# Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://developer.europace.de/terms/) zur Verfügung gestellt
