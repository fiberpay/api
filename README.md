# FiberPay API dokumentacja

## Adresy serwerów
Produkcyjne API: https://api.fiberpay.pl 
Frontend produkcyjny: https://fibepay.pl

API testowe: https://apitest.fiberpay.pl
Frontend testowy: https://test.fiberpay.pl

Są to zupełnie rozdzielone środowiska (łącznie z infrastrukturą). **Klucze API z jednego środowiska nie będą działać w drugim**.

## Klucze API
Do korzystania z API konieczne jest wygenerowanie kluczy:
- jawnego (publicKey) - używanego do przesyłania w ramach żądań API
- tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

## Nagłówki 
Każde żądanie do API powinno posiadać następujące nagłówki.

- Content-Type - application/json
- X-API-Key – wygenerowany klucz jawny
- X-API-Nonce – nonce w postaci liczba naturalnej, każdy następny nonce powinien być większy od poprzedniego (przykładowo można użyć zaokrąglonego microtime lub timestampa)
- X-API-Message – informacja o wywoływanej metodzie HTTP oraz URI
  - np. POST /api/order/create/massoutbound
  - np. POST /api/order/create/massoutbound/item
- X-API-Signature – podpis z użyciem skrótu **sha512** stworzonego przy użyciu **_‘message’_**, **_‘nonce’_**, **_‘apikey’_**, **_‘requestBody’_** i **_‘secretkey’_**

### Instrukcja utworzenia X-API-Signature
Należy utworzyć połączony ciąg znaków (concatenated string) składający się z message, nonce, apikey oraz requestBody (kolejność ma znaczenie!), a następnie 
wygenerować skrót **sha512** z utworzonego ciągu używając **secretkey**.
- przykładowa impelmentacj w PHP:    
```php
	  $implodeParams = implode( ‘’, [$message, $nonce, $apikey, $requestBody]);
	  hash_hmac(‘sha512’, $implodeParams, $secretkey)
```

$requestBody to dane zawarte w request. Gdy ich brak to powinien zostać użyty pusty string ('').

## Mechanizm callbacków

FiberPay posiada mechanizm tzw. callbacków. W skrócie to po aktualizacji zlecenia system FiberPay może wywołać żądanie HTTP na wskazany uprzednio adres, gdzie:
- w ciele (body) żądania będzie zawarty token JWT z aktualnymi danymi zlecenia,
- w nagłówku (header) żądania będzie

### Nagłówki
W nagłówku żadania HTTP headers’ach jest wywyłany jawny klucz API wysłany API-Key, który jest wygenerowanym kluczem jawnym. 

### Dane
Przesyłane informacje zawarte są w postaci tokenu JWT, którego odkodowanie daje możliwość zweryfikowania poprawności danych oraz ich autentyczność.  
Do odkodowania zawartości należy użyć tajnego klucza, czyli odpowiednika wysyłanego w headers klucza API.  
Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT znajdziesz pod adresem: https://jwt.io/  

Dane po odkogowaniu mają postać:
```json
	{
		payload: {dane zależne od rodzaju zamówienia},
		iss: "Fiberpay",
		iat: timestamp
	}
```

