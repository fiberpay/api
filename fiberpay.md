---
Automatyzacja obsługi płatności
---

# FiberPay

## Informacje ogólne

### Adresy serwerów

| Funkcjonalność     | Środowisko  | URL                                                                |
| ------------------ | ----------- | ------------------------------------------------------------------ |
| Serwer API         | Produkcyjne | [https://api.fiberpay.pl/1.0](https://api.fiberpay.pl/1.0)         |
| Panel deweloperski | Produkcyjne | [https://fiberpay.pl](https://fiberpay.pl)                         |
| Serwer API         | Testowe     | [https://apitest.fiberpay.pl/1.0](https://apitest.fiberpay.pl/1.0) |
| Panel deweloperski | Testowe     | [https://test.fiberpay.pl](https://test.fiberpay.pl)               |

#### UWAGA!

Są to zupełnie rozdzielone środowiska (łącznie z infrastrukturą). **Klucze API z jednego środowiska nie będą działać w drugim**.

### Klucze API

Do korzystania z API konieczne jest wygenerowanie kluczy:

* jawnego (publicKey) - używanego do przesyłania w ramach żądań API
* tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

Klucze możesz wygenerować korzystając z panelu użytkownika Fiberpay.

### Biblioteka API

Fiberpay posiada publicznie dostępną bibliotekę umożliwiającą komunikację z API utworzoną w języku PHP.

Jej instalacja możliwa jest przy użyciu narzędzia Composer \[[link](https://packagist.org/packages/fiberpay/fiberpay-php)]

```bash
composer require fiberpay/fiberpay-php
```

### Nagłówki

Każde żądanie do API powinno posiadać następujące nagłówki:

* Content-Type - application/json
* Accept - application/json
* X-API-Route – informacja o wywoływanej metodzie HTTP oraz URI
  * np. `POST /1.0/orders/collect`
  * np. `GET /1.0/orders/collect/12345678`
* X-API-Nonce – nonce w postaci liczba naturalnej, każdy następny nonce powinien być większy od poprzedniego (przykładowo można użyć timestampa lub zaokrąglonego microtime jeśli używamy PHP)
* X-API-Key – wygenerowany klucz jawny
* X-API-Signature – podpis z użyciem skrótu **sha512** stworzonego przy użyciu _**‘route’**_, _**‘nonce’**_, _**‘apiKey’**_, _**‘requestBody’**_ i _**‘secretKey’**_

#### Instrukcja utworzenia X-API-Signature

Należy utworzyć połączony ciąg znaków (concatenated string) składający się z route, nonce, apiKey oraz requestBody (kolejność ma znaczenie!).

Wartość requestBody to treść (body) żądania HTTP, które przesyłamy do API. W wybranych przypadkach wartość requestBody będzie pusta, bo całość wywołania API może ograniczać się do URLa (np. `GET /1.0/orders/collect/12345678`). W takim przypadku należy użyć pustego ciągu tekstowego ('').

Następnie należy wygenerować skrót **sha512** z utworzonego ciągu używając **secretKey**.

Przykładowa implementacja w PHP:

```php
$implodeParams = implode( ‘’, [$route, $nonce, $apiKey, $requestBody]);
hash_hmac(‘sha512’, $implodeParams, $secretKey)
```

### Mechanizm callbacków

FiberPay posiada mechanizm tzw. callbacków. Po aktualizacji zlecenia system FiberPay może wywołać żądanie HTTP na wskazany uprzednio adres, gdzie:

* w ciele (body) żądania będzie zawarty token JWT z aktualnymi danymi zlecenia,
* w nagłówku (header) żądania będzie zawarty jawny klucz API, wskazujacy który klucz tajny został wykorzystany do utworzenia JWT

Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT znajdziesz pod adresem: [https://jwt.io/](https://jwt.io/)

### Faktury

Możliwe jest pobranie za pośrednictwem API informacji o fakturach wystawionych przez nas za zrealizowane usługi.

#### GET /invoices

Pobranie informacji o wystawionych fakturach

Przykładowa odpowiedź serwera:

```javascript
{
  "data": [
    {
      "data": {
        "code": "8vdf57y9zwqe",
        "number": "FP/bcde/2021/03/1",
        "amount": 100,
        "currency": "PLN",
        "downloadUrl": "https://fiberpay.fakturownia.net/f/FP-bcde-2021-03-1/ZYzAfJL0f0LnKLajhsdM.pdf",
        "billingPeriod": "2021-03",
        "issueDate": "2021-04-01"
      },
      "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/invoices/8vdf57y9zwqe"
      }
    },
    {
      "data": {
        "code": "5ped9nrvyzs3",
        "number": "FP/bcde/2021/03/2",
        "amount": 500,
        "currency": "PLN",
        "downloadUrl": "https://fiberpay.fakturownia.net/f/FP-bcde-2021-03-2/mrWTlweIAPpdrbgBop80.pdf",
        "billingPeriod": "2021-03",
        "issueDate": "2021-04-01"
      },
      "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/invoices/5ped9nrvyzs3"
      }
    }
  ],
  "links": {
    "first": "https://apitest.fiberpay.pl/1.0/invoices?page=1",
    "last": "https://apitest.fiberpay.pl/1.0/invoices?page=1",
    "prev": null,
    "next": null
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "links": [
      {
        "url": null,
        "label": "&laquo; Previous",
        "active": false
      },
      {
        "url": "https://apitest.fiberpay.pl/1.0/invoices?page=1",
        "label": 1,
        "active": true
      },
      {
        "url": null,
        "label": "Next &raquo;",
        "active": false
      }
    ],
    "path": "https://apitest.fiberpay.pl/1.0/invoices",
    "per_page": 15,
    "to": 4,
    "total": 4
  }
}
```

#### GET /invoices/{code}

Pobranie informacji o pojedynczej fakturze

| Parametr | Wymagane | Opis        |
| -------- | -------- | ----------- |
| **code** | tak      | kod faktury |

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
    "code": "c7r3j5xaw9v8",
    "number": "FP/bcde/2021/03/3",
    "amount": 19,
    "currency": "PLN",
    "downloadUrl": "https://fiberpay.fakturownia.net/f/FP-bcde-2021-03-3/vV0Wlx2GX96ZZzQXp.pdf",
    "billingPeriod": "2021-03",
    "issueDate": "2021-04-01",
    "orders": {
      "data": [
        {
          "data": {
            "code": "685ztudjnmk2",
            "status": "completed",
            "type": "forward",
            "currency": "PLN",
            "targetAmount": "1800.0000",
            "targetName": "targetName",
            "targetIban": "PL61109010140000071219812874",
            "targetDescription": "targetDescription",
            "brokerAmount": "181.0000",
            "brokerName": "targetName",
            "brokerIban": "PL61109010140000071219812874",
            "feeAmount": "19.00",
            "redirectUrl": null,
            "beforePaymentInfo": null,
            "afterPaymentInfo": null,
            "createdAt": "2021-04-09 12:44:21",
            "updatedAt": "2021-04-09 12:44:21"
          },
          "invoice": {
            "amount": "2000.00",
            "currency": "PLN",
            "iban": "PL123400005",
            "bban": "123400005",
            "description": "g92e3nb6"
          },
          "links": {
            "rel": "self",
            "href": "https://apitest.fiberpay.pl/1.0/orders/forward/685ztudjnmk2"
          }
        }
      ],
      "links": {
        "first": "https://apitest.fiberpay.pl/1.0/invoices/c7r3j5xaw9v8?page=1",
        "last": "https://apitest.fiberpay.pl/1.0/invoices/c7r3j5xaw9v8?page=1",
        "prev": null,
        "next": null
      },
      "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "links": [
          {
            "url": null,
            "label": "&laquo; Previous",
            "active": false
          },
          {
            "url": "https://apitest.fiberpay.pl/1.0/invoices/c7r3j5xaw9v8?page=1",
            "label": 1,
            "active": true
          },
          {
            "url": null,
            "label": "Next &raquo;",
            "active": false
          }
        ],
        "path": "https://apitest.fiberpay.pl/1.0/invoices/c7r3j5xaw9v8",
        "per_page": 15,
        "to": 1,
        "total": 1
      }
    }
  }
}
```

### Rozliczenia

#### GET /settlements

Pobranie informacji o rozliczeniach

Przykładowa odpowiedź serwera:

```javascript
{
  "data": [
    {
      "data": {
        "code": "6a3mukzxtv",
        "status": "open",
        "amount": "0.0000",
        "feeAmount": "0.0000",
        "currency": "PLN",
        "closedAt": null,
        "createdAt": "2021-04-08 11:43:05",
        "updatedAt": "2021-04-08 11:43:05"
      }
    },
    {
      "data": {
        "code": "w6ny79cqma",
        "status": "open",
        "amount": "0.0000",
        "feeAmount": "0.0000",
        "currency": "PLN",
        "closedAt": null,
        "createdAt": "2021-04-09 11:09:15",
        "updatedAt": "2021-04-09 11:09:15"
      }
    }
  ]
}
```

#### GET /settlements/{code}

Pobranie informacji o pojedynczym rozliczeniu

| Parametr | Wymagane | Opis            |
| -------- | -------- | --------------- |
| **code** | tak      | kod rozliczenia |

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
    "code": "4jscrftnk5",
    "status": "open",
    "amount": "0.0000",
    "feeAmount": "0.0000",
    "currency": "PLN",
    "closedAt": null,
    "createdAt": "2021-04-09 12:03:33",
    "updatedAt": "2021-04-09 12:03:33"
  }
}
```

## Opis usług

### FiberDirect

Usługa pojedynczego przekazu pieniężnego na wskazane konto bankowe. Platforma korzystająca z API dostanie powiadomienie, gdy dany przekaz zostanie opłacony, a następnie gdy Fiberpay przekaże środki na docelowe konto.

#### POST /orders/direct

Utworzenie zlecenia. Parametry żądania:

| Parametr           | Wymagane | Opis                                                                  |
| ------------------ | -------- | --------------------------------------------------------------------- |
| **amount**         | tak      | kwota przekazu, decimal z maks. 2 miejscami po przecinku (np. 100.50) |
| **currency**       | tak      | waluta (aktualnie dostępne tylko PLN)                                 |
| **toName**         | tak      | nazwa odbiorcy                                                        |
| **toIban**         | tak      | IBAN odbiorcy                                                         |
| **description**    | nie      | tytuł przelewu                                                        |
| **metadata**       | nie      | opcjonalne dane przekazywane przez FiberPay w odpowiedzi              |
| **callbackUrl**    | nie      | URL na który ma być wywołany callback                                 |
| **callbackParams** | nie      | opcjonalne parametry callbacka                                        |

Przykładowa odpowiedź serwera

```javascript
{
  "data": {
      "code": "gjrbwhcx96v7",
      "status": "defined",
      "type": "direct",
      "currency": "PLN",
      "amount": "100.0000",
      "feeAmount": "0.5000",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "metadata": null,
      "createdAt": "2020-09-18 17:02:42",
      "updatedAt": "2020-09-18 17:02:42"
  },
  "invoice": {
      "amount": "100.50",
      "currency": "PLN",
      "iban": "PL123400007",
      "description": "gjrbwhcx96v7"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/direct/gjrbwhcx96v7"
  }
}
```

#### GET /orders/direct/{code}

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
      "code": "gjrbwhcx96v7",
      "status": "defined",
      "type": "direct",
      "currency": "PLN",
      "amount": "100.0000",
      "feeAmount": "0.5000",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "metadata": null,
      "createdAt": "2020-09-18 17:02:42",
      "updatedAt": "2020-09-18 17:02:42"
  },
  "invoice": {
      "amount": "100.50",
      "currency": "PLN",
      "iban": "PL123400007",
      "description": "gjrbwhcx96v7"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/direct/gjrbwhcx96v7"
  }
}
```

#### DELETE /orders/direct/{code}

Anulowanie wcześniej utworzonego zlecenia (możliwe tylko dla jeszcze nieopłaconych zleceń).

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

```javascript
{
  "data": {
      "code": "gjrbwhcx96v7",
      "status": "cancelled",
      "type": "direct",
      "currency": "PLN",
      "amount": "100.0000",
      "feeAmount": "0.5000",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "metadata": null,
      "createdAt": "2020-09-18 17:02:42",
      "updatedAt": "2020-09-18 17:03:34"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/direct/gjrbwhcx96v7"
  }
}
```

### FiberSplit

Usługa FiberSplit jest stosowana do obsługi masowych wypłat środków do wielu odbiorców. Pozwala wykonać wiele przekazów pieniężnych opłaconych jednym przelewem - przykładowo może to być 100 przelewów na wskazane konta bankowe po 100 zł zrealizowane w wyniku zasilenia zlecenia przelewem na 10.000 zł. Aplikacja korzystająca z API po założeniu zlecenia tego typu dostanie powiadomienie, gdy zostanie ono opłacone przelewem zasilającym. Następnie aplikacja będzie otrzymywała powiadomienia dot. statusu każdego ze jednostkowych przekazów pieniężnych w zleceniu.

Aby skorzystać z usługi należy:

* założyć zlecenie (POST /orders/split)
* dodać poszczególne przekazy pieniężne (POST /orders/split/item)
* zakończyć definicję zlecenia (PUT /orders/split/{code}/define)
* opłacić utworzone zlecenie

#### POST /orders/split

Tworzy zlecenie

| Parametr     | Wymagane | Opis                                                    |
| ------------ | -------- | ------------------------------------------------------- |
| **currency** | tak      | waluta zlecenia (aktualnie wspieramy tylko PLN)         |
| **metadata** | nie      | opcjonalne dane przekazywane przez FiberPay w callbacku |

Przykładowa odpowiedź

```javascript
{
  "data": {
      "code": "zc6ta75gfpme",
      "status": "open",
      "type": "split",
      "currency": "PLN",
      "metadata": null,
      "createdAt": "2020-09-18 17:05:32",
      "updatedAt": "2020-09-18 17:05:32",
      "items": []
  },
  "invoice": {
      "amount": "0.00",
      "currency": "PLN",
      "iban": null,
      "description": "zc6ta75gfpme"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/split/zc6ta75gfpme"
  }
}
```

#### POST /orders/split/item

Tworzy pojedynczy przelew do wysłania w ramach całego zlecenia.

| Parametr           | Wymagane | Opis                                                      |
| ------------------ | -------- | --------------------------------------------------------- |
| **amount**         | tak      | kwota przelewu                                            |
| **currency**       | tak      | waluta przelewu (aktualnie wspierane tylko PLN)           |
| **parentCode**     | tak      | kod zlecenia nadrzędnego                                  |
| **toName**         | tak      | nazwa odbiorcy                                            |
| **toIban**         | tak      | IBAN odbiorcy                                             |
| **description**    | tak      | tytuł przelewu                                            |
| **metadata**       | nie      | opcjonalne dane przekazywane przez FiberPay w odpowiedzi, |
| **callbackUrl**    | nie      | URL na który ma być wywołany callback                     |
| **callbackParams** | nie      | opcjonalne parametry callbacka                            |

Przykładowa odpowiedź

```javascript
{
  "data": {
      "code": "gmats9v3",
      "description": "testowy przelew",
      "status": "open",
      "type": "split_item",
      "currency": "PLN",
      "amount": "100",
      "feeAmount": "1.50",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "parentCode": "m9cqbfhk5x4e",
      "metadata": null,
      "createdAt": "2020-09-18 17:07:53",
      "updatedAt": "2020-09-18 17:07:53"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/split/item/gmats9v3"
  }
}
```

#### PUT /orders/split/{code}/define

Kończy tworzenie orderu (zamyka definicję).

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź:

```javascript
{
  "data": {
      "code": "m9cqbfhk5x4e",
      "status": "defined",
      "type": "split",
      "currency": "PLN",
      "metadata": null,
      "createdAt": "2020-09-18 17:06:30",
      "updatedAt": "2020-09-18 17:08:38",
      "items": [
          {
              "data": {
                  "code": "gmats9v3",
                  "description": "testowy przelew",
                  "status": "open",
                  "type": "split_item",
                  "currency": "PLN",
                  "amount": "100.0000",
                  "feeAmount": "1.5000",
                  "toName": "Michał",
                  "toIban": "PL27114020040000300201355387",
                  "parentCode": "m9cqbfhk5x4e",
                  "metadata": null,
                  "createdAt": "2020-09-18 17:07:53",
                  "updatedAt": "2020-09-18 17:07:53"
              },
              "links": {
                  "rel": "self",
                  "href": "https://apitest.fiberpay.pl/1.0/orders/split/item/gmats9v3"
              }
          }
      ]
  },
  "invoice": {
      "amount": "101.50",
      "currency": "PLN",
      "iban": "PL123400003",
      "description": "m9cqbfhk5x4e"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/split/m9cqbfhk5x4e"
  }
}
```

#### GET /orders/split/{code}

Pobranie informacji o całym zleceniu

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
      "code": "m9cqbfhk5x4e",
      "status": "defined",
      "type": "split",
      "currency": "PLN",
      "metadata": null,
      "createdAt": "2020-09-18 17:06:30",
      "updatedAt": "2020-09-18 17:08:38",
      "items": [
          {
              "data": {
                  "code": "gmats9v3",
                  "description": "testowy przelew",
                  "status": "open",
                  "type": "split_item",
                  "currency": "PLN",
                  "amount": "100.0000",
                  "feeAmount": "1.5000",
                  "toName": "Michał",
                  "toIban": "PL27114020040000300201355387",
                  "parentCode": "m9cqbfhk5x4e",
                  "metadata": null,
                  "createdAt": "2020-09-18 17:07:53",
                  "updatedAt": "2020-09-18 17:07:53"
              },
              "links": {
                  "rel": "self",
                  "href": "https://apitest.fiberpay.pl/1.0/orders/split/item/gmats9v3"
              }
          }
      ]
  },
  "invoice": {
      "amount": "101.50",
      "currency": "PLN",
      "iban": "PL123400003",
      "description": "m9cqbfhk5x4e"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/split/m9cqbfhk5x4e"
  }
}
```

#### GET /orders/split/item/{code}

Pobranie informacji o poszczególnym przekazie pieniężnym

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź serwera:

```javascript
{
    "data": {
        "code": "gmats9v3",
        "description": "testowy przelew",
        "status": "open",
        "type": "split_item",
        "currency": "PLN",
        "amount": "100.0000",
        "feeAmount": "1.5000",
        "toName": "Michał",
        "toIban": "PL27114020040000300201355387",
        "parentCode": "m9cqbfhk5x4e",
        "metadata": null,
        "createdAt": "2020-09-18 17:07:53",
        "updatedAt": "2020-09-18 17:07:53"
    },
    "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/orders/split/item/gmats9v3"
    }
}
```

### FiberCollect

Usługa pozwalająca na przyjęcie jednej lub więcej wpłat od płatników (lub płatnika), które następnie są agregowane i przekazywane w trybie dziennym na wskazane konto odbiorcy. Aplikacja korzystająca z API może założyć zlecenie, a następnie otrzymywać potwierdzenia o opłaceniu każdej ze zdefiniowanej w ramach zlecenia płatności.

#### POST /orders/collect

Tworzy zlecenie

| Parametr     | Wymagane | Opis                                                      |
| ------------ | -------- | --------------------------------------------------------- |
| **currency** | tak      | waluta przelewu (aktualnie wspierane tylko PLN)           |
| **toName**   | tak      | nazwa odbiorcy                                            |
| **toIban**   | tak      | IBAN odbiorcy                                             |
| **metadata** | nie      | opcjonalne dane przekazywane przez FiberPay w odpowiedzi, |

Przykładowa odpowiedź:

```javascript
{
  "data": {
      "code": "w3taegy6fzuj",
      "status": "open",
      "type": "collect",
      "currency": "PLN",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "metadata": null,
      "createdAt": "2020-09-18 17:11:46",
      "updatedAt": "2020-09-18 17:11:46"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/collect/w3taegy6fzuj"
  }
}
```

#### POST /orders/collect/item

Tworzy pojedynczy przelew do wysłania w ramach całego zlecenia.

| Parametr           | Wymagane | Opis                                                                   |
| ------------------ | -------- | ---------------------------------------------------------------------- |
| **amount**         | tak      | kwota przelewu                                                         |
| **currency**       | tak      | waluta przelewu (aktualnie wspierane tylko PLN)                        |
| **parentCode**     | tak      | kod zlecenia nadrzędnego                                               |
| **description**    | tak      | tytuł przelewu                                                         |
| **metadata**       | nie      | dane przekazywane przez FiberPay w odpowiedzi                          |
| **callbackUrl**    | nie      | URL na który ma być wywołany callback                                  |
| **callbackParams** | nie      | parametry callbacka                                                    |
| **redirectUrl**    | nie      | URL, na który użytkownik zostanie przekierowany po dokonaniu płatności |

Przykładowa odpowiedź:

```javascript
{
  "data": {
      "code": "xwbmedu9",
      "status": "open",
      "type": "collect_item",
      "currency": "PLN",
      "amount": "100",
      "feeAmount": "0.50",
      "toName": "Michał",
      "parentCode": "p4rtzey3hnwv",
      "description": "testowy przelew",
      "metadata": null,
      "createdAt": "2020-09-18 17:13:42",
      "updatedAt": "2020-09-18 17:13:42",
      "redirect": "https://test.fiberpay.pl/order/xwbmedu9"
  },
  "invoice": {
      "amount": "100.50",
      "currency": "PLN",
      "iban": "PL123400001",
      "bban": "123400001",
      "description": "xwbmedu9"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/collect/item/xwbmedu9"
  }
}
```

#### GET /orders/collect/{code}

Pobranie informacji o całym zleceniu

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
      "code": "w3taegy6fzuj",
      "status": "open",
      "type": "collect",
      "currency": "PLN",
      "toName": "Michał",
      "toIban": "PL27114020040000300201355387",
      "metadata": null,
      "createdAt": "2020-09-18 17:11:46",
      "updatedAt": "2020-09-18 17:11:46"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/collect/w3taegy6fzuj"
  }
}
```

#### GET /orders/collect/item/{code}

Przykładowa odpowiedź serwera:

```javascript
{
  "data": {
      "code": "xwbmedu9",
      "status": "open",
      "type": "collect_item",
      "currency": "PLN",
      "amount": "100",
      "feeAmount": "0.50",
      "toName": "Michał",
      "parentCode": "p4rtzey3hnwv",
      "description": "testowy przelew",
      "metadata": null,
      "createdAt": "2020-09-18 17:13:42",
      "updatedAt": "2020-09-18 17:13:42",
      "redirect": "https://test.fiberpay.pl/order/xwbmedu9"
  },
  "invoice": {
      "amount": "100.50",
      "currency": "PLN",
      "iban": "PL123400001",
      "bban": "123400001",
      "description": "xwbmedu9"
  },
  "links": {
      "rel": "self",
      "href": "https://apitest.fiberpay.pl/1.0/orders/collect/item/xwbmedu9"
  }
}
```

Pobranie informacji o pojedynczej wpłacie

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

#### DELETE /orders/collect/item/{code}

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź:

```javascript
{
    "data": {
        "code": "xwbmedu9",
        "status": "cancelled",
        "type": "collect_item",
        "currency": "PLN",
        "amount": "100.0000",
        "feeAmount": "0.5000",
        "toName": "Michał",
        "parentCode": "p4rtzey3hnwv",
        "description": "testowy przelew",
        "metadata": null,
        "createdAt": "2020-09-18 17:13:42",
        "updatedAt": "2020-09-18 17:25:46",
        "redirect": "https://test.fiberpay.pl/order/xwbmedu9"
    },
    "invoice": [],
    "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/orders/collect/item/xwbmedu9"
    }
}
```

### FiberForward

Usługa dwóch powiązanych przekazów pieniężnych, pozwalająca na przyjęcie przekazu pieniężnego, a następnie opłacenie innego przekazu ze środków pierwszego.

* Source - osoba opłacająca przekaz
* Target - odbiorca przekazu
* Broker - pośrednik obsługujący osobę opłacającą oraz odbiorcę przekazu

#### POST /orders/forward

Utworzenie zlecenia. Parametry żądania:

| Parametr              | Wymagane | Opis                                                                                           |
| --------------------- | -------- | ---------------------------------------------------------------------------------------------- |
| **sourceAmount**      | tak      | całościowa kwota przekazu, decimal z maks. 2 miejscami po przecinku (np. 100.50)               |
| **targetAmount**      | tak      | kwota przekazu, która trafi do odbiorcy, decimal z maks. 2 miejscami po przecinku (np. 100.50) |
| **currency**          | tak      | waluta, aktualnie dostępne tylko PLN                                                           |
| **targetName**        | tak      | nazwa odbiorcy                                                                                 |
| **targetIban**        | tak      | IBAN odbiorcy                                                                                  |
| **brokerName**        | tak      | nazwa pośrednika                                                                               |
| **brokerIban**        | tak      | IBAN pośrednika                                                                                |
| **description**       | nie      | tytuł przelewu                                                                                 |
| **targetDescription** | tak      | opis dla odbiorcy                                                                              |
| **beforePaymentInfo** | nie      | dodatkowy komunikat wyświetlany przed płatnością                                               |
| **afterPaymentInfo**  | nie      | dodatkowy komunikat wyświetlany po płatności                                                   |
| **metadata**          | nie      | opcjonalne dane przekazywane przez FiberPay w callbacku                                        |
| **callbackUrl**       | nie      | URL na który ma być wywołany callback                                                          |
| **callbackParams**    | nie      | opcjonalne parametry callbacka                                                                 |

Przykładowa odpowiedź serwera:

```javascript
{
    "data": {
        "code": "xbwucsgfn5pa",
        "status": "defined",
        "type": "forward",
        "currency": "PLN",
        "targetAmount": "100.0000",
        "targetName": "Target",
        "targetIban": "PL12340000TARGET",
        "targetDescription": "targetDescription",
        "brokerAmount": "48.7500",
        "brokerName": "Broker",
        "brokerIban": "PL12340000BROKER",
        "feeAmount": "1.2500",
        "createdAt": "2020-09-16 18:43:37",
        "updatedAt": "2020-09-16 18:43:37"
    },
    "invoice": {
        "amount": "150.00",
        "currency": "PLN",
        "iban": "PL123400005",
        "description": "n9t6k4zw"
    },
    "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/orders/forward/xbwucsgfn5pa"
    }
}
```

#### GET /orders/forward/{code}

| Parametr | Wymagane | Opis         |
| -------- | -------- | ------------ |
| **code** | tak      | kod zlecenia |

Przykładowa odpowiedź serwera:

```javascript
{
    "data": {
        "code": "xbwucsgfn5pa",
        "status": "defined",
        "type": "forward",
        "currency": "PLN",
        "targetAmount": "100.0000",
        "targetName": "Target",
        "targetIban": "PL12340000TARGET",
        "targetDescription": "targetDescription",
        "brokerAmount": "48.7500",
        "brokerName": "Broker",
        "brokerIban": "PL12340000BROKER",
        "feeAmount": "1.2500",
        "createdAt": "2020-09-16 18:43:37",
        "updatedAt": "2020-09-16 18:43:37"
    },
    "invoice": {
        "amount": "150.00",
        "currency": "PLN",
        "iban": "PL123400005",
        "description": "n9t6k4zw"
    },
    "links": {
        "rel": "self",
        "href": "https://apitest.fiberpay.pl/1.0/orders/forward/xbwucsgfn5pa"
    }
}
```

## Sandbox (środowisko testowe)

Dostępne są tutaj dodatkowe metody ułatwiające sprawdzanie działalności systemu oraz automatyzacja testów.

#### Automatyzacja testów

* Sprawdzenie nowych przelewów w systemie - co minutę
* Uruchomienie przetwarzania zleceń po otrzymaniu dla nich wpłaty - co minutę
* Wysyłanie callbacków - co minutę
* Przetwarzanie zleceń typu Collect - co godzinę
* Przetworzenie przelewów wychodzących - co minutę

#### Dodatkowe ścieżki

#### POST /bankTransactions

Symulacja przychodzącego przelewu bankowego.

| Parametr        | Wymagane | Opis                                            |
| --------------- | -------- | ----------------------------------------------- |
| **amount**      | tak      | kwota przelewu                                  |
| **currency**    | tak      | waluta przelewu (aktualnie wspierane tylko PLN) |
| **fromName**    | tak      | nazwa nadawcy                                   |
| **fromIban**    | tak      | IBAN nadawcy                                    |
| **description** | tak      | tytuł przelewu (kod zlecenia)                   |

Przykładowa odpowiedź serwera:

```javascript
{
    "data": {
        "contractorName": "TEST",
        "contractorIban": "PL12340000TEST",
        "amount": 100.5,
        "currency": "PLN",
        "description": "kn8sf7amj6xy",
        "bankReferenceCode": "kn8sf7amj6xy",
        "operationCode": "kn8sf7amj6xy",
        "accountIban": "PL123400007",
        "bookedAt": "2020-09-18 13:30:28",
        "createdAt": "2020-09-18 13:30:28",
        "updatedAt": "2020-09-18 13:30:28"
    }
}
```
