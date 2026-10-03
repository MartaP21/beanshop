# Przepływ: od dodania produktu do złożenia zamówienia

Diagram obejmuje zalogowanego klienta. Endpointy koszyka wymagają uwierzytelnienia.

```mermaid
flowchart TD
    A([Klient zalogowany]) --> B[POST /api/cart/items<br/>productId, quantity]
    B --> C{Produkt istnieje?}
    C -- "nie" --> C1[404 NOT_FOUND]
    C -- "tak" --> D{Ilość <= 10<br/>i <= stan magazynowy?}
    D -- "nie" --> D1[400 MAX_QTY<br/>lub 409 OUT_OF_STOCK]
    D -- "tak" --> E[Dodanie albo zwiększenie pozycji<br/>Odpowiedź zawiera summary]

    E --> F{Zmiana koszyka?}
    F -- "tak" --> G[PATCH /api/cart/items/:productId<br/>lub DELETE /api/cart/items/:productId]
    G --> H[Przeliczenie widoku koszyka<br/>i summary]
    H --> F
    F -- "nie" --> I{Kod rabatowy?}
    I -- "tak" --> J[POST /api/cart/discount<br/>code]
    J --> K{Kod poprawny<br/>i warunki spełnione?}
    K -- "nie" --> K1[422 CODE_UNKNOWN,<br/>CODE_EXPIRED lub CODE_MIN_SUBTOTAL]
    K -- "tak" --> L[Zastosowanie kodu<br/>i przeliczenie summary]
    K1 --> I
    L --> I
    I -- "nie" --> M{Wybór dostawy?}
    M -- "tak" --> N[PUT /api/cart/shipping<br/>STANDARD lub EXPRESS]
    N --> O{Metoda poprawna?}
    O -- "nie" --> O1[400 VALIDATION]
    O -- "tak" --> P[Zapisanie dostawy<br/>i przeliczenie summary]
    O1 --> M
    P --> M
    M -- "nie" --> Q[POST /api/orders]
    Q --> R{Koszyk niepusty<br/>i towar nadal dostępny?}
    R -- "nie" --> R1[400 EMPTY_CART<br/>lub 409 OUT_OF_STOCK]
    R -- "tak" --> S[Utworzenie zamówienia<br/>status NEW]
    S --> T[Odjęcie towaru ze stanu]
    T --> U[Wyczyszczenie koszyka]
    U --> V([201: złożone zamówienie])
```

## Źródła

- `docs/architektura.md`, sekcja „Przepływ „złóż zamówienie””.
- `src/routes/cart.ts`: `cartRouter.post('/items')`, `cartRouter.patch('/items/:productId')`, `cartRouter.delete('/items/:productId')`, `cartRouter.post('/discount')`, `cartRouter.put('/shipping')`, `cartView`.
- `src/routes/orders.ts`: `ordersRouter.post('/')`.
