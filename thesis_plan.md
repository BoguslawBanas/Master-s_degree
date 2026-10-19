# Plan pracy magisterskiej: Postawy Polaków wobec Ukrainy w komentarzach na YouTube (2022–2026)

## 1. Tytuł roboczy

*Analiza sentymentu i emocji względem celów w polskich komentarzach na YouTube do wiadomości o Ukrainie, 2022–2026*

## 2. Pytania badawcze

- **PB1:** Jak zmieniał się sentyment komentarzy wobec każdego celu (zob. §3) między 2022 a 2026 rokiem?
- **PB2:** Czy zmiany odpowiadają kluczowym wydarzeniom (zob. §7) oraz trendom w sondażach opinii (CBOS i inne)?
- **PB3:** Czy kanały lewicowe i prawicowe różnią się poziomem i momentem tych zmian?
- **PB4 (opcjonalne):** Czy negatywność przesunęła się z rządów na Ukraińców jako ludzi? Jakie tematy ją napędzają (świadczenia, historia, rolnictwo, ...)?
- **PB5 (metodologiczne):** Jak porównuje się dostrojony model HerBERT z etykietowaniem zero-shot/few-shot przez LLM dla stanowiska względem celów w języku polskim?

## 3. Cele (aspekty)

Każdy komentarz może dotyczyć kilku celów (multi-label). Każdy cel w komentarzu otrzymuje własne stanowisko.

| # | Cel | Obejmuje |
|---|-----|----------|
| T1 | polski rząd | Rząd, ministrowie, prezydent, partie rządzące |
| T2 | ukraiński rząd | Zełenski, ministrowie, konkretni ukraińscy politycy oraz ich decyzje lub wypowiedzi |
| T3 | Ukraińcy (ludzie) | Ukraińcy jako naród, uchodźcy, migranci w Polsce, Ukraińcy w Ukrainie |
| T4 | Ukraina (państwo) | Ukraina jako kraj, jej armia, wysiłek wojenny, pomoc militarna, akcesja do UE/NATO |
| T5 | Rosja | Rosja jako państwo, Putin, armia rosyjska (cel kontrolny / referencyjny)

**Zasady graniczne (do dopracowania po pilotażu):**

- T2 vs T4: konkretne osoby lub decyzje → T2. Państwo, armia, pomoc lub akcesja → T4. Jeśli zgodność po pilotażu będzie niska, połączyć je.
- T1 zmienia znaczenie w czasie: rząd PiS do grudnia 2023, koalicja Tuska po tym terminie. Prezydent zmienił się z Dudy na Nawrockiego w sierpniu 2025. Zapisywać okres przy każdej etykiecie i interpretować wyniki według okresu i kanału.
- Zwykłe „Ukraińcy” liczy się jako T3.

**Etykiety dla każdej wzmianki o celu:**

- Nastawienie: `positive` / `negative` / `neutral`
- Emocja (opcjonalnie, podstawowy zestaw Plutchika): gniew, strach, smutek, radość, zaufanie, wstręt, zaskoczenie, oczekiwanie
- Etykieta tematyczna (opcjonalnie, głównie dla T3): korzyści/800+, rynek pracy, historia/Wołyń, rolnictwo/zboże, wojna/bezpieczeństwo, przestępczość, inne

## 4. Zbieranie danych

**Kanały:** TVN24, TVP Info, Republika, wPolsce24, a także ewentualnie Polsat News i RMF/Onet dla zachowania równowagi. Sprawdzić, czy każdy kanał istniał w całym okresie; wPolsce24 w obecnej formie nie istniał.

**Zasady kolekcji:**

- Wybieraj filmy według jasnej, powtarzalnej reguły, np. wszystkie filmy, których tytuł lub opis pasuje do listy słów kluczowych („Ukraina”, „Ukraińcy”, „uchodźcy”, „Zełenski”, „Wołyń”, „zboże”, „800+”, „Rosja”, „Putin”, ...) w każdym miesiącu.
- Nie wybieraj wyłącznie filmów z największą liczbą wyświetleń, albo przynajmniej zgłoś taki wybór jako źródło stronniczości.
- Użyj oficjalnego **YouTube Data API v3** (`commentThreads.list`, `comments.list`), aby uzyskać dokładne znaczniki czasu `publishedAt`.
- ⚠️ `youtube-comment-downloader` zwraca czasy względne („2 lata temu”), więc `time_parsed` jest dokładny tylko mniej więcej do roku dla starych komentarzy. To nie jest wystarczająco precyzyjne do analizy szeregów czasowych.
- Zapisuj: identyfikator komentarza, identyfikator filmu, kanał, datę publikacji filmu, datę publikacji komentarza, tekst, liczbę polubień, flagę odpowiedzi, identyfikator nadrzędny, pseudonimizowany identyfikator autora.
- Zarejestruj datę kolekcji. Usunięte lub moderowane komentarze są nieobecne, więc zgłoś to jako ograniczenie.

**Szacunek limitu:** `commentThreads.list` kosztuje 1 jednostkę na wywołanie i zwraca do 100 komentarzy. Domyślne 10 000 jednostek/dzień wystarcza na setki tysięcy komentarzy w ciągu kilku dni.

## 5. Przetwarzanie wstępne

1. Usuń duplikaty i spam (identyczne teksty, komentarze zawierające tylko link).
2. Wykryj język; zachowaj komentarze po polsku i zdecyduj, jak obsłużyć komentarze po ukraińsku i rosyjsku.
3. Znormalizuj tekst (URL-e, wzmianki, powtarzające się znaki). Zachowaj emoji, ponieważ niosą ładunek emocjonalny.
4. Przefiltruj według tematu, tak aby pozostały tylko komentarze związane z Ukrainą/Rosją (słowa kluczowe + BERTopic lub klasyfikator).
5. Pseudonimizuj autorów (zahaszowane identyfikatory) na potrzeby RODO.

## 6. Oznaczanie i modelowanie

### 6.1 Oznaczanie

- **Pilotaż:** około 500 komentarzy, aby sprawdzić częstości celów, równowagę klas i reguły brzegowe.
- **Zbiór złoty:** około 3 000–5 000 komentarzy, z warstwowaniem po kanale, roku i celu.
- **Drugi anotator** na około 300–500 komentarzach; raportuj kappę Cohena dla każdego celu i postawy.
- Napisz wytyczne do anotacji z przykładami; dołącz je jako aneks.
- Możliwe narzędzia: Label Studio, Doccano.

### 6.2 Modele

1. **Bazowe:** `Voicelab/herbert-base-cased-sentiment` (ogólny sentyment, brak celu), oparte na słowniku.
2. **Model główny:** dostrojenie HerBERT-a (`allegro/herbert-base-cased` lub large):
- detekcja celu (wieloe­tykietowa), potem stanowisko dla każdego celu, lub
- pojedynczy model wielozadaniowy z jedną głowicą klasyfikacyjną na cel.
3. **Podejście LLM:** etykietowanie zero-shot / few-shot (Bielik, modele z rodziny GPT), ewaluowane na zbiorze złotym.
4. **Opcjonalna destylacja:** użycie etykiet LLM do trenowania mniejszego modelu HerBERT, następnie porównanie.
5. **Emocje:** dostrojenie na PolEmo 2.0 i/lub własnych etykietach.

**Metryki:** makro-F1 dla każdego celu i stanowiska, macierze pomyłek, analiza błędów skupiona na sarkazmie i ironii.

## 7. Analiza

- **Szereg czasowy:** udział negatywnych/pozytywnych stanowisk na cel, na kanał, tygodniowo lub miesięcznie. Znormalizować względem wolumenu komentarzy.
- **Detekcja punktów zmiany:** np. `ruptures` (PELT, segmentacja binarna).
- **Dopasowanie do wydarzeń.** Kandydackie wydarzenia:
- luty 2022: pełnoskalowa inwazja, fala uchodźców
- 2023: spór o import zboża; przemówienie Zełenskiego w ONZ (wrzesień 2023)
- listopad 2023 – wczesny 2024: blokada granicy przez kierowców ciężarówek, protesty rolników
- grudzień 2023: zmiana rządu w Polsce; przejęcie TVP
- 2025: debata o ekshumacjach na Wołyniu, kampania prezydencka, debata o świadczeniach 800+ dla Ukraińców, weto Nawrockiego (sierpień 2025)
- Dodać kolejne wydarzenia do 2026
- **Walidacja względem sondaży:** korelować miesięczny sentyment wobec T3 i T4 z postawami CBOS wobec Ukraińców i poparciem dla pomocy Ukrainie. Testować wyprzedzenie/opóźnienie (korelacja krzyżowa, przyczynowość Grangera).
- **Porównanie kanałów:** TVN24 vs Republika/wPolsce24. TVP przed i po grudniu 2023 jako eksperyment naturalny.
- **Czynniki tematyczne:** BERTopic lub tagi tematyczne w obrębie negatywności T3.
- **Opcjonalnie:** wzorce między celami, np. antyukraińskie i prorosyjskie treści w tym samym komentarzu jako sygnał możliwych trolli lub kont dezinformacyjnych.

## 8. Ograniczenia i zagrożenia dla trafności

- Komentujący są grupą samodzielnie dobierającą się, niereprezentatywną dla społeczeństwa polskiego. Wyniki należy przedstawiać jako dyskurs online.
- Boty, trolle i skoordynowane kampanie.
- Moderacja i usuwanie przez kanały; efekt przeżywalności.
- Stronniczy dobór filmów (słowa kluczowe i popularność).
- Sarkazm i ironia; ukryte cele.
- T1 zmienia znaczenie, gdy zmieniają się rządy.
- Dostępność kanałów na całej badanej osi czasu.

## 9. Etyka i kwestie prawne

- Komentarze zawierają dane osobowe w rozumieniu RODO. Należy je pseudonimizować, minimalizować zakres przechowywanych danych i nie publikować surowych, możliwych do zidentyfikowania danych.
- Sprawdzić warunki korzystania z YouTube API (zasady dotyczące przechowywania danych i ich odświeżania).
- Zapytać wydział, czy potrzebna jest zgoda komisji etycznej.
- Udostępniać wyłącznie wyniki zagregowane lub identyfikatory komentarzy, a nie pełne treści z autorami.

## 10. Przegląd pipeline’u

```
YouTube Data API ─► surowe komentarze (per kanał/wideo)
│
▼
czyszczenie + usuwanie duplikatów + filtr językowy + pseudonimizacja
│
▼
filtr tematyczny (słowa kluczowe + BERTopic) ─► komentarze związane z Ukrainą/Rosją
│
├─► pilotażowa anotacja ─► wytyczne ─► zbiór złoty (+ 2. anotator, kappa)
│
▼
modele: baseline │ dostrojony HerBERT (target + stanowisko + emocja) │ LLM zero/few-shot
│
▼
ewaluacja na zbiorze złotym ─► najlepszy model oznacza cały korpus
│
▼
szeregi czasowe per target × kanał ─► punkty zmian ─► wydarzenia ─► korelacja z CBOS
│
▼
wyniki, dyskusja, ograniczenia
```

## 11. Proponowana struktura pracy

1. Wprowadzenie: motywacja, pytania badawcze, wkład
2. Tło: relacje polsko-ukraińskie 2022–2026; analiza sentymentu/stanowiska/emocji; polskie zasoby NLP
3. Prace powiązane: analiza opinii w mediach społecznościowych na temat wojny w Ukrainie, sentyment ukierunkowany na obiekt
4. Dane: pozyskiwanie, statystyki korpusu, kwestie etyczne
5. Adnotacja: schemat, wytyczne, zgodność
6. Metody: modele, trenowanie, ewaluacja
7. Wyniki I: ewaluacja modelu
8. Wyniki II: analiza szeregów czasowych, kanałów, wydarzeń i sondaży
9. Dyskusja i ograniczenia
10. Wnioski i przyszła praca

## 12. Przybliżony harmonogram

| Faza | Zadania |
|-------|--------|
| 1 | Przegląd literatury, ostateczne pytania badawcze, lista kanałów i słów kluczowych |
| 2 | Zbieranie danych przez API, czyszczenie, filtrowanie tematów |
| 3 | Pilotażowa anotacja, wytyczne, zbiór wzorcowy (gold set) |
| 4 | Trenowanie i ewaluacja modelu |
| 5 | Analiza: szeregi czasowe, wydarzenia, sondaże, kanały |
| 6 | Pisanie i poprawki |

## 13. Aktualny stan kodu (ten folder)

- `youtube_top_videos_by_month.py`: najlepsze filmy w miesiącu przez API. Przełącz wybór z „najczęściej oglądane” na „dopasowanie słów kluczowych” albo zgłoś stronniczość.
- `youtube_channel_top_comments_scraper.py`, `yt_scrapper.py`: używają `youtube-comment-downloader`, który daje tylko względne znaczniki czasu. Zastąpić `commentThreads.list` z API.
- `polish_sentiment.py`: bazowa analiza sentymentu HerBERT. Przydatne jako Baseline 1.
- `sentiment_translation.py`: tłumaczenie + VADER. Możliwa dodatkowa baza porównawcza, żeby pokazać, dlaczego potrzebne są natywne polskie modele.

## 14. Kolejne kroki

- [ ] Sfinalizować listę kanałów i listę słów kluczowych
- [ ] Przepisać kolektor komentarzy, aby używał YouTube Data API z dokładnymi znacznikami czasu
- [ ] Zebrać korpus pilotażowy (np. 2 kanały × 3 miesiące)
- [ ] Oznakować pilotażowe 500 komentarzy i doprecyzować granice celu
- [ ] Uzyskać raporty CBOS dotyczące postaw wobec Ukraińców, 2022–2026
- [ ] Omówić zakres z promotorem
