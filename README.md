# KEX-Angebote-API
 
# Table of Contents

- [Allgemeines](#allgemeines)
  * [Authentifizierung](#authentifizierung)
  * [Nachverfolgbarkeit von Requests](#nachverfolgbarkeit-von-requests)
  * [Request](#request)
  * [Fehlercodes](#fehlercodes)
    + [HTTP-Status Errors](#http-status-errors)
    + [Weitere Fehler](#weitere-fehler)
  * [Tools](#tools)
- [Schaufensterkonditionen](#schaufensterkonditionen)
  * [Query Top-Schaufensterkondition](#query-top-schaufensterkondition)
    + [Request](#request-1)
    + [Response](#response)
    + [Beispiel](#beispiel)
      - [POST Request](#post-request)
      - [POST Response](#post-response)
  * [Query Schaufensterkonditionen](#query-schaufensterkonditionen)
    + [Request](#request-2)
    + [Response](#response-1)
    + [Beispiel](#beispiel-1)
      - [POST Request](#post-request-1)
      - [POST Response](#post-response-1)
- [Angebote](#angebote)
  * [Query Angebote ermitteln](#query-angebote-ermitteln)
    + [Request](#request-3)
    + [Response](#response-2)
    + [Beispiel](#beispiel-2)
      - [POST Request](#post-request-2)
      - [POST Response](#post-response-2)
- [Request-Datentypen](#request-datentypen)
  * [Partner-ID](#partner-id)
  * [Datenkontext](#datenkontext)
  * [Finanzierungszweck](#finanzierungszweck)
- [Response-Datentypen](#response-datentypen)
  * [Schaufensterkondition](#schaufensterkondition)
  * [Angebot](#angebot)
    + [Gesamtkonditionen](#gesamtkonditionen)
      - [Konditionsspanne](#konditionsspanne)
        * [Konditionsgrenze](#konditionsgrenze)
    + [Ratenkredit](#ratenkredit)
      - [Produktanbieter](#produktanbieter)
      - [Anschrift](#anschrift)
      - [Logo](#logo)
- [Nutzungsbedingungen](#nutzungsbedingungen)

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

Alle hier dokumentierten Schnittstellen sind GraphQL-Schnittstellen, sie unterstützen u.a. Schema-Introspection
und gängige [Tools](#tools). Eine Einführung zu GraphQL gibt es z.B. unter [https://graphql.org](https://graphql.org/)

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
| X-TraceId           | eindeutige ID für jeden Request | sys12345678 |

## Request

Die Angaben werden als JSON mit UTF-8 Encoding im Body des Requests gesendet. 
Die Attribute innerhalb eines Blocks können dabei in beliebiger Reihenfolge angegeben werden.  

Die Schnittstelle unterstützt alle gängigen GraphQL Formate, genaueres kann man z.B. unter [https://graphql.org/learn/queries/](https://graphql.org/learn/queries/) nachlesen. 

Im Body des Requests wird die GraphQL-Query als String im Property `query` mitgeschickt. Falls die Query
Parameter enthält, können diese Werte direkt in der Query gesetzt werden oder es können in der Query 
Variablen definiert werden, deren konkrete Werte dann im Property `variables` als Key-Value-Map übermittelt werden.
In unseren Beispielen nutzen wir die Notation mit Variablen.  

    {
      "query": "...",
      "variables": { ... }
    }

## Fehlercodes

Die Besonderheit in GraphQL ist u.a., dass die meisten Fehler nicht über HTTP-Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es das Attribut `errors` in der Response.

### HTTP-Status Errors

| Fehlercode | Nachricht             | Erklärung                                                                                                   |
|------------|-----------------------|-------------------------------------------------------------------------------------------------------------|
| 400        | Bad Request           | Request Format ist ungültig, z.B. Pflichtfelder fehlen, Parameternamen, -typen oder -werte sind falsch, ... |
| 401        | Unauthorized          | Authentifizierung ist fehlgeschlagen                                                                        |
| 403        | Forbidden             | Der API-Client besitzt nicht den richtigen Scope                                                            |
| 415        | Unsupported MediaType | Es wurde ein anderer content-type angegeben                                                                 |

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

## Tools

Das GraphQL-Schema kann man z.B. mit dem Tool [GraphiQL](https://electronjs.org/apps/graphiql) analysieren 
und sich per Autocomplete bequem die Query zusammenbauen.

# Schaufensterkonditionen

Schaufensterkonditionen, sowohl die Top-Schaufensterkondition als auch eine komplette Liste, können über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Ermitteln von Schaufensterkonditionen ist:

    https://kex-angebote.ratenkredit.api.europace.de/schaufenster
    

## Query Top-Schaufensterkondition

### Request

Die GraphQL-Query heißt `topSchaufensterkondition` und hat folgende Parameter:

| Parametername      | Typ                                        | Default                           |
|--------------------|--------------------------------------------|-----------------------------------|
| partnerId          | [Partner-ID](#partner-id)                  | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!                                      | Pflichtfeld                       | 
| laufzeitInMonaten  | Int                                        | -                                 | 
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck)  | Alle Finanzierungszwecke          |
| datenkontext       | [Datenkontext](#datenkontext)              | TESTUMGEBUNG                      |

Der Default-Wert wird verwenden, wenn der jeweilige Parameter nicht gesetzt ist.

### Response

Diese Query liefert als Rückgabewert eine [Schaufensterkondition](#schaufensterkondition)

### Beispiel

#### POST Request

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
        
#### POST Response

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

## Query Schaufensterkonditionen

### Request

Die GraphQL-Query heißt `schaufensterkonditionen` und hat folgende Parameter:

| Parametername      | Typ                                       | Default                           |
|--------------------|-------------------------------------------|-----------------------------------|
| partnerId          | [Partner-ID](#partner-id)                 | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!                                     | Pflichtfeld                       |
| laufzeitInMonaten  | Int                                       | -                                 |
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck) | FREIE_VERWENDUNG                  |
| datenkontext       | [Datenkontext](#datenkontext)             | TESTUMGEBUNG                      |

Der Default-Wert wird verwenden, wenn der jeweilige Parameter nicht gesetzt ist.

### Response

Diese Query liefert als Rückgabewert eine Liste [Schaufensterkondition](#schaufensterkondition)

### Beispiel

#### POST Request

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
        
#### POST Response

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
   
    

# Angebote

Eine Liste von machbaren Angeboten auf Basis von Vorgangsdaten kann über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Ermitteln von Angeboten auf Basis von Vorgangsdaten ist:

    https://kex-angebote.ratenkredit.api.europace.de/angebote  


## Query Angebote ermitteln

### Request

Die GraphQL-Query heißt `angebote` und hat folgende Parameter:

| Parametername      | Typ       | Default      |
|--------------------|-----------|--------------|
| vorgangsnummer     | String!   | Pflichtfeld  |


### Response

Diese Query liefert als Rückgabewert eine Liste von [Angebot](#angebot)

### Beispiel

#### POST Request

    POST https://kex-angebote.ratenkredit.api.europace.de/angebote
    Authorization: Bearer xxxx
    Content-Type: application/json

    {
      "query": "query angebote($vorgangsnummer: String!) {
        angebote(vorgangsnummer: $vorgangsnummer) {
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
        "vorgangsnummer": "ABC123"
      }
    }
        
#### POST Response

    {
        "data": {
            "angbote": {
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

# Request-Datentypen

## Partner-ID

Dieser Typ ist ein 5-stelliger String und identifiziert eine Plakette aus dem Europace-Partnermanagement.  
Die angegebene Partner-ID muss unterhalb der Partner-ID des API-Clients liegen oder mit ihr identisch sein.

## Datenkontext 

Dieser Typ ist ein String, der aktuell folgende Werte annehmen kann
* TESTUMGEBUNG
* ECHTGESCHAEFT

## Finanzierungszweck

Dieser Typ ist ein String, der aktuell folgende Werte annehmen kann
* UMSCHULDUNG
* FREIE_VERWENDUNG
* FAHRZEUGKAUF
* MODERNISIEREN

# Response-Datentypen

Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen.  
Es gibt die Scalare `Euro` und `Prozent`, die jeweils Wrapper für BigDecimal sind.

    
## Schaufensterkondition

    {
        gesamtkonditionen: Gesamtkonditionen
        ratenkredit: Ratenkredit
    }
    
## Angebot

    {
        gesamtkonditionen: Gesamtkonditionen
        ratenkredit: Ratenkredit
    }

### Gesamtkonditionen

    {
        effektivzins: Prozent,
        gesamtkreditbetrag: Euro,
        laufzeitInMonaten: Int,
        nettokreditbetrag: Euro,
        rateMonatlich: Euro,
        sollzins: Prozent,
        konditionsspanne: Konditionsspanne
    }
    
#### Konditionsspanne

    {
        minimum: Konditionsgrenze
        maximum: Konditionsgrenze
    }

##### Konditionsgrenze

    {
        sollzins: Prozent
        effektivzins: Prozent
        rateMonatlich: Euro
        gesamtkreditbetrag: Euro
    }

### Ratenkredit

    {
        produktanbieter: Produktanbieter,
        produktbezeichnung: String,
        schlussrate: Euro
    }

#### Produktanbieter

    {
        name: String
        anschrift: Anschrift
        logo: Logo
    }

#### Anschrift

    {
        strasse: String
        hausnummer: String
        plz: String
        ort: String
    }
    
#### Logo
    
    {
        svg: String
    }    
    
Das Property `svg` enthält die URL auf das SVG.

# Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt
