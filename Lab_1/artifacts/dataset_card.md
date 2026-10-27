# Dataset Card: PolEmo 2.0 / KLEJ PolEmo2 In-Domain

## Dataset

Polskie recenzje konsumenckie z zadania `allegro/klej-polemo2-in` w benchmarku KLEJ. Zbiór zawiera teksty opinii z czterech sektorów: medycyna, hotelarstwo, produckty i uniwersytety. Dane te zostały ręcznie sklasyfikowane.

##### Źródło: [Hugging Face: allegro/klej-polemo2-in](https://huggingface.co/datasets/allegro/klej-polemo2-in).

## Task

Klasyfikacja semantyczna tekstu recenzji. Zbiór PolEmo 2.0 obejmuje recenzje z kilku domen. Wynik modelu nie powinien być automatycznie uogólniany na inne domeny ani na wszystkie polskie teksty.
__Input__: Opinie (`text` )
__Output__: Kategoria(`text`: `plus`, `minus`, `zero`, `amb`)

## Schema

| Kolumna            | Znaczenie                                                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `text`           | Oryginalna treść recenzji; podczas czyszczenia nie była modyfikowana.                                                      |
| `target`         | Etykieta sentymentu po mapowaniu na `plus`, `minus`, `zero`, `amb`.                                                   |
| `split`          | Oficjalny podział: `train`, `valid` lub `test`.                                                                        |
| `text_norm`      | Pomocnicza, znormalizowana kopia tekstu używana do audytu duplikatów; nie zastępuje `text`.                              |
| `duplicate_flag` | Flaga duplikatu na podstawie `text_norm`; w `valid` i `test` służy wyłącznie do oznaczenia, bez usuwania rekordów. |

## Labels

| Etykieta        | Znaczenie       |           Train |         Valid |          Test |
| --------------- | --------------- | --------------: | ------------: | ------------: |
| `plus`        | pozytywny       |           1,568 |           209 |           197 |
| `minus`       | negatywny       |           2,194 |           271 |           300 |
| `zero`        | neutralny       |             971 |           127 |           117 |
| `amb`         | niejednoznaczny |           1,050 |           116 |           108 |
| **Razem** |                 | **5,783** | **723** | **722** |

Mapowanie oryginalnych etykiet: `__label__meta_plus_m` → `plus`, `__label__meta_minus_m` → `minus`, `__label__meta_zero` → `zero`, `__label__meta_amb` → `amb`. Nazwy klas i ich znaczenia są zgodne z opisem zadania w karcie Hugging Face. `minus` jest najliczniejszą klasą w każdym lokalnym splicie, więc rozkład nie jest równomierny.

## Splits

Zachowano oficjalne podziały ze źródłowych plików `train.csv`, `valid.csv` i `test.csv`, nie tworzono nowego podziału. `valid` i `test` nie są modyfikowane ani pomniejszane podczas czyszczenia; ich oficjalne rozmiary pozostają odpowiednio 723 i 722 rekordy. Duplikaty w tych splitach są oznaczane flagą `duplicate_flag`. Lokalny `train` zawiera 5,783 rekordy po czyszczeniu.

## Data quality

- W danych nie znaleziono brakującego ani pustego tekstu lub etykiety: z `train` usunięto 0 rekordów.
- Z `train` nie usunięto duplikatów dokładnego tekstu: 0 rekordów.
- Porównanie znormalizowanych tekstów nie zwrócił rekordów współdzielonych między oficjalnymi splitami, z `train` nie usunięto z tego powodu żadnych rekordów..
- Klasy w każdym splicie są niezbalansowane, z przewagą klasy `minus`.
- Wynik porównania exact match nie wyklucza istnienia near-duplicates ani podobieństw semantycznych, jednak w tym eksperymencie takie oponie nie były usuwane.

## Cleaning

**W ramach czyszczenia danych przyjęto następujące reguły:**

* z `train` usunięto rekordy z brakującym tekstem lub etykietą,
* `valid` i `test` zachowano w oficjalnych rozmiarach; duplikaty w tych splitach oznaczono flagą `duplicate_flag`, nie usuwając ich,
* ze zbioru `train` usunięto dokładne duplikaty oraz wpisy, których znormalizowany tekst pojawiał się w innym splicie.

Normalizację (`NFKC`, ujednolicenie białych znaków, `casefold`, przycięcie brzegów) zastosowano wyłącznie na etapie audytu – oryginalna kolumna `text` pozostała nienaruszona, dołożona nowa kolumna `text_norm`.

W tym zbiorze żadna z tych operacji nie wymagała usunięcia wierszy. Dokładne statystyki i opis kroków zapisano w `artifacts/splits.json`, a oczyszczony zbiór wyeksportowano do `artifacts/polemo2_clean.parquet`.

## Leakage audit

Analiza dokładnego dopasowania `text_norm` nie wykryła tekstów występujących w więcej niż jednym oficjalnym splicie. Jest to wynik dla zastosowanej normalizacji, a nie dowód braku wszystkich form leakage.

## License / provenance

Dane pochodzą z projektu PolEmo 2.0 (*Kocoń et al., 2019*) oraz benchmarku KLEJ (Allegro) i zawierają polskie recenzje medyczne i hotelarskie.

- **Licencja:** `CC BY-NC-SA 4.0` (Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International).
- **Zasady:** Dozwolone użycie niekomercyjne, modyfikacja i redystrybucja pod warunkiem podania autorów oraz zachowania tej samej licencji.
- **Druga licencja:** metadane zbioru na Hugging Face deklarują `CC BY-SA 4.0` (Creative Commons Attribution-ShareAlike 4.0 International), która nie zawiera ograniczenia użycia komercyjnego. Jest to sprzeczne z licencją podaną w treści karty zbioru.
- **Przyjęte podejście:** do czasu wyjaśnienia rozbieżności ze źródłem należy stosować bardziej restrykcyjną interpretację `CC BY-NC-SA 4.0` i nie zakładać, że użycie komercyjne lub redystrybucja są dozwolone.

## Limitations

- Klasa`minus` dominuje liczebnie pozostałe więc accuracy może ukrywać słabsze wyniki dla mniejszych klas. Należy użyć dodatkowo też innych metrym np. macro-F1 i wyniki per klasa.
- Etykiety opinii, zwłaszcza `amb`, mogą zależeć od interpretacji danego człowieka, nienależy więc traktować ich bezwzględnie.
- Brak wykrytych exact duplicates nie wyklucza near-duplicates, leakage grupowego ani innych zależności między przykładami.
- Wyniki dotyczą konkretnego zadania i oficjalnych splitów. Nie gwarantują jakości w nowych domenach, okresach ani źródłach danych.
- Licencja wymaga wyjaśnienia z powodu rozbieżności między metadanymi a treścią karty źródłowej.
