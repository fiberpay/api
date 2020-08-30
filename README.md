# FiberPay API dokumentacja


## Adresy serwerów
Produkcyjne API: https://api.fiberpay.pl 
Frontend produkcyjny: https://fibepay.pl

API testowe: https://apitest.fiberpay.pl
Frontend testowy: https://test.fiberpay.pl

Są to zupełnie rozdzielone środowiska (łącznie z infrastrukturą). **Klucze API z jednego środowiska nie będą działać w drugim**.


## Nagłówki 

- Content-Type - application/json
- X-API-Key – Twój wygenerowany klucz publiczny
- X-API-Nonce – Rosnący integer np. timestamp. (timestamp powinien być w milisekundach lub microsekundach)
- X-API-Message – Wymaga metody Http oraz URI
  - np. POST /api/order/create/massoutbound
  - np. POST /api/order/create/massoutbound/item
- X-API-Signature – Wymaga klucza HASH HMAC z wykorzystaniem algorytmu **sha512** stworzonego przy użyciu **_‘message’_**, **_‘nonce’_**, **_‘apikey’_**, **_‘requestBody’_** i **_‘secretkey’_**

## Instrukcja utworzenia X-API-Signature
Stwórz połączony ciąg znaków(**concatenated string**) składający się z message, nonce, apikey oraz requestBody (kolejność ma znaczenie!).

Następnie wygeneruj klucz HMAC używając **sha512** z **concatenated string** oraz **secretkey**.
- np. w php:    
	  $implodeParams = implode( ‘’, [$message, $nonce, $apikey, $requestBody]);
	  hash_hmac(‘sha512’, $implodeParams, $secretkey)

$requestBody to dane zawarte w request. Gdy ich brak to powinien być pustym stringiem ('').

# Callbacks

## Headers
W headers’ach będzie wysłany API-Key, który jest wygenerowanym kluczem publicznym. 

## Data
Przesyłane informacje zawarte są w postaci tokenu JWT, którego odkodowanie daje możliwość zweryfikowania poprawności danych oraz ich autentyczność.  
Do odkodowania zawartości należy użyć sekretnego klucza, czyli odpowiednika wysyłanego w headers klucza API.  
Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT znajdziesz pod adresem: https://jwt.io/  

#### Dane po odkodowaniu:  
	{
		payload: {dane zależne od rodzaju zamówienia},
		iss: „Fiberpay”,
		iat: timestamp
	}

#### INBOUND
Dane wysyłane przez Fiberpay:   

	{
		type: inbound_order_item_received
		data: {
			orderItem: {object}
			transaction:{
				type: type bankTransaction,
				data: {object}
			}
		}
	}

Callback jest wysyłany gdy poprawny przelew związany z order_item został odebrany.

### OUTBOUND
Dane wysyłane przez Fiberpay:

	{
		type: outbound_order_item_sent
		data: {
			orderItem: {object}
			transaction:{
				type: type bankTransaction,
				data: {object}
			}
		}
	}

Callback jest wysyłany gdy Fiberpay wykona przelew związany z order_item.
