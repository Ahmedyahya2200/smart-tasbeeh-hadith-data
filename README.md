# Smart Tasbeeh — Hadith Data Mirror

Multi-language Hadith text data for the Smart Tasbeeh app's Hadith library feature.

This repo exists so the app can serve full Hadith text in more languages than the primary
upstream API covers, and so that data isn't a single point of failure — Cloudflare D1 stays
light (book/chapter metadata only, in Arabic + English), while full multi-language hadith text
is served from here via [jsDelivr](https://www.jsdelivr.com/)'s GitHub CDN
(`cdn.jsdelivr.net/gh/<owner>/smart-tasbeeh-hadith-data@<tag>/...`), the same delivery mechanism
used by the primary source below.

## Structure

- `manifest.json` — index of every `(book, language)` file: version, hadith count, last updated,
  and source attribution. Small (a few KB) so clients can cheaply check for updates before
  pulling a full per-language file.
- `{book_id}/{lang}.json` — full chapters + hadith text for one book in one language. Shape:

  ```json
  {
    "book_id": "muslim",
    "language": "en",
    "source": "eng-muslim (fawazahmed0/hadith-api mirror)",
    "version": 1,
    "chapters": [{"id": "muslim_ch_1", "title": "...", "sort_order": 1}],
    "hadiths": [{"id": "muslim_1", "chapter_id": "muslim_ch_1", "hadith_number": 1,
                 "narrator": "...", "text": "...", "grade": "..."}]
  }
  ```

## Books covered

- Sahih Muslim (`muslim`)
- Sunan Abu Dawud (`abudawud`)
- Jami at-Tirmidhi (`tirmidhi`)
- Sunan an-Nasa'i (`nasai`)
- Sunan Ibn Majah (`ibnmajah`)

**Riyad as-Salihin is not currently included.** The primary upstream source (below) no longer
hosts it. Adding it back is tracked as a separate, future data-sourcing effort.

## Sources & attribution

- **Primary mirrored source**: [fawazahmed0/hadith-api](https://github.com/fawazahmed0/hadith-api),
  released under [The Unlicense](https://unlicense.org) (public domain — verified before this repo
  was created). This repo mirrors that data verbatim, for the languages it already covers (see
  `manifest.json`), as a fallback source and to keep availability independent of a third-party
  API's uptime.
- **Languages with no upstream coverage are intentionally left out of this mirror rather than
  machine-translated.** Full Hadith text is never auto-translated for this project — that
  includes DeepL or any other MT tool. A language is only added here once a trusted human or
  scholarly translation has been sourced and reviewed.

## Versioning

Each entry in `manifest.json` carries a `version` integer, and releases of this repo are tagged
(`v1`, `v2`, ...) so jsDelivr's CDN cache naturally busts on a new tag. Consumers should compare
their locally cached version against the manifest before re-downloading a `(book, language)` file.
