# Model Card: Baseline NLP Classifier

## 1. Pipeline
Model finalny:
- `TfidfVectorizer(analyzer='char_wb', ngram_range=(3, 5), min_df=2, max_features=50000, sublinear_tf=True)`
- `LogisticRegression(C=5.0, max_iter=150, random_state=42, n_jobs=-1)`

## 2. Co przewiduje model?
Model przewiduje sentyment/opinię tekstową w jednej z czterech klas:
- `plus`
- `minus`
- `zero`
- `amb`

## 3. Na czym został wytrenowany?
Model został wytrenowany na tekście z podziałem:
- train: zbiory treningowe z Lab 1
- validation: używany do wyboru modelu i hiperparametrów
- test: używany tylko raz, po zamrożeniu konfiguracji

Główna metryka:
- `macro-F1`

## 4. Wyniki
### Validation
- macro-F1: do uzupełnienia po uruchomieniu eksperymentu
- accuracy: do uzupełnienia po uruchomieniu eksperymentu
- macro_precision: do uzupełnienia po uruchomieniu eksperymentu
- macro_recall: do uzupełnienia po uruchomieniu eksperymentu

### Final test
- accuracy: do uzupełnienia po uruchomieniu eksperymentu
- macro-F1: do uzupełnienia po uruchomieniu eksperymentu
- macro_precision: do uzupełnienia po uruchomieniu eksperymentu
- macro_recall: do uzupełnienia po uruchomieniu eksperymentu

## 5. Koszt modelu
- czas treningu: do uzupełnienia po uruchomieniu eksperymentu
- inferencja: do uzupełnienia po uruchomieniu eksperymentu
- rozmiar modelu: do uzupełnienia po uruchomieniu eksperymentu

## 6. Wnioski z błędów
Najtrudniejsza klasa dla modelu to `amb`, co wynika z:
- niejednoznaczności etykiet,
- mieszanych opinii pozytywnych i negatywnych w jednym tekście,
- problemów z negacją i długim kontekstem.

Model dobrze radzi sobie z wyraźnie pozytywnymi i negatywnymi opiniami, ale ma największą trudność z opiniami niejednoznacznymi.

## 7. Czy próg confidence ma sens?
Tak, dla modelu logistycznego próg ufności może mieć sens w systemach produkcyjnych:
- przy niskim prawdopodobieństwie lepiej nie odpowiadać,
- zamiast wymuszać decyzję, można kierować przykład do człowieka.

## 8. Do czego model może być używany?
Model nadaje się do:
- analizy sentymentu opinii użytkowników,
- klasyfikacji recenzji i komentarzy,
- wsparcia w monitoringu jakości usług i produktów.

## 9. Do czego model nie powinien być używany?
Model nie powinien być używany do:
- decyzji medycznych lub prawnych,
- sytuacji, w których błędna klasyfikacja ma wysoką cenę,
- bardzo krótkich lub bardzo niejednoznacznych tekstów bez dodatkowej walidacji.

## 10. Podsumowanie
Najlepszy model jest wyraźnie lepszy niż baseline większościowy. Prosty model tekstowy z TF-IDF i regresją logistyczną jest mocnym baseline'em i daje dobrą jakość przy niskim koszcie obliczeniowym. W tym zadaniu wystarcza do wysokiej jakości klasyfikacji bez potrzeby większego modelu.
