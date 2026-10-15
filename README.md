# Arabic text in code: a short field guide

Practical notes for developers who store, search, compare or send Arabic text: Unicode ranges, vowel marks (harakat), normalization, digits, and one encoding trap that silently turns Arabic into question marks.

**Need a translation with harakat and a transliteration while you test?** https://translatearabic.org/?utm_source=github&utm_medium=referral&utm_campaign=links-2026-10

## Unicode ranges you will meet

| range | what |
|---|---|
| U+0600–U+06FF | Arabic (letters, harakat, Arabic-Indic digits) |
| U+0750–U+077F | Arabic Supplement |
| U+08A0–U+08FF | Arabic Extended-A |
| U+FB50–U+FDFF, U+FE70–U+FEFF | Presentation forms (legacy shaped glyphs; normalize them away with NFKC) |

## Harakat (short vowel marks)

Most Arabic text is written without them, so the same word can arrive with or without marks. Strip them before you compare or search.

| code point | name | mark |
|---|---|---|
| U+064B | fathatan | ً |
| U+064C | dammatan | ٌ |
| U+064D | kasratan | ٍ |
| U+064E | fatha | َ |
| U+064F | damma | ُ |
| U+0650 | kasra | ِ |
| U+0651 | shadda | ّ |
| U+0652 | sukun | ْ |
| U+0670 | superscript alef | ٰ |

```python
import re
HARAKAT = re.compile(r"[\u064B-\u0652\u0670]")
TATWEEL = "\u0640"   # the stretching character: كـتـاب

def strip_marks(s: str) -> str:
    return HARAKAT.sub("", s).replace(TATWEEL, "")

assert strip_marks("كِتَابٌ") == "كتاب"
```

## Light normalization for search

Users type the same word with different alef and ending forms. A common search key:

```python
ALEF = str.maketrans({"أ": "ا", "إ": "ا", "آ": "ا", "ٱ": "ا"})

def search_key(s: str) -> str:
    s = strip_marks(s).translate(ALEF)
    return s.replace("ى", "ي").replace("ة", "ه")
```

Use this only for matching. Store and display the original text: in real spelling ة and ه are different letters.

## Digits

Arabic text may use Western digits (0–9), Arabic-Indic digits (U+0660–U+0669: ٠١٢٣٤٥٦٧٨٩) or Extended Arabic-Indic digits used in Persian and Urdu (U+06F0–U+06F9). In Python, `int("١٢٣")` returns 123, and `str.isdigit()` is true for all three sets. Numbers are written left to right even inside right-to-left text.

## Direction

Arabic is right to left. In HTML set `dir="rtl"` and `lang="ar"` on the element, not only `text-align: right`, or punctuation and mixed English words end up on the wrong side. For user input of unknown direction, `dir="auto"` picks it from the first strong character.

## The question-mark trap

If Arabic arrives at your API as `????`, the text was converted to a legacy code page somewhere before it reached you. A frequent source on Windows is sending JSON from a shell with inline `-d '...'`. Write the request body to a file as UTF-8 and send the file:

```python
import json
with open("body.json", "w", encoding="utf-8") as f:
    json.dump({"text": "مرحبا"}, f, ensure_ascii=False)
```

```bash
curl -H "Content-Type: application/json; charset=utf-8" --data-binary @body.json https://example.com/api
```

To confirm what the server got, log the code points (`[hex(ord(c)) for c in s]`), not the rendered string: a terminal without an Arabic font can make correct text look broken and the other way round.

## Transliteration

There is no single standard. Academic texts use ALA-LC or DIN 31635 (ā, ḥ, ʿ). Arabs typing on Latin keyboards use the chat alphabet, where digits stand for letters English lacks (3 = ع, 7 = ح, 2 = ء). For the full letter table with examples, see [transliteration Arabic to English](https://translatearabic.org/transliteration/arabic-to-english/?utm_source=github&utm_medium=referral&utm_campaign=links-2026-10).

---

This repository holds documentation only. Corrections are welcome in Issues.
