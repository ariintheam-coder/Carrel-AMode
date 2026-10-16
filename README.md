# Carrel

**A complete EPUB library and reader in one HTML file.** No build step, no dependencies, no server, no account. Drag books in; they stay on your device.

> *carrel* — the small enclosed desk in the library stacks, where one person reads alone.

[**Live demo**](https://smadgulkar.github.io/carrel/) · [**Download**](https://github.com/smadgulkar/carrel/releases/latest/download/index.html) · ~64 KB

<img src="docs/screenshots/library.png" alt="The Carrel library: book covers on a dark background, with search and sort controls above" width="100%">

---

## Why this exists

Every browser EPUB reader I could find bundles a ZIP library — JSZip, zip.js, or whatever epub.js pulls in. That made sense for a long time, because browsers couldn't unzip anything.

They can now. `DecompressionStream('deflate-raw')` shipped in Chrome 103, Firefox 113 and Safari 16.4, and an EPUB is just a ZIP full of XHTML. That removes the last reason to bundle anything at all.

So this is one file you can read top to bottom in an afternoon: a ZIP reader, an OPF parser, an IndexedDB library, and a paginator. Save it, double-click it, read a book on a plane.

## Use it

**Hosted** — open the [demo](https://smadgulkar.github.io/carrel/) and drag EPUBs onto the page.

**Local file** — download `index.html`, double-click. Works offline, forever, with no toolchain.

**Self-host** — copy `index.html` to any static host. That's the whole deployment.

```sh
git clone https://github.com/smadgulkar/carrel.git
cd carrel
python3 -m http.server        # then open http://localhost:8000
```

Serving over `http(s)` is recommended over `file://`: browsers disagree about whether IndexedDB works on `file://` origins, and a real origin means your library reliably persists.

## What it does

**Library** — drag and drop import, metadata and cover extraction, search, sort by title / author / date added / date opened / rating, 5-star ratings, reading progress on each cover. Books without cover art get a generated cloth binding, hashed to a set of book-cloth colours with gilt lettering, so a folder of random EPUBs still looks like a shelf.

**Reader** — paginated text (not scrolling), whole-book page numbers, table of contents from EPUB 3 `nav` or EPUB 2 NCX, six typefaces, three page themes, adjustable size / leading / margins, keyboard and swipe navigation, position saved per book.

**Privacy** — no network requests, no analytics, no accounts. The file makes zero outbound connections, which you can verify by reading it or by watching your network tab.

## Screenshots

<table>
<tr>
<td width="50%">
<img src="docs/screenshots/details.png" alt="Book detail panel showing cover, description, publisher, star rating and reading progress, over a dimmed library">
<br><sub><b>Everything from the file itself.</b> Description, publisher, date and cover come out of the book's own package file. Nothing is looked up online.</sub>
</td>
<td width="50%">
<img src="docs/screenshots/typefaces.png" alt="Reading view with the preferences panel open, showing six typeface options each rendered in its own font">
<br><sub><b>Six typefaces, three page themes.</b> Each option previews in the face it selects. System stacks only — no webfonts, no network.</sub>
</td>
</tr>
</table>

<!-- Add as you capture them: drop the file in docs/screenshots/ and move a row
     above this comment. `node tools/verify.mjs` lists anything missing.

<table>
<tr>
<td width="50%">
<img src="docs/screenshots/reader-night.png" alt="A page of text in night theme, light type on near-black">
<br><sub><b>Night theme.</b> The same page after dark.</sub>
</td>
<td width="50%">
<img src="docs/screenshots/bindings.png" alt="Generated cloth bindings in green, oxblood and navy with gold lettering">
<br><sub><b>Generated cloth bindings.</b> Books with no cover art get one, hashed to a set of real book-cloth colours.</sub>
</td>
</tr>
</table>

-->

## How it works

| Part | Approach |
|---|---|
| **Unzip** | Parses the ZIP central directory by hand, inflates entries with `DecompressionStream('deflate-raw')`. Handles stored and deflated entries, plus ZIP64. |
| **Parse** | `META-INF/container.xml` → OPF package → manifest, spine, Dublin Core metadata, cover. TOC from the EPUB 3 nav document, falling back to NCX. |
| **Store** | IndexedDB, holding the original file as a `Blob`. No base64 inflation. |
| **Render** | Chapter XHTML written into a same-origin iframe. Scripts and `on*` handlers stripped; images inlined as data URLs so it works from `file://` too. |
| **Paginate** | CSS multi-column with the gap set to twice the margin, so one column advance equals exactly one viewport width. Pages move by `transform`, not `scrollLeft` — no scroll container ambiguity, no clamping on the last page. Page count comes from measuring where a zero-width probe at the end of the text actually landed. |
| **Count** | Whole-book page numbers need every chapter laid out, so chapters are paginated in a hidden iframe in the background and cached in IndexedDB against a signature of everything that affects reflow. A calibrated estimate shows until the exact count lands. |

## Limitations

Known and mostly deliberate:

- **DRM'd books won't open.** They're encrypted. This project will not add circumvention — see [CONTRIBUTING](CONTRIBUTING.md).
- **Fixed-layout EPUBs** (comics, picture books) reflow into one column instead of honouring their fixed pages.
- **Reading position is a spine index plus a fraction**, not a proper [EPUB CFI](https://idpf.org/epub/linking/cfi/). It survives font changes to within about a page.
- **No annotations, highlights or bookmarks** yet.
- **EPUB only.** No MOBI, AZW3 or PDF.
- **Nested tables of contents** are flattened to one level.
- **Typefaces are system stacks**, so Literary and Minimal render differently on macOS and Windows. Loading real fonts would mean a network request.

## Browser support

Needs `DecompressionStream`: **Chrome / Edge 103+, Firefox 113+, Safari 16.4+**. The app detects this and says so rather than failing silently.

## Contributing

Issues and pull requests welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) — the short version is that it stays a single file with no dependencies, and CI enforces that.

## Prior art

Worth your time if this doesn't fit:

- [**Readk.it**](https://readk.it/) — also a single-file reader; bundles zip.js, aimed at publishers shipping one book
- [**epub.js**](https://github.com/futurepress/epub.js) / [**foliate-js**](https://github.com/johnfactotum/foliate-js) — libraries to build readers on, far more complete than this parser
- [**Readest**](https://readest.com/), [**Koodo Reader**](https://koodo.960960.xyz/) — full cross-platform apps with sync, notes and TTS
- [**Thorium**](https://www.edrlab.org/software/thorium-reader/) — the reference EPUB 3 implementation, with real accessibility support
- [**calibre**](https://calibre-ebook.com/) — the library manager everyone actually uses

## License

[MIT](LICENSE).
