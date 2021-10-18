# KEX-Angebote-API

## Allgemeines

Die Schnittstelle ermöglicht die Ermittlung von Ratenkredit-Angeboten.  
Alle hier dokumentierten Schnittstellen sind [GraphQL-Schnittstellen](https://docs.api.europace.de/privatkredit/graphql/).

> ⚠️ Diese Schnittstelle wird kontinuierlich weiterentwickelt. Daher erwarten wir 
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

### Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über den OAuth 2.0 Client-Credentials Flow. 

| Request Header Name | Beschreibung           |
|---------------------|------------------------|
| Authorization       | OAuth 2.0 Bearer Token |


Das Bearer Token kann über die [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/) angefordert werden. 
Dazu wird ein Client benötigt der vorher von einer berechtigten Person über das Partnermanagement angelegt wurde. 
Eine Anleitung dafür befindet sich im [Help Center](https://europace2.zendesk.com/hc/de/articles/360012514780).

Damit der Client für die Schaufensterkonditionen-APIs und die Ermittlung von Angeboten genutzt werden können, muss im Partnermanagement die Berechtigung **KreditSmart-Angebote ermitteln** (Scope `privatkredit:angebot:ermitteln`) aktiviert sein.  
Damit der Client für das Annehmen von Angeboten genutzt werden kann, muss im Partnermanagement die Berechtigung **KreditSmart-Anträge anlegen/verändern** (Scope `privatkredit:antrag:schreiben`) aktiviert sein.  
 
Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

Hat der Client nicht die benötigte Berechtigung um die Resource abzurufen, erhält der Aufrufer eine HTTP Response mit Statuscode **403 FORBIDDEN**.

### Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional. 
Wenn er nicht gesetzt ist, wird eine ID vom System generiert.
Hilfreich für die Analyse ist es, wenn die TraceId mit einem System-Kürzel beginnt (im Beispiel unten 'sys').

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige ID für jeden Request | sys12345678 |

### Request

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

### Fehlercodes

Die Besonderheit in GraphQL ist u.a., dass die meisten Fehler nicht über HTTP-Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es das Attribut `errors` in der Response.
Weitere Infos gibt es [hier](https://docs.api.europace.de/privatkredit/graphql/)

### HTTP-Status Errors

| Fehlercode | Nachricht             | Erklärung                                                                                                   |
|------------|-----------------------|-------------------------------------------------------------------------------------------------------------|
| 400        | Bad Request           | Request Format ist ungültig, z.B. Pflichtfelder fehlen, Parameternamen, -typen oder -werte sind falsch, ... |
| 401        | Unauthorized          | Authentifizierung ist fehlgeschlagen                                                                        |
| 403        | Forbidden             | Der API-Client besitzt einen falschen Scope                                                                 |
| 409        | Conflict              | Der Vorgang ist nicht aktuell                                                                               |
| 415        | Unsupported MediaType | Es wurde ein anderer content-type angegeben                                                                 |

## Schaufensterkonditionen

Schaufensterkonditionen, sowohl die Top-Schaufensterkondition als auch eine komplette Liste, können über unsere GraphQL Schnittstelle via **HTTP POST** ermittelt werden.  
Die URL für das Ermitteln von Schaufensterkonditionen ist:

    https://kex-angebote.ratenkredit.api.europace.de/schaufenster
    

### Query Top-Schaufensterkondition

#### Request

Die GraphQL-Query heißt `topSchaufensterkondition` und hat folgende Parameter:

| Parametername      | Typ                                        | Default                           |
|--------------------|--------------------------------------------|-----------------------------------|
| partnerId          | [Partner-ID](#partner-id)                  | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!                                      | - (Pflichtfeld)                       | 
| laufzeitInMonaten  | Int                                        | -                                 | 
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck)  | Alle Finanzierungszwecke          |
| datenkontext       | [Datenkontext](#datenkontext)              | TESTUMGEBUNG                      |

Der Default-Wert wird verwendet, wenn der jeweilige Parameter nicht gesetzt ist.

#### Response

Diese Query liefert als Rückgabewert eine [Schaufensterkondition](#schaufensterkondition).

#### Beispiel

##### POST Request

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
        
##### POST Response

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

### Query Schaufensterkonditionen

#### Request

Die GraphQL-Query heißt `schaufensterkonditionen` und hat folgende Parameter:

| Parametername      | Typ                                       | Default                           |
|--------------------|-------------------------------------------|-----------------------------------|
| partnerId          | [Partner-ID](#partner-id)                 | Die Partner-ID aus dem API-Client |
| auszahlungsbetrag  | Euro!                                     | - (Pflichtfeld)                   |
| laufzeitInMonaten  | Int                                       | -                                 |
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck) | FREIE_VERWENDUNG                  |
| datenkontext       | [Datenkontext](#datenkontext)             | TESTUMGEBUNG                      |

Der Default-Wert wird verwendet, wenn der jeweilige Parameter nicht gesetzt ist.

#### Response

Diese Query liefert als Rückgabewert eine Liste [Schaufensterkonditionen](#schaufensterkondition).

#### Beispiel

##### POST Request

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
        
##### POST Response

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
   
    

## Angebote

Eine Liste von machbaren und vollständigen Angeboten auf Basis von Vorgangsdaten kann über unsere GraphQL-Schnittstelle via **HTTP POST** ermittelt werden.
Mit der ID aus einem ermittelten Angebot kann dieses danach angenommen werden.
Die URL für das Ermitteln und Annehmen auf Basis von Vorgangsdaten ist:

    https://kex-angebote.ratenkredit.api.europace.de/angebote  


### Query Angebote

#### Hinweise

* Das Ermitteln von Angeboten ist nur möglich, wenn der Vorgang einen gültigen **Kreditbetrag** und **Laufzeit oder Rate** enthält. Sollte das nicht der Fall sein, so erhält der API-Nutzer einen [GraphQL-Error](#fehlercodes) mit dem Statuscode `400`. Der Vorgang muss dann zuerst entsprechend korrigiert werden.


#### Request

Die GraphQL-Query heißt `angebote` und hat folgende Parameter:

| Parametername      | Typ       | Default          |
|--------------------|-----------|------------------|
| vorgangsnummer     | String!   | - (Pflichtfeld)  |


#### Response

Diese Query liefert als Rückgabewert eine Liste von [Angeboten](#angebot).

#### Beispiel

##### POST Request

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

##### POST Response

    {
        "data": {
            "angebote": [
                {
                    "gesamtkonditionen": {
                        "effektivzins": 2.99,
                        "gesamtbetrag": 10916.88,
                        "sollzins": 2.95
                    },
                    "ratenkredit": {
                        "produktanbieter": {
                            "name": "Testbank AG"
                        }
                    }
                }
            ]
        }
    }

### Mutation Angebot-Annehmen

#### Hinweise

* Aktuell unterstützt die API nur das Fernabsatzgeschäft.
* Der authentifizierte Nutzer muss zum Zeitpunkt der Annahme eine Handelsbeziehung für die Bank besitzen, in der die Annahme erlaubt ist.  
  Andernfalls erhält der Nutzer einen [GraphQL-Error](#fehlercodes) mit dem Statuscode `403`
* Das Annehmen von ermittelten Angeboten ist nur möglich, wenn der Vorgang aktuell ist. Sollte nach der Ermittlung und vor der Annahme eine Änderung am Vorgang vorgenommen werden, so erhält der Nutzer der API einen [GraphQL-Error](#fehlercodes) mit dem Statuscode `409`. In diesem Fall ist eine erneute Ermittlung notwendig.
* Zur Optimierung des Angebotsprozess ermitteln wir unter Umständen zusätzliche Alternativangebote unter Adjustierung der Kreditparameter.
* Der im Vorgang eingetragene Kundenbetreuer ist für die Annahme wichtig, da Name und Kontaktdaten an die Banken geschickt werden.
* Wenn sich das Angebot im Annahmeprozess als nicht machbar herausstellt, wird in der Schnittstelle kein Antrag zurückgegeben.


#### Request

Die GraphQL-Mutation heißt `angebotAnnehmen` und hat folgende Parameter:

| Parametername      | Typ       | Default          | Kommentar                                                  |
|--------------------|-----------|------------------|------------------------------------------------------------|
| vorgangsnummer     | String!   | - (Pflichtfeld)  |                                                            |
| angebotId          | String!   | - (Pflichtfeld)  | Die ID des ermittelten Angebots, das angenommen werden soll |

#### Response

Diese Mutation liefert als Rückgabewert eine JobId.

#### Beispiel

##### POST Request

    POST https://kex-angebote.ratenkredit.api.europace.de/angebote
    Authorization: Bearer xxxx
    Content-Type: application/json

    {
      "query": "mutation annehmen($vorgangsnummer: String!, $angebotId: String!) {  
        angebotAnnehmen(angebotId: $angebotId,  vorgangsnummer: $vorgangsnummer ){
            jobId
          }
        }
      }",
      "variables": {
        "vorgangsnummer": "ABC123"
        "angebotId": "angebotId"
      }
    }

##### POST Response

    {
      "data": {
        "angebotAnnehmen": {
          "jobId": "AB12345678"
        }
      },
      "errors": []
    }

### Query Annahme-Job

#### Hinweise

* Wenn sich das Angebot im Annahmeprozess als nicht machbar herausstellt, wird in der Schnittstelle kein Antrag zurückgegeben.

#### Request

Die GraphQL-Query heißt `annahmeJob` und hat folgende Parameter:

| Parametername      | Typ       | Default          | Kommentar                                                  |
|--------------------|-----------|------------------|------------------------------------------------------------|
| jobId     | String!   | - (Pflichtfeld)  |   Die ID des Jobs, die bei der Initiierung der Annahme zurückgegeben wurde |

#### Response

Diese Query liefert als Rückgabewert einen [AnnahmeJob](#annahmejob). Enthalten sind der Status der Annahme und bei erfolgreicher Annahme der Antrag.

#### Beispiel

##### POST Request

    POST https://kex-angebote.ratenkredit.api.europace.de/angebote
    Authorization: Bearer xxxx
    Content-Type: application/json

    {
      "query": "query annahmeJob($jobId: String!) {  
        annahmeJob(jobId: $jobId) {
          status
          antrag {
            antragsnummer
            gesamtkonditionen {
              sollzins
              effektivzins
              laufzeitInMonaten
              gesamtkreditbetrag
              nettokreditbetrag
              rateMonatlich
            }
          }
        }
      }",
      "variables": {
        "jobId": "$jobId"
      }
    }

##### POST Response

    {
      "data": {
        "angebotAnnehmen": {
          "status": "SUCCESS",
          "antrag": {
            "antragsnummer": "ABC123/1/1",
            "gesamtkonditionen": {
              "sollzins": 2.95,
              "effektivzins": 2.99,
              "laufzeitInMonaten": 60,
              "gesamtkreditbetrag": 10762.19,
              "nettokreditbetrag": 10000,
              "rateMonatlich": 179.47
            }
          }
        }
      },
      "errors": []
    }

## Request-Datentypen

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

## Response-Datentypen

Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen.  
Es gibt die Scalare `Euro` und `Prozent`, die jeweils Wrapper für BigDecimal sind.

    
### Schaufensterkondition

    {
        gesamtkonditionen: Gesamtkonditionen
        ratenkredit: Ratenkredit
    }
    
### Angebot

    {
        id: String!
        gesamtkonditionen: AngebotGesamtkonditionen
        ratenkredit: Ratenkredit
    }

#### AngebotGesamtkonditionen

    {
        effektivzins: Prozent,
        gesamtkreditbetrag: Euro
        laufzeitInMonaten: Int
        nettokreditbetrag: Euro
        rateMonatlich: Euro
        sollzins: Prozent
        konditionsspanne: Konditionsspanne
    }
    
##### Konditionsspanne

    {
        minimum: Konditionsgrenze
        maximum: Konditionsgrenze
    }

###### Konditionsgrenze

    {
        sollzins: Prozent
        effektivzins: Prozent
        rateMonatlich: Euro
        gesamtkreditbetrag: Euro
    }

#### AngebotRatenkredit

    {
        produktanbieter: Produktanbieter
        produktbezeichnung: String
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

### AnnahmeJob

    {
        status: JobStatus!
        antrag: Antrag
    }

#### JobStatus

    "FAILURE" | "PENDING" | "SUCCESS"


#### Antrag

    {
        antragsnummer: String!
        gesamtkonditionen: AntragGesamtkonditionen
        ratenkredit: AntragRatenkredit
        ratenschutz: AntragRatenschutz
        dokumente: [Dokument!]
        identifikationAntragsteller1: Identifikation
        identifikationAntragsteller2: Identifikation
    }

##### AntragGesamtkonditionen

    {
        effektivzins: Prozent
        gesamtkreditbetrag: Euro
        laufzeitInMonaten: Int
        nettokreditbetrag: Euro
        rateMonatlich: Euro
        sollzins: Prozent
    }
    
##### AntragRatenkredit

    {
        schlussrate: Euro
    }

##### AntragRatenschutz

    {
        praemieMonatlich: Euro
        versicherteRisikenAntragsteller1: [VersichertesRisiko!]!
        versicherteRisikenAntragsteller2: [VersichertesRisiko!]! 
    }

##### VersichertesRisiko

    {
          ARBEITSLOSIGKEIT
          ARBEITSUNFAEHIGKEIT
          LEBEN
    }

##### Dokument

    {
        url: String
    }

##### Identifikation

    {
        antragstellername: String 
        qesUrl: String
        referenznummer: String
        videolegitimationUrl: String
    }

Das Property `antragstellername` enthält den Namen im Format "<vorname> <nachname>".



## Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt
