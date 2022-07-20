# KEX-Angebote-API

> ⚠️ You'll find German domain-specific terms in the documentation, for translations and further explanations please refer to our [glossary](https://docs.api.europace.de/common/glossary/)

## General

These APIs allow the calculation of Schaufensterkonditionen as well as calculating and accepting Angebote based on a Vorgang.
All APIs documented here are [GraphQL-APIs](https://docs.api.europace.de/privatkredit/graphql/).

> ⚠️ This API is continuously developed. Therefore we expect
> all users to align with the "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)", which requires clients to be
> tolerant towards compatible API changes when reading and processing the data. This means:
>
> 1. unknown properties must not result in errors
>
> 2. Strings with a restricted set of values (Enums) must support new unknown values
>
> 3. sensible usage of HTTP status codes, even if they are not explicitly documented
>

<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->

### Authentication

These APIs are secured by the OAuth 2.0 client credentials flow using the [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/).
To use these APIs your OAuth2-Client needs the following scopes:

| Scope                          | Label in Partnermanagement            | Description                                                |
|--------------------------------|---------------------------------------|------------------------------------------------------------|
| privatkredit:angebot:ermitteln | KreditSmart-Angebote ermitteln        | Scope for calculating Schaufensterkonditionen and Angebote |
| privatkredit:antrag:schreiben  | KreditSmart-Anträge anlegen/verändern | Scope for accepting an Angebot (creating an Antrag)        |

### Request

These APIs accept data with the content-type **application/json** with UTF-8 encoding.
The fields inside a block can be sent in any order.

The APIs support all common GraphQL formats. More information can be found at [https://graphql.org/learn/queries/](https://graphql.org/learn/queries/).

The body of a GraphQL request contains the field `query`, which includes the GraphQL query as a String. Parameters can be set directly in the query or defined as variables.
The variables can be sent in the `variables` field of the body as a key-value map. All our examples use variables.

    {
      "query": "...",
      "variables": { ... }
    }

### Error codes

One of the special features in GraphQL is that most errors are not reflected via HTTP error codes.
In many cases you receive a status code 200, even though an error has occurred. These GraphQL errors can be found in the `errors` field of the response body.
More information about error codes can be found [here](https://docs.api.europace.de/privatkredit/graphql/#error-handling).

### HTTP-Status Errors

| Error Code | Message               | Description                     |
|------------|-----------------------|---------------------------------|
| 401        | Unauthorized          | Authentication failed           |
| 403        | Forbidden             | The API client misses a scope   |
| 415        | Unsupported MediaType | The wrong content type was used |

#### GraphQL Errors

| Error Code | Message     | Description                                                                                     |
|------------|-------------|-------------------------------------------------------------------------------------------------|
| 400        | Bad Request | Request format is invalid (mandatory fields are missing, wrong parameter names or values, ...) |
| 403        | Forbidden   | The authenticated user does not have sufficient rights                                          |
| 409        | Conflict    | The Vorgang was updated after the offer was calculated and before the offer was accepted        |

## Schaufensterkonditionen

Schaufensterkonditionen, both the top Schaufensterkondition and a complete list, can be retrieved via our GraphQL API using **HTTP POST**.

The URL for the Schaufensterkonditionen is:

    https://kex-angebote.ratenkredit.api.europace.de/schaufenster

### Query Top-Schaufensterkondition

#### Request

The GraphQL-Query is called `topSchaufensterkondition` and contains the following parameter:

| Parameter Name     | Type                                      | Default Value                                     |
|--------------------|-------------------------------------------|---------------------------------------------------|
| partnerId          | [Partner-ID](#partner-id)                 | The Partner-ID of the API Client                  |
| auszahlungsbetrag  | Euro!                                     | - (mandatory field)                               | 
| laufzeitInMonaten  | Int                                       | -                                                 | 
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck) | Calculation over all values of Finanzierungszweck |
| datenkontext       | [Datenkontext](#datenkontext)             | `TESTUMGEBUNG`                                    |

#### Response

The Query returns a [Schaufensterkondition](#schaufensterkondition).

#### Example

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

The GraphQL-Query is called `schaufensterkonditionen` and has the following parameter:

| Parameter Name     | Type                                      | Default Value                    |
|--------------------|-------------------------------------------|----------------------------------|
| partnerId          | [Partner-ID](#partner-id)                 | The Partner-ID of the API Client |
| auszahlungsbetrag  | Euro!                                     | - (mandatory field)              |
| laufzeitInMonaten  | Int                                       | -                                |
| finanzierungszweck | [Finanzierungszweck](#finanzierungszweck) | `FREIE_VERWENDUNG`               |
| datenkontext       | [Datenkontext](#datenkontext)             | `TESTUMGEBUNG`                   |

#### Response

The Query returns a list of [Schaufensterkonditionen](#schaufensterkondition).

#### Example

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

You can calculate a list of machbare and vollständige Angebote using data of a Vorgang via our GraphQL-API using **HTTP POST**.
Using the id of the corresponding Angebot you can accept it.

The URL for calculating and accepting Angebote is:

    https://kex-angebote.ratenkredit.api.europace.de/angebote  

### Query Angebote

#### Hints

* The calculation of Angebote is only possible if the corresponding Vorgang includes a valid **Kreditbetrag** and **Laufzeit or Rate**. Should that not be the case the API user receives
  a [GraphQL-Error](#graphql-errors) with the status code `400`. The Vorgang has to be corrected before you can continue.

#### Request

The GraphQL-Query is called `angebote` and has the following parameters.
If only the vorgangsnummer is provided, only complete offers will be returned.

| Parameter Name | Type           | Default Value                                                               |
|----------------|----------------|-----------------------------------------------------------------------------|
| vorgangsnummer | String!        | - (mandatory field)                                                         |
| options        | AngebotOptions | `{ includeVollstaendigkeitsstatus: [VOLLSTAENDIG], vertriebskanal: B2B2C }` |

#### Response

The Query returns a list of [Angebote](#angebot).

#### Example

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

#### Hints

* Currently the API only supports **Fernabsatzgeschäft**.
* The vertriebskanal that was used to query the offers will be used when an offer is accepted, if no vertriebskanal was provided the default value **B2B2C** is used.
* When accepting an offer the authenticated user needs to have a Handelsbeziehung for the corresponding bank, which allows the acceptance. Otherwise the user receives
  a [GraphQL-Error](#graphql-errors) with the status code `403`.
* Accepting an offer is only possible if the Vorgang did not change in the meantime. Should there be updates between the calculation of offers and accepting an offer the user receives
  a [GraphQL-Error](#graphql-errors) with the status code `409`. In this case the calculation of the offers needs to be done again.
* To optimize the process we might calculate alternative offers.
* The Kundenbetreuer of a Vorgang is important for accepting an offer as the name and contact details will be sent to the bank.

#### Request

The GraphQL-Mutation is called `angebotAnnehmen`and has the following parameter:

| Parameter Name | Type           | Default Value                 | Comment                                               |
|----------------|----------------|-------------------------------|-------------------------------------------------------|
| vorgangsnummer | String!        | - (mandatory field)           |                                                       |
| angebotId      | String!        | - (mandatory field)           | The ID of the calculated offer that is to be accepted |

#### Response

This Mutation returns a `jobId`.

#### Example

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

#### Hints

* If the offer cannot be accepted from the bank, the response will contain `status=SUCCESS` but not contain any Antrag.
* A partner can only query jobs that were created by this partner otherwise a 403 FORBIDDEN is returned.
* Only complete offers can be accepted, if an incomplete offer is requested, it will result in a 400 BAD REQUEST

#### Request

The GraphQL-Query is called `annahmeJob` and has the following parameter:

| Parameter Name | Type    | Default Value       | Comment                                                                      |
|----------------|---------|---------------------|------------------------------------------------------------------------------|
| jobId          | String! | - (mandatory field) | The ID of the job, which was returned when initiating the acceptance process |

#### Response

This Query returns an [AnnahmeJob](#annahmejob). It contains the Status of the acceptance process and the Antrag if the process was successful.

#### Example

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

## Request Datatypes

### Partner-ID

This type is a 5-digit String and identifies a Plakette of the Europace-Partnermanagement.  
The Partner-ID has to be underneath or identical to the Partner-ID of the API-Client.

### Datenkontext

This type is a String which can currently have one of the following values

* TESTUMGEBUNG
* ECHTGESCHAEFT

### Finanzierungszweck

This type is a String which can currently have one of the following values

* UMSCHULDUNG
* FREIE_VERWENDUNG
* FAHRZEUGKAUF
* MODERNISIEREN

## Response Datatypes

For better readability, the overall format is broken down into *types* that are defined separately but should be used at the corresponding positions.
The attributes within a block can be specified in any order. There are the scalars `Euro` and `Prozent`, which are wrappers for `BigDecimal`.

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
        sofortkredit: Boolean
        vollstaendigkeit: Vollstaendigkeit
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

The field `svg` contains the URL of the svg and not the content.

### AngebotOptions

    {
        includeVollstaendigkeitsstatus: [VollstaendigkeitStatus]
        vertriebskanal: Vertriebskanal
    }

#### Vertriebskanal

    {
        B2B2C
        B2B
    }

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

#### Vollstaendigkeit

    {
        messages: [VollstaendigkeitMessage]
        status: VollstaendigkeitStatus
    }

##### VollstaendigkeitMessage

    {
        category: VollstaendigkeitCategory
        property: String
        reason: VollstaendigkeitReason
        text: String
        type: VollstaendigkeitType
    }

###### VollstaendigkeitCategory

    {
        AUSGABEN
        BESCHAEFTIGUNG
        EINNAHMEN
        FINANZBEDARF
        HERKUNFT
        IMMOBILIEN
        KINDER
        PERSONENDATEN
        SOFORTKREDIT
        VERBINDLICHKEITEN
        WOHNSITUATION
    }

###### VollstaendigkeitReason

    {
        VALUE_IS_EMPTY
        VALUE_IS_INVALID
        VALUE_OUT_OF_RANGE
    }

###### VollstaendigkeitType

    {
        ANTRAGSTELLER
        VERMITTLER
    }

#### VollstaendigkeitStatus

    {
        UNVOLLSTAENDIG
        VOLLSTAENDIG
    }

The field `antragstellername` contains the name in the format "\<first name\> \<last name\>".

## Terms of use

The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/terms/).
