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
Stan na 2026-10-01 06:17 UTC (liczone z `index.csv`, aktualizowane automatycznie).

| rok | aktów w indeksie | przekonwertowanych | błędów |
|---|---:|---:|---:|
| 2000 | 1102 | 1102 | 0 |
| 2001 | 1511 | 1511 | 0 |
| 2002 | 1763 | 1763 | 0 |
| 2003 | 2007 | 2007 | 0 |
| 2004 | 2511 | 2511 | 0 |
| 2005 | 1951 | 1951 | 0 |

Akty ze stronami bez warstwy tekstowej (skany, grafiki): 2055, razem 28639 z 80712 stron. Tekst z OCR (oznaczony) ma 27083 z nich w 1999 aktach; treści pozostałych brak.
Akty ze stronami z dużymi obrazami (wzory, rysunki; ich treści brak): 2888.

Rodzaje aktów: Rozporządzenie 10072, Oświadczenie rządowe 370, Umowa międzynarodowa 165, Konwencja 71, Protokół 53, Obwieszczenie 39, Porozumienie 25, Uchwała 15, Postanowienie 14, Traktat 8, Dokument wypowiedzenia 8, Decyzja 2, Ustawa 1, Deklaracja 1, Statut 1.
Wersje konwertera: eli2md 0.6.17 (6712), eli2md 0.6.18 (3589), eli2md 0.6.16 (544).
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
  (2026-10-01, opis w README eli2md). Zostaje 1 akt, w którym ten tekst pochodzi z OCR skanu (DU/2000/175).

W żadnym z 13 obejrzanych aktów nie brakowało tekstu aktu i nie było na początku tekstu sąsiedniego aktu. Próba jest
mała: odsetka błędnych aktów na tej podstawie nie da się ocenić.

**Czego te liczby nie mówią:**

- Wzorzec HTML mają głównie ustawy, obwieszczenia i orzeczenia. Większość aktów w tym zbiorze to rozporządzenia,
  dla których wzorca nie ma. Ich jakość sprawdzam tylko kontrolą wzrokową (wyżej, 13 aktów).
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
