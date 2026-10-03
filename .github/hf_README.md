---
language:
- pl
license: cc0-1.0
pretty_name: Dziennik Ustaw 2000–2011 (akty bez HTML w API ELI) w Markdown/JSON
size_categories:
- 10K<n<100K
tags:
- legal
- law
- poland
configs:
- config_name: default
  data_files:
  - split: train
    path: data/*.parquet
---

# Dziennik Ustaw 2000–2011 — teksty aktów, których API ELI nie ma w HTML

Nieoficjalne teksty aktów z Dziennika Ustaw z lat 2000–2011, które API ELI Sejmu podaje tylko jako PDF (19 615
z 23 604 aktów tych lat, w większości rozporządzenia), przekonwertowane z urzędowych PDF-ów otwartym konwerterem
[eli2md](https://github.com/PolskiAgentW/eli2md). Wszystkie lata 2000–2011 są w zbiorze
(19 614 aktów z tekstem; brakuje DU/2007/189, którego PDF w API ELI jest uszkodzony).

*Unofficial plain-text (Markdown) and structured (JSON tree of units) versions of the acts of the Polish Journal of
Laws (Dziennik Ustaw) of 2000–2011 that the Sejm ELI API serves only as PDF. Converted automatically; the PDF is the
binding text.*

**Dlaczego:** PDF-y z tych lat to strony całych zeszytów: dwa łamy, kilka aktów na stronie, a w 2000–2009 polskie
litery w kodach Mac CE opisanych jako Mac Roman. pdftotext, pdfplumber, pypdf, PyMuPDF dają „ROZPORZÑDZENIE”,
„Si∏ Zbrojnych” i mieszają łamy. Szczegóły i pomiar:
[repozytorium na GitHubie](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md#co-daje-pdftotext-i-podobne-narzędzia).

## Użycie

```python
from datasets import load_dataset
import json

ds = load_dataset("PolskiAgentW/dziennik-ustaw-2000-2011-md", split="train")
print(ds[0]["eli"], ds[0]["title"])
tree = json.loads(ds[0]["tree"])  # drzewo jednostek
```

## Kolumny

Jeden wiersz = jeden akt.
- `eli` (np. `DU/2003/991`), `year`, `pos`, `type`, `title`, `display_address`, `announcement_date`, `promulgation`,
  `entry_into_force`, `legal_status`, `keywords`, `change_date`, `source_pdf`, `pdf_sha256`: metadane z API ELI
  (bez poprawek, więc z jego błędami; `legal_status` — stan w chwili konwersji);
- `pages`, `words`, `no_text_pages`, `image_pages`, `ocr_pages`, `image_ocr_pages`: strony PDF, słowa wyniku,
  strony bez warstwy tekstowej (skany), strony z dużymi obrazami (ich treści brak), strony odczytane przez OCR,
  strony, na których OCR odczytał obraz tekstu;
- `markdown`: tekst aktu (tekst z OCR jako cytaty `> …` z notką przed stroną);
- `tree`: ten sam akt jako drzewo jednostek w JSON (tekst; opis formatu w
  [README eli2md](https://github.com/PolskiAgentW/eli2md#json-drzewo-jednostek-od-053));
- `converter`, `converted_at`: wersja eli2md i czas konwersji.

## Jakość

Wzorcem są akty z lat 2000–2011, które mają HTML (głównie ustawy). Na losowej próbie 62 takich aktów (eli2md 0.6.12)
odsetek słów oficjalnego tekstu odzyskanych we właściwej kolejności wynosi 0.9991, a odsetek słów wyniku obecnych
w oficjalnym tekście 0.9926. Akty w tym zbiorze (bez HTML, głównie rozporządzenia) nie mają wzorca. Sprawdzam je
kontrolą wzrokową: w 19 obejrzanych aktach nie brakowało tekstu samego aktu i nie było w nim sąsiedniego aktu;
błędy dotyczyły tabel (spłaszczone, komórki przeplecione), załączników-obrazów i OCR. Próba jest mała. Szczegóły
i lista błędów: [README na GitHubie](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md#jak-dobre-jest).

**To nie jest urzędowy tekst.** Wiążący jest PDF w Dzienniku Ustaw (`source_pdf`). Błędy konwersji zgłaszaj
w [Issues na GitHubie](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md/issues).

## Źródło i licencja

Źródło: [API ELI Sejmu](https://api.sejm.gov.pl/eli/acts/DU). Ten sam zbiór jako pliki `.md`/`.json`:
[github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md](https://github.com/PolskiAgentW/dziennik-ustaw-2000-2011-md).
Monitor Polski 2000–2011: [PolskiAgentW/monitor-polski-2000-2011-md](https://huggingface.co/datasets/PolskiAgentW/monitor-polski-2000-2011-md).
Akty normatywne i urzędowe dokumenty nie są przedmiotem prawa autorskiego (art. 4 pkt 1 i 2 ustawy o prawie
autorskim i prawach pokrewnych); pozostała zawartość: CC0 1.0.
