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

## Direct transfer

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
**metadata** | opcjonalne dane przekazywane przez FiberPay w callbacku,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

### GET /orders/direct/{code}
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

Przykładowa odpowiedź serwera
```json
{}
```

### DELETE /orders/direct/{code}
Anulowanie wcześniej utworzonego zlecenia (możliwe tylko dla jeszcze nieopłaconych zleceń).
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia
```json
{}
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

Przykładowa odpowiedź
```json
{
  "data": {
    "code": "mju35wnebp6z",
    "status": "open",
    "type": "split",
    "currency": "PLN",
    "metadata": null,
    "createdAt": "2020-09-10 02:00:59",
    "updatedAt": "2020-09-10 02:00:59"
  },
  "links": {
    "rel": "self",
    "href": "https:\/\/apitest.fiberpay.pl\/1.0\/orders\/split\/mju35wnebp6z"
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

Przykładowa odpowiedź
```json
{
  "data": {
    "code": "ezumcdag",
    "description": "testowy przelew",
    "status": "open",
    "type": "splitItem",
    "currency": "PLN",
    "amount": "100",
    "feeAmount": "1.50",
    "toName": "Jan Kowalski",
    "toIban": "PL109023980000000143071844",
    "parentCode": "mju35wnebp6z",
    "metadata": null,
    "createdAt": "2020-09-10 02:08:25",
    "updatedAt": "2020-09-10 02:08:25"
  },
  "links": {
    "rel": "self",
    "href": "https:\/\/apitest.fiberpay.pl\/1.0\/orders\/split\/item\/ezumcdag"
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
    "code": "mju35wnebp6z",
    "status": "defined",
    "type": "split",
    "currency": "PLN",
    "metadata": null,
    "createdAt": "2020-09-10 02:00:59",
    "updatedAt": "2020-09-10 02:26:05",
    "payment": {
      "amount": "203.0000",
      "iban": "PL123400003",
      "description": "mju35wnebp6z"
    }
  },
  "links": {
    "rel": "self",
    "href": "https:\/\/apitest.fiberpay.pl\/1.0\/orders\/split\/mju35wnebp6z"
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

### POST /orders/collect/item

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

## Forward transfer

Usługa dwóch powiązanych przekazów pieniężnych, pozwalająca na przyjęcie przekazu pieniężnego, a następnie opłacenie innego przekazu ze środków pierwszego.

Source - osoba opłacająca przekaz
Target - Odbiorca przekazu
Broker - Pośrednik (poza FiberPayem)

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
**metadata** | opcjonalne dane przekazywane przez FiberPay w callbacku,
**callbackUrl** | URL na który ma być wywołany callback
**callbackParams** | opcjonalne parametry callbacka

### GET /orders/forward/{code}
Parametr | Opis
------------ | -------------
**code** | (wymagane) kod zlecenia

Przykładowa odpowiedź serwera
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

### GET /settlements

### GET /settlements/{code}

