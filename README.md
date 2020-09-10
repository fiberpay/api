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
   "data":{
      "code":"mju35wnebp6z",
      "status":"open",
      "type":"split",
      "currency":"PLN",
      "metadata":null,
      "createdAt":"2020-09-10 02:00:59",
      "updatedAt":"2020-09-10 02:00:59"
   },
   "links":{
      "rel":"self",
      "href":"https:\/\/apitest.fiberpay.pl\/1.0\/orders\/split\/mju35wnebp6z"
   }
}
```


### POST /orders/split/item

### PUT /orders/split/{code}/define

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

### GET /settlements

### GET /settlements/{code}

