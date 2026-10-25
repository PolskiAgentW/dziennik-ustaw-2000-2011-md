# Dziennik Ustaw 2000–2011 w Markdown

Teksty aktów z **Dziennika Ustaw** z lat 2000–2011, które API ELI Sejmu podaje tylko jako PDF, w Markdown i jako
drzewo jednostek w JSON, z metadanymi z API ELI.
*Texts of Polish Journal of Laws acts of 2000–2011 that the Sejm ELI API serves only as PDF, as Markdown and as
a JSON tree of units (art./§/ust./pkt/lit.), converted from the official PDFs.*

> **Nieoficjalne.** Teksty powstają przez automatyczną konwersję PDF-ów, więc mogą zawierać błędy.
> Wiążący jest PDF w Dzienniku Ustaw (link `source_pdf` w każdym pliku).

<!-- zbiory:start -->
**Wszystkie zbiory** (ten sam format plików, konwerter [eli2md](https://github.com/PolskiAgentW/eli2md)). Akty, które API ELI
podaje w HTML (np. większość Dziennika Ustaw 2012–2024), nie są tu powielane; wyjątek: teksty jednolite ustaw
z lat 2012–2024 w zbiorze od 2012 (zapas, gdy HTML z API nie działa).

| lata | Dziennik Ustaw | Monitor Polski |
|---|---|---|
| od 2012 | [GitHub](https://github.com/PolskiAgentW/dziennik-ustaw-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/dziennik-ustaw-md): od 2025 r. wszystkie, wcześniej 98 aktów bez HTML i 3 862 teksty jednolite ustaw; codziennie | [GitHub](https://github.com/PolskiAgentW/monitor-polski-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/monitor-polski-md): wszystkie z PDF (API nie ma HTML); codziennie |
| 2000–2011 | [GitHub](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/dziennik-ustaw-2000-2011-md): akty bez HTML w API | [GitHub](https://github.com/PolskiAgentW/monitor-polski-2000-2011-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/monitor-polski-2000-2011-md): wszystkie z PDF |
| 1990–1999 | [GitHub](https://github.com/PolskiAgentW/dziennik-ustaw-1990-1999-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/dziennik-ustaw-1990-1999-md): akty bez HTML w API (OCR skanów) | brak |
| 1918–1989 | [GitHub](https://github.com/PolskiAgentW/dziennik-ustaw-1918-1989-md) · [HF](https://huggingface.co/datasets/PolskiAgentW/dziennik-ustaw-1918-1989-md): akty bez HTML w API (OCR skanów; pomiar jakości w README) | brak |

Kolumny są we wszystkich zbiorach te same, więc lata można wczytać razem (nadal bez większości aktów, które API ELI podaje w HTML):

```python
from datasets import load_dataset

du = load_dataset("parquet", split="train", data_files=[
    "hf://datasets/PolskiAgentW/dziennik-ustaw-1918-1989-md/data/*.parquet",
    "hf://datasets/PolskiAgentW/dziennik-ustaw-1990-1999-md/data/*.parquet",
    "hf://datasets/PolskiAgentW/dziennik-ustaw-2000-2011-md/data/*.parquet",
    "hf://datasets/PolskiAgentW/dziennik-ustaw-md/data/*.parquet",
])  # 62 627 aktów (2026-10-11); Monitor Polski: monitor-polski-2000-2011-md + monitor-polski-md
```
<!-- zbiory:end -->

## Dlaczego

W latach 2000–2011 Dziennik Ustaw ma 23 604 akty. API ELI Sejmu (`api.sejm.gov.pl/eli`) podaje tekst HTML dla
3 984 z nich (głównie ustaw, obwieszczeń z tekstami jednolitymi i orzeczeń TK). Pozostałe 19 615, w większości
rozporządzenia, są tylko w PDF (stan z 2026-09-30). Tutaj jest ich tekst. Monitor Polski z tych samych lat:
[monitor-polski-2000-2011-md](https://github.com/PolskiAgentW/monitor-polski-2000-2011-md). Lata 1990–1999 (OCR skanów):
[dziennik-ustaw-1990-1999-md](https://github.com/PolskiAgentW/dziennik-ustaw-1990-1999-md).

PDF-y z tych lat to strony całych zeszytów: dwa łamy, kilka aktów na jednej stronie, czcionki QuarkXPress
z błędnym kodowaniem polskich liter (2000–2009). Konwerter [eli2md](https://github.com/PolskiAgentW/eli2md)
od wersji 0.6.8 poprawia kodowanie, czyta łamy po kolei i wycina akt z sąsiednich.

### Co daje `pdftotext` i podobne narzędzia

Fonty QuarkXPress „…PL” (2000–2009) mają polskie litery w kodach Mac Central European, a PDF opisuje je jako
Mac Roman. pdftotext, pdfplumber, pypdf, PyMuPDF i opendataloader-pdf dają więc „ROZPORZÑDZENIE”, „Si∏ Zbrojnych”,
„u˝ytkowej”, „zarzàdza si´” zamiast „ROZPORZĄDZENIE”, „Sił Zbrojnych”, „użytkowej”, „zarządza się”. Do tego
czytają oba łamy wierszami w poprzek strony albo bloki nie po kolei i biorą sąsiednie akty z tych samych stron.

Na 53 losowych aktach 2000–2009, które mają też HTML, odsetek słów oficjalnego tekstu odczytanych we właściwej
kolejności wynosi: pdftotext 0,641 (z `-layout` 0,406), pdfplumber 0,358, pypdf 0,621, PyMuPDF 0,676,
opendataloader-pdf 0,488, eli2md 0,993; z tabelą liter niżej pdftotext 0,825, PyMuPDF 0,858
([pomiar](https://github.com/PolskiAgentW/eli2md/blob/main/eval/extractors_2000_2009_s5207.md)). Poprawione
2026-10-01: pierwsza wersja podawała zaniżone liczby dla tych narzędzi (0,23–0,46), bo liczyła słowa w kolejności
blokami difflib zamiast najdłuższego wspólnego podciągu.

Jeśli masz już tekst wyciągnięty z takiego PDF-u, same litery naprawia tabela. Stosuj ją tylko do Dz.U. 2000–2009,
bo zmienia też prawdziwe „à”, „ç”, „è”, „ê” (np. we francuskich tekstach umów):

```python
TABLE = str.maketrans({bytes([b]).decode("mac_roman"): bytes([b]).decode("mac_latin2")
                       for b in range(128, 256) if bytes([b]).decode("mac_latin2") in "ąćęłńśźżĄĆĘŁŃŚŹŻ"})
tekst = tekst.translate(TABLE)
```

## Stan

Wszystkie lata 2000–2011 są w zbiorze (ostatni rocznik dodany 2026-10-02): 19 615 aktów bez HTML w API ELI
(stan API z 2026-09-30), z nich 19 614 z tekstem. Brakuje tekstu DU/2007/189, bo jego PDF w API ELI jest uszkodzony
(opis niżej, w części o OCR). Akty, którym API później doda HTML albo zmieni PDF, nie są tu aktualizowane.
Metadane (daty, tytuł, status, słowa kluczowe) uzgadnia z API ELI codziennie workflow „Metadane” (od 2026-10-14):
akty, którym w API zmienił się `changeDate`, dostają nowe metadane w `index.csv`, front matter, linii `# tytuł`
i `title` w `.json`; tekst zostaje z konwersji. Potem workflow „Eksport” odświeża plik w wydaniu „dane” i na HF.

<!-- stats:start -->
Stan na 2026-10-25 15:47 UTC (liczone z `index.csv`).

| rok | aktów w indeksie | przekonwertowanych | błędów |
|---|---:|---:|---:|
| 2000 | 1102 | 1102 | 0 |
| 2001 | 1511 | 1511 | 0 |
| 2002 | 1763 | 1763 | 0 |
| 2003 | 2007 | 2007 | 0 |
| 2004 | 2511 | 2511 | 0 |
| 2005 | 1951 | 1951 | 0 |
| 2006 | 1517 | 1517 | 0 |
| 2007 | 1591 | 1590 | 1 |
| 2008 | 1314 | 1314 | 0 |
| 2009 | 1492 | 1492 | 0 |
| 2010 | 1427 | 1427 | 0 |
| 2011 | 1429 | 1429 | 0 |

Akty ze stronami bez warstwy tekstowej (skany, grafiki): 3483, razem 66484 z 160308 stron. Tekst z OCR (oznaczony) ma 63985 z nich w 3424 aktach; treści pozostałych brak.
Akty ze stronami z dużymi obrazami (wzory, rysunki; ich treści brak): 5400.

Rodzaje aktów: Rozporządzenie 18305, Oświadczenie rządowe 633, Umowa międzynarodowa 342, Konwencja 98, Protokół 78, Obwieszczenie 39, Porozumienie 37, Uchwała 26, Postanowienie 25, Dokument wypowiedzenia 12, Traktat 11, Decyzja 2, Statut 2, Ustawa 1, Deklaracja 1, Akt 1, Układ 1.
Wersje konwertera: eli2md 0.6.18 (6505), eli2md 0.6.17 (5300), eli2md 0.6.22 (3564), eli2md 0.6.24 (2612), eli2md 0.6.62 (1013), eli2md 0.6.16 (319), eli2md 0.6.37 (294), eli2md 0.6.61 (6), eli2md 0.6.39 (1).
<!-- stats:end -->

## Zawartość

- `DU/<rok>/DU-<rok>-<pozycja>.md`: jeden akt. Front matter YAML z metadanymi ELI, potem tekst:
  `##### Art. N.` (albo `##### § N.`), akapity, `## Załącznik …`, przypisy `[^n]`. Ust., pkt i lit. zaczynają
  akapit. Cytowane przepisy (nowelizacje) nie są nagłówkami.
- `DU/<rok>/DU-<rok>-<pozycja>.json`: ten sam akt jako drzewo jednostek (`art`, `par` (§), `ust`, `pkt`, `lit`,
  `tir`) z numerem, ścieżką (`art_5/ust_2/pkt_3`), tekstem i dziećmi. Opis:
  [README eli2md](https://github.com/PolskiAgentW/eli2md#json-drzewo-jednostek-od-053).
- Cały zbiór w jednym pliku: `dziennik-ustaw-2000-2011-md.jsonl.gz` w wydaniu
  [„dane”](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md/releases/tag/dane) (jeden akt w wierszu)
  i Parquet na Hugging Face: [PolskiAgentW/dziennik-ustaw-2000-2011-md](https://huggingface.co/datasets/PolskiAgentW/dziennik-ustaw-2000-2011-md).
  Stan obu plików: 2026-10-13, 19 614 aktów.
- `index.csv`: jeden wiersz na akt, także nieudany: `eli, year, pos, type, title, announcement_date,
  promulgation, change_date, pdf_sha256, pages, words, no_text_pages, image_pages, ocr_pages, image_ocr_pages,
  status, error, converter, converted_at`.

Strony bez warstwy tekstowej czyta OCR (tesseract). Takich aktów jest dużo w 2000 r. (15 z pierwszych 50
przekonwertowanych: PDF-y „Distiller 4.0 for Macintosh; modified using iText” bez fontów). Tekst z OCR jest
oznaczony: przed stroną stoi notka `> [Strona 1 PDF nie ma czytelnej warstwy tekstowej. Tekst poniżej odczytał OCR …]`,
a akapity OCR są cytatami blokowymi (`> …`); w JSON to węzły `ocr`, bez podziału na jednostki. Wyjątek: 300 aktów,
których wszystkie strony są skanami (`no_text_pages` = `pages`; 297 z 2000 r.), od 2026-10-05 w eli2md 0.6.37: tekst
z OCR ma w nich jednostki (`##### § N.`, ust., pkt) i podpis, a w JSON drzewo jednostek, jak tekst z warstwy PDF.

## Jak dobre jest

Akty z lat 2000–2011, które mają HTML, służą za wzorzec: konwertuję ich PDF i porównuję słowa z oficjalnym tekstem
(`eval/evaluate.py` w eli2md) oraz drzewo jednostek z drzewem z HTML (`eval/tree_eval.py`). Każda wersja
konwertera jest mierzona na nowej, losowej próbie zapisanej przed oceną. Ostatnie pomiary (bez OCR; akty bez
warstwy tekstowej osobno):

| próba (wersja)     | aktów | treść R | treść P | przypisy R | drzewo treści R | drzewo treści P |
|--------------------|------:|--------:|--------:|-----------:|----------------:|----------------:|
| s5205 (0.6.11)     | 56    | 0.9984  | 0.9921  | 0.9777     | 0.9981          | 0.9990          |
| s5206 (0.6.12)     | 62    | 0.9991  | 0.9926  | 0.9696     | 0.9994          | 0.9997          |

R (recall) to odsetek słów oficjalnego tekstu odzyskanych we właściwej kolejności, P (precision) to odsetek słów
wyniku, które są w oficjalnym tekście. Szczegóły, wcześniejsze próby i historia zmian: README eli2md.

Ta miara dotyczy aktów, które mają HTML, czyli głównie ustaw. Akty w tym zbiorze (bez HTML, głównie rozporządzenia)
mogą wypadać inaczej, a dla nich wzorca nie ma. Miara bez wzorca, której używam dla Monitora Polskiego (słowa wyniku
wobec warstwy tekstowej PDF), tu nie działa: PDF aktu to całe strony zeszytu, razem z sąsiednimi aktami.

**Kontrola wzrokowa** (2026-09-30, eli2md 0.6.16/0.6.17, strona 1 PDF obok wyniku, 3 akty wylosowane z lat 2000–2002):
- DU/2001/1595 (oświadczenie rządowe, 2 łamy, strona wspólna z poz. 1596): akt wycięty poprawnie, łamy czytane
  po kolei, lista państw i dat kompletna. Bez uwag.
- DU/2000/1213 (skan, cały tekst z OCR, strona wspólna z tabelą poz. 1212): akt wycięty poprawnie, kolejność
  poprawna. Błędy OCR w znakach: „$ 2.” zamiast „§ 2.”, „w 8 1” zamiast „w § 1”, „tytutu” i „stużbę” zamiast
  „tytułu” i „służbę”. Akapit § 3 rozcięty na dwa na granicy łamów („wy-” / „płaca”).
- DU/2002/62 (rozporządzenie zmieniające tabele): **błąd kolejności**. Pod dwoma łamami jest tabela na całą
  szerokość strony. Fragmenty jej komórek („Stanowiska koordynują- radca generalny, …”) wpadają w środek
  akapitu „Na podstawie …”, a sama tabela jest spłaszczona i poszatkowana. Tak mogą wyglądać akty z tabelami
  w treści.

**Kontrola wzrokowa 2** (2026-10-01, eli2md 0.6.16/0.6.17, pierwsza i ostatnia strona PDF obok wyniku, 10 aktów
wylosowanych z lat 2000–2003, ziarno 5301):
- bez uwag, 5: DU/2002/1579, DU/2002/1537, DU/2003/951, DU/2001/927, DU/2003/1333 (schemat w załączniku to obraz;
  w tekście jest tylko znacznik, zgodnie z opisem wyżej);
- drobne błędy, 4: DU/2003/1192 (w załącznikach, które są rysunkami: dwa nagłówki stron w tekście, obrócony nagłówek
  „Załącznik nr 3” odczytany wspak, „Załącznik nr 2”, „12”, „14” nie są nagłówkami), DU/2000/831 (notka „Uwaga: Opis
  granic…” z dołu strony 4 przeniesiona na koniec aktu), DU/2002/1045 (tabela: kod „0105” w środku wielowierszowej
  komórki), DU/2002/947 (przypis „*” jako akapit między akapitami treści);
- poważny błąd, 1: DU/2003/2317. Ostatnia strona zeszytu (informacja wydawcy: gdzie kupić egzemplarze, reklamacje)
  była w tekście ostatniego aktu zeszytu. Dotyczyło to 276 aktów z lat 2000–2003; poprawione w eli2md 0.6.18
  (2026-10-01, opis w README eli2md). Gdy ostatnią stronę zeszytu czytał OCR, 0.6.18 usuwał ją całą razem z tą
  informacją: 15 aktów z 2000 r. straciło ostatnią stronę, 5 z nich było pustych (DU/2000/48, 681, 682, 786, 886);
  przeliczone w eli2md 0.6.19 (2026-10-01; razem +3064 słów; DU/2000/175 bez informacji wydawcy).

**Kontrola wzrokowa 3** (2026-10-01, eli2md 0.6.18, pierwsza i ostatnia strona PDF obok wyniku, 6 aktów wylosowanych
z opublikowanych lat 2004–2005, ziarno 5302):
- bez uwag, 4: DU/2005/2222, DU/2005/1642, DU/2004/2585, DU/2004/1210 (ostatni wiersz tabeli sygnałów nurka jest
  obrazem; w tekście jest tylko znacznik, zgodnie z opisem wyżej);
- drobne błędy, 2: DU/2005/1383 (tabela w załączniku 2: wiersze dwóch komórek przeplecione, „problewystąpień mów”),
  DU/2005/1323 (załącznik to skan tabeli obwodów głosowania: OCR miesza komórki, a wierszy 158–163 z ostatniej
  strony, wspólnej z poz. 1324, brak; jest tylko znacznik obrazu).

W żadnym z 19 obejrzanych aktów nie brakowało tekstu samego aktu (poza załącznikami-obrazami opisanymi wyżej) i nie
było na początku tekstu sąsiedniego aktu. Próba jest mała: odsetka błędnych aktów na tej podstawie nie da się ocenić.

Zmiana 2026-10-02 (eli2md 0.6.22, 365 aktów z lat 2000–2007, w których OCR nie dał tekstu z części stron). Strona skanu,
z której OCR nie odczytał użytecznego tekstu, jest czytana drugi raz z podaną rozdzielczością obrazu (wcześniej tesseract
jej nie dostawał i na stronach z tabelami gubił odstępy między słowami). Strony odczytane wcześniej czyta się jak przedtem.
W tych 365 aktach: strony z tekstem z OCR 9 707 → 10 417, słowa 3 298 694 → 3 475 234; żaden akt nie ma mniej stron
z OCR ani mniej słów. Aktów, w których OCR odczytał wszystkie strony bez warstwy tekstowej: 1 → 148. Strony z tabelami
odczytane teraz przez OCR są często słabej jakości (np. DU/2006/398 s. 44: tabela obrócona o 90°). DU/2007/189 nadal
nie ma tekstu: jego PDF w API ELI (413 392 257 bajtów) jest uszkodzony, także na serwerze: czytniki PDF go nie otwierają, a końcowa
część pliku jest przesunięta o 4 bity (po cofnięciu przesunięcia koniec pliku jest poprawny; sprawdzone 2026-10-02). DU/2009/1788: API ELI podaje pod `text.pdf` plik podpisu XAdES (XML) z PDF-em w środku (base64);
tekst jest z tego wewnętrznego PDF-u, `pdf_sha256` to skrót pliku z API (2026-10-02, jedyny taki plik na 42 102). Tak samo w 2008 r. (47 aktów, 2026-10-02): strony z OCR 1 483 → 1 646, słowa 366 628 → 433 060, żaden akt
nie ma mniej stron z OCR ani mniej słów; wszystkie strony bez warstwy tekstowej odczytane w 1 → 24 aktach.

Zmiana 2026-10-04 (eli2md 0.6.24, wszystkie 3 425 aktów ze stronami z OCR przeliczone ponownie; wcześniej miały wersje
0.6.16–0.6.22; porównanie `index.csv` przed i po):
- 0.6.23: gdy OCR skleił nagłówek strony zeszytu („Dziennik Ustaw Nr 186 — 10061 — Poz. 1149”) z dalszym tekstem
  w jeden akapit, konwerter usuwał cały akapit. Teraz usuwa tylko nagłówek. Np. w DU/2005/1016 (poprawki do konwencji
  SOLAS, s. 71–191 to skan) 26 stron odczytanych przez OCR nie miało w wyniku żadnego tekstu, teraz ma (+9 916 słów);
  w DU/2008/1149 wrócił pierwszy wiersz tabeli ze s. 12 („11 Zakręcie”, „12 Zażółkiew”; sprawdzone z PDF).
- 0.6.24: na stronach z OCR akt jest wycinany także wtedy, gdy OCR zgubił polskie litery w rodzaju aktu
  („ROZPORZADZENIE PREZESA RADY MINISTROW”). Mniej słów ma 5 aktów z 2000 r. (DU/2000/71, 89, 90, 101, 1343, razem
  −1 209): wcześniej plik zawierał też tekst sąsiednich pozycji ze wspólnej strony (np. w DU/2000/90 koniec załącznika
  poz. 89), teraz kończy się podpisem pod własnym aktem. Dla DU/2000/71 i 89 sprawdzone, że 0.6.23 dawał jeszcze stary
  wynik. W DU/2000/138 i 886 zniknął tylko numer pozycji (po 1 słowie).
- Skutek dla 3 425 aktów: więcej słów w 939 (razem +80 341), tyle samo w 2 479, mniej w 7 (razem −1 211). Stron
  odczytanych przez OCR jest o 4 więcej: w DU/2000/1047 (24 → 26) i DU/2002/337 (16 → 18) OCR czyta teraz strony
  formularzy, których warstwa tekstowa nie ma kodów Unicode (wcześniej w wyniku były z nich śmieci). DU/2007/189 nadal
  bez tekstu (uszkodzony PDF, opis wyżej). Pozostałe akty (bez stron z OCR) mają w polu `converter` wcześniejsze
  wersje: zmiany 0.6.23 i 0.6.24 dotyczą tylko stron z OCR.

Zmiana 2026-10-05 (eli2md 0.6.37, 300 aktów, których wszystkie strony są skanami: 297 z 2000 r., DU/2005/312, 779,
DU/2008/599; wcześniej 0.6.24). Od 0.6.26 eli2md czyta skany zeszytów sprzed 2012 r. inaczej: układa wiersze OCR
w kolejności łamów, wycina akt spośród sąsiednich na tych samych stronach i rozpoznaje jednostki, załączniki i podpis
(opis w README eli2md, wpisy 0.6.26–0.6.37).
- Pomiar na wszystkich 54 skanach wśród 205 pobranych aktów DU 2000 z HTML (`eval/scans_2000` w eli2md): treść
  R 0,946 → 0,981, P 0,592 → 0,952 (0.6.24 → 0.6.37); w żadnym z 54 aktów R ani P nie spadło o więcej niż 0,01.
- W 300 aktach (bez notek o OCR): słowa 331 640 → 330 254, wspólne 325 276. O ponad 30% mniej słów mają 3 akty:
  w DU/2000/57 i 159 nie ma już początku następnej pozycji (poz. 58, 160), w DU/2000/56 nie ma poz. 57, a fragment
  statutu z poz. 56, który był w pliku poz. 57, wrócił do poz. 56. Strony odczytane przez OCR: bez zmian.
- Struktura (z 300 aktów): z rozpoznanymi § lub artykułami 0 → 288, z podpisem 0 → 291, z załącznikiem 0 → 93.

Zmiana 2026-10-07 (eli2md 0.6.39, 326 aktów; opis w README eli2md, wpis 0.6.39):
- 325 umów międzynarodowych i podobnych aktów: w JSON „Artykuł N” jest węzłem `art` z polem `label`. Ich `.md` się
  nie zmienił, więc pole `converter` (w `.md`, `.json` i `index.csv`) zostaje wersją, w której powstał `.md`; `.json`
  zbudowano z niego kodem drzewa 0.6.39. Słowa w drzewach: zgubione 0.
- DU/2000/612 przeliczony z PDF (`converter`: eli2md 0.6.39): pozycja listy „50. Płynne metale” nie jest już nagłówkiem
  „§ 0.”. Innych zmian tekstu nie ma.
- Pozostałe akty bez zmian (drzewa wszystkich aktów zbioru i `.md` 300 skanów porównane w 0.6.38 i 0.6.39).

Zmiana 2026-10-10 (drzewo JSON, kod drzewa eli2md 0.6.50, 16 aktów; opis w README eli2md, wpis 0.6.50): nagłówki z
liczebnikiem słownym („DZIAŁ PIĄTY”, „CZĘŚĆ PIERWSZA”), z numerem rzymskim z wielką literą („DZIAŁ IVA”) albo z
odnośnikiem po numerze („Rozdział 5a[^28]”) są w `.json` węzłami `heading`. Wcześniej trafiały jako tekst do artykułu,
paragrafu albo punktu przed nimi. Nowych nagłówków: 39, żaden nie zniknął. `.md` się nie zmienił, więc pole
`converter` (w `.md`, `.json` i `index.csv`) zostaje wersją, w której powstał `.md`. Liczba jednostek bez zmian, słowa
w drzewach: zgubione 0. Pozostałe akty bez zmian (porównanie drzew wszystkich aktów zbioru). Wątpliwe: w formularzu
DU/2010/1581 „Dział I²⁾” i „Część C⁶⁾ ❏ …” są teraz nagłówkami, choć to części formularza (tekst ten sam, zmienia się
tylko typ węzła; odnośnik jest oddzielony od numeru: „I” + „²⁾”).

Zmiana 2026-10-13 (metadane, 90 aktów; tekst bez zmian): API ELI zmieniło po konwersji metadane 90 aktów (`changeDate`
2026-10-06–09). 79 z nich to rekordy z list niezgodności z drukiem, które ISAP poprawił po zgłoszeniu
([daty-ogloszenia](https://github.com/PolskiAgentW/daty-ogloszenia)). W tych aktach `index.csv` (`title`,
`announcement_date`, `promulgation`, `change_date`), front matter `.md` (klucze z API), linia `# tytuł` w `.md` i pole
`title` w `.json` mają teraz wartości z API z 2026-10-13. Zmienione: data ogłoszenia w 67 aktach, tytuł w 16, data
wejścia w życie w 1 (DU/2007/443: API jej już nie podaje). W 7 aktach zmienił się tylko `change_date`. Tekst, `converter`,
`pdf_sha256` i liczby w `index.csv` zostają z konwersji; PDF w API jest w tych 90 aktach ten sam (sha256). Porównanie
całego zbioru z listami API z 2026-10-13: poza tymi 90 aktami różnic w polach z API nie ma.

Zmiana 2026-10-24 (eli2md 0.6.61, 6 aktów; opis w README eli2md, wpis 0.6.61): oznaczenie litery albo punktu
z odnośnikiem na początku wiersza doklejało się do poprzedniego akapitu: „3) właściwe postanowienia dyrektywy 64/432/EEC
zostały spełnione; 4)[^3] powyższe zwierzęta…” (DU/2004/1637). Teraz zaczyna osobną jednostkę. Przeliczyłem ponownie
(z eli2md 0.6.18 na 0.6.61) 6 aktów, w których to wystąpiło: DU/2004/1637, DU/2005/937, 1947, 1997, 2019, 2143.
We wszystkich 6 słowa są te same co poprzednio. W DU/2004/1637 jedno takie miejsce zostało („…EC[^3]; 3)[^3] jest
zwierzęciem…”). W DU/2005/937 i 1947 jedna strona mniej jest liczona jako strona z obrazem (`image_pages` 2 → 1;
w 1947 OCR tej strony nie dawał tekstu).

Zmiana 2026-10-25 (eli2md 0.6.62, 1013 aktów; opis w README eli2md, wpis 0.6.62): nagłówek strony numeru
(„Dziennik Ustaw Nr 97 — 5671 — Poz. 563”) i notka drukarni („Zam. 688 Korekta III: 17.12.01”) zostawały w tekście aktu,
gdy nie były pierwszą linią strony (strona z obróconą tabelą, nagłówek pod notką, strona czytana przez OCR). Przeliczyłem
ponownie 1058 aktów, w których był taki nagłówek (4043 wystąpienia). Nowy plik zostaje tylko wtedy, gdy nagłówków jest
mniej, a reszta tekstu (bez nagłówków, notek, białych znaków i ogonków) różni się od poprzedniej najwyżej o 0,5% znaków.
Tak jest w 1013 aktach (w 698 z nich reszta tekstu ma te same znaki). W 1012 z nich nagłówka już nie ma; w DU/2005/1681
został w 156 miejscach z 157 (nad nagłówkiem jest tam linia drukarni „… #… Page …”). W 23 z 1013 aktów zmieniła się
w `index.csv` liczba stron z OCR albo z obrazem (bez linii nagłówka OCR czyta stronę inaczej), np. DU/2000/680
`ocr_pages` 120 → 119. W DU/2004/1728 odczyt OCR formularza ze strony 7 („% zł zł zł % % .. % % % zł”) nie przechodzi
już progu czytelności; zamiast niego jest notka, że treść tej strony jest tylko w PDF.
Bez zmian zostało 45 aktów. W 30 nagłówka w nowym pliku nie było, ale reszta tekstu różniła się o więcej niż 0,5%
(głównie OCR tabel i obrazów przeczytany inaczej; 3 obejrzane odczyty były lepsze, ale reguła ich nie przyjmuje):
DU/2000/299, DU/2001/533, 1616, DU/2002/272, 274, 326, 666, 1223, 1311, 1375, DU/2003/106, 259, 2305, DU/2004/494, 741,
1699, DU/2006/707, 848, DU/2007/695, DU/2008/809, 945, 1149, DU/2009/858, 1335, 1492, 1647, DU/2010/731, DU/2011/653,
681, 1191. W 15 nagłówek zostawał także w nowym pliku: w 10 nad nagłówkiem jest notka drukarni w innym brzmieniu
(„Korekta: 24.05.02”, „Zam. 432 — Łamanie: 7.07.03”; DU/2002/595, 659, 836, DU/2003/805, 1174, DU/2005/1014, 1016,
1034, DU/2006/1241, DU/2007/1390), w 3 linia drukarni „Dz. U. Nr 172 #1975 7/9/05 11:32 Page 10539” (DU/2005/1443,
1444, 1679), w DU/2000/129 OCR czyta znak przed nagłówkiem, w DU/2011/1427 przyczyny nie znam.

**Czego te liczby nie mówią:**

- Wzorzec HTML mają głównie ustawy, obwieszczenia i orzeczenia. Większość aktów w tym zbiorze to rozporządzenia,
  dla których wzorca nie ma. Ich jakość sprawdzam tylko kontrolą wzrokową (wyżej, 19 aktów).
- Akty bez warstwy tekstowej (OCR): na 3 takich aktach z prób (DU/2000/70, 179, 985) R 0.937–0.991,
  P 0.912–0.987. OCR myli znaki (np. „8 1.” zamiast „§ 1.”). Na stronach w dwóch łamach OCR potrafi pomieszać
  kolejność; koniec aktu z OCR bywa wtedy nieodcięty i do aktu trafia początek następnego.
- Tabele są spłaszczone do akapitów, wiersz po wierszu; komórki wielowierszowe mogą się przeplatać.
- Załączniki, które są grafiką albo skanem, czyta OCR; tabele z OCR są słabej jakości.

Błędy konwersji zgłaszaj w Issues. Najlepiej podaj pozycję aktu i fragment.

## Licencja

Akty normatywne i ich urzędowe projekty oraz urzędowe dokumenty i materiały nie są przedmiotem prawa
autorskiego (art. 4 pkt 1 i 2 ustawy o prawie autorskim i prawach pokrewnych). Pozostała zawartość
(indeks, skrypty): CC0 1.0.
