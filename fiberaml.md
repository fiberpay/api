---
Wspomaganie działań przeciwdziałania praniu pieniędzy i finansowania terroryzmu
---

# FiberAML

###

## Informacje ogólne

Głównym zastosowaniem API jest katalogowanie podmiotów, przez działalności od których wymagane jest prowadzenie ewidencji klientów.

### Klucze API

Do korzystania z API konieczne jest wygenerowanie kluczy:

* jawnego (apiKey) - używanego do przesyłania w ramach żądań API
* tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

W celu uzyskania danych dostępowych niezbędnych do poprawnego korzystania z API należy skontaktować się bezpośrednio z usługodawcą.

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

* Api-Key – wygenerowany klucz jawny

W przypadku zapytań nie posiadających body, należy wysłąć żądanie z nagłówkiem Authorization o wartości "Bearer {token}" (pusty string w postaci JWT z odpowiednią sygnaturą).

### Ciało zapytania

Każde ciało zapytania jest przekazywane za pomocą JWT z wykorzystaniem odpowiedniej sygnatury. Body żądania powinno być tekstem (JWT).

## Opis usług

### POST /parties

Utworzenie nowego podmiotu. Parametry żądania:

| Parametr | Wymagane | Opis                                                                         |
| -------- | -------- | ---------------------------------------------------------------------------- |
| **type** | TAK      | Typ podmiotu. Aktualnie wspierane: individual, sole\_proprietorship, company |

W zależności od wybranego typu wymagane są następujące parametry:

a) individual:

| Parametr                   | Wymagane | Opis                                                                                                                    |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------- |
| **firstName**              | NIE      | Imie podmiotu                                                                                                           |
| **lastName**               | NIE      | Nazwisko podmiotu                                                                                                       |
| **personalIdentityNumber** | TAK      | Numer PESEL podmiotu (w przypadku braku numeru pesel wymagany jest parametr personalIdentifier)                         |
| **personalIdentifier**     | NIE      | Numer identifykacyjny podmiotu (wymagany jeśli nie ma numeru pesel)                                                     |
| **documentType**           | TAK      | Rodzaj dokumentu Aktualnie wspierane: id\_card, passport, residency\_card (nie jest wymagany jeśli nie ma numeru pesel) |
| **documentNumber**         | TAK      | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                                           |
| **documentExpirationDate** | NIE      | Termin ważnosci dokumentu                                                                                               |
| **withoutExpirationDate**  | NIE      | Informacja czy dokument posiada datę ważności (bool)                                                                    |
| **citizenship**            | NIE      | Obywatelstwo (kod kraju standardzie ISO)                                                                                |
| **birthCity**              | NIE      | Miasto urodzenia                                                                                                        |
| **birthCountry**           | NIE      | Kraj urodzenia                                                                                                          |
| **politicallyExposed**     | NIE      | Informacja czy podmiot jest eksponowany politycznie (bool)                                                              |

b) sole\_proprietorship - wszystkie powyższe oraz:

| Parametr                           | Wymagane | Opis                                                         |
| ---------------------------------- | -------- | ------------------------------------------------------------ |
| **companyName**                    | NIE      | Nazwa prowadzonej działalności                               |
| **taxIdNumber**                    | TAK      | NIP prowadzonej działalności                                 |
| **nationalBusinessRegistryNumber** | NIE      | Regon prowadzonej działalności                               |
| **companyIdentifier**              | NIE      | Numer identyfikujący (wymagany jeśli nie ma numeru NIP)      |
| **registrationCountry**            | NIE      | Kraj rejestracji podmiotu (podawany jeśli nie ma numeru NIP) |

c) company:

| Parametr                           | Wymagane | Opis                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **companyName**                    | NIE      | Nazwa firmy                                                                                                                                                                                                                                                                                                               |
| **taxIdNumber**                    | TAK      | Numer NIP                                                                                                                                                                                                                                                                                                                 |
| **nationalBusinessRegistryNumber** | NIE      | Numer Regon                                                                                                                                                                                                                                                                                                               |
| **tradeName**                      | NIE      | Nazwa handlowa firmy                                                                                                                                                                                                                                                                                                      |
| **nationalCourtRegistryNumber**    | NIE      | Numer KRS                                                                                                                                                                                                                                                                                                                 |
| **businessActivityForm**           | TAK      | Rodzaj prowadzonej działalności (Nie jest wymagane jeśli nie ma numeru NIP). Aktualnie wspierane: limited\_liability\_company, civil\_partnership\_company, general\_partnership\_company, professional\_partnership\_company, limited\_partnership\_company, limited\_joint\_stock\_partnership\_company, stock\_company |
| **industry**                       | NIE      | Branża                                                                                                                                                                                                                                                                                                                    |
| **servicesDescription**            | NIE      | Opis usług                                                                                                                                                                                                                                                                                                                |
| **companyIdentifier**              | NIE      | Numer identyfikujący (wymagany jeśli nie ma numeru NIP)                                                                                                                                                                                                                                                                   |
| **registrationCountry**            | NIE      | Kraj rejestracji podmiotu (podawany jeśli nie ma numeru NIP)                                                                                                                                                                                                                                                              |

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
| **email**        | NIE      | Adres email            |
| **phoneCountry** | NIE      | Prefix numeru telefonu |
| **phoneNumber**  | NIE      | Numer telefonu         |

#### Przykładowe dane do utworzenia podmiotu typu 'individual':

```json
{
  "type": "individual",
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
  "accommodationAddress": {
    "country": "PL",
    "city": "Warszawa",
    "street": "Grzybowska",
    "houseNumber": "4",
    "flatNumber": "106",
    "postalCode": "00-131"
  }
}
```

#### Przykładowe dane do utworzenia podmiotu typu 'sole\_proprietorship':

```json
{
  "type": "sole_proprietorship",
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
  "companyName": "Usługi programistyczne",
  "taxIdNumber": "7010634566",
  "nationalBusinessRegistryNumber": "365899489",
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
    "email": "fiberpay@fiberpay.pl",
    "phoneCountry": "48",
    "phoneNumber": "123123123"
  },
  "companyContact": {
    "email": "fiberpay@fiberpay.pl",
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
  "nationalBusinessRegistryNumber": "365899489",
  "tradeName": "FiberPay",
  "nationalCourtRegistryNumber": "1234567890",
  "businessActivityForm": "limited_liability_company",
  "industry": "usługi programistyczne",
  "servicesDescription": "tworzenie aplikacji webowych"
}
```

#### Przykładowa odpowiedź serwera:

* **STATUS 201 CREATED**

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
      "birthCity": "Dębica",
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

### PATCH /parties

Edycja utworzonego wcześniej podmiotu. Parametry żądania:

| Parametr | Wymagane | Opis                                                                        |
| -------- | -------- | --------------------------------------------------------------------------- |
| **code** | TAK      | Kod podmiotu.                                       |

Pozostałe parametry żądania są identyczne jak w przypadku tworzenia podmiotu.

#### Przykładowe dane do edycji podmiotu typu 'individual':

```json
{
  "code": "xswyvgqa37zp",
  "type": "individual",
  "firstName": "Jan",
  "lastName": "Kowalski",
  "personalIdentityNumber": "01234567880",
  "documentType": "id_card",
  "documentNumber": "AAA123456",
  "documentExpirationDate": "2022-10-15",
  "withoutExpirationDate" : false,
  "citizenship": "PL",
  "birthCity": "Warszawa",
  "birthCountry": "PL",
  "politicallyExposed": false,
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
    "postalCode": "11-131"
  }
}
```

#### Przykładowa odpowiedź serwera:

- **STATUS 200 OK**

```json
{
  "data": {
    "code": "htu7evj63xkf",
    "type": "individual",
    "entity": {
      "code": "1jz4p297vgxd",
      "firstName": "Jan",
      "lastName": "Kowalski",
      "personalIdentityNumber": "50201204873",
      "documentType": "id_card",
      "documentNumber": "aze123456",
      "documentExpirationDate": "2022-10-15",
      "withoutExpirationDate": false,
      "citizenship": "PL",
      "birthCity": "Dębica",
      "birthCountry": "PL",
      "politicallyExposed": false,
      "createdAt": "2022-06-01T14:38:22.000000Z",
      "birthDate": null
    },
    "addresses": [
      {
        "code": "jnh1s2854a6f",
        "type": "accommodation_address",
        "country": "PL",
        "city": "Warszawa",
        "street": "Grzybowska",
        "houseNumber": "4",
        "flatNumber": "106",
        "postalCode": "00-131",
        "createdAt": "2022-06-01T14:38:22.000000Z"
      },
      {
        "code": "bhjp8254amxw",
        "type": "forwarding_address",
        "country": "PL",
        "city": "Warszawa",
        "street": "Grzybowska",
        "houseNumber": "4",
        "flatNumber": "106",
        "postalCode": "00-131",
        "createdAt": "2022-07-18T11:44:46.000000Z"
      }
    ],
    "contacts": []
  }
}
```

### GET /parties

Zwraca podmioty utworzone przez użytkownika.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

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

* **STATUS 200 OK**

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

### POST /parties/{code}/beneficiares

Dodanie beneficjenta rzeczywistego do podmiotu typu company. Parametry żądania:

| Parametr        | Wymagane | Opis                                                          |
| --------------- | -------- | ------------------------------------------------------------- |
| **ownedShares** | TAK      | Procent posiadanych udziałów. Przyjmuje wartości od 1 do 100. |
| **beneficiary** | TAK      | Obiekt zawierający dane beneficjenta                          |

Struktura obiektu z danymi beneficjenta:

| Parametr                   | Wymagane | Opis                                                                                                                    |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------- |
| **firstName**              | NIE      | Imie podmiotu                                                                                                           |
| **lastName**               | NIE      | Nazwisko podmiotu                                                                                                       |
| **personalIdentityNumber** | TAK      | Numer PESEL podmiotu (w przypadku braku numeru pesel wymagany jest parametr personalIdentifier)                         |
| **documentType**           | TAK      | Rodzaj dokumentu Aktualnie wspierane: id\_card, passport, residency\_card (nie jest wymagany jeśli nie ma numeru pesel) |
| **documentNumber**         | TAK      | Numer dokumentu (nie jest wymagany jeśli nie ma numeru pesel)                                                           |
| **documentExpirationDate** | NIE      | Termin ważnosci dokumentu                                                                                               |
| **citizenship**            | NIE      | Obywatelstwo (kod kraju standardzie ISO)                                                                                |
| **birthCity**              | NIE      | Miasto urodzenia                                                                                                        |
| **birthCountry**           | NIE      | Kraj urodzenia                                                                                                          |
| **politicallyExposed**     | NIE      | Informacja czy podmiot jest eksponowany politycznie (bool)                                                              |

#### Przykładowe dane do dodania beneficjenta rzeczywistego:

```json
{
  "ownedShares": 10,
  "beneficiary": {
    "firstName": "Jan",
    "lastName": "Kowalski",
    "personalIdentityNumber": "01234567890",
    "documentType": "id_card",
    "documentNumber": "AZE123456",
    "documentExpirationDate": "2023-10-15",
    "citizenship": "PL",
    "birthCity": "Warszawa",
    "birthCountry": "PL",
    "politicallyExposed": false
  }
}
```

#### Przykładowa odpowiedź serwera:

* **STATUS 201 CREATED**

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

* **STATUS 200 OK**

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

### DELETE /parties/{code}

Usunięcie podmiotu wskazanego kodem identyfikującym.

## POST /transactions

Utworzenie nowej transakcji. Parametry żądania:

| Parametr | Wymagane | Opis                                                                        |
| -------- | -------- | --------------------------------------------------------------------------- |
| **type** | TAK      | Typ podmiotu. Aktualnie wspierane: buyer, vender, broker |
| **occasionalTransaction** | TAK      | Informacja czy transakcja jest okazjonalna (bool) |

Jeśli transakcja jest oznaczona jako okazjonalna wymagane są następujące parametry:

| Parametr          | Wymagane | Opis                                                               |
| --------          | -------- | ------------------------------------------------------------------ |
| **amount**        | TAK      | Kwota transakcji                                                   |
| **currency**      | TAK      | Waluta transakcji                                                  |
| **location**      | NIe      | Kraj w którym została przeprowadzona transakcja                    |
| **bookedAt**      | NIE      | Data zaksięgowania transakcji                                      |
| **description**   | NIE      | Opis transakcji (tytuł, przedmiot transakcji itd.)                 |
| **references**    | NIE      | Referencje własne                                                  |
| **paymentMethod** | NIE      | Sposób płatności                                                   |
| **senderIban**    | NIE      | Numer konta nadawcy (gdy sposób płatności bank_transfer)           |
| **receiverIban**  | NIE      | Numer konta odbiorcy (gdy sposób płatności brank_transfer)         |

Jeśli transakcja nie jest oznaczona jako okazjonalna dodatkowo należy podać poniższe parametry:

| Parametr                | Wymagane | Opis                                                         |
| --------                | -------- | ------------------------------------------------------------ |
| **senderFirstName**     | NIE      | Imie nadawcy (jeśli nie jest podany kod nadawcy)             |
| **senderLastName**      | NIE      | Nazwisko nadawcy (jeśli nie jest podany kod nadawcy)         |
| **senderCompanyName**   | NIE      | Nazwa firmy nadawcy (jeśli nie jest podany kod nadawcy)      |
| **senderCode**          | TAK      | Kod podmiotu nadawcy                                         |
| **receiverFirstName**   | NIE      | Imie odbiorcy (jeśli nie jest podany kod odbiorcy)           |
| **receiverLastName**    | NIE      | Nazwisko odbiorcy (jeśli nie jest podany kod odbiorcy)       |
| **receiverCompanyName** | NIE      | Nazwa firmy odbiorcy (jeśli nie jest podany kod odbiorcy)    |
| **receiverCode**        | TAK      | Kod podmiotu odbiorcy                                        |

#### Przykładowe dane do utworzenia transakcji:

```json
  {
    "type": "broker",
    "occasionalTransaction": false,
    "amount": 1250,
    "currency": "PLN",
    "location": "PL",
    "bookedAt": "2021-05-10 11:06:29",
    "description": "zapłata za rower",
    "references": "ABC123123123",
    "paymentMethod": "bank_transfer",
    "senderIban": "PL12341234123412341234123412",
    "receiverIban": "PL34123412341234123412341234",
    "senderFirstName": "",
    "senderLastName": "",
    "senderCode": "htu7evj63xkf",
    "receiverFirstName": "",
    "receiverLastName": "",
    "receiverCode": "8mjken1c725h"
  }
```

#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**
```json
{
  "data": {
    "code": "wz2xfsumnjrv",
    "type": "broker",
    "status": "new",
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

### DELETE /transactions/{code}

Usunięcie transakcji wskazanej kodem identyfikującym.

### POST /events

Tworzenie nowego zdarzenia w systemie. Parametry żądania:

| Parametr         | Wymagane | Opis                                                          |
| ---------------- | -------- | ------------------------------------------------------------- |
| **description**  | TAK      | Opis zdarzenia                                                |
| **significance** | TAK      | Ważność zdarzenia. Aktualnie wspierane: info, warning, urgent |
| **partyCode**    | NIE      | Kod powiązanego podmiotu                                      |

#### Przykładowe dane do utworzenia zdarzenia:

```json
{
  "description": "zdarzenie testowe",
  "significance": "urgent",
  "partyCode": null
}
```

#### Przykładowa odpowiedź serwera:

* **STATUS 201 CREATED**

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

* **STATUS 200 OK**

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

### GET /events/party/{code}

Zwraca zdarzenia przypisane do użytkownika i powiązane z podmiotem o podanym identyfikatorze.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "7mfv2d45tnqy",
      "partyCode": "eh46xfadk8n3",
      "description": "Party created",
      "type": "create",
      "significance": "info",
      "createdAt": "2022-03-15T12:45:12.000000Z",
      "hasComments": false
    }
  ]
}
```

Jeśli użytkownik nie posiada żadnych zdarzeń zwracany jest adekwatny komunikat ze statusem 200.

### GET /events/{code}

Zwraca zdarzenie o podanym identyfikatorze wraz z powiązanymi z nim komentarzami.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

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
  "eventCode": "2f4pxnwqvtzg"
}
```

#### Przykładowa odpowiedź serwera:

* **STATUS 201 CREATED**

```json
{
  "data": {
    "code": "j38fbv25qw4a",
    "content": "testowy komentarz",
    "created": "2022-03-15T15:41:20.000000Z"
  }
}
```

### POST /alerts

Utworzenie nowego alertu. Tworzony przez użytkownika alert będzie miał przypisany rodzaj "zadanie". Parametry żądania:

| Parametr            | Wymagane | Opis                                                                        |
| --------            | -------- | --------------------------------------------------------------------------- |
| **content**         | TAK      | Treść alertu.                                                               |
| **partyCode**       | NIE      | Kod powiązanego podmiotu.                                                   |
| **transactionCode** | NIE      | Kod powiązanej transakcji.                                                  |

#### Przykładowe dane do utworzenia alertu:

```json
{
  "content": "alert testowy api",
  "transactionCode": "",
  "partyCode": "",
}
```
#### Przykładowa odpowiedź serwera:

- **STATUS 201 CREATED**

```json
{
  "data":
    {
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

Pobranie szczegółów wskazanego kodem alertu.

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

### GET /subscriptions

Zwraca możliwe do wykupienia plany subskrypcji.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

```json
{
  "data": [
    {
      "name": "monthPlan",
      "duration": "month",
      "price": "50.00",
      "currency": "PLN"
    },
    {
      "name": "quarterPlan",
      "duration": "3 months",
      "price": "125.00",
      "currency": "PLN"
    },
    {
      "name": "yearPlan",
      "duration": "12 months",
      "price": "450.00",
      "currency": "PLN"
    }
  ]
}
```

### GET /subscriptions/mine

Zwraca informacje o aktywnych i utworzonych subskrypcjach.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

```json
{
  "data": [
    {
      "code": "w1qdj84u6hta",
      "type": "monthPlan",
      "status": "active",
      "availableFrom": "2022-06-01 00:00:00",
      "availableTo": "2022-07-01 00:00:00",
      "redirect": null
    }
  ]
}
```

### GET /subscriptions/mine

Zwraca informacje o aktywnej subskrypcji.

#### Przykładowa odpowiedź serwera:

* **STATUS 200 OK**

```json
{
  "data": {
    "code": "w1qdj84u6hta",
    "type": "monthPlan",
    "status": "active",
    "availableFrom": "2022-06-01 00:00:00",
    "availableTo": "2022-07-01 00:00:00"
  }
}
```

### POST /subscriptions/create

Utworzenie nowej subskrypcji. Parametry żądania:

| Parametr          | Wymagane | Opis                                                               |
| ----------------- | -------- | ------------------------------------------------------------------ |
| **type**          | TAK      | Nazwa subskrypcji                                                  |
| **availableFrom** | TAK      | Data od kiedy subskrypcja ma zacząć działać, wymagany format Y-m-d |

#### Przykładowe dane do utworzenia komentarza:

```json
{
  "type": "monthPlan",
  "availableFrom": "2022-07-01"
}
```

#### Przykładowa odpowiedź serwera:

* **STATUS 201 CREATED**

```json
{
    "data": {
        "code": "f6xuvsw5jgz4",
        "type": "monthPlan",
        "status": "waiting_for_payment",
        "availableFrom": "2022-06-30T22:00:00.000000Z",
        "availableTo": "2022-07-31T22:00:00.000000Z",
        "redirect": "http://fiberpay.pl/"
    }
}
```
## DELETE /subscriptions/{code}

Usunięcie subksrypcji wskazanej kodem identyfikującym. Subksrypcja może zostać usunięta tylko wtedy gdy nie jest jeszcze opłacona.