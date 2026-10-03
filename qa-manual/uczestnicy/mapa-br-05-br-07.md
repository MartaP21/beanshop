# Mapa reguł BR-05–BR-07

Założenia przykładów: data testu **03.10.2026**, dostawa **STANDARD**, a „oczekiwana kwota” oznacza kwotę `total` po rabacie i dostawie. Ceny produktów pochodzą z danych startowych w `src/store.ts`.

| BR-05: jeden kod, zastępowanie i wielkość liter | BR-06: definicje oraz ważność kodów | BR-07: ponowna kontrola minimum przy zmianie koszyka |
|---|---|---|
| **Koszyk:** 1 × Espresso Blend 500 g (54,99 zł)<br>**Kod:** najpierw `KAWA10`, potem `JESIEN15`<br>**Oczekiwanie:** działa tylko `JESIEN15`; rabat 8,25 zł, kwota **61,73 zł**. | **Koszyk:** 2 × Kolumbia Supremo 250 g (79,98 zł)<br>**Kod:** `KAWA10`<br>**Oczekiwanie:** rabat 8,00 zł, kwota **86,97 zł**. | **Koszyk:** 1 × Brazylia Santos 1 kg + 1 × Filtry papierowe 100 szt. (109,98 zł)<br>**Kod:** `MINUS20`, następnie usunięcie filtrów<br>**Oczekiwanie:** po zmianie subtotal 89,99 zł, kod usunięty, komunikat dla klienta, kwota **104,98 zł**. |
| **Koszyk:** 2 × Kolumbia Supremo 250 g (79,98 zł)<br>**Kod:** najpierw `KAWA10`, potem `MINUS20`<br>**Oczekiwanie:** działa tylko `MINUS20`? **Nie:** kod nie spełnia minimum 100 zł, więc należy pokazać błąd i pozostawić `KAWA10`; kwota **86,97 zł**. | **Koszyk:** 1 × Dzbanek do przelewów 600 ml (100,00 zł)<br>**Kod:** `MINUS20`<br>**Oczekiwanie:** rabat 20,00 zł, kwota **94,99 zł**. | **Koszyk:** 1 × Espresso Blend 500 g + 1 × Kolumbia Supremo 250 g + 1 × Filtry papierowe 100 szt. (114,97 zł)<br>**Kod:** `MINUS20`, następnie usunięcie Espresso Blend<br>**Oczekiwanie:** po zmianie subtotal 59,98 zł, kod usunięty i pokazany komunikat, kwota **74,97 zł**. |
| **Koszyk:** 1 × Dzbanek do przelewów 600 ml (100,00 zł)<br>**Kod:** `kAwA10`<br>**Oczekiwanie:** kod rozpoznany bez względu na wielkość liter, rabat 10,00 zł, kwota **104,99 zł**. | **Koszyk:** 1 × Młynek ręczny Stalowy (159,00 zł)<br>**Kod:** `JESIEN15`<br>**Oczekiwanie:** kod ważny 03.10.2026, rabat 23,85 zł, kwota **150,14 zł**. | **Koszyk:** 1 × Dzbanek do przelewów 600 ml + 1 × Filtry papierowe 100 szt. (119,99 zł)<br>**Kod:** `MINUS20`, następnie usunięcie dzbanka<br>**Oczekiwanie:** po zmianie subtotal 19,99 zł, kod usunięty i pokazany komunikat, kwota **34,98 zł**. |
| **Koszyk:** 1 × Espresso Blend 500 g (54,99 zł)<br>**Kod:** próba `MINUS20` po `KAWA10`<br>**Oczekiwanie:** `MINUS20` odrzucony, bo subtotal jest mniejszy niż 100,00 zł; poprzedni kod pozostaje, kwota **64,48 zł**. | **Koszyk:** 1 × Espresso Blend 500 g (54,99 zł)<br>**Kod:** `LATO25`<br>**Oczekiwanie:** kod odrzucony jako wygasły, brak rabatu, kwota **69,98 zł**. | **Koszyk:** dowolny subtotal poniżej 100,00 zł<br>**Kod:** `MINUS20`<br>**Oczekiwanie:** nie można zastosować kodu; komunikat o zbyt niskiej wartości produktów i brak rabatu. |

## Pytania do PO

1. Czy „zastosowanie nowego kodu zastępuje poprzedni” oznacza, że nowy kod ma najpierw zastąpić poprzedni, a dopiero potem być walidowany? W przykładzie z `KAWA10` i niespełniającym minimum `MINUS20` przyjąłem, że poprzedni kod pozostaje.
2. Jaka dokładnie treść komunikatu ma być pokazana po automatycznym usunięciu kodu zgodnie z BR-07?
3. Czy kontrola daty `JESIEN15` ma być wykonywana według daty serwera, czy daty ustawionej przez środowisko testowe?
4. Czy kwota oczekiwana w scenariuszach kodów ma zawsze obejmować dostawę standardową, czy PO chce osobno porównywać subtotal po rabacie i `total`?

## Rozbieżności do sprawdzenia

- **MOŻLIWY BŁĄD (BR-05):** `src/routes/cart.ts`, funkcja obsługi `POST /api/cart/discount`, dopisuje kolejny kod do `cart.codes` zamiast zastępować poprzedni.
- **MOŻLIWY BŁĄD (BR-07):** w `src/routes/cart.ts` zmiana pozycji koszyka nie usuwa automatycznie kodu, gdy po zmianie przestaje być spełnione minimum, ani nie pokazuje komunikatu.

## Źródła

- `docs/wymagania.md`: BR-05–BR-08.
- `src/store.ts`: `seedProducts()`.
- `src/domain/discounts.ts`: `findDiscount()` i definicje kodów.
- `src/domain/pricing.ts`: `priceCart()` i wyliczanie rabatu.
- `src/routes/cart.ts`: obsługa kodów i zmian koszyka.
