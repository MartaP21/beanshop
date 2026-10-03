# Przepływ od dodania produktu do złożenia zamówienia

Diagram pokazuje główną ścieżkę klienta. Logowanie jest warunkiem wykonania operacji na koszyku i złożenia zamówienia.

```mermaid
flowchart TD
    A["Zalogowany klient"] --> B["POST /api/cart/items<br/>Dodaj produkt i ilość"]
    B --> C{"Produkt istnieje,<br/>ilość 1–10 i stan magazynowy wystarcza?"}
    C -- "Nie" --> D["Błąd: 400 lub 404/409<br/>Koszyk bez zmian"]
    D --> B
    C -- "Tak" --> E["Produkt dodany do koszyka<br/>Odpowiedź zawiera podsumowanie"]

    E --> F{"Zmień ilość produktu?"}
    F -- "Tak" --> G["PATCH /api/cart/items/:productId"]
    G --> H{"Ilość jest poprawna<br/>i dostępna?"}
    H -- "Nie" --> I["Błąd: 400 lub 409"]
    I --> G
    H -- "Tak" --> J["Przelicz koszyk"]
    F -- "Nie" --> J

    J --> K{"Zastosuj kod rabatowy?"}
    K -- "Tak" --> L["POST /api/cart/discount"]
    L --> M{"Kod poprawny i spełnia warunki?"}
    M -- "Nie" --> N["Błąd 422<br/>Koszyk bez zmiany"]
    N --> K
    M -- "Tak" --> O["Rabat zapisany<br/>Przelicz koszyk"]
    K -- "Nie" --> O

    O --> P["PUT /api/cart/shipping<br/>STANDARD albo EXPRESS"]
    P --> Q["Podsumowanie: subtotal,<br/>rabat, dostawa, suma"]
    Q --> R{"Koszyk zawiera produkt<br/>i stan nadal wystarcza?"}
    R -- "Nie" --> S["Błąd 400/409<br/>Zamówienie nieutworzone"]
    S --> E
    R -- "Tak" --> T["POST /api/orders"]
    T --> U["Utwórz zamówienie NEW<br/>Zdejmij produkty ze stanu<br/>Wyczyść koszyk"]

    classDef error fill:#ffe0e0,stroke:#b00020
    class D,I,N,S error
```

## Uwagi dla testera

- **BR-03:** przy dodawaniu i zmianie ilości sprawdź limit 1–10 sztuk oraz stan magazynowy.
- **MOŻLIWY BŁĄD (BR-03):** walidacja `PATCH /api/cart/items/:productId` w `src/routes/cart.ts` ogranicza ilość tylko od góry (`max(10)`), więc może przyjąć 0 lub wartość ujemną zamiast wymagać minimum 1.
- **BR-04:** sprawdź koszt dostawy po rabacie, w tym próg 200,00 zł.
- **BR-05:** wymaganie mówi, że zastosowanie nowego kodu zastępuje poprzedni; kod w `src/routes/cart.ts` odrzuca już zastosowany kod i nie zastępuje go. **MOŻLIWY BŁĄD (BR-05).**
- **MOŻLIWY BŁĄD (BR-04):** `src/domain/pricing.ts` sprawdza darmową dostawę warunkiem `afterDiscount > 200`, więc dla dokładnie 200,00 zł zwraca płatną dostawę, mimo że wymaganie mówi „od 200,00 zł”.

Źródła:

- `docs/wymagania.md`: BR-03, BR-04, BR-05.
- `docs/architektura.md`: opis przepływu „złóż zamówienie”.
- `src/routes/cart.ts`: `cartRouter.post('/items')`, `cartRouter.patch('/items/:productId')`, `cartRouter.post('/discount')`, `cartRouter.put('/shipping')`.
- `src/routes/orders.ts`: `ordersRouter.post('/')`.
- `src/domain/pricing.ts`: `shippingCost()` i `priceCart()`.
