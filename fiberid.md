---
Weryfikacja tożsamości użytkowników
---

# FiberID

## Informacje ogólne

### Adresy serwerów

| Funkcjonalność     | Środowisko  | URL                                                              |
| ------------------ | ----------- | ---------------------------------------------------------------- |
| Serwer API         | Produkcyjne | [https://apiver.fiberpay.pl/api](https://apiver.fiberpay.pl/api) |
| Panel deweloperski | Produkcyjne | [https://verify.fiberpay.pl](https://verify.fiberpay.pl)         |
| Serwer API         | Testowe     | [https://apiver3.fiberpay.pl/api/](https://apiver3.fiberpay.pl/) |
| Panel deweloperski | Testowe     | [https://vertest3.fiberpay.pl/](https://vertest3.fiberpay.pl/)   |

### Klucze API

Do korzystania z API konieczne jest wygenerowanie kluczy za pomocą panelu deweloperskiego:

* jawnego (apiKey) - używanego do przesyłania w ramach żądań API
* tajnego (secretKey) - używanego do podpisywania żądań (nigdy nie powinien by przesyłany lub ujawniany)

### Nagłówki

Każde żądanie do API powinno posiadać następujące nagłówki:

* Accept - application/json
* API-Key – wygenerowany klucz jawny

Każda odpowiedź API oraz żądanie zwrotne (callback) również posiada wskazane powyżej nagłówki.

### Kodowanie i szyfrowanie

#### Token JWT

Ciało (body) każdego żądania HTTP powinno być kodowane jako token JWT z użyciem klucza tajnego (secret) jako sygnatury. W przypadku metody GET token powinien zawierać pusty string("") i być przekazany w nagłówku Authorization w następującej formie: "Bearer {token}".

Informacje jak odkodować lub jakich bibliotek użyć do obsługi tokenu JWT są pod adresem: [https://jwt.io/](https://jwt.io/)

#### Szyfrowanie

Ciało odpowiedzi z API jest zawsze kodowane jako token JWT podpisany z użyciem klucza tajnego, a **dane (payload) są dodatkowo szyfrowane za pomocą AES-256 w trybie CBC**.

Do odszyfrowania należy użyć klucza tajnego (secret) oraz wektora inicjującego (initial vector - iv, jawnego i unikalnego dla każdej odpowiedzi serwera). Jest on przekazywany w tokenie.

**Należy pamiętać, że biblioteki służące do odszyfrowania wymagają podania klucza i wektora inicjującego w zapisie binarnym**

Przykładowa implementacja w PHP:

```php
openssl_decrypt($ciphertext, 'aes-256-cbc', hex2bin($secret), 0, hex2bin($iv));
```

Przykładowa implementacja w NodeJS:

```javascript
const crypto = require("crypto");

const decrypt = (ciphertext, secret, iv) => {
    const decipher = crypto.createDecipheriv(
        "aes-256-cbc",
        Buffer.from(secret, "hex"),
        Buffer.from(iv, "hex")
    );
    let plaintext = decipher.update(ciphertext, "base64", "utf8");
    plaintext += decipher.final("utf8");
    return plaintext;
};
```

### Mechanizm callbacków

FiberId posiada mechanizm tzw. callbacków. Po zmianie statusu weryfikacji system wywołuje żądanie HTTP (metoda POST) na wskazany uprzednio adres (callbackUrl), gdzie:

* w ciele (body) żądania zawarty jest token JWT z zaszyfrowanymi (opis powyżej) aktualnymi danymi zlecenia (payload),
* w nagłówku API-Key żądania zawarty jest jawny klucz API

Kształt ciała (body) żądania jest identyczny jak w przypadku odpowiedzi systemu na żądanie GET /orders/{code}

## Opis usług

### POST /orders

Utworzenie zlecenia weryfikacyjnego. Parametry żądania:

| Parametr        | Opis                                                                                                                                                                                                                       |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **type**        | (wymagane) schemat formularza wykorzystywanego podczas weryfikacji. Aktualnie wspierane: 'individual', 'sole\_proprietorship' , 'company'. W chwili obecnej tylko typ 'individual' jest obsługiwany w pełni automatycznie. |
| **description** | (wymagane) Opis wskazujący cel, zleceniodawcę weryfikacji                                                                                                                                                                  |
| **callbackUrl** | (wymagane) URL na który ma być wywołany callback.                                                                                                                                                                          |
| **redirectUrl** | (wymagane) URL na który ma zostać wykonane przekierowanie po zakończeniu weryfikacji                                                                                                                                       |

Przykładowa odpowiedź serwera (po odkodowaniu tokenu)

```javascript
{
    "code": "fq9z2gmpr4bc",
    "status": "new",
    "description": "Weryfikacja danych jest prowadzona na potrzeby serwisu fiberpay.pl.",
    "callbackUrl": "https://api.yourpage.pl/callbacks/fiber-id",
    "redirectUrl": "https://yourpage.pl/verifications",
    "url": "https://api.fiber.id/orders/fq9z2gmpr4bc",
    "createdAt": "2021-01-07T12:06:46.000000Z"
}
```

| Parametr        | Opis                                                                                    |
| --------------- | --------------------------------------------------------------------------------------- |
| **code**        | kod zlecenia - np. do wywoływań sprawdzających jego status                              |
| **status**      | status zlecenia                                                                         |
| **description** | opis zlecenia                                                                           |
| **callbackUrl** | URL, na który zostanie wywołany callback                                                |
| **redirectUrl** | URL, na który ma zostać wykonane przekierowanie po zakończeniu weryfikacji              |
| **url**         | URL, na który powinieneś przekierować użytkownika w celu realizacji procesu weryfikacji |
| **createdAt**   | data i czas utworzenia zlecenia                                                         |

Po utworzeniu zlecenia Twoja aplikacja powinna przekierować użytkownika na adres FiberID, który zlecenie zwróci w parametrze `url`. Pod tym adresem zostanie przeprowadzona dalsza część procesu weryfikacji użytkownika. Po jej zakończeniu FiberID przekieruje użytkownika z powrotem do Twojej aplikacji - na adres URL, który podałeś w parametrze `redirectUrl` przy tworzeniu zlecenia.

Niezależnie od przekierowania użytkownika pod wskazany adres, FiberID zrealizuje żądanie HTTP POST pod adres `callbackUrl`, który podałeś przy tworzeniu zlecenia. Żądanie to będzie zawierało wyniki procesu weryfikacji.

Przykładowa treść przekazywana w żądaniu zwrotnym (callback)

```javascript
{
  "code": "zh4t6xj2gpc8",
  "status": "accepted",
  "type": "individual",
  "description": "Weryfikacja prowadzona na potrzeby serwisu ABC",
  "callbackUrl": "https://apidev3.fiberpay.pl/ab/partners/callbacks",
  "redirectUrl": "https://beta.zaplatomat.pl/user/identity",
  "createdAt": "2021-07-22T11:58:27.000000Z",
  "data": {
    "firstName": "Adam",
    "lastName": "Kowalski",
    "isNotPoliticallyExposed": true,
    "country": "Polska",
    "address": "Marszałkowska 1",
    "postalCode": "00-001",
    "city": "Warszawa",
    "phoneNumber": "+48 121212121",
    "idNumber": "CBD123456",
    "idExpirationDate": "31-07-2029",
    "nationality": "Polska",
    "personalIdentityNumber": "ABC1111111",
    "sourceOfIncome": "Własna działalność",
    "bankAccount": "PL66114020040000320111111123"
  }
}
```

### GET /orders/{code}

Pobranie informacji o bieżącym statusie zlecenia.

**Ciało (body) żądania powinien stanowić pusty string("")** zakodowany jako token JWT podpisany za pomocą klucza tajnego (secret).

#### Przykładowe odpowiedzi serwera (po odkodowaniu i odszyfrowaniu payloadu)

Dla zlecenia zweryfikowanego pozytywnie

```javascript
{
    "payload": {
        "code": "fq9z2gmpr4bc",
        "status": "verified",
        "verification": {
            "identity": {
                "status": "accepted",
                "method": "id_document"
            },
            "proofOfResidence": {
                "status": "accepted"
            }
        },
        "form": {
            "answers": {
                "firstName": "Jan",
                "lastName": "Kowalski",
                "isNotPoliticallyExposed": true,
                "country": "Polska",
                "personalIdentityNumber": "01234567890",
                "address": "ul. Nowy Świat 1/10",
                "postalCode": "00-001",
                "city": "Warszawa",
                "idNumber": "XYZ123456",
                "idExpirationDate": "2021-01-08",
                "sourceOfIncome": "Wynagrodzenie"
            },
            "status": "submitted"
        }
    },
    "iss": "Verification",
    "iat": 1610323876,
    "iv": "bcdb04fdb9b8d9715fe0075c1fbc0519"
}
```

Dla zlecenia z negatywną weryfikacją tożsamości

```javascript
{
    "payload": {
        "code": "fq9z2gmpr4bc",
        "status": "verified",
        "verification": {
            "identity": {
                "status": "rejected",
                "method": "id_document"
            },
            "proofOfResidence": {
                "status": "accepted"
            }
        },
        "form": {
            "answers": {
                "firstName": "Jan",
                "lastName": "Kowalski",
                "isNotPoliticallyExposed": true,
                "country": "PL",
                "personalIdentityNumber": "01234567890",
                "address": "ul. Nowy Świat 1/10",
                "postalCode": "00-001",
                "city": "Warszawa",
                "idNumber": "XYZ123456",
                "idExpirationDate": "2021-01-08",
                "sourceOfIncome": "Wynagrodzenie"
            },
            "status": "submitted"
        }
    },
    "iss": "Verification",
    "iat": 1610323876,
    "iv": "bcdb04fdb9b8d9715fe0075c1fbc0519"
}
```

##
