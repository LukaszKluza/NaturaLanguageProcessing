Model

- Checkpoint: intfloat/multilingual-e5-small
- Wymiar: 384, float32
- Maksymalna długość sekwencji: 512
- Normalizacja: L2 (`normalize_embeddings=True`), podobieństwo = iloczyn skalarny
- Prefiksy: `query: ` (zapytania i zadania symetryczne), `passage: ` (dokumenty w wyszukiwaniu)

## Dane

- Zbiór: PolEmo 2.0 (artifacts/polemo2_clean.parquet), tylko split train
- Próbka: 1000 losowych dokumentów, zachowany rozkład ze zbioru oryginalnego, random_state=42
- Teksty dłuższe niż okno (512 tokenów): 3.2%

## Koszt (CPU)

- Czas: 67.0 s / 1000 dok.
- 67.0 ms/dokument
- Rozmiar: 1.46 MB

## Jakość sąsiedztwa

- purity@1: 0.706
- purity@5: 0.665
- purity@10: 0.638
- (losowo: 0.2786)

## Wyszukiwanie (8 zapytań)

- Precision@5: 0.525
- Precision@10: 0.500

## Analiza jakościowa

- PCA: Dwa najbardziej znaczące komponenty wyjaśniają 17.92% wariancji; `zero` tworzy osobne skupisko, `plus`/`minus` się nakładają, `amb` leży pomiędzy.
- Dominujące błędy: wspólny temat (recenzje o przeciwnym sentymencie z cos 0.92-0.93), negacja (cos 0.96 dla „Pokój był/nie był czysty”).

## Zastosowanie

- Rekomendowany do: wyszukiwania semantycznego i RAG na długich tekstach (okno 512), grupowania tematycznego.
- Nie wybierać wyłącznie na podstawie tego lab do: klasyfikacji sentymentu, wykrywania negacji, dokładnego dopasowania nazw.

## Ograniczenia

- Dane: 1000 dokumentów, tylko 2 dziedziny (hotele, lekarze) + artykuły w `zero`.
- Benchmark: 8 zapytań, etykieta sentymentu jako przybliżenie relewancji, różnice < 0.1 nieistotne.
- Reprezentacja: słabo rozróżnia negację i przeciwny sentyment.
