---
  Zwiększ bezpieczeństwo swoich użytkowników dzięki autoryzacji operacji w
  dedykowanej aplikacji mobilnej
---

# FiberToken

## Informacje ogólne

Głównym zastosowaniem API jest zatwierdzanie utworzonych zleceń za pomocą zewnętrznego urządzenia zgodnie z wymaganiami autoryzacji dwuskładnikowej. API obsługuje komunikację pomiędzy usługą a urządeniem na którym dokonywana jest autoryzacja.

### Klucze API

Do korzystania z API konieczne jest wygenerowanie kluczy:

* jawnego (apiKey) - używanego do przesyłania w ramach żądań API
* tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

W celu uzyskania danych dostępowych niezbędnych do poprawnego korzystania z API należy skontaktować się bezpośrednio z usługodawcą.

### Nagłówek żądania

Każde żądanie do API powinno posiadać następujący nagłówek:

* `Api-Key` – wygenerowany klucz jawny

Każda odpowiedź API oraz żądanie zwrotne (callback) również posiada wskazany nagłówek.

### Ciało żądania

Ciało (body) każdego żądania HTTP powinno być kodowane jako token JWT z użyciem klucza tajnego (secret) jako sygnatury.

Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT są pod adresem: [https://jwt.io/](https://jwt.io/)

W przypadku niepoprawnego wykorzystania kluczy dostępowych serwer zwraca następujące błędy:

**STATUS 400 Bad Request**

```javascript
{
    "status": "ERROR",
    "error": "No Api Key provided"
}
```

```javascript
{
    "status": "ERROR",
    "error": "Api key invalid"
}
```

```javascript
{
    "status": "ERROR",
    "error": "Wrong signature"
}
```

### Mechanizm callbacków

FiberToken posiada mechanizm tzw. callbacków. Po zmianie statusu autoryzacji (Accept/Decline) lub w przypadku modyfikacji urządzenia system wywołuje żądanie HTTP (metoda POST) na wskazany uprzednio adres (callbackUrl), gdzie:

* w ciele (body) żądania zawarty jest token JWT z zaszyfrowanymi (opis powyżej) aktualnymi danymi zlecenia (payload),
* w nagłówku Api-Key żądania zawarty jest jawny klucz API

Przykładowy callback po odszyfrowaniu:

```javascript
{
    "type": "DeviceUpdate",
    "data": {
        "code": "ag2qndc1dusrzw",
        "name": "testEmail@mail.com",
        "status": "active",
        "callback_url": "https://fiberpay.pl/",
        "api_key": "ajHIscfrclNBjXZl",
        "created_at": "2021-09-30T11:00:34.000000Z",
        "updated_at": "2021-09-30T11:00:50.000000Z"
    }
}
```

## Opis usług

### POST /devices

Utworzenie nowego urządzenia autorazacyjnego. Parametry żądania:

| Parametr        | Wymagane | Typ          | Opis                                  |
| --------------- | -------- | ------------ | ------------------------------------- |
| **name**        | TAK      | String       | Nazwa nowego urządzenia               |
| **callbackUrl** | TAK      | URL (String) | URL na który ma być wywołany callback |

#### Przykładowe odpowiedzi serwera:

* **STATUS 200 OK**

```javascript
{
    "data": {
        "name": "testName",
        "callback_url": "http://fiberpay.pl",
        "code": "e56ydwovnshabc",
        "updated_at": "2021-09-29T12:10:52.000000Z",
        "created_at": "2021-09-29T12:10:52.000000Z"
    },
    "pair": {
        "pairing_code": "BSV3WNYZ",
        "expired_at": "2021-09-29T12:15:52.000000Z",
        "code": "kcsjynza5ab7b3cp",
        "updated_at": "2021-09-29T12:10:52.000000Z",
        "created_at": "2021-09-29T12:10:52.000000Z"
    }
}
```

W odpowiedzi serwera otrzymujemy kod (data.code) niezbędny do wykonania kolejnych żądań. Kod (pair.code) to identyfikator parowania, natomiast (pair.pairing\_code) jest kodem który należy podać na urządzeniu w celu zatwierdzenia parowania.

* **STATUS 400 Bad Request**

```javascript
{
    "status": "ERROR",
    "message": "The given data was invalid.",
    "errors": {
        "name": "The name field is required."
    }
}
```

```javascript
{
    "status": "ERROR",
    "message": "The given data was invalid.",
    "errors": {
        "callbackUrl": "The callback url field is required."
    }
}
```

### POST /devices/pair/renew

Odnowienie parowania urządzenia. Parametry żądania:

| Parametr | Wymagane | Typ    | Opis                           |
| -------- | -------- | ------ | ------------------------------ |
| **code** | TAK      | String | Kod identyfikacyjny urządzenia |

* **STATUS 200 OK**

```javascript
{
    "data": {
        "code": "nt28erab17xjdz",
        "name": "testName",
        "status": "new",
        "callback_url": "http://fiberpay.pl",
        "api_key": null,
        "created_at": "2021-09-29T13:08:00.000000Z",
        "updated_at": "2021-09-29T13:08:00.000000Z",
        "pairing": {
            "code": "hk3j2pqv1xc9ea6n",
            "pairing_code": "DMTWEYR4",
            "expired_at": "2021-09-29T13:23:09.000000Z",
            "created_at": "2021-09-29T13:08:00.000000Z",
            "updated_at": "2021-09-29T13:18:09.000000Z"
        }
    },
    "pair": {
        "code": "hk3j2pqv1xc9ea6n",
        "pairing_code": "DMTWEYR4",
        "expired_at": "2021-09-29T13:23:09.000000Z",
        "created_at": "2021-09-29T13:08:00.000000Z",
        "updated_at": "2021-09-29T13:18:09.000000Z"
    }
}
```

* **STATUS 404 Not Found**

```javascript
{
    "status": "ERROR",
    "error": "Device with that code not found"
}
```

```javascript
{
    "status": "ERROR",
    "error": "You have no permission for this device"
}
```

### POST /devices/{code}/auth

Utworzenie zlecenia do autoryzacji. W miejscu {code} należy podać kod identyfikacyjny urządzenia na który ma zostać wysłane zapytanie. Parametry żądania:

| Parametr | Wymagane | Typ    | Opis                                |
| -------- | -------- | ------ | ----------------------------------- |
| **data** | TAK      | String | Krótki opis zlecenia do autoryzacji |

* **STATUS 201 CREATED**

```javascript
{
    "data": "\"Prosba o zatwierdzenie zlecenia\"",
    "code": "gzu9rm31yqx62tk",
    "updated_at": "2021-09-29T14:22:25.000000Z",
    "created_at": "2021-09-29T14:22:25.000000Z",
    "device": {
        "code": "nt28erab17xjdz",
        "name": "testName",
        "status": "new",
        "callback_url": "http://fiberpay.pl",
        "api_key": null,
        "created_at": "2021-09-29T13:08:00.000000Z",
        "updated_at": "2021-09-29T13:08:00.000000Z"
    }
}
```

* **STATUS 400 Bad Request**

```javascript
{
    "status": "ERROR",
    "message": "The given data was invalid.",
    "errors": {
        "data": "The data field is required."
    }
}
```

* **STATUS 404 Not Found**

```javascript
{
    "status": "ERROR",
    "error": "Device with that code not found"
}
```

```javascript
{
    "status": "ERROR",
    "error": "You have no permission for this device"
}
```
