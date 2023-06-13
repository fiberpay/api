---
Wspomaganie działań przeciwdziałania praniu pieniędzy i finansowania terroryzmu
---

# SystemAML

###

## Spis treści:

- 1. [Informacje ogólne](#informacje-ogólne)
  - 1.1. [Klucze API](#klucze-api)
  - 1.2. [Nagłówek zapytania](#nagłówek-zapytania)
  - 1.3. [Ciało zapytania](#ciało-zapytania)
- 2. [Opis usług](#opis-usług)
  - 2.1. [POST /parties](#post-parties)
  - 2.2. [GET /parties](#get-parties)
  - 2.3. [GET /parties/{code}](#get-partiescode)
  - 2.4. [GET /parties/pdf/{code}](#get-partiespdfcode)
  - 2.5. [DELETE /parties/{code}](#delete-partiescode)
  - 2.6. [POST /parties/{code}/beneficiaries](#post-partiescodebeneficiaries)
  - 2.7. [GET /parties/{code}/beneficiaries](#get-partiescodebeneficiaries)
  - 2.8. [DELETE /beneficiaries/{code}](#delete-beneficiariescode)
  - 2.9. [POST /parties/{code}/boardmembers](#post-partiescodeboardmembers)
  - 2.10. [GET /parties/{code}/boardmembers](#get-partiescodeboardmembers)
  - 2.11. [DELETE /boardmembers/{code}](#delete-boardmemberscode)
  - 2.12. [POST /transactions](#post-transactions)
  - 2.13. [GET /transactions](#get-transactions)
  - 2.14. [GET /transactions/{code}](#get-transactionscode)
  - 2.15. [GET /transactions/pdf/{code}](#get-transactionspdfcode)
  - 2.16. [DELETE /transactions/{code}](#delete-transactionscode)
  - 2.17. [POST /events](#post-events)
  - 2.18. [GET /events](#get-events)
  - 2.19. [GET /events/{code}](#get-eventscode)
  - 2.20. [DELETE /events/{code}](#delete-eventscode)
  - 2.21. [POST /comments](#post-comments)
  - 2.22. [DELETE /comments/{code}](#delete-commentscode)
  - 2.23. [GET /alerts](#get-alerts)
  - 2.24. [GET /alerts/{code}](#get-alertscode)
  - 2.25. [DELETE /alerts/{code}]($delete-alertscode)
  - 2.26. [POST /tasks](#post-tasks)
  - 2.27. [GET /tasks](#get-tasks)
  - 2.28. [GET /tasks/{code}](#get-taskscode)
  - 2.29. [DELETE /tasks/{code}](#delete-taskscode)
#

###

## Informacje ogólne

Głównym zastosowaniem API jest katalogowanie podmiotów, przez działalności od których wymagane jest prowadzenie ewidencji klientów.

### Adresy serwerów

| Funkcjonalność | Środowisko | URL                                                                 |
| -------- | -------- | --------------------------------------------------------------------------- |
| Serwer API | Produkcyjne   | https://api.systemaml.pl/1.0/ |
| Frontend aplikacji | Produkcyjne   | https://systemaml.pl/ |
| Serwer API | Testowe   | https://amlapitest.fiberpay.pl/1.0/ |
| Frontend aplikacji | Testowe   | https://test.systemaml.pl/ |

### UWAGA!

Są to zupełnie rozdzielone środowiska (łącznie z infrastrukturą). Klucze API z jednego środowiska nie będą działać w drugim.

### Klucze API

Do korzystania z API konieczne jest wygenerowanie kluczy:

- jawnego (apiKey) - używanego do przesyłania w ramach żądań API
- tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

W celu uzyskania danych dostępowych niezbędnych do poprawnego korzystania z API należy wygenerować klucze na frontendzie aplikacji lub skontaktować się bezpośrednio z usługodawcą (info@fiberpay.pl).

W przypadku niepoprawnego wykorzystania kluczy dostępowych serwer zwraca następujące błędy:

**STATUS 401 Unathorized**

```json
{
  "status": "ERROR",
  "error": "No Api Key provided"
}
```

```json
{
  "status": "ERROR",
  "error": "Api key invalid"
}
```

```json
{
  "status": "ERROR",
  "error": "Wrong signature"
}
```

### Nagłówek zapytania

Każde żądanie do API powinno posiadać następujący nagłówek:

- Api-Key – wygenerowany klucz jawny

W przypadku zapytań nie posiadających body, należy wysłać żądanie z nagłówkiem Authorization o wartości "Bearer {token}" (token - pusty string w postaci JWT z odpowiednią sygnaturą).

### Ciało zapytania

Każde ciało zapytania jest przekazywane za pomocą JWT z wykorzystaniem odpowiedniej sygnatury. Body żądania powinno być tekstem (JWT).

## Opis usług

### POST /parties

Utworzenie nowego podmiotu. Parametry żądania:

| Parametr | Wymagane | Opis                                                                        |
| -------- | -------- | --------------------------------------------------------------------------- |
| **type** | TAK      | Typ podmiotu. Aktualnie wspierane: individual, sole_proprietorship, company |
| **status** | TAK    | Status podmiotu. Aktualnie wspierane: draft, active, inactive, in_acceptance |

W zależności od wybranego typu wymagane są następujące parametry:

a) individual:

| Parametr                   | Wymagane | Opis                                                                                            |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| **firstName**              | NIE      | Imie podmiotu                                                                                   |
| **lastName**               | NIE      | Nazwisko podmiotu                                                                               |
| **personalIdentityNumber** | TAK      | Numer PESEL podmiotu (w przypadku braku numeru pesel wymagany jest parametr personalIdentifier) |
| **personalIdentifier**     | NIE      | Numer identifykacyjny podmiotu (wymagany jeśli nie ma numeru pesel)                             |
| **birthDate**              | NIE      | Data urodzenia (wymagana jeśli nie ma numeru pesel)                                             |
| **birthCountry**           | NIE      | Kraj urodzenia (wymagany jeśli nie ma numeru pesel)                                             |
| **citizenship**            | NIE      | Obywatelstwo (kod kraju standardzie ISO)                                                        |
| **birthCity**              | NIE      | Miasto urodzenia                                                                                |
| **documentType**           | TAK      | Rodzaj dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                  |
| **documentNumber**         | TAK      | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                   |
| **documentExpirationDate** | NIE      | Termin ważnosci dokumentu                                                                       |
| **withoutExpirationDate**  | NIE      | Informacja czy dokument posiada datę ważności (bool)                                            |
| **references**             | NIE      | Referencje własne                                                                               |
| **politicallyExposed**     | NIE      | Informacja czy podmiot jest eksponowany politycznie (bool)                                      |
| **politicallyExposedFamily**     | NIE      | Informacja czy podmiot jest rodziną osoby eksponowanej politycznie (bool)                 |
| **politicallyExposedCoworker**     | NIE      | Informacja czy podmiot jest bliskim współpracownikiem osoby eksponowanej politycznie (bool)|

b) sole_proprietorship - wszystkie powyższe oraz:

| Parametr                           | Wymagane | Opis                                                                |
| ---------------------------------- | -------- | ------------------------------------------------------------------- |
| **taxIdNumber**                    | TAK      | NIP prowadzonej działalności                                        |
| **registrationCountry**            | NIE      | Kraj rejestracji podmiotu (wymagany jeśli nie ma numeru NIP)        |
| **companyIdentifier**              | NIE      | Numer identyfikujący (wymagany jeśli nie ma numeru NIP)             |
| **references**                     | NIE      | Referencje własne                                                   |
| **nationalBusinessRegistryNumber** | NIE      | Regon prowadzonej działalności                                      |
| **companyName**                    | NIE      | Nazwa prowadzonej działalności                                      |
| **mainPkdCode**                    | TAK      | Obiekt z przeważającym kodem PKD (nie jest wymagany gdy nie ma NIP) |
| **pkdCodes**                       | NIE      | Tablica z pozostałymi kodami PKD (tablica zawierająca obiekty jw.)  |

Struktura obiektu z kodem PKD:

| Parametr    | Wymagane | Opis                                |
| ----------- | -------- | ----------------------------------- |
| **pkdCode** | TAK      | Numer kodu PKD w formacie (00.00.X) |
| **pkdName** | NIE      | Opis kodu PKD                       |

c) company:

| Parametr                           | Wymagane | Opis                                                                         |
| ---------------------------------- | -------- | ---------------------------------------------------------------------------- |
| **taxIdNumber**                    | TAK      | Numer NIP                                                                    |
| **companyIdentifier**              | NIE      | Numer identyfikujący (wymagany jeśli nie ma numeru NIP)                      |
| **registrationCountry**            | NIE      | Kraj rejestracji podmiotu (podawany jeśli nie ma numeru NIP)                 |
| **companyName**                    | NIE      | Nazwa firmy                                                                  |
| **tradeName**                      | NIE      | Nazwa handlowa firmy                                                         |
| **references**                     | NIE      | Referencje własne                                                            |
| **nationalBusinessRegistryNumber** | NIE      | Numer Regon                                                                  |
| **nationalCourtRegistryNumber**    | NIE      | Numer KRS                                                                    |
| **businessActivityForm**           | TAK      | Rodzaj prowadzonej działalności (Nie jest wymagane jeśli nie ma numeru NIP). |
| **website**                        | NIE      | Strona internetowa                                                           |
| **servicesDescription**            | NIE      | Opis usług                                                                   |
| **mainPkdCode**                    | TAK      | Obiekt z przeważającym kodem PKD (nie jest wymagany gdy nie ma NIP)          |
| **pkdCodes**                       | NIE      | Tablica z pozostałymi kodami PKD (tablica zawierająca obiekty jw.)           |
| **beneficiaries**                  | NIE      | Tablica obiektów z danymi beneficjentów                                      |
| **boardMembers**                   | NIE      | Tablica obiektów z danymi członków zarządu                                   |

Struktura obiektu beneficjenta:

| Parametr                     | Wymagane | Opis                                                                    |
| ---------------------------- | -------- | ----------------------------------------------------------------------- |
| **firstName**                | NIE      | Imię beneficjenta                                                       |
| **lastName**                 | NIE      | Nazwisko beneficjenta                                                   |
| **personalIdentityNumber**   | TAK      | Numer PESEL beneficjenta (w przypadku braku numeru pesel wymagany jest  |
| parametr personalIdentifier) |
| **documentType**             | TAK      | Rodzaj dokumentu (nie jest wymagany jeśli nie ma numeru pesel)          |
| **documentNumber**           | TAK      | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)           |
| **documentExpirationDate**   | NIE      | Termin ważnosci dokumentu                                               |
| **personalIdentifier**       | NIE      | Numer identifykacyjny beneficjenta (wymagany jeśli nie ma numeru pesel) |
| **birthDate**                | NIE      | Data urodzenia (wymagana jeśli nie ma numeru pesel)                     |
| **birthCountry**             | NIE      | Kraj urodzenia (wymagany jeśli nie ma numeru pesel)                     |
| **citizenship**              | NIE      | Obywatelstwo (kod kraju standardzie ISO)                                |
| **birthCity**                | NIE      | Miasto urodzenia                                                        |
| **withoutExpirationDate**    | NIE      | Informacja czy dokument posiada datę ważności (bool)                    |
| **references**               | NIE      | Referencje własne                                                       |
| **politicallyExposed**       | NIE      | Informacja czy beneficjent jest eksponowany politycznie (bool)          |
| **politicallyExposedFamily**     | NIE      | Informacja czy beneficjent jest rodziną osoby eksponowanej politycznie (bool)                 |
| **politicallyExposedCoworker**     | NIE      | Informacja czy beneficjent jest bliskim współpracownikiem osoby eksponowanej politycznie (bool)|
| **ownedSharesAmount**              | TAK      | Liczba posiadanych udziałów                                          |
| **ownedSharesUnit**              | TAK      | Jednostka posiadanych udziałów ('%' lub 'PLN')                                 |
| **description**              | NIE      | Opis beneficjenta                                                       |

Struktura obiektu członka zarządu:

| Parametr                     | Wymagane | Opis                                                                       |
| ---------------------------- | -------- | -------------------------------------------------------------------------- |
| **firstName**                | NIE      | Imię członka zarządu                                                       |
| **lastName**                 | NIE      | Nazwisko członka zarządu                                                   |
| **personalIdentityNumber**   | TAK      | Numer PESEL członka zarządu (w przypadku braku numeru pesel wymagany jest  |
| parametr personalIdentifier) |
| **documentType**             | TAK      | Rodzaj dokumentu (nie jest wymagany jeśli nie ma numeru pesel)             |
| **documentNumber**           | TAK      | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)              |
| **documentExpirationDate**   | NIE      | Termin ważnosci dokumentu                                                  |
| **personalIdentifier**       | NIE      | Numer identifykacyjny członka zarządu (wymagany jeśli nie ma numeru pesel) |
| **birthDate**                | NIE      | Data urodzenia (wymagana jeśli nie ma numeru pesel)                        |
| **birthCountry**             | NIE      | Kraj urodzenia (wymagany jeśli nie ma numeru pesel)                        |
| **citizenship**              | NIE      | Obywatelstwo (kod kraju standardzie ISO)                                   |
| **birthCity**                | NIE      | Miasto urodzenia                                                           |
| **withoutExpirationDate**    | NIE      | Informacja czy dokument posiada datę ważności (bool)                       |
| **references**               | NIE      | Referencje własne                                                          |
| **politicallyExposed**       | NIE      | Informacja czy członek zarządu jest eksponowany politycznie (bool)             |
| **politicallyExposedFamily**     | NIE      | Informacja czy członek zarządu jest rodziną osoby eksponowanej politycznie (bool)                 |
| **politicallyExposedCoworker**     | NIE      | Informacja czy członek zarządu jest bliskim współpracownikiem osoby eksponowanej politycznie (bool)|
| **description**              | NIE      | Opis członka zarządu                                                       |

Do każdego z typów podmiotu można dodać dane kontaktowe.

| Parametr                 | Wymagane | Opis                                              |
| ------------------------ | -------- | ------------------------------------------------- |
| **accommodationAddress** | NIE      | Obiekt zawierający adres zamieszkania             |
| **forwardAddress**       | NIE      | Obiekt zawierający adres korespondencyjny         |
| **businessAddress**      | NIE      | Obiekt zawierający adres prowadzenia działalności |
| **personalContact**      | NIE      | Obiekt zawierający dane kontaktowe                |
| **companyContact**       | NIE      | Obiekt zawierający dane kontaktowe działalności   |

Struktura obiektów:&#x20;

a) adres:

| Parametr        | Wymagane | Opis                              |
| --------------- | -------- | --------------------------------- |
| **country**     | NIE      | Nazwa kraju (kod standardzie ISO) |
| **city**        | NIE      | Miasto                            |
| **street**      | NIE      | Ulica                             |
| **houseNumber** | NIE      | Numer domu                        |
| **flatNumber**  | NIE      | Numer mieszkania                  |
| **postalCode**  | NIE      | Kod pocztowy                      |

a) kontakt:

| Parametr         | Wymagane | Opis                   |
| ---------------- | -------- | ---------------------- |
| **emailAdress**  | NIE      | Adres email            |
| **phoneCountry** | NIE      | Prefix numeru telefonu |
| **phoneNumber**  | NIE      | Numer telefonu         |

#### Przykładowe dane do utworzenia podmiotu typu 'individual':

```json
{
  "birthCity": "Warszawa",
  "birthCountry": "PL",
  "citizenship": "PL",
  "createdByName": "Wojtek",
  "documentExpirationDate": "2025-05-15",
  "documentNumber": "aze123123",
  "documentType": "id_card",
  "firstName": "Jan",
  "lastName": "Kowalski",
  "personalIdentityNumber": "09271573233",
  "politicallyExposed": false,
  "references": "qwerty",
  "status": "active",
  "type": "individual",
  "withoutExpirationDate": false,
  "accommodationAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "forwardAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "personalContact": {
    "emailAdress": "info@fiberpay.pl",
    "phoneCountry": "48",
    "phoneNumber": "123123123"
  }
}
```

#### Przykładowe dane do utworzenia podmiotu typu 'sole_proprietorship':

```json
{
  "birthCity": "Warszawa",
  "nationalBusinessRegistryNumber": "123456789",
  "birthCountry": "PL",
  "citizenship": "PL",
  "companyName": "Usługi programistyczne",
  "createdByName": "Adam",
  "documentExpirationDate": "2025-05-08",
  "documentNumber": "aze123123",
  "documentType": "passport",
  "firstName": "Jan",
  "lastName": "Kowalski",
  "mainPkdCode": {
    "pkdCode": "01.12.Z",
    "pkdName": "Uprawa ryżu"
  },
  "personalIdentityNumber": "99120234518",
  "pkdCodes": [
    {
      "pkdCode": "01.15.Z",
      "pkdName": "Uprawa tytoniu"
    }
  ],
  "politicallyExposed": false,
  "references": "qwerty",
  "status": "active",
  "taxIdNumber": "3765151981",
  "type": "sole_proprietorship",
  "withoutExpirationDate": false,
  "forwardAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "businessAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "accommodationAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "personalContact": {
    "emailAdress": "info@fiberpay.pl",
    "phoneCountry": "48",
    "phoneNumber": "123123123"
  },
  "companyContact": {
    "emailAdress": "info@fiberpay.pl",
    "phoneCountry": "48",
    "phoneNumber": "123123123"
  }
}
```

#### Przykładowe dane do utworzenia podmiotu typu 'company':

```json
{
  "type": "company",
  "companyName": "FiberPay",
  "taxIdNumber": "7010634566",
  "nationalBusinessRegistryNumber": "147302566",
  "tradeName": "FiberPay",
  "nationalCourtRegistryNumber": "0000512707",
  "businessActivityForm": "stock_company",
  "website": "fiberpay.pl",
  "references": "qwerty",
  "status": "active",
  "businessAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  },
  "companyContact": {
    "emailAdress": "info@fiberpay.pl",
    "phoneCountry": "48",
    "phoneNumber": "123123123"
  },
  "mainPkdCode": {
    "pkdCode": "64.99.Z",
    "pkdName": "POZOSTAŁA FINANSOWA DZIAŁALNOŚĆ USŁUGOWA, GDZIE INDZIEJ NIESKLASYFIKOWANA, Z WYŁĄCZENIEM UBEZPIECZEŃ I FUNDUSZÓW EMERYTALNYCH"
  },
  "pkdCodes": [
    {
      "pkdCode": "58.29.Z",
      "pkdName": "DZIAŁALNOŚĆ WYDAWNICZA W ZAKRESIE POZOSTAŁEGO OPROGRAMOWANIA"
    },
    {
      "pkdCode": "62.01.Z",
      "pkdName": "DZIAŁALNOŚĆ ZWIĄZANA Z OPROGRAMOWANIEM"
    }
  ],
  "beneficiaries": [
    {
      "firstName": "Jan",
      "lastName": "Kowalski",
      "personalIdentityNumber": "64091098920",
      "documentType": "id_card",
      "documentNumber": "aze123123",
      "citizenship": "PL",
      "birthCity": "Warszawa",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "withoutExpirationDate": false,
      "ownedSharesAmount": "45",
      "ownedSharesUnit": "%",
      "description": ""
    },
    {
      "firstName": "Adam",
      "lastName": "Nowak",
      "documentType": "id_card",
      "documentNumber": "aze123123",
      "citizenship": "PL",
      "birthCity": "Warszawa",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "withoutExpirationDate": false,
      "birthDate": "2001-01-01",
      "ownedSharesAmount": "50",
      "ownedSharesUnit": "%",
      "description": ""
    }
  ],
  "boardMembers": [
    {
      "firstName": "Jan",
      "lastName": "Kowalski",
      "personalIdentityNumber": "31111161119",
      "documentType": "id_card",
      "documentNumber": "aze123123",
      "citizenship": "PL",
      "birthCity": "Warszawa",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "withoutExpirationDate": false,
      "description": "Prezes"
    },
    {
      "firstName": "Adam",
      "lastName": "Nowak",
      "documentType": "id_card",
      "documentNumber": "aze123123",
      "citizenship": "PL",
      "birthCity": "Warszawa",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "withoutExpirationDate": false,
      "birthDate": "2001-01-01",
      "description": "Wiceprezes"
    }
  ]
}
```

#### Przykładowa odpowiedź serwera:
- **STATUS 201 CREATED**

```json
{
    "data": {
        "code": "93sc6hgq4jvk",
        "type": "company",
        "status": "active",
        "riskStatus": null,
        "riskExplanation": "",
        "riskStatusChangedBy": "",
        "createdByName": null,
        "references": "qwerty",
        "entity": {
            "code": "z7tbqvf6ed4g",
            "companyName": "FiberPay",
            "tradeName": "FiberPay",
            "taxIdNumber": "7010634566",
            "nationalBusinessRegistryNumber": "147302566",
            "nationalCourtRegistryNumber": "0000512707",
            "businessActivityForm": "stock_company",
            "industry": null,
            "servicesDescription": null,
            "website": "fiberpay.pl",
            "createdAt": "2023-06-12T15:12:02.000000Z",
            "registrationCountry": null,
            "companyIdentifier": null,
            "pkdCodes": [
                {
                    "code": "ym78kp3asq9f",
                    "pkdCode": "58.29.Z",
                    "pkdName": "DZIAŁALNOŚĆ WYDAWNICZA W ZAKRESIE POZOSTAŁEGO OPROGRAMOWANIA",
                    "mainPkd": false
                },
                {
                    "code": "1tqhkfn2erd5",
                    "pkdCode": "62.01.Z",
                    "pkdName": "DZIAŁALNOŚĆ ZWIĄZANA Z OPROGRAMOWANIEM",
                    "mainPkd": false
                }
            ],
            "mainPkd": {
                "code": "pwn9e5hfm1jc",
                "pkdCode": "64.99.Z",
                "pkdName": "POZOSTAŁA FINANSOWA DZIAŁALNOŚĆ USŁUGOWA, GDZIE INDZIEJ NIESKLASYFIKOWANA, Z WYŁĄCZENIEM UBEZPIECZEŃ I FUNDUSZÓW EMERYTALNYCH",
                "mainPkd": true
            }
        },
        "addresses": [
            {
                "code": "jznt4rpyqskc",
                "type": "business_address",
                "country": "PL",
                "city": "Warszawa",
                "street": "Grzybowska",
                "houseNumber": "4",
                "flatNumber": "106",
                "postalCode": "00-131",
                "createdAt": "2023-06-12T15:12:02.000000Z"
            }
        ],
        "contacts": [
            {
                "code": "ycx4qjhn9pg5",
                "type": "company",
                "emailAdress": "info@fiberpay.pl",
                "phoneCountry": "48",
                "phoneNumber": "123123123",
                "createdAt": "2023-06-12T15:12:02.000000Z"
            }
        ],
        "creationIp": null,
        "creationUserAgent": null
    }
}
```

### GET /parties

Zwraca podmioty utworzone przez użytkownika.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "v7udgqy1rh8e",
      "type": "individual",
      "firstName": "Jan",
      "lastName": "Kowalski",
      "companyName": null,
      "personalIdentityNumber": "01234567890",
      "taxIdNumber": null
    },
    {
      "code": "3g9mkv5nz48w",
      "type": "sole_proprietorship",
      "firstName": "Jan",
      "lastName": "Kowalski",
      "companyName": "FiberPay",
      "personalIdentityNumber": "01234567890",
      "taxIdNumber": "7010634566"
    },
    {
      "code": "a1bwfup8rmx5",
      "type": "company",
      "firstName": null,
      "lastName": null,
      "companyName": "FiberPay",
      "personalIdentityNumber": null,
      "taxIdNumber": "7010634560"
    }
  ]
}
```

W przypadku gdy użytkownik nie posiada utworzonych podmiotów, serwer zwraca pustą tablicę.

### GET /parties/{code}

Pobranie szczegółów danego podmiotu.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "xswyvgqa37zp",
    "type": "sole_proprietorship",
    "entity": {
      "code": "gq91wkstnae5",
      "firstName": "Jan",
      "lastName": "Kowalski",
      "personalIdentityNumber": "01234567880",
      "documentType": "id_card",
      "documentNumber": "AAA123456",
      "documentExpirationDate": "2022-10-15",
      "citizenship": "PL",
      "birthCity": "Warszawa",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "createdAt": "2022-06-01T14:52:12.000000Z",
      "soleProprietorship": {
        "code": "8p39ze4rsthc",
        "companyName": "Usługi programistyczne",
        "taxIdNumber": "7010634566",
        "nationalBusinessRegistryNumber": "365899489",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      }
    },
    "addresses": [
      {
        "code": "qvj18ftkacrs",
        "type": "forwarding_address",
        "country": "PL",
        "city": "Warszawa",
        "street": "Grzybowska",
        "houseNumber": "4",
        "flatNumber": "106",
        "postalCode": "00-131",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      },
      {
        "code": "a732sgfwx8e9",
        "type": "business_address",
        "country": "PL",
        "city": "Warszawa",
        "street": "Grzybowska",
        "houseNumber": "4",
        "flatNumber": "106",
        "postalCode": "00-131",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      },
      {
        "code": "jt1vwpg7ra69",
        "type": "accommodation_address",
        "country": "PL",
        "city": "Warszawa",
        "street": "Grzybowska",
        "houseNumber": "4",
        "flatNumber": "106",
        "postalCode": "00-131",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      }
    ],
    "contacts": [
      {
        "code": "3u16mygz58v4",
        "type": "personal",
        "email": "fiberpay@fiberpay.pl",
        "phoneCountry": "48",
        "phoneNumber": "123123123",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      },
      {
        "code": "4a9fz3nky2tq",
        "type": "company",
        "email": "fiberpay@fiberpay.pl",
        "phoneCountry": "48",
        "phoneNumber": "123123123",
        "createdAt": "2022-06-01T14:52:12.000000Z"
      }
    ]
  }
}
```

### GET /parties/pdf/{code}

Rozpoczyna pobieranie raportu pdf z podmiotu wskazanego kodem identyfikującym.

### DELETE /parties/{code}

Usunięcie podmiotu wskazanego kodem identyfikującym.


### POST /parties/{code}/beneficiaries

Dodanie beneficjenta rzeczywistego do podmiotu typu company. Parametry żądania:

| Parametr        | Wymagane | Opis                                                          |
| --------------- | -------- | ------------------------------------------------------------- |
| **ownedSharesAmount**              | TAK      | Liczba posiadanych udziałów                                          |
| **ownedSharesUnit**              | TAK      | Jednostka posiadanych udziałów ('%' lub 'PLN')                                 |
| **personalIdentityNumber** | TAK      | Numer PESEL beneficjenta (w przypadku braku numeru pesel wymagany jest parametr personalIdentifier)                       |
| **documentType**| TAK      | Rodzaj dokumentu (nie jest wymagany jeśli nie ma numeru pesel)|
| **documentNumber**| TAK    | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                                         |
| **description** | NIE      | Opis benficjenta                                              |
| **firstName**   | NIE      | Imię benficjenta                                              |
| **lastName**    | NIE      | Nazwisko beneficjenta                                         |
| **documentExpirationDate** | NIE      | Termin ważnosci dokumentu                          |
| **citizenship**            | NIE      | Obywatelstwo (kod kraju standardzie ISO)           |
| **birthCity**              | NIE      | Miasto urodzenia                                   |
| **birthDate**              | NIE      | Data urodzenia (wymagana jeśli nie ma numeru pesel)|
| **birthCountry**           | NIE      | Kraj urodzenia                                     |
| **politicallyExposed**     | NIE      | Informacja czy beneficjent jest eksponowany politycznie (bool)                       |
| **politicallyExposedFamily**     | NIE      | Informacja czy beneficjent jest rodziną osoby eksponowanej politycznie (bool)                 |
| **politicallyExposedCoworker**     | NIE      | Informacja czy beneficjent jest bliskim współpracownikiem osoby eksponowanej politycznie (bool)|
| **withoutExpirationDate**  | NIE      | Informacja czy dokument beneficjenta jest bezterminowy (bool)   |
#### Przykładowe dane do dodania beneficjenta rzeczywistego:

```json
{
  "description": "Pierwszy beneficjent",
  "birthCountry":  "AF",
  "citizenship": "AQ",
  "documentNumber": "ABC123",
  "documentType": "Dowód osobisty",
  "firstName": "Jan",
  "lastName":  "Kowalski",
  "ownedSharesAmount": "2",
  "ownedSharesUnit": "%",
  "personalIdentityNumber": "65122666817",
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "8b6kmdapy1nt",
    "ownedShares": 10,
    "beneficiary": {
      "individualEntity": {
        "code": "8pzqmn1g3r4k",
        "firstName": "Jan",
        "lastName": "Kowalski",
        "personalIdentityNumber": "01234567890",
        "documentType": "id_card",
        "documentNumber": "aze123456",
        "documentExpirationDate": "2022-10-15",
        "citizenship": "PL",
        "birthCity": "Warszawa",
        "birthCountry": "PL",
        "politicallyExposed": false,
        "createdAt": "2022-06-06T09:15:08.000000Z",
        "personalIdentifier": null
      }
    },
    "company": {
      "legalEntity": {
        "code": "2v8rjpf63a9g",
        "companyName": "FiberPay",
        "tradeName": "FiberPay",
        "taxIdNumber": "3210213293",
        "nationalBusinessRegistryNumber": "123456789",
        "nationalCourtRegistryNumber": "1234567890",
        "businessActivityForm": "limited_liability_company",
        "industry": "soccer",
        "servicesDescription": "Usługi programistyczne",
        "website": "www.fiberpay.pl",
        "createdAt": "2022-06-01T15:04:38.000000Z",
        "registrationCountry": null,
        "companyIdentifier": null
      },
      "addresses": [
        {
          "code": "xen4vgz63c8w",
          "type": "business_address",
          "country": "PL",
          "city": "Warszawa",
          "street": "Grzybowska",
          "houseNumber": "4",
          "flatNumber": "106",
          "postalCode": "00-131",
          "createdAt": "2022-06-01T15:04:38.000000Z"
        }
      ],
      "contacts": [
        {
          "code": "5hcu2j7z3wxg",
          "type": "company",
          "email": "fiberpay@fiberpay.pl",
          "phoneCountry": "48",
          "phoneNumber": "123123123",
          "createdAt": "2022-06-01T15:04:38.000000Z"
        }
      ]
    }
  }
}
```

### GET /parties/{code}/beneficiaries

Pobranie beneficjentów rzeczywistych wskazanego kodem podmiotu typu company.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "8b6kmdapy1nt",
      "ownedShares": "10",
      "individualEntity": {
        "code": "8pzqmn1g3r4k",
        "firstName": "Jan",
        "lastName": "Kowalski",
        "personalIdentityNumber": "01234567890",
        "documentType": "id_card",
        "documentNumber": "aze123456",
        "documentExpirationDate": "2022-10-15",
        "citizenship": "PL",
        "birthCity": "Warszawa",
        "birthCountry": "PL",
        "politicallyExposed": false,
        "createdAt": "2022-06-06T09:15:08.000000Z",
        "personalIdentifier": null
      }
    }
  ]
}
```

W przypadku gdy podmiot nie posiada dodanych beneficjentów rzeczywistych zwracana tablica jest pusta

### DELETE /beneficiaries/{code}

Usunięcie beneficjenta rzeczywistego wskazanego kodem identyfikującym.


### POST /parties/{code}/boardmembers

Dodanie członka zarządu do podmiotu typu osoba prawna (company). Parametry żądania:

| Parametr        | Wymagane | Opis                                                          |
| --------------- | -------- | ------------------------------------------------------------- |
| **personalIdentityNumber** | TAK      | Numer PESEL członka zarządu (w przypadku braku numeru pesel wymagany jest parametr personalIdentifier)                       |
| **documentType**| TAK      | Rodzaj dokumentu (nie jest wymagany jeśli nie ma numeru pesel)|
| **documentNumber**| TAK    | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                                         |
| **description** | NIE      | Opis benficjenta                                              |
| **firstName**   | NIE      | Imię benficjenta                                              |
| **lastName**    | NIE      | Nazwisko członka zarządu                                         |
| **documentExpirationDate** | NIE      | Termin ważnosci dokumentu                          |
| **citizenship**            | NIE      | Obywatelstwo (kod kraju standardzie ISO)           |
| **birthCity**              | NIE      | Miasto urodzenia                                   |
| **birthDate**              | NIE      | Data urodzenia (wymagana jeśli nie ma numeru pesel)|
| **birthCountry**           | NIE      | Kraj urodzenia                                     |
| **politicallyExposed**     | NIE      | Informacja czy członek zarządu jest eksponowany politycznie (bool)                       |
| **politicallyExposedFamily**     | NIE      | Informacja czy członek zarządu jest rodziną osoby eksponowanej politycznie (bool)                 |
| **politicallyExposedCoworker**     | NIE      | Informacja czy członek zarządu jest bliskim współpracownikiem osoby eksponowanej politycznie (bool)|
| **withoutExpirationDate**  | NIE      | Informacja czy dokument członka zarządu jest bezterminowy (bool)   |

#### Przykładowe dane do dodania członka zarządu:

```json
{
  "birthCountry":  "AF",
  "citizenship": "AQ",
  "documentNumber": "ABC123",
  "documentType": "Dowód osobisty",
  "firstName": "Jan",
  "lastName":  "Kowalski",
  "personalIdentityNumber": "65122666817",
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "8b6kmdapy1nt",
    "boardmember": {
      "individualEntity": {
        "code": "8pzqmn1g3r4k",
        "firstName": "Jan",
        "lastName": "Kowalski",
        "personalIdentityNumber": "01234567890",
        "documentType": "id_card",
        "documentNumber": "aze123456",
        "documentExpirationDate": "2022-10-15",
        "citizenship": "PL",
        "birthCity": "Warszawa",
        "birthCountry": "PL",
        "politicallyExposed": false,
        "createdAt": "2022-06-06T09:15:08.000000Z",
        "personalIdentifier": null
      }
    },
    "company": {
      "legalEntity": {
        "code": "2v8rjpf63a9g",
        "companyName": "FiberPay",
        "tradeName": "FiberPay",
        "taxIdNumber": "3210213293",
        "nationalBusinessRegistryNumber": "123456789",
        "nationalCourtRegistryNumber": "1234567890",
        "businessActivityForm": "limited_liability_company",
        "industry": "soccer",
        "servicesDescription": "Usługi programistyczne",
        "website": "www.fiberpay.pl",
        "createdAt": "2022-06-01T15:04:38.000000Z",
        "registrationCountry": null,
        "companyIdentifier": null
      },
      "addresses": [
        {
          "code": "xen4vgz63c8w",
          "type": "business_address",
          "country": "PL",
          "city": "Warszawa",
          "street": "Grzybowska",
          "houseNumber": "4",
          "flatNumber": "106",
          "postalCode": "00-131",
          "createdAt": "2022-06-01T15:04:38.000000Z"
        }
      ],
      "contacts": [
        {
          "code": "5hcu2j7z3wxg",
          "type": "company",
          "email": "fiberpay@fiberpay.pl",
          "phoneCountry": "48",
          "phoneNumber": "123123123",
          "createdAt": "2022-06-01T15:04:38.000000Z"
        }
      ]
    }
  }
}
```

### GET /parties/{code}/boardmembers

Pobranie beneficjentów rzeczywistych wskazanego kodem podmiotu typu company.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "t8svea4gnpu7",
      "description": null,
      "individualEntity": {
        "code": "5vghqraxtucn",
        "firstName": "Jan",
        "lastName": "Kowalski",
        "personalIdentityNumber": "84022598949",
        "documentType": "Paszport",
        "documentNumber": "aze123123",
        "documentExpirationDate": null,
        "withoutExpirationDate": false,
        "citizenship": null,
        "birthCity": null,
        "birthCountry": null,
        "politicallyExposed": false,
        "createdAt": "2023-04-24T09:20:06.000000Z",
        "birthDate": null
      }
    }
  ]
}
```
W przypadku gdy podmiot nie posiada dodanych członków zarządu zwracana tablica jest pusta

### DELETE /boardmembers/{code}

Usunięcie członka zarządu wskazanego kodem identyfikującym.

## POST /transactions

Utworzenie nowej transakcji. Parametry żądania:

| Parametr                  | Wymagane | Opis                                                     |
| ------------------------- | -------- | -------------------------------------------------------- |
| **type**                  | TAK      | Typ transakcji. Aktualnie wspierane: buyer, vender, broker |
| **status**                | TAK      | Status transakcji. Aktualnie wspierane: new, draft       |
| **occasionalTransaction** | TAK      | Informacja czy transakcja jest okazjonalna (bool)        |

Jeśli transakcja jest oznaczona jako okazjonalna wymagane są następujące parametry:

| Parametr          | Wymagane | Opis                                                       |
| ----------------- | -------- | ---------------------------------------------------------- |
| **amount**        | TAK      | Kwota transakcji                                           |
| **currency**      | TAK      | Waluta transakcji                                          |
| **bookedAt**      | TAK      | Data zaksięgowania transakcji                              |
| **paymentMethod** | TAK      | Sposób płatności                                           |
| **location**      | NIE      | Kraj w którym została przeprowadzona transakcja            |
| **description**   | NIE      | Opis transakcji (tytuł, przedmiot transakcji itd.)         |
| **references**    | NIE      | Referencje własne                                          |
| **senderIban**    | NIE      | Numer konta nadawcy (wymagany gdy sposób płatności to bank_transfer)   |
| **receiverIban**  | NIE      | Numer konta odbiorcy (wymagany gdy sposób płatności to brank_transfer) |

Jeśli transakcja nie jest oznaczona jako okazjonalna dodatkowo należy podać poniższe parametry:

| Parametr                | Wymagane | Opis                                                      |
| ----------------------- | -------- | --------------------------------------------------------- |
| **senderFirstName**     | NIE      | Imie nadawcy (wymagany jeśli nie jest podany kod nadawcy)          |
| **senderLastName**      | NIE      | Nazwisko nadawcy (wymagany jeśli nie jest podany kod nadawcy)      |
| **senderCompanyName**   | NIE      | Nazwa firmy nadawcy (wymagany jeśli nie jest podany kod nadawcy)   |
| **senderCode**          | TAK      | Kod podmiotu nadawcy                                      |
| **receiverFirstName**   | NIE      | Imie odbiorcy (wymagany jeśli nie jest podany kod odbiorcy)        |
| **receiverLastName**    | NIE      | Nazwisko odbiorcy (wymagany jeśli nie jest podany kod odbiorcy)    |
| **receiverCompanyName** | NIE      | Nazwa firmy odbiorcy (wymagany jeśli nie jest podany kod odbiorcy) |
| **receiverCode**        | TAK      | Kod podmiotu odbiorcy                                     |

#### Przykładowe dane do utworzenia transakcji:

```json
{
  "type": "buyer",
  "amount": "1250",
  "currency": "PLN",
  "location": "PL",
  "bookedAt": "2023-02-14 15:15:00",
  "description": "zapłata za rower",
  "paymentMethod": "Gotówka",
  "receiverFirstName": "Jan",
  "receiverLastName": "Kowalski",
  "occasionalTransaction": false,
  "createdByName": "Tester",
  "status": "accepted"
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "wz2xfsumnjrv",
    "type": "broker",
    "status": "accepted",
    "amount": "1250.00",
    "currency": "PLN",
    "location": "PL",
    "bookedAt": "2021-05-10 11:06:29",
    "description": "zapłata za rower",
    "paymentMethod": "bank_transfer",
    "senderFirstName": "Jan",
    "senderLastName": "Kowalski",
    "senderCompanyName": null,
    "senderIban": "PL12341234123412341234123412",
    "senderCode": "htu7evj63xkf",
    "receiverFirstName": null,
    "receiverLastName": null,
    "receiverCompanyName": "super firma",
    "receiverIban": "PL34123412341234123412341234",
    "receiverCode": "8mjken1c725h",
    "references": "ABC123123123",
    "createdAt": "2022-07-18T12:39:00.000000Z",
    "occasionalTransaction": null
  }
}
```

### GET /transactions

Zwraca transakcje utworzone przez użytkownika.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "nx9dpmhkrqz7",
      "type": "broker",
      "senderFirstName": "Jan",
      "senderLastName": "Kowalski",
      "senderCompanyName": null,
      "amount": "11250.00",
      "description": "auto",
      "receiverFirstName": "Adam",
      "receiverLastName": "Nowak",
      "receiverCompanyName": null
    },
    {
      "code": "gwt3kpbqr1hy",
      "type": "buyer",
      "senderFirstName": null,
      "senderLastName": null,
      "senderCompanyName": null,
      "amount": "1250.00",
      "description": "rower",
      "receiverFirstName": "Adam",
      "receiverLastName": "Nowak",
      "receiverCompanyName": null
    },
    {
      "code": "2fyu8m6e19qd",
      "type": "vender",
      "senderFirstName": "Jan",
      "senderLastName": "Kowalski",
      "senderCompanyName": null,
      "amount": "25.00",
      "description": "bilety",
      "receiverFirstName": null,
      "receiverLastName": null,
      "receiverCompanyName": null
    }
  ]
}
```

### GET /transactions/{code}

Pobranie szczegółów danej transakcji.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "zf14tw35pcqu",
    "type": "broker",
    "status": "new",
    "amount": "1250.00",
    "currency": "PLN",
    "location": "PL",
    "bookedAt": "2021-05-10 11:06:29",
    "description": "rower",
    "paymentMethod": "cash",
    "senderFirstName": "Jan",
    "senderLastName": "Kowalski",
    "senderCompanyName": null,
    "senderIban": "PL34123412341234123412341234",
    "senderCode": null,
    "receiverFirstName": "Adam",
    "receiverLastName": "Nowak",
    "receiverCompanyName": null,
    "receiverIban": "PL12341234123412341234123412",
    "receiverCode": null,
    "references": "kupno roweru",
    "createdAt": "2022-06-20T14:04:38.000000Z",
    "occasionalTransaction": 0
  }
}
```

### GET /transactions/pdf/{code}

Rozpoczyna pobieranie raportu pdf z transakcji wskazanej kodem identyfikującym.

### DELETE /transactions/{code}

Usunięcie transakcji wskazanej kodem identyfikującym.

### POST /events

Tworzenie nowego zdarzenia w systemie. Parametry żądania:

| Parametr         | Wymagane | Opis                                                          |
| ---------------- | -------- | ------------------------------------------------------------- |
| **description**  | TAK      | Opis zdarzenia                                                |
| **significance** | TAK      | Ważność zdarzenia. Aktualnie wspierane: info, warning, urgent |
| **partyCode**    | NIE      | Kod powiązanego podmiotu                                      |
| **transactionCode**| NIE      | Kod powiązanej transakcji                                   |

#### Przykładowe dane do utworzenia zdarzenia:

```json
{
  "description": "zdarzenie testowe",
  "significance": "urgent",
  "partyCode": null
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "rbdm8nh6gpc1",
    "partyCode": null,
    "description": "zdarzenie testowe",
    "type": "user",
    "significance": "info",
    "createdAt": "2022-03-15T13:05:44.000000Z",
    "hasComments": false
  }
}
```

### GET /events

Zwraca zdarzenia przypisane do użytkownika.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "rbdm8nh6gpc1",
      "partyCode": null,
      "description": "zdarzenie testowe",
      "type": "user",
      "significance": "urgent",
      "createdAt": "2022-03-15T13:05:44.000000Z",
      "hasComments": false
    },
    {
      "code": "4vhtcg7ry85m",
      "partyCode": null,
      "description": "Rule assigned",
      "type": "system",
      "significance": "info",
      "createdAt": "2022-03-09T11:05:28.000000Z",
      "hasComments": true
    }
  ]
}
```

Jeśli użytkownik nie posiada żadnych zdarzeń zwracany jest adekwatny komunikat ze statusem 200.

### GET /events/{code}

Zwraca zdarzenie o podanym identyfikatorze wraz z powiązanymi z nim komentarzami.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "2f4pxnwqvtzg",
    "partyCode": "eh46xfadk8n3",
    "description": "Party account created",
    "type": "create",
    "significance": "info",
    "createdAt": "2022-03-15T12:47:02.000000Z",
    "comments": [
      {
        "code": "j38fbv25qw4a",
        "content": "testowy komentarz",
        "created": "2022-03-15T15:41:20.000000Z"
      }
    ]
  }
}
```

Jeśli zdarzenie nie posiada przypisanych komentarzy tablica zwracana przy kluczu "comments" jest pusta.

### DELETE /events/{code}

Usunięcie zdarzenia wskazanego kodem identyfikującym.

### POST /comments

Tworzenie nowego zdarzenia w systemie. Parametry żądania:

| Parametr      | Wymagane | Opis                                                           |
| ------------- | -------- | -------------------------------------------------------------- |
| **content**   | TAK      | Treść komentarza                                               |
| **eventCode** | TAK      | Identyfikator zdarzenia do którego będzie przypisany komentarz |

#### Przykładowe dane do utworzenia komentarza:

```json
{
  "content": "testowy komentarz",
  "eventCode": "6jxh93gpasv2"
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "j38fbv25qw4a",
    "content": "testowy komentarz",
    "created": "2022-03-15T15:41:20.000000Z"
  }
}
```
### DELETE /comments/{code}

Usunięcie komentarza wskazanego kodem identyfikującym.
### GET /alerts

Pobranie alertów powiązanych z danym użytkownikiem.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "9u187g4y2fcq",
      "content": "alert testowy api",
      "type": "task",
      "status": "new",
      "partyCode": null,
      "transactionCode": null
    }
  ]
}
```

### GET /alerts/{code}

Pobranie szczegółów alertu wskazanego kodem.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "9u187g4y2fcq",
    "content": "alert testowy api",
    "type": "task",
    "status": "new",
    "taskDone": false,
    "partyCode": null,
    "transactionCode": null,
    "createdAt": "2022-07-18T13:22:42.000000Z"
  }
}
```

### DELETE /alerts/{code}

Usunięcie alertu wskazanego kodem identyfikującym.

### POST /tasks

Tworzenie nowego zadania w systemie. Parametry żądania:

| Parametr            | Wymagane | Opis                                             |
| ------------------- | -------- | ------------------------------------------------ |
| **content**         | TAK      | Treść zadania                                    |
| **expirationDate**  | NIE      | Data przed którą zadanie powinno zostać wykonane |
| **alertCode**       | NIE      | Kod alertu który będzie powiązany z zadaniem     |
| **partyCode**       | NIE      | Kod podmiotu który będzie powiązany z zadaniem   |
| **transactionCode** | NIE      | Kod transakcji która będzie powiązana z zadaniem |

#### Przykładowe dane do utworzenia zadania:

```json
{
  "content": "Utworzyć przykładowe zadanie do celów reprezentacyjnych w dokumentacji",
  "expirationDate": "2023-04-24",
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "nga1jz6c8v45",
    "content": "Utworzyć przykładowe zadanie do celów reprezentacyjnych w dokumentacji",
    "status": "new",
    "createdAt": "2023-04-21T11:08:42.000000Z"
  }
}
```

### GET /tasks
Pobranie zadań powiązanych z danym użytkownikiem.

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "x59w7bjksazc",
    "content": "Uzupełnij dane niezbędne do wystawienia faktury za abonament",
    "status": "displayed",
    "alertCode": "6eky4dmqvcwf",
    "partyCode": null,
    "transactionCode": null,
    "expirationDate": null
  },
  {
    "code": "nga1jz6c8v45",
    "content": "Utworzyć przykładowe zadanie do celów reprezentacyjnych w dokumentacji",
    "status": "new",
    "alertCode": null,
    "partyCode": null,
    "transactionCode": null,
    "expirationDate": null,
    "createdAt": "2023-04-21T11:08:42.000000Z"
  }
}
```

### GET /tasks/{code}
Pobranie szczegółów zadania wskazanego kodem.

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "x59w7bjksazc",
    "content": "Uzupełnij dane niezbędne do wystawienia faktury za abonament",
    "status": "displayed",
    "alertCode": "6eky4dmqvcwf",
    "partyCode": null,
    "transactionCode": null,
    "expirationDate": null
  }
}
```

### DELETE /tasks/{code}

Usunięcie zadania wskazanego kodem identyfikującym.