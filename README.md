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
- X-API-Nonce – nonce w postaci liczba naturalnej, każdy następny nonce powinien być większy od poprzedniego (przykładowo można użyć zaokrąglonego microtime lub timestampa)
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

FiberPay posiada mechanizm tzw. callbacków. W skrócie to po aktualizacji zlecenia system FiberPay może wywołać żądanie HTTP na wskazany uprzednio adres, gdzie:
- w ciele (body) żądania będzie zawarty token JWT z aktualnymi danymi zlecenia,
- w nagłówku (header) żądania będzie

### Nagłówki
W nagłówku żadania HTTP headers’ach jest wywyłany jawny klucz API wysłany API-Key, który jest wygenerowanym kluczem jawnym. 

### Dane
Przesyłane informacje mają postać tokenu JWT, którego odkodowanie daje możliwość zweryfikowania poprawności danych oraz ich autentyczność.  
Do odkodowania zawartości należy użyć tajnego klucza, odpowiedniego dla jawnego klucza API, który został wysłany w nagłówku żądania.  
Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT są pod adresem: https://jwt.io/  


# Opis usług

## Direct transfer

Usługa pojedyńczego przekazu pieniężnego na konkretne konto bankowe. Platforma korzystająca z API może założyć zlecenie, a następnie dostać potwierdzenie, gdy dany przekaz zostanie opłacony, a także gdy FiberPay wykonana już dany przekaz na wskazane konto.

### POST /orders/directtransfer
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


### GET /orders/directtransfer/{code}

### DELETE /orders/directtransfer/{code}


## Mass outbound transfers
Usługa gdzie FiberPay może wykonać wiele przekazów pieniężnych w zamian za jedną opłatę (np. 100 przelewów na wybrane konta bankowe). Platforma korzystająca z API może założyć zlecenie, a następnie dostać potwierdzenie, gdy dany zostanie ono płacone, a także dostać informacje dot. statusu każdego ze zleconych przekazów pieniężnych.


## Mass inbound transfers



