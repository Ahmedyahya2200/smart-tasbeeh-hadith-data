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
    "book_id": "sahih_muslim",
    "language": "en",
    "source": "eng-muslim (fawazahmed0/hadith-api mirror)",
    "chapters": [{"id": "sahih_muslim_ch_1", "title": "...", "sort_order": 1}],
    "hadiths": [{"id": "sahih_muslim_1", "chapter_id": "sahih_muslim_ch_1",
                 "hadith_number": 1, "narrator": null, "text": "...", "grade": "..."}]
  }
  ```

  `book_id` values match production Cloudflare D1's `hadith_books.id` exactly (verified against
  a production dump) — **not** the upstream API's own short book keys. This matters: the Flutter
  client looks up files here using `book.id` as synced from D1.

## Books covered

- Sahih Muslim (`sahih_muslim`)
- Sunan Abu Dawud (`sunan_abi_dawud`)
- Jami at-Tirmidhi (`jami_tirmidhi`)
- Sunan an-Nasa'i (`sunan_nasai`)
- Sunan Ibn Majah (`sunan_ibn_majah`)

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

## Versioning — why `@master` is never used for content

Every release of this repo is an immutable git tag (`v1`, `v2`, ...). **Consumers must never
fetch `manifest.json` or any `{book_id}/{lang}.json` via `@master`** — jsDelivr's GitHub CDN
resolves a branch tag to a commit through its own backend, separately from its edge cache, and
that resolution can stay stuck on an old commit indefinitely after a push; purging the edge
cache does not force it to re-check. Immutable tags don't have this problem: the content behind
`@v1` can never change, so jsDelivr caches it aggressively and correctly forever.

- `latest.json` (root, **the one file still fetched via `@master`** — intentionally, see below):
  `{"tag": "v3"}` — tells clients which tag is current.
- Every other file is fetched at the resolved tag, e.g.
  `cdn.jsdelivr.net/gh/<owner>/smart-tasbeeh-hadith-data@v3/manifest.json`.
- `latest.json` is tiny and changes rarely, so its own `@master` staleness (same class of lag
  described above) is an acceptable, cheap-to-purge tradeoff — clients should treat it as
  eventually-consistent (cache the last known-good tag, never block on a fresh fetch) rather than
  assume a just-published tag is visible immediately.
- Each entry in `manifest.json` also carries its own `version` integer (independent of the repo
  tag) — this is what the client compares against its local SQLite cache to decide whether to
  re-download a specific `(book, language)` file, and works the same regardless of which tag it
  was fetched from.
- **Never move or reuse a tag.** Bumping content behind an existing tag reintroduces the exact
  staleness bug this scheme exists to avoid. Always create a new tag.
