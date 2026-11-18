# Model Card: Baseline NLP Classifier (PolEmo 2.0)

## 1. Pipeline

```python
Pipeline([
    ("vec", TfidfVectorizer(analyzer="char_wb", ngram_range=(3, 5), min_df=2,
                            max_features=50_000, sublinear_tf=True)),
    ("cls", LogisticRegression(C=15.0, max_iter=150, random_state=42)),
])
```

- Reprezentacja: TF-IDF na n-gramach znakowych (3–5) w obrębie słów, 50 000 cech.
- Klasyfikator: regresja logistyczna, C wybrane na validation.
- Plik: `artifacts/char_wb_+_LogisticRegression.joblib`

## 2. Co przewiduje model?

Sentyment całej recenzji w jednej z 4 klas:

- `plus`: opinia pozytywna,
- `minus`: opinia negatywna,
- `zero`: tekst neutralny (w praktyce często teksty prasowe/informacyjne),
- `amb`: opinia niejednoznaczna / mieszana.

Dla każdej klasy zwraca też prawdopodobieństwo (`predict_proba`).

## 3. Dane i protokół

- Zbiór: `allegro/klej-polemo2-in` (PolEmo 2.0-IN), recenzje m.in. lekarzy, hoteli, produktów; dane po czyszczeniu z `artifacts/polemo2_clean.parquet`.
- Oryginalny podział: **train = 5783**, **validation = 723**, **test = 722**.
- Model finalny wytrenowano tylko na **train (5783 tekstów)**.
- Główna metryka: **macro-F1** (każda klasa ma równą wagę, słaba klasa `amb` nie chowa się za `minus`). Pomocnicze: accuracy, macro-precision, macro-recall, bootstrap CI 95%.

## 4. Wyniki

### Validation (modele trenowane na train)

| model                                |        accuracy |        macro-F1 | macro-precision |    macro-recall |
| ------------------------------------ | --------------: | --------------: | --------------: | --------------: |
| Majority baseline                    |           0.375 |           0.136 |           0.094 |           0.250 |
| Stratified baseline                  |           0.268 |           0.231 |           0.231 |           0.231 |
| TF-IDF word + LinearSVC              |           0.819 |           0.784 |           0.791 |           0.782 |
| TF-IDF word + LR                     |           0.827 |           0.795 |           0.817 |           0.786 |
| **TF-IDF char_wb + LR (C=15)** | **0.827** | **0.803** | **0.807** | **0.801** |

Macro-F1 wybranego modelu na validation: 0.803 (95% CI: 0.775–0.831).

### Final test (ten sam model, trenowany na train+valid)

- accuracy: **0.823**
- macro-F1: **0.802** (95% CI: 0.774–0.832)
- macro-precision: 0.807
- macro-recall: 0.798

| accuracy | macro-F1 |                                              macro_f1_ci95 | macro-precision | macro-recall | infer_ms/doc |
| -------- | -------: | ---------------------------------------------------------: | --------------: | -----------: | ------------ |
| 0.83241  | 0.815354 | (0.815354409394541, 0.7865427432220571, 0.844480330414648) |        0.820348 |     0.810915 | 0.927827     |

Wynik na teście (0.802) jest praktycznie równy wynikowi na validation (0.803), więc nie ma oznak przeuczenia do zbioru walidacyjnego.
Poprawa względem majority baseline: **+0.666 macro-F1** (0.136 → 0.802).

## 5. Koszt modelu

- Czas treningu: około **10.3 s** (5783 tekstów, CPU, cały pipeline z TF-IDF).
- Inferencja: około **0.77 ms / dokument** (mediana z 3 powtórzeń).
- Rozmiar zapisanego pipeline'u: **3.18 MB**.
- Dla porównania word-TF-IDF + LR: około 3 s treningu i około 5× szybsza inferencja przy macro-F1 niższym o 0.008 na validation (różnica w granicach CI).

## 6. Wnioski z błędów

- Najtrudniejsza klasa to **`amb`** (F1 ≈ 0.59 na teście). Najczęstsze pomyłki to `amb → minus` i `amb ↔ plus`.
- Analiza cech pokazała skróty: tokeny typu `pap`, `proc`, `np` silnie wskazują klasę `zero`. Model rozpoznaje „styl prasowy”, a nie neutralny sentyment. Słowa tematyczne (`doktora`, `lekarzy`) mają wagę w `plus`, więc model częściowo uczy się domeny, a nie samego sentymentu.
- Długość tekstu: brak monotonicznej zależności. Najsłabsze są najkrótsze teksty, a najlepszy koszyk zawdzięcza wynik dużemu udziałowi łatwej klasy `zero`.
- Odporność na szum (brak polskich znaków, literówki, wielkość liter): macro-F1 spada z 0.803 do 0.759 (−0.044), podczas gdy word-TF-IDF traci −0.149. To główny powód wyboru n-gramów znakowych.

## 7. Czy próg confidence ma sens?

Tak. Odrzucanie predykcji z `max(predict_proba) < próg` podnosi accuracy na zaakceptowanych przykładach kosztem coverage (dla LR: próg 0.5 daje accuracy ≈ 0.93 przy coverage ≈ 0.72). Po kalibracji `CalibratedClassifierCV` próg 0.6 daje accuracy ≈ 0.90 przy coverage ≈ 0.82.

## 8. Do czego model może być używany?

- Wstępna, automatyczna klasyfikacja sentymentu polskich recenzji z domen podobnych do PolEmo (usługi medyczne, hotele, produkty).
- Monitoring trendów opinii.
- Mocny, tani baseline do porównań z transformerami i LLM-ami.
- System z progiem confidence: pewne przypadki automatycznie, niepewne do człowieka.

## 9. Do czego model nie powinien być używany?

- Do decyzji o konkretnych osobach (np. ocena lekarza na podstawie pojedynczej opinii).
- Do tekstów spoza domeny i spoza języka polskiego bez ponownej walidacji.
- Do wykrywania ironii, sarkazmu i subtelnych opinii mieszanych (klasa `amb` ma F1 ≈ 0.59).
- Jako detektor „neutralności”: klasa `zero` w dużej mierze wykrywa teksty prasowe.

## 10. Podsumowanie

Prosty pipeline char-TF-IDF + regresja logistyczna (C=15), trenowany na samym train, osiąga macro-F1 = 0.802 na teście, czyli +0.666 względem majority baseline, przy treningu około 10 s, inferencji poniżej 1 ms/dokument i rozmiarze około 3 MB. Na tym etapie większy model nie jest konieczny. Ewentualny transformer musiałby przede wszystkim poprawić klasę `amb` i opinie mieszane, bo to tam leży większość błędów.
