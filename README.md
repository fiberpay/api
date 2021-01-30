# Informacje ogólne

## Adresy serwerów
Produkcyjne API: https://api.fiberpay.pl/1.0
frontend produkcyjny: https://fibepay.pl

API testowe: https://apitest.fiberpay.pl/1.0
frontend testowy: https://test.fiberpay.pl

Są to zupełnie rozdzielone środowiska (łącznie z infrastrukturą). **Klucze API z jednego środowiska nie będą działać w drugim**.

## Klucze API
Do korzystania z API konieczne jest wygenerowanie kluczy:
- jawnego (publicKey) - używanego do przesyłania w ramach żądań API
- tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

## Nagłówki 
Każde żądanie do API powinno posiadać następujące nagłówki:
- Content-Type - application/json
- Accept - application/json
- X-API-Key – wygenerowany klucz jawny
- X-API-Nonce – nonce w postaci liczba naturalnej, każdy następny nonce powinien być większy od poprzedniego (przykładowo można użyć timestampa lub zaokrąglonego microtime jeśli używamy PHP)
- X-API-Method-And-Uri – informacja o wywoływanej metodzie HTTP oraz URI
  - np. POST /api/order/create/massoutbound
  - np. POST /api/order/create/massoutbound/item
- X-API-Signature – podpis z użyciem skrótu **sha512** stworzonego przy użyciu **_‘message’_**, **_‘nonce’_**, **_‘apikey’_**, **_‘requestBody’_** i **_‘secretkey’_**

### Instrukcja utworzenia X-API-Signature
Należy utworzyć połączony ciąg znaków (concatenated string) składający się z message, nonce, apikey oraz requestBody (kolejność ma znaczenie!), a następnie 
wygenerować skrót **sha512** z utworzonego ciągu używając **secretkey**.
- przykładowa impelmentacja w PHP:    
```php
$implodeParams = implode( ‘’, [$message, $nonce, $apikey, $requestBody]);
hash_hmac(‘sha512’, $implodeParams, $secretkey)
```
$requestBody to ciało (body) żądania HTTP. Gdy go brak to powinien zostać użyty pusty string ('').

## Mechanizm callbacków

FiberPay posiada mechanizm tzw. callbacków. Poktualizacji zlecenia system FiberPay może wywołać żądanie HTTP na wskazany uprzednio adres, gdzie:
- w ciele (body) żądania będzie zawarty token JWT z aktualnymi danymi zlecenia,
- w nagłówku (header) żądania będzie zawarty jawny klucz API, wskazujacy który klucz tajny został wykorzystany do utworzenia JWT

Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT są pod adresem: https://jwt.io/  


# Opis usług

## FiberDirect

Usługa pojedynczego przekazu pieniężnego na konkretne konto bankowe. Platforma korzystająca z API może założyć zlecenie, a następnie dostać potwierdzenie, gdy dany przekaz zostanie opłacony, a także gdy FiberPay wykonana już dany przekaz na wskazane konto.

### POST /orders/direct
Utworzenie zlecenia. Parametry żądania:
Parametr | Opis
------------ | -------------
**amount** | (wymagane) kwota przekazu, decimal z maks. 2 miejscami po przecinku (np. 100.50)
**currency** | (wymagane) waluta, aktualnie dostępne tylko PLN
**toName** | (wymagane) nazwa odbiorcy,
**toIban** | (wymagane) IBAN odbiorcy,
**description** | tytuł przelewu,
**metadata** | opcjonalne dane przekazywane przez FiberPay w odpowiedzi,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

Przykładowa odpowiedź serwera
```json
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

### GET /orders/direct/{code}
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

### DELETE /orders/direct/{code}
Anulowanie wcześniej utworzonego zlecenia (możliwe tylko dla jeszcze nieopłaconych zleceń).
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia
```json
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

## FiberSplit
Usługa gdzie FiberPay może wykonać wiele przekazów pieniężnych w zamian za jedną opłatę (np. 100 przelewów na wybrane konta bankowe). Platforma korzystająca z API może założyć zlecenie, a następnie dostać potwierdzenie, gdy dany zostanie ono płacone, a także dostać informacje dot. statusu każdego ze zleconych przekazów pieniężnych.

Aby skorzystać z usługi należy:
- założyć zlecenie (POST /orders/massoutbound)
- dodać poszczególne przekazy pieniężne (POST /orders/massoutbound/item)
- zakończyć definicję zlecenia (PUT /orders/massoutbound/{code}/define)
- opłacić utworzone zlecenie

### POST /orders/split
Tworzy zlecenie

Parametr | Opis
------------ | -------------
**currency** | (wymagane) waluta zlecenia, aktualnie wspieramy tylko PLN
**metadata** | opcjonalne dane przekazywane przez FiberPay w callbacku,

Przykładowa odpowiedź
```json
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


### POST /orders/split/item
Tworzy pojedynczy przelew do wysłania w ramach całego zlecenia.

Parametr | Opis
------------ | -------------
**amount** | (wymagane) kwota przelewu
**currency** | (wymagane) waluta przelewu (aktualnie wspierane tylko PLN)
**parentCode** | (wymagane) kod zlecenia nadrzędnego
**toName** | (wymagane) nazwa odbiorcy
**toIban** | (wymagane) IBAN odbiorcy
**description** | (wymagane) tytuł przelewu
**metadata** | opcjonalne dane przekazywane przez FiberPay w odpowiedzi,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

Przykładowa odpowiedź
```json
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

### PUT /orders/split/{code}/define
Kończy tworzenie orderu (zamyka definicję).

Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

Przykładowa odpowiedź:
```json
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
### GET /orders/split/{code}
Pobranie informacji o całym zleceniu
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

### GET /orders/split/item/{code}
Pobranie informacji o poszczególnym przekazie pieniężnym
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia



## FiberCollect


### POST /orders/collect
Tworzy zlecenie

Parametr | Opis
------------ | -------------
**currency** | (wymagane) waluta przelewu (aktualnie wspierane tylko PLN)
**toName** | (wymagane) nazwa odbiorcy
**toIban** | (wymagane) IBAN odbiorcy
**metadata** | opcjonalne dane przekazywane przez FiberPay w odpowiedzi,

Przykładowa odpowiedź:
```json
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

### POST /orders/collect/item
Tworzy pojedynczy przelew do wysłania w ramach całego zlecenia.

Parametr | Opis
------------ | -------------
**amount** | (wymagane) kwota przelewu
**currency** | (wymagane) waluta przelewu (aktualnie wspierane tylko PLN)
**parentCode** | (wymagane) kod zlecenia nadrzędnego
**description** | (wymagane) tytuł przelewu
**metadata** | opcjonalne dane przekazywane przez FiberPay w odpowiedzi,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

Przykładowa odpowiedź:
```json
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

### GET /orders/collect/{code}
Pobranie informacji o całym zleceniu
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

### GET /orders/collect/item/{code}
Pobranie informacji o pojedynczej wpłacie
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

### DELETE /orders/collect/item/{code}
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

Przykładowa odpowiedź:
```json
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

## FiberForward

Usługa dwóch powiązanych przekazów pieniężnych, pozwalająca na przyjęcie przekazu pieniężnego, a następnie opłacenie innego przekazu ze środków pierwszego.

- Source - osoba opłacająca przekaz
- Target - odbiorca przekazu
- Broker - pośrednik (poza FiberPayem)

### POST /orders/forward
Utworzenie zlecenia. Parametry żądania:
Parametr | Opis
------------ | -------------
**sourceAmount** | (wymagane) całościowa kwota przekazu, decimal z maks. 2 miejscami po przecinku (np. 100.50)
**targetAmount** | (wymagane) kwota przekazu, która trafi do odbiorcy, decimal z maks. 2 miejscami po przecinku (np. 100.50)
**currency** | (wymagane) waluta, aktualnie dostępne tylko PLN
**targetName** | (wymagane) nazwa odbiorcy,
**targetIban** | (wymagane) IBAN odbiorcy,
**brokerName** | (wymagane) nazwa pośrednika,
**brokerIban** | (wymagane) IBAN pośrednika,
**description** | tytuł przelewu,
**targetDescription** | (wymagane) Opis dla odbiorcy,
**metadata** | opcjonalne dane przekazywane przez FiberPay w callbacku,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

Przykładowa odpowiedź serwera:
```json
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

### GET /orders/forward/{code}
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

### GET /settlements

### GET /settlements/{code}

## Sandbox (środowisko testowe)

Dostępne są tutaj dodatkowe metody ułatwiające sprawdzanie działalności systemu oraz automatyzacja testów.

### Automatyzacja testów
- Sprawdzenie nowych przelewów w systemie - co minutę
- Uruchomienie przetwarzania zleceń po otrzymaniu dla nich wpłaty - co minutę
- Wysyłanie callbacków - co minutę
- Przetwarzanie zleceń typu Collect - co godzinę
- Przetworzenie przelewów wychodzących - co minutę

### Dodatkowe ścieżki

### POST /bankTransactions

Symulacja przychodzącego przelewu bankowego.

Parametr | Opis
------------ | -------------
**amount** | (wymagane) kwota przelewu
**currency** | (wymagane) waluta przelewu (aktualnie wspierane tylko PLN)
**fromName** | (wymagane) nazwa nadawcy
**fromIban** | (wymagane) IBAN nadawcy
**description** | (wymagane) tytuł przelewu (kod zlecenia)

Przykładowa odpowiedź serwera:
```json
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
