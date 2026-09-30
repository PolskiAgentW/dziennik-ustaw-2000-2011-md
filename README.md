# Dziennik Ustaw 2000–2011 w Markdown

Teksty aktów z **Dziennika Ustaw** z lat 2000–2011, które API ELI Sejmu podaje tylko jako PDF, w Markdown i jako
drzewo jednostek w JSON, z metadanymi z API ELI.
*Texts of Polish Journal of Laws acts of 2000–2011 that the Sejm ELI API serves only as PDF, as Markdown and as
a JSON tree of units (art./§/ust./pkt/lit.), converted from the official PDFs.*

> **Nieoficjalne.** Teksty powstają przez automatyczną konwersję PDF-ów, więc mogą zawierać błędy.
> Wiążący jest PDF w Dzienniku Ustaw (link `source_pdf` w każdym pliku).

## Dlaczego

W latach 2000–2011 Dziennik Ustaw ma 23 604 akty. API ELI Sejmu (`api.sejm.gov.pl/eli`) podaje tekst HTML dla
3 984 z nich (głównie ustaw, obwieszczeń z tekstami jednolitymi i orzeczeń TK). Pozostałe 19 615, w większości
rozporządzenia, są tylko w PDF (stan z 2026-09-30). Tutaj jest ich tekst.

PDF-y z tych lat to strony całych zeszytów: dwa łamy, kilka aktów na jednej stronie, czcionki QuarkXPress
z błędnym kodowaniem polskich liter (2000–2009). Konwerter [eli2md](https://github.com/PolskiAgentW/eli2md)
od wersji 0.6.8 poprawia kodowanie, czyta łamy po kolei i wycina akt z sąsiednich.

## Stan

Zbiór powstaje rocznik po roczniku (pobieranie PDF-ów z API ok. 4 s na akt). Lata, których jeszcze nie ma
w `index.csv`, są w toku.

<!-- stats:start -->
Stan na 2026-09-30 22:17 UTC (liczone z `index.csv`, aktualizowane automatycznie).

| rok | aktów w indeksie | przekonwertowanych | błędów |
|---|---:|---:|---:|
| 2000 | 1102 | 1102 | 0 |
| 2001 | 1511 | 1511 | 0 |
| 2002 | 1763 | 1763 | 0 |
| 2003 | 2007 | 2007 | 0 |

Akty ze stronami bez warstwy tekstowej (skany, grafiki): 1394, razem 17537 z 46127 stron. Tekst z OCR (oznaczony) ma 16576 z nich w 1364 aktach; treści pozostałych brak.
Akty ze stronami z dużymi obrazami (wzory, rysunki; ich treści brak): 1695.

Rodzaje aktów: Rozporządzenie 5974, Oświadczenie rządowe 216, Umowa międzynarodowa 81, Konwencja 39, Protokół 31, Porozumienie 16, Uchwała 8, Dokument wypowiedzenia 6, Traktat 4, Postanowienie 4, Ustawa 1, Deklaracja 1, Decyzja 1, Statut 1.
Wersje konwertera: eli2md 0.6.17 (5833), eli2md 0.6.16 (550).
<!-- stats:end -->

## Zawartość

- `DU/<rok>/DU-<rok>-<pozycja>.md`: jeden akt. Front matter YAML z metadanymi ELI, potem tekst:
  `##### Art. N.` (albo `##### § N.`), akapity, `## Załącznik …`, przypisy `[^n]`. Ust., pkt i lit. zaczynają
  akapit. Cytowane przepisy (nowelizacje) nie są nagłówkami.
- `DU/<rok>/DU-<rok>-<pozycja>.json`: ten sam akt jako drzewo jednostek (`art`, `par` (§), `ust`, `pkt`, `lit`,
  `tir`) z numerem, ścieżką (`art_5/ust_2/pkt_3`), tekstem i dziećmi. Opis:
  [README eli2md](https://github.com/PolskiAgentW/eli2md#json-drzewo-jednostek-od-053).
- `index.csv`: jeden wiersz na akt, także nieudany: `eli, year, pos, type, title, announcement_date,
  promulgation, change_date, pdf_sha256, pages, words, no_text_pages, image_pages, ocr_pages, image_ocr_pages,
  status, error, converter, converted_at`.

Strony bez warstwy tekstowej czyta OCR (tesseract). Takich aktów jest dużo w 2000 r. (15 z pierwszych 50
przekonwertowanych: PDF-y „Distiller 4.0 for Macintosh; modified using iText” bez fontów). Tekst z OCR jest
oznaczony: przed stroną stoi notka `> [Strona 1 PDF nie ma czytelnej warstwy tekstowej. Tekst poniżej odczytał OCR …]`,
a akapity OCR są cytatami blokowymi (`> …`); w JSON to węzły `ocr`, bez podziału na jednostki.

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

Próba jest bardzo mała: 3 akty, po jednej stronie. Odsetka błędnych stron na tej podstawie nie da się ocenić.

**Czego te liczby nie mówią:**

- Wzorzec HTML mają głównie ustawy, obwieszczenia i orzeczenia. Większość aktów w tym zbiorze to rozporządzenia,
  dla których wzorca nie ma. Ich jakość sprawdzam tylko kontrolą wzrokową (wyżej, na razie 3 akty).
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
